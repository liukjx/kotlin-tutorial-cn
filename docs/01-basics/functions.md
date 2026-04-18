# 函数

> Kotlin 的函数功能强大，支持默认参数、命名参数、扩展函数、尾递归等特性。

## 函数定义

### 基本语法

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

### 表达式体

单行函数可以简化：

```kotlin
fun sum(a: Int, b: Int) = a + b

fun greet(name: String) = "Hello, $name!"
```

- 返回类型可省略（编译器自动推断）
- 使用 `=` 替代花括号和 `return`

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

---

## 参数

### 默认参数

```kotlin
fun greet(name: String, greeting: String = "Hello") {
    println("$greeting, $name!")
}

greet("Kotlin")                    // Hello, Kotlin!
greet("Kotlin", "Hi")              // Hi, Kotlin!
greet(greeting = "Hey", name = "Kotlin")  // 命名参数
```

### 命名参数

调用时可指定参数名，提高可读性，顺序可变：

```kotlin
fun createUser(name: String, age: Int, city: String, active: Boolean = true) {
    // ...
}

createUser(
    name = "张三",
    city = "北京",
    age = 25
)

// 与默认参数结合，跳过中间参数
createUser(
    name = "李四",
    age = 30
    // city 使用默认值（如果有的话）
)
```

### 可变参数 vararg

```kotlin
fun sum(vararg numbers: Int): Int {
    return numbers.sum()
}

sum(1, 2, 3, 4, 5)        // 15
sum(1)                    // 1
sum()                     // 0

// 传入数组
val arr = intArrayOf(1, 2, 3)
sum(*arr)                 // 展开运算符 *
```

可变参数在函数内部是一个数组。

---

## 返回值

### 单表达式返回

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b

fun isPositive(n: Int) = n > 0
```

### 提前返回

```kotlin
fun findUser(id: Int): User? {
    if (id < 0) return null
    // ... 查询逻辑
    return user
}
```

### 多返回值（使用 Pair 或 Triple）

```kotlin
fun getCoordinates(): Pair<Int, Int> {
    return Pair(10, 20)
}

val (x, y) = getCoordinates()

// 或使用 Triple
fun getPoint(): Triple<Int, Int, Int> {
    return Triple(1, 2, 3)
}

val (a, b, c) = getPoint()
```

---

## 局部函数

函数内部可以定义函数：

```kotlin
fun processUser(user: User): String {
    // 局部函数，可以访问外部变量
    fun validateName(): Boolean {
        return user.name.isNotBlank()
    }

    fun formatName(): String {
        return user.name.trim().uppercase()
    }

    if (!validateName()) {
        return "Invalid user"
    }
    return formatName()
}
```

---

## 尾递归函数

Kotlin 支持尾递归优化，避免栈溢出：

```kotlin
tailrec fun factorial(n: Int, acc: Int = 1): Int {
    return if (n <= 1) acc else factorial(n - 1, n * acc)
}

factorial(5)  // 120

// 等价于普通递归，但不会溢出
tailrec fun fibonacci(n: Int, a: Int = 0, b: Int = 1): Int {
    return if (n == 0) a else fibonacci(n - 1, b, a + b)
}

fibonacci(10)  // 55
```

**注意**：函数必须以递归调用作为最后一步操作才能使用 `tailrec`。

---

## 高阶函数

函数可以作为参数传递或返回：

```kotlin
// 函数类型：(Int, Int) -> Int
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

val sum = calculate(1, 2) { x, y -> x + y }      // 3
val product = calculate(2, 3) { x, y -> x * y }  // 6
```

### 函数作为返回值

```kotlin
fun getOperation(type: String): (Int, Int) -> Int {
    return when (type) {
        "add" -> { a, b -> a + b }
        "multiply" -> { a, b -> a * b }
        else -> { _, _ -> 0 }
    }
}

val op = getOperation("add")
op(2, 3)  // 5
```

---

## 扩展函数

为现有类添加新函数，无需继承：

```kotlin
fun String.isEmail(): Boolean {
    return this.contains("@")
}

"user@example.com".isEmail()  // true

fun Int.square(): Int = this * this

5.square()  // 25
```

### 扩展属性

```kotlin
val String.firstChar: Char?
    get() = if (isNotEmpty()) this[0] else null

"Kotlin".firstChar  // 'K'
```

---

## 中缀函数 infix

```kotlin
infix fun Int.times(str: String) = str.repeat(this)

5 times "Ha"  // "HaHaHaHaHa"

// 标准库示例
val map = mapOf("a" to 1)  // to 是中缀函数
```

**条件**：
- 必须是成员函数或扩展函数
- 只有一个参数
- 不能有默认参数

---

## 实战案例：PICO Spatial SDK

### 顶级函数 - 应用入口

在 PICO 项目中，每个示例应用都使用顶级函数作为入口：

```kotlin
// 文件：animation-0.10.7/app/src/main/java/.../Main.kt

/**
 * 主应用函数
 *
 * 这是一个顶级函数：
 * - 不属于任何类
 * - 可直接调用
 * - 类似于 Java 的静态方法
 *
 * @param scope 空间应用作用域，由 SDK 在应用启动时注入
 */
fun mainApp(scope: SpatialAppScope) =
    with(scope) {
        DefaultWindowContainer {
            PicoTheme {
                Box {
                    AnimationTypeTabBar()
                    HomePage()
                }
            }
        }
    }
```

**为什么用顶级函数？**
- Kotlin 风格：不强制所有代码都在类中
- 简洁：不需要创建无意义的类
- 与框架集成：SDK 通过反射调用顶级函数

### 扩展函数 - SDK 集成

PICO SDK 提供了大量扩展函数来简化开发：

```kotlin
// SDK 内部定义的扩展函数
fun PhysicalLengthConverter.dpToLength(dp: Float, unit: LengthUnit): Float {
    // 将 dp 转换为物理长度（米）
}

// 使用示例：spatialvideo 项目
VideoEntityAssembler.assembleVideoPanel(
    videoPanel,
    manager.player,
    converter.dpToLength(VideoEntityConfig.PANEL_WIDTH, LengthUnit.Meters),
    converter.dpToLength(VideoEntityConfig.PANEL_HEIGHT, LengthUnit.Meters),
)
```

**扩展函数的优势**：
- 不修改原类代码
- 调用方式自然 `converter.dpToLength(...)`
- 按需导入，避免污染命名空间

### 带默认参数的函数

```kotlin
// 文件：ComponentInfo.kt

data class ComponentInfo(
    val name: String,
    val description: String,
    val usage: String,
    val codeExample: String? = null,  // 默认 null
    val tips: String? = null          // 默认 null
)

// 创建时可选参数
val simple = ComponentInfo(
    name = "Button",
    description = "按钮",
    usage = "点击触发"
    // codeExample 和 tips 使用默认值
)

val detailed = ComponentInfo(
    name = "Slider",
    description = "滑动条",
    usage = "数值选择",
    codeExample = "Slider(value = 50f) { }",
    tips = "支持范围限制"
)
```

### 高阶函数 - UI 组件

在 Compose UI 中，高阶函数随处可见：

```kotlin
// Button 是一个高阶函数，onClick 参数是 Lambda
Button(
    onClick = { 
        viewModel.onPlayPauseClicked() 
    }
) {
    Text("播放")
}

// 自定义高阶函数
@Composable
fun ComponentCard(
    info: ComponentInfo,
    onExpanded: (Boolean) -> Unit  // 高阶函数参数
) {
    Card {
        Column {
            Text(text = info.name)
            IconButton(onClick = { onExpanded(true) }) {
                Icon(Icons.Default.Expand, "展开")
            }
        }
    }
}

// 使用
ComponentCard(
    info = ComponentInfos.BUTTON,
    onExpanded = { expanded ->
        println("展开状态: $expanded")
    }
)
```

### 函数设计最佳实践

```kotlin
// ✅ 好的设计：使用默认参数
fun loadData(
    url: String,
    timeout: Long = 5000,
    retryCount: Int = 3
) { ... }

// ✅ 好的设计：扩展函数增强 SDK
fun Entity.setPosition(x: Float, y: Float, z: Float) {
    transform.position = Vector3(x, y, z)
}

// ✅ 好的设计：顶级函数作为入口
fun mainApp(scope: SpatialAppScope) { ... }

// ❌ 不好的设计：过多的参数
fun loadData(url: String, timeout: Long, retry: Int, headers: Map<String, String>, ...)

// ✅ 改进：使用配置对象
data class LoadConfig(
    val url: String,
    val timeout: Long = 5000,
    val retryCount: Int = 3,
    val headers: Map<String, String> = emptyMap()
)

fun loadData(config: LoadConfig) { ... }
```

---

## 运算符重载

使用 `operator` 关键字重载运算符：

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }

    operator fun times(scalar: Int): Point {
        return Point(x * scalar, y * scalar)
    }
}

val p1 = Point(1, 2)
val p2 = Point(3, 4)

p1 + p2     // Point(4, 6)
p1 * 2      // Point(2, 4)
```

常用运算符：

| 表达式   | 函数名        |
|----------|---------------|
| `a + b`  | `plus`        |
| `a - b`  | `minus`       |
| `a * b`  | `times`       |
| `a / b`  | `div`         |
| `a % b`  | `rem`         |
| `a == b` | `equals`      |
| `a > b`  | `compareTo`   |
| `a[i]`   | `get` / `set` |
| `a in b` | `contains`    |

---

## 练习

### 1. 计算器

用默认参数和命名参数实现一个计算器函数，支持加减乘除。

### 2. 可变参数求平均

写一个函数 `average(vararg numbers: Double): Double`，计算平均值。

### 3. 扩展函数

为 `String` 添加扩展函数 `reverse()`，反转字符串。

### 4. 尾递归

用尾递归实现二分查找。

### 5. 高阶函数

实现一个 `repeat(n: Int, action: () -> Unit)` 函数，执行 action n 次。

---

## 下一步

- [Lambda 表达式](lambdas.md) - 函数式编程核心
- [内联函数](inline-functions.md) - 优化高阶函数性能
