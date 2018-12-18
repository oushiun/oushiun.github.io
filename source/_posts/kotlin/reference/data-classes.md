---
title: 数据类

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 类与对象
date: 2018-05-17 10:31:00
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

我们经常创建一些只保存数据的类。在这些类中，一些标准函数往往是从数据机械推导而来的。在 Kotlin 中，这叫做 _数据类_ 并标记为 `data`：

``` kotlin
data class User(val name: String, val age: Int)
```

<!-- more -->

编译器自动从主构造函数中声明的所有属性导出以下成员：

*   `equals()`/`hashCode()` 对；
*   `toString()` 格式是 `"User(name=John, age=42)"`；
*   [`componentN()` 函数](multi-declarations.html) 按声明顺序对应于所有属性；
*   `copy()` 函数（见下文）。

为了确保生成的代码的一致性和有意义的行为，数据类必须满足以下要求：

*   主构造函数需要至少有一个参数；
*   主构造函数的所有参数需要标记为 `val` 或 `var`；
*   数据类不能是抽象、开放、密封或者内部的；
*   （在 1.1 之前）数据类只能实现接口。

此外，成员生成遵循关于成员继承的这些规则：

*   如果在数据类体中有显式实现 `equals()`、 `hashCode()` 或者 `toString()`，或者这些函数在父类中有 *final* 实现，那么不会生成这些函数，而会使用现有函数；
*   如果超类型具有 *open* 的 `componentN()` 函数并且返回兼容的类型，那么会为数据类生成相应的函数，并覆盖超类的实现。如果超类型的这些函数由于签名不兼容或者是 final 而导致无法覆盖，那么会报错；
*   从一个已具 `copy(……)` 函数且签名匹配的类型派生一个数据类在 Kotlin 1.2 中已弃用，并且会在 Kotlin 1.3 中禁用。
*   不允许为 `componentN()` 以及 `copy()` 函数提供显式实现。

自 1.1 起，数据类可以扩展其他类（示例请参见[密封类](sealed-classes.html)）。

在 JVM 中，如果生成的类需要含有一个无参的构造函数，则所有的属性必须指定默认值。（参见[构造函数](classes.html#构造函数)）。

``` kotlin
data class User(val name: String = "", val age: Int = 0)
```

### 在类体中声明的属性

请注意，对于那些自动生成的函数，编译器只使用在主构造函数内部定义的属性。如需在生成的实现中排出一个属性，请将其声明在类体中：

``` kotlin
data class Person(val name: String) {
    var age: Int = 0
}
```

在 `toString()`、 `equals()`、 `hashCode()` 以及 `copy()` 的实现中只会用到 `name` 属性，并且只有一个 component 函数 `component1()`。虽然两个 `Person` 对象可以有不同的年龄，但它们会视为相等。

``` kotlin
data class Person(val name: String) {
    var age: Int = 0
}

fun main(args: Array<String>) {
    //sampleStart
    val person1 = Person("John")
    val person2 = Person("John")

    person1.age = 10
    person2.age = 20
    //sampleEnd

    println("person1 == person2: ${person1 == person2}")
    println("person1 with age ${person1.age}: ${person1}")
    println("person2 with age ${person2.age}: ${person2}")
}
```

### 复制

在很多情况下，我们需要复制一个对象改变它的一些属性，但其余部分保持不变。
`copy()` 函数就是为此而生成。对于上文的 `User` 类，其实现会类似下面这样：

``` kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)
```

这让我们可以写：

``` kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

### 数据类和解构声明

为数据类生成的 _Component 函数_ 使它们可在[解构声明](multi-declarations.html)中使用：

``` kotlin
val jane = User("Jane", 35)
val (name, age) = jane
println("$name, $age years of age") // 输出 "Jane, 35 years of age"
```

### 标准数据类

标准库提供了 `Pair` 和 `Triple`。尽管在很多情况下命名数据类是更好的设计选择，因为它们通过为属性提供有意义的名称使代码更具可读性。
