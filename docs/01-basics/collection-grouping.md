# 集合排序与分组

> Kotlin 提供丰富的排序、分组和聚合操作。

## 排序

### sorted / sortedDescending

```kotlin
val numbers = listOf(3, 1, 4, 1, 5, 9)

numbers.sorted()              // [1, 1, 3, 4, 5, 9]
numbers.sortedDescending()    // [9, 5, 4, 3, 1, 1]
```

### sortedBy / sortedByDescending

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(
    Person("Charlie", 30),
    Person("Alice", 25),
    Person("Bob", 35)
)

people.sortedBy { it.age }         // 按年龄升序
people.sortedByDescending { it.age }  // 按年龄降序
people.sortedBy { it.name }        // 按名字排序
```

### sortedWith（自定义比较器）

```kotlin
people.sortedWith(compareBy({ it.age }, { it.name }))
// 先按年龄排序，年龄相同按名字排序

people.sortedWith(compareByDescending<Person> { it.age }.thenBy { it.name })
```

### shuffle / reversed

```kotlin
val list = listOf(1, 2, 3, 4, 5)

list.shuffled()   // 随机打乱
list.reversed()   // 反转 [5, 4, 3, 2, 1]
```

### 原地排序（可变集合）

```kotlin
val mutable = mutableListOf(3, 1, 4, 1, 5)

mutable.sort()        // 原地排序
mutable.sortDescending()
mutable.sortBy { it }
mutable.shuffle()
mutable.reverse()
```

---

## 分组 groupBy

### 基本分组

```kotlin
data class Person(val name: String, val city: String)

val people = listOf(
    Person("Alice", "北京"),
    Person("Bob", "上海"),
    Person("Charlie", "北京"),
    Person("David", "上海")
)

val byCity = people.groupBy { it.city }
// {
//   "北京" = [Person(Alice, 北京), Person(Charlie, 北京)],
//   "上海" = [Person(Bob, 上海), Person(David, 上海)]
// }
```

### 分组计数

```kotlin
val words = listOf("apple", "banana", "apricot", "blueberry")

val byFirstLetter = words.groupingBy { it.first() }.eachCount()
// {a=2, b=2}
```

### 分组并转换

```kotlin
val byCityNames = people.groupBy(
    keySelector = { it.city },
    valueTransform = { it.name }
)
// {"北京"=["Alice", "Charlie"], "上海"=["Bob", "David"]}
```

---

## 关联 associate

### associate

转换为 Map：

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(
    Person("Alice", 25),
    Person("Bob", 30)
)

val nameToAge = people.associate { it.name to it.age }
// {Alice=25, Bob=30}
```

### associateBy

指定 key：

```kotlin
val byName = people.associateBy { it.name }
// {Alice=Person(Alice, 25), Bob=Person(Bob, 30)}

// 自定义 value
val nameToUpper = people.associateBy(
    keySelector = { it.name },
    valueTransform = { it.name.uppercase() }
)
// {Alice=ALICE, Bob=BOB}
```

### associateWith

指定 value（key 是元素本身）：

```kotlin
val names = listOf("Alice", "Bob")
val nameLengths = names.associateWith { it.length }
// {Alice=5, Bob=3}
```

---

## 聚合

### 基本聚合

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

numbers.sum()           // 15
numbers.sumOf { it * 2 }  // 30
numbers.average()       // 3.0
numbers.max()           // 5
numbers.min()           // 1
numbers.count()         // 5
```

### reduce

累积计算：

```kotlin
numbers.reduce { acc, n -> acc + n }  // 15
numbers.reduce { acc, n -> acc * n }  // 120
```

### fold

带初始值：

```kotlin
numbers.fold(0) { acc, n -> acc + n }    // 15
numbers.fold(100) { acc, n -> acc + n }  // 115
numbers.fold(1) { acc, n -> acc * n }    // 120
```

### 分组聚合

```kotlin
data class Person(val name: String, val city: String, val salary: Int)

val people = listOf(
    Person("Alice", "北京", 10000),
    Person("Bob", "上海", 15000),
    Person("Charlie", "北京", 12000)
)

// 按城市统计薪资总和
val citySalary = people.groupBy { it.city }
    .mapValues { (_, list) -> list.sumOf { it.salary } }
// {北京=22000, 上海=15000}
```

---

## 统计

```kotlin
val numbers = listOf(1, 2, 2, 3, 3, 3)

// 元素出现次数
numbers.groupingBy { it }.eachCount()
// {1=1, 2=2, 3=3}

// 或使用 count
numbers.count { it == 3 }  // 3
```

---

## 练习

### 1. 排序练习

对学生列表按成绩降序排序，成绩相同按名字排序。

### 2. 分组统计

给定单词列表，按首字母分组并统计每组数量。

### 3. 复杂聚合

计算每个部门的平均薪资和最高薪资。

---

## 下一步

- [函数](functions.md) - 函数进阶
- [协程基础](../02-coroutines/01-coroutines-basics.md) - 异步编程
