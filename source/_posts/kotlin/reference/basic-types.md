---
title: 基本类型

toc: true
tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 基础
date: 2018-05-15 17:01:18
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

在 Kotlin 中，所有东西都是对象，在这个意义上讲我们可以在任何变量上调用成员函数和属性。
一些类型可以有特殊的内部表示——例如，数字、字符和布尔值可以在运行时表示为原生类型值，但是对于用户来说，它们看起来就像普通的类。

<!-- more -->

## 数字

Kotlin 处理数字在某种程度上接近 Java，但是并不完全相同。例如，对于数字没有隐式拓宽转换（如 Java 中 `int` 可以隐式转换为`long`——译者注)，另外有些情况的字面值略有不同。

Kotlin 提供了如下的内置类型来表示数字（与 Java 很相近）：

| Type     | Bit width  |
| -------- | ---------- |
| Double   | 64         |
| Float    | 32         |
| Long     | 64         |
| Int      | 32         |
| Short    | 16         |
| Byte     | 8          |

注意在 Kotlin 中字符不是数字

### 字面常量

数值常量字面值有以下几种:

* 十进制: `123`
  * Long 类型用大写 `L` 标记: `123L`
* 十六进制: `0x0F`
* 二进制: `0b00001011`

注意: 不支持八进制

Kotlin 同样支持浮点数的常规表示方法:

* 默认 double：`123.5`、`123.5e10`
* Float 用 `f` 或者 `F` 标记: `123.5f`
 
### 数字字面值中的下划线（自 1.1 起）
 
你可以使用下划线使数字常量更易读：

``` kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 表示方式

在 Java 平台数字是物理存储为 JVM 的原生类型，除非我们需要一个可空的引用（如 `Int?`）或泛型。
后者情况下会把数字装箱。

注意数字装箱不必保留同一性:

``` kotlin
val a: Int = 10000
print(a === a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // ！！！输出“false”！！！
```

另一方面，它保留了相等性:

``` kotlin
val a: Int = 10000
print(a == a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // 输出“true”
```

### 显式转换

由于不同的表示方式，较小类型并不是较大类型的子类型。
如果它们是的话，就会出现下述问题：

``` kotlin
// 假想的代码，实际上并不能编译：
val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)
val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)
print(a == b) // 惊！这将输出“false”鉴于 Int 的 equals() 会检测另一个是否也为 Long
```

所以相等性会在所有地方悄无声息地失去，更别说同一性了。

因此较小的类型**不能**隐式转换为较大的类型。
这意味着在不进行显式转换的情况下我们不能把 `Byte` 型值赋给一个 `Int` 变量。

``` kotlin
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误
```

我们可以显式转换来拓宽数字

``` kotlin
val i: Int = b.toInt() // OK: 显式拓宽
```

每个数字类型支持如下的转换:

* `toByte(): Byte`
* `toShort(): Short`
* `toInt(): Int`
* `toLong(): Long`
* `toFloat(): Float`
* `toDouble(): Double`
* `toChar(): Char`

缺乏隐式类型转换并不显著，因为类型会从上下文推断出来，而算术运算会有重载做适当转换，例如：

``` kotlin
val l = 1L + 3 // Long + Int => Long
```

### 运算

Kotlin支持数字运算的标准集，运算被定义为相应的类成员（但编译器会将函数调用优化为相应的指令）。
参见[运算符重载](operator-overloading.html)。

对于位运算，没有特殊字符来表示，而只可用中缀方式调用命名函数，例如:

``` kotlin
val x = (1 shl 2) and 0x000FF000
```

这是完整的位运算列表（只用于 `Int` 和 `Long`）：

* `shl(bits)` – 有符号左移 (Java 的 `<<`)
* `shr(bits)` – 有符号右移 (Java 的 `>>`)
* `ushr(bits)` – 无符号右移 (Java 的 `>>>`)
* `and(bits)` – 位与
* `or(bits)` – 位或
* `xor(bits)` – 位异或
* `inv()` – 位非

### 浮点数比较

本节讨论的浮点数操作如下：

* 相等性检测：`a == b` 与 `a != b`
* 比较操作符：`a < b`、 `a > b`、 `a <= b`、 `a >= b`
* 区间实例以及区间检测：`a..b`、 `x in a..b`、 `x !in a..b`

当其中的操作数 `a` 与 `b` 都是静态已知的 `Float` 或 `Double` 或者它们对应的可空类型（声明为该类型，或者推断为该类型，或者[智能类型转换](typecasts.html#智能转换)的结果是该类型），两数字所形成的操作或者区间遵循 IEEE 754 浮点运算标准。

然而，为了支持泛型场景并提供全序支持，当这些操作符**并非**静态类型为<!-- 
-->浮点数（例如是 `Any`、 `Comparable<……>`、 类型参数）时，这些操作使用为 `Float` 与 `Double` 实现的不符合标准的 `equals` 与 `compareTo`，这会出现：

* 认为 `NaN` 与其自身相等
* 认为 `NaN` 比包括正无穷大（`POSITIVE_INFINITY`）在内的任何其他元素都大
* 认为 `-0.0` 小于 `0.0`

## 字符

字符用 `Char` 类型表示。它们不能直接当作数字

``` kotlin
fun check(c: Char) {
    if (c == 1) { // 错误：类型不兼容
        // ……
    }
}
```

字符字面值用单引号括起来: `'1'`。
特殊字符可以用反斜杠转义。
支持这几个转义序列：`\t`、 `\b`、`\n`、`\r`、`\'`、`\"`、`\\` 和 `\$`。
编码其他字符要用 Unicode 转义序列语法：`'\uFF00'`。

我们可以显式把字符转换为 `Int` 数字：

``` kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // 显式转换为数字
}
```

当需要可空引用时，像数字、字符会被装箱。装箱操作不会保留同一性。

## 布尔

布尔用 `Boolean` 类型表示，它有两个值：`true` 和 `false`。

若需要可空引用布尔会被装箱。

内置的布尔运算有：

* `||` – 短路逻辑或
* `&&` – 短路逻辑与
* `!` - 逻辑非

## 数组

数组在 Kotlin 中使用 `Array` 类来表示，它定义了 `get` 和 `set` 函数（按照运算符重载约定这会转变为 `[]`）和 `size` 属性，以及一些其他有用的成员函数：

``` kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ……
}
```

我们可以使用库函数 `arrayOf()` 来创建一个数组并传递元素值给它，这样 `arrayOf(1, 2, 3)` 创建了 array [1, 2, 3]。
或者，库函数 `arrayOfNulls()` 可以用于创建一个指定大小的、所有元素都为空的数组。

另一个选项是用接受数组大小和一个函数参数的 `Array` 构造函数，用作参数的函数能够返回给定索引的每个元素初始值：

``` kotlin
// 创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

如上所述，`[]` 运算符代表调用成员函数 `get()` 和 `set()`。

注意: 与 Java 不同的是，Kotlin 中数组是不型变的（invariant）。这意味着 Kotlin 不让我们把 `Array<String>` 赋值给 `Array<Any>`，以防止可能的运行时失败（但是你可以使用 `Array<out Any>`,
参见[类型投影](generics.html#类型投影)）。

Kotlin 也有无装箱开销的专门的类来表示原生类型数组: `ByteArray`、`ShortArray`、`IntArray` 等等。这些类和 `Array` 并没有继承关系，但是它们有同样的方法属性集。它们也都有相应的工厂方法:

``` kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

## 字符串

字符串用 `String` 类型表示。字符串是不可变的。
字符串的元素——字符可以使用索引运算符访问: `s[i]`。
可以用 `for` 循环迭代字符串:

``` kotlin
fun main(args: Array<String>) {
val str = "abcd"
//sampleStart
for (c in str) {
    println(c)
}
//sampleEnd
}
```

可以用 `+` 操作符连接字符串。这也适用于连接字符串与其他类型的值，只要表达式中的第一个元素是字符串：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
val s = "abc" + 1
println(s + "def")
//sampleEnd
}
```

请注意，在大多数情况下，优先使用[字符串模板](字符串模板)或原始字符串而不是字符串连接。

### 字符串字面值

Kotlin 有两种类型的字符串字面值: 转义字符串可以有转义字符，以及原始字符串可以包含换行和任意文本。转义字符串很像 Java 字符串:

``` kotlin
val s = "Hello, world!\n"
```

转义采用传统的反斜杠方式。参见上面的 [字符](#字符) 查看支持的转义序列。

*原始字符串* 使用三个引号（`"""`）分界符括起来，内部没有转义并且可以包含换行和任何其他字符:

``` kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

你可以通过 [`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html) 函数去除前导空格：

``` kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

默认 `|` 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 `trimMargin(">")`。

### 字符串模板

字符串可以包含*模板表达式* ，即一些小段代码，会求值并把结果合并到字符串中。
模板表达式以美元符（`$`）开头，由一个简单的名字构成:

``` kotlin
fun main(args: Array<String>) {
//sampleStart
val i = 10
println("i = $i") // 输出“i = 10”
//sampleEnd
}
```

或者用花括号括起来的任意表达式:

``` kotlin
fun main(args: Array<String>) {
//sampleStart
val s = "abc"
println("$s.length is ${s.length}") // 输出“abc.length is 3”
//sampleEnd
}
```

原始字符串和转义字符串内部都支持模板。
如果你需要在原始字符串中表示字面值 `$` 字符（它不支持反斜杠转义），你可以用下列语法：

``` kotlin
val price = """
${'$'}9.99
"""
```