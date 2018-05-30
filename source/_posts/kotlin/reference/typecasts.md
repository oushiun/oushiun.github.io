---
title: 类型的检查与转换“is”与“as”

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 其他
date: 2018-05-22 09:40:43
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

### `is` 与 `!is` 操作符

我们可以在运行时通过使用 `is` 操作符或其否定形式 `!is` 来检查对象是否符合给定类型：

```kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // 与 !(obj is String) 相同
    print("Not a String")
}
else {
    print(obj.length)
}
```

<!-- more -->

### 智能转换

在许多情况下，不需要在 Kotlin 中使用显式转换操作符，因为编译器跟踪不可变值的 `is`-检查以及[显式转换](#“不安全的”转换操作符)，并在需要时自动插入（安全的）转换：

```kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x 自动转换为字符串
    }
}
```

编译器足够聪明，能够知道如果反向检查导致返回那么该转换是安全的：

```kotlin
    if (x !is String) return
    print(x.length) // x 自动转换为字符串
```

或者在 `&&` 和 `||` 的右侧：

```kotlin
    // `||` 右侧的 x 自动转换为字符串
    if (x !is String || x.length == 0) return

    // `&&` 右侧的 x 自动转换为字符串
    if (x is String && x.length > 0) {
        print(x.length) // x 自动转换为字符串
    }
```

这些 _智能转换_ 用于 [_when_-表达式](control-flow.html#When-表达式) 和 [_while_-循环 ](control-flow.html#While-循环) 也一样：

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

请注意，当编译器不能保证变量在检查和使用之间不可改变时，智能转换不能用。更具体地，智能转换能否适用根据以下规则：

*   _val_ 局部变量——总是可以，[局部委托属性除外](delegated-properties.html#局部委托属性（自-1-1-起）)；
*   _val_ 属性——如果属性是 private 或 internal，或者该检查在声明属性的同一模块中执行。智能转换不适用于 open 的属性或者具有自定义 getter 的属性；
*   _var_ 局部变量——如果变量在检查和使用之间没有修改、没有在会修改它的 lambda 中捕获、并且不是局部委托属性；
*   _var_ 属性——决不可能（因为该变量可以随时被其他代码修改）。

### “不安全的”转换操作符

通常，如果转换是不可能的，转换操作符会抛出一个异常。因此，我们称之为*不安全的*。
Kotlin 中的不安全转换由中缀操作符 _as_（参见[operator precedence](http://kotlinlang.org/docs/reference/grammar.html#precedence)）完成：

```kotlin
val x: String = y as String
```

请注意，_null_ 不能转换为 `String` 因该类型不是[可空的](null-safety.html)，即如果 `y` 为空，上面的代码会抛出一个异常。为了匹配 Java 转换语义，我们必须在转换右边有可空类型，就像：

```kotlin
val x: String? = y as String?
```

### “安全的”（可空）转换操作符

为了避免抛出异常，可以使用*安全*转换操作符 _as?_，它可以在失败时返回 _null_：

```kotlin
val x: String? = y as? String
```

请注意，尽管事实上 _as?_ 的右边是一个非空类型的 `String`，但是其转换的结果是可空的。

### 类型擦除与泛型检测

Kotlin 在编译时确保涉及[泛型](generics.html)操作的类型安全性，而在运行时，泛型类型的实例并无未带有关于它们实际类型参数的信息。例如， `List<Foo>` 会被擦除为 `List<*>`。通常，在运行时无法检测一个实例是否属于带有某个类型参数的泛型类型。

为此，编译器会禁止由于类型擦除而无法执行的 _is_ 检测，例如 `ints is List<Int>` 或者 `list is T`（类型参数）。当然，你可以对一个实例检测[星投影的类型](generics.html#星投影)：

```kotlin
if (something is List<*>) {
    something.forEach { println(it) } // 这些项的类型都是 `Any?`
}
```

类似地，当已经让一个实例的类型参数（在编译期）静态检测，就可以对涉及非泛型部分做 _is_ 检测或者类型转换。请注意，在这种情况下，会省略尖括号：

```kotlin
fun handleStrings(list: List<String>) {
    if (list is ArrayList) {
        // `list` 会智能转换为 `ArrayList<String>`
    }
}
```

省略类型参数的这种语法可用于不考虑类型参数的类型转换：`list as ArrayList`。

带有[具体化的类型参数](inline-functions.html#具体化的类型参数)的内联函数使其类型实参在每个调用处内联，这就能够对类型参数进行 `arg is T` 检测，但是如果 `arg` 自身是一个泛型实例，**其**类型参数还是会被擦除。例如：

```kotlin
//sampleStart
inline fun <reified A, reified B> Pair<*, *>.asPairOf(): Pair<A, B>? {
    if (first !is A || second !is B) return null
    return first as A to second as B
}

val somePair: Pair<Any?, Any?> = "items" to listOf(1, 2, 3)

val stringToSomething = somePair.asPairOf<String, Any>()
val stringToInt = somePair.asPairOf<String, Int>()
val stringToList = somePair.asPairOf<String, List<*>>()
val stringToStringList = somePair.asPairOf<String, List<String>>() // 破坏类型安全！
//sampleEnd

fun main(args: Array<String>) {
    println("stringToSomething = " + stringToSomething)
    println("stringToInt = " + stringToInt)
    println("stringToList = " + stringToList)
    println("stringToStringList = " + stringToStringList)
}
```

### 非受检类型转换

如上所述，类型擦除使运行时不可能对泛型类型实例的类型实参进行检测，并且代码中的泛型可能相互连接不够紧密，以致于编译器无法确保类型安全。

即便如此，有时候我们有高级的程序逻辑来暗示类型安全。例如：

```kotlin
fun readDictionary(file: File): Map<String, *> = file.inputStream().use {
    TODO("Read a mapping of strings to arbitrary elements.")
}

// 我们已将存有一些 `Int` 的映射保存到该文件
val intsFile = File("ints.dictionary")

// Warning: Unchecked cast: `Map<String, *>` to `Map<String, Int>`
val intsDictionary: Map<String, Int> = readDictionary(intsFile) as Map<String, Int>
```

编译器会对最后一行的类型转换产生一个警告。该类型转换不能在运行时完全检测，并且不能保证映射中的值是“Int”。

为避免未受检类型转换，可以重新设计程序结构：在上例中，可以使用具有类型安全实现的不同接口 `DictionaryReader<T>` 与 `DictionaryWriter<T>`。可以引入合理的抽象，将未受检的类型转换从调用代码移动到实现细节中。正确使用[泛型型变](generics.html#型变)也有帮助。

对于泛型函数，使用[具体化的类型参数](inline-functions.html#具体化的类型参数)可以使诸如 `arg as T` 这样的类型转换受检，除非 `arg` 对应类型的*自身*类型参数已被擦除。

可以通过在产生警告的语句或声明上用注解 `@Suppress("UNCHECKED_CAST")` [标注](annotations.html#注解)来禁止未受检类型转换警告：

```kotlin
inline fun <reified T> List<*>.asListOfType(): List<T>? =
    if (all { it is T })
        @Suppress("UNCHECKED_CAST")
        this as List<T> else
        null
```

在 JVM 平台中，[数组类型](basic-types.html#数组)（`Array<Foo>`）会保留关于其元素被擦除类型的信息，并且类型转换为一个数组类型可以部分受检：元素类型的可空性与类型实参仍然会被擦除。例如，如果 `foo` 是一个保存了任何 `List<*>`（无论可不可空）的数组的话，类型转换 `foo as Array<List<String>?>` 都会成功。
