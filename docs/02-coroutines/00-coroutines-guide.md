# 协程指南

> 本指南涵盖 `kotlinx.coroutines` 的核心功能，通过一系列示例介绍协程的主要特性。

Kotlin 标准库只提供 minimal low-level API 来支持协程。与其他具有类似能力的语言不同，`async` 和 `await` 在 Kotlin 中不是关键字，甚至不是标准库的一部分。此外，Kotlin 的**挂起函数**概念为异步操作提供了比 futures 和 promises 更安全、更不易出错的抽象。

`kotlinx.coroutines` 是 JetBrains 开发的丰富协程库。它包含许多高级协程原语，本指南将涵盖 `launch`、`async` 等。

要使用协程并遵循本指南中的示例，你需要添加 `kotlinx-coroutines-core` 模块的依赖，如[项目 README](https://github.com/Kotlin/kotlinx.coroutines/blob/master/README.md#using-in-your-projects) 中所述。

---

## 目录

- [协程基础](01-coroutines-basics.md) - 第一个协程、挂起函数、作用域
- [协程与通道教程](coroutines-and-channels.md) - 入门教程
- [取消与超时](02-cancellation-and-timeouts.md) - 协程取消、超时处理
- [组合挂起函数](03-composing-suspending-functions.md) - async、顺序执行、并发
- [上下文与调度器](04-coroutine-context-and-dispatchers.md) - Dispatcher、Job、作用域
- [异步流 Flow](05-flow.md) - 冷流、操作符、异常处理
- [通道 Channel](06-channels.md) - 生产者-消费者、管道
- [异常处理](07-exception-handling.md) - CoroutineExceptionHandler、监督
- [共享状态与并发](08-shared-mutable-state-and-concurrency.md) - 线程安全、Mutex、原子操作
- [Select 表达式（实验性）](select-expression.md)
- [使用 IntelliJ IDEA 调试协程](debug-coroutines-with-idea.md)
- [使用 IntelliJ IDEA 调试 Flow](debug-flow-with-idea.md)

---

## 额外资源

- [UI 编程协程指南](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/coroutines-guide-ui.md)
- [协程设计文档（KEEP）](https://github.com/Kotlin/KEEP/blob/master/proposals/coroutines.md)
- [kotlinx.coroutines API 参考](https://kotlinlang.org/api/kotlinx.coroutines/)
- [Android 协程最佳实践](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
- [Android Kotlin 协程和 Flow 额外资源](https://developer.android.com/kotlin/coroutines/additional-resources)
