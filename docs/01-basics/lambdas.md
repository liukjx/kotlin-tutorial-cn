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

---

## 实战案例：PICO Spatial SDK

### with() - 简化作用域访问

在 PICO 动画示例中，`with(scope)` 用于简化 `SpatialAppScope` 的成员访问：

```kotlin
// 文件：animation-0.10.7/app/src/main/java/.../Main.kt

fun mainApp(scope: SpatialAppScope) =
    with(scope) {  // 👈 在闭包内直接访问 scope 的成员
        DefaultWindowContainer {  // 等价于 scope.DefaultWindowContainer
            PicoTheme {
                Box {
                    AnimationTypeTabBar()
                    HomePage()
                }
            }
        }
    }
```

**为什么用 with？**
- `SpatialAppScope` 需要频繁调用其成员
- `with` 让代码更简洁，避免重复写 `scope.xxx`
- 类似于 "用 scope 做以下操作" 的语义

### let() - 空安全转换

在视频播放器初始化中，`let` 用于资源初始化后的链式调用：

```kotlin
// 文件：spatialvideo-0.10.7/app/src/main/java/.../VideoViewModel.kt

suspend fun initialize(converter: PhysicalLengthConverter, context: Context) {
    manager.setup(context, VIDEO_PATH)
    
    // 使用 let 进行链式操作
    VideoEntityAssembler.assembleVideoPanel(
        videoPanel,
        manager.player,
        converter.dpToLength(VideoEntityConfig.PANEL_WIDTH, LengthUnit.Meters),
        converter.dpToLength(VideoEntityConfig.PANEL_HEIGHT, LengthUnit.Meters),
    )
}
```

### apply() - 配置对象

在实体配置中，`apply` 用于配置 3D 实体属性：

```kotlin
// 文件：physics-0.10.7/app/src/main/java/.../Domino.kt

Entity().apply {
    name = "Domino"
    transform.position = Vector3(0f, 1f, 0f)
    transform.rotation = Quaternion.identity
}
```

**为什么用 apply？**
- 返回对象本身，适合链式调用
- 在闭包内直接访问属性
- 类似于 "对对象进行配置" 的语义

### also() - 副作用日志

在组件信息定义中，`also` 可用于调试日志：

```kotlin
// 文件：component-playground-0.10.7/app/src/main/java/.../ComponentInfo.kt

val BUTTON = ComponentInfo(
    name = "Button",
    description = "响应用户点击行为的基础按钮",
    usage = "用于触发操作，如提交表单、确认操作等"
).also {
    println("注册组件: ${it.name}")  // 副作用：打印日志
}
```

### 作用域函数选择指南

| 函数    | 返回值      | 使用场景 |
|---------|-------------|----------|
| `let`   | Lambda 结果 | 空安全转换、链式调用 |
| `run`   | Lambda 结果 | 同时需要对象和返回值 |
| `with`  | Lambda 结果 | 非空对象的多个操作 |
| `apply` | 对象本身    | 配置对象属性 |
| `also`  | 对象本身    | 副作用（日志、验证） |

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
