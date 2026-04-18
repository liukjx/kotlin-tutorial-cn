# 通道 Channel

> Deferred 值提供了一种在协程之间传输单个值的便捷方式。通道提供了一种传输值流的方式。

## Channel 基础

[Channel] 在概念上与 `BlockingQueue` 非常相似。一个关键区别是，它用挂起的 [send][SendChannel.send] 代替阻塞的 `put`，用挂起的 [receive][ReceiveChannel.receive] 代替阻塞的 `take`。

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        for (x in 1..5) channel.send(x * x)
    }
    repeat(5) { println(channel.receive()) }
    println("完成!")
}
```

**输出**：

```text
1
4
9
16
25
完成!
```

---

## 关闭和迭代 Channel

与队列不同，通道可以关闭以表示不再有元素到来。在接收端，可以使用常规 `for` 循环接收元素。

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        for (x in 1..5) channel.send(x * x)
        channel.close()  // 发送完成
    }
    for (y in channel) println(y)
    println("完成!")
}
```

---

## 构建生产者

生产者-消费者模式在并发代码中很常见。使用 [produce] 构建器可以轻松创建生产者：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceSquares(): ReceiveChannel<Int> = produce {
    for (x in 1..5) send(x * x)
}

fun main() = runBlocking {
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("完成!")
}
```

---

## 管道 Pipeline

管道是一种模式，一个协程生产（可能是无限的）值流，另一个协程消费并处理：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++)  // 无限整数流
}

fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}

fun main() = runBlocking {
    val numbers = produceNumbers()
    val squares = square(numbers)
    repeat(5) { println(squares.receive()) }
    println("完成!")
    coroutineContext.cancelChildren()
}
```

### 素数管道示例

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
    var cur = numbersFrom(2)
    repeat(10) {
        val prime = cur.receive()
        println(prime)
        cur = filter(cur, prime)
    }
    coroutineContext.cancelChildren()
}

fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) send(x++)
}

fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int) = produce<Int> {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

**输出**（前 10 个素数）：

```text
2
3
5
7
11
13
17
19
23
29
```

---

## Fan-out（扇出）

多个协程可以从同一个通道接收，在它们之间分配工作：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
        delay(100)
    }
}

fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    for (msg in channel) {
        println("处理器 #$id 收到 $msg")
    }
}

fun main() = runBlocking {
    val producer = produceNumbers()
    repeat(5) { launchProcessor(it, producer) }
    delay(950)
    producer.cancel()
}
```

---

## Fan-in（扇入）

多个协程可以发送到同一个通道：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlin.random.*

fun CoroutineScope.producer(id: Int, channel: SendChannel<Int>) = launch {
    repeat(5) {
        delay(Random.nextLong(100))
        channel.send(id * 10 + it)
    }
}

fun main() = runBlocking {
    val channel = Channel<Int>()
    repeat(3) { producer(it, channel) }
    repeat(15) { println(channel.receive()) }
    println("完成!")
}
```

---

## 缓冲通道

通道可以有缓冲区：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
    val channel = Channel<Int>(4)  // 缓冲区大小为 4
    launch {
        repeat(10) {
            println("发送 $it")
            channel.send(it)
        }
    }
    repeat(10) {
        delay(100)
        println("接收 ${channel.receive()}")
    }
    channel.close()
}
```

---

## Ticker Channel

定时发送值的通道：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
    val tickerChannel = ticker(100)  // 每 100ms 发送一次
    repeat(5) {
        println("收到: ${tickerChannel.receive()}")
    }
    tickerChannel.cancel()
}
```

---

## Select 表达式

`select` 允许等待多个挂起操作：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlinx.coroutines.selects.*

fun main() = runBlocking {
    val channel1 = Channel<String>()
    val channel2 = Channel<String>()
    
    launch {
        delay(100)
        channel1.send("来自 channel1")
    }
    launch {
        delay(50)
        channel2.send("来自 channel2")
    }
    
    repeat(2) {
        select<Unit> {
            channel1.onReceive { value -> println(value) }
            channel2.onReceive { value -> println(value) }
        }
    }
}
```

---

## 最佳实践

### ✅ 推荐

```kotlin
// 使用 produce 创建生产者
fun CoroutineScope.producer(): ReceiveChannel<Int> = produce {
    // 发送值
}

// 使用 consumeEach 消费
channel.consumeEach { value ->
    // 处理值
}

// 及时关闭通道
channel.close()
```

### ❌ 不推荐

```kotlin
// 忘记关闭通道（可能导致泄漏）
launch {
    channel.send(value)
    // 没有 close()
}

// 使用 blocking 操作
channel.send(value)  // 应该在协程中
```

---

## Channel vs Flow

| 特性 | Channel | Flow |
|------|---------|------|
| 热还是冷 | 热（活跃的） | 冷（被动激活） |
| 多消费者 | ✅ 支持 | ❌ 单消费者 |
| 缓冲 | ✅ 可配置 | ✅ buffer() |
| 取消传播 | ✅ 双向 | ✅ 协作取消 |
| 适用场景 | 事件流、消息队列 | 数据流、响应式 |

---

## 下一步

- [异常处理](07-exception-handling.md) - 协程异常处理机制
- [共享状态与并发](08-shared-mutable-state-and-concurrency.md) - 线程安全问题
