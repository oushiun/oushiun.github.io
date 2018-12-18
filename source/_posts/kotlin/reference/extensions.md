---
title: 扩展

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 类与对象
date: 2018-05-17 10:24:03
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

Kotlin 同 C# 和 Gosu 类似，能够扩展一个类的新功能而无需继承该类或使用像装饰者这样的任何类型的设计模式。这通过叫做 *扩展* 的特殊声明完成。Kotlin 支持 _扩展函数_ 和 _扩展属性_。

<!-- more -->

### 扩展函数

声明一个扩展函数，我们需要用一个 _接收者类型_ 也就是被扩展的类型来作为他的前缀。下面代码为 `MutableList<Int>` 添加一个`swap` 函数：

``` kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```

这个 *this* 关键字在扩展函数内部对应到接收者对象（传过来的在点符号前的对象）现在，我们对任意 `MutableList<Int>` 调用该函数了：

``` kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // “swap()”内部的“this”得到“l”的值
```

当然，这个函数对任何 `MutableList<T>` 起作用，我们可以泛化它：

``` kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```

为了在接收者类型表达式中使用泛型，我们要在函数名前声明泛型参数。参见[泛型函数](generics.html)。

### 扩展是静态解析的

扩展不能真正的修改他们所扩展的类。通过定义一个扩展，你并没有在一个类中插入新成员，仅仅是可以通过该类型的变量用点表达式去调用这个新函数。

我们想强调的是扩展函数是静态分发的，即他们不是根据接收者类型的虚方法。这意味着调用的扩展函数是由函数调用所在的表达式的类型来决定的，而不是由表达式运行时求值结果决定的。例如：

``` kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```

这个例子会输出 "c"，因为调用的扩展函数只取决于参数 `c` 的声明类型，该类型是 `C` 类。

如果一个类定义有一个成员函数和一个扩展函数，而这两个函数又有相同的接收者类型、相同的名字并且都适用给定的参数，这种情况**总是取成员函数**。例如：

``` kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```

如果我们调用 `C` 类型 `c`的 `c.foo()`，它将输出“member”，而不是“extension”。

当然，扩展函数重载同样名字但不同签名成员函数也完全可以：

``` kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo(i: Int) { println("extension") }
```

调用 `C().foo(1)` 将输出 "extension"。

### 可空接收者

注意可以为可空的接收者类型定义扩展。这样的扩展可以在对象变量上调用，即使其值为 null，并且可以在函数体内检测 `this == null`，这能让你在没有检测 null 的时候调用 Kotlin 中的 toString()：检测发生在扩展函数的内部。

``` kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // 空检测之后，“this”会自动转换为非空类型，所以下面的 toString()
    // 解析为 Any 类的成员函数
    return toString()
}
```

### 扩展属性

和函数类似，Kotlin 支持扩展属性：

``` kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

注意：由于扩展没有实际的将成员插入类中，因此对扩展属性来说[幕后字段](properties.html#幕后字段)是无效的。这就是为什么**扩展属性不能有初始化器**。他们的行为只能由显式提供的 getters/setters 定义。

例如:

``` kotlin
val Foo.bar = 1 // 错误：扩展属性不能有初始化器
```

### 伴生对象的扩展

如果一个类定义有一个[伴生对象](object-declarations.html#伴生对象) ，你也可以为伴生对象定义扩展函数和属性：

``` kotlin
class MyClass {
    companion object { }  // 将被称为 "Companion"
}

fun MyClass.Companion.foo() {
    // ……
}
```

就像伴生对象的其他普通成员，只需用类名作为限定符去调用他们

``` kotlin
MyClass.foo()
```

### 扩展的作用域

大多数时候我们在顶层定义扩展，即直接在包里：

``` kotlin
package foo.bar

fun Baz.goo() { …… }
```

要使用所定义包之外的一个扩展，我们需要在调用方导入它：

``` kotlin
package com.example.usage

import foo.bar.goo // 导入所有名为“goo”的扩展
                   // 或者
import foo.bar.*   // 从“foo.bar”导入一切

fun usage(baz: Baz) {
    baz.goo()
}
```

更多信息参见[导入](packages.html#导入)

### 扩展声明为成员

在一个类内部你可以为另一个类声明扩展。在这样的扩展内部，有多个 _隐式接收者_ ——其中的对象成员可以无需通过限定符访问。扩展声明所在的类的实例称为 _分发接收者_，扩展方法调用所在的接收者类型的实例称为 _扩展接收者_ 。

``` kotlin
class D {
    fun bar() { …… }
}

class C {
    fun baz() { …… }

    fun D.foo() {
        bar()   // 调用 D.bar
        baz()   // 调用 C.baz
    }

    fun caller(d: D) {
        d.foo()   // 调用扩展函数
    }
}
```

对于分发接收者和扩展接收者的成员名字冲突的情况，扩展接收者优先。要引用分发接收者的成员你可以使用 [限定的 this 语法](this-expressions.html#限定的-this)。

``` kotlin
class C {
    fun D.foo() {
        toString()         // 调用 D.toString()
        this@C.toString()  // 调用 C.toString()
    }
```

声明为成员的扩展可以声明为 `open` 并在子类中覆盖。这意味着这些函数的分发对于分发接收者类型是虚拟的，但对于扩展接收者类型是静态的。

``` kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()   // 调用扩展函数
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

C().caller(D())   // 输出 "D.foo in C"
C1().caller(D())  // 输出 "D.foo in C1" —— 分发接收者虚拟解析
C().caller(D1())  // 输出 "D.foo in C" —— 扩展接收者静态解析
```

### 动机

在 Java 中，我们将类命名为“\*Utils”：`FileUtils`、`StringUtils` 等，著名的 `java.util.Collections` 也属于同一种命名方式。关于这些 Utils-类的不愉快的部分是代码写成这样：

```java
// Java
Collections.swap(list, Collections.binarySearch(list, Collections.max(otherList)), Collections.max(list));
```

这些类名总是碍手碍脚的，我们可以通过静态导入达到这样效果：

```java
// Java
swap(list, binarySearch(list, max(otherList)), max(list));
```

这会变得好一点，但是我们并没有从 IDE 强大的自动补全功能中得到帮助。如果能这样就更好了：

```java
// Java
list.swap(list.binarySearch(otherList.max()), list.max());
```

但是我们不希望在 `List` 类内实现这些所有可能的方法，对吧？这时候扩展将会帮助我们。
