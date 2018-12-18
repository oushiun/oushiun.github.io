---
title: 使用 Maven

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 工具
date: 2018-05-25 10:02:55
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

## 插件与版本

*kotlin-maven-plugin* 用于编译 Kotlin 源代码与模块，目前只支持 Maven V3。

通过 *kotlin.version* 属性定义要使用的 Kotlin 版本：

``` xml
<properties>
    <kotlin.version>{{ site.data.releases.latest.version }}</kotlin.version>
</properties>
```

<!-- more -->

## 依赖

Kotlin 有一个广泛的标准库可用于应用程序。在 pom 文件中配置以下依赖关系：

``` xml
<dependencies>
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-stdlib</artifactId>
        <version>${kotlin.version}</version>
    </dependency>
</dependencies>
```

如果你是针对 JDK 7 或 JDK 8，那么可以使用扩展版本的 Kotlin 标准库，其中包含为新版 JDK 所增 API 而加的额外的扩展函数。使用 `kotlin-stdlib-jdk7` 或 `kotlin-stdlib-jdk8` 取代 `kotlin-stdlib`，这取决于你的 JDK 版本（对于 Kotlin 1.1.x 用 `kotlin-stdlib-jre7` 与 `kotlin-stdlib-jre8`)。

如果你的项目使用 [Kotlin 反射](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect.full/index.html) 或者测试设施，那么你还需要添加相应的依赖项。其构件 ID 对于反射库是 `kotlin-reflect`，对于测试库是 `kotlin-test` 与 `kotlin-test-junit`。

## 编译只有 Kotlin 的源代码

要编译源代码，请在 `<build>` 标签中指定源代码目录：

``` xml
<build>
    <sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
</build>
```

需要引用 Kotlin Maven 插件来编译源代码：

``` xml
<build>
    <plugins>
        <plugin>
            <artifactId>kotlin-maven-plugin</artifactId>
            <groupId>org.jetbrains.kotlin</groupId>
            <version>${kotlin.version}</version>

            <executions>
                <execution>
                    <id>compile</id>
                    <goals> <goal>compile</goal> </goals>
                </execution>

                <execution>
                    <id>test-compile</id>
                    <goals> <goal>test-compile</goal> </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 同时编译 Kotlin 与 Java 源代码

要编译混合代码应用程序，必须在 Java 编译器之前调用 Kotlin 编译器。
按照 maven 的方式，这意味着应该使用以下方法在 maven-compiler-plugin 之前运行 kotlin-maven-plugin，确保 pom.xml 文件中的 kotlin 插件位于 maven-compiler-plugin 上面：

``` xml
<build>
    <plugins>
        <plugin>
            <artifactId>kotlin-maven-plugin</artifactId>
            <groupId>org.jetbrains.kotlin</groupId>
            <version>${kotlin.version}</version>
            <executions>
                <execution>
                    <id>compile</id>
                    <goals> <goal>compile</goal> </goals>
                    <configuration>
                        <sourceDirs>
                            <sourceDir>${project.basedir}/src/main/kotlin</sourceDir>
                            <sourceDir>${project.basedir}/src/main/java</sourceDir>
                        </sourceDirs>
                    </configuration>
                </execution>
                <execution>
                    <id>test-compile</id>
                    <goals> <goal>test-compile</goal> </goals>
                    <configuration>
                        <sourceDirs>
                            <sourceDir>${project.basedir}/src/test/kotlin</sourceDir>
                            <sourceDir>${project.basedir}/src/test/java</sourceDir>
                        </sourceDirs>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <executions>
                <!-- 替换会被 maven 特别处理的 default-compile -->
                <execution>
                    <id>default-compile</id>
                    <phase>none</phase>
                </execution>
                <!-- 替换会被 maven 特别处理的 default-testCompile -->
                <execution>
                    <id>default-testCompile</id>
                    <phase>none</phase>
                </execution>
                <execution>
                    <id>java-compile</id>
                    <phase>compile</phase>
                    <goals> <goal>compile</goal> </goals>
                </execution>
                <execution>
                    <id>java-test-compile</id>
                    <phase>test-compile</phase>
                    <goals> <goal>testCompile</goal> </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 增量编译

为了使构建更快，可以为 Maven 启用增量编译（从 Kotlin 1.1.2 起支持）。
为了做到这一点，需要定义 `kotlin.compiler.incremental` 属性：

``` xml
<properties>
    <kotlin.compiler.incremental>true</kotlin.compiler.incremental>
</properties>
```

或者，使用 `-Dkotlin.compiler.incremental=true` 选项运行构建。

## 注解处理

请参见 [Kotlin 注解处理工具](kapt.html)（`kapt`）的描述。

## 协程支持

在 Kotlin 1.2 中[协程](coroutines.html)支持是一项实验性的功能，因此当你在项目中使用协程时 Kotlin 编译器会报警告。
可以将以下代码块添加到 `pom.xml` 文件中来关闭这一警告：

``` xml
<configuration>
    <experimentalCoroutines>enable</experimentalCoroutines>
</configuration>
```

## Jar 文件

要创建一个仅包含模块代码的小型 Jar 文件，请在 Maven pom.xml 文件中的 `build->plugins` 下面包含以下内容，其中 `main.class` 定义为一个属性，并指向主 Kotlin 或 Java 类：

``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>${main.class}</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

## 独立的 Jar 文件

要创建一个独立的（self-contained）Jar 文件，包含模块中的代码及其依赖项，请在 Maven pom.xml 文件中的 `build->plugins` 下面包含以下内容其中 `main.class` 定义为一个属性，并指向主 Kotlin 或 Java 类：

``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals> <goal>single</goal> </goals>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>${main.class}</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这个独立的 jar 文件可以直接传给 JRE 来运行应用程序：

``` bash
java -jar target/mymodule-0.0.1-SNAPSHOT-jar-with-dependencies.jar
```

## 针对 JavaScript

为了编译 JavaScript 代码，需要使用 `js` 和 `test-js` 目标来执行 `compile`：

``` xml
<plugin>
    <groupId>org.jetbrains.kotlin</groupId>
    <artifactId>kotlin-maven-plugin</artifactId>
    <version>${kotlin.version}</version>
    <executions>
        <execution>
            <id>compile</id>
            <phase>compile</phase>
            <goals>
                <goal>js</goal>
            </goals>
        </execution>
        <execution>
            <id>test-compile</id>
            <phase>test-compile</phase>
            <goals>
                <goal>test-js</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

你还需要更改标准库依赖：

``` xml
<groupId>org.jetbrains.kotlin</groupId>
<artifactId>kotlin-stdlib-js</artifactId>
<version>${kotlin.version}</version>
```

对于单元测试支持，你还需要添加对 `kotlin-test-js` 构件的依赖。

更多信息请参阅[以 Maven 入门使用 Kotlin 与 JavaScript](/docs/tutorials/javascript/getting-started-maven/getting-started-with-maven.html)教程。

## 指定编译器选项

可以将额外的编译器选项与参数指定为 Maven 插件节点的 `<configuration>` 元素下的标签：

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>
    <executions>……</executions>
    <configuration>
        <nowarn>true</nowarn>  <!-- 禁用警告 -->
        <args>
            <arg>-Xjsr305=strict</arg> <!-- 对 JSR-305 注解启用严格模式 -->
            ...
        </args>
    </configuration>
</plugin>
```

许多选项还可以通过属性来配置：

``` xml
<project ……>
    <properties>
        <kotlin.compiler.languageVersion>1.0</kotlin.compiler.languageVersion>
    </properties>
</project>
```

支持以下属性：

### JVM 和 JS 的公共属性

| 名称            | 属性名                          | 描述                                    | 可能的值                                    | 默认值             |
| --------------- | ------------------------------- | --------------------------------------- | ------------------------------------------- | ------------------ |
| nowarn          |                                 | 不生成警告                              | true、 false                                | false              |
| languageVersion | kotlin.compiler.languageVersion | 提供与指定语言版本源代码兼容性          | "1.0"、 "1.1"、 "1.2"、"1.3 (EXPERIMENTAL)" |
| apiVersion      | kotlin.compiler.apiVersion      | 只允许使用来自捆绑库的指定版本中的声明  | "1.0"、 "1.1"、 "1.2"、"1.3 (EXPERIMENTAL)" |
| sourceDirs      |                                 | 包含要编译源文件的目录                  |                                             | 该项目源代码根目录 |
| compilerPlugins |                                 | 启用[编译器插件](compiler-plugins.html) |                                             | []                 |
| pluginOptions   |                                 | 编译器插件的选项                        |                                             | []                 |
| args            |                                 | 额外的编译器参数                        |                                             | []                 |


### JVM 特有的属性

| 名称      | 属性名                    | 描述                                                                  | 可能的值      | 默认值 |
| --------- | ------------------------- | --------------------------------------------------------------------- | ------------- | ------ |
| jvmTarget | kotlin.compiler.jvmTarget | 生成的 JVM 字节码的目标版本                                           | "1.6"、 "1.8" | "1.6"  |
| jdkHome   | kotlin.compiler.jdkHome   | 要包含到 classpath 中的 JDK 主目录路径，如果与默认 JAVA_HOME 不同的话 |               | &nbsp; |

### JS 特有的属性

| 名称                  | 属性名 | 描述                                              | 可能的值                             | 默认值     |
| --------------------- | ------ | ------------------------------------------------- | ------------------------------------ | ---------- |
| outputFile            |        | 输出文件路径                                      |                                      |            |
| metaInfo              |        | 使用元数据生成 .meta.js 与 .kjsm 文件。用于创建库 | true、 false                         | true       |
| sourceMap             |        | 生成源代码映射（source map）                      | true、 false                         | false      |
| sourceMapEmbedSources |        | 将源代码嵌入到源代码映射中                        | "never"、 "always"、 "inlining"      | "inlining" |
| sourceMapPrefix       |        | 源代码映射中路径的前缀                            |                                      |            |
| moduleKind            |        | 编译器生成的模块类型                              | "plain"、 "amd"、 "commonjs"、 "umd" | "plain"    |

## 生成文档

标准的 JavaDoc 生成插件（`maven-javadoc-plugin`）不支持 Kotlin 代码。
要生成 Kotlin 项目的文档，请使用 [Dokka](https://github.com/Kotlin/dokka)；
相关配置说明请参见 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md#using-the-maven-plugin)。
Dokka 支持混合语言项目，并且可以生成多种格式的输出，包括标准 JavaDoc。

## OSGi

对于 OSGi 支持，请参见 [Kotlin OSGi 页](kotlin-osgi.html)。

## 示例

一个示例 Maven 项目可以[从 Github 版本库直接下载](https://github.com/JetBrains/kotlin-examples/archive/master/maven.zip)