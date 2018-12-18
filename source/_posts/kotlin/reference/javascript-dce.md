---
title: JavaScript DCE

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - JavaScript
date: 2018-05-22 11:31:56
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

自版本 1.1.4 起，Kotlin/JS 包含了一个无用代码消除（DCE，dead code elimination）工具。该工具允许在生成的 JS 中删除未使用的属性、函数和类。出现未使用的声明有这几种可能情况：

*   函数可以内联并且从未直接调用（除少数情况之外，这总会出现）。
*   你所使用的共享库提供了比实际需要更多的功能/函数。例如，标准库（`kotlin.js`）包含用于操作列表、数组、字符序列、DOM 适配器等的函数/功能，这些一起提供了大约 1.3 mb 的文件。一个简单的“Hello, world”应用程序只需要控制台程序，整个文件只有几千字节。

无用代码消除通常也称为“tree shaking”。

<!-- more -->

## 如何使用

DCE 工具目前对 Gradle 可用。

要激活 DCE 工具，请将以下这行添加到 `build.gradle` 中：

``` groovy
apply plugin: 'kotlin-dce-js'
```

请注意，如果你正在使用多项目构建，那么应该将插件应用在作为应用程序入口点的主项目。

默认情况下，可以在路径 `$BUILD_DIR/min/` 中找到生成的一组 JavaScript 文件（你的应用程序与所有依赖关系），其中 `$BUILD_DIR` 是生成 JavaScript 的路径（通常是 `build/classes/main`）。

### 配置

要在主源集上配置 DCE，可以使用 `runDceKotlinJs` 任务（以及用于其他源集对应的 `runDce<sourceSetName>KotlinJs`）。

有时你直接在 JavaScript 中使用一个 Kotlin 声明，而被 DCE 给去除掉了。你可能想保留这个声明。 为此，你可以在 `build.gradle` 中使用以下语法：

``` groovy
runDceKotlinJs.keep "declarationToKeep"[, "declarationToKeep", ...]
```

其中 `declarationToKeep` 具有以下语法：

```
moduleName.dot.separated.package.name.declarationName
```

例如，考虑一个模块命名为 `kotlin-js-example`，它在 `org.jetbrains.kotlin.examples` 包中包含一个名为 `toKeep` 的函数。使用以下这行：

``` groovy
runDceKotlinJs.keep "kotlin-js-example_main.org.jetbrains.kotlin.examples.toKeep"
```

请注意，如果函数具有参数，它的名称会被修饰，因此在 keep 指令中应该使用修饰后的名称。

### 开发模式

运行 DCE 在每次构建时会额外花费一些时间，而且输出大小在开发过程中无关紧要。可以通过 DCE 任务的 `dceOptions.devMode` 标志使 DCE 工具跳过实际的无效代码消除从而缩短开发构建时间。

例如，如需根据自定义条件禁用 `main` 源集的 DCE 并且总是禁用 `test` 代码的 DCE，请将下述几行添加到构建脚本中：

``` groovy
runDceKotlinJs.dceOptions.devMode = isDevMode
runDceTestKotlinJs.dceOptions.devMode = true
```

### 示例

显示如何将 Kotlin 与 DCE 及 webpack 集成并得到一个小的捆绑的完整示例，可以在[这里](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/js-dce)找到。

## 注意事项

*   对于 1.1.x 版本，DCE 工具是一个 _实验性的_ 功能。这并不意味着我们要删除它，或者它不能用于生产。这意味着我们可能更改配置参数的名称、默认设置等等。
*   目前，如果你的项目是共享库，那么不应使用 DCE 工具。它只适用于开发应用程序（可能使用共享库）时。原因是：DCE 不知道库的哪些部分会被用户的应用程序所使用。
*   DCE 不会通过删除不必要的空格及缩短标识符来执行代码压缩（丑化）。对于此目的，你应该使用现有的工具，如 UglifyJS（https://github.com/mishoo/UglifyJS2 ）或者 Google Closure Compiler（https://developers.google.com/closure/compiler/ ）。
