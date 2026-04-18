# 组合挂起函数

> 本节介绍组合挂起函数的各种方法。

## 默认顺序执行

假设我们有两个挂起函数，它们执行一些有用的操作（如远程服务调用或计算）：

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L)  // 模拟有用的工作
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L)  // 模拟有用的工作
    return 29
}
```

如果我们需要**顺序**调用它们——先 `doSomethingUsefulOne` **然后** `doSomethingUsefulTwo`，并计算它们的总和怎么办？在实践中，当我们需要第一个函数的结果来决定是否需要调用第二个函数或如何调用它时，我们会这样做。

我们使用正常的顺序调用，因为协程中的代码就像常规代码一样，**默认是顺序的**：

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        val two = doSomethingUsefulTwo()
        println("结果是 ${one + two}")
    }
    println("完成耗时 $time ms")
}

// 输出：
// 结果是 42
// 完成耗时 2017 ms
```

两个函数总共耗时约 2 秒。

---

## 使用 async 并发执行

如果 `doSomethingUsefulOne` 和 `doSomethingUsefulTwo` 的调用之间没有依赖关系，我们想要**并发**执行它们以更快获得结果怎么办？这就是 [async] 发挥作用的地方。

从概念上讲，[async] 就像 [launch]。它启动一个单独的协程，这是一个与其他所有协程并发工作的轻量级线程。区别在于 `launch` 返回 [Job] 且不携带任何结果值，而 `async` 返回 [Deferred] —— 一个轻量级的非阻塞 future，表示稍后提供结果的承诺。你可以在 deferred 值上使用 `.await()` 获取其最终结果，但 `Deferred` 也是 `Job`，所以如果需要可以取消它。

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("结果是 ${one.await() + two.await()}")
    }
    println("完成耗时 $time ms")
}

// 输出：
// 结果是 42
// 完成耗时 1017 ms
```

这快了两倍，因为两个协程并发执行。**注意：协程的并发总是显式的。**

---

## 惰性启动的 async

可选地，[async] 可以通过将其 `start` 参数设置为 [CoroutineStart.LAZY] 来变为惰性。在这种模式下，只有当其结果被 [await][Deferred.await] 需要，或调用其 `Job` 的 [start][Job.start] 函数时，它才会启动协程：

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
        
        // 一些计算
        one.start()  // 启动第一个
        two.start()  // 启动第二个
        
        println("结果是 ${one.await() + two.await()}")
    }
    println("完成耗时 $time ms")
}
```

这产生了相同的输出，但提供了更精细的控制：你可以决定何时开始每个协程。

---

## async 风格函数

我们可以定义异步风格的函数，使用带有显式 `CoroutineScope` 参数的 `async` 协程构建器：

```kotlin
// doSomethingUsefulOne 和 doSomethingUsefulTwo 的异步风格函数
fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}
```

> 这种风格的使用**不推荐**，因为它是错误倾向的。详见下文。

这样的函数可以像这样使用：

```kotlin
// 不推荐的做法
fun main() {
    val time = measureTimeMillis {
        val one = somethingUsefulOneAsync()
        val two = somethingUsefulTwoAsync()
        
        runBlocking {
            println("结果是 ${one.await() + two.await()}")
        }
    }
    println("完成耗时 $time ms")
}
```

### 为什么不推荐？

考虑如果 `val one = somethingUsefulOneAsync()` 和 `one.await()` 之间某行代码抛出异常会发生什么。程序将永远无法等待 `two`，但它已经在后台运行，尽管没有人需要它的结果。这会导致资源泄漏。

**推荐做法**：使用结构化并发，确保在作用域内进行并发操作。

---

## 使用 async 进行结构化并发

让我们使用 `somethingUsefulOneAsync` 和 `somethingUsefulTwoAsync` 的示例，提取它们的并发执行到一个函数中，该函数返回包含两个结果的对象：

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

// 将两个结果封装在数据类中
data class Result(val one: Int, val two: Int)

suspend fun concurrentSum(): Result = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    Result(one.await(), two.await())
}

fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        println("结果是 ${concurrentSum()}")
    }
    println("完成耗时 $time ms")
}
```

### 取消行为

如果 `concurrentSum` 内部的代码失败并抛出异常，所有在其作用域内启动的协程都会被取消：

```kotlin
import kotlinx.coroutines.*

suspend fun failConcurrentSum(): Int = coroutineScope {
    val one = async { 
        try {
            delay(Long.MAX_VALUE)  // 模拟长时间工作
            13
        } finally {
            println("第一个子协程被取消")
        }
    }
    val two = async<Int> { 
        println("第二个子协程抛出异常")
        throw ArithmeticException("除以零")
    }
    one.await() + two.await()
}

fun main() = runBlocking<Unit> {
    try {
        failConcurrentSum()
    } catch (e: ArithmeticException) {
        println("捕获异常: $e")
    }
}

// 输出：
// 第二个子协程抛出异常
// 第一个子协程被取消
// 捕获异常: ArithmeticException: 除以零
```

当 `two` 失败时，`one` 被取消，确保资源不会泄漏。

---

## 正确的 async 错误处理

使用 `async` 时，异常会在调用 `await()` 时抛出：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    val deferred = async {
        throw ArithmeticException("计算错误")
    }
    
    try {
        deferred.await()
    } catch (e: ArithmeticException) {
        println("捕获异常: $e")
    }
}
```

如果使用 `CoroutineStart.LAZY`，异常会在调用 `start()` 或 `await()` 时抛出。

---

## 最佳实践

### ✅ 推荐

```kotlin
// 在协程作用域内使用 async
suspend fun fetchUserData(): UserData = coroutineScope {
    val profile = async { fetchProfile() }
    val settings = async { fetchSettings() }
    UserData(profile.await(), settings.await())
}
```

### ❌ 不推荐

```kotlin
// 返回 Deferred 的全局异步函数（破坏结构化并发）
fun fetchProfileAsync() = GlobalScope.async { fetchProfile() }
```

---

## 总结

| 方式           | 用途                           | 耗时     |
|----------------|--------------------------------|----------|
| 顺序调用       | 有依赖关系，需要前一个结果     | 2秒      |
| async 并发     | 无依赖，并发执行               | 1秒      |
| async LAZY     | 精细控制启动时机               | 1秒      |

**记住**：
- 默认是顺序执行
- 使用 `async` 显式并发
- 在 `coroutineScope` 内使用 `async` 保证结构化并发
- 避免使用 `GlobalScope.async`

---

## 下一步

- [协程上下文与调度器](04-coroutine-context-and-dispatchers.md) - 深入理解协程上下文
- [异步流 Flow](05-flow.md) - 处理数据流
