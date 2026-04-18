# 集合过滤与映射

> filter、map、flatMap 是 Kotlin 集合操作的核心函数。

## 过滤 filter

### 基本过滤

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6)

// 过滤满足条件的元素
numbers.filter { it > 3 }       // [4, 5, 6]
numbers.filter { it % 2 == 0 }  // [2, 4, 6]
```

### filterNot

```kotlin
numbers.filterNot { it > 3 }    // [1, 2, 3]
```

### filterNotNull

```kotlin
val list = listOf(1, null, 2, null, 3)
list.filterNotNull()            // [1, 2, 3]（返回非空 List<Int>）
```

### filterIsInstance

```kotlin
val list: List<Any> = listOf(1, "a", 2, "b")
list.filterIsInstance<String>()  // ["a", "b"]
list.filterIsInstance<Int>()     // [1, 2]
```

### 分区 partition

将集合分成两部分：

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6)
val (evens, odds) = numbers.partition { it % 2 == 0 }
// evens = [2, 4, 6]
// odds = [1, 3, 5]
```

### take / drop

```kotlin
val list = listOf(1, 2, 3, 4, 5)

list.take(3)         // [1, 2, 3]（取前 n 个）
list.takeLast(3)     // [3, 4, 5]
list.drop(2)         // [3, 4, 5]（丢弃前 n 个）
list.dropLast(2)     // [1, 2, 3]

// 条件版本
list.takeWhile { it < 4 }   // [1, 2, 3]
list.dropWhile { it < 3 }   // [3, 4, 5]
```

---

## 映射 map

### 基本映射

```kotlin
val numbers = listOf(1, 2, 3)

numbers.map { it * 2 }           // [2, 4, 6]
numbers.map { "Num: $it" }       // ["Num: 1", "Num: 2", "Num: 3"]
```

### mapNotNull

```kotlin
val list = listOf(1, 2, 3, null)
list.mapNotNull { it?.let { it * 2 } }  // [2, 4, 6]
```

### mapIndexed

```kotlin
val list = listOf("a", "b", "c")
list.mapIndexed { index, value -> "$index: $value" }
// ["0: a", "1: b", "2: c"]
```

### mapKeys / mapValues（Map）

```kotlin
val map = mapOf("a" to 1, "b" to 2)

map.mapKeys { it.key.uppercase() }   // {"A" to 1, "B" to 2}
map.mapValues { it.value * 2 }       // {"a" to 2, "b" to 4}
```

---

## 扁平化 flatMap

### flatMap

先映射再扁平化：

```kotlin
val lists = listOf(
    listOf(1, 2),
    listOf(3, 4),
    listOf(5)
)

lists.flatMap { it }  // [1, 2, 3, 4, 5]

// 实际应用：提取嵌套属性
data class Person(val name: String, val hobbies: List<String>)

val people = listOf(
    Person("Alice", listOf("Reading", "Music")),
    Person("Bob", listOf("Gaming", "Music"))
)

val allHobbies = people.flatMap { it.hobbies }.toSet()
// [Reading, Music, Gaming]
```

### flatten

仅扁平化：

```kotlin
val lists = listOf(listOf(1, 2), listOf(3, 4))
lists.flatten()  // [1, 2, 3, 4]
```

---

## 链式操作

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(
    Person("Alice", 25),
    Person("Bob", 17),
    Person("Charlie", 30)
)

// 找出成年人的名字
val adultNames = people
    .filter { it.age >= 18 }
    .map { it.name }
// [Alice, Charlie]

// 计算成年人平均年龄
val avgAge = people
    .filter { it.age >= 18 }
    .map { it.age }
    .average()
// 27.5
```

---

## 序列 Sequence

对于大量数据或链式操作，使用 Sequence 提高性能：

```kotlin
// List（急求值）
listOf(1, 2, 3, 4, 5)
    .filter { it > 2 }     // 创建中间列表 [3, 4, 5]
    .map { it * 2 }        // 创建中间列表 [6, 8, 10]

// Sequence（惰性求值）
sequenceOf(1, 2, 3, 4, 5)
    .filter { it > 2 }
    .map { it * 2 }
    .toList()              // 只在此时计算

// 或用 asSequence()
listOf(1, 2, 3, 4, 5)
    .asSequence()
    .filter { println("filter: $it"); it > 2 }
    .map { println("map: $it"); it * 2 }
    .take(1)               // 只处理到第一个满足条件的
    .toList()
// 输出：
// filter: 1
// filter: 2
// filter: 3
// map: 3
```

---

## 实战案例：PICO UI 组件集合操作

### forEach - 遍历集合

在 PICO 组件展示项目中，大量使用 `forEach` 渲染 UI 列表：

```kotlin
// 文件：component-playground-0.10.7/.../DialogPage.kt

@Composable
fun MenuDemo() {
    // Kotlin 知识点：forEach 遍历集合
    // 在 Compose 中用于渲染多个同类组件
    listOf("编辑", "分享", "删除", "取消").forEach { item ->
        DropdownMenuItem(
            onClick = { /* 处理点击 */ }
        ) {
            Text(item)
        }
    }
}

// 文件：InteractionPage.kt

@Composable
fun CategoryTabs(categories: List<String>, onCategorySelected: (Int) -> Unit) {
    // forEachIndexed: 获取索引和值
    categories.forEachIndexed { index, category ->
        Chip(
            selected = selectedCategory == index,
            onClick = { onCategorySelected(index) }
        ) {
            Text(category)
        }
    }
}
```

**forEach vs for 循环**：
- `forEach` 更 Kotlin 风格
- `forEachIndexed` 提供索引
- 可以与 Lambda 链式调用

### 链式操作实战

```kotlin
// 文件：physics-0.10.7/.../PhysicsManager.kt

/**
 * 重置所有多米诺骨牌
 */
fun resetAllDominoes(dominoNames: List<String>) {
    dominoNames
        .mapNotNull { name -> rootEntity.findEntity(name) }  // 过滤掉 null
        .filter { it.enabled }  // 只处理启用的实体
        .forEach { domino ->
            domino.components[TransformComponent::class.java]?.let { transform ->
                transform.position = originalPositions[domino.name] ?: Vector3.ZERO
            }
        }
}

// 等价于传统的 for 循环，但更简洁
fun resetAllDominoesOld(dominoNames: List<String>) {
    for (name in dominoNames) {
        val domino = rootEntity.findEntity(name)
        if (domino != null && domino.enabled) {
            val transform = domino.components[TransformComponent::class.java]
            if (transform != null) {
                transform.position = originalPositions[domino.name] ?: Vector3.ZERO
            }
        }
    }
}
```

### 空安全与集合操作结合

```kotlin
// 文件：spatialvideo-0.10.7/.../PlaybackManager.kt

/**
 * 获取视频时长
 *
 * Kotlin 知识点：
 * - takeIf { }: 满足条件返回原值，否则返回 null
 * - ?:: Elvis 运算符，提供默认值
 */
fun getDuration(): Long {
    return player.getDuration().takeIf { it > 0 } ?: 1L
}

// 等价于
fun getDurationOld(): Long {
    val duration = player.getDuration()
    return if (duration > 0) duration else 1L
}
```

### filterNotNull 实战

```kotlin
// 文件：physics-0.10.7/.../PhysicsManager.kt

/**
 * 获取所有有效的物理实体
 */
fun getValidEntities(names: List<String>): List<Entity> {
    return names
        .map { rootEntity.findEntity(it) }  // List<Entity?>
        .filterNotNull()  // List<Entity>（过滤掉 null）
        .filter { it.enabled }
}
```

### 实际应用总结

| 操作符 | 使用场景 | PICO 示例 |
|--------|----------|----------|
| `forEach` | 遍历渲染 UI | 渲染菜单项 |
| `forEachIndexed` | 需要索引 | 标签页索引 |
| `map` | 转换类型 | 实体名称 -> 实体对象 |
| `filter` | 过滤条件 | 只处理启用的实体 |
| `filterNotNull` | 过滤空值 | 有效的实体列表 |
| `mapNotNull` | map + filterNotNull | 一步完成转换和过滤 |
| `takeIf` | 条件判断 | 获取有效时长 |
| `?:` | 提供默认值 | 时长默认 1L |

---

## 练习

### 1. 过滤质数

从 1-100 中过滤出所有质数。

### 2. 字符串处理

将字符串列表转为大写，过滤长度大于 5 的。

### 3. 扁平化练习

给定 `List<List<Int>>`，过滤出所有偶数并去重。

---

## 下一步

- [排序与分组](collection-grouping.md) - sort、groupBy
