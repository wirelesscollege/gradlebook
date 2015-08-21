# **第二十三章. Java 插件**
Java 插件向一个项目添加了 Java 编译、 测试和 bundling 的能力。它是很多其他 Gradle 插件的基础服务。

## **23.1. 用法**
要使用 Java 插件，请在构建脚本中加入：

示例 23.1. 使用 Java 插件

build.gradle
```
apply plugin: 'java'
```
## **23.2 源集**
Java 插件引入了一个源集的概念。一个源集只是一组用于编译并一起执行的源文件。这些源文件可能包括 Java 源代码文件和资源文件。其他有一些插件添加了在源集里包含 Groovy 和 Scala 的源代码文件的能力。一个源集有一个相关联的编译类路径和运行时类路径。

源集的一个用途是，把源文件进行逻辑上的分组，以描述它们的目的。例如，你可能会使用一个源集来定义一个集成测试套件，或者你可能会使用单独的源集来定义你的项目的 API 和实现类。

Java 插件定义了两个标准的源集，分别是main和test。main源集包含你产品的源代码，它们将被编译并组装成一个 JAR 文件。test源集包含你的单元测试的源代码，它们将被编译并使用 JUnit 或 TestNG来执行。

## **23.3. 任务**
Java 插件向你的项目添加了大量的任务，如下所示。

表 23.1. Java 插件-任务

|任务名称	|依赖于	|类型	|描述|
|--
|compileJava	|产生编译类路径中的所有任务。这包括了用于编译配置中包含的项目依赖关系的jar任务。|	JavaCompile	|使用 javac 编译产品中的 Java 源文件。|
|processResources	|-|	Copy	|把生产资源文件拷贝到生产的类目录中。|
|classes	|compileJava和processResources。一些插件添加了额外的编译任务。|	Task|	组装生产的类目录。|
|compileTestJava	|compile，再加上所有能产生测试编译类路径的任务。	|JavaCompile	|使用 javac 编译 Java 的测试源文件。|
|processTestResources	|-	|Copy	|把测试的资源文件拷贝到测试的类目录中。|
|testClasses	|compileTestJava和processTestResources。一些插件添加了额外的测试编译任务。|	Task	|组装测试的类目录。|
|jar	|compile	|Jar	|组装 JAR 文件|
|javadoc	|compile	|Javadoc	|使用 Javadoc 生成生产的 Java 源代码的API文档|
|test	|compile， compileTest，再加上所有产生测试运行时类路径的任务。	|Test	|使用 JUnit 或 TestNG运行单元测试。|
|uploadArchives	|使用archives的配置生成构件的任务，包括jar。	|Upload	|使用archives配置上传包括 JAR 文件的构件。|
|clean	|-	|Delete	|删除项目的build目录。|
|cleanTaskName	|-	|Delete	|删除由指定的任务所产生的输出文件。例如， cleanJar将删除由jar任务中所创建的 JAR 文件，cleanTest将删除由test任务所创建的测试结果。|

对于每个你添加到该项目中的源集，Java 插件将添加以下的编译任务：

表 23.2. Java 插件-源集任务

|任务名称	|依赖于	|类型	|描述|
|--
|compileSourceSetJava|	所有产生源集编译类路径的任务。	|JavaCompile	|使用 javac 编译给定的源集中的 Java 源文件。|
|processSourceSetResources|	-	|Copy|	把给定的源集的资源文件拷贝到类目录中。|d
|sourceSetClasses	|compileSourceSetJava 和 processSourceSetResources。某些插件还为源集添加了额外的编译任务。|	Task	|组装给定源集的类目录。|

Java 插件还增加了大量的任务构成该项目的生命周期：

表 23.3. Java 插件-生命周期任务

|任务名称	|依赖于	|类型	|描述|
|--
|assemble	|项目中的所有归档项目，包括jar任务。某些插件还向项目添加额外的归档任务。	|Task	|组装项目中所有的归类文件。|
|check	|项目中的所有核查项目，包括test任务。某些插件还向项目添加额外的核查任务。|	Task|	执行项目中所有的核查任务。|
|build	|check 和 assemble|	Task	|执行项目的完事构建。|
|buildNeeded|	build任务，以及在testRuntime配置的所有项目库依赖项的build任务。|	Task	|执行项目本身及它所依赖的其他所有项目的完整构建。|
|buildDependents	|build任务，以及在testRuntime配置中对此项目有库依赖的所有项目的build任务。	|Task|	执行项目本身及依赖它的其他所有项目的完整构建。|
|buildConfigurationName	|使用配置ConfigurationName生成构件的任务。|	Task	|组装指定配置的构件。该任务由Base插件添加，并由Java插件隐式实现。|
|uploadConfigurationName|	使用配置ConfigurationName上传构件的任务。|	Upload	|组装并上传指定配置的构件。该任务由Base插件添加，并由Java插件隐式实现。|

下图显示了这些任务之间的关系。

图23.1. Java插件 ​​- 任务
![](https://docs.gradle.org/current/userguide/img/javaPluginTasks.png)

Java插件 ​​- 任务
## **23.4. 项目布局**
Java 插件会假定如下所示的项目布局。这些目录都不需要一定存在，或者是里面有什么内容。Java插件将会进行编译，不管它发现什么，并处理缺少的任何东西。

表 23.4. Java 插件-默认项目布局

|目录	|意义|
|--
|src/main/java	|产品的Java源代码|
|src/main/resources	|产品的资源|
|src/test/java	|Java 测试源代码|
|src/test/resources	|测试资源|
|src/sourceSet/java	|给定的源集的Java源代码|
|src/sourceSet/resources	|给定的源集的资源|

### 23.4.1. 更改项目布局

你可以通过配置适当的源集，来配置项目的布局。这一点将在以下各节中详细讨论。这里是如何更改main Java 和资源源目录的一个简短的例子。

示例 23.2. 自定义Java 源代码布局

build.gradle
```
sourceSets {
    main {
        java {
            srcDir 'src/java'
        }
        resources {
            srcDir 'src/resources'
        }
    }
}
```
## **23.5. 依赖管理**
Java插件向项目添加了许多依赖配置，如下图所示。它对一些任务指定了这些配置，如compileJava和test。

表23.5. Java插件 ​​- 依赖配置

|名称	|继承自	|在哪些任务中使用|	意义|
|--
|compile	|-	|compileJava|	编译时依赖|
|runtime	|compile	|-	|运行时依赖|
|testCompile|	compile	|compileTestJava|	用于编译测试的其他依赖|
|testRuntime	|runtime, testCompile|	test|	只用于运行测试的其他依赖|
|archives|	-	|uploadArchives	|由本项目生产的构件（如jar包）。|
|default	|runtime|	-	|本项目上的默认项目依赖配置。包含本项目运行时所需要的构件和依赖。|
图23.2. Java插件 ​​- 依赖配置

Java插件 ​​- 依赖配置
对于每个你添加到项目中的源集，Java插件都会添加以下的依赖配置：

![](https://docs.gradle.org/current/userguide/img/javaPluginConfigurations.png)

表23.6. Java插件 ​​- 源集依赖配置

|名称	|继承自	|在哪些任务中使用	|意义|
|--
|sourceSetCompile	|-	|compileSourceSetJava	|给定源集的编译时依赖|
|sourceSetRuntime	|sourceSetCompile|	-	|给定源集的运行时依赖|

## **23.6. 常规属性**
Java插件向项目添加了许多常规属性，如下图所示。您可以在构建脚本中使用这些属性，就像它们是project对象的属性一样（见第21.3节，“约定”）。

表23.7. Java插件 ​​- 目录属性

|属性名称	|类型	|默认值	|描述|
|--
|reportsDirName|	String	|reports|	相对于build目录的目录名称，报告将生成到此目录。|
|reportsDir|	File (read-only)	|buildDir/reportsDirName	|报告将生成到此目录。|
|testResultsDirName|	String	|test-results|	相对于build目录的目录名称，测试报告的.xml文件将生成到此目录。|
|testResultsDir	|File (read-only)|	buildDir/testResultsDirName	|测试报告的.xml文件将生成到此目录。|
|testReportDirName	|String	|tests	|相对于build目录的目录名称，测试报告将生成到此目录。|
|testReportDir	|File (read-only)|	reportsDir/testReportDirName|	测试报告生成到此目录。|
|libsDirName	|String|	libs	|相对于build目录的目录名称，类库将生成到此目录中。|
|libsDir	|File (read-only)|	buildDir/libsDirName|	类库将生成到此目录中。|
|distsDirName	|String|	distributions	|相对于build目录的目录名称，发布的文件将生成到此目录中。|
|distsDir	|File (read-only)|	buildDir/distsDirName	|要发布的文件将生成到此目录。|
|docsDirName	|String	|docs	|相对于build目录的目录名称，文档将生成到此目录。|
|docsDir	|File (read-only)|	buildDir/docsDirName|	要生成文档的目录。|
|dependencyCacheDirName	|String	|dependency-cache|	相对于build目录的目录名称，该目录用于缓存源代码的依赖信息。|
|dependencyCacheDir	|File (read-only)|	buildDir/dependencyCacheDirName	|该目录用于缓存源代码的依赖信息。|

表 23.8. Java 插件 - 其他属性

|属性名称	|类型	|默认值	|描述|
|--
|sourceSets	|SourceSetContainer (read-only)	|非空|	包含项目的源集。|
|sourceCompatibility	|JavaVersion. |可以使用字符串或数字来设置，例如"1.5"或1.5。|	当前JVM所使用的值	当编译Java源代码时所使用的Java版本兼容性。|
|targetCompatibility	|JavaVersion. |可以使用字符串或数字来设置，例如"1.5"或1.5。|	sourceCompatibility	要生成的类的 Java 版本。|
|archivesBaseName	|String	|projectName	|像JAR或ZIP文件这样的构件的basename|
|manifest	|Manifest|	一个空的清单|	要包括的所有 JAR 文件的清单。|

这些属性由JavaPluginConvention， BasePluginConvention和ReportingBasePluginConvention这些类型的常规对象提供。

## **23.7. 使用源集**
你可以使用sourceSets属性访问项目的源集。这是项目的源集的容器，它的类型是 SourceSetContainer。除此之后，还有一个sourceSets {}的脚本块，可以传入一个闭包来配置源集容器。源集容器的使用方式几乎与其他容器一样，例如tasks。

示例 23.3. 访问源集

build.gradle
```
// Various ways to access the main source set
println sourceSets.main.output.classesDir
println sourceSets['main'].output.classesDir
sourceSets {
    println main.output.classesDir
}
sourceSets {
    main {
        println output.classesDir
    }
}

// Iterate over the source sets
sourceSets.all {
    println name
}
```
要配置一个现有的源集，你只需使用上面的其中一种访问方法来设置源集的属性。这些属性将在下文中进行介绍。下面是一个配置main 的 Java 和资源目录的例子：

示例 23.4. 配置源集的源代码目录

build.gradle
```
sourceSets {
    main {
        java {
            srcDir 'src/java'
        }
        resources {
            srcDir 'src/resources'
        }
    }
}
```
### 23.7.1. 源集属性

下表列出了一些重要的源集属性。你可以在SourceSet的 API 文档中查看更多的详细信息。

表 23.9. Java 插件 - 源集属性

|属性名称	|类型	|默认值	|描述|
|--
|name	|String (read-only)	|非空	|用来确定一个源集的源集名称。|
|output	|SourceSetOutput (read-only)|	非空	|源集的输出文件，包含它编译过的类和资源。|
|output.classesDir	|File	|buildDir/classes/name	|要生成的该源集的类的目录。|
|output.resourcesDir	|File	|buildDir/resources/name	|要生成的该源集的资源的目录。|
|compileClasspath	|FileCollection	
compileSourceSet 配置。	|该类路径在编译该源集的源文件时使用。|
|runtimeClasspath	|FileCollection	|output + runtimeSourceSet 配置。	|该类路径在执行该源集的类时使用。|
|java	|SourceDirectorySet (read-only)|	非空	|该源集的Java源文件。仅包含Java源文件目录里的.java文件，并排除其他所有文件。|
|java.srcDirs	|Set<File>. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一个来设置。|	[projectDir/src/name/java]	|该源目录包含了此源集的所有Java源文件。|
|resources	|SourceDirectorySet (read-only)	|非空	|此源集的资源文件。仅包含资源文件，并且排除在资源源目录中找到的所有 .java文件。其他插件，如Groovy 插件，会从该集合中排除其他类型的文件。|
|resources.srcDirs|	Set<File>. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一个来设置。|	[projectDir/src/name/resources]	|该源目录包含了此源集的资源文件。|
|allJava	|SourceDirectorySet (read-only)	|java	|该源集的所有.java 文件。有些插件，如Groovy 插件，会从该集合中增加其他的Java源文件。|
|allSource	|SourceDirectorySet (read-only)|	resources + java	|该源集的所有源文件。包含所有的资源文件和Java源文件。有些插件，如Groovy 插件，会从该集合中增加其他的源文件。|


### 23.7.2. 定义新的源集

要定义一个新的源集，你只需在sourceSets {}块中引用它。下面是一个示例：

示例 23.5. 定义一个源集

build.gradle
```
sourceSets {
    intTest
}
```
当您定义一个新的源集时，Java 插件会为该源集添加一些依赖配置，如表 22.6，“Java 插件 - 源集依赖项配置”所示。你可以使用这些配置来定义源集的编译和运行时的依赖。

示例 23.6. 定义源集依赖

build.gradle
```
sourceSets {
    intTest
}

dependencies {
    intTestCompile 'junit:junit:4.11'
    intTestRuntime 'org.ow2.asm:asm-all:4.0'
}
```
Java 插件还添加了大量的任务，用于组装源集的类，如表 22.2，“Java 插件 - 源设置任务”所示。例如，对于一个被叫做intTest的源集，你可以运行gradle intTestClasses来编译 int 测试类。

示例 23.7. 编译源集

gradle intTestClasses的输出结果
```
> gradle intTestClasses
:compileIntTestJava
:processIntTestResources
:intTestClasses

BUILD SUCCESSFUL

Total time: 1 secs
```
### 23.7.3. 一些源集的范例

添加一个包含了源集的类的JAR包

示例 23.8. 为一个源集装配一个JAR文件

build.gradle
```
task intTestJar(type: Jar) {
    from sourceSets.intTest.output
}
```
为一个源集生成 Javadoc：

示例 23.9. 为一个源集生成 Javadoc：

build.gradle
```
task intTestJavadoc(type: Javadoc) {
    source sourceSets.intTest.allJava
}
```
添加一个测试套件以运行一个源集里的测试

示例 23.10. 运行源集里的测试

build.gradle
```
task intTest(type: Test) {
    testClassesDir = sourceSets.intTest.output.classesDir
    classpath = sourceSets.intTest.runtimeClasspath
}
```
## 23.8. Javadoc
Javadoc任务是Javadoc的一个实例。它支持核心的 javadoc 参数选项，以及在Javadoc可执行文件的参考文档中描述的标准 doclet 参数选项。对于支持的 Javadoc 参数选项的完整列表，请参考下面的类的 API 文档： CoreJavadocOptions和StandardJavadocDocletOptions。

表 23.10. Java 插件 - Javadoc 属性

|任务属性	|类型	|默认值|
|--
|classpath	|FileCollection	|sourceSets.main.output + sourceSets.main.compileClasspath|
|source	|FileTree. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一种方式来设置。	|sourceSets.main.allJava|
|destinationDir	|File	|docsDir/javadoc|
|title	|String	|project的名称和版本|

## 23.9. 清理
clean 任务是Delete的一个实例。它只是删除由其dir属性表示的目录。

表 23.11. Java 插件 - Clean 性能

|任务属性|	类型	|默认值|
|--
|dir	|File	|buildDir|

## 23.10. 资源
Java 插件使用Copy任务进行资源的处理。它为该project里的每个源集添加一个实例。你可以在16.6章节，“复制文件”中找到关于copy任务的更多信息。

表 23.12. Java 插件-ProcessResources 属性

|任务属性|	类型|	默认值|
|srcDirs	|Object. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一个来设置。	|sourceSet.resources|
|destinationDir	|File. 可以使用 16.1 章节，“查找文件”中所讲到的任何一种方式来设置。|	sourceSet.output.resourcesDir|

## 23.11. CompileJava
Java 插件为该project里的每个源集添加一个JavaCompile实例。一些最常见的配置选项如下所示。

表 23.13. Java 插件- Compile属性

|任务属性	|类型	|默认值|
|--
|classpath	|FileCollection	|sourceSet.compileClasspath|
|source	|FileTree. 可以使用 16.5 章节，"指定一组输入文件"中所讲到的任何一种方式来设置。|	sourceSet.java|
|destinationDir	|File.	|sourceSet.output.classesDir|

compile任务委派了Ant 的 javac 任务。将options.useAnt设置为false将绕过 Ant 任务，而激活 Gradle 的直接编译器集成。在未来的 Gradle 版本中，将把它作为默认设置。

默认情况下，Java 编译器在 Gradle 过程中运行。将options.fork设置为true将会使编译出现在一个单独的进程中。在 Ant javac 任务中，这意味着将会为每一个compile任务fork一个新的进程，而这将会使编译变慢。相反，Gradle 的直接编译器集成 （见上文） 将尽可能多地重用相同的编译器进程。在这两种情况下，使用options.forkOptions指定的所有fork选项都将得到重视。

## 23.12. Test
test任务是Test的一个实例。它会自动检测和执行test源集中的所有单元测试。测试执行完成后，它还会生成一份报告。同时支持 JUnit 和 TestNG。可以看一看Test的完整的 API。

### 23.12.1. 测试执行

测试在单独的 JVM中执行，与main构建进程隔离。Test任务的 API 可以让你控制什么时候开始。

有大量的属性用于控制测试进程的启动。这包括系统属性、 JVM 参数和使用的Java 可执行文件。

你可以指定是否要并行运行你的测试。Gradle 通过同时运行多个测试进程来提供并行测试的执行。每个测试进程会依次执行一个单一的测试，所以你一般不需要对你的测试做任何的配置来利用这一点。MaxParallelForks属性指定在给定的时间可以运行的测试进程的最大数。它的默认值是 1，也就是说，不并行执行测试。

测试进程程会为其将org.gradle.test.worker系统属性设置为一个唯一标识符，这个标识符可以用于文件名称或其他资源标识符。

你可以指定在执行了一定数量的测试类之后，重启那个测试进程。这是一个很有用的替代方案，让你的测试进程可以有很大的堆内存。forkEvery属性指定了要在测试进程中执行的测试类的最大数。默认是每个测试进程中执行的测试数量不限。

该任务有一个ignoreFailures属性，用以控制测试失败时的行为。test 会始终执行它检测到的每一个测试。如果ignoreFailures为 false，并且有测试不通过，那它会停止继续构建。IgnoreFailures的默认值为 false。

testLogging属性可以配置哪些测试事件需要记录，并且使用什么样的日志级别。默认情况下，对于每个失败的测试都只会打印一个简洁的消息。请参阅TestLoggingContainer，查看如何把你的测试日志打印调整为你的偏好设置。

### 23.12.2. 调试

test任务提供了一个Test.getDebug()属性，可以设置为启动，使JVM在执行测试之前，等待一个debugger连接到它的5005端口上。

这也可以在调用时通过--debug-vm task 选项进行启用。

### 23.12.3. 测试过滤

从Gradle 1.10 开始，可以根据测试的名称模式，只包含指定的测试。过滤，相对于测试类的包含或排除，是一个不同的机制。它将在接下来的段落中进行描述（-Dtest.single， test.include和friends）。后者基于文件，比如测试实现类的物理位置。file-level 的测试选择不支持的很多有趣的情况，都可以用 test-level 过滤来做到。以下这些场景中，有一些Gradle现在就可以处理，而有一些则将在未来得到实现：

* 在指定的测试的方法级别上进行过滤；执行单个测试方法
* 基于自定义注解（以后实现）进行过滤
* 基于测试层次结构进行过滤 ；执行所有继承了某一基类（以后实现） 的测试
* 基于一些自定义的运行时的规则进行过滤，例如某个系统属性或一些静态的特定值（以后实现）
测试过滤功能具有以下的特征：

支持完整的限定类名称或完整的限定的方法名称，例如“org.gradle.SomeTest”、“org.gradle.SomeTest.someMethod”
通配符 “*” 支付匹配任何字符
提供了“--tests”的命令行选项，以方便地设置测试过滤器。这对“单一测试方法的执行”的经典用例特别有用。当使用该命令行选项选项时，在构建脚本中声明的列入过滤器的测试将会被忽略。
Gradle 尽最大努力对有着特定的测试框架 API 的局限的测试进行过滤。一些高级的、 综合的测试可能不完全符合过滤条件。然而，绝大多数的测试和用例都会被很好地处理。
测试过滤将会取代基于文件的测试选择。后者可能在将来会被完全地取代掉。我们将会继续改进测试过滤的API，并添加更多种类的过滤器。

示例 23.11. 在构建脚本中过滤测试

build.gradle
```
test {
    filter {
        //include specific method in any of the tests
        includeTestsMatching "*UiCheck"

        //include all tests from package
        includeTestsMatching "org.gradle.internal.*"

        //include all integration tests
        includeTestsMatching "*IntegTest"
    }
}
```
有关更多的详细信息和示例，请参阅TestFilter的文档。

使用命令行选项的一些示例：

* gradle test --tests org.gradle.SomeTest.someSpecificFeature
* gradle test --tests *SomeTest.someSpecificFeature
* gradle test --tests *SomeSpecificTest
* gradle test --tests all.in.specific.package*
* gradle test --tests *IntegTest
* gradle test --tests *IntegTest*ui*
* gradle someTestTask --tests *UiTest someOtherTestTask --tests *WebTest*ui
### 23.12.4. 通过系统属性执行单一的测试

这种机制已经被上述的“测试过滤”所取代。
设置一个 taskName.single = testNamePattern 的系统属性将会只执行匹配 testNamePattern的那些测试。这个taskName 可以是一个完整的多项目路径，比如“sub1:sub2:test”，或者仅是一个任务名称。testNamePattern将用于形成一个“**/testNamePattern*.class”的包含模式。如果这个模式无法找到任何测试，那么将会抛出一个异常。这是为了使你不会误以为测试通过。如果执行了一个以上的子项目的测试，该模式会被应用于每一个子项目中。如果在一个特定的子项目中，找不到测试用例，那么将会抛出异常。在这种情况下你可以使用路径标记法的模式，这样该模式就会只应用于特定的子项目的测试任务中。或者你可以指定要执行的任务的完整限定名称。你还可以指定多个模式。示例：

* gradle -Dtest.single=ThisUniquelyNamedTest test
* gradle -Dtest.single=a/b/ test
* gradle -DintegTest.single=*IntegrationTest integTest
* gradle -Dtest.single=:proj1:test:Customer build
* gradle -DintegTest.single=c/d/ :proj1:integTest

### 23.12.5. 测试检测

Test任务通过检查编译过的测试类来检测哪些类是测试类。默认情况下它会扫描所有的.class文件。您可以设置自定义的includes 或 excludes，这样就只有这些类才会被扫描。根据所使用的测试框架 （JUnit 或 TestNG），会使用不同的标准进行测试类的检测。

当使用 JUnit 时，我们扫描 JUnit 3 和 4 的测试类。如果满足以下的任何一个条件，这个类就被认为是一个 JUnit 测试类：

类或超类继承自TestCase或GroovyTestCase
类或超类使用了@RunWith进行注解
类或超类含有一个带@Test注解的方法
当使用 TestNG 时，我们扫描所有带有@Test注解的方法。

请注意，抽象类不会被执行。Gradle 还将扫描测试类路径中的 jar 文件里的继承树。

如果你不想要使用测试类检测，可以通过设置scanForTestClasses为 false 来禁用它。这将使test任务只使用includes / excludes 来找到测试类。如果scanForTestClasses为disabled，并且没有指定 include或exclude模式，则使用各自的默认值。对于 include 的默认值是 "**/*Tests.class"， "**/*Test.class"，而对于exclude它的默认值是 "**/Abstract*.class"。

### 23.12.6. 测试分组

JUnit 和 TestNG 可以对测试方法进行复杂的分组。

为对Junit 测试类和方法进行分组，JUnit 4.8引入了类别的概念。[9] test任务可以实现一个规范，让你include和exclude 想要的 JUnit 类别。

示例 23.12. JUnit 类别

build.gradle
```
test {
    useJUnit {
        includeCategories 'org.gradle.junit.CategoryA'
        excludeCategories 'org.gradle.junit.CategoryB'
    }
}
```
TestNG 框架有一个非常相似的概念。在 TestNG 中你可以指定不同的测试组。[10]应从测试执行中include或exclude的测试组，可以在test任务中配置。

示例 23.13. 对TestNG 测试分组

build.gradle
```
test {
    useTestNG {
        excludeGroups 'integrationTests'
        includeGroups 'unitTests'
    }
}
```

### 23.12.7. 测试报告

test任务默认情况下会生成以下结果。

一个 HTML 测试报告。
与Ant Junit report 任务兼容的 XML 格式的结果。这种格式可以被许多工具所支持，比如CI服务器。
有效二进制格式的结果。这个任务会从这些二进制结果生成其他的结果。
您可以使用Test.setTestReport()方法来禁用 HTML 测试报告。目前不能禁用其他的结果。

这里还有一个独立的TestReport任务类型，它可以从一个或多个Test任务实例生成的二进制结果中生成 HTML 测试报告。要使用这个任务类型，你需要定义一个destinationDir和要包含到报告的测试结果。这里是一个范例，从子项目的单元测试中生成一个联合报告：

示例 23.14. 为多个子项目创建一个单元测试报告

build.gradle
```
subprojects {
    apply plugin: 'java'

    // Disable the test report for the individual test task
    test {
        reports.html.enabled = false
    }
}

task testReport(type: TestReport) {
    destinationDir = file("$buildDir/reports/allTests")
    // Include the results from the `test` task in all subprojects
    reportOn subprojects*.test
}
```
你应该注意到，TestReport类型组合了多个测试任务的结果，并且需要聚合各个测试类的结果。这意味着，如果一个给定的测试类被多个test任务所执行，那么测试报告将包括那个类的所有执行结果，但它难以区分那个类的每次执行和它们的输出。

### 23.12.7.1. TestNG 参数化方法和报告

TestNG 支持参数化测试方法，允许一个特定的测试方法使用不同的输入执行多次。Gradle 会在这个方法的执行报告中包含进它的参数值。

给定一个带有两个参数，名为aParameterizedTestMethod参数化测试方法，它将使用名称这个名称进行报告 ：aParameterizedTestMethod(toStringValueOfParam1, toStringValueOfParam2)。这使得在特定的迭代过程，很容易确定参数值。

### 23.12.8. 常规值

表 23.14. Java 插件 - test属性

|任务属性|	类型	|默认值|
|--
|testClassesDir|	File	|sourceSets.test.output.classesDir|
|classpath|	FileCollection|	sourceSets.test.runtimeClasspath|
|testResultsDir	|File	|testResultsDir|
|testReportDir	|File	|testReportDir|
|testSrcDirs	|List<File>	|sourceSets.test.java.srcDirs|

## 23.13. Jar
Jar任务创建一个包含类文件和项目资源的 JAR 文件。JAR 文件在archives依赖配置中被声明为一个构件。这意味着这个JAR文件被包含在一个依赖它的项目的类路径中。如果你把你的项目上传到仓库，这个JAR 文件会被声明为依赖描述符的一部分。你可以在第16.8节，“创建档案”和第51章， 发布artifact中了解如何使用archives和配置artifact。

### 23.13.1. Manifest

每个 jar 或war 对象都有一个单独的Manifest实例的manifest属性。当生成archive时，相应的MANIFEST.MF文件也会被写入进去。

示例 23.15. 自定义的MANIFEST.MF

build.gradle
```
jar {
    manifest {
        attributes("Implementation-Title": "Gradle", "Implementation-Version": version)
    }
}
```
您可以创建一个单独的Manifest实例。它可以用于共享两个jar包的manifest 信息。

示例 23.16. 创建一个manifest 对象。

build.gradle
```
ext.sharedManifest = manifest {
    attributes("Implementation-Title": "Gradle", "Implementation-Version": version)
}
task fooJar(type: Jar) {
    manifest = project.manifest {
        from sharedManifest
    }
}
```
你可以把其他的manifest 合并到任何一个Manifest对象中。其他的manifest可能使用文件路径来描述，像上面的例子，使用对另一个Manifest对象的引用。

示例 23.17. 指定archive的单独MANIFEST.MF

build.gradle
```
task barJar(type: Jar) {
    manifest {
        attributes key1: 'value1'
        from sharedManifest, 'src/config/basemanifest.txt'
        from('src/config/javabasemanifest.txt', 'src/config/libbasemanifest.txt') {
            eachEntry { details ->
                if (details.baseValue != details.mergeValue) {
                    details.value = baseValue
                }
                if (details.key == 'foo') {
                    details.exclude()
                }
            }
        }
    }
}
```
Manifest会根据在from语句中所声明的顺序进行合并。如果基本的manifest和要合并的manifest都定义了同一个key的值，那么默认情况下会采用要合并的manifest的值。你可以通过添加eachEntry action 来完全自定义合并行为，它可以让你对每一项生成的manifest 访问它的一个ManifestMergeDetails实例。这个合并操作不会在from语句中就马上被触发。它是懒执行的，不管是对于生成一个jar包，还是调用了writeTo 或者 effectiveManifest

你可以很轻松地把一个manifest写入磁盘中。

示例 23.18. 指定archive的单独 MANIFEST.MF

build.gradle
```
jar.manifest.writeTo("$buildDir/mymanifest.mf")
```

## 23.14. 上传
关于如何上传archives，将在第51章，发布artifacts中进行描述。


[9] JUnit wiki 包含了有关如何使用 JUnit 类别的详细说明： https://github.com/junit-team/junit/wiki/Categories。

[10] TestNG 文档包含了有关测试组的更多详细信息： http://testng.org/doc/documentation-main.html#test-groups。

