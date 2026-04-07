# Kotlin 入门教程

一套完整的 Kotlin 入门教程，涵盖基础语法、面向对象、集合操作和协程。

## 📚 目录

### 第一部分：基础语法
- [简介与环境搭建](docs/01-basics/README.md)
- 变量与类型
- 函数与表达式
- 控制流

### 第二部分：协程 ✨
> 来自 Kotlin 官方协程文档

| 章节 | 内容 |
|------|------|
| [协程指南](docs/02-coroutines/coroutines-guide.md) | 目录与概述 |
| [协程基础](docs/02-coroutines/coroutines-basics.md) | 第一个协程、挂起函数、作用域 |
| [取消与超时](docs/02-coroutines/cancellation-and-timeouts.md) | 协程取消、超时处理 |
| [组合挂起函数](docs/02-coroutines/composing-suspending-functions.md) | async、顺序执行、并发 |
| [上下文与调度器](docs/02-coroutines/coroutine-context-and-dispatchers.md) | Dispatcher、Job、作用域 |
| [异步流 Flow](docs/02-coroutines/flow.md) | 冷流、操作符、异常处理 |
| [通道 Channel](docs/02-coroutines/channels.md) | 生产者-消费者、管道 |
| [异常处理](docs/02-coroutines/exception-handling.md) | CoroutineExceptionHandler、监督 |
| [共享状态与并发](docs/02-coroutines/shared-mutable-state-and-concurrency.md) | 线程安全、Mutex、原子操作 |

## 🎯 练习题

来自 [Kotlin Koans](https://github.com/Kotlin/kotlin-koans) 官方练习：

| 目录 | 主题 |
|------|------|
| `exercises/i_introduction` | 入门基础 |
| `exercises/ii_collections` | 集合操作 |
| `exercises/iii_conventions` | 约定（运算符重载等） |
| `exercises/iv_properties` | 属性与委托 |
| `exercises/v_builders` | 构建器 DSL |
| `exercises/vi_generics` | 泛型 |

## 🚀 快速开始

```bash
# 克隆仓库
git clone https://github.com/YOUR_USERNAME/kotlin-tutorial-cn.git

# 运行练习题（需要 Gradle）
cd kotlin-tutorial-cn/exercises
./gradlew test
```

## 📖 资料来源

- [Kotlin 官方文档](https://kotlinlang.org/docs/)
- [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) - 官方协程库
- [Kotlin Koans](https://github.com/Kotlin/kotlin-koans) - 官方练习题

## 📄 License

- 练习题代码：Apache 2.0 (来自 Kotlin/Koans)
- 协程文档：Apache 2.0 (来自 Kotlin/kotlinx.coroutines)
