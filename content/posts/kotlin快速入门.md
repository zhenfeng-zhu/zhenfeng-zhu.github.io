---
date: "2018-03-24 17:56:45"
title: kotlin快速入门
---
快速浏览一下 Kotlin 的语法。

# 基本语法

### 包定义和引用

在源文件头部：

```Kotlin
package my.demo
import java.util.*

```

### 方法定义

- 带有方法体，并且返回确定类型数据的定义方式，例如接受 `Int` 类型的参数并返回 `Int` 类型的值：

```Kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}

```

- 带有方法体，返回推断类型数据的定义方式，例如：

```Kotlin
fun sum(a: Int, b: Int) = a + b

```

- 返回无意义类型的定义方式：

```Kotlin
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}

```

或者省略 `Unit`：

```kotlin
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}

```

### 变量定义

- 只赋值一次（只读）本地变量，**val**：

```kotlin
val a:Int = 1    // 指定初始值
val b = 2        // 类型自推断为 `Int`
val c:Int        // 当不指定初始值时需要指定类型
c = 3            // 延迟赋值

```

- 可变变量， **var**：

```Kotlin
var x = 5    // 类型自推断为 `Int`
x += 1

```

- 顶层变量

```kotlin
val PI = 3.14
var x = 0

fun incrementX() {
    x += 1
}

```

### 注释

与 Java 和 JavaScript 一样，Kotlin 支持行尾注释和块注释：

```kotlin
// 行尾注释
/* 多行
   块注释 */

```

与 Java 不同，Kotlin 中的块注释可以嵌套。

### string 模板

```kotlin
var a = 1
val s1 = "a is $a"

a = 2
val s2 = "${s1.replace("is", "was")}, but now is $a}" 

```

### 条件表达式

```Kotlin
fun maxOf(a:Int, b:Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}

```

使用 `if` 做为表达式：

```kotlin
fun maxOf(a:Int, b:Int) = if (a > b) a else b

```

### 可能为 null 的值，检查是否为 null

如果值可能为 `null` 时，必须显示的指出。
例如：

```kotlin
fun parseInt(str: String): Int? {
    // ...
}

```

使用上面定义的方法：

```kotlin
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    if (x != null && y != null) {
        println(x * y)
    } else {
        println(either '$arg1' or '$arg2' is not a number)
}

```

或者：

```kotlin
if (x == null) {
    println("Wrong number format in arg1: '$arg1'")
    return
}
if (y == null) {
    println("Wrong number format in arg2: '$arg2'")
    return
}

println(x * y)

```

### 类型检查和自动转换

`is` 操作符用于检查某个实例是否为某种类型。如果一个不可变本地变量或属性已经做过类型检查，那么可以不必显示的进行类型转换就可以使用对应类型的属性或方法。

```kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        return obj.length    // 在这个类型检查分支中，`obj` 自动转换为 `String`
    }

    return null              // 在类型检查分支外，`obj` 仍然为 `Any`
}

```

或者：

```kotlin
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null
    
    return obj.length    // 在这个分支中，`obj` 自动转换为 `String`
}

```

再或者：

```kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String && obj.length > 0) {    //  在 `&&` 操作符的右侧，`obj` 自动转换为 `String`
        return obj.length
    }

    return null 
}

```

### for 循环

```kotlin
val items = listOf("apple", "banana", "kiwi")
for (item in items) {
    println(item)
}

```

或者：

```kotlin
val items = listOf("apple", "banana", "kiwi")
for (index in items.indices) {
    println("item at $index is ${items[index]}")
}

```

### while 循环

```kotlin
val items = listOf("apple", "banana", "kiwi")
var index = 0
while (index < items.size) {
    println("item at $index is ${items[index]}")
    index ++
}

```

### when 表达式

```kotlin
fun describe(obj: Any): String = 
when (obj) {
    1          -> "one"
    "hello"    -> "Greeting"
    is Long    -> "Long"
    !is String -> "Not a String"
    else       -> "Unknown"
}

```

### 区间

- 使用 `in` 操作符检查数字是否在区间内：

```kotlin
val x = 10
val y = 9
if (x in 1..y+1) {
    println("fits in range")
}

```

- 检查数字是否在范围外：

```kotlin
val list = listOf("a", "b", "c")

if (-1 !in 0..list.lastIndex) {
    println("-1 is out of range")
}
if (list.size !in list.indices) {
    println("list size is out of valid list indices range too")
}

```

- 区间遍历

```kotlin
for (x in 1..5) {
    print(x)
}

```

### 集合

- 遍历集合：

```kotlin
for (item in items) {
    println(item)
}

```

- 使用 `in` 操作符判断集合中是否包含某个对象：

```kotlin
when {
    "orange" in items  -> println("juicy")
    "apple" in items   -> println("apple is fine too")
}

```

- 使用 lambda 表达式过滤和 map 集合：

```kotlin
val fruits = listOf("banana", "avocado", "apple", "kiwi")
fruits
.filter {it.startWith("a")}
.sortedBy {it}
.map {it.upperCase()}
.forEach {println(it)}

```

### 创建基本类和实例

```kotlin
fun main(args: Array<String>) {
    val rectangle = Rectangle(5.0, 2.0) // 不需要使用 'new' 关键词
    val triangle = Triangle(3.0, 4.0, 5.0)
    println("Area of rectangle is ${rectangle.calculateArea()}, its perimeter is ${rectangle.perimeter}")
    println("Area of triangle is ${triangle.calculateArea()}, its perimeter is ${triangle.perimeter}")
}

abstract class Shape(val sides: List<Double>) {
    val perimeter: Double get() = sides.sum()
    abstract fun calculateArea(): Double
}

interface RectangleProperties {
    val isSquare: Boolean
}

class Rectangle(
    var height: Double,
    var length: Double
) : Shape(listOf(height, length, height, length)), RectangleProperties {
    override val isSquare: Boolean get() = height == length
    override fun calculateArea(): Double = height * length
}

class Triangle(
    var sideA: Double,
    var sideB: Double,
    var sideC: Double
) : Shape(listOf(sideA, sideB, sideC)) {
    override fyb calculateArre(): Double {
        val s = perimeter / 2
        return Math.sqrt(s * (s - sideA) * (s - sideB) * (s - sideC))
    }
}

```

------

以上引自：

> [http://kotlinlang.org/docs/reference/basic-syntax.html](https://link.jianshu.com?t=http://kotlinlang.org/docs/reference/basic-syntax.html)