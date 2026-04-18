# 数据类

> `data class` 自动生成 `equals`、`hashCode`、`toString`、`copy` 等方法，专为数据存储设计。

## 定义数据类

```kotlin
data class User(val name: String, val age: Int)

val u1 = User("Kotlin", 10)
val u2 = User("Kotlin", 10)

println(u1)           // User(name=Kotlin, age=10)
u1 == u2              // true（内容相等）
u1.hashCode() == u2.hashCode()  // true
```

---

## 自动生成的方法

### toString()

```kotlin
data class User(val name: String, val age: Int)

val user = User("Kotlin", 10)
println(user)  // User(name=Kotlin, age=10)
```

### equals() / hashCode()

```kotlin
val u1 = User("Kotlin", 10)
val u2 = User("Kotlin", 10)
val u3 = User("Java", 10)

u1 == u2  // true
u1 == u3  // false

// 基于内容生成的 hashCode
u1.hashCode() == u2.hashCode()  // true
```

### copy()

复制对象并修改部分属性：

```kotlin
val u1 = User("Kotlin", 10)
val u2 = u1.copy(age = 11)
val u3 = u1.copy(name = "Java")

println(u2)  // User(name=Kotlin, age=11)
println(u3)  // User(name=Java, age=10)
```

### componentN()

支持解构声明：

```kotlin
val user = User("Kotlin", 10)

val (name, age) = user
println(name)  // Kotlin
println(age)   // 10

// 等价于
val name = user.component1()
val age = user.component2()
```

---

## 限制

### 必须满足的条件

1. **主构造函数至少有一个参数**
2. **主构造函数的参数必须标记为 `val` 或 `var`**
3. **不能是 abstract、open、sealed、inner**

```kotlin
// ✅ 正确
data class User(val name: String, val age: Int)

// ❌ 错误：无主构造函数参数
data class Empty

// ❌ 错误：参数未标记为属性
data class Invalid(name: String, age: Int)

// ❌ 错误：不能是 open
open data class OpenUser(val name: String)
```

---

## 内容相等 vs 引用相等

```kotlin
data class User(val name: String)

val u1 = User("Kotlin")
val u2 = User("Kotlin")
val u3 = u1

u1 == u2   // true（内容相等）
u1 === u2  // false（引用不相等）
u1 === u3  // true（同一对象）
```

---

## 在集合中使用

```kotlin
data class User(val name: String, val age: Int)

val users = listOf(
    User("Alice", 25),
    User("Bob", 30),
    User("Alice", 25)
)

// 去重（基于 equals）
users.toSet()  // [User(Alice, 25), User(Bob, 30)]

// 作为 Map 的 key
val map = mapOf(
    User("Alice", 25) to "Admin",
    User("Bob", 30) to "User"
)
```

---

## 与普通类对比

### 普通类

```kotlin
class User(val name: String, val age: Int)

val u1 = User("Kotlin", 10)
val u2 = User("Kotlin", 10)

println(u1)  // User@hashcode（默认 toString）
u1 == u2     // false（引用比较）
```

### 数据类

```kotlin
data class User(val name: String, val age: Int)

val u1 = User("Kotlin", 10)
val u2 = User("Kotlin", 10)

println(u1)  // User(name=Kotlin, age=10)
u1 == u2     // true（内容比较）
```

---

## 解构声明

### 基本用法

```kotlin
data class User(val name: String, val age: Int)

val (name, age) = User("Kotlin", 10)

// 在 for 循环中
val users = listOf(User("A", 1), User("B", 2))
for ((name, age) in users) {
    println("$name is $age years old")
}
```

### Map 的解构

```kotlin
val map = mapOf("a" to 1, "b" to 2)

for ((key, value) in map) {
    println("$key = $value")
}
```

### 忽略某些值

```kotlin
val user = User("Kotlin", 10)
val (name, _) = user  // 忽略 age
```

---

## 自定义方法

数据类可以定义额外的方法和属性：

```kotlin
data class User(val name: String, val age: Int) {
    val isAdult: Boolean
        get() = age >= 18
    
    fun greet() = "Hello, I'm $name"
}

val user = User("Kotlin", 20)
user.isAdult  // true
user.greet()  // Hello, I'm Kotlin
```

---

---

## 实战案例：PICO Spatial SDK

### 组件说明数据类

在 PICO 组件展示项目中，`ComponentInfo` 数据类用于存储 UI 组件的说明信息：

```kotlin
// 文件：component-playground-0.10.7/app/src/main/java/.../ComponentInfo.kt

/**
 * 单个组件的讲解信息
 *
 * 这是一个 Kotlin `data class`：
 * - 适合表示“纯数据”
 * - 自动生成 copy、toString、equals 等方法
 */
data class ComponentInfo(
    val name: String,              // 组件名称
    val description: String,       // 它是什么
    val usage: String,             // 拿它干什么
    val codeExample: String? = null,  // 代码示例（可选）
    val tips: String? = null          // 使用建议（可选）
)
```

**为什么用 data class？**
- 纯数据存储，不需要复杂逻辑
- 自动生成 `toString()` 方便调试
- `copy()` 方便创建变体
- `equals()` 基于内容比较

### 使用示例

```kotlin
// 创建组件信息
val buttonInfo = ComponentInfo(
    name = "Button",
    description = "响应用户点击行为的基础按钮",
    usage = "用于触发操作，如提交表单、确认操作等",
    codeExample = """
        Button(onClick = { /* 处理点击 */ }) {
            Text("确定")
        }
    """.trimIndent(),
    tips = "配合 OutlinedButton 和 TextButton 实现不同视觉层级。"
)

// copy() 创建变体
val outlinedButton = buttonInfo.copy(
    name = "OutlinedButton",
    tips = "边框按钮，用于次要操作。"
)

// 解构获取字段
val (name, desc, usage) = buttonInfo
println("组件: $name, 描述: $desc")

// 在列表中使用
val components = listOf(
    buttonInfo,
    ComponentInfo("Switch", "开关控件", "用于设置开关"),
    ComponentInfo("Slider", "滑动条", "用于连续数值选择")
)

// 基于 name 去重
components.distinctBy { it.name }
```

### 带默认参数的数据类

```kotlin
data class ComponentInfo(
    val name: String,
    val description: String,
    val usage: String,
    val codeExample: String? = null,  // 默认 null
    val tips: String? = null          // 默认 null
)

// 创建时可选字段
val simple = ComponentInfo(
    name = "Button",
    description = "按钮",
    usage = "点击触发"
    // codeExample 和 tips 使用默认值 null
)
```

### 在 UI 中展示

```kotlin
// 文件：PlaygroundScreen.kt

@Composable
fun ComponentInfoCard(info: ComponentInfo) {
    Column {
        Text(text = info.name, style = MaterialTheme.typography.h6)
        Text(text = info.description)
        Text(text = info.usage)
        
        info.codeExample?.let { code ->
            CodeBlock(code)
        }
        
        info.tips?.let { tip ->
            TipCard(tip)
        }
    }
}
```

**设计优势**：
- `data class` 让数据结构清晰
- 默认参数让创建更灵活
- 可空属性让可选信息不强制
- 解构让使用更简洁

---

## 练习

### 1. 学生类

创建 `Student` 数据类，包含 id、name、score，实现复制和比较。

### 2. 解构练习

创建 `Point(x, y)` 数据类，解构并计算距离。

### 3. 集合操作

用数据类表示商品，去重并按价格排序。

---

## 下一步

- [密封类](sealed-classes.md) - 限制继承
- [泛型](generics.md) - 类型参数
