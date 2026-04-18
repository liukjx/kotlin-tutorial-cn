# 协程基础

> 本节介绍协程的核心概念：挂起函数、协程作用域、协程构建器、调度器等。

要创建同时执行多个任务的应用程序（称为并发），Kotlin 使用**协程**。协程是一种可挂起的计算，让你能够以清晰的顺序风格编写并发代码。协程可以与其他协程并发运行，甚至可能并行运行。

在 JVM 和 Kotlin/Native 上，所有并发代码（如协程）都在操作系统管理的**线程**上运行。协程可以挂起其执行而不是阻塞线程。这允许一个协程在等待数据到达时挂起，而另一个协程在同一线程上运行，确保有效的资源利用。

![并行与并发线程对比](parallelism-and-concurrency.svg){width="700"}

---

## 挂起函数

协程最基本的构建块是**挂起函数**。它允许正在运行的操作暂停并稍后恢复，而不影响代码结构。

要声明挂起函数，使用 `suspend` 关键字：

```kotlin
suspend fun greet() {
    println("Hello world from a suspending function")
}
```

你只能从另一个挂起函数调用挂起函数。要在 Kotlin 应用程序的入口点调用挂起函数，用 `suspend` 关键字标记 `main()` 函数：

```kotlin
suspend fun main() {
    showUserInfo()
}

suspend fun showUserInfo() {
    println("Loading user...")
    greet()
    println("User: John Smith")
}

suspend fun greet() {
    println("Hello world from a suspending function")
}
```

虽然这个示例还没有使用并发，但通过用 `suspend` 关键字标记函数，你允许它们调用其他挂起函数并在其中运行并发代码。

---

## 添加 kotlinx.coroutines 库

要在项目中包含 `kotlinx.coroutines` 库，根据你的构建工具添加相应的依赖配置：

**Gradle (Kotlin DSL)**

```kotlin
// build.gradle.kts
repositories {
    mavenCentral()
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
}
```

**Gradle (Groovy)**

```groovy
// build.gradle
repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3'
}
```

**Maven**

```xml
<!-- pom.xml -->
<project>
    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlinx</groupId>
            <artifactId>kotlinx-coroutines-core</artifactId>
            <version>1.7.3</version>
        </dependency>
    </dependencies>
    ...
</project>
```

---

## 创建你的第一个协程

要在 Kotlin 中创建协程，你需要：

- 一个[挂起函数](#挂起函数)
- 一个可以运行它的[协程作用域](#协程作用域与结构化并发)
- 一个[协程构建器](#协程构建器函数)如 `CoroutineScope.launch()` 来启动它
- 一个[调度器](#协程调度器)来控制它使用哪些线程

让我们看一个在多线程环境中使用多个协程的示例：

```kotlin
// 导入协程库
import kotlinx.coroutines.*

// 导入 kotlin.time.Duration 以秒为单位表达时长
import kotlin.time.Duration.Companion.seconds

// 定义挂起函数
suspend fun greet() {
    println("greet() 运行在线程: ${Thread.currentThread().name}")
    // 挂起 1 秒并释放线程
    delay(1.seconds)
    // delay() 函数在这里模拟挂起的 API 调用
    // 你可以在这里添加挂起的 API 调用，如网络请求
}

suspend fun main() {
    // 在共享线程池上运行此块中的代码
    withContext(Dispatchers.Default) { // this: CoroutineScope
        this.launch {
            greet()
        }

        // 启动另一个协程
        this.launch {
            println("CoroutineScope.launch() 运行在线程: ${Thread.currentThread().name}")
            delay(1.seconds)
        }

        println("withContext() 运行在线程: ${Thread.currentThread().name}")
    }
}
```

尝试多次运行此示例。你可能会注意到每次运行程序时输出顺序和线程名可能会改变，因为操作系统决定线程何时运行。

---

## 协程作用域与结构化并发

当你在应用程序中运行许多协程时，你需要一种方法将它们作为组来管理。Kotlin 协程依赖于**结构化并发**原则来提供这种结构。

根据这一原则，协程形成父子任务的树形层次结构，具有链接的生命周期。协程的生命周期是从创建到完成、失败或取消的状态序列。

父协程在完成之前等待其子协程完成。如果父协程失败或被取消，其所有子协程也会递归取消。以这种方式保持协程连接使取消和错误处理变得可预测和安全。

要维护结构化并发，新协程只能在 [`CoroutineScope`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/) 中启动，该作用域定义并管理它们的生命周期。`CoroutineScope` 包含**协程上下文**，它定义调度器和其他执行属性。当你在另一个协程内启动协程时，它会自动成为其父作用域的子级。

### 使用 coroutineScope() 创建协程作用域

要使用当前协程上下文创建新的协程作用域，使用 [`coroutineScope()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) 函数。此函数创建协程子树的根协程。它是在块中启动的协程的直接父级，以及它们启动的任何协程的间接父级。`coroutineScope()` 执行挂起块并等待该块及其启动的任何协程完成。

示例：

```kotlin
import kotlin.time.Duration.Companion.seconds
import kotlinx.coroutines.*

suspend fun main() {
    // 协程子树的根
    coroutineScope { // this: CoroutineScope
        this.launch {
            this.launch {
                delay(2.seconds)
                println("子协程完成")
            }
            println("子协程 1 完成")
        }
        this.launch {
            delay(1.seconds)
            println("子协程 2 完成")
        }
    }
    // 只有在 coroutineScope 中的所有子协程完成后才运行
    println("协程作用域完成")
}
```

由于此示例中没有指定[调度器](#协程调度器)，`coroutineScope()` 块中的 `CoroutineScope.launch()` 构建器函数继承当前上下文。如果该上下文没有指定的调度器，`CoroutineScope.launch()` 使用 `Dispatchers.Default`，它在共享线程池上运行。

---

## 协程构建器函数

协程构建器函数是接受 `suspend` [lambda](../01-basics/lambdas.md) 的函数，该 lambda 定义要运行的协程。以下是一些示例：

- [`CoroutineScope.launch()`](#coroutinescope-launch)
- [`CoroutineScope.async()`](#coroutinescope-async)
- [`runBlocking()`](#runblocking)
- [`withContext()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)
- [`coroutineScope()`](#使用-coroutinescope-创建协程作用域)

协程构建器函数需要在 `CoroutineScope` 中运行。这可以是现有作用域或你使用辅助函数如 `coroutineScope()`、[`runBlocking()`](#runblocking) 或 [`withContext()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html#) 创建的作用域。每个构建器定义协程如何启动以及你如何与其结果交互。

### CoroutineScope.launch()

[`CoroutineScope.launch()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html#) 协程构建器函数是 `CoroutineScope` 上的扩展函数。它在现有[协程作用域](#协程作用域与结构化并发)内启动新协程而不阻塞作用域的其余部分。

使用 `CoroutineScope.launch()` 在不需要结果或不想等待结果时与其他工作并行运行任务：

```kotlin
import kotlin.time.Duration.Companion.milliseconds
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        performBackgroundWork()
    }
}

suspend fun performBackgroundWork() = coroutineScope { // this: CoroutineScope
    // 启动一个在不阻塞作用域的情况下运行的协程
    this.launch {
        // 挂起以模拟后台工作
        delay(100.milliseconds)
        println("在后台发送通知")
    }

    // 主协程继续，而前一个协程在后台工作
    println("作用域继续")
}
```

运行此示例后，你可以看到 `main()` 函数没有被 `CoroutineScope.launch()` 阻塞，并在协程在后台工作时继续运行其他代码。

> `CoroutineScope.launch()` 函数返回 [`Job`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/) 句柄。使用此句柄等待启动的协程完成。有关更多信息，请参阅[取消与超时](02-cancellation-and-timeouts.md#取消协程)。

### CoroutineScope.async()

[`CoroutineScope.async()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 协程构建器函数是 `CoroutineScope` 上的扩展函数。它在现有[协程作用域](#协程作用域与结构化并发)内启动并发计算并返回表示最终结果的 [`Deferred`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/) 句柄。使用 `.await()` 函数挂起代码直到结果准备好：

```kotlin
import kotlin.time.Duration.Companion.milliseconds
import kotlinx.coroutines.*

suspend fun main() = withContext(Dispatchers.Default) { // this: CoroutineScope
    // 开始下载第一页
    val firstPage = this.async {
        delay(50.milliseconds)
        "第一页"
    }

    // 并行开始下载第二页
    val secondPage = this.async {
        delay(100.milliseconds)
        "第二页"
    }

    // 等待两个结果并比较它们
    val pagesAreEqual = firstPage.await() == secondPage.await()
    println("页面是否相等: $pagesAreEqual")
}
```

### runBlocking()

[`runBlocking()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 协程构建器函数创建协程作用域并阻塞当前[线程](#协程与-jvm-线程对比)直到该作用域中启动的协程完成。

只在无法从非挂起代码调用挂起代码时才使用 `runBlocking()`：

```kotlin
import kotlin.time.Duration.Companion.milliseconds
import kotlinx.coroutines.*

// 你无法更改的第三方接口
interface Repository {
    fun readItem(): Int
}

object MyRepository : Repository {
    override fun readItem(): Int {
        // 桥接到挂起函数
        return runBlocking {
            myReadItem()
        }
    }
}

suspend fun myReadItem(): Int {
    delay(100.milliseconds)
    return 4
}
```

---

## 协程调度器

**协程调度器**控制协程使用哪个线程或线程池进行执行。协程并不总是绑定到单个线程。它们可以在一个线程上挂起并在另一个线程上恢复，具体取决于调度器。这让你可以同时运行许多协程而不为每个协程分配单独的线程。

> 即使协程可以在不同线程上挂起和恢复，在协程挂起之前写入的值仍然保证在同一协程恢复时可用。

调度器与[协程作用域](#协程作用域与结构化并发)一起工作，定义协程何时运行以及在哪里运行。协程作用域控制协程的生命周期，调度器控制用于执行的线程。

> 你不需要为每个协程指定调度器。默认情况下，协程从其父作用域继承调度器。你可以指定调度器以在不同的上下文中运行协程。
>
> 如果协程上下文不包含调度器，协程构建器使用 `Dispatchers.Default`。

`kotlinx.coroutines` 库包含用于不同用例的不同调度器。例如，[`Dispatchers.Default`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html) 在共享线程池上运行协程，在与主线程分离的后台执行工作。这使其成为数据处理等 CPU 密集型操作的理想选择。

要为 `CoroutineScope.launch()` 等协程构建器指定调度器，将其作为参数传递：

```kotlin
suspend fun runWithDispatcher() = coroutineScope { // this: CoroutineScope
    this.launch(Dispatchers.Default) {
        println("运行在 ${Thread.currentThread().name}")
    }
}
```

或者，你可以使用 `withContext()` 块在其中所有代码在指定调度器上运行：

```kotlin
import kotlin.time.Duration.Companion.milliseconds
import kotlinx.coroutines.*

suspend fun main() = withContext(Dispatchers.Default) { // this: CoroutineScope
    println("withContext 块运行在 ${Thread.currentThread().name}")

    val one = this.async {
        println("第一次计算开始于 ${Thread.currentThread().name}")
        val sum = (1L..500_000L).sum()
        delay(200L)
        println("第一次计算完成于 ${Thread.currentThread().name}")
        sum
    }

    val two = this.async {
        println("第二次计算开始于 ${Thread.currentThread().name}")
        val sum = (500_001L..1_000_000L).sum()
        println("第二次计算完成于 ${Thread.currentThread().name}")
        sum
    }

    // 等待两个计算并打印结果
    println("总和: ${one.await() + two.await()}")
}
```

要了解更多关于协程调度器及其用法，包括 [`Dispatchers.IO`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html) 和 [`Dispatchers.Main`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html) 等其他调度器，请参阅[协程上下文与调度器](04-coroutine-context-and-dispatchers.md)。

---

## 协程与 JVM 线程对比

虽然协程是在 JVM 上并发运行代码的可挂起计算，类似于线程，但它们在底层工作方式不同。

**线程**由操作系统管理。线程可以在多个 CPU 核心上并行运行任务，代表 JVM 上标准并发方法。创建线程时，操作系统为其栈分配内存并使用内核在线程之间切换。这使线程功能强大但也资源密集。每个线程通常需要几兆字节的内存，通常 JVM 一次只能处理几千个线程。

另一方面，**协程**不绑定到特定线程。它可以在一个线程上挂起并在另一个线程上恢复，因此许多协程可以共享同一线程池。当协程挂起时，线程不会被阻塞，仍然可以运行其他任务。这使协程比线程轻量得多，允许在一个进程中运行数百万个协程而不耗尽系统资源。

![协程与线程对比](coroutines-and-threads.svg){width="700"}

让我们看一个示例，其中 50,000 个协程各等待 5 秒然后打印一个点（`.`）：

```kotlin
import kotlin.time.Duration.Companion.seconds
import kotlinx.coroutines.*

suspend fun main() {
    withContext(Dispatchers.Default) {
        // 启动 50,000 个协程，各等待 5 秒然后打印一个点
        printPeriods()
    }
}

suspend fun printPeriods() = coroutineScope { // this: CoroutineScope
    // 启动 50,000 个协程，各等待 5 秒然后打印一个点
    repeat(50_000) {
        this.launch {
            delay(5.seconds)
            print(".")
        }
    }
}
```

现在让我们看看使用 JVM 线程的相同示例：

```kotlin
import kotlin.concurrent.thread

fun main() {
    repeat(50_000) {
        thread {
            Thread.sleep(5000L)
            print(".")
        }
    }
}
```

运行此版本使用更多内存，因为每个线程需要自己的内存栈。对于 50,000 个线程，这可能高达 100 GB，而相同数量的协程大约只需 500 MB。

根据你的操作系统、JDK 版本和设置，JVM 线程版本可能会抛出内存不足错误或减慢线程创建以避免一次运行太多线程。

---

## 下一步

- 在[组合挂起函数](03-composing-suspending-functions.md)中发现更多关于组合挂起函数的内容。
- 在[取消与超时](02-cancellation-and-timeouts.md)中学习如何取消协程和处理超时。
- 在[协程上下文与调度器](04-coroutine-context-and-dispatchers.md)中深入了解协程执行和线程管理。
- 在[异步流](05-flow.md)中学习如何返回多个异步计算的值。
