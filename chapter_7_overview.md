# **第七章.Java构建入门**

Chapter 7. Java Quickstart

## **7.1. Java插件**

7.1. The Java plugin

如你所见,Gradle是一个通用工具.它可以通过脚本构建任何你想要实现的东西.真正实现开箱即用. 但前提是你需要在脚本中编写好代码才行.

As we have seen, Gradle is a general-purpose build tool. It can build pretty much anything you care to implement in your build script. Out-of-the-box, however, it doesn't build anything unless you add code to your build script to do so.

大部分Java项目基本流程都是相似的:编译源文件,进行单元测试,创建Jar包. 使用Gradle做这些工作不必为每个工程都编写代码.Gradle已经提供了完美的插件来解决这些问题. 插件就是Gradle的扩展,简而言之就是为你添加一些非常有用的 默认配置.Gradle自带了很多插件,并且你也可以很容易的编写和分享自己的插件. Java plugin作为其中之一,为你提供了诸如编译,测试,打包等一些功能.

Most Java projects are pretty similar as far as the basics go: you need to compile your Java source files, run some unit tests, and create a JAR file containing your classes. It would be nice if you didn't have to code all this up for every project. Luckily, you don't have to. Gradle solves this problem through the use of plugins. A plugin is an extension to Gradle which configures your project in some way, typically by adding some pre-configured tasks which together do something useful. Gradle ships with a number of plugins, and you can easily write your own and share them with others. One such plugin is the Java plugin. This plugin adds some tasks to your project which will compile and unit test your Java source code, and bundle it into a JAR file.

Java插件为工程定义了许多默认值,如Java源文件位置.如果你遵循这些默认规则,那么你无需在你的脚本文件中书写太多代码. 当然,Gradle也允许你自定义项目中的一些规则.实际上,由于对Java工程的构建是基于插件的,那么你也可以完全不用插件自己编写代码来进行构建.

The Java plugin is convention based. This means that the plugin defines default values for many aspects of the project, such as where the Java source files are located. If you follow the convention in your project, you generally don't need to do much in your build script to get a useful build. Gradle allows you to customize your project if you don't want to or cannot follow the convention in some way. In fact, because support for Java projects is implemented as a plugin, you don't have to use the plugin at all to build a Java project, if you don't want to.

后面的章节我们通过许多深入的例子介绍了如何使用Java插件来进行以来管理和多项目构建等.但在这个章节我们需要先了解Java插件的基本用法.

We have in-depth coverage with many examples about the Java plugin, dependency management and multi-project builds in later chapters. In this chapter we want to give you an initial idea of how to use the Java plugin to build a Java project.

## **7.2. 一个基本Java项目**

7.2. A basic Java project

来看一下下面这个小例子,想用Java插件,只需增加如下代码到你的脚本里.

Let's look at a simple example. To use the Java plugin, add the following to your build file:

例 7.1. 采用Java插件

Example 7.1. Using the Java plugin

build.gradle
apply plugin: 'java'
					
备注:示例代码可以在Gralde发行包中的 samples/java/quickstart下找到.

Note: The code for this example can be found at samples/java/quickstart which is in both the binary and source distributions of Gradle.

定义一个Java项目只需如此而已.这将会为你添加Java插件及其一些内置任务.

This is all you need to define a Java project. This will apply the Java plugin to your project, which adds a number of tasks to your project.

添加了哪些任务?

What tasks are available?

你可以运行gradle tasks列出任务列表.这样便可以看到Java插件为你添加了哪些任务

You can use gradle tasks to list the tasks of a project. This will let you see the tasks that the Java plugin has added to your project.
标准目录结构如下:
```
project
    +build
    +src/main/java
    +src/main/resources
    +src/test/java
    +src/test/resources
```

Gradle默认会从src/main/java搜寻打包源码,在src/test/java下搜寻测试源码. 
并且src/main/resources下的所有文件按都会被打包,所有src/test/resources下的文件 都会被添加到类路径用以执行测试.所有文件都输出到build下,打包的文件输出到 build/libs下

Gradle expects to find your production source code under src/main/java and your test source code under src/test/java . In addition, any files under src/main/resources will be included in the JAR file as resources, and any files under src/test/resources will be included in the classpath used to run the tests. All output files are created under the build directory, with the JAR file ending up in the build/libs directory.

## **7.2.1. 构建项目**

7.2.1. Building the project

Java插件为你添加了众多任务.但是它们只是在你需要构建项目的时候才能发挥作用. 最常用的就是build任务,这会构建整个项目.当你执行 gradle build时,Gralde会编译并执行单元测试,并且将src/main/*下面class和资源文件打包 

The Java plugin adds quite a few tasks to your project. However, there are only a handful of tasks that you will need to use to build the project. The most commonly used task is the build task, which does a full build of the project. When you run gradle build, Gradle will compile and test your code, and create a JAR file containing your main classes and resources:

例 7.2. 构建Java项目

Example 7.2. Building a Java project

运行gradle build的输出结果

Output of gradle build

```
> gradle build
:compileJava
:processResources
:classes
:jar
:assemble
:compileTestJava
:processTestResources
:testClasses
:test
:check
:build


BUILD SUCCESSFUL

Total time: 1 secs

```

其余一些较常用的任务有如下几个:

Some other useful tasks are:

* clean

删除build目录以及所有构建完成的文件.

Deletes the build directory, removing all built files.

* assemble

编译并打包jar文件,但不会执行单元测试.一些其他插件可能会增强这个任务的功能.例如,如果采用了War插件, 这个任务便会为你的项目打出War包

Compiles and jars your code, but does not run the unit tests. Other plugins add more artifacts to this task. For example, if you use the War plugin, this task will also build the WAR file for your project.

* check

编译并测试代码.一些其他插件也可能会增强这个任务的功能.例如,如果采用了Code-quality插件,这个任务会额外执行Checkstyle 

Compiles and tests your code. Other plugins add more checks to this task. For example, if you use the Code-quality plugin, this task will also run Checkstyle against your source code.

## **7.2.2. 外部依赖**

7.2.2. External dependencies

通常,一个Java项目拥有许多外部依赖.你需要告诉Gradle如何找到并引用这些外部文件. 在Gradle中通常Jar包都存在于仓库中.仓库可以用来搜寻依赖或发布 项目产物.下面是一个采用Maven仓库的例子.

Usually, a Java project will have some dependencies on external JAR files. To reference these JAR files in the project, you need to tell Gradle where to find them. In Gradle, artifacts such as JAR files, are located in a repository. A repository can be used for fetching the dependencies of a project, or for publishing the artifacts of a project, or both. For this example, we will use the public Maven repository:

例 7.3 添加Maven仓库

Example 7.3. Adding Maven repository

```
build.gradle
repositories {
    mavenCentral()
}
```

添加依赖.这里声明了编译期所需依赖commons-collections和测试期所需依赖junit.

Let's add some dependencies. Here, we will declare that our production classes have a compile-time dependency on commons collections, and that our test classes have a compile-time dependency on junit:

例 7.4 添加依赖

Example 7.4. Adding dependencies

```
build.gradle
dependencies {
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

了解更多可参阅第 8&nsbp;章, 依赖管理基础.

You can find out more in Chapter 8, Dependency Management Basics.

## **7.2.3. 自定义项目**

7.2.3. customizing the project

Java插件为你的项目添加了众多默认配置.这些默认值通常对于一个普通项目来说已经足够了.但如果你觉得不适用修改起来也很简单. 看下面的例子,我们为Java项目指定了版本号以及所用的Jdk版本,并且添加一些属性到mainfest中.

The Java plugin adds a number of properties to your project. These properties have default values which are usually sufficient to get started. It's easy to change these values if they don't suit. Let's look at this for our sample. Here we will specify the version number for our Java project, along with the Java version our source is written in. We also add some attributes to the JAR manifest.

例 7.5. 自定义 MANIFEST.MF

Example 7.5. Customization of MANIFEST.MF

build.gradle

```
sourceCompatibility = 1.5
version = '1.0'
jar {
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
    }
}
```
都有哪些可用属性?

What properties are available?

可以执行gradle properties来得到项目的属性列表. 用这条命令可以看到插件添加的属性以及默认值.

You can use gradle properties to list the properties of a project. This will allow you to see the properties added by the Java plugin, and their default values.

Java插件添加的都是一些普通任务,如同他们写在Build文件中一样.这意味着前面章节展示的机制都可以用来修改这些任务的行为. 例如,可以设置任务的属性,添加任务行为,更改任务依赖,甚至是重写覆盖整个任务.在下面的例子中,我们将修改 test任务, 这是一个Test类型任务.让我们来在它执行时为它添加一些系统属性.

The tasks which the Java plugin adds are regular tasks, exactly the same as if they were declared in the build file. This means you can use any of the mechanisms shown in earlier chapters to customize these tasks. For example, you can set the properties of a task, add behaviour to a task, change the dependencies of a task, or replace a task entirely. In our sample, we will configure the test task, which is of type Test, to add a system property when the tests are executed:

例 7.6 为test添加系统属性

Example 7.6. Adding a test system property

build.gradle

```
test {
    systemProperties 'property': 'value'
}

```

## **7.2.4. 发布Jar包**

7.2.4. Publishing the JAR file

如何发布Jar包?你需要告诉Gradle发布到到哪.在Gradle中Jar包通常被发布到某个仓库中. 在下面的例子中,我们会将Jar包发布到本地目录.当然你也可以发布到远程仓库或多个远程仓库中.

Usually the JAR file needs to be published somewhere. To do this, you need to tell Gradle where to publish the JAR file. In Gradle, artifacts such as JAR files are published to repositories. In our sample, we will publish to a local directory. You can also publish to a remote location, or multiple locations.

例 7.7. 发布Jar包

Example 7.7. Publishing the JAR file

build.gradle

```
uploadArchives {
    repositories {
       flatDir {
           dirs 'repos'
       }
    }
}
```

执行 gradle uploadArchives以发布Jar包.

To publish the JAR file, run gradle uploadArchives.

## **7.2.5. 创建Eclipse文件**

7.2.5. Creating an Eclipse project

若要把项目导入Eclipse中,你需要添加另外一个插件到你的脚本文件中.

To import your project into Eclipse, you need to add another plugin to your build file:

例 7.8. Eclipse plugin

Example 7.8. Eclipse plugin

build.gradle

```
apply plugin: 'eclipse'
```
						
执行gradle eclipse来生成Eclipse项目文件.了解更多内容请参阅 第 38 章, The Eclipse 插件.

Now execute gradle eclipse command to generate Eclipse project files. More on Eclipse task can be found in Chapter 38, The Eclipse Plugin.

## **7.2.6. 示例汇总**

7.2.6. Summary

这是示例代码汇总得到的一个完整脚本:

Here's the complete build file for our sample:

例 7.9. Java 示例 - 一个完整构建脚本

Example 7.9. Java example - complete build file

build.gradle

```
apply plugin: 'java'
apply plugin: 'eclipse'

sourceCompatibility = 1.5
version = '1.0'
jar {
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
    }
}

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}

test {
    systemProperties 'property': 'value'
}

uploadArchives {
    repositories {
       flatDir {
           dirs 'repos'
       }
    }
}
```

## **7.3. 多项目构建**

7.3. Multi-project Java build

现在来看一个典型的多项目构建的例子.项目结构如下:

Now let's look at a typical multi-project build. Below is the layout for the project:

例 7.10. 多项目构建-项目结构 Example 7.10. Multi-project build - hierarchical layout

```
Build layout
multiproject/
  api/
  services/webservice/
  shared/

```

备注: 本示例代码可在Gradle发行包的samples/java/multiproject位置找到

Note: The code for this example can be found at samples/java/multiproject which is in both the binary and source distributions of Gradle.

此处有三个工程. api 工程用来生成给客户端用的jar文件，这个jar文件可以为XML webservice 提供Java客户端. webservice 是一个web应用,生成 XML. shared 工程包含的是前述两个工程共用的代码. 

Here we have three projects. Project api produces a JAR file which is shipped to the client to provide them a Java client for your XML webservice. Project webservice is a webapp which returns XML. Project shared contains code used both by api and webservice .

## **7.3.1. 多项目构建定义**

7.3.1. Defining a multi-project build

定义一个多项目构建工程需要在根目录(本例中与multiproject同级)创建一个setting 配置文件来指明构建包含哪些项目.并且这个文件必需叫settings.gradle 本例的配置文件如下:

To define a multi-project build, you need to create a settings file. The settings file lives in the root directory of the source tree, and specifies which projects to include in the build. It must be called settings.gradle . For this example, we are using a simple hierarchical layout. Here is the corresponding settings file:

例 7.11. 多项目构建中的settings.gradle Example 7.11. Multi-project build - settings.gradle file

settings.gradle

```
include "shared", "api", "services:webservice", "services:shared"
```
						
了解更多可参阅第 56 章, 多项目构建.

You can find out more about the settings file in Chapter 56, Multi-project Builds.

## **7.3.2. 公共配置**

7.3.2. Common configuration

对多项目构建而言,总有一些共同的配置.在本例中,我们会在根项目上采用配置注入的方式定义一些公共配置.根项目就像一个容器,子项目会迭代访问它的配置并注入到自己的配置中. 这样我们就可以简单的为所有工程定义主配置单了:

For most multi-project builds, there is some configuration which is common to all projects. In our sample, we will define this common configuration in the root project, using a technique called configuration injection. Here, the root project is like a container and the subprojects method iterates over the elements of this container - the projects in this instance - and injects the specified configuration. This way we can easily define the manifest content for all archives, and some common dependencies:

例 7.12. 多项目构建-公共配置

Example 7.12. Multi-project build - common configuration

build.gradle

```
subprojects {
    apply plugin: 'java'
    apply plugin: 'eclipse-wtp'

    repositories {
       mavenCentral()
    }

    dependencies {
        testCompile 'junit:junit:4.11'
    }

    version = '1.0'

    jar {
        manifest.attributes provider: 'gradle'
    }
}
````

值得注意的是我们为每个子项目都应用了Java插件.这意味着我们在前面章节学习的内容在子项目中也都是可用的. 所以你可以在根项目目录进行编译,测试,打包等所有操作.

Notice that our sample applies the Java plugin to each subproject. This means the tasks and configuration properties we have seen in the previous section are available in each subproject. So, you can compile, test, and JAR all the projects by running gradle build from the root project directory.

7.3.3. 工程依赖

7.3.3. Dependencies between projects

同一个构建中可以建立工程依赖,一个工程的jar包可以提供给另外一个工程使用.例如我们可以让 api工程以依赖于shared工程的jar包. 这样Gradle在构建api之前总是会先构建shared工程.

You can add dependencies between projects in the same build, so that, for example, the JAR file of one project is used to compile another project. In the api build file we will add a dependency on the JAR produced by the shared project. Due to this dependency, Gradle will ensure that project shared always gets built before project api .

例 7.13. 多项目构建-工程依赖
Example 7.13. Multi-project build - dependencies between projects

api/build.gradle

```
dependencies {
    compile project(':shared')
}
```

参阅 56.7.1 小节, “停用项目依赖”来了解如何停用此功能.

See Section 56.7.1, “Disabling the build of dependency projects” for how to disable this functionality.

## **7.3.4. 打包发布**

7.3.4. Creating a distribution

如何发布,请看下文:

We also add a distribution, that gets shipped to the client:

例 7.14. 多项目构建-发布

Example 7.14. Multi-project build - distribution file


```
api/build.gradle
task dist(type: Zip) {
    dependsOn spiJar
    from 'src/dist'
    into('libs') {
        from spiJar.archivePath
        from configurations.runtime
    }
}

artifacts {
   archives dist
}
```

## **7.4. 下一步目标?**

7.4. Where to next?

本章中,我们了解了如何构建一个基本Java工程.但这都是一小部分基础,用Gradle还可以做很多事.关于Java插件想了解更多可参阅 第 23 章, The Java Plugin,并且你可以在Gradle发行包中的 samples/java目录找到更多例子.

In this chapter, you have seen how to do some of the things you commonly need to build a Java based project. This chapter is not exhaustive, and there are many other things you can do with Java projects in Gradle. You can find out more about the Java plugin in Chapter 23, The Java Plugin, and you can find more sample Java projects in the samples/java directory in the Gradle distribution.

另外,不要忘了继续阅读第 8 章, 依赖管理基础 内容哟~~~

Otherwise, continue on to Chapter 8, Dependency Management Basics.

百度搜索[无线学院](http://wirelesscollege.cn)