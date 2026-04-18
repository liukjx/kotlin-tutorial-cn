# 协程上下文与调度器

> 协程总是在某个上下文中执行，由 [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/) 类型的值表示。

协程上下文是一组各种元素。主要元素是协程的 [Job]（我们之前见过）和它的调度器，本节将介绍调度器。

---

## 调度器与线程

协程上下文包含一个**协程调度器**（见 [CoroutineDispatcher]），它确定相应协程使用哪个或哪些线程执行。协程调度器可以将协程执行限制到特定线程、调度到线程池，或让它不受限制地运行。

所有协程构建器如 [launch] 和 [async] 都接受可选的 [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/) 参数，可用于显式指定新协程的调度器和其他上下文元素。

### 示例

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch { // 父协程的上下文，main runBlocking 协程
        println("main runBlocking      : 我在 ${Thread.currentThread().name} 线程工作")
    }
    launch(Dispatchers.Unconfined) { // 不受限制 -- 将在主线程工作
        println("Unconfined            : 我在 ${Thread.currentThread().name} 线程工作")
    }
    launch(Dispatchers.Default) { // 将被调度到 DefaultDispatcher
        println("Default               : 我在 ${Thread.currentThread().name} 线程工作")
    }
    launch(newSingleThreadContext("MyOwnThread")) { // 将获得自己的新线程
        println("newSingleThreadContext: 我在 ${Thread.currentThread().name} 线程工作")
    }
}
```

**输出**（顺序可能不同）：

```text
Unconfined            : 我在 main 线程工作
Default               : 我在 DefaultDispatcher-worker-1 线程工作
newSingleThreadContext: 我在 MyOwnThread 线程工作
main runBlocking      : 我在 main 线程工作
```

### 调度器说明

| 调度器 | 用途 |
|--------|------|
| `Dispatchers.Default` | 默认调度器，CPU 密集型任务 |
| `Dispatchers.IO` | I/O 密集型任务（网络、文件） |
| `Dispatchers.Main` | UI 线程（Android、JavaFX、Swing） |
| `Dispatchers.Unconfined` | 不受限制，在调用者线程启动 |
| `newSingleThreadContext("name")` | 创建专用线程 |

---

## Unconfined vs Confined 调度器

[Dispatchers.Unconfined] 协程调度器在调用者线程启动协程，但只持续到第一个挂起点。挂起后，它由被调用的挂起函数完全决定的线程恢复。Unconfined 调度器适用于既不消耗 CPU 时间也不更新限制到特定线程的共享数据（如 UI）的协程。

另一方面，调度器默认从外部 [CoroutineScope] 继承。[runBlocking] 协程的默认调度器特别限制到调用者线程，因此继承它具有将执行限制到此线程并具有可预测 FIFO 调度的效果。

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch(Dispatchers.Unconfined) {
        println("Unconfined      : 我在 ${Thread.currentThread().name} 线程工作")
        delay(500)
        println("Unconfined      : delay 后在 ${Thread.currentThread().name} 线程工作")
    }
    launch { // 父协程的上下文
        println("main runBlocking: 我在 ${Thread.currentThread().name} 线程工作")
        delay(1000)
        println("main runBlocking: delay 后在 ${Thread.currentThread().name} 线程工作")
    }
}
```

**输出**：

```text
Unconfined      : 我在 main 线程工作
main runBlocking: 我在 main 线程工作
Unconfined      : delay 后在 kotlinx.coroutines.DefaultExecutor 线程工作
main runBlocking: delay 后在 main 线程工作
```

> Unconfined 调度器是一个高级机制，在某些不需要或会产生不良副作用的协程调度情况下可能有帮助。不应在一般代码中使用。

---

## 线程间跳转

使用 [withContext] 在协程中切换上下文：

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() {
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("在 ctx1 启动")
                withContext(ctx2) {
                    log("在 ctx2 工作")
                }
                log("回到 ctx1")
            }
        }
    }
}
```

**输出**：

```text
[Ctx1 @coroutine#1] 在 ctx1 启动
[Ctx2 @coroutine#1] 在 ctx2 工作
[Ctx1 @coroutine#1] 回到 ctx1
```

---

## 上下文中的 Job

协程的 [Job] 是其上下文的一部分，可以使用 `coroutineContext[Job]` 表达式检索：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    println("我的 Job 是 ${coroutineContext[Job]}")
}
```

在调试模式下输出类似：

```
我的 Job 是 "coroutine#1":BlockingCoroutine{Active}@6d311334
```

> [CoroutineScope] 中的 [isActive] 只是 `coroutineContext[Job]?.isActive == true` 的便捷快捷方式。

---

## 协程的子级

当在另一个协程的 [CoroutineScope] 中启动协程时，它通过 [CoroutineScope.coroutineContext] 继承其上下文，新协程的 [Job] 成为父协程 Job 的**子级**。当父协程被取消时，其所有子协程也会递归取消。

### 覆盖父子关系

可以通过以下方式显式覆盖：

1. 启动协程时显式指定不同作用域（如 `GlobalScope.launch`）
2. 为新协程传递不同的 `Job` 对象作为上下文

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    val request = launch {
        // 使用独立 Job 启动
        launch(Job()) {
            println("job1: 我在自己的 Job 中运行，独立执行！")
            delay(1000)
            println("job1: 我不受 request 取消的影响")
        }
        // 继承父上下文
        launch {
            delay(100)
            println("job2: 我是 request 协程的子级")
            delay(1000)
            println("job2: 如果父 request 被取消，这行不会执行")
        }
    }
    delay(500)
    request.cancel()
    println("main: request 取消后谁存活了？")
    delay(1000)
}
```

---

## 父级责任

父协程总是等待其所有子协程完成。父级不显式跟踪所有子级，而是使用 Job 引用自己。好处是子协程可以在任何线程启动，异步工作也不会阻塞父级。

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    val request = launch {
        repeat(3) { i ->
            launch {
                delay((i + 1) * 200L)
                println("协程 $i 完成")
            }
        }
        println("request: 我启动了 ${children.count()} 个子协程")
    }
    request.join() // 等待 request 及其所有子协程完成
    println("main: 现在所有协程都完成了")
}

val CoroutineScope.children: Sequence<Job>
    get() = coroutineContext[Job]?.children?.asSequence() ?: emptySequence()
```

---

## 组合上下文元素

使用 `+` 运算符组合上下文元素：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch(Dispatchers.Default + CoroutineName("test")) {
        println("我在 ${coroutineContext[CoroutineName]} 中工作")
    }
}
```

---

## 协程命名

使用 [CoroutineName] 为协程命名，便于调试：

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking<Unit> {
    log("Started main coroutine")
    
    val v1 = async(CoroutineName("v1coroutine")) {
        delay(500)
        log("Computing v1")
        252
    }
    
    val v2 = async(CoroutineName("v2coroutine")) {
        delay(500)
        log("Computing v2")
        6
    }
    
    log("The result is ${v1.await() * v2.await()}")
}
```

---

## 取消与异常传播

### 取消传播

- 取消是双向传播：子协程取消会传播到父级，父级取消会传播到所有子级
- 使用 `SupervisorJob` 阻止取消向上传播

### SupervisorJob

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    val supervisor = SupervisorJob()
    withContext(CoroutineScope(supervisor).coroutineContext) {
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
        println("子协程完成，但作用域仍活跃")
    }
}
```

---

## 最佳实践

### 选择正确的调度器

| 场景 | 推荐调度器 |
|------|-----------|
| CPU 密集型计算 | `Dispatchers.Default` |
| 网络/文件 I/O | `Dispatchers.IO` |
| UI 更新 | `Dispatchers.Main` |
| 后台任务 | 自定义线程池或 `Dispatchers.Default` |

### 避免阻塞主线程

```kotlin
// ❌ 不好：阻塞主线程
fun main() = runBlocking {
    Thread.sleep(1000)  // 阻塞
}

// ✅ 好：使用挂起函数
fun main() = runBlocking {
    delay(1000)  // 不阻塞
}
```

### 正确释放资源

```kotlin
// 专用线程使用后必须释放
newSingleThreadContext("MyThread").use { ctx ->
    withContext(ctx) {
        // 工作
    }
}
```

---

## 下一步

- [异步流 Flow](05-flow.md) - 处理数据流
- [异常处理](07-exception-handling.md) - 更详细的异常处理机制
