# 属性与字段

> Kotlin 的属性语法简洁，底层自动生成 getter/setter，支持委托、延迟初始化等高级特性。

## 基本语法

```kotlin
class Person {
    var name: String = "Kotlin"  // 可变属性
    val id: Int = 1              // 只读属性
}
```

### 编译后等价的 Java 代码

```java
public class Person {
    private String name = "Kotlin";
    private final int id = 1;
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getId() { return id; }
}
```

---

## 自定义 Getter/Setter

### Getter

```kotlin
class Person(val name: String) {
    val greeting: String
        get() = "Hello, $name!"
    
    // 或简写
    val length: Int get() = name.length
}
```

### Setter

```kotlin
class Person {
    var age: Int = 0
        set(value) {
            if (value >= 0) field = value
        }
}
```

### 幕后字段 field

在 getter/setter 中访问实际存储的值：

```kotlin
class Person {
    var name: String = ""
        get() = field.uppercase()   // field 是实际存储的值
        set(value) {
            field = value.trim()
        }
}
```

**注意**：`field` 只能在 getter/setter 中使用。

---

## 计算属性

没有幕后字段，每次访问时计算：

```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = width * height
    
    val isSquare: Boolean
        get() = width == height
}
```

---

## 延迟初始化

### lateinit

用于稍后初始化的 `var`：

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```

**检查是否已初始化**：

```kotlin
if (::binding.isInitialized) {
    binding.textView.text = "Hello"
}
```

**限制**：
- 只能用于 `var`
- 不能用于基本类型（Int、Boolean 等）
- 不能用于可空类型

### by lazy

首次访问时初始化，线程安全：

```kotlin
class Person {
    val name: String by lazy {
        println("Computing name...")
        "Kotlin"
    }
}

val p = Person()
println(p.name)  // 首次访问时初始化
println(p.name)  // 之后直接返回缓存值
```

**参数**：

```kotlin
val name: String by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
    "Kotlin"  // 默认模式，线程安全
}

val name: String by lazy(LazyThreadSafetyMode.PUBLICATION) {
    "Kotlin"  // 可能多次初始化，但只有首次返回值有效
}

val name: String by lazy(LazyThreadSafetyMode.NONE) {
    "Kotlin"  // 无同步开销，单线程场景
}
```

---

## 委托属性

### by 关键字

将属性的 getter/setter 委托给另一个对象：

```kotlin
class Person {
    var name: String by NameDelegate()
}
```

### 自定义委托

```kotlin
import kotlin.reflect.KProperty

class NameDelegate {
    private var value: String = ""
    
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        println("Getting ${property.name}")
        return value
    }
    
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("Setting ${property.name} to $value")
        this.value = value
    }
}

val p = Person()
p.name = "Kotlin"  // Setting name to Kotlin
println(p.name)    // Getting name
                    // Kotlin
```

---

## 标准委托

### observable

属性变化时触发回调：

```kotlin
import kotlin.properties.Delegates

class Person {
    var name: String by Delegates.observable("Initial") { property, oldValue, newValue ->
        println("${property.name} changed from $oldValue to $newValue")
    }
}

val p = Person()
p.name = "Kotlin"  // name changed from Initial to Kotlin
```

### vetoable

在赋值前验证：

```kotlin
var age: Int by Delegates.vetoable(0) { property, oldValue, newValue ->
    if (newValue >= 0) {
        true  // 允许赋值
    } else {
        println("Age cannot be negative")
        false  // 拒绝赋值
    }
}
```

### notNull

确保属性在使用前已赋值：

```kotlin
var name: String by Delegates.notNull()

name = "Kotlin"
println(name)  // Kotlin
// 如果使用前未赋值，抛出 IllegalStateException
```

---

## Map 委托

用 Map 存储属性值：

```kotlin
class Person(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}

val p = Person(mapOf(
    "name" to "Kotlin",
    "age" to 10
))

println(p.name)  // Kotlin
println(p.age)   // 10

// 可变版本
class MutablePerson(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int by map
}
```

---

## 属性可见性

### Getter 和 Setter 的可见性

```kotlin
class Person {
    var name: String = "Kotlin"
        private set        // setter 私有
    
    var age: Int = 0
        internal set       // setter 模块内可见
    
    val id: Int = 1        // 只读属性无 setter
}
```

---

## 扩展属性

为现有类添加属性：

```kotlin
val String.firstChar: Char?
    get() = if (isNotEmpty()) this[0] else null

val String.lastChar: Char?
    get() = if (isNotEmpty()) this[lastIndex] else null

"Kotlin".firstChar  // 'K'
"".firstChar        // null
```

**注意**：扩展属性没有幕后字段，不能存储状态。

---

## 实战案例：PICO 项目中的属性应用

### by lazy - 延迟初始化单例

在 PICO 动画项目中，使用 `by lazy` 实现线程安全的单例模式：

```kotlin
// 文件：animation-0.10.7/app/src/main/java/.../Extensions.kt

/**
 * 枚举值缓存类
 */
class EnumValuesCache {
    private val cache = mutableMapOf<KClass<out Enum<*>>, Array<out Enum<*>>>()

    fun <T : Enum<T>> getValues(enumClass: KClass<T>): Array<T> {
        return cache.getOrPut(enumClass) {
            enumClass.java.enumConstants
                ?: error("No enum constants found")
        } as Array<T>
    }

    companion object {
        /**
         * 单例实例
         *
         * Kotlin 知识点：by lazy
         * - 延迟初始化：首次访问时才创建
         * - 线程安全：默认使用同步锁
         * - 全局复用：避免创建多个实例
         */
        val instance: EnumValuesCache by lazy { EnumValuesCache() }
    }
}

// 使用示例
val cache = EnumValuesCache.instance
// 首次访问时才创建实例
```

**为什么用 by lazy？**
- 延迟初始化：不使用时不创建，节省资源
- 线程安全：多线程环境下保证单例
- 简洁：比双重检查锁定更简洁

### lateinit - 延迟初始化属性

在 PICO ML 项目中，使用 `lateinit` 延迟初始化属性：

```kotlin
// 文件：spatialml-0.10.7/.../SrViewModel.kt

/**
 * 超分辨率视图模型
 *
 * Kotlin 知识点：lateinit var
 * - 延迟初始化：声明时不赋值
 * - 用于 var（可变属性）
 * - 不能用于基本类型
 */
class SrViewModel : ViewModel() {
    // 延迟初始化，在 init 块中赋值
    private lateinit var superResolution: SrAlgorithmImpl

    init {
        // 在合适的时机初始化
        viewModelScope.launch {
            superResolution = SrAlgorithmImpl()
        }
    }

    fun processImage(image: Bitmap) {
        // 检查是否已初始化
        if (::superResolution.isInitialized) {
            superResolution.process(image)
        }
    }
}
```

**lateinit vs by lazy 选择**：

| 特性 | lateinit | by lazy |
|------|----------|----------|
| 类型 | var | val |
| 初始化时机 | 手动控制 | 首次访问 |
| 可变性 | 可重新赋值 | 不可变 |
| 线程安全 | 需手动保证 | 自动保证 |
| 基本类型 | ❌ 不支持 | ✅ 支持 |
| 空检查 | 需手动检查 | 自动管理 |

### by remember - Compose 状态管理

在 PICO Compose UI 中，大量使用 `by remember` 委托：

```kotlin
// 文件：component-playground-0.10.7/.../InputPage.kt

@Composable
fun InputPage(onComponentClick: (ComponentInfo) -> Unit) {
    // Kotlin 知识点：属性委托
    // by remember: 将状态委托给 Compose 记忆系统
    var username by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    var passwordVisible by remember { mutableStateOf(false) }
    var searchQuery by remember { mutableStateOf("") }
    var volume by remember { mutableStateOf(50f) }

    // UI 组件
    TextField(
        value = username,
        onValueChange = { username = it }
    )

    Slider(
        value = volume,
        onValueChange = { volume = it }
    )
}
```

**by remember 的优势**：
- 简化状态访问：`username` 而不是 `username.value`
- 自动类型推断
- 与 Compose 重组机制完美配合

### 实际应用总结

| 场景 | 推荐方案 | PICO 示例 |
|------|----------|----------|
| 单例模式 | `by lazy` | `EnumValuesCache.instance` |
| 延迟初始化对象 | `lateinit var` | `SrAlgorithmImpl` |
| Compose 状态 | `by remember { mutableStateOf() }` | UI 组件状态 |
| 配置加载 | `by lazy` | 配置文件读取 |
| 依赖注入 | `lateinit var` | View Binding |

---

## 练习

### 1. 懒加载配置

创建一个 `Config` 类，用 `by lazy` 加载配置文件（首次访问时读取）。

### 2. 可观察计数器

创建一个 `Counter` 类，用 `observable` 在值变化时打印日志。

### 3. Map 委托

用 Map 委托实现动态属性的 `DynamicObject` 类。

---

## 下一步

- [接口](interfaces.md) - 接口与抽象
- [数据类](data-classes.md) - 数据类与解构
