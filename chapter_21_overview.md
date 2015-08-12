# **第二十一章. Gradle 插件**
Gradle 在它的核心中有意地提供了一些小但有用的功能，用于在真实世界中的自动化。所有有用的功能，例如以能够编译 Java 代码为例，都是通过插件进行添加的。插件添加了新任务 （例如JavaCompile），域对象 （例如SourceSet），约定（例如主要的 Java 源代码是位于src/main/java），以及扩展的核心对象和其他插件的对象。

在这一章中，我们将讨论如何使用插件以及术语和插件相关的概念。

## **21.1. 应用插件**
插件都认为是被应用，通过Project.apply()方法来完成。

示例 21.1. 应用插件

build.gradle
```
apply plugin: 'java'
```
插件都有表示它们自己的一个短名称。. 在上述例子中，我们使用短名称java去应用JavaPlugin。

我们还可以使用下面的语法：

示例 21.2. 通过类型应用插件

build.gradle
```
apply plugin: org.gradle.api.plugins.JavaPlugin
```
由于 Gradle 的默认导入 （见附录 E，现有的 IDE 支持和如何应对不支持的情况），您还可以这样写：

示例 21.3. 通过类型应用插件

build.gradle
```
apply plugin: JavaPlugin
```
插件的应用是幂等的。也就是说，一个插件可以被应用多次。如果以前已应用了该插件，任何进一步的应用都不会再有任何效果。

一个插件是任何实现了 Plugin 接口的简单的类。Gradle 提供了核心插件作为其发行包的一部分，所以简单地应用如上插件是你所需要做的。然而，对于第三方插件，你需要进行配置以使插件在构建类路径中可用。有关如何进行此操作的详细信息，请参阅 59.5章节，“构建脚本的外部依赖”。

关于编写自己的插件的详细信息，请参阅 58章节，编写自定义插件。

## **21.2. 插件都做了什么**
把插件应用到项目中可以让插件来扩展项目的功能。它可以做的事情如：

将任务添加到项目 （如编译、 测试）
使用有用的默认设置对已添加的任务进行预配置。
向项目中添加依赖配置 （见第 8.3 节，“依赖配置”）。
通过扩展对现有类型添加新的属性和方法。
让我们来看看：

示例 21.4. 通过插件添加任务

build.gradle
```
apply plugin: 'java'

task show << {
    println relativePath(compileJava.destinationDir)
    println relativePath(processResources.destinationDir)
}
```
gradle -q show 的输出结果
```
> gradle -q show
build/classes/main
build/resources/main
```
Java 插件已经向项目添加了 compileJava任务和processResources任务，并且配置了这两个任务的destinationDir属性。

## **21.3. 约定**
插件可以通过智能的方法对项目进行预配置以支持约定优于配置。Gradle 对此提供了机制和完善的支持，而它是强大-然而-简洁的构建脚本中的一个关键因素。

在上面的示例中我们看到，Java 插件添加了一个任务，名字为compileJava ，有一个名为destinationDir 的属性（即配置编译的 Java 代码存放的地方）。Java 插件默认此属性指向项目目录中的build/classes/main。这是通过一个合理的默认的约定优于配置的例子。

我们可以简单地通过给它一个新的值来更改此属性。

示例 21.5. 更改插件的默认设置

build.gradle
```
apply plugin: 'java'

compileJava.destinationDir = file("$buildDir/output/classes")

task show << {
    println relativePath(compileJava.destinationDir)
}
```
gradle -q show 的输出结果
```
> gradle -q show
build/output/classes
```
然而， compileJava任务很可能不是唯 一需要知道类文件在哪里的任务。

Java 插件添加了 source sets 的概念 （见SourceSet） 来描述的源文件集的各个方面，其中一个方面是在编译的时候这些类文件应该被写到哪个地方。Java 插件将compileJava任务的destinationDir属性映射到源文件集的这一个方面。

我们可以通过这个源码集修改写入类文件的位置。

示例 21.6. 插件中的约定对象

build.gradle
```
apply plugin: 'java'

sourceSets.main.output.classesDir = file("$buildDir/output/classes")

task show << {
    println relativePath(compileJava.destinationDir)
}
```
gradle -q show 的输出结果
```
> gradle -q show
build/output/classes
```
在上面的示例中，我们应用 Java 插件，除其他外，还做了下列操作：

添加了一个新的域对象类型： SourceSet
通过属性的默认（即常规）配置了 main 源码集
配置支持使用这些属性来执行工作的任务
所有这一切都发生在apply plugin: "java"这一步过程中。在上面例子中，我们在约定配置被执行之后，修改了类文件所需的位置。在上面的示例中可以注意到，compileJava.destinationDir的值也被修改了，以反映出配置的修改。

考虑一下另一种消费类文件的任务的情况。如果这个任务使用sourceSets.main.output.classesDir的值来配置，那么修改了这个位置的值，无论它是什么时候被修改，将同时更新compileJava任务和这一个消费者任务。

这种配置对象的属性以在所有时间内（甚至当它更改的时候）反映另一个对象的任务的值的能力被称为“映射约定”。它可以令 Gradle 通过约定优于配置及合理的默认值来实现简洁的配置方式。而且，如果默认约定需要进行修改时，也不需要进行完全的重新配置。如果没有这一点，在上面的例子中，我们将不得不重新配置需要使用类文件的每个对象。

## **21.4. 更多关于插件的内容**
这一章旨在作为对插件和 Gradle 及他们扮演的角色的导言。关于插件的内部运作的详细信息，请参阅第 58 章，编写自定义插件。

百度搜索[无线学院](http://wirelesscollege.cn)