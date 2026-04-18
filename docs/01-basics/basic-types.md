# 基本类型

> Kotlin 中一切皆对象，即使是基本类型也有对应的方法和属性。

## 数值类型

| 类型  | 大小    | 范围                          |
|-------|---------|-------------------------------|
| Byte  | 8 位    | -128 ~ 127                    |
| Short | 16 位   | -32768 ~ 32767                |
| Int   | 32 位   | -2³¹ ~ 2³¹-1                  |
| Long  | 64 位   | -2⁶³ ~ 2⁶³-1                  |
| Float | 32 位   | 单精度浮点数                   |
| Double| 64 位   | 双精度浮点数                   |

### 字面量

```kotlin
val decimal = 123
val long = 123L
val hex = 0xFF
val binary = 0b0101
val double = 123.5
val float = 123.5f

// 数字下划线（提高可读性）
val million = 1_000_000
val creditCard = 1234_5678_9012_3456L
```

### 类型转换

Kotlin **不会自动转换**数值类型（不同于 Java）：

```kotlin
val a: Int = 100
// val b: Long = a          // ❌ 编译错误
val b: Long = a.toLong()    // ✅ 显式转换
```

所有数值类型都提供转换方法：
- `toByte()`, `toShort()`, `toInt()`, `toLong()`
- `toFloat()`, `toDouble()`

---

## 字符 Char

```kotlin
val letter: Char = 'A'
val digit: Char = '9'
val escape: Char = '\n'      // 换行符

// Char 不能直接当数字用
// val code: Int = letter    // ❌ 错误
val code: Int = letter.code  // ✅ 获取 Unicode 编码
```

### 转义字符

| 字符   | 含义     |
|--------|----------|
| `\t`   | 制表符   |
| `\n`   | 换行     |
| `\\`   | 反斜杠   |
| `\'`   | 单引号   |
| `\"`   | 双引号   |
| `\$`   | 美元符号 |

---

## 布尔 Boolean

```kotlin
val flag: Boolean = true
val enabled = false

// 逻辑运算
val result = flag && enabled   // 与
val result = flag || enabled   // 或
val result = !flag             // 非
```

---

## 数组 Array

### 创建数组

```kotlin
val numbers = arrayOf(1, 2, 3, 4, 5)
val strings = arrayOf("a", "b", "c")

// 指定类型
val ints = intArrayOf(1, 2, 3)
val longs = longArrayOf(1L, 2L, 3L)

// 创建空数组
val empty = emptyArray<String>()

// 创建指定大小的数组
val squares = Array(5) { i -> i * i }  // [0, 1, 4, 9, 16]
```

### 操作数组

```kotlin
val arr = arrayOf(1, 2, 3)

arr[0]                          // 1（访问）
arr.size                        // 3（长度）
arr[1] = 10                     // 修改（arrayOf 创建的是可变数组）

// 遍历
for (item in arr) { ... }
for (i in arr.indices) { arr[i] }
```

---

## 字符串 String

### 创建字符串

```kotlin
val text = "Hello, Kotlin!"
val empty = String()            // 空字符串
```

### 字符串模板

```kotlin
val name = "Kotlin"
val greeting = "Hello, $name!"
val expression = "Length: ${name.length}"
```

### 多行字符串

```kotlin
val json = """
{
    "name": "Kotlin",
    "version": 2.0
}
""".trimIndent()

// trimIndent() 去除公共缩进
```

### 常用方法

```kotlin
val s = "Kotlin"

s.length                        // 6
s.isEmpty()                     // false
s.isBlank()                     // false（全是空白字符）
s.substring(0, 3)               // "Kot"
s.split(",")                    // 分割
s.replace("K", "J")             // "Jotlin"
s.uppercase()                   // "KOTLIN"
s.lowercase()                   // "kotlin"
s.trim()                        // 去除首尾空白
s.startsWith("Kot")             // true
s.contains("lin")               // true
```

### 字符串比较

```kotlin
val a = "Kotlin"
val b = "Kotlin"

a == b                          // ✅ 内容比较（推荐）
a === b                         // 引用比较

a.compareTo("Java") > 0         // 字典序比较
```

---

## 无符号类型（Unsigned）

Kotlin 1.3+ 支持无符号整数：

```kotlin
val uByte: UByte = 255u
val uShort: UShort = 65535u
val uInt: UInt = 0xFFFFFFFFu
val uLong: ULong = 0xFFFFFFFFFFFFFFFFu

val arr = ubyteArrayOf(1u, 2u, 255u)
```

---

## 类型检查与转换

### is 与 !is

```kotlin
val obj: Any = "Hello"

if (obj is String) {
    println(obj.length)         // 自动智能转换为 String
}

if (obj !is Int) {
    println("Not an Int")
}
```

### 智能转换

```kotlin
fun printLength(obj: Any) {
    if (obj is String) {
        // 此处 obj 自动被识别为 String
        println(obj.length)
    }
}
```

### 显式转换 as

```kotlin
val obj: Any = "Hello"

val s: String = obj as String   // 可能抛出 ClassCastException
val s: String? = obj as? String // 安全转换，失败返回 null
```

---

## 练习

1. 创建一个包含 1-10 的数组，计算所有元素之和
2. 写一个函数判断字符串是否是回文
3. 将字符串 `"123"` 转换为整数（不用 `toInt()`，手动实现）

---

## 下一步

- [包与导入](packages.md) - 组织代码结构
- [控制流](control-flow.md) - 条件与循环
