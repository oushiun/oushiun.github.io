---
title: 嵌套类与内部类

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 类与对象
date: 2018-05-17 11:38:12
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

类可以嵌套在其他类中：

``` kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

<!-- more -->

### 内部类

类可以标记为 `inner` 以便能够访问外部类的成员。内部类会带有一个对外部类的对象的引用：

``` kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

参见[限定的 this 表达式](this-expressions.html)以了解内部类中的 `this` 的消歧义用法。

### 匿名内部类

使用[对象表达式](object-declarations.html#对象表达式)创建匿名内部类实例：

``` kotlin
window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ……
    }

    override fun mouseEntered(e: MouseEvent) {
        // ……
    }
})
```

如果对象是函数式 Java 接口（即具有单个抽象方法的 Java 接口）的实例，你可以使用带接口类型前缀的 lambda 表达式创建它：

``` kotlin
val listener = ActionListener { println("clicked") }
```
