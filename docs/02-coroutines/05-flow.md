# 异步流 Flow

> 挂起函数异步返回单个值，但如何返回多个异步计算的值？这就是 Kotlin Flow 的用武之地。

## 表示多个值

多个值可以用集合表示：

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)

fun main() {
    simple().forEach { value -> println(value) }
}
// 输出：1 2 3
```

### 序列

如果计算是 CPU 密集型的：

```kotlin
fun simple(): Sequence<Int> = sequence {
    for (i in 1..3) {
        Thread.sleep(100)  // 阻塞
        yield(i)
    }
}
```

### 挂起函数

```kotlin
suspend fun simple(): List<Int> {
    delay(1000)  // 非阻塞
    return listOf(1, 2, 3)
}

fun main() = runBlocking {
    simple().forEach { println(it) }
}
```

### Flow

使用 `Flow<Int>` 表示异步计算的值流：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)  // 非阻塞
        emit(i)     // 发射值
    }
}

fun main() = runBlocking {
    launch {
        for (k in 1..3) {
            println("我没被阻塞 $k")
            delay(100)
        }
    }
    simple().collect { value -> println(value) }
}
```

---

## Flow 是冷的

Flow 是**冷流**——`flow { ... }` 构建器中的代码在流被收集之前不会运行：

```kotlin
fun simple(): Flow<Int> = flow {
    println("Flow 启动")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking {
    println("调用 simple 函数...")
    val flow = simple()
    println("调用 collect...")
    flow.collect { println(it) }
    println("再次调用 collect...")
    flow.collect { println(it) }
}
```

**输出**：

```text
调用 simple 函数...
调用 collect...
Flow 启动
1
2
3
再次调用 collect...
Flow 启动
1
2
3
```

---

## Flow 取消

Flow 遵循协程的协作取消：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("发射 $i")
        emit(i)
    }
}

fun main() = runBlocking {
    withTimeoutOrNull(250) {
        simple().collect { println(it) }
    }
    println("完成")
}
```

**输出**（只发射了两个数字）：

```text
发射 1
1
发射 2
2
完成
```

---

## Flow 构建器

```kotlin
// flow { ... } 构建器
fun flowBuilder(): Flow<Int> = flow {
    emit(1)
    emit(2)
    emit(3)
}

// flowOf 构建器
val flow = flowOf(1, 2, 3)

// 集合转换为 Flow
val flow = (1..3).asFlow()
```

---

## 中间操作符

### map 和 filter

```kotlin
suspend fun performRequest(request: Int): String {
    delay(1000)
    return "响应 $request"
}

fun main() = runBlocking {
    (1..3).asFlow()
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
}
// 每秒输出一个响应
```

### transform

最通用的转换操作符：

```kotlin
(1..3).asFlow()
    .transform { request ->
        emit("发起请求 $request")
        emit(performRequest(request))
    }
    .collect { println(it) }
```

### take

限制大小：

```kotlin
fun numbers(): Flow<Int> = flow {
    try {
        emit(1)
        emit(2)
        println("这行不会执行")
        emit(3)
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking {
    numbers()
        .take(2)
        .collect { println(it) }
}
// 输出：1, 2, Finally in numbers
```

---

## 终端操作符

```kotlin
// collect - 收集所有值
flow.collect { println(it) }

// toList / toSet - 转换为集合
val list = flow.toList()
val set = flow.toSet()

// first / single - 获取特定值
val first = flow.first()
val single = flow.single()

// reduce / fold - 归约
val sum = (1..5).asFlow()
    .map { it * it }
    .reduce { a, b -> a + b }
println(sum)  // 55
```

---

## Flow 上下文

### 上下文保留

Flow 的收集总是在调用协程的上下文中进行：

```kotlin
fun simple(): Flow<Int> = flow {
    log("启动 simple flow")
    for (i in 1..3) {
        emit(i)
    }
}

fun main() = runBlocking {
    simple().collect { value -> log("收集 $value") }
}

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
```

### flowOn 操作符

使用 `flowOn` 改变 Flow 的执行上下文：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100)  // 模拟 CPU 密集型
        log("发射 $i")
        emit(i)
    }
}.flowOn(Dispatchers.Default)  // 正确的方式

fun main() = runBlocking {
    simple().collect { value -> log("收集 $value") }
}
```

**输出**（发射在后台线程，收集在主线程）：

```text
[DefaultDispatcher-worker-1] 发射 1
[main] 收集 1
[DefaultDispatcher-worker-1] 发射 2
[main] 收集 2
[DefaultDispatcher-worker-1] 发射 3
[main] 收集 3
```

> ❌ **错误**：在 `flow { ... }` 中使用 `withContext`
> ✅ **正确**：使用 `flowOn` 操作符

---

## 缓冲

### buffer

当发射和收集都慢时，使用缓冲提高性能：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)  // 发射耗时 100ms
        emit(i)
    }
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        simple()
            .buffer()  // 缓冲发射，不等待
            .collect { value ->
                delay(300)  // 收集耗时 300ms
                println(value)
            }
    }
    println("收集耗时 $time ms")
}
// 无缓冲：~1200ms
// 有缓冲：~1000ms
```

### conflate

合并中间值，只处理最新的：

```kotlin
simple()
    .conflate()
    .collect { value ->
        delay(300)
        println(value)
    }
// 输出：1, 3（跳过了 2）
```

### collectLatest

取消慢收集器，重新开始：

```kotlin
simple()
    .collectLatest { value ->
        println("开始收集 $value")
        delay(300)
        println("完成 $value")
    }
// 输出：开始收集 1, 开始收集 2, 开始收集 3, 完成 3
```

---

## 组合多个 Flow

### zip

组合两个 Flow 的值：

```kotlin
val nums = (1..3).asFlow()
val strs = flowOf("one", "two", "three")

nums.zip(strs) { a, b -> "$a -> $b" }
    .collect { println(it) }
// 输出：1 -> one, 2 -> two, 3 -> three
```

### combine

当任一 Flow 发射时重新计算：

```kotlin
val nums = (1..3).asFlow().onEach { delay(300) }
val strs = flowOf("one", "two", "three").onEach { delay(400) }

nums.combine(strs) { a, b -> "$a -> $b" }
    .collect { println(it) }
// 每当任一 Flow 发射时输出
```

### flattenConcat

扁平化 Flow 的 Flow：

```kotlin
val flowOfFlows = flow {
    emit(flowOf(1, 2))
    emit(flowOf(3, 4))
}

flowOfFlows.flattenConcat()
    .collect { println(it) }
// 输出：1, 2, 3, 4
```

### flattenMerge

并发扁平化：

```kotlin
flowOfFlows.flattenMerge(concurrency = 2)
    .collect { println(it) }
```

---

## Flow 异常处理

### catch 操作符

```kotlin
fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException("错误")
}

fun main() = runBlocking {
    simple()
        .catch { e -> println("捕获异常: $e") }
        .collect { println(it) }
}
// 输出：1, 捕获异常: RuntimeException: 错误
```

### onCompletion

完成时的回调（无论成功还是异常）：

```kotlin
simple()
    .onCompletion { cause ->
        if (cause != null) println("Flow 异常完成: $cause")
        else println("Flow 正常完成")
    }
    .catch { e -> println("捕获: $e") }
    .collect { println(it) }
```

---

## Flow 取消检查

使用 `currentCoroutineContext().isActive` 或 `ensureActive()`：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..5) {
        ensureActive()  // 检查取消
        emit(i)
    }
}
```

---

## SharedFlow 和 StateFlow

### StateFlow

状态持有者，总是有值：

```kotlin
class ViewModel {
    private val _state = MutableStateFlow(0)
    val state: StateFlow<Int> = _state.asStateFlow()
    
    fun increment() {
        _state.value++
    }
}
```

### SharedFlow

事件广播：

```kotlin
class EventBus {
    private val _events = MutableSharedFlow<String>()
    val events: SharedFlow<String> = _events
    
    suspend fun send(event: String) {
        _events.emit(event)
    }
}
```

---

## 实战案例：PICO 视频播放器

### StateFlow 状态管理

在 PICO 视频播放器中，`StateFlow` 用于管理播放状态：

```kotlin
// 文件：spatialvideo-0.10.7/app/src/main/java/.../VideoViewModel.kt

class VideoViewModel : ViewModel() {
    
    // 私有可变状态
    private val _videoState = MutableStateFlow(PlaybackState.READY)
    
    // 公开只读状态
    val videoState: StateFlow<PlaybackState> = _videoState.asStateFlow()
    
    // 播放时间
    private val _playbackTime = MutableStateFlow(0L)
    
    // 是否正在拖拽进度条
    private val _isSeeking = MutableStateFlow(false)
    
    // 拖拽位置
    private val _seekingPosition = MutableStateFlow(0f)
}
```

**为什么用 StateFlow？**
- 总是有值，适合 UI 状态
- 支持多个订阅者
- 线程安全
- 与 Compose/LiveData 集成良好

### combine - 合并多个 Flow

在视频播放器中，需要根据多个状态计算进度显示：

```kotlin
/**
 * 视频进度
 * 
 * 【逻辑】
 * - 拖拽中: 显示拖拽位置
 * - 非拖拽: 显示实际播放时间
 */
val videoProgress: StateFlow<Float> =
    combine(
        _playbackTime,      // 实际播放时间
        _isSeeking,         // 是否正在拖拽
        _seekingPosition    // 拖拽位置
    ) { playbackTime, isSeekingProgress, seekingPosition ->
        // 拖拽时显示拖拽位置，否则显示实际时间
        if (isSeekingProgress) seekingPosition else playbackTime.toFloat()
    }
    .stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),  // 5秒后停止收集
        initialValue = 0f
    )
```

**combine 的优势**：
- 任一 Flow 变化时重新计算
- 自动取消上游 Flow
- 将冷流转为热流（stateIn）

### collectLatest - 收集最新值

在播放状态监听中，使用 `collectLatest` 确保只处理最新状态：

```kotlin
init {
    viewModelScope.launch {
        // collectLatest: 收集最新值，新值到来时取消之前的处理
        videoState.collectLatest { state ->
            if (state == PlaybackState.PLAYING) {
                // 播放中：持续更新进度
                while (isActive) {
                    _playbackTime.value = getCurrentTime()
                    delay(PLAYBACK_UNIT_TIME)  // 每 50ms 更新一次
                    
                    // 检查是否播放完成
                    if (hasCompleted()) {
                        _videoState.value = PlaybackState.READY
                        break
                    }
                }
            }
        }
    }
}
```

**collectLatest vs collect**：
- `collect`：顺序处理所有值
- `collectLatest`：新值到来时取消之前的处理，适合搜索、状态切换等场景

### 播放状态机

完整的播放状态流转：

```kotlin
fun onPlayPauseClicked() {
    when (manager.state) {
        PlaybackState.READY -> {
            manager.play()
            _videoState.value = PlaybackState.PLAYING
        }
        PlaybackState.PLAYING -> {
            manager.pause()
            _videoState.value = PlaybackState.PAUSED
        }
        PlaybackState.PAUSED -> {
            manager.resume()
            _videoState.value = PlaybackState.PLAYING
        }
        else -> {}
    }
}
```

### 状态架构图

```
┌─────────────────────────────────────────────────────────┐
│  VideoViewModel                                          │
├─────────────────────────────────────────────────────────┤
│  _videoState: MutableStateFlow<PlaybackState>           │
│  _playbackTime: MutableStateFlow<Long>                  │
│  _isSeeking: MutableStateFlow<Boolean>                  │
│  _seekingPosition: MutableStateFlow<Float>              │
│                                                          │
│  videoProgress: StateFlow<Float> (combine)              │
│     └─ 合并 3 个状态 → 计算显示进度                      │
├─────────────────────────────────────────────────────────┤
│  UI 层订阅                                               │
│  - PlaybackToolbar: 播放/暂停按钮                        │
│  - ProgressBar: 进度条                                   │
│  - TimeText: 时间显示                                    │
└─────────────────────────────────────────────────────────┘
```

---

## 最佳实践

### ✅ 推荐

```kotlin
// 使用 flowOn 切换上下文
flow { ... }
    .flowOn(Dispatchers.IO)
    .collect { ... }

// 使用 catch 处理异常
flow { ... }
    .catch { e -> handleError(e) }
    .collect { ... }

// 使用合适的终端操作符
val list = flow.toList()
val first = flow.first()
```

### ❌ 不推荐

```kotlin
// 在 flow 构建器中使用 withContext
flow {
    withContext(Dispatchers.IO) {  // 错误！
        emit(value)
    }
}

// 在 collect 中处理业务逻辑（应使用操作符）
flow.collect { value ->
    // 复杂业务逻辑
}
```

---

## 下一步

- [通道 Channel](06-channels.md) - 生产者-消费者模式
- [异常处理](07-exception-handling.md) - 更详细的异常处理
