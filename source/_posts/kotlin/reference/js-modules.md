---
title: JavaScript 模块

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - JavaScript
date: 2018-05-22 11:28:55
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

Kotlin 允许你将 Kotlin 项目编译为热门模块系统的 JavaScript 模块。以下是可用选项的列表：

1.  无模块（Plain）。不为任何模块系统编译。像往常一样，你可以在全局作用域中以其名称访问模块。默认使用此选项。
2.  [异步模块定义（AMD，Asynchronous Module Definition）](https://github.com/amdjs/amdjs-api/wiki/AMD)，它尤其为 require.js 库所使用。
3.  [CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1) 约定，广泛用于 node.js/npm（`require` 函数和 `module.exports` 对象）
4.  统一模块定义（UMD，Unified Module Definitions），它与 _AMD_ 和 _CommonJS_ 兼容，并且当在运行时 _AMD_ 和 _CommonJS_ 都不可用时，作为“plain”使用。

<!-- more -->

## 选择目标模块系统

选择目标模块系统的方式取决于你的构建环境：

### 在 IntelliJ IDEA 中

设置每个模块：打开“File → Project Structure...”，在“Modules”中找到你的模块并选择其下的“Kotlin”facet。在“Module kind”字段中选择合适的模块系统。

为整个项目设置：打开“File → Settings”，选择“Build, Execution, Deployment”→“Compiler”→“Kotlin compiler”。 在“Module kind”字段中选择合适的模块系统。

### 在 Maven 中

要选择通过 Maven 编译时的模块系统，你应该设置 `moduleKind` 配置属性，即你的 `pom.xml` 应该看起来像这样：

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>
    <executions>
        <execution>
            <id>compile</id>
            <goals>
                <goal>js</goal>
            </goals>
        </execution>
    </executions>
    <!-- 插入这些行 -->
    <configuration>
        <moduleKind>commonjs</moduleKind>
    </configuration>
    <!-- 插入文本结束 -->
</plugin>
```

可用值包括：`plain`、 `amd`、 `commonjs`、 `umd`。

### 在 Gradle 中

要选择通过 Gradle 编译时的模块系统，你应该设置 `moduleKind` 属性，即

``` groovy
compileKotlin2Js.kotlinOptions.moduleKind = "commonjs"
```

可用的值类似于 Maven。

## `@JsModule` 注解

要告诉 Kotlin 一个 `external` 类、 包、 函数或者属性是一个 JavaScript 模块，你可以使用 `@JsModule`
注解。考虑你有以下 CommonJS 模块叫“hello”：

``` javascript
module.exports.sayHello = function(name) {
    alert('Hello, ' + name)
}
```

你应该在 Kotlin 中这样声明：

``` kotlin
@JsModule("hello")
external fun sayHello(name: String)
```

### 将 `@JsModule` 应用到包

一些 JavaScript 库导出包（命名空间）而不是函数和类。从 JavaScript 角度讲 它是一个具有一些成员的对象，这些成员*是*类、函数和属性。将这些包作为 Kotlin 对象导入通常看起来不自然。编译器允许使用以下助记符将导入的 JavaScript 包映射到 Kotlin 包：

``` kotlin
@file:JsModule("extModule")
package ext.jspackage.name

external fun foo()

external class C
```

其中相应的 JavaScript 模块的声明如下：

``` javascript
module.exports = {
    foo: {
        /* 此处一些代码 */
    },
    C: {
        /* 此处一些代码 */
    }
}
```

重要提示：标有 `@file:JsModule` 注解的文件无法声明非外部成员。下面的示例会产生编译期错误：

``` kotlin
@file:JsModule("extModule")
package ext.jspackage.name

external fun foo()

fun bar() = "!" + foo() + "!" // 此处报错
```

### 导入更深的包层次结构

在前文示例中，JavaScript 模块导出单个包。但是，一些 JavaScript 库会从模块中导出多个包。
Kotlin 也支持这种场景，尽管你必须为每个导入的包声明一个新的 `.kt` 文件。

例如，让我们的示例更复杂一些：

``` javascript
module.exports = {
    mylib: {
        pkg1: {
            foo: function() {
                /* 此处一些代码 */
            },
            bar: function() {
                /* 此处一些代码 */
            }
        },
        pkg2: {
            baz: function() {
                /* 此处一些代码 */
            }
        }
    }
}
```

要在 Kotlin 中导入该模块，你必须编写两个 Kotlin 源文件：

``` kotlin
@file:JsModule("extModule")
@file:JsQualifier("mylib.pkg1")
package extlib.pkg1

external fun foo()

external fun bar()
```

以及

``` kotlin
@file:JsModule("extModule")
@file:JsQualifier("mylib.pkg2")
package extlib.pkg2

external fun baz()
```

### `@JsNonModule` 注解

当一个声明具有 `@JsModule`、当你并不把它编译到一个 JavaScript 模块时，你不能在 Kotlin 代码中使用它。通常，开发人员将他们的库既作为 JavaScript 模块也作为可下载的`.js` 文件分发，你可以将这些文件复制到项目的静态资源，并通过 `<script>` 元素包含。 要告诉 Kotlin，可以在非模块环境中使用一个 `@JsModule` 声明，你应该放置 `@JsNonModule` 声明。例如，给定 JavaScript 代码：

``` javascript
function topLevelSayHello(name) {
    alert('Hello, ' + name)
}
if (module && module.exports) {
    module.exports = topLevelSayHello
}
```

可以这样描述：

``` kotlin
@JsModule("hello")
@JsNonModule
@JsName("topLevelSayHello")
external fun sayHello(name: String)
```

### 备注

Kotlin 以 `kotlin.js` 标准库作为单个文件分发，该文件本身被编译为 UMD 模块，因此你可以使用上述任何模块系统。也可以在 NPM 上使用 [*kotlin* 包](https://www.npmjs.com/package/kotlin)
