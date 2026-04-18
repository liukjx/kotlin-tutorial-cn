# 协程异常处理

> 本节介绍异常处理和异常时的取消。

## 异常传播

协程构建器有两种类型：
- **自动传播异常**：[launch] —— 类似 Java 的未捕获异常
- **暴露异常给用户**：[async]、[produce] —— 需要通过 `await()` 或 `receive()` 处理

```kotlin
import kotlinx.coroutines.*

@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    val job = GlobalScope.launch {
        println("从 launch 抛出异常")
        throw IndexOutOfBoundsException()
    }
    job.join()
    println("已 join 失败的 job")
    
    val deferred = GlobalScope.async {
        println("从 async 抛出异常")
        throw ArithmeticException()
    }
    try {
        deferred.await()
    } catch (e: ArithmeticException) {
        println("捕获 ArithmeticException")
    }
}
```

**输出**：

```text
从 launch 抛出异常
Exception in thread "..." java.lang.IndexOutOfBoundsException
已 join 失败的 job
从 async 抛出异常
捕获 ArithmeticException
```

---

## CoroutineExceptionHandler

自定义未捕获异常的处理：

```kotlin
import kotlinx.coroutines.*

@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler 捕获 $exception")
    }
    val job = GlobalScope.launch(handler) {
        throw AssertionError()
    }
    val deferred = GlobalScope.async(handler) {
        throw ArithmeticException()  // 不会打印，需要 await
    }
    joinAll(job, deferred)
}
```

**输出**：

```text
CoroutineExceptionHandler 捕获 java.lang.AssertionError
```

> 注意：`CoroutineExceptionHandler` 只对**根协程**有效，子协程的异常会传播到父级。

---

## 取消与异常

取消使用 `CancellationException`，被所有处理器忽略：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        val child = launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                println("子协程被取消")
            }
        }
        yield()
        println("取消子协程")
        child.cancel()
        child.join()
        yield()
        println("父协程未取消")
    }
    job.join()
}
```

**输出**：

```text
取消子协程
子协程被取消
父协程未取消
```

---

## 异常聚合

当多个子协程失败时，"第一个异常获胜"，其他异常作为被抑制异常附加：

```kotlin
import kotlinx.coroutines.*

@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("捕获 $exception，被抑制: ${exception.suppressed.contentToString()}")
    }
    val job = GlobalScope.launch(handler) {
        launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                throw ArithmeticException()  // 第二个异常
            }
        }
        launch {
            delay(100)
            throw IOException()  // 第一个异常
        }
        delay(Long.MAX_VALUE)
    }
    job.join()
}
```

**输出**：

```text
捕获 java.io.IOException，被抑制: [java.lang.ArithmeticException]
```

---

## 监督 (Supervision)

当需要**单向取消**时使用监督。子协程失败不会取消父级和其他子级。

### SupervisorJob

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val supervisor = SupervisorJob()
    with(CoroutineScope(coroutineContext + supervisor)) {
        val firstChild = launch(CoroutineExceptionHandler { _, _ -> }) {
            println("第一个子协程失败")
            throw AssertionError("第一个子协程被取消")
        }
        val secondChild = launch {
            firstChild.join()
            println("第一个子协程取消: ${firstChild.isCancelled}，但第二个仍然活跃")
            try {
                delay(Long.MAX_VALUE)
            } finally {
                println("第二个子协程因 supervisor 被取消而取消")
            }
        }
        firstChild.join()
        println("取消 supervisor")
        supervisor.cancel()
        secondChild.join()
    }
}
```

### supervisorScope

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    try {
        supervisorScope {
            val child = launch {
                try {
                    println("子协程睡眠中")
                    delay(Long.MAX_VALUE)
                } finally {
                    println("子协程被取消")
                }
            }
            yield()
            println("从作用域抛出异常")
            throw AssertionError()
        }
    } catch (e: AssertionError) {
        println("捕获断言错误")
    }
}
```

### 监督作用域中的异常

子协程需要自己处理异常：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler 捕获 $exception")
    }
    supervisorScope {
        val child = launch(handler) {
            println("子协程抛出异常")
            throw AssertionError()
        }
        println("作用域正在完成")
    }
    println("作用域已完成")
}
```

**输出**：

```text
作用域正在完成
子协程抛出异常
CoroutineExceptionHandler 捕获 java.lang.AssertionError
作用域已完成
```

---

## 最佳实践

### ✅ 推荐

```kotlin
// 使用 try-catch 处理 async 异常
try {
    deferred.await()
} catch (e: Exception) {
    // 处理异常
}

// 使用 CoroutineExceptionHandler 处理 launch 异常
val handler = CoroutineExceptionHandler { _, e ->
    log("未捕获异常", e)
}
GlobalScope.launch(handler) { ... }
```

### ❌ 不推荐

```kotlin
// 在子协程中使用 CoroutineExceptionHandler（无效）
launch(CoroutineExceptionHandler { ... }) {
    // 这个 handler 不会被调用
}

// 忽略 async 的异常
val deferred = async { throw Exception() }
// 没有 await()，异常被忽略
```

---

## 异常处理总结

| 场景 | 处理方式 |
|------|----------|
| launch 根协程 | CoroutineExceptionHandler |
| async 根协程 | try-catch 包裹 await() |
| 子协程 | 异常传播到父级 |
| 监督作用域 | 子协程自己处理 |

---

## 下一步

- [共享状态与并发](08-shared-mutable-state-and-concurrency.md) - 线程安全问题
