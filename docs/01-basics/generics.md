# 泛型

> 泛型让类型参数化，实现代码复用和类型安全。

## 基本语法

```kotlin
class Box<T>(val value: T)

val intBox = Box(10)       // Box<Int>
val strBox = Box("Hello")  // Box<String>

val value: Int = intBox.value  // 类型安全
```

---

## 泛型函数

```kotlin
fun <T> singletonList(item: T): List<T> {
    return listOf(item)
}

val list = singletonList("Kotlin")  // List<String>
```

### 多类型参数

```kotlin
fun <K, V> toMap(pair: Pair<K, V>): Map<K, V> {
    return mapOf(pair.first to pair.second)
}

val map = toMap("key" to 1)  // Map<String, Int>
```

---

## 泛型类

```kotlin
class Container<T>(var item: T) {
    fun get(): T = item
    fun set(value: T) {
        item = value
    }
}

val container = Container("Kotlin")
container.set("Java")
val value: String = container.get()
```

---

## 类型约束

### 上界约束

限制类型参数必须是某个类型的子类型：

```kotlin
fun <T : Comparable<T>> max(a: T, b: T): T {
    return if (a > b) a else b
}

max(1, 2)         // ✅ Int 实现 Comparable
max("a", "b")     // ✅ String 实现 Comparable
// max(1, "a")    // ❌ 编译错误
```

### 多重约束

```kotlin
fun <T> process(item: T) where T : CharSequence, T : Comparable<T> {
    if (item > "a") {
        println(item)
    }
}

process("Kotlin")  // ✅
```

### 默认上界

```kotlin
// 默认上界是 Any?
fun <T> nullable(item: T): T? = item

// 显式指定非空上界
fun <T : Any> nonNull(item: T): T = item
```

---

## 型变

### 不变（Invariant）

默认情况下，泛型是不变的：

```kotlin
class Box<T>(val value: T)

val intBox: Box<Int> = Box(1)
// val anyBox: Box<Any> = intBox  // ❌ 编译错误
```

`Box<Int>` 不是 `Box<Any>` 的子类型。

### 协变（Covariant）- out

只能生产（输出），不能消费（输入）：

```kotlin
class Producer<out T>(private val value: T) {
    fun get(): T = value  // ✅ 只输出
    // fun set(value: T) { }  // ❌ 不能输入
}

val intProducer: Producer<Int> = Producer(1)
val anyProducer: Producer<Any> = intProducer  // ✅ 协变允许

// 原理：Producer<Int> 是 Producer<Any> 的子类型
```

**规则**：`out` 修饰符表示 T 只出现在返回值位置。

### 逆变（Contravariant）- in

只能消费（输入），不能生产（输出）：

```kotlin
class Consumer<in T> {
    fun consume(value: T) {  // ✅ 只输入
        println(value)
    }
    // fun get(): T { }  // ❌ 不能输出
}

val anyConsumer: Consumer<Any> = Consumer<Any>()
val intConsumer: Consumer<Int> = anyConsumer  // ✅ 逆变允许

// 原理：Consumer<Any> 是 Consumer<Int> 的子类型
```

**规则**：`in` 修饰符表示 T 只出现在参数位置。

---

## 类型投影

### 使用处型变

```kotlin
fun copy(from: Array<out Any>, to: Array<in Any>) {
    for (i in from.indices) {
        to[i] = from[i]
    }
}

val ints = arrayOf(1, 2, 3)
val anys = arrayOfNulls<Any>(3)
copy(ints, anys)  // ✅ 允许
```

### 星投影

当类型参数不重要时使用：

```kotlin
fun printSize(list: List<*>) {
    println(list.size)
}

printSize(listOf(1, 2, 3))
printSize(listOf("a", "b"))
```

---

## 具体化类型参数

内联函数可以用 `reified` 保留类型信息：

```kotlin
inline fun <reified T> isType(value: Any): Boolean {
    return value is T
}

isType<String>("Hello")  // true
isType<Int>("Hello")     // false

// 获取 Class
inline fun <reified T> getKClass(): KClass<T> = T::class
```

详见 [内联函数](inline-functions.md#具体化类型参数-reified)。

---

## 泛型擦除

运行时泛型类型信息被擦除：

```kotlin
val list1: List<Int> = listOf(1, 2, 3)
val list2: List<String> = listOf("a", "b")

// 运行时 list1 和 list2 都是 List
list1.javaClass == list2.javaClass  // true

// 不能检查泛型类型
// if (list is List<Int>) { }  // ❌ 编译错误
if (list is List<*>) { }       // ✅ 星投影
```

---

## 练习

### 1. 泛型栈

实现一个泛型 `Stack<T>`，支持 push、pop、peek。

### 2. 过滤器

实现 `filterByType<T>(list: List<Any>): List<T>`，过滤指定类型。

### 3. 型变

创建 `Producer<out T>` 和 `Consumer<in T>`，演示协变和逆变。

---

## 下一步

- [集合概述](collections-overview.md) - Kotlin 集合框架
- [协程基础](../02-coroutines/01-coroutines-basics.md) - 异步编程
