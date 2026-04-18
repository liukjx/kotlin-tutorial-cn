# Kotlin 入门教程

一套完整的 Kotlin 入门教程，涵盖基础语法、面向对象、集合操作和协程。

## 📚 目录

### 第一部分：基础语法 ⏱️ 约 4-6 小时

> 学习 Kotlin 核心语法，适合有其他编程语言基础的开发者

| 章节 | 内容 | 预计时间 |
|------|------|----------|
| [简介与环境搭建](docs/01-basics/README.md) | 目录与学习路径 | 5分钟 |
| [基本语法](docs/01-basics/basic-syntax.md) | 变量、函数、字符串模板、空安全 | 30分钟 |
| [基本类型](docs/01-basics/basic-types.md) | 数值、字符、布尔、数组、字符串 | 20分钟 |
| [包与导入](docs/01-basics/packages.md) | 包声明、导入、可见性修饰符 | 10分钟 |
| [控制流](docs/01-basics/control-flow.md) | if、when、for、while、范围 | 25分钟 |
| [函数](docs/01-basics/functions.md) | 默认参数、命名参数、扩展函数 | 25分钟 |
| [Lambda 表达式](docs/01-basics/lambdas.md) | 高阶函数、集合操作、作用域函数 | 30分钟 |
| [内联函数](docs/01-basics/inline-functions.md) | inline、noinline、reified | 15分钟 |
| [类与继承](docs/01-basics/classes.md) | 构造函数、继承、抽象类、接口 | 30分钟 |
| [属性与字段](docs/01-basics/properties.md) | getter/setter、延迟初始化、委托 | 20分钟 |
| [接口](docs/01-basics/interfaces.md) | 接口定义、多继承、SAM 转换 | 15分钟 |
| [数据类](docs/01-basics/data-classes.md) | data class、解构、copy | 15分钟 |
| [密封类](docs/01-basics/sealed-classes.md) | sealed class、状态管理 | 10分钟 |
| [泛型](docs/01-basics/generics.md) | 泛型类、型变、reified | 20分钟 |
| [集合概述](docs/01-basics/collections-overview.md) | List、Set、Map | 15分钟 |
| [过滤与映射](docs/01-basics/collection-filtering.md) | filter、map、flatMap | 20分钟 |
| [排序与分组](docs/01-basics/collection-grouping.md) | sort、groupBy、聚合 | 15分钟 |

### 第二部分：协程 ✨ ⏱️ 约 3-5 小时

> 异步编程核心，来自 Kotlin 官方协程文档（已翻译）

| 章节 | 内容 | 预计时间 |
|------|------|----------|
| [协程指南](docs/02-coroutines/00-coroutines-guide.md) | 目录与概述 | 5分钟 |
| [协程基础](docs/02-coroutines/01-coroutines-basics.md) | 挂起函数、作用域、构建器、调度器 | 40分钟 |
| [取消与超时](docs/02-coroutines/02-cancellation-and-timeouts.md) | 协程取消、超时处理、资源释放 | 30分钟 |
| [组合挂起函数](docs/02-coroutines/03-composing-suspending-functions.md) | async、顺序执行、并发 | 25分钟 |
| [上下文与调度器](docs/02-coroutines/04-coroutine-context-and-dispatchers.md) | Dispatcher、Job、作用域 | 30分钟 |
| [异步流 Flow](docs/02-coroutines/05-flow.md) | 冷流、操作符、异常处理 | 40分钟 |
| [通道 Channel](docs/02-coroutines/06-channels.md) | 生产者-消费者、管道 | 30分钟 |
| [异常处理](docs/02-coroutines/07-exception-handling.md) | CoroutineExceptionHandler、监督 | 25分钟 |
| [共享状态与并发](docs/02-coroutines/08-shared-mutable-state-and-concurrency.md) | 线程安全、Mutex、原子操作 | 25分钟 |

---

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

---

## 🚀 快速开始

```bash
# 克隆仓库
git clone https://github.com/liukjx/kotlin-tutorial-cn.git

# 运行练习题（需要 Gradle）
cd kotlin-tutorial-cn/exercises
./gradlew test
```

---

## 📖 学习建议

### 初学者路径
1. **第 1-2 天**：基本语法 → 基本类型 → 控制流 → 函数
2. **第 3-4 天**：类与继承 → 属性 → 接口 → 数据类
3. **第 5 天**：集合操作 → 泛型
4. **第 6-7 天**：协程基础 → 取消与超时 → 组合挂起函数

### 有 Java 经验的开发者
- 可跳过：基本语法、基本类型、包与导入
- 重点学习：空安全、扩展函数、协程
- 预计 2-3 天完成

### 有 Kotlin 基础的开发者
- 直接学习：协程部分
- 预计 1-2 天完成

---

## 📖 资料来源

- [Kotlin 官方文档](https://kotlinlang.org/docs/)
- [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) - 官方协程库
- [Kotlin Koans](https://github.com/Kotlin/kotlin-koans) - 官方练习题

---

## 📄 License

- 练习题代码：Apache 2.0 (来自 Kotlin/Koans)
- 协程文档：Apache 2.0 (来自 Kotlin/kotlinx.coroutines)
- 中文翻译：MIT License
