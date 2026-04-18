# 内联函数

> 使用 `inline` 关键字优化高阶函数性能，消除 Lambda 的运行时开销。

## 为什么需要内联？

Lambda 在 Kotlin 中会被编译成匿名类，每次调用都会创建对象，有性能开销。

```kotlin
// 不使用 inline
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// 编译后会创建 Function2 对象
calculate(1, 2) { x, y -> x + y }
```

---

## inline 关键字

使用 `inline` 编译器会将函数代码直接"复制"到调用处：

```kotlin
inline fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// 调用处编译后等价于：
// val a = 1
// val b = 2
// val result = a + b
```

### 示例对比

```kotlin
// 不使用 inline
fun doSomething(action: () -> Unit) {
    println("Before")
    action()
    println("After")
}

// 调用 1000 次会创建 1000 个 Function 对象


// 使用 inline
inline fun doSomething(action: () -> Unit) {
    println("Before")
    action()
    println("After")
}

// 调用 1000 次，零对象创建
```

---

## noinline

如果不想内联某个 Lambda 参数，用 `noinline`：

```kotlin
inline fun process(
    a: () -> Unit,        // 会被内联
    noinline b: () -> Unit  // 不会被内联
) {
    a()  // 内联展开
    b()  // 仍然是函数调用
    
    // 可以将 noinline 参数赋值给变量
    val func = b
}
```

**何时使用 noinline**：
- 需要将 Lambda 存储为变量
- 需要将 Lambda 作为参数传递给其他函数

---

## crossinline

当内联函数在另一个执行上下文（如嵌套类、另一个 Lambda）中调用 Lambda 时，用 `crossinline`：

```kotlin
inline fun runInThread(crossinline action: () -> Unit) {
    Thread {
        action()  // 在新线程中调用
    }.start()
}

// crossinline 禁止在 action 中使用 return
runInThread {
    // return  // ❌ 编译错误
    println("Running in thread")
}
```

**为什么需要**：内联函数的 `return` 会直接返回外层函数，但在嵌套上下文中这是不安全的。

---

## 非局部返回

内联函数支持"非局部返回"——在 Lambda 中直接返回外层函数：

```kotlin
inline fun findUser(users: List<User>, predicate: (User) -> Boolean): User? {
    users.forEach {
        if (predicate(it)) {
            return it  // 直接返回 findUser
        }
    }
    return null
}

fun main() {
    val users = listOf(User("Alice"), User("Bob"))
    
    val result = findUser(users) {
        if (it.name == "Bob") {
            return  // 直接返回 main 函数！
        }
        false
    }
}
```

---

## 具体化类型参数 reified

泛型类型在运行时会被擦除，但内联函数可以用 `reified` 保留类型信息：

```kotlin
// 普通泛型，运行时无法获取 T 的类型
fun <T> isType(value: Any): Boolean {
    // return value is T  // ❌ 编译错误
    return false
}

// 使用 reified
inline fun <reified T> isType(value: Any): Boolean {
    return value is T  // ✅ 可以运行
}

isType<String>("Hello")  // true
isType<Int>("Hello")     // false

// 获取泛型的 Class
inline fun <reified T> getKClass(): KClass<T> = T::class

// 创建泛型数组
inline fun <reified T> createArray(size: Int): Array<T> = arrayOfNulls<T>(size) as Array<T>
```

### 常见用途

```kotlin
// Gson 解析
inline fun <reified T> fromJson(json: String): T {
    return Gson().fromJson(json, T::class.java)
}

val user: User = fromJson("""{"name":"Kotlin"}""")

// Intent 获取 Parcelable
inline fun <reified T : Parcelable> Intent.getParcelableExtra(key: String): T? {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        getParcelableExtra(key, T::class.java)
    } else {
        @Suppress("DEPRECATION")
        getParcelableExtra(key)
    }
}
```

---

## 何时使用 inline

### ✅ 应该使用

- 高阶函数（函数参数为 Lambda）
- 需要具体化类型参数（`reified`）
- 性能敏感的热点代码

### ❌ 不应该使用

- 普通函数（无 Lambda 参数）
- 函数体很大的函数（会导致代码膨胀）
- 递归函数
- 私有函数（内联对私有函数意义不大）

---

## 标准库内联函数

Kotlin 标准库中的许多函数都是内联的：

```kotlin
// 集合操作
inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T>
inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R>
inline fun <T> Iterable<T>.forEach(action: (T) -> Unit)

// 作用域函数
inline fun <T, R> with(receiver: T, block: T.() -> R): R
inline fun <T> T.apply(block: T.() -> Unit): T
inline fun <T, R> T.let(block: (T) -> R): R
inline fun <T> T.also(block: (T) -> Unit): T
inline fun <T, R> T.run(block: T.() -> R): R

// 工具函数
inline fun <T> synchronized(lock: Any, block: () -> T): T
inline fun repeat(times: Int, action: (Int) -> Unit)
```

---

## 练习

### 1. 内联计时器

实现一个内联函数 `measureTime(action: () -> T): Pair<T, Long>`，返回执行结果和耗时（毫秒）。

### 2. reified 过滤器

实现 `filterByType<T>(list: List<Any>): List<T>`，过滤出指定类型的元素。

### 3. 性能对比

对比 `inline` 和普通高阶函数在大循环中的性能差异。

---

## 下一步

- [类与继承](classes.md) - 面向对象编程
- [集合概述](collections-overview.md) - 集合框架
