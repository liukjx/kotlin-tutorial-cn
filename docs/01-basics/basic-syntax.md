# 基本语法

> 本节介绍 Kotlin 的核心语法特性，适合有其他编程语言基础的开发者快速上手。

## Hello World

```kotlin
fun main() {
    println("Hello, World!")
}
```

- `fun` 关键字声明函数
- `main` 是程序入口
- `println` 输出并换行（类似 Java 的 `System.out.println`）
- 语句末尾**不需要分号**

---

## 变量声明

### val - 只读变量（推荐）

```kotlin
val name = "Kotlin"       // 类型推断为 String
val year: Int = 2024      // 显式指定类型

// name = "Java"          // ❌ 编译错误，val 不可重新赋值
```

### var - 可变变量

```kotlin
var count = 0
count = 1                  // ✅ 允许修改

var message: String        // 声明但不初始化（需要指定类型）
message = "Hello"
```

### 优先使用 val

Kotlin 风格指南建议：**默认使用 val**，只有确实需要修改时才用 var。这能让代码更安全、更易推理。

---

## 类型推断

Kotlin 编译器能根据初始值推断类型：

```kotlin
val text = "Hello"         // String
val number = 42            // Int
val pi = 3.14              // Double
val flag = true            // Boolean
```

---

## 函数定义

### 基本形式

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

### 表达式体（单行函数）

```kotlin
fun sum(a: Int, b: Int) = a + b

fun greet(name: String) = "Hello, $name!"
```

- 返回类型可省略（编译器自动推断）
- 等号 `=` 替代花括号和 `return`

### 无返回值

```kotlin
fun printSum(a: Int, b: Int): Unit {
    println("Sum: ${a + b}")
}

// Unit 可省略
fun printSum(a: Int, b: Int) {
    println("Sum: ${a + b}")
}
```

### 默认参数

```kotlin
fun greet(name: String, greeting: String = "Hello") {
    println("$greeting, $name!")
}

greet("Kotlin")                    // Hello, Kotlin!
greet("Kotlin", "Hi")              // Hi, Kotlin!
```

### 命名参数

```kotlin
fun createUser(name: String, age: Int, city: String) { ... }

createUser(
    name = "张三",
    city = "北京",
    age = 25           // 顺序可以打乱
)
```

---

## 字符串模板

### 变量插值

```kotlin
val name = "Kotlin"
println("Hello, $name!")          // Hello, Kotlin!
```

### 表达式插值

```kotlin
val a = 10
val b = 20
println("Sum: ${a + b}")          // Sum: 30
println("Length: ${name.length}") // Length: 6
```

### 原始字符串（多行）

```kotlin
val text = """
    |这是
    |多行文本
""".trimMargin()

println(text)
// 输出：
// 这是
// 多行文本
```

---

## 条件表达式

### if 表达式

Kotlin 的 `if` 是表达式，有返回值：

```kotlin
val max = if (a > b) a else b

// 等价于
val max = if (a > b) {
    println("a is bigger")
    a
} else {
    println("b is bigger")
    b
}
```

### when 表达式

替代 Java 的 `switch`，更强大：

```kotlin
val result = when (x) {
    1 -> "One"
    2, 3 -> "Two or Three"
    in 4..10 -> "Between 4 and 10"
    is Int -> "Integer"           // 类型检查
    else -> "Unknown"
}
```

---

## 空安全

Kotlin 区分可空和不可空类型：

```kotlin
var name: String = "Kotlin"
// name = null                  // ❌ 编译错误

var nickname: String? = "Kot"
nickname = null                  // ✅ 允许
```

### 安全调用 ?.

```kotlin
val length = nickname?.length    // 如果 nickname 为 null，返回 null
```

### Elvis 运算符 ?:

```kotlin
val len = nickname?.length ?: 0  // 如果为 null，返回默认值 0
```

### 非空断言 !!

```kotlin
val len = nickname!!.length      // 如果为 null，抛出 NullPointerException
// 慎用！只在你确定不可能为 null 时使用
```

---

## 循环

### for 循环

```kotlin
for (i in 1..5) {                // 1 到 5（包含）
    println(i)
}

for (i in 1 until 5) {           // 1 到 4（不包含 5）
    println(i)
}

for (i in 10 downTo 1 step 2) {  // 10, 8, 6, 4, 2
    println(i)
}

val items = listOf("apple", "banana", "orange")
for (item in items) {
    println(item)
}

for ((index, value) in items.withIndex()) {
    println("$index: $value")
}
```

### while 循环

```kotlin
var i = 0
while (i < 5) {
    println(i)
    i++
}
```

---

## 区间（Range）

```kotlin
val range = 1..10                // 1 到 10
val chars = 'a'..'z'             // a 到 z

if (5 in range) { ... }
if ('e' in chars) { ... }
```

---

## 集合基础

### List

```kotlin
val list = listOf("a", "b", "c")           // 不可变
val mutableList = mutableListOf("a", "b")  // 可变

list[0]                                     // 访问
list.size                                   // 长度
```

### Map

```kotlin
val map = mapOf("a" to 1, "b" to 2)
map["a"]                                    // 返回 1
map["c"]                                    // 返回 null
```

### Set

```kotlin
val set = setOf(1, 2, 3)
1 in set                                    // true
```

---

## 练习

1. 写一个函数 `maxOf(a: Int, b: Int)`，返回较大的数
2. 用 `when` 实现一个简易计算器（加减乘除）
3. 遍历 1-100，输出所有能被 3 整除的数

---

## 下一步

- [基本类型](basic-types.md) - 了解 Kotlin 的类型系统
- [控制流](control-flow.md) - 深入学习条件与循环
- [函数](functions.md) - 更多函数特性
