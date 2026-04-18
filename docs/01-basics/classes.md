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

---

## 实战案例：PICO Spatial SDK

### 枚举类：播放状态管理

在 PICO 视频播放器中，`PlaybackState` 枚举用于管理播放状态：

```kotlin
// 文件：spatialvideo-0.10.7/app/src/main/java/.../PlaybackState.kt

/**
 * 播放状态枚举
 *
 * @property value 状态值（用于序列化或跨进程传递）
 */
enum class PlaybackState(val value: Int) {
    INIT(0),      // 初始状态，播放器未初始化
    READY(1),     // 准备就绪，可以开始播放
    PLAYING(2),   // 正在播放
    PAUSED(3);    // 已暂停
}
```

**状态机设计**：
```
INIT ──setup()──> READY ──play()──> PLAYING
                      │                 │
                 completed          pause()
                      │                 │
                      ▼                 ▼
                   READY            PAUSED
                                          │
                                      resume()
                                          │
                                          ▼
                                      PLAYING
```

**使用示例**：
```kotlin
class VideoViewModel : ViewModel() {
    private val _videoState = MutableStateFlow(PlaybackState.READY)
    
    fun onPlayPauseClicked() {
        when (manager.state) {
            PlaybackState.READY -> {
                manager.play()
                _videoState.value = PlaybackState.PLAYING
            }
            PlaybackState.PLAYING -> {
                manager.pause()
                _videoState.value = PlaybackState.PAUSED
            }
            PlaybackState.PAUSED -> {
                manager.resume()
                _videoState.value = PlaybackState.PLAYING
            }
            else -> {}
        }
    }
}
```

### 单例对象：组件信息仓库

在 PICO 组件展示项目中，`object` 用于管理全局组件信息：

```kotlin
// 文件：component-playground-0.10.7/app/src/main/java/.../ComponentInfo.kt

/**
 * 所有预定义组件说明的集中入口
 *
 * 使用 `object` 表示整个应用只需要这一份静态数据
 */
object ComponentInfos {
    // 交互控件
    val BUTTON = ComponentInfo(
        name = "Button",
        description = "响应用户点击行为的基础按钮",
        usage = "用于触发操作，如提交表单、确认操作等"
    )
    
    val SWITCH = ComponentInfo(
        name = "Switch",
        description = "开关控件，在开/关两种状态之间切换",
        usage = "用于设置开关，如通知开关、深色模式等"
    )
    
    val SLIDER = ComponentInfo(
        name = "Slider",
        description = "滑动条，通过拖拽选择数值",
        usage = "用于连续数值选择，如音量、亮度等"
    )
    
    // 更多组件...
}

// 使用方式：类似常量仓库
val info = ComponentInfos.BUTTON
println(info.name)  // Button
```

**为什么用 object？**
- 整个应用只需要一份实例
- 类似静态常量仓库
- 统一管理所有组件说明
- 修改时不需要翻多个文件

### 数据类：组件信息模型

```kotlin
// 与 enum 和 object 配合使用
data class ComponentInfo(
    val name: String,
    val description: String,
    val usage: String,
    val codeExample: String? = null,
    val tips: String? = null
)

// 在 UI 中使用
@Composable
fun ComponentList() {
    val components = listOf(
        ComponentInfos.BUTTON,
        ComponentInfos.SWITCH,
        ComponentInfos.SLIDER
    )
    
    LazyColumn {
        items(components) { info ->
            ComponentCard(info)
        }
    }
}
```

### 设计模式对比

| 模式 | 使用场景 | 示例 |
|------|----------|------|
| `enum class` | 有限状态集合 | PlaybackState |
| `object` | 单例、常量仓库 | ComponentInfos |
| `data class` | 纯数据存储 | ComponentInfo |
| `sealed class` | 有限继承层级 | Result.Success/Error |

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

---

## 补充实战案例：伴生对象

### companion object - 类级成员

在 PICO 项目中，大量使用 `companion object` 管理类级别的常量和方法：

```kotlin
// 文件：physics-0.10.7/.../PhysicsManager.kt

/**
 * 物理管理器
 */
class PhysicsManager {
    
    companion object {
        // 类级常量
        const val DEFAULT_GRAVITY = -9.81f
        const val MAX_OBJECTS = 100
        
        // 工厂方法
        fun createDefault(): PhysicsManager {
            return PhysicsManager().apply {
                gravity = DEFAULT_GRAVITY
            }
        }
    }
    
    var gravity: Float = DEFAULT_GRAVITY
    // ...
}

// 使用
val manager = PhysicsManager.createDefault()
println(PhysicsManager.DEFAULT_GRAVITY)
```

```kotlin
// 文件：welcomespace-0.10.7/.../FullSpaceRoomViewModel.kt

/**
 * 房间视图模型
 *
 * Kotlin 知识点：companion object
 * - companion object 中的成员可以用类名直接访问
 * - 类似 Java 的 static，但更强大
 * - 可以实现接口、扩展函数
 */
class FullSpaceRoomViewModel : ViewModel() {
    
    companion object {
        // 资源路径常量
        const val SCENE_ROOM = "scene_room.glb"
        const val SCENE_SKY = "scene_sky.glb"
        
        // 标签
        private const val TAG = "FullSpaceRoomViewModel"
    }
    
    fun loadScene() {
        viewModelScope.launch {
            // 使用 companion object 中的常量
            val room = assetBundle.await().loadModel(SCENE_ROOM)
        }
    }
}
```

**companion object vs object 选择**：

| 特性 | companion object | object |
|------|------------------|--------|
| 所在位置 | 类内部 | 顶层 |
| 访问方式 | `类名.成员` | `对象名.成员` |
| 生命周期 | 跟随类 | 应用级单例 |
| 可实现接口 | ✅ 可以 | ✅ 可以 |
| 适用场景 | 类的静态成员 | 全局单例 |

**使用建议**：
- **object**：全局唯一、应用级共享（如日志工具、配置仓库）
- **companion object**：类相关的常量、工厂方法、静态工具
