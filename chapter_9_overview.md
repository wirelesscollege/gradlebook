# **Chapter 9. Groovy Quickstart**

第9章Groovy快速入门

构建一个Groovy的项目，你需要使用Groovy插件。Groovy插件继承Java插件更好的适配你项目的Groovy编译，你项目可以包含Groovy源码、java源码或者两者兼有。其他方面Groovy其实与java项目是一样的，你可以参考第7章，Java快速入门。

To build a Groovy project, you use the Groovy plugin. This plugin extends the Java plugin to add Groovy compilation capabilities to your project. Your project can contain Groovy source code, Java source code, or a mix of the two. In every other respect, a Groovy project is identical to a Java project, which we have already seen in Chapter 7, Java Quickstart.

## **9.1. A basic Groovy project**

让我们来看一个例子，使用Groovy插件，build文件将如何配置：

Let's look at an example. To use the Groovy plugin, add the following to your build file:

Example 9.1. Groovy plugin

build.gradle

```
apply plugin: 'groovy'
```

注意：例子代码在samples/groovy/quickstart目录下，既包括源码也包括编译好的二进制文件

Note: The code for this example can be found at samples/groovy/quickstart which is in both the binary and source distributions of Gradle.


This will also apply the Java plugin to the project, if it has not already been applied. The Groovy plugin extends the compile task to look for source files in directory src/main/groovy, and the compileTest task to look for test source files in directory src/test/groovy. The compile tasks use joint compilation for these directories, which means they can contain a mixture of java and groovy source files.

为了使用Groovy编译任务，你必须指定Groovy的版本以及lib仓库，你需要在Goovy得配置里添加依赖关系，并且在编译Groovy和java项目时需要配置Goovy的环境变量，例子里，我们使用Groovy2.2.0版本

To use the groovy compilation tasks, you must also declare the Groovy version to use and where to find the Groovy libraries. You do this by adding a dependency to the groovy configuration. The compile configuration inherits this dependency, so the groovy libraries will be included in classpath when compiling Groovy and Java source. For our sample, we will use Groovy 2.2.0 from the public Maven repository:

Example 9.2. Dependency on Groovy 2.2.0

build.gradle

```
repositories {
    mavenCentral()
}

dependencies {
    compile 'org.codehaus.groovy:groovy-all:2.3.3'
}

```

这是我们的编译配置文件

Here is our complete build file:

Example 9.3. Groovy example - complete build file

build.gradle

```
apply plugin: 'eclipse'
apply plugin: 'groovy'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.codehaus.groovy:groovy-all:2.3.3'
    testCompile 'junit:junit:4.11'
}
```

运行Gradle build 将会编译，测试并且把你的项目打成jar包
Running gradle build will compile, test and JAR your project.

## **9.2. Summary**

这一章描述了一个简单的Groovy项目，通常的项目会比例子要求的更多。因为Groovy项目其实就是一个java项目，所以你可以用它来做Groovy项目配置当然Java也是一样的。

This chapter describes a very simple Groovy project. Usually, a real project will require more than this. Because a Groovy project is a Java project, whatever you can do with a Java project, you can also do with a Groovy project.

你能在24章里找到更多Groovy插件的信息，也可以找到更多的例子在Gradle发行版的samples/groovy目录

You can find out more about the Groovy plugin in Chapter 24, The Groovy Plugin, and you can find more sample Groovy projects in the samples/groovy directory in the Gradle distribution.


