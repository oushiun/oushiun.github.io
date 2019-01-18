---
title: 使用 Gradle

toc: true
tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 工具
date: 2018-05-25 09:34:05
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

为了用 Gradle 构建 Kotlin，你应该[设置好 *kotlin-gradle* 插件](#插件和版本)，[将其应用](#针对-jvm)到你的项目中，并且[添加 *kotlin-stdlib* 依赖](#配置依赖)。这些操作也可以在 IntelliJ IDEA 中通过调用 Project action 中的 Tools \| Kotlin \| Configure Kotlin 自动执行。

<!-- more -->

## 插件和版本

使用 `kotlin-gradle-plugin` 编译 Kotlin 源代码和模块.

要使用的 Kotlin 版本通常定义为 `kotlin_version` 属性：

``` groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'

    repositories {
        mavenCentral()
    }

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
```

当通过 [Gradle 插件 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block) 与 [Gradle Kotlin DSL](https://github.com/gradle/kotlin-dsl) 使用 Kotlin Gradle 插件 1.1.1 及以上版本时，这不是必需的。

## 针对 JVM

针对 JVM，需要应用 Kotlin 插件：

``` groovy
apply plugin: "kotlin"
```

或者，从 Kotlin 1.1.1 起，可以使用 [Gradle 插件 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block) 来应用该插件：

``` groovy
plugins {
    id "org.jetbrains.kotlin.jvm" version "{{ site.data.releases.latest.version }}"
}
```
在这个块中的 `version` 必须是字面值，并且不能从其他构建脚本中应用。

对于 Gradle Kotlin DSL，请按以下方式应用插件：

``` kotlin
plugins {
    kotlin("jvm") version "{{ site.data.releases.latest.version }}"
}
```

Kotlin 源代码可以与同一个文件夹或不同文件夹中的 Java 源代码混用。默认约定是使用不同的文件夹：

``` groovy
project
    - src
        - main (root)
            - kotlin
            - java
```

如果不使用默认约定，那么应该更新相应的 *sourceSets* 属性：

``` groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
    main.java.srcDirs += 'src/main/myJava'
}
```

对于 Gradle Kotlin DSL，请改用 `java.sourceSets { …… }` 配置源集。

## 针对 JavaScript

当针对 JavaScript 时，须应用不同的插件：

``` groovy
apply plugin: "kotlin2js"
```

这个插件只适用于 Kotlin 文件，因此建议将 Kotlin 和 Java 文件分开（如果是同一项目包含 Java 文件的情况）。与针对 JVM 一样，如果不使用默认约定，我们需要使用 *sourceSets* 来指定源代码文件夹：

``` groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
}
```

除了输出的 JavaScript 文件，该插件默认会创建一个带二进制描述符的额外 JS 文件。
如果你是构建其他 Kotlin 模块可以依赖的可重用库，那么该文件是必需的，并且应该与转换结果一起分发。
其生成由 `kotlinOptions.metaInfo` 选项控制：

``` groovy
compileKotlin2Js {
    kotlinOptions.metaInfo = true
}
```

## 针对 Android

Android 的 Gradle 模型与普通 Gradle 有点不同，所以如果我们要构建一个用 Kotlin 编写的 Android 项目，我们需要用 *kotlin-android* 插件取代 *kotlin* 插件：

``` groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'

    ……

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
```

不要忘记配置[标准库依赖关系](#配置依赖)。

### Android Studio

如果使用 Android Studio，那么需要在 android 下添加以下内容：

``` groovy
android {
  ……

  sourceSets {
    main.java.srcDirs += 'src/main/kotlin'
  }
}
```

这让 Android Studio 知道该 kotlin 目录是源代码根目录，所以当项目模型加载到 IDE 中时，它会被正确识别。或者，你可以将 Kotlin 类放在 Java 源代码目录中，该目录通常位于 `src/main/java`。


## 配置依赖

除了上面显示的 `kotlin-gradle-plugin` 依赖之外，还需要添加 Kotlin 标准库的依赖：

``` groovy
repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib"
}
```

如果针对 JavaScript，请使用 `compile "org.jetbrains.kotlin:kotlin-stdlib-js"` 替代之。

如果是针对 JDK 7 或 JDK 8，那么可以使用扩展版本的 Kotlin 标准库，其中包含为新版 JDK 增加的额外的扩展函数。使用以下依赖之一来取代 `kotlin-stdlib`：

``` groovy
compile "org.jetbrains.kotlin:kotlin-stdlib-jdk7"
compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
```

对于 Gradle Kotlin DSL，以下表示法的依赖关系与其等价：

``` kotlin
dependencies {
    compile(kotlin("stdlib"))
    // 或者以下之一：
    compile(kotlin("stdlib-jdk7"))
    compile(kotlin("stdlib-jdk8"))
}
```

在 Kotlin 1.1.x 中，请使用 `kotlin-stdlib-jre7` 与 `kotlin-stdlib-jre8`。

如果你的项目中使用 [Kotlin 反射](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect.full/index.html)或者测试设施，你也需要添加相应的依赖：

``` groovy
compile "org.jetbrains.kotlin:kotlin-reflect"
testCompile "org.jetbrains.kotlin:kotlin-test"
testCompile "org.jetbrains.kotlin:kotlin-test-junit"
```

或者，对于 Gradle Kotlin DSL：

``` kotlin
compile(kotlin("reflect"))
testCompile(kotlin("test"))
testCompile(kotlin("test-junit"))
```

从 Kotlin 1.1.2 起，使用 `org.jetbrains.kotlin` group 的依赖项默认使用从已应用的插件获得的版本来解析。你可以用完整的依赖关系符号（如 `compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"`，或者在 Gradle Kotlin DSL 中用 `kotlin("stdlib", kotlinVersion)`）手动提供其版本。

## 注解处理

请参见 [Kotlin 注解处理工具](kapt.html)（`kapt`）的描述。

## 增量编译

Kotlin 支持 Gradle 中可选的增量编译。
增量编译跟踪构建之间源文件的改动，因此只有受这些改动影响的文件才会被编译。

从 Kotlin 1.1.1 起，默认启用增量编译。

有几种方法来覆盖默认设置：

  1. 将 `kotlin.incremental=true` 或者 `kotlin.incremental=false` 行添加到一个 `gradle.properties` 或者一个 `local.properties` 文件中；

  2. 将 `-Pkotlin.incremental=true` 或 `-Pkotlin.incremental=false` 添加到 Gradle 命令行参数。请注意，这样用法中，该参数必须添加到后续每个子构建，并且任何具有禁用增量编译的构建将使增量缓存失效。

请注意，第一次构建不会是增量的。

## 协程支持

[协程](coroutines.html)支持是 Kotlin 1.2 中的一个实验性的功能，所以当你在项目中使用协程时，Kotlin 编译器会报告一个警告。
如果要关闭该警告，可以将以下块添加到你的 `build.gradle` 文件中：

``` groovy
kotlin {
    experimental {
        coroutines 'enable'
    }
}
```

或者，对于 Gradle Kotlin DSL：

``` kotlin
import org.jetbrains.kotlin.gradle.dsl.Coroutines
// ……

kotlin.experimental.coroutines = Coroutines.ENABLE
```

## 模块名称

构建生成的 Kotlin 模块会按照该项目的 `archivesBaseName` 属性命名。 如果一个项目具有宽泛的名称如 `lib` 或者 `jvm`——这在子项目中很常见，与该模块相关的 Kotlin 输出文件（`*.kotlin_module`）可能会与来自第三方的同名模块发生冲突。 当项目打包成单一归档（例如 APK）时这会出问题。

为了避免这种情况，请考虑手动设置唯一的 `archivesBaseName`：

``` groovy
archivesBaseName = 'myExampleProject_lib'
```

对于 Gradle Kotlin DSL，要这样：

``` kotlin
setProperty("archivesBaseName", "myExampleProject_lib")
```

## Gradle 构建缓存支持（自 1.2.20 起）

Kotlin 插件支持 [Gradle 构建缓存](https://guides.gradle.org/using-build-cache/)（需要 Gradle 4.3 及以上版本；低版本则禁用缓存）。

由于注解处理器运行的任意代码可能不一定会将任务输入转换为输出、可能访问与修改 Gradle 未跟踪的文件等，因此默认不缓存 kapt 注解处理任务。要启用 kapt 缓存，请将以下列几行添加到构建脚本中：

``` groovy
kapt {
    useBuildCache = true
}
```

要禁用所有 Kotlin 任务的缓存，请将系统属性标志 `kotlin.caching.enabled` 设置为 `false`（运行构建带上参数 `-Dkotlin.caching.enabled=false`）。

## 编译器选项

要指定附加的编译选项，请使用 Kotlin 编译任务的 `kotlinOptions` 属性。

当针对 JVM 时，对于生产代码这些任务称为 `compileKotlin` 而对于测试代码称为 `compileTestKotlin`。对于自定义源文件集（source set）这些任务称呼取决于 `compile＜Name＞Kotlin` 模式。

Android 项目中的任务名称包含[构建变体](https://developer.android.com/studio/build/build-variants.html) 名称，并遵循 `compile<BuildVariant>Kotlin` 的模式，例如 `compileDebugKotlin`、 `compileReleaseUnitTestKotlin`。

当针对 JavaScript 时，这些任务分别称为 `compileKotlin2Js` 与 `compileTestKotlin2Js`，以及对于自定义源文件集称为 `compile＜Name＞Kotlin2Js`。

要配置单个任务，请使用其名称。示例：

``` groovy
compileKotlin {
    kotlinOptions.suppressWarnings = true
}

compileKotlin {
    kotlinOptions {
        suppressWarnings = true
    }
}
```

对于 Gradle Kotlin DSL，首先从项目的 `tasks` 中获取任务：

``` kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile
// ……

val kotlinCompile: KotlinCompile by tasks

kotlinCompile.kotlinOptions.suppressWarnings = true
```

相应地，为 JS 与 Common 目标使用类型 `Kotlin2JsCompile` 与 `KotlinCommonCompile`。

也可以在项目中配置所有 Kotlin 编译任务：

``` groovy
tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).all {
    kotlinOptions {
        // ……
    }
}
```

对于 Gradle 任务的完整选项列表如下：

### JVM、JS 与 JS DCE 的公共属性

| 名称                  | 描述                 | 可能的值 | 默认值 |
| --------------------- | -------------------- | -------- | ------ |
| `allWarningsAsErrors` | 任何警告都报告为错误 |          | false  |
| `suppressWarnings`    | 不生成警告           |          | false  |
| `verbose`             | 启用详细日志输出     |          | false  |
| `freeCompilerArgs`    | 附加编译器参数的列表 |          | []     |

### JVM 与 JS 的公共属性

| Name              | Description                            | Possible values                           | Default value |
| ----------------- | -------------------------------------- | ----------------------------------------- | ------------- |
| `apiVersion`      | 只允许使用来自捆绑库的指定版本中的声明 | "1.0", "1.1", "1.2", "1.3 (EXPERIMENTAL)" |               |
| `languageVersion` | 提供与指定语言版本源代码兼容性         | "1.0", "1.1", "1.2", "1.3 (EXPERIMENTAL)" |               |

### JVM 特有的属性

| 名称             | 描述                                                                  | 可能的值      | 默认值 |
| ---------------- | --------------------------------------------------------------------- | ------------- | ------ |
| `javaParameters` | 为方法参数生成 Java 1.8 反射的元数据                                  |               | false  |
| `jdkHome`        | 要包含到 classpath 中的 JDK 主目录路径，如果与默认 JAVA_HOME 不同的话 |               |        |
| `jvmTarget`      | 生成的 JVM 字节码的目标版本（1.6 或 1.8），默认为 1.6                 | "1.6"、 "1.8" | "1.6"  |
| `noJdk`          | 不要在 classpath 中包含 Java 运行时                                   |               | false  |
| `noReflect`      | 不要在 classpath 中包含 Kotlin 反射实现                               |               | true   |
| `noStdlib`       | 不要在 classpath 中包含 Kotlin 运行时                                 |               | true   |

### JS 特有的属性

| 名称                    | 描述                                              | 可能的值                           | 默认值  |
| ----------------------- | ------------------------------------------------ | --------------------------------- |------ |
| `friendModulesDisabled` | 禁用内部声明导出                                    |                                   | false |
| `main`                  | 是否要调用 main 函数                               | "call"、 "noCall"                  | "call"|
| `metaInfo`              | 使用元数据生成 .meta.js 与 .kjsm 文件。用于创建库     |                                    | true |
| `moduleKind`            | 编译器生成的模块类型                              | "plain"、 "amd"、 "commonjs"、 "umd" | "plain"|
| `noStdlib`              | 不使用捆绑的 Kotlin stdlib                        |                                      | true |
| `outputFile`            | 输出文件路径                                      |                                      |      |
| `sourceMap`             | 生成源代码映射（source map）                      |                                      | false |
| `sourceMapEmbedSources` | 将源代码嵌入到源代码映射中                        | "never"、 "always"、 "inlining"      |         |
| `sourceMapPrefix`       | 源代码映射中路径的前缀                            |                                      |       |
| `target`                | 生成指定 ECMA 版本的 JS 文件                      | "v5"                                 | "v5"  |
| `typedArrays`           | 将原生数组转换为 JS 带类型数组                    |                                      | true   |


## 生成文档

要生成 Kotlin 项目的文档，请使用 [Dokka](https://github.com/Kotlin/dokka)；相关配置说明请参见 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md#using-the-maven-plugin)。
Dokka 支持混合语言项目，并且可以生成多种格式的输出，包括标准 JavaDoc。

## OSGi

关于 OSGi 支持请参见 [Kotlin OSGi 页](kotlin-osgi.html)。

## 使用 Gradle Kotlin DSL

使用 [Gradle Kotlin DSL](https://github.com/gradle/kotlin-dsl) 时，请使用 `plugins { …… }` 块应用 Kotlin 插件。如果使用 `apply { plugin(……) }` 来应用的话，可能会遇到未解析的到由 Gradle Kotlin DSL 所生成扩展的引用问题。为了解决这个问题，可以注释掉出错的用法，运行 Gradle 任务 `kotlinDslAccessorsSnapshot`，然后解除该用法注释并重新运行构建或者重新将项目导入到 IDE 中。

## 示例

以下示例显示了配置 Gradle 插件的不同可能性：

* [Kotlin](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/hello-world)
* [混用 Java 与 Kotlin](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/mixed-java-kotlin-hello-world)
* [Android](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-mixed-java-kotlin-project)
* [JavaScript](https://github.com/JetBrains/kotlin/tree/master/libraries/tools/kotlin-gradle-plugin-integration-tests/src/test/resources/testProject/kotlin2JsProject)