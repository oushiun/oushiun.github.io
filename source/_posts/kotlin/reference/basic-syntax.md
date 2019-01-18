---
title: 基本语法

toc: true
tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - Getting Started
date: 2018-05-15 12:29:00
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

#### 定义包

包的声明应处于源文件顶部：

``` kotlin
package my.demo

import java.util.*

// ……
```

目录与包的结构无需匹配：源代码可以在文件系统的任意位置。

<!-- more -->

参见[包](packages.html)。

#### 定义函数

带有两个 `Int` 参数、返回 `Int` 的函数：

``` kotlin
//sampleStart
fun sum(a: Int, b: Int): Int {
    return a + b
}
//sampleEnd

fun main(args: Array<String>) {
    print("sum of 3 and 5 is ")
    println(sum(3, 5))
}
```

将表达式作为函数体、返回值类型自动推断的函数：

``` kotlin
//sampleStart
fun sum(a: Int, b: Int) = a + b
//sampleEnd

fun main(args: Array<String>) {
    println("sum of 19 and 23 is ${sum(19, 23)}")
}
```

函数返回无意义的值：

``` kotlin
//sampleStart
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main(args: Array<String>) {
    printSum(-1, 8)
}
```

`Unit` 返回类型可以省略：

``` kotlin
//sampleStart
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main(args: Array<String>) {
    printSum(-1, 8)
}
```

参见[函数](functions.html)。

#### 定义变量

一次赋值（只读）的局部变量:

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val a: Int = 1  // 立即赋值
    val b = 2   // 自动推断出 `Int` 类型
    val c: Int  // 如果没有初始值类型不能省略
    c = 3       // 明确赋值
//sampleEnd
    println("a = $a, b = $b, c = $c")
}
```

可变变量：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    var x = 5 // 自动推断出 `Int` 类型
    x += 1
//sampleEnd
    println("x = $x")
}
```

顶层变量：

``` kotlin
//sampleStart
val PI = 3.14
var x = 0

fun incrementX() {
    x += 1
}
//sampleEnd

fun main(args: Array<String>) {
    println("x = $x; PI = $PI")
    incrementX()
    println("incrementX()")
    println("x = $x; PI = $PI")
}
```

参见[属性和字段](properties.html)。

#### 注释

正如 Java 和 JavaScript，Kotlin 支持行注释及块注释。

``` kotlin
// 这是一个行注释

/* 这是一个多行的
   块注释。 */
```

与 Java 不同的是，Kotlin 的块注释可以嵌套。

参见[编写 Kotlin 代码文档](kotlin-doc.html) 查看关于文档注释语法的信息。

#### 使用字符串模板

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    var a = 1
    // 模板中的简单名称：
    val s1 = "a is $a"

    a = 2
    // 模板中的任意表达式：
    val s2 = "${s1.replace("is", "was")}, but now is $a"
//sampleEnd
    println(s2)
}
```

参见[字符串模板](basic-types.html#字符串模板)。

#### 使用条件表达式

``` kotlin
//sampleStart
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
//sampleEnd

fun main(args: Array<String>) {
    println("max of 0 and 42 is ${maxOf(0, 42)}")
}
```

使用 `if` 作为表达式:

``` kotlin
//sampleStart
fun maxOf(a: Int, b: Int) = if (a > b) a else b
//sampleEnd

fun main(args: Array<String>) {
    println("max of 0 and 42 is ${maxOf(0, 42)}")
}
```

参见 [*if* 表达式](control-flow.html#If-表达式)。

#### 使用可空值及 `null` 检测

当某个变量的值可以为 `null` 的时候，必须在声明处的类型后添加 `?` 来标识该引用可为空。

如果 `str` 的内容不是数字返回 `null`：

``` kotlin
fun parseInt(str: String): Int? {
    // ……
}
```

使用返回可空值的函数:

``` kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

//sampleStart
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // 直接使用 `x * y` 会导致编译错误，因为他们可能为 null
    if (x != null && y != null) {
        // 在空检测后，x 和 y 会自动转换为非空值（non-nullable）
        println(x * y)
    }
    else {
        println("either '$arg1' or '$arg2' is not a number")
    }
}
//sampleEnd


fun main(args: Array<String>) {
    printProduct("6", "7")
    printProduct("a", "7")
    printProduct("a", "b")
}
```

或者

``` kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

//sampleStart
    // ……
    if (x == null) {
        println("Wrong number format in arg1: '$arg1'")
        return
    }
    if (y == null) {
        println("Wrong number format in arg2: '$arg2'")
        return
    }

    // 在空检测后，x 和 y 会自动转换为非空值
    println(x * y)
//sampleEnd
}

fun main(args: Array<String>) {
    printProduct("6", "7")
    printProduct("a", "7")
    printProduct("99", "b")
}
```

参见[空安全](null-safety.html)。

#### 使用类型检测及自动类型转换

`is` 运算符检测一个表达式是否某类型的一个实例。如果一个不可变的局部变量或属性已经判断出为某类型，那么检测后的分支中可以直接当作该类型使用，无需显式转换：

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` 在该条件分支内自动转换成 `String`
        return obj.length
    }

    // 在离开类型检测分支后，`obj` 仍然是 `Any` 类型
    return null
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")
    printLength(1000)
    printLength(listOf(Any()))
}
```

或者

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` 在这一分支自动转换为 `String`
    return obj.length
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")
    printLength(1000)
    printLength(listOf(Any()))
}
```

甚至

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    // `obj` 在 `&&` 右边自动转换成 `String` 类型
    if (obj is String && obj.length > 0) {
      return obj.length
    }

    return null
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, is empty or not a string at all"} ")
    }
    printLength("Incomprehensibilities")
    printLength("")
    printLength(1000)
}
```

参见[类](classes.html) 和 [类型转换](typecasts.html)。

#### 使用 `for` 循环

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    for (item in items) {
        println(item)
    }
//sampleEnd
}
```

或者

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    for (index in items.indices) {
        println("item at $index is ${items[index]}")
    }
//sampleEnd
}
```

参见 [for 循环](control-flow.html#For-循环)。

#### 使用 `while` 循环

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index++
    }
//sampleEnd
}
```

参见 [while 循环](control-flow.html#While-循环)。

#### 使用 `when` 表达式

``` kotlin
//sampleStart
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long"
        !is String -> "Not a string"
        else       -> "Unknown"
    }
//sampleEnd

fun main(args: Array<String>) {
    println(describe(1))
    println(describe("Hello"))
    println(describe(1000L))
    println(describe(2))
    println(describe("other"))
}
```

参见 [when 表达式](control-flow.html#When-表达式)。

#### 使用区间（range）

使用 `in` 运算符来检测某个数字是否在指定区间内：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val x = 10
    val y = 9
    if (x in 1..y+1) {
        println("fits in range")
    }
//sampleEnd
}
```

检测某个数字是否在指定区间外:

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val list = listOf("a", "b", "c")

    if (-1 !in 0..list.lastIndex) {
        println("-1 is out of range")
    }
    if (list.size !in list.indices) {
        println("list size is out of valid list indices range too")
    }
//sampleEnd
}
```

区间迭代:

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    for (x in 1..5) {
        print(x)
    }
//sampleEnd
}
```

或数列迭代：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    for (x in 1..10 step 2) {
        print(x)
    }
    println()
    for (x in 9 downTo 0 step 3) {
        print(x)
    }
//sampleEnd
}
```

参见[区间](ranges.html)。

#### 使用集合

对集合进行迭代:

``` kotlin
fun main(args: Array<String>) {
    val items = listOf("apple", "banana", "kiwifruit")
//sampleStart
    for (item in items) {
        println(item)
    }
//sampleEnd
}
```

使用 `in` 运算符来判断集合内是否包含某实例：

``` kotlin
fun main(args: Array<String>) {
    val items = setOf("apple", "banana", "kiwifruit")
//sampleStart
    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
//sampleEnd
}
```

使用 lambda 表达式来过滤（filter）和映射（map）集合：

``` kotlin
fun main(args: Array<String>) {
    val fruits = listOf("banana", "avocado", "apple", "kiwifruit")
//sampleStart
    fruits
        .filter { it.startsWith("a") }
        .sortedBy { it }
        .map { it.toUpperCase() }
        .forEach { println(it) }
//sampleEnd
}
```

参见[高阶函数及 Lambda 表达式](lambdas.html)。

#### 创建基本类及其实例：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val rectangle = Rectangle(5.0, 2.0) // 不需要“new”关键字
    val triangle = Triangle(3.0, 4.0, 5.0)
//sampleEnd
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
    override val isSquare: Boolean get() = length == height
    override fun calculateArea(): Double = height * length
}

class Triangle(
    var sideA: Double,
    var sideB: Double,
    var sideC: Double
) : Shape(listOf(sideA, sideB, sideC)) {
    override fun calculateArea(): Double {
        val s = perimeter / 2
        return Math.sqrt(s * (s - sideA) * (s - sideB) * (s - sideC))
    }
}
```

参见[类](classes.html)以及[对象与实例](object-declarations.html)。
