# 共享可变状态与并发

> 协程可以使用多线程调度器（如 Dispatchers.Default）并行执行。这带来了所有常见的并发问题。

## 问题示例

启动 100 个协程，每个执行 1000 次增量操作：

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val time = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("完成 ${n * k} 次操作，耗时 $time ms")
}

var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

**问题**：输出很可能不是 100000，因为 100 个协程并发修改变量没有同步。

---

## Volatile 无效

`@Volatile` 不能解决问题：

```kotlin
@Volatile
var counter = 0
```

volatile 保证读写的原子性，但不保证 `++` 操作的原子性（读取-修改-写入）。

---

## 解决方案

### 1. 线程安全数据结构

使用 `AtomicInteger`：

```kotlin
import java.util.concurrent.atomic.*

val counter = AtomicInteger()

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.incrementAndGet()
        }
    }
    println("Counter = $counter")  // 100000
}
```

**优点**：最快
**缺点**：不适用于复杂状态

---

### 2. 细粒度线程限制

将每次访问限制到单线程：

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            withContext(counterContext) {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
```

**缺点**：慢，因为每次操作都切换上下文

---

### 3. 粗粒度线程限制

整个操作在单线程中执行：

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    withContext(counterContext) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")  // 100000，且更快
}
```

---

### 4. 互斥锁 (Mutex)

使用非阻塞锁：

```kotlin
import kotlinx.coroutines.sync.*

val mutex = Mutex()
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            mutex.withLock {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
```

**优点**：不阻塞线程
**缺点**：细粒度锁定较慢

---

### 5. Actors

使用 actor 封装状态：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

sealed class CounterMsg
object IncCounter : CounterMsg()
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg()

fun CoroutineScope.counterActor() = actor<CounterMsg> {
    var counter = 0
    for (msg in channel) {
        when (msg) {
            is IncCounter -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}

fun main() = runBlocking {
    val counter = counterActor()
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.send(IncCounter)
        }
    }
    val response = CompletableDeferred<Int>()
    counter.send(GetCounter(response))
    println("Counter = ${response.await()}")
    counter.close()
}
```

---

## 方案对比

| 方案 | 性能 | 适用场景 |
|------|------|----------|
| 原子类 | 最快 | 简单计数、单个变量 |
| 粗粒度线程限制 | 快 | 大块状态更新 |
| Mutex | 中等 | 需要灵活锁定的场景 |
| Actor | 中等 | 复杂状态、消息驱动 |
| 细粒度线程限制 | 慢 | 不推荐 |

---

## 最佳实践

### ✅ 推荐

```kotlin
// 简单计数器用原子类
val counter = AtomicInteger()

// 复杂状态用 Mutex 或 Actor
mutex.withLock {
    // 修改共享状态
}

// UI 应用用主线程限制
withContext(Dispatchers.Main) {
    // 更新 UI
}
```

### ❌ 不推荐

```kotlin
// 裸变量多线程修改
var counter = 0  // 线程不安全！

// volatile 误用
@Volatile var counter = 0  // ++ 仍然不安全

// 过度锁定
mutex.withLock {
    mutex.withLock {  // 嵌套锁，死锁风险
    }
}
```

---

## 总结

并发问题的核心：**共享可变状态**

解决思路：
1. **避免共享**：使用局部变量、消息传递
2. **不可变**：使用不可变数据结构
3. **同步访问**：锁、原子操作、线程限制

> 记住：最好的并发是没有并发。优先考虑无状态设计。

---

## 下一步

恭喜！你已完成 Kotlin 协程核心章节的学习。建议：

1. 实践练习：尝试修改示例代码
2. 深入阅读：[Kotlin 官方协程文档](https://kotlinlang.org/docs/coroutines-guide.html)
3. 项目应用：在实际项目中使用协程
