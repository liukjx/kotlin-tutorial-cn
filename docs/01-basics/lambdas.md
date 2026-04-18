# Lambda 表达式

> Lambda 是匿名函数的简洁写法，是 Kotlin 函数式编程的核心。

## 基本语法

```kotlin
{ x: Int, y: Int -> x + y }
```

- 用花括号 `{}` 包裹
- 参数在 `->` 前面
- 函数体在 `->` 后面
- 最后一行是返回值

### 完整写法

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }

sum(1, 2)  // 3
```

### 类型推断简化

```kotlin
val sum = { x: Int, y: Int -> x + y }

// 如果类型已知，参数类型可省略
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

### 无参数

```kotlin
val greet: () -> String = { "Hello, Kotlin!" }

greet()  // "Hello, Kotlin!"
```

---

## 高阶函数

### Lambda 作为参数

```kotlin
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// 传入 Lambda
calculate(1, 2) { x, y -> x + y }      // 3
calculate(1, 2) { x, y -> x * y }      // 2
```

**约定**：如果 Lambda 是最后一个参数，可以放在括号外面：

```kotlin
calculate(1, 2) { x, y -> x + y }
// 等价于
calculate(1, 2, { x, y -> x + y })
```

### Lambda 作为返回值

```kotlin
fun getGreeter(greeting: String): (String) -> String {
    return { name -> "$greeting, $name!" }
}

val greeter = getGreeter("Hello")
greeter("Kotlin")  // "Hello, Kotlin!"
```

---

## it 参数

当 Lambda 只有一个参数时，可以用 `it` 代替：

```kotlin
val double: (Int) -> Int = { it * 2 }

double(5)  // 10

// 集合操作
val numbers = listOf(1, 2, 3, 4, 5)

numbers.filter { it > 2 }     // [3, 4, 5]
numbers.map { it * 2 }        // [2, 4, 6, 8, 10]
```

---

## 集合操作

Lambda 常用于集合操作：

### filter - 过滤

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6)

val evens = numbers.filter { it % 2 == 0 }  // [2, 4, 6]
val odds = numbers.filter { it % 2 == 1 }   // [1, 3, 5]
```

### map - 映射

```kotlin
val numbers = listOf(1, 2, 3)

val doubled = numbers.map { it * 2 }        // [2, 4, 6]
val strings = numbers.map { "Num: $it" }    // ["Num: 1", "Num: 2", "Num: 3"]
```

### reduce - 累积

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

val sum = numbers.reduce { acc, n -> acc + n }      // 15
val product = numbers.reduce { acc, n -> acc * n }  // 120
```

### fold - 带初始值的累积

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

val sum = numbers.fold(0) { acc, n -> acc + n }     // 15
val sumFrom10 = numbers.fold(10) { acc, n -> acc + n }  // 25
```

### forEach - 遍历

```kotlin
numbers.forEach { println(it) }

// 带索引
numbers.forEachIndexed { index, value ->
    println("$index: $value")
}
```

### all / any / none

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

numbers.all { it > 0 }     // true（全部满足）
numbers.any { it > 4 }     // true（存在满足）
numbers.none { it < 0 }    // true（没有满足）
```

### find / firstOrNull

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

numbers.find { it > 3 }         // 4（第一个满足的）
numbers.firstOrNull { it > 10 } // null
```

### groupBy - 分组

```kotlin
data class Person(val name: String, val city: String)

val people = listOf(
    Person("张三", "北京"),
    Person("李四", "上海"),
    Person("王五", "北京")
)

val byCity = people.groupBy { it.city }
// {北京=[Person("张三", "北京"), Person("王五", "北京")], 上海=[Person("李四", "上海")]}
```

### partition - 拆分

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6)

val (evens, odds) = numbers.partition { it % 2 == 0 }
// evens = [2, 4, 6]
// odds = [1, 3, 5]
```

### flatMap - 扁平化

```kotlin
val lists = listOf(listOf(1, 2), listOf(3, 4), listOf(5))

val flattened = lists.flatMap { it }  // [1, 2, 3, 4, 5]
```

---

## 闭包

Lambda 可以捕获外部变量：

```kotlin
fun counter(): () -> Int {
    var count = 0
    return {
        count++
        count
    }
}

val c = counter()
c()  // 1
c()  // 2
c()  // 3
```

---

## 函数引用

使用 `::` 将函数转换为 Lambda：

```kotlin
fun isEven(n: Int) = n % 2 == 0

val numbers = listOf(1, 2, 3, 4, 5)
numbers.filter(::isEven)  // [2, 4]

// 类方法引用
val isBlank: (String) -> Boolean = String::isBlank
```

### 构造函数引用

```kotlin
data class User(val name: String)

val createUser: (String) -> User = ::User
createUser("Kotlin")  // User("Kotlin")

// 在 map 中使用
val names = listOf("Alice", "Bob", "Charlie")
val users = names.map(::User)
```

---

## 带接收者的 Lambda

Lambda 可以有接收者，类似扩展函数：

```kotlin
val buildString: StringBuilder.() -> Unit = {
    append("Hello, ")
    append("Kotlin!")
}

val sb = StringBuilder()
sb.buildString()
println(sb)  // Hello, Kotlin!

// 或使用 apply
val result = StringBuilder().apply {
    append("Hello, ")
    append("Kotlin!")
}.toString()
```

### 标准库函数

```kotlin
// apply - 返回接收者
val user = User().apply {
    name = "Kotlin"
    age = 10
}

// let - 返回 Lambda 结果
val length = "Kotlin".let {
    println(it)  // Kotlin
    it.length    // 返回值
}

// run - 带接收者的 let
val result = "Kotlin".run {
    length.uppercase()
}

// with - 非扩展形式的 run
val result = with(user) {
    "$name is $age years old"
}

// also - 返回接收者（用于副作用）
val user = createUser().also {
    println("Created user: ${it.name}")
}

// takeIf - 条件满足返回接收者，否则 null
val file = File("data.txt")
    .takeIf { it.exists() }
    ?.readText()

// takeUnless - 条件不满足返回接收者
val file = File("data.txt")
    .takeUnless { it.isHidden }
    ?.readText()
```

---

## 练习

### 1. 过滤与映射

给定学生列表 `List<Student>`，找出年龄大于 20 的学生姓名。

### 2. 分组统计

给定单词列表，按首字母分组，统计每组数量。

### 3. 链式操作

将 `[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]`：
- 过滤出偶数
- 映射为平方
- 求和

### 4. 自定义高阶函数

实现 `repeat(times: Int, action: (Int) -> Unit)`，执行 action times 次，传入当前索引。

### 5. 构建器模式

用带接收者的 Lambda 实现一个简单的 HTML 构建器：

```kotlin
html {
    body {
        p("Hello, Kotlin!")
    }
}
```

---

## 下一步

- [内联函数](inline-functions.md) - 优化 Lambda 性能
- [集合操作](collection-filtering.md) - 更多集合函数
