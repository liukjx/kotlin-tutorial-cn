# 接口

> Kotlin 接口支持抽象方法和默认实现，可以实现多继承效果。

## 定义接口

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}
```

- `click()` 是抽象方法，必须由实现类覆写
- `showOff()` 有默认实现，可覆写可不覆写

---

## 实现接口

```kotlin
class Button : Clickable {
    override fun click() {
        println("Button clicked")
    }
}

val button = Button()
button.click()    // Button clicked
button.showOff()  // I'm clickable!
```

---

## 多继承

### 实现多个接口

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("Clickable")
}

interface Focusable {
    fun setFocus(b: Boolean)
    fun showOff() = println("Focusable")
}

class Button : Clickable, Focusable {
    override fun click() {
        println("Clicked")
    }
    
    override fun setFocus(b: Boolean) {
        println("Focus: $b")
    }
    
    // 必须覆写冲突的方法
    override fun showOff() {
        super<Clickable>.showOff()  // 调用 Clickable 的实现
        super<Focusable>.showOff()  // 调用 Focusable 的实现
    }
}
```

---

## 接口中的属性

### 抽象属性

```kotlin
interface Person {
    val name: String  // 抽象属性，实现类必须提供值
    
    fun greet() = println("Hello, $name")
}

class Student(override val name: String) : Person

val s = Student("Kotlin")
s.greet()  // Hello, Kotlin
```

### 带 getter 的属性

```kotlin
interface Person {
    val name: String
    
    // 不能初始化，但可以有 getter
    val greeting: String
        get() = "Hello, $name"
}
```

---

## 接口继承

```kotlin
interface Clickable {
    fun click()
}

interface Animated {
    fun animate()
}

// 接口继承多个接口
interface Widget : Clickable, Animated {
    fun render()
}

class Button : Widget {
    override fun click() { }
    override fun animate() { }
    override fun render() { }
}
```

---

## 接口 vs 抽象类

| 特性           | 接口           | 抽象类          |
|----------------|----------------|-----------------|
| 多继承         | ✅ 支持        | ❌ 单继承       |
| 状态           | ❌ 无状态      | ✅ 有状态       |
| 构造函数       | ❌ 无          | ✅ 有           |
| 默认实现       | ✅ 支持        | ✅ 支持         |
| 属性           | 抽象或有 getter| 任意类型        |

### 何时用接口

- 定义行为（如 `Comparable`、`Runnable`）
- 需要多继承
- 无状态

### 何时用抽象类

- 需要共享状态
- 需要构造函数
- 有部分实现

---

## 函数式接口（SAM）

只有一个抽象方法的接口称为函数式接口（SAM），可用 Lambda 转换：

```kotlin
fun interface ClickListener {
    fun onClick()
}

fun setClickListener(listener: ClickListener) {
    listener.onClick()
}

// 传统写法
setClickListener(object : ClickListener {
    override fun onClick() {
        println("Clicked")
    }
})

// SAM 转换（Lambda）
setClickListener { println("Clicked") }
```

---

## 接口中的默认方法冲突

当多个接口有相同签名的默认方法时：

```kotlin
interface A {
    fun foo() { println("A") }
}

interface B {
    fun foo() { println("B") }
}

class C : A, B {
    override fun foo() {
        super<A>.foo()  // 调用 A 的实现
        super<B>.foo()  // 调用 B 的实现
        println("C")    // 自己的实现
    }
}
```

---

## 实战案例：PICO 接口委托模式

### 接口委托 - 组合优于继承

在 PICO 欢迎空间项目中，使用接口委托实现多重继承效果：

```kotlin
// 文件：welcomespace-0.10.7/.../ItemSelection.kt

/**
 * 选中状态管理接口
 */
interface ItemSelector {
    fun select(name: String)
    fun deselect(name: String)
    fun deselectAll()
    fun isSelected(name: String): Boolean
}

/**
 * 接口实现
 */
class ItemSelectorImpl : ItemSelector {
    private val selectedItems = mutableStateMapOf<String, Boolean>()
    
    override fun select(name: String) {
        selectedItems[name] = true
    }
    
    override fun deselect(name: String) {
        selectedItems[name] = false
    }
    
    override fun deselectAll() {
        selectedItems.clear()
    }
    
    override fun isSelected(name: String): Boolean {
        return selectedItems[name] ?: false
    }
}

/**
 * 视图模型 - 使用委托模式
 *
 * Kotlin 知识点：接口委托
 * - by 关键字将接口实现委托给另一个对象
 * - 类似多重继承，但更灵活
 * - ViewModel 自动拥有 ItemSelector 的所有方法
 */
class FurnitureLibraryViewModel : 
    ViewModel(),
    ItemSelector by ItemSelectorImpl() {  // 委托给 ItemSelectorImpl
    
    // ViewModel 自动拥有 select、deselect、isSelected 等方法
    // 无需手动实现，无需手动转发调用
    
    fun onFurnitureClicked(name: String) {
        if (isSelected(name)) {
            deselect(name)
        } else {
            select(name)
        }
    }
}

// 使用示例
val viewModel = FurnitureLibraryViewModel()
viewModel.select("chair")  // 自动调用 ItemSelectorImpl 的方法
viewModel.isSelected("chair")  // true
```

**委托模式的优势**：
- **代码复用**：多个类可以共享同一个接口实现
- **组合优于继承**：更灵活的代码组织
- **减少样板代码**：无需手动实现每个方法
- **运行时替换**：可以动态替换委托对象

### 委托 vs 继承对比

```kotlin
// ❌ 继承方式：Java 只能单继承
class FurnitureLibraryViewModel : ViewModel(), ItemSelector {
    private val selector = ItemSelectorImpl()
    
    // 必须手动转发所有方法
    override fun select(name: String) = selector.select(name)
    override fun deselect(name: String) = selector.deselect(name)
    override fun deselectAll() = selector.deselectAll()
    override fun isSelected(name: String) = selector.isSelected(name)
}

// ✅ 委托方式：Kotlin 自动生成转发代码
class FurnitureLibraryViewModel : 
    ViewModel(),
    ItemSelector by ItemSelectorImpl()
```

### 实际应用场景

| 场景 | 委托优势 | 示例 |
|------|----------|----------|
| 多个接口实现 | 避免方法冲突 | ViewModel 实现多个功能接口 |
| 代码复用 | 共享实现 | 多个类共享同一套选中逻辑 |
| 测试 | 可注入 mock | 单元测试时替换委托对象 |
| 运行时切换 | 动态替换策略 | 根据配置切换实现 |

---

## 练习

### 1. 可比较接口

创建 `Comparable` 接口，让 `Person` 类按年龄比较。

### 2. 多接口实现

创建 `Swimmer`、`Flyer` 接口，让 `Duck` 类实现两者。

### 3. SAM 转换

创建 `Calculator` 函数式接口，用 Lambda 实现。

---

## 下一步

- [数据类](data-classes.md) - 简化数据类定义
- [密封类](sealed-classes.md) - 限制继承层级
