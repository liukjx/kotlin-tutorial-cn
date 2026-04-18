# 控制流

> Kotlin 的控制流语句包括条件表达式、when 表达式、循环和范围表达式。

## If 表达式

Kotlin 的 `if` 是表达式，有返回值：

```kotlin
// 基本用法
if (a > b) {
    println("a is bigger")
} else {
    println("b is bigger")
}

// 作为表达式
val max = if (a > b) a else b

// 多分支
val result = if (score >= 90) {
    "A"
} else if (score >= 80) {
    "B"
} else if (score >= 70) {
    "C"
} else {
    "D"
}
```

### 注意事项

- 如果 `if` 作为表达式使用，**必须有 `else` 分支**
- 代码块中的最后一行是返回值

```kotlin
val max = if (a > b) {
    println("Choosing a")
    a
} else {
    println("Choosing b")
    b
}
```

---

## When 表达式

Kotlin 的 `when` 替代 Java 的 `switch`，功能更强大。

### 基本用法

```kotlin
when (x) {
    1 -> println("One")
    2 -> println("Two")
    3, 4 -> println("Three or Four")
    else -> println("Unknown")
}
```

### 作为表达式

```kotlin
val description = when (x) {
    1 -> "One"
    2 -> "Two"
    else -> "Other"
}
// 作为表达式必须有 else 分支（除非编译器能证明已覆盖所有情况）
```

### 多条件分支

```kotlin
when (x) {
    0, 1 -> println("0 or 1")
    in 2..10 -> println("2-10")
    !in 11..20 -> println("Not in 11-20")
    else -> println("Other")
}
```

### 范围检查

```kotlin
val grade = when (score) {
    in 90..100 -> "A"
    in 80..89 -> "B"
    in 70..79 -> "C"
    in 60..69 -> "D"
    else -> "F"
}
```

### 类型检查

```kotlin
when (obj) {
    is Int -> println("Integer: $obj")
    is String -> println("String: ${obj.length}")
    is Double -> println("Double: $obj")
    else -> println("Unknown type")
}
```

### 无参数 when

```kotlin
when {
    score >= 90 -> println("Excellent")
    score >= 60 -> println("Passed")
    else -> println("Failed")
}
```

### 捕获条件值

```kotlin
when (val result = compute()) {
    is Success -> println(result.data)
    is Error -> println(result.message)
}
// result 只在 when 块内可见
```

---

## For 循环

### 基本用法

```kotlin
for (i in 1..5) {
    println(i)  // 1, 2, 3, 4, 5
}
```

### 范围表达式

```kotlin
for (i in 1..10) {           // 1 到 10（包含）
    println(i)
}

for (i in 1 until 10) {      // 1 到 9（不包含 10）
    println(i)
}

for (i in 10 downTo 1) {     // 10 到 1（递减）
    println(i)
}

for (i in 1..10 step 2) {    // 1, 3, 5, 7, 9
    println(i)
}

for (i in 10 downTo 1 step 2) {  // 10, 8, 6, 4, 2
    println(i)
}
```

### 遍历集合

```kotlin
val items = listOf("apple", "banana", "cherry")

for (item in items) {
    println(item)
}

// 带索引
for ((index, value) in items.withIndex()) {
    println("$index: $value")
}

// 只用索引
for (i in items.indices) {
    println("$i: ${items[i]}")
}
```

### 遍历 Map

```kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)

for ((key, value) in map) {
    println("$key = $value")
}
```

---

## While 循环

### while

```kotlin
var i = 0
while (i < 5) {
    println(i)
    i++
}
```

### do-while

```kotlin
var input: String
do {
    input = readln()
    println("You entered: $input")
} while (input != "quit")
```

---

## 循环控制

### break - 终止循环

```kotlin
for (i in 1..10) {
    if (i == 5) break
    println(i)  // 1, 2, 3, 4
}
```

### continue - 跳过本次迭代

```kotlin
for (i in 1..10) {
    if (i % 2 == 0) continue
    println(i)  // 1, 3, 5, 7, 9
}
```

### 标签（Label）

Kotlin 支持标签，用于控制嵌套循环：

```kotlin
outer@ for (i in 1..3) {
    for (j in 1..3) {
        if (i == 2 && j == 2) {
            break@outer  // 跳出外层循环
        }
        println("i=$i, j=$j")
    }
}
```

```kotlin
outer@ for (i in 1..3) {
    for (j in 1..3) {
        if (j == 2) {
            continue@outer  // 跳到外层循环的下一次迭代
        }
        println("i=$i, j=$j")
    }
}
```

---

## 范围（Range）

### 创建范围

```kotlin
val range = 1..10          // 1 到 10（包含）
val chars = 'a'..'z'       // a 到 z
val descending = 10 downTo 1  // 递减范围
val stepRange = 1..10 step 2  // 步进范围
```

### 检查包含

```kotlin
if (5 in 1..10) {
    println("5 is in range")
}

if ('e' in 'a'..'z') {
    println("e is a lowercase letter")
}

if (15 !in 1..10) {
    println("15 is not in range")
}
```

### 遍历范围

```kotlin
for (i in 1..10) { ... }
for (c in 'a'..'z') { ... }
```

### 范围方法

```kotlin
val range = 1..10

range.first              // 1
range.last               // 10
range.step               // 1（默认步长）

(1..10).random()         // 随机数
```

---

## 练习

### 1. 成绩等级

写一个函数 `getGrade(score: Int): String`，根据分数返回等级：
- 90-100: A
- 80-89: B
- 70-79: C
- 60-69: D
- 0-59: F

### 2. 九九乘法表

使用嵌套循环打印九九乘法表：
```
1x1=1
1x2=2  2x2=4
1x3=3  2x3=6  3x3=9
...
```

### 3. 猜数字游戏

生成 1-100 的随机数，让用户猜测，给出"太大"、"太小"提示，直到猜对。

### 4. 质数判断

写一个函数判断一个数是否是质数。

### 5. 标签练习

使用标签跳出嵌套循环，找到第一个能被 7 整除的数（在 10x10 的范围内）。

---

## 下一步

- [函数](functions.md) - 深入学习函数特性
- [Lambda 表达式](lambdas.md) - 函数式编程基础
