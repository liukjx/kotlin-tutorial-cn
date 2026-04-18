# 包与导入

> Kotlin 使用包来组织代码，避免命名冲突。

## 包声明

每个 Kotlin 文件开头可以声明包：

```kotlin
// 文件: com/example/demo/Utils.kt
package com.example.demo

fun hello() = "Hello from demo"
```

如果没有声明包，文件属于默认包（无包名）。

---

## 导入

### 导入单个

```kotlin
import com.example.demo.hello

fun main() {
    hello()
}
```

### 导入整个包

```kotlin
import com.example.demo.*

hello()
```

### 导入类

```kotlin
import com.example.demo.User
import com.example.demo.*  // 导入包下所有类
```

### 导入函数和属性

Kotlin 可以导入顶层函数和属性：

```kotlin
import com.example.utils.formatDate
import com.example.utils.DEFAULT_FORMAT

val result = formatDate(Date())
```

---

## 别名 as

当类名或函数名冲突时，使用别名：

```kotlin
import com.example.demo.User as DemoUser
import com.example.test.User as TestUser

val demoUser = DemoUser()
val testUser = TestUser()
```

---

## 默认导入

以下包会自动导入，无需手动 `import`：

```kotlin
kotlin.*
kotlin.annotation.*
kotlin.collections.*
kotlin.comparisons.*
kotlin.io.*
kotlin.ranges.*
kotlin.sequences.*
kotlin.text.*

// JVM 平台额外导入
java.lang.*
kotlin.jvm.*
```

---

## 可见性修饰符

### 四种修饰符

| 修饰符        | 类成员       | 顶层声明     |
|---------------|--------------|--------------|
| `public`      | 所有可见     | 所有可见     |
| `internal`    | 模块内可见   | 模块内可见   |
| `protected`   | 子类可见     | ❌ 不可用    |
| `private`     | 类内可见     | 文件内可见   |

### 默认可见性

- **默认是 `public`**（不同于 Java 的 package-private）

### 示例

```kotlin
// 文件: Utils.kt

private const val SECRET = "hidden"  // 仅本文件可见

internal fun moduleHelper() {}        // 模块内可见

public fun greet() {}                 // 默认 public，可省略

class User {
    private val id: Int = 0           // 类内可见
    protected val name: String = ""   // 子类可见
    internal val age: Int = 0         // 模块内可见
    public val email: String = ""     // 所有可见
}
```

---

## 模块（Module）

`internal` 修饰符的作用域是模块。

### 什么是模块？

一组一起编译的 Kotlin 文件，例如：

- 一个 IntelliJ IDEA 模块
- 一个 Maven 项目
- 一个 Gradle 源集（source set）
- 一个 `<kotlinc>` Ant 任务编译的一组文件

```kotlin
// 模块 A
internal class InternalHelper  // 只在模块 A 内可见

// 模块 B
// val helper = InternalHelper()  // ❌ 编译错误
```

---

## 目录结构约定

Kotlin **不要求**文件目录与包名一致（不同于 Java），但建议遵循：

```
src/
└── com/
    └── example/
        └── demo/
            ├── Main.kt          # package com.example.demo
            ├── Utils.kt         # package com.example.demo
            └── model/
                └── User.kt      # package com.example.demo.model
```

### 多个类在同一文件

Kotlin 允许一个文件包含多个类：

```kotlin
// Person.kt
package com.example.model

class Person(val name: String)
class Address(val city: String)
```

### 文件名与类名

文件名可以与类名不同（不同于 Java）：

```kotlin
// Utils.kt
package com.example

fun helper1() {}
class Helper {}
```

---

## 顶层声明

Kotlin 允许在文件顶层声明：

```kotlin
// Constants.kt
package com.example

const val APP_NAME = "MyApp"      // 顶层常量
val appVersion = "1.0"            // 顶层属性

fun greet() = "Hello"             // 顶层函数

class User(val name: String)      // 类
```

---

## 练习

1. 创建两个不同包的文件，各定义一个同名函数，使用别名导入并调用
2. 创建一个 `internal` 类，验证它在另一个模块中无法访问
3. 创建一个工具类文件，包含多个顶层函数，观察生成的字节码

---

## 下一步

- [控制流](control-flow.md) - 学习条件与循环
- [函数](functions.md) - 深入函数特性
