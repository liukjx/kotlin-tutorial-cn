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
