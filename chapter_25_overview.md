# **第二十五chapter章. Scala 插件**

Scala 的插件继承自 Java 插件并添加了对 Scala 项目的支持。它可以处理 Scala 代码，以及混合的 Scala 和 Java 代码，甚至是纯 Java 代码（尽管我们不一定推荐使用）。该插件支持联合编译，联合编译可以通过 Scala 及 Java 的各自的依赖任意地混合及匹配它们的代码。例如，一个 Scala 类可以继承自一个 Java 类，而这个 Java 类也可以继承自一个 Scala 类。这样一来，我们就能够在项目中使用最适合的语言，并且在有需要的情况下用其他的语言重写其中的任何类。

## **25.1. 用法**
要使用 Scala 插件，请在构建脚本中包含以下语句：

示例 25.1. 使用 Scala 插件

build.gradle
```
apply plugin: 'scala'
```

## **25.2. 任务**
Scala 的插件向project 中添加了以下任务。

表 25.1. Scala 插件 - 任务

|任务名称|	依赖于|	类型|	描述|
|--
|compileScala|	compileJava	|ScalaCompile|	编译production 的 Scala 源文件。|
|compileTestScala	|compileTestJava	|ScalaCompile	|编译test 的 Scala 的源文件。|
|compileSourceSetScala|	compileSourceSetJava|	ScalaCompile	|编译给定的source set 里的 Scala 源文件。|
|scaladoc|	-	|scaladoc|	为production 里的 Scala 源文件生成 API 文档。|

Scala 插件向 Java 插件所加入的 tasks 添加了以下的依赖。

表 25.2. Scala感觉 插件 - 额外的task 依赖

|任务名称	|依赖于|
|--
|classes	|compileScala|
|testClasses	|compileTestScala|
|sourceSetClasses	|compileSourceSetScala|
图 25.1. Scala 插件-任务
![](https://docs.gradle.org/current/userguide/img/scalaPluginTasks.png)

## **25.3. 项目布局**
Scala 插件会假定如下所示的项目布局。所有 Scala 的源目录都可以包含 Scala<s1><e2>和</e2></s1>Java 代码。Java 源目录只能包含 Java 源代码。这些目录不一定是存在的，或是里面包含有什么内容；Scala 插件只会进行编译，而不管它发现什么。

表 25.3. Scala 插件 - 项目布局

|目录	|意义|
|**
|src/main/java	|产品的Java源代码|
|src/main/resources|	产品的资源|
|src/main/scala|	Production Scala 源代码。此外可能包含联合编译的 Java 源代码。|
|src/test/java	|Java 测试源代码|
|src/test/resources	|测试资源|
|src/test/scala	|Test Scala 源代码。此外可能包含联合编译的 Java 源代码。|
|src/sourceSet/java	|给定的源集的Java源代码|
|src/sourceSet/resources	|给定的源集的资源|
|src/sourceSet/scala	|给定的source set 的 Scala 源代码。此外可能包含联合编译的 Java 源代码。|

### 25.3.1. 更改项目布局

和 Java 插件一样，Scala 插件允许把 Scala 的production 和test 的源文件配置为自定义的位置。

示例 25.2. 自定义 Scala 源文件布局

build.gradle
```
sourceSets {
    main {
        scala
            srcDirs = ['src/scala']
        }
    }
    test {
        scala
            srcDirs = ['test/scala']
        }
    }
}
```

## 25.4. 依赖管理
Scala 项目需要声明一个scala-library依赖项。这个依赖会在编译和运行的类路径时用到。它还将用于分别获取Scala 编译器及 Scaladoc 工具。[12]

如果 Scala 用于production 代码， scala-library 依赖应该添加到compile的配置中：

示例 25.3. 为production 代码定义一个Scala 依赖

build.gradle
```
repositories {
    mavenCentral()
}

dependencies {
    compile 'org.scala-lang:scala-library:2.9.1'
}
```
如果 Scala 仅用于测试代码， scala-library依赖应被添加到testCompile配置中：

示例 25.4. 为 test 代码定义一个Scala 依赖

build.gradle
```
dependencies {
    testCompile "org.scala-lang:scala-library:2.9.2"
}
```

## **25.5. scalaClasspath 的自动配置**
ScalaCompile和ScalaDoc tasks 会以两种方式使用 Scala： 在它们的classpath以及scalaClasspath上。前者用于在源代码中查找类的引用，通常会包含 scala-library和其他库。后者用来分别加载和执行 Scala 编译器和 Scala 工具，并且应该只包含 scala-library及其依赖项。

除非显式配置了一个 task 的scalaClasspath ，否则 Scala（基础）插件会尝试推断该 task 的classpath。以如下方式进行：

如果在classpath中找到scala-library Jar ，并且该项目已经在至少一个仓库中声明了它，那么相应的scala-compiler的仓库依赖将添加到scalaClasspath中。
其他情况，该task 将执行失败，并提示无法推断 scalaClasspath。

## **25.6. 公约属性**
Scala 插件没有向 project 添加任何的公约属性。

## **25.7. source set 属性**
Scala 的插件向 project 的每一个source set 添加了下列的公约属性。你可以在你的构建脚本中，把这些属性当成是source set 对象中的属性一样使用 （见第 21.3，“公约”）。

表 25.4. Scala 插件 - source set 属性

|属性名称	|类型|	默认值|	描述|
|--
|scala|	SourceDirectorySet (read-only)|	非空|	该source set 中的 Scala 源文件。包含在 Scala 源目录中找到的所有的.scala和.java文件，并排除所有其他类型的文件。|
|scala.srcDirs|	Set<File>. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一个来设置。|	[projectDir/src/name/scala]	|源目录包含该 source set 中的 Scala 源文件。此外可能还包含用于联合编译的 Java 源文件。|
|allScala	|FileTree (read-only)|	非空|	该source set 中的所有 Scala 源文件。包含在 Scala 源目录中找到的所有的.scala文件。|

这些属性由一个ScalaSourceSet 的约定对象提供。

Scala 的插件还修改了一些 source set 的属性：

表 25.5. Scala 插件 - source set 属性

|属性名称|	修改的内容|
|--
|allJava	|添加在 Scala 源目录中找到的所有.java文件。|
|allSource	|添加在 Scala 的源目录中找到的所有源文件。|

## **25.8. Fast Scala Compiler**
Scala 插件包含了对fsc，即 Fast Scala Compiler 的支持。fsc运行在一个单独的进程中，并且可以显著地提高编译速度。

示例 25.5. 启用 Fast Scala Compiler

build.gradle
```
compileScala
    scalaCompileOptions.useCompileDaemon = true

    // optionally specify host and port of the daemon:
    scalaCompileOptions.daemonServer = "localhost:4243"
}
```
注意，每当 fsc的编译类路径的内容发生变化时，它都需要重新启动。(它本身不会去检测编译类路径的更改。）这使得它不太适合于多项目的构建。

## **25.9. 在外部进程中编译**
当scalaCompileOptions.fork设置为true时，编译会在外部进程中进行。fork 的详细情况依赖于所使用的编译器。基于 Ant 的编译器 (scalaCompileOptions.useAnt = true) 将为每个ScalaCompile任务fork 一个新进程，而默认情况下它不进行fork。基于 Zinc 的编译器 (scalaCompileOptions.useAnt = false) 将利用 Gradle 编译器守护进程，且默认情况下也是这样。

外部过程默认使用JVM 的的默认内存设置。如果要调整内存设置，请根据需要配置scalaCompileOptions.forkOptions ：

示例 25.6. 调整内存设置

build.gradle
```
tasks.withType(ScalaCompile) {
    configure(scalaCompileOptions.forkOptions) {
        memoryMaximumSize = '1g'
        jvmArgs = ['-XX:MaxPermSize=512m']
    }
}
```

## **25.10. 增量编译**
增量编译是只编译那些源代码在上一次编译之后有修改的类，及那些受这些修改影响到的类，它可以大大减少 Scala 的编译时间。频繁编译代码的增量部分是非常有用的，因为在开发时我们经常要这样做。

Scala 插件现在通过集成 Zinc 来支持增量编译， 它是sbt增量 Scala 编译器的一个单机版本。若要把ScalaCompile任务从默认的基于 Ant 的编译器切换为新的基于 Zinc 的编译器，需要将scalaCompileOptions.useAnt设置为false：

示例 25.7. 激活基于 Zinc 编译器

build.gradle
```
tasks.withType(ScalaCompile) {
    scalaCompileOptions.useAnt = false
}
```
除非在API 文档中另有说明，否则基于 Zinc 的据编译器支持与基于 Ant 的编译器完全相同的配置选项。但是，要注意的是，Zinc 编译器需要 Java 6 或其以上版本来运行。这意味着 Gradle 本身要使用 Java 6 或其以上版本。

Scala 插件添加了一个名为zinc的配置，以解析 Zinc 库及其依赖。如果要重写 Gradle 默认情况下使用的 Zinc 版本，请添加一个显式的 Zinc 依赖项 （例如zinc "com.typesafe.zinc:zinc:0.1.4"）。无论使用哪一个 Zinc 版本，Zinc 都是使用在scalaTools配置上找到的 Scala 编译器。

就像Gradle 上基于Ant 的编译器一样，基于 Zinc 的编译器支持 Java 和 Scala 代码的联合编译。默认情况下，在src/main/scala 下的所有 Java 和 Scala 代码都会进行联合编译。使用基于 Zinc 的编译器时，即使是 Java 代码也将会进行增量编译。

增量编译需要源代码的相关性分析。解析结果进入由 scalaCompileOptions.incrementalOptions.analysisFile 所指定的文件（它有一个合理的默认值）。在多项目构建中，分析文件被传递给下游的ScalaCompile任务，以启用跨项目的增量编译。对于由 Scala 插件添加的ScalaCompile任务，无需对这一点进行配置。对于其他的 ScalaCompile 任务，需要根据类文件夹或 Jar archive 的代码中，是哪一个的代码被传递给ScalaCompile任务的下游类路径，把ScalaCompileOptions.incrementalOptions.publishedCode配置为指向它们。注意，如果publishedCode设置不正确，上游代码发生变化时下游任务可能不会重新编译代码，导致编译结果不正确。

由于依赖分析的系统开销，一次干净的编译或在代码有了较大的更改之后的编译，可能花费的时间要长于使用基于 Ant 的编译器。对于 CI 构建和版本的构建中，我们目前推荐使用基于 Ant 的编译器。

注意现在 Zinc 基于守护进程模式的Nailgun 还不支持。相反，我们打算加强 Gradle 自己的编译器守护进程，使得在跨 Gradle 调用时继续存活，利用同一个 Scala 编译器。这将会为 Scala 编译带来另一个方面上的明显加速。

## **25.11. eclipse 集成**
当 Eclipse 插件遇到 Scala 项目时，它将添加额外的配置，使得项目能够在使用 Scala IDE 时开箱即用。具体而言，该插件添加一个 Scala 性质和依赖的容器。

## **25.12. IntelliJ 集成**
当 IDEA 插件遇到 Scala 项目时，它将添加额外的配置，使得项目能够在使用 IDEA 时开箱即用。具体而言，该插件添加了一个 Scala facet 和一个匹配项目的类路径上的 Scala 版本的 Scala 编译器类库。

百度搜索[无线学院](http://wirelesscollege.cn)
