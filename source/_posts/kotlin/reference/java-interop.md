---
title: 在 Kotlin 中调用 Java 代码

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - Java 互操作
date: 2018-05-22 11:13:16
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

Kotlin 在设计时就考虑了 Java 互操作性。可以从 Kotlin 中自然地调用现存的 Java 代码，并且在 Java 代码中也可以很顺利地调用 Kotlin 代码。在本节中我们会介绍从 Kotlin 中调用 Java 代码的一些细节。

<!-- more -->

几乎所有 Java 代码都可以使用而没有任何问题：

``` kotlin
import java.util.*

fun demo(source: List<Int>) {
    val list = ArrayList<Int>()
    // “for”-循环用于 Java 集合：
    for (item in source) {
        list.add(item)
    }
    // 操作符约定同样有效：
    for (i in 0..source.size - 1) {
        list[i] = source[i] // 调用 get 和 set
    }
}
```

## Getter 和 Setter

遵循 Java 约定的 getter 和 setter 的方法（名称以 `get` 开头的无参数方法和以 `set` 开头的单参数方法）在 Kotlin 中表示为属性。 `Boolean` 访问器方法（其中 getter 的名称以 `is` 开头而 setter 的名称以 `set` 开头）会表示为与 getter 方法具有相同名称的属性。

例如：

``` kotlin
import java.util.Calendar

fun calendarDemo() {
    val calendar = Calendar.getInstance()
    if (calendar.firstDayOfWeek == Calendar.SUNDAY) {  // 调用 getFirstDayOfWeek()
        calendar.firstDayOfWeek = Calendar.MONDAY      // 调用ll setFirstDayOfWeek()
    }
    if (!calendar.isLenient) {                         // 调用 isLenient()
        calendar.isLenient = true                      // 调用 setLenient()
    }
}
```

请注意，如果 Java 类只有一个 setter，它在 Kotlin 中不会作为属性可见，因为 Kotlin 目前不支持只写（set-only）属性。

## 返回 void 的方法

如果一个 Java 方法返回 void，那么从 Kotlin 调用时中返回 `Unit`。
万一有人使用其返回值，它将由 Kotlin 编译器在调用处赋值，
因为该值本身是预先知道的（是 `Unit`）。

## 将 Kotlin 中是关键字的 Java 标识符进行转义

一些 Kotlin 关键字在 Java 中是有效标识符：`in`、 `object`、 `is` 等等。
如果一个 Java 库使用了 Kotlin 关键字作为方法，你仍然可以通过反引号（`）字符转义它来调用该方法：

``` kotlin
foo.`is`(bar)
```

## 空安全与平台类型

Java 中的任何引用都可能是 `null`，这使得 Kotlin 对来自 Java 的对象要求严格空安全是不现实的。
Java 声明的类型在 Kotlin 中会被特别对待并称为*平台类型*。对这种类型的空检查会放宽，
因此它们的安全保证与在 Java 中相同（更多请参见[下文](#已映射类型)）。

考虑以下示例：

``` kotlin
val list = ArrayList<String>() // 非空（构造函数结果）
list.add("Item")
val size = list.size // 非空（原生 int）
val item = list[0] // 推断为平台类型（普通 Java 对象）
```

当我们调用平台类型变量的方法时，Kotlin 不会在编译时报告可空性错误，
但在运行时调用可能会失败，因为空指针异常或者 Kotlin 生成的阻止空值传播的断言：

``` kotlin
item.substring(1) // 允许，如果 item == null 可能会抛出异常
```

平台类型是*不可标示*的，意味着不能在语言中明确地写下它们。
当把一个平台值赋值给一个 Kotlin 变量时，可以依赖类型推断（该变量会具有推断出的的平台类型，
如上例中 `item` 所具有的类型），或者我们可以选择我们期望的类型（可空或非空类型均可）：

``` kotlin
val nullable: String? = item // 允许，没有问题
val notNull: String = item // 允许，运行时可能失败
```

如果我们选择非空类型，编译器会在赋值时触发一个断言。这防止 Kotlin 的非空变量保存空值。当我们把平台值传递给期待非空值等的 Kotlin 函数时，也会触发断言。
总的来说，编译器尽力阻止空值通过程序向远传播（尽管鉴于泛型的原因，有时这不可能完全消除）。

### 平台类型表示法

如上所述，平台类型不能在程序中显式表述，因此在语言中没有相应语法。
然而，编译器和 IDE 有时需要（在错误信息中、参数信息中等）显示他们，所以我们用一个助记符来表示他们：

* `T!` 表示“`T` 或者 `T?`”，
* `(Mutable)Collection<T>!` 表示“可以可变或不可变、可空或不可空的 `T` 的 Java 集合”，
* `Array<(out) T>!` 表示“可空或者不可空的 `T`（或 `T` 的子类型）的 Java 数组”

### 可空性注解

具有可空性注解的Java类型并不表示为平台类型，而是表示为实际可空或非空的
Kotlin 类型。编译器支持多种可空性注解，包括：

  * [JetBrains](https://www.jetbrains.com/idea/help/nullable-and-notnull-annotations.html)
（`org.jetbrains.annotations` 包中的  `@Nullable` 和 `@NotNull`）
  * Android（`com.android.annotations` 和 `android.support.annotations`)
  * JSR-305（`javax.annotation`，详见下文）
  * FindBugs（`edu.umd.cs.findbugs.annotations`）
  * Eclipse（`org.eclipse.jdt.annotation`）
  * Lombok（`lombok.NonNull`）。

你可以在 [Kotlin 编译器源代码](https://github.com/JetBrains/kotlin/blob/master/core/descriptor.loader.java/src/org/jetbrains/kotlin/load/java/JvmAnnotationNames.kt)中找到完整的列表。

### Annotating type parameters

It is possible to annotate type arguments of generic types to provide nullability information for them as well. For example, consider these annotations on a Java declaration:

```java
@NotNull
Set<@NotNull String> toSet(@NotNull Collection<@NotNull String> elements) { ... }
```

It leads to the following signature seen in Kotlin:

``` kotlin
fun toSet(elements: (Mutable)Collection<String>) : (Mutable)Set<String> { ... }
```

Note the `@NotNull` annotations on `String` type arguments. Without them, we get platform types in the type arguments:

``` kotlin
fun toSet(elements: (Mutable)Collection<String!>) : (Mutable)Set<String!> { ... }
```

Annotating type arguments works with Java 8 target or higher and requires the nullability annotations to support the `TYPE_USE` target (`org.jetbrains.annotations` supports this in version 15 and above).

### JSR-305 支持

已支持 [JSR-305](https://jcp.org/en/jsr/detail?id=305) 中定义的 [`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html) 注解来表示 Java 类型的可空性。

如果 `@Nonnull(when = ...)` 值为 `When.ALWAYS`，那么该注解类型会被视为非空；`When.MAYBE` 与
`When.NEVER` 表示可空类型；而 `When.UNKNOWN` 强制类型为[平台类型](#空安全与平台类型)。

可针对 JSR-305 注解编译库，但不需要为库的消费者将注解构件（如 `jsr305.jar`）指定为编译依赖。Kotlin 编译器可以从库中读取 JSR-305 注解，并不需要该注解出现在类路径中。

自 Kotlin 1.1.50 起，也支持[自定义可空限定符（KEEP-79）](https://github.com/Kotlin/KEEP/blob/41091f1cc7045142181d8c89645059f4a15cc91a/proposals/jsr-305-custom-nullability-qualifiers.md)（见下文）。

#### 类型限定符别称（自 1.1.50 起）

如果一个注解类型同时标注有 [`@TypeQualifierNickname`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierNickname.html) 与 JSR-305 `@Nonnull`（或者它的其他别称，如 `@CheckForNull`），那么该注解类型自身将用于检索精确的可空性，且具有与该可空性注解相同的含义：

``` java
@TypeQualifierNickname
@Nonnull(when = When.ALWAYS)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyNonnull {
}

@TypeQualifierNickname
@CheckForNull // 另一个类型限定符别称的别称
@Retention(RetentionPolicy.RUNTIME)
public @interface MyNullable {
}

interface A {
    @MyNullable String foo(@MyNonnull String x); 
    // 在 Kotlin（严格模式）中：`fun foo(x: String): String?`
    
    String bar(List<@MyNonnull String> x);       
    // 在 Kotlin（严格模式）中：`fun bar(x: List<String>!): String!`
}
```

#### 类型限定符默认值（自 1.1.50 起）

[`@TypeQualifierDefault`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierDefault.html) 引入应用时在所标注元素的作用域内定义默认可空性的注解。

这些注解类型应自身同时标注有 `@Nonnull`（或其别称）与 `@TypeQualifierDefault(...)` 注解，
后者带有一到多个 `ElementType` 值：
* `ElementType.METHOD` 用于方法的返回值；
* `ElementType.PARAMETER` 用于值参数；
* `ElementType.FIELD` 用于字段；以及
* `ElementType.TYPE_USE`（自 1.1.60 起）适用于任何类型，包括类型参数、类型参数的上界与通配符类型。


当类型并未标注可空性注解时使用默认可空性，并且该默认值是由最内层标注有带有与所用类型相匹配的 `ElementType` 的类型限定符默认注解的元素确定。

```java
@Nonnull
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER})
public @interface NonNullApi {
}

@Nonnull(when = When.MAYBE)
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE_USE})
public @interface NullableApi {
}

@NullableApi
interface A {
    String foo(String x); // fun foo(x: String?): String?
 
    @NotNullApi // 覆盖来自接口的默认值
    String bar(String x, @Nullable String y); // fun bar(x: String, y: String?): String 
    
    // 由于 `@NullableApi` 具有 `TYPE_USE` 元素类型，
    // 因此认为 List<String> 类型参数是可空的：
    String baz(List<String> x); // fun baz(List<String?>?): String?

    // “x”参数仍然是平台类型，因为有显式
    // UNKNOWN 标记的可空性注解：
    String qux(@Nonnull(when = When.UNKNOWN) String x); // fun baz(x: String!): String?
}
```

> 注意：本例中的类型只在启用了严格模式时出现，否则仍是平台类型。参见 [*@UnderMigration* 注解](#UnderMigration-注解（自-1-1-60-起）)与[编译器配置](#编译器配置)两节。

也支持包级的默认可空性：

```java
// 文件：test/package-info.java
@NonNullApi // 默认将“test”包中所有类型声明为不可空
package test;
```

#### `@UnderMigration` 注解（自 1.1.60 起）

库的维护者可以使用 `@UnderMigration` 注解（在单独的构件 `kotlin-annotations-jvm` 中提供）来定义可为空性类型限定符的迁移状态。

`@UnderMigration(status = ...)` 中的状态值指定了编译器如何处理 Kotlin 中注解类型的不当用法（例如，使用 `@MyNullable` 标注的类型值作为非空值）：

* `MigrationStatus.STRICT` 使注解像任何纯可空性注解一样工作，即对不当用法报错并影响注解声明内的类型在 Kotlin 中的呈现；

* 对于 `MigrationStatus.WARN`，不当用法报为警告而不是错误；但注解声明内的类型仍是平台类型；

* 而 `MigrationStatus.IGNORE` 则使编译器完全忽略可空性注解。

库的维护者还可以将 `@UnderMigration` 状态添加到类型限定符别称与类型限定符默认值：

```java
@Nonnull(when = When.ALWAYS)
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER})
@UnderMigration(status = MigrationStatus.WARN)
public @interface NonNullApi {
}

// 类中的类型是非空的，但是只报警告
// 因为 `@NonNullApi` 标注了 `@UnderMigration(status = MigrationStatus.WARN)`
@NonNullApi 
public class Test {}
```

注意：可空性注解的迁移状态并不会从其类型限定符别称继承，而是适用于默认类型限定符的用法。

如果默认类型限定符使用类型限定符别称，并且它们都标注有 `@UnderMigration`，那么使用默认类型限定符的状态。

#### 编译器配置

可以通过添加带有以下选项的 `-Xjsr305` 编译器标志来配置 JSR-305 检测：

* `-Xjsr305={strict|warn|ignore}` 设置非 `@UnderMigration` 注解的行为。
自定义的可空性限定符，尤其是 `@TypeQualifierDefault` 已经在很多知名库中流传，而用户更新到包含 JSR-305 支持的 Kotlin 版本时可能需要平滑迁移。自 Kotlin 1.1.60 起，这一标志只影响非 `@UnderMigration` 注解。

* `-Xjsr305=under-migration:{strict|warn|ignore}`（自 1.1.60 起）覆盖 `@UnderMigration` 注解的行为。
用户可能对库的迁移状态有不同的看法：
他们可能希望在官方迁移状态为 `WARN` 时报错误，反之亦然，他们可能希望推迟错误报告直到他们完成迁移。

* `-Xjsr305=@<fq.name>:{strict|warn|ignore}`（自 1.1.60 起）覆盖单个注解的行为，其中 `<fq.name>` 是该注解的完整限定类名。对于不同的注解可以多次出现。这对于管理特定库的迁移状态非常有用。

其中 `strict`、 `warn` 与 `ignore` 值的含义与 `MigrationStatus` 中的相同，并且只有 `strict` 模式会影响注解声明中的类型在 Kotlin 中的呈现。

> 注意：内置的 JSR-305 注解 [*@Nonnull*](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html)、 [*@Nullable*](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nullable.html) 与 [*@CheckForNull*](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/CheckForNull.html) 总是启用并影响所注解的声明在 Kotlin 中呈现，无论如何配置编译器的 `-Xjsr305` 标志。

例如，将 `-Xjsr305=ignore -Xjsr305=under-migration:ignore -Xjsr305=@org.library.MyNullable:warn` 添加到编译器参数中，会使编译器对由 `@org.library.MyNullable` 标注的不当用法生成警告，而忽略所有其他 JSR-305 注解。

对于 kotlin 1.1.50+/1.2 版本，其默认行为等同于 `-Xjsr305=warn`。
`strict` 值应认为是实验性的（以后可能添加更多检测）。

## 已映射类型

Kotlin 特殊处理一部分 Java 类型。这样的类型不是“按原样”从 Java 加载，而是 _映射_ 到相应的 Kotlin 类型。
映射只发生在编译期间，运行时表示保持不变。
Java 的原生类型映射到相应的 Kotlin 类型（请记住[平台类型](#空安全与平台类型)）：

| **Java 类型** | **Kotlin 类型**  |
| ------------- | ---------------- |
| `byte`        | `kotlin.Byte`    |
| `short`       | `kotlin.Short`   |
| `int`         | `kotlin.Int`     |
| `long`        | `kotlin.Long`    |
| `char`        | `kotlin.Char`    |
| `float`       | `kotlin.Float`   |
| `double`      | `kotlin.Double`  |
| `boolean`     | `kotlin.Boolean` |

一些非原生的内置类型也会作映射：

| **Java 类型**            | **Kotlin 类型**        |
| ------------------------ | ---------------------- |
| `java.lang.Object`       | `kotlin.Any!`          |
| `java.lang.Cloneable`    | `kotlin.Cloneable!`    |
| `java.lang.Comparable`   | `kotlin.Comparable!`   |
| `java.lang.Enum`         | `kotlin.Enum!`         |
| `java.lang.Annotation`   | `kotlin.Annotation!`   |
| `java.lang.Deprecated`   | `kotlin.Deprecated!`   |
| `java.lang.CharSequence` | `kotlin.CharSequence!` |
| `java.lang.String`       | `kotlin.String!`       |
| `java.lang.Number`       | `kotlin.Number!`       |
| `java.lang.Throwable`    | `kotlin.Throwable!`    |

Java 的装箱原始类型映射到可空的 Kotlin 类型：

| **Java type**         | **Kotlin type**   |
| --------------------- | ----------------- |
| `java.lang.Byte`      | `kotlin.Byte?`    |
| `java.lang.Short`     | `kotlin.Short?`   |
| `java.lang.Integer`   | `kotlin.Int?`     |
| `java.lang.Long`      | `kotlin.Long?`    |
| `java.lang.Character` | `kotlin.Char?`    |
| `java.lang.Float`     | `kotlin.Float?`   |
| `java.lang.Double`    | `kotlin.Double?`  |
| `java.lang.Boolean`   | `kotlin.Boolean?` |

请注意，用作类型参数的装箱原始类型映射到平台类型：
例如，`List<java.lang.Integer>` 在 Kotlin 中会成为 `List<Int!>`。

集合类型在 Kotlin 中可以是只读的或可变的，因此 Java 集合类型作如下映射：
（下表中的所有 Kotlin 类型都驻留在 `kotlin.collections`包中）:

| **Java 类型**     | **Kotlin 只读类型** | **Kotlin 可变类型**            | **加载的平台类型**                   |
| ----------------- | ------------------- | ------------------------------ | ------------------------------------ |
| `Iterator<T>`     | `Iterator<T>`       | `MutableIterator<T>`           | `(Mutable)Iterator<T>!`              |
| `Iterable<T>`     | `Iterable<T>`       | `MutableIterable<T>`           | `(Mutable)Iterable<T>!`              |
| `Collection<T>`   | `Collection<T>`     | `MutableCollection<T>`         | `(Mutable)Collection<T>!`            |
| `Set<T>`          | `Set<T>`            | `MutableSet<T>`                | `(Mutable)Set<T>!`                   |
| `List<T>`         | `List<T>`           | `MutableList<T>`               | `(Mutable)List<T>!`                  |
| `ListIterator<T>` | `ListIterator<T>`   | `MutableListIterator<T>`       | `(Mutable)ListIterator<T>!`          |
| `Map<K, V>`       | `Map<K, V>`         | `MutableMap<K, V>`             | `(Mutable)Map<K, V>!`                |
| `Map.Entry<K, V>` | `Map.Entry<K, V>`   | `MutableMap.MutableEntry<K,V>` | `(Mutable)Map.(Mutable)Entry<K, V>!` |

Java 的数组按[下文](java-interop.html#Java-数组)所述映射：

| **Java 类型** | **Kotlin 类型**               |
| ------------- | ----------------------------- |
| `int[]`       | `kotlin.IntArray!`            |
| `String[]`    | `kotlin.Array<(out) String>!` |

注意：这些 Java 类型的静态成员不能在相应 Kotlin 类型的[伴生对象](object-declarations.html#伴生对象)中直接访问。要调用它们，请使用 Java 类型的完整限定名，例如 `java.lang.Integer.toHexString(foo)`。

## Kotlin 中的 Java 泛型

Kotlin 的泛型与 Java 有点不同（参见[泛型](generics.html)）。当将 Java 类型导入 Kotlin 时，我们会执行一些转换：

* Java 的通配符转换成类型投影，
  * `Foo<? extends Bar>` 转换成 `Foo<out Bar!>!`，
  * `Foo<? super Bar>` 转换成 `Foo<in Bar!>!`；

* Java的原始类型转换成星投影，
  * `List` 转换成 `List<*>!`，即 `List<out Any?>!`。

和 Java 一样，Kotlin 在运行时不保留泛型，即对象不携带传递到他们构造器中的那些类型参数的实际类型。
即 `ArrayList<Integer>()` 和 `ArrayList<Character>()` 是不能区分的。
这使得执行 `is` 检测不可能照顾到泛型。
Kotlin 只允许 `is` 检测星投影的泛型类型：

``` kotlin
if (a is List<Int>) // 错误：无法检查它是否真的是一个 Int 列表
// but
if (a is List<*>) // OK：不保证列表的内容
```

### Java 数组

与 Java 不同，Kotlin 中的数组是不型变的。这意味着 Kotlin 不允许我们把一个 `Array<String>` 赋值给一个 `Array<Any>`，
从而避免了可能的运行时故障。Kotlin 也禁止我们把一个子类的数组当做超类的数组传递给 Kotlin 的方法，
但是对于 Java 方法，这是允许的（通过 `Array<(out) String>!` 这种形式的[平台类型](#空安全与平台类型)）。

Java 平台上，数组会使用原生数据类型以避免装箱/拆箱操作的开销。
由于 Kotlin 隐藏了这些实现细节，因此需要一个变通方法来与 Java 代码进行交互。
对于每种原生类型的数组都有一个特化的类（`IntArray`、 `DoubleArray`、 `CharArray` 等等）来处理这种情况。
它们与 `Array` 类无关，并且会编译成 Java 原生类型数组以获得最佳性能。

假设有一个接受 int 数组索引的 Java 方法：

``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // 在此编码……
    }
}
```

在 Kotlin 中你可以这样传递一个原生类型的数组：

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndices(array)  // 将 int[] 传给方法
```

当编译为 JVM 字节代码时，编译器会优化对数组的访问，这样就不会引入任何开销：

``` kotlin
val array = arrayOf(1, 2, 3, 4)
array[1] = array[1] * 2 // 不会实际生成对 get() 和 set() 的调用
for (x in array) { // 不会创建迭代器
    print(x)
}
```

即使当我们使用索引定位时，也不会引入任何开销：

``` kotlin
for (i in array.indices) {// 不会创建迭代器
    array[i] += 2
}
```

最后，`in`-检测也没有额外开销：

``` kotlin
if (i in array.indices) { // 同 (i >= 0 && i < array.size)
    print(array[i])
}
```

## Java 可变参数

Java 类有时声明一个具有可变数量参数（varargs）的方法来使用索引：

``` java
public class JavaArrayExample {

    public void removeIndicesVarArg(int... indices) {
        // 在此编码……
    }
}
```

在这种情况下，你需要使用展开运算符 `*` 来传递 `IntArray`：

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

目前无法传递 `null` 给一个声明为可变参数的方法。

## 操作符

由于 Java 无法标记用于运算符语法的方法，Kotlin 允许具有正确名称和签名的任何 Java 方法作为运算符重载和其他约定（`invoke()` 等）使用。
不允许使用中缀调用语法调用 Java 方法。


## 受检异常

在 Kotlin 中，所有异常都是非受检的，这意味着编译器不会强迫你捕获其中的任何一个。
因此，当你调用一个声明受检异常的 Java 方法时，Kotlin 不会强迫你做任何事情：

``` kotlin
fun render(list: List<*>, to: Appendable) {
    for (item in list) {
        to.append(item.toString()) // Java 会要求我们在这里捕获 IOException
    }
}
```

## 对象方法

当 Java 类型导入到 Kotlin 中时，类型 `java.lang.Object` 的所有引用都成了 `Any`。
而因为 `Any` 不是平台指定的，它只声明了 `toString()`、`hashCode()` 和 `equals()` 作为其成员，
所以为了能用到 `java.lang.Object` 的其他成员，Kotlin 要用到[扩展函数](extensions.html)。

### wait()/notify()

[《Effective Java》第三版](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 第 81 条善意地建议优先使用并发工具（concurrency utilities）而不是 `wait()` 和 `notify()`。
因此，类型 `Any` 的引用不提供这两个方法。
如果你真的需要调用它们的话，你可以将其转换为 `java.lang.Object`：

``` kotlin
(foo as java.lang.Object).wait()
```

### getClass()

要取得对象的 Java 类，请在[类引用](reflection.html#类引用)上使用 `java` 扩展属性：

``` kotlin
val fooClass = foo::class.java
```

上面的代码使用了自 Kotlin 1.1 起支持的[绑定的类引用](reflection.html#绑定的类引用（自-1-1-起）)。你也可以使用 `javaClass` 扩展属性：

``` kotlin
val fooClass = foo.javaClass
```

### clone()

要覆盖 `clone()`，需要继承 `kotlin.Cloneable`：

``` kotlin

class Example : Cloneable {
    override fun clone(): Any { …… }
}
```

不要忘记[《Effective Java》第三版](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 的第 13 条: *谨慎地改写clone*。

### finalize()

要覆盖 `finalize()`，所有你需要做的就是简单地声明它，而不需要 *override*{:.keyword} 关键字：

``` kotlin
class C {
    protected fun finalize() {
        // 终止化逻辑
    }
}
```

根据 Java 的规则，`finalize()` 不能是 *private*{: .keyword } 的。

## 从 Java 类继承

在 kotlin 中，类的超类中最多只能有一个 Java 类（以及按你所需的多个 Java 接口）。

## 访问静态成员

Java 类的静态成员会形成该类的“伴生对象”。我们无法将这样的“伴生对象”作为值来传递，
但可以显式访问其成员，例如：

``` kotlin
if (Character.isLetter(a)) {
    // ……
}
```

要访问[已映射](#已映射类型)到 Kotlin 类型的 Java 类型的静态成员，请使用 Java 类型的完整限定名：`java.lang.Integer.bitCount(foo)`。

## Java 反射

Java 反射适用于 Kotlin 类，反之亦然。如上所述，你可以使用 `instance::class.java`,
`ClassName::class.java` 或者 `instance.javaClass` 通过 `java.lang.Class` 来进入 Java 反射。

其他支持的情况包括为一个 Kotlin 属性获取一个 Java 的 getter/setter 方法或者幕后字段、为一个 Java 字段获取一个 `KProperty`、为一个 `KFunction` 获取一个 Java 方法或者构造函数，反之亦然。

## SAM 转换

就像 Java 8 一样，Kotlin 支持 SAM 转换。这意味着 Kotlin 函数字面值可以被自动的转换成只有一个非默认方法的 Java 接口的实现，只要这个方法的参数类型能够与这个 Kotlin 函数的参数类型相匹配。

你可以这样创建 SAM 接口的实例：

``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

……以及在方法调用中：

``` kotlin
val executor = ThreadPoolExecutor()
// Java 签名：void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```

如果 Java 类有多个接受函数式接口的方法，那么可以通过使用将 lambda 表达式转换为特定的 SAM 类型的适配器函数来选择需要调用的方法。这些适配器函数也会按需由编译器生成：

``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```

请注意，SAM 转换只适用于接口，而不适用于抽象类，即使这些抽象类也只有一个抽象方法。

还要注意，此功能只适用于 Java 互操作；因为 Kotlin 具有合适的函数类型，所以不需要将函数自动转换为 Kotlin 接口的实现，因此不受支持。

## 在 Kotlin 中使用 JNI

要声明一个在本地（C 或 C++）代码中实现的函数，你需要使用 `external` 修饰符来标记它：

``` kotlin
external fun foo(x: Int): Double
```

其余的过程与 Java 中的工作方式完全相同。