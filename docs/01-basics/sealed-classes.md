# 密封类

> 密封类（sealed class）限制继承层级，用于表示受限的类型集合，常用于状态管理。

## 定义密封类

```kotlin
sealed class Result

class Success(val data: String) : Result()
class Error(val message: String) : Result()
object Loading : Result()
```

- 所有子类必须在同一文件中定义（Kotlin 1.5+ 允许同一包）
- 编译器知道所有可能的子类
- `when` 表达式不需要 `else` 分支

---

## when 表达式

```kotlin
fun handle(result: Result): String = when (result) {
    is Success -> "Data: ${result.data}"
    is Error -> "Error: ${result.message}"
    Loading -> "Loading..."
    // 不需要 else！编译器保证覆盖所有情况
}
```

**优势**：新增子类时，编译器会提示更新所有 `when` 分支。

---

## 密封类 vs 枚举

| 特性         | 密封类         | 枚举           |
|--------------|----------------|----------------|
| 实例数量     | 多个实例       | 单例           |
| 状态         | 可以有状态     | 单例无状态     |
| 继承         | 可以有子类     | 无子类         |
| 类型检查     | is 检查        | == 检查        |

### 枚举示例

```kotlin
enum class Direction { NORTH, SOUTH, EAST, WEST }

// 所有实例都是单例
Direction.NORTH === Direction.NORTH  // true
```

### 密封类示例

```kotlin
sealed class Result
class Success(val data: String) : Result()
class Error(val message: String) : Result()

// 可以创建多个实例
val s1 = Success("A")
val s2 = Success("B")
s1 === s2  // false（不同实例）
```

---

## 实际应用

### 网络请求状态

```kotlin
sealed class UiState {
    object Loading : UiState()
    data class Success(val data: List<User>) : UiState()
    data class Error(val message: String) : UiState()
}

fun render(state: UiState) {
    when (state) {
        is UiState.Loading -> showProgress()
        is UiState.Success -> showData(state.data)
        is UiState.Error -> showError(state.message)
    }
}
```

### 表达式求值

```kotlin
sealed class Expr {
    data class Const(val number: Double) : Expr()
    data class Sum(val e1: Expr, val e2: Expr) : Expr()
    object NotANumber : Expr()
}

fun eval(expr: Expr): Double = when (expr) {
    is Expr.Const -> expr.number
    is Expr.Sum -> eval(expr.e1) + eval(expr.e2)
    Expr.NotANumber -> Double.NaN
}
```

### 支付方式

```kotlin
sealed class PaymentMethod {
    data class CreditCard(val number: String, val cvv: String) : PaymentMethod()
    data class PayPal(val email: String) : PaymentMethod()
    data class BankTransfer(val account: String) : PaymentMethod()
}

fun processPayment(method: PaymentMethod): Boolean {
    return when (method) {
        is PaymentMethod.CreditCard -> chargeCard(method.number, method.cvv)
        is PaymentMethod.PayPal -> chargePayPal(method.email)
        is PaymentMethod.BankTransfer -> initiateTransfer(method.account)
    }
}
```

---

## 密封接口

Kotlin 1.5+ 支持密封接口：

```kotlin
sealed interface Animal
sealed interface Pet

class Dog : Animal, Pet
class Cat : Pet

// 多重继承密封接口
```

---

## 密封类与数据类结合

```kotlin
sealed class Result {
    data class Success<T>(val data: T) : Result()
    data class Error(val exception: Throwable) : Result()
    object Loading : Result()
}

fun <T> handleResult(result: Result): T? = when (result) {
    is Result.Success<T> -> result.data
    is Result.Error -> {
        logError(result.exception)
        null
    }
    Result.Loading -> null
}
```

---

## 实战案例：PICO 动画控制 UI

### 密封类限制参数类型

在 PICO 动画控制组件中，使用密封类限制 TabRow 的参数类型：

```kotlin
// 文件：animation-0.10.7/app/src/main/java/.../ControlWidgets.kt

/**
 * TabRow 参数的密封类
 *
 * Kotlin 知识点：sealed class
 * - 限制继承范围，所有子类在同一文件中定义
 * - 编译器知道所有可能的类型，when 表达式无需 else
 * - 适合表示有限的类型集合
 */
sealed class TabRowParams {
    // 子类定义...
}

// 使用示例：when 表达式处理不同类型
fun renderTabRow(params: TabRowParams) = when (params) {
    is TabRowParams.Text -> TextTabRow(params.text)
    is TabRowParams.Icon -> IconTabRow(params.icon)
    is TabRowParams.Both -> BothTabRow(params.text, params.icon)
    // 不需要 else！编译器保证覆盖所有情况
}
```

**为什么用 sealed class？**
- TabRow 的参数类型是有限的（文本、图标、或两者）
- 编译器强制处理所有情况，避免遗漏
- 新增类型时编译器会提示更新所有 when 分支

### 密封类 vs 枚举的选择

| 场景 | 推荐 | PICO 示例 |
|------|------|----------|
| 固定常量值 | 枚举 | `PlaybackState.PLAYING` |
| 不同状态带数据 | 密封类 | `TabRowParams.Text("播放")` |
| 类型检查 | 枚举用 == | `state == PlaybackState.PLAYING` |
| 模式匹配 | 密封类用 is | `when (params) { is Text -> ... }` |

---

## 练习

### 1. UI 状态管理

用密封类表示列表状态：Loading、Empty、Success(data)、Error(message)。

### 2. 表达式求值

实现加减乘除的表达式求值器。

### 3. 表单验证

用密封类表示表单字段验证结果：Valid、Invalid(reason)。

---

## 下一步

- [泛型](generics.md) - 类型参数化
- [集合概述](collections-overview.md) - Kotlin 集合框架
