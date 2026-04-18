# 类与继承

> Kotlin 的类设计简洁而强大，支持主构造函数、数据类、密封类等特性。

## 定义类

### 基本语法

```kotlin
class Person(val name: String, var age: Int)

// 使用
val person = Person("Kotlin", 10)
println(person.name)  // Kotlin
person.age = 11
```

### 类体

```kotlin
class Person(val name: String) {
    var age: Int = 0
    
    fun greet() {
        println("Hello, I'm $name")
    }
}
```

---

## 构造函数

### 主构造函数

在类头声明，是类定义的一部分：

```kotlin
class Person(val name: String, var age: Int)

// 如果没有注解或可见性修饰符，可省略 constructor 关键字
class Person constructor(val name: String)
```

### 初始化块

主构造函数不能包含代码，用 `init` 块初始化：

```kotlin
class Person(val name: String) {
    var age: Int = 0
    
    init {
        println("Person created: $name")
        require(name.isNotBlank()) { "Name cannot be blank" }
    }
}
```

### 次构造函数

用 `constructor` 关键字声明：

```kotlin
class Person(val name: String) {
    var age: Int = 0
    
    constructor(name: String, age: Int) : this(name) {
        this.age = age
    }
}

val p1 = Person("Kotlin")
val p2 = Person("Kotlin", 10)
```

### 默认参数（推荐）

用默认参数替代次构造函数：

```kotlin
class Person(
    val name: String,
    var age: Int = 0,
    var city: String = "Beijing"
)

val p1 = Person("Kotlin")
val p2 = Person("Kotlin", 10)
val p3 = Person("Kotlin", city = "Shanghai")
```

---

## 属性

### 读写属性

```kotlin
class Person {
    var name: String = "Unknown"  // 可变
    val id: Int = 0               // 只读
}
```

### 延迟初始化 lateinit

```kotlin
class Person {
    lateinit var name: String
    
    fun init() {
        name = "Kotlin"  // 在使用前必须初始化
    }
}

// 检查是否已初始化
if (::name.isInitialized) {
    println(name)
}
```

**限制**：
- 只能用于 `var`
- 不能用于基本类型
- 不能用于可空属性

### 懒初始化 by lazy

```kotlin
class Person {
    val name: String by lazy {
        println("Computing name...")
        "Kotlin"
    }
}

val p = Person()
// 此时 name 还没计算
println(p.name)  // 首次访问时才初始化
// 输出：
// Computing name...
// Kotlin
```

---

## 继承

所有类默认继承自 `Any`（类似 Java 的 Object）。

### open 关键字

Kotlin 类默认是 `final`，不能继承。用 `open` 允许继承：

```kotlin
open class Animal(val name: String) {
    open fun speak() {
        println("$name makes a sound")
    }
}

class Dog(name: String) : Animal(name) {
    override fun speak() {
        println("$name barks")
    }
}
```

### 调用父类方法

```kotlin
open class Animal {
    open fun speak() { println("Sound") }
}

class Dog : Animal() {
    override fun speak() {
        super.speak()  // 调用父类方法
        println("Bark")
    }
}
```

### 覆盖属性

```kotlin
open class Animal {
    open val legs: Int = 4
}

class Dog : Animal() {
    override val legs: Int = 4
}
```

---

## 抽象类

```kotlin
abstract class Animal {
    abstract val name: String
    abstract fun speak()
    
    fun sleep() {
        println("$name is sleeping")
    }
}

class Dog(override val name: String) : Animal() {
    override fun speak() {
        println("$name barks")
    }
}
```

---

## 接口

```kotlin
interface Swimmer {
    val speed: Int
    
    fun swim() {
        println("Swimming at speed $speed")
    }
}

interface Flyer {
    fun fly()
}

class Duck : Swimmer, Flyer {
    override val speed: Int = 10
    
    override fun fly() {
        println("Duck is flying")
    }
}
```

### 解决钻石问题

```kotlin
interface A {
    fun foo() { println("A") }
}

interface B {
    fun foo() { println("B") }
}

class C : A, B {
    override fun foo() {
        super<A>.foo()  // 指定调用哪个接口的实现
        super<B>.foo()
    }
}
```

---

## 可见性修饰符

| 修饰符      | 类成员       | 顶层声明     |
|-------------|--------------|--------------|
| `public`    | 所有可见     | 所有可见     |
| `internal`  | 模块内可见   | 模块内可见   |
| `protected` | 子类可见     | ❌ 不可用    |
| `private`   | 类内可见     | 文件内可见   |

```kotlin
open class Person {
    public val name: String = ""      // 默认 public
    private val id: Int = 0           // 类内可见
    protected val age: Int = 0        // 子类可见
    internal val email: String = ""   // 模块内可见
}
```

---

## 数据类 data class

自动生成 `equals`、`hashCode`、`toString`、`copy`：

```kotlin
data class User(val name: String, val age: Int)

val u1 = User("Kotlin", 10)
val u2 = User("Kotlin", 10)

u1 == u2              // true（内容相等）
u1.hashCode() == u2.hashCode()  // true
println(u1)           // User(name=Kotlin, age=10)

val u3 = u1.copy(age = 11)  // 复制并修改部分属性
```

---

## 密封类 sealed class

限制继承层级，用于状态管理：

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
    object Loading : Result()
}

fun handle(result: Result): String = when (result) {
    is Result.Success -> "Data: ${result.data}"
    is Result.Error -> "Error: ${result.message}"
    Result.Loading -> "Loading..."
    // 不需要 else 分支，编译器保证覆盖所有情况
}
```

---

## 枚举类 enum class

```kotlin
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}

enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}

Direction.NORTH.name      // "NORTH"
Direction.NORTH.ordinal   // 0
Direction.values()        // 所有值
Direction.valueOf("NORTH")  // NORTH
```

---

## 对象声明 object

### 单例

```kotlin
object Database {
    fun connect() { println("Connecting...") }
}

Database.connect()  // 直接通过类名调用
```

### 伴生对象

类似静态成员：

```kotlin
class Person(val name: String) {
    companion object {
        val DEFAULT_NAME = "Unknown"
        
        fun create(): Person = Person(DEFAULT_NAME)
    }
}

Person.DEFAULT_NAME
Person.create()
```

---

## 练习

### 1. 用户类

创建 `User` 数据类，包含 name、email、age，实现复制和比较。

### 2. 动物继承

创建 Animal 抽象类，Dog、Cat 子类，实现 speak() 方法。

### 3. 密封类状态

用密封类表示网络请求状态：Loading、Success、Error。

---

## 下一步

- [属性与字段](properties.md) - 深入属性机制
- [接口](interfaces.md) - 接口与多继承
