# **第二十四章. Groovy 插件**

Groovy 的插件继承自 Java 插件并添加了对 Groovy 项目的支持。它可以处理 Groovy 代码，以及混合的 Groovy 和 Java 代码，甚至是纯 Java 代码（尽管我们不一定推荐使用）。该插件支持联合编译，可以任意地混合及匹配Groovy 和 Java 代码各自的依赖。例如，一个 Groovy 类可以继承自一个 Java 类，而这个 Java 类也可以继承自一个 Groovy 类。这样一来，我们就能够在项目中使用最适合的语言，并且在有需要的情况下用其他的语言重写其中的任何类。

## **24.1. 用法**
要使用 Groovy 的插件，请在构建脚本中包含以下语句：

示例 24.1. 使用 Groovy 插件

build.gradle
```
apply plugin: 'groovy'
```

## **24.2. 任务**
Groovy 的插件向project 中添加了以下任务。

表 24.1. Groovy 插件 - 任务

|任务名称|	依赖于|	类型	|描述|
|--
|compileGroovy	|compileJava	|GroovyCompile	|编译production 的 Groovy 源文件。|
|compileTestGroovy|	compileTestJava	|GroovyCompile	|编译test 的 Groovy 的源文件。|
|compileSourceSetGroovy	|compileSourceSetJava	|GroovyCompile	|编译给定的source set 里的 Groovy 源文件。|
|groovydoc|	-	|Groovydoc|	为production 里的 Groovy 源文件生成 API 文档。|

Groovy 的插件向 Java 插件所加入的 tasks 添加了以下的依赖。

表 24.2. Groovy 插件 - 额外的task 依赖

|任务名称	|依赖于|
|--
|classes	|compileGroovy|
|testClasses|	compileTestGroovy|
|sourceSetClasses	|compileSourceSetGroovy|
图 23.1. Groovy 插件 - tasks
![](https://docs.gradle.org/current/userguide/img/groovyPluginTasks.png)


## **24.3. 项目布局**
Groovy 的插件会假定项目的布局如表 24.3，“Groovy 插件 - 项目布局”中所示。所有 Groovy 的源目录都可以包含 Groovy和Java 代码。Java 源目录只能包含 Java 源代码。[11]这些目录不一定得存在或是里面包含有什么内容；Groovy 的插件只会进行编译，而不管它发现什么。

表 24.3. Groovy 插件 - 项目布局

|目录	|意义|
|--
|src/main/java	|产品的Java源代码|
|src/main/resources|	产品的资源|
|src/main/groovy|	Production Groovy 源代码。此外可能包含联合编译的 Java 源代码。|
|src/test/java	|Java 测试源代码|
|src/test/resources	|测试资源|
|src/test/groovy	|Test Groovy 源代码。此外可能包含联合编译的 Java 源代码。|
|src/sourceSet/java	|给定的源集的Java源代码|
|src/sourceSet/resources|	给定的源集的资源|
|src/sourceSet/groovy	|给定的source set 的 Groovy 源代码。此外可能包含联合编译的 Java 源代码。|

### 24.3.1. 更改项目布局

和 Java 插件一样，Groovy 插件允许把 Groovy 的production 和test 的源文件配置为自定义的位置。

示例 24.2. 自定义 Groovy 自定义源文件布局

build.gradle
```
sourceSets {
    main {
        groovy {
            srcDirs = ['src/groovy']
        }
    }

    test {
        groovy {
            srcDirs = ['test/groovy']
        }
    }
}
```

## 24.4. 依赖管理
由于 Gradle 的构建语言基于 Groovy 的，且部分的 Groovy 使用 Groovy 实现，因此 Gradle 已经带有了一个 Groovy 库 （截至 Gradle 1.6所带的 Groovy 库的版本是 1.8.6）。然而，Groovy 项目需要显式地声明一个 Groovy 依赖。这个依赖会在编译和运行的类路径时用到。它还将用于分别获取Groovy 编译器及 Groovydoc 工具。

如果 Groovy 用于production 代码，Groovy 依赖应该添加到compile配置中：

示例 24.3. Groovy 的依赖配置

build.gradle
```
repositories {
    mavenCentral()
}

dependencies {
    compile 'org.codehaus.groovy:groovy-all:2.2.0'
}
```
如果 Groovy 仅用于测试代码，Groovy 的依赖应该被添加到testCompile配置中：

示例 24.4. 配置Groovy 测试依赖

build.gradle
```
dependencies {
    testCompile "org.codehaus.groovy:groovy-all:2.2.0"
}
```
如果要使用 Gradle 所带的 Groovy 库，请声明localGroovy()依赖。注意，不同 Gradle 版本附带的 Groovy 版本不同；因此，声明一个固定的 Groovy 依赖要比使用localGroovy()更安全一些。

示例 24.5. 配置捆绑的 Groovy 依赖

build.gradle
```
dependencies {
    compile localGroovy()
}
```
Groovy 库不一定得从远程仓库中获取。它也可以获取自本地中可能检入版本控制的lib目录：

示例 24.6. 配置Groovy 文件依赖

build.gradle
```
repositories {
    flatDir { dirs 'lib' }
}

dependencies {
    compile module('org.codehaus.groovy:groovy:1.6.0') {
        dependency('asm:asm-all:2.2.3')
        dependency('antlr:antlr:2.7.7')
        dependency('commons-cli:commons-cli:1.2')
        module('org.apache.ant:ant:1.9.3') {
            dependencies('org.apache.ant:ant-junit:1.9.3@jar', 'org.apache.ant:ant-launcher:1.9.3')
        }
    }
}
```

## **24.5. groovyClasspath 的自动配置**

GroovyCompile和Groovydoc tasks 会以两种方式使用 Groovy： 在它们的classpath以及它们的groovyClasspath上。前者用于在源代码中查找类的引用，通常会包含 Groovy 库和其他库。后者用来分别加载和执行 Groovy 编译器和 Groovydoc 工具，并且应该只包含 Groovy 库及其依赖项。

除非显式配置了一个 task 的groovyClasspath ，否则 Groovy（基础）插件会尝试推断该 task 的classpath。以如下方式进行：

如果在在classpath中找到groovy-all(-indy) Jar，相同的 Jar 将添加到groovyClasspath中。
如果在classpath中找到groovy(-indy) Jar ，并且该项目已经在至少一个仓库中声明了它，那么相应的groovy(-indy)的仓库依赖将添加到groovyClasspath中。
其他情况，该task 将执行失败，并提示无法推断 groovyClasspath。

## **24.6. 常规属性**
Groovy 的插件没有向 project 添加任何的常规属性。

## **24.7. source set 属性**
Groovy 的插件向 project 的每一个source set 添加了下列的常规属性。你可以在你的构建脚本中，把这些属性当成是source set 对象中的属性一样使用 （见第 21.3，“公约”）。

表 24.4. Groovy 插件 - source set 属性

|属性名称	|类型	|默认值|	描述|
|--
|groovy	|SourceDirectorySet (read-only)	|非空|	该source set 中的 Groovy 源文件。包含在 Groovy 源目录中找到的所有的.groovy和.java文件，并排除所有其他类型的文件。|
|groovy.srcDirs	|Set<File>. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一个来设置。|	[projectDir/src/name/groovy]	|源目录包含该 source set 中的 Groovy 源文件。此外可能还包含用于联合编译的 Java 源文件。|
|allGroovy	|FileTree (read-only)	|非空	|该source set 中的所有 Groovy 源文件。包含在 Groovy 源目录中找到的所有的.groovy文件。|

这些属性由一个GroovySourceSet 的约定对象提供。

Groovy 的插件还修改了一些 source set 的属性：

表 24.5. Groovy 的插件 - source set 属性

|属性名称	|修改的内容|
|--
|allJava	|添加在 Groovy 源目录中找到的所有.java文件。|
|allSource	|添加在 Groovy 的源目录中找到的所有源文件。|

## **24.8. GroovyCompile**
Java 插件向 project 里的每个source set 添加了一个JavaCompile task。这个task 的类型继承自JavaCompile task（见第 23.11 节，“CompileJava”）。除非groovyOptions.useAnt设置为true，否则将使用 Gradle 集成的本地的 Groovy 编译器。对于大多数项目而言，这相比基于 Ant 编译器，是个更好的选择。GroovyCompile task 支持官方的 Groovy 编译器的大多数配置选项。

表 24.6. Groovy 插件 - GroovyCompile 属性

|任务属性	|类型	|默认值|
|--
|classpath|	FileCollection|	sourceSet.compileClasspath|
|source	|FileTree. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一种方式来设置。|	sourceSet.groovy|
|destinationDir	|File.|	sourceSet.output.classesDir|
|groovyClasspath	|FileCollection	|如果groovy 配置不为空，则为该配置；否则为 classpath 中找到的 Groovy 库|

[11] 我们使用和 Russel Winder 的 Gant 工具同样的约定（http://gant.codehaus.org）。

百度搜索[无线学院](http://wirelesscollege.cn)

