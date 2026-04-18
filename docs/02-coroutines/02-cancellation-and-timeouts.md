# 取消与超时

> 取消让你可以在协程完成之前停止它。这对于不再需要的工作（如用户关闭窗口或导航离开）非常有用。

取消通过 [`Job`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/) 句柄工作，它代表协程的生命周期及其父子关系。`Job` 允许你检查协程是否活跃，并允许你取消它及其子协程（根据[结构化并发](01-coroutines-basics.md#协程作用域与结构化并发)的定义）。

---

## 取消协程

当在协程的 `Job` 句柄上调用 [`cancel()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) 函数时，协程被取消。

[协程构建器函数](01-coroutines-basics.md#协程构建器函数)如 [`.launch()`](01-coroutines-basics.md#coroutinescope-launch) 返回 `Job`。[`.async()`](01-coroutines-basics.md#coroutinescope-async) 函数返回 [`Deferred`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/)，它实现了 `Job` 并支持相同的取消行为。

### 示例

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val job = launch {
            println("协程已启动")
            try {
                delay(Long.MAX_VALUE)  // 无限挂起
            } catch (e: CancellationException) {
                println("协程被取消: $e")
                throw e  // 始终重新抛出取消异常！
            }
            println("这行永远不会执行")
        }
        
        delay(100)  // 等待协程启动
        job.cancel()  // 取消协程
    }
    println("所有协程已完成")
}
```

> 捕获 `CancellationException` 可能会破坏取消传播。如果必须捕获它，请重新抛出以让取消正确传播。

---

## 取消传播

[结构化并发](01-coroutines-basics.md#协程作用域与结构化并发)确保取消一个协程也会取消其所有子协程。这防止子协程在父协程停止后继续工作。

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val parentJob = launch {
            launch {
                println("子协程 1 已启动")
                try {
                    awaitCancellation()
                } finally {
                    println("子协程 1 已被取消")
                }
            }
            launch {
                println("子协程 2 已启动")
                try {
                    awaitCancellation()
                } finally {
                    println("子协程 2 已被取消")
                }
            }
        }
        
        delay(100)  // 等待子协程启动
        parentJob.cancel()  // 取消父协程，所有子协程也会被取消
    }
}
```

---

## 让协程响应取消

在 Kotlin 中，协程取消是**协作的**。这意味着协程只有在协作时才会响应取消——通过[挂起](#挂起点与取消)或[显式检查取消](#显式检查取消)。

### 挂起点与取消

当协程被取消时，它会继续运行直到到达代码中可能挂起的点，也称为**挂起点**。如果协程在那里挂起，挂起函数会检查它是否已被取消。如果是，协程停止并抛出 `CancellationException`。

常见挂起函数：

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val jobs = listOf(
            launch { awaitCancellation() },      // 挂起直到取消
            launch { delay(Long.MAX_VALUE) },     // 挂起直到取消
            launch { 
                val channel = Channel<Int>()
                channel.receive()  // 挂起等待永远不会发送的值
            },
            launch {
                val deferred = CompletableDeferred<Int>()
                deferred.await()  // 挂起等待永远不会完成的值
            }
        )
        
        delay(100)  // 给子协程时间启动和挂起
        jobs.forEach { it.cancel() }  // 取消所有子协程
    }
    println("所有子协程已完成！")
}
```

> `kotlinx.coroutines` 库中的所有挂起函数都与取消协作，因为它们内部使用 `suspendCancellableCoroutine()`，在协程挂起时检查取消。

### 显式检查取消

如果协程长时间不挂起，除非显式检查取消，否则被取消时不会停止。

#### isActive

在长时间运行的计算中使用 [`isActive`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/is-active.html) 属性定期检查取消：

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val job = launch {
            var i = 0
            while (isActive) {  // 检查是否活跃
                // 执行长时间计算
                ++i
            }
            println("停止计算，已迭代 $i 次")
        }
        
        delay(100)
        job.cancel()
    }
}
```

#### ensureActive()

使用 [`ensureActive()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-active.html) 函数检查取消，如果协程被取消则立即抛出 `CancellationException`：

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val job = launch {
            repeat(1000) {
                ensureActive()  // 检查取消
                // 执行计算
            }
        }
        
        delay(100)
        job.cancel()
    }
}
```

#### yield()

[`yield()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html) 函数挂起协程，在恢复前检查取消。它还让其他协程有机会在同一线程上运行：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    repeat(5) { index ->
        launch {
            repeat(5) { iteration ->
                yield()  // 让其他协程运行
                println("协程 ${index + 1} - 迭代 ${iteration + 1}")
            }
        }
    }
}
```

---

## 中断阻塞代码

在 JVM 上，`Thread.sleep()` 或 `BlockingQueue.take()` 等函数可以阻塞当前线程。这些阻塞函数可以被中断。但是，从协程调用时，取消不会中断线程。

要在取消协程时中断线程，使用 [`runInterruptible()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-interruptible.html)：

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val job = launch {
            try {
                runInterruptible {
                    try {
                        Thread.sleep(Long.MAX_VALUE)  // 阻塞线程
                    } catch (e: InterruptedException) {
                        println("线程被中断: $e")
                        throw e
                    }
                }
            } catch (e: CancellationException) {
                println("协程被取消: $e")
                throw e
            }
        }
        
        delay(100)
        job.cancel()  // 取消协程并中断线程
    }
}
```

---

## 安全处理取消时的值

当被挂起的协程被取消时，它会以 `CancellationException` 恢复而不是返回任何值。这防止代码在已取消的协程作用域中继续执行。

### 使用 finally 块释放资源

```kotlin
import kotlinx.coroutines.*
import java.io.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val job = launch {
            var reader: BufferedReader? = null
            try {
                withContext(Dispatchers.IO) {
                    reader = File("data.txt").bufferedReader()
                }
                // 使用 reader
            } finally {
                reader?.close()  // 确保资源被关闭
            }
        }
        
        delay(100)
        job.cancel()
    }
}
```

### 运行不可取消的代码块

使用 [`NonCancellable`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-non-cancellable/) 防止取消影响某些操作：

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        val job = launch {
            try {
                awaitCancellation()
            } finally {
                withContext(NonCancellable) {
                    println("执行清理...")
                    delay(100)  // 即使协程被取消也会完成
                    println("清理完成")
                }
            }
        }
        
        delay(100)
        job.cancel()
    }
}
```

> 避免在 `.launch()` 或 `.async()` 中使用 `NonCancellable`，这会破坏结构化并发。

---

## 超时

超时允许你在指定时间后自动取消协程。这对于停止耗时过长的操作很有用。

### withTimeoutOrNull

```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration.Companion.milliseconds

suspend fun main() {
    withContext(Dispatchers.Default) {
        val result1 = withTimeoutOrNull(100.milliseconds) {
            delay(300)  // 模拟慢操作
            "完成"
        }
        println("慢操作结果: $result1")  // null（超时）
        
        val result2 = withTimeoutOrNull(100.milliseconds) {
            delay(15)  // 模拟快操作
            "完成"
        }
        println("快操作结果: $result2")  // "完成"
    }
}
```

### withTimeout

`withTimeout` 在超时时抛出 `TimeoutCancellationException`：

```kotlin
try {
    withTimeout(100.milliseconds) {
        delay(300)
        "完成"
    }
} catch (e: TimeoutCancellationException) {
    println("操作超时")
}
```

---

## 最佳实践

1. **始终重新抛出 CancellationException** - 它是协程取消的机制
2. **使用 finally 块释放资源** - 确保资源被正确关闭
3. **在长时间计算中检查取消** - 使用 `isActive` 或 `ensureActive()`
4. **使用 withTimeoutOrNull 而非 withTimeout** - 更安全，返回 null 而非抛出异常
5. **避免在 finally 块中使用挂起函数** - 除非使用 `NonCancellable`

---

## 下一步

- [组合挂起函数](03-composing-suspending-functions.md) - async、顺序执行、并发
- [异常处理](07-exception-handling.md) - 更详细的异常处理机制
