# 集合概述

> Kotlin 集合框架基于 Java，但提供了更丰富的操作函数和不可变集合支持。

## 集合类型

| 类型       | 只读         | 可变            | 特点           |
|------------|--------------|-----------------|----------------|
| List       | List<T>      | MutableList<T>  | 有序、可重复   |
| Set        | Set<T>       | MutableSet<T>   | 无序、不重复   |
| Map        | Map<K, V>    | MutableMap<K, V>| 键值对、键唯一 |

---

## 创建集合

### List

```kotlin
// 不可变
val list = listOf(1, 2, 3)
val empty = emptyList<Int>()

// 可变
val mutable = mutableListOf(1, 2, 3)
mutable.add(4)
mutable[0] = 10

// 使用构建器
val list = buildList {
    add(1)
    add(2)
    add(3)
}
```

### Set

```kotlin
// 不可变
val set = setOf(1, 2, 3, 2)  // [1, 2, 3]
val empty = emptySet<Int>()

// 可变
val mutable = mutableSetOf(1, 2, 3)
mutable.add(4)
mutable.add(1)  // 无效，已存在

// 有序 Set（保持插入顺序）
val linked = linkedSetOf(1, 2, 3)
// 排序 Set
val sorted = sortedSetOf(3, 1, 2)  // [1, 2, 3]
```

### Map

```kotlin
// 不可变
val map = mapOf("a" to 1, "b" to 2)
val empty = emptyMap<String, Int>()

// 可变
val mutable = mutableMapOf("a" to 1)
mutable["b"] = 2
mutable.put("c", 3)

// 使用构建器
val map = buildMap {
    put("a", 1)
    put("b", 2)
}
```

---

## 访问元素

### List

```kotlin
val list = listOf("a", "b", "c")

list[0]           // "a"（索引访问）
list.get(0)       // "a"
list.getOrNull(10) // null
list.getOrElse(10) { "default" }  // "default"

list.first()      // "a"
list.last()       // "c"
list.firstOrNull()
list.lastOrNull()
```

### Map

```kotlin
val map = mapOf("a" to 1, "b" to 2)

map["a"]           // 1
map.get("c")       // null
map.getValue("a")  // 1（键不存在抛异常）
map.getOrDefault("c", 0)  // 0
map.getOrElse("c") { 0 }  // 0
```

---

## 遍历

### List

```kotlin
val list = listOf("a", "b", "c")

for (item in list) {
    println(item)
}

for ((index, value) in list.withIndex()) {
    println("$index: $value")
}

list.forEach { println(it) }
list.forEachIndexed { i, v -> println("$i: $v") }
```

### Map

```kotlin
val map = mapOf("a" to 1, "b" to 2)

for (key in map.keys) { }
for (value in map.values) { }
for ((key, value) in map) { }

map.forEach { (k, v) -> println("$k = $v") }
```

---

## 常用操作

### 检查

```kotlin
val list = listOf(1, 2, 3)

list.isEmpty()
list.isNotEmpty()
list.contains(2)      // true
2 in list             // true（运算符形式）
list.containsAll(listOf(1, 2))
```

### 大小

```kotlin
list.size
list.count()
list.count { it > 1 }  // 满足条件的数量
```

### 切片

```kotlin
val list = listOf(0, 1, 2, 3, 4, 5)

list.subList(1, 4)     // [1, 2, 3]
list.slice(1..3)       // [1, 2, 3]
list.slice(listOf(0, 2, 4))  // [0, 2, 4]
```

### 转换

```kotlin
// 转为数组
list.toTypedArray()
list.toIntArray()

// 转为可变集合
list.toMutableList()

// 转为 Set（去重）
list.toSet()
```

---

## 只读 vs 可变

Kotlin 区分只读和可变集合：

```kotlin
// 只读
val list: List<Int> = listOf(1, 2, 3)
// list.add(4)  // ❌ 没有 add 方法

// 可变
val mutable: MutableList<Int> = mutableListOf(1, 2, 3)
mutable.add(4)  // ✅
```

### 向上转型

```kotlin
val mutable: MutableList<Int> = mutableListOf(1, 2, 3)
val readonly: List<Int> = mutable  // ✅ 向上转型

// readonly 实际是可变的，只是没有暴露修改方法
mutable.add(4)
println(readonly)  // [1, 2, 3, 4]
```

---

## 集合与数组

```kotlin
// 数组转集合
val arr = arrayOf(1, 2, 3)
val list = arr.toList()

// 集合转数组
val list = listOf(1, 2, 3)
val arr = list.toTypedArray()
```

---

## 练习

### 1. 去重

给定列表 `[1, 2, 2, 3, 3, 3]`，去重并保持顺序。

### 2. 统计词频

统计字符串中每个字符出现的次数。

### 3. 合并 Map

合并两个 Map，相同 key 的 value 相加。

---

## 下一步

- [过滤与映射](collection-filtering.md) - filter、map
- [排序与分组](collection-grouping.md) - sort、groupBy
