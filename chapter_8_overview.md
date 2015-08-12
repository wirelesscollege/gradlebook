# **第八章.依赖管理基础**

Chapter 8. Dependency Management Basics

本章节介绍如何使用Gradle进行基本的依赖管理.

This chapter introduces some of the basics of dependency management in Gradle.

## **8.1. 神马是依赖管理?**

8.1. What is dependency management?

通俗来讲,依赖管理由如下两部分组成.首先,Gradle需要知道项目构建或运行所需要的一些文件,以便于找到这些需要的文件. 我们称这些输入的文件为项目的依赖.其次,你可能需要构建完成后自动上传到某个地方. 我们称这些输出为发布.下面来仔细介绍一下这两部分:

Very roughly, dependency management is made up of two pieces. Firstly, Gradle needs to know about the things that your project needs to build or run, in order to find them. We call these incoming files the dependencies of the project. Secondly, Gradle needs to build and upload the things that your project produces. We call these outgoing files the publications of the project. Let's look at these two pieces in more detail:

大部分工程都不太可能完全自给自足,一般你都会用到其他工程的文件. 比如我工程需要Hibernate就得把它的类库加进来,比如测试的时候可能需要某些额外jar包,例如JDBC驱动或Ehcache之类的Jar包. 

Most projects are not completely self-contained. They need files built by other projects in order to be compiled or tested and so on. For example, in order to use Hibernate in my project, I need to include some Hibernate jars in the classpath when I compile my source. To run my tests, I might also need to include some additional jars in the test classpath, such as a particular JDBC driver or the Ehcache jars.

这些文件就是工程的依赖.Gradle需要你告诉它工程的依赖是什么,它们在哪,然后帮你加入构建中. 依赖可能需要去远程库下载,比如Maven 或者 Ivy 库.也可以是本地库,甚至可能是另一个工程. 我们称这个过程叫依赖解决

These incoming files form the dependencies of the project. Gradle allows you to tell it what the dependencies of your project are, so that it can take care of finding these dependencies, and making them available in your build. The dependencies might need to be downloaded from a remote Maven or Ivy repository, or located in a local directory, or may need to be built by another project in the same multi-project build. We call this process dependency resolution.

通常,依赖的自身也有依赖.例如,Hibernate 核心类库就依赖于一些其他的类库.所以,当Gradle构建你的工程时,会去找到这些依赖. 我们称之为依赖传递.

Often, the dependencies of a project will themselves have dependencies. For example, Hibernate core requires several other libraries to be present on the classpath with it runs. So, when Gradle runs the tests for your project, it also needs to find these dependencies and make them available. We call these transitive dependencies.

大部分工程构建的主要目的是脱离工程使用.例如,生成jar包,包括源代码、文档等,然后发布出去.

The main purpose of most projects is to build some files that are to be used outside the project. For example, if your project produces a java library, you need to build a jar, and maybe a source jar and some documentation, and publish them somewhere.

这些输出的文件构成了项目的发布内容.Gralde也会为你分担这些工作.你声明了发布到到哪,Gradle就会发布到哪. "发布"的意思就是你想做什么.比如,复制到某个目录,上传到Maven或Ivy仓库.或者在其它项目里使用,这些都可以称之为 发行.

These outgoing files form the publications of the project. Gradle also takes care of this important work for you. You declare the publications of your project, and Gradle take care of building them and publishing them somewhere. Exactly what "publishing" means depends on what you want to do. You might want to copy the files to a local directory, or upload them to a remote Maven or Ivy repository. Or you might use the files in another project in the same multi-project build. We call this process publication.

## **8.2. 依赖声明**

8.2. Declaring your dependencies

来看一下这个脚本里声明依赖的部分:

Let's look at some dependency declarations. Here's a basic build script:

例 8.1. 声明依赖 Example 8.1. Declaring dependencies

build.gradle

```
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

这是毛意思呢?这段脚本是这么个意思.首先,Hibernate-core.3.6.7.final.jar这货是编译期必需的依赖.并且这货相关的依赖也会一并被加载进来 该段脚本同时还声明项目测试阶段需要 4.0版本以上的Junit.同时告诉Gradle可以去Maven中央仓库去找这些依赖.下面的章节会进行更详细的描述.

What's going on here? This build script says a few things about the project. Firstly, it states that Hibernate core 3.6.7.Final is required to compile the project's production source. By implication, Hibernate core and its dependencies are also required at runtime. The build script also states that any junit >= 4.0 is required to compile the project's tests. It also tells Gradle to look in the Maven central repository for any dependencies that are required. The following sections go into the details.

## **8.3. 依赖配置**

8.3. Dependency configurations

Gradle中依赖以组的形式来划分不同的配置. 每个配置都是一套具体名称组成的依赖. 我们称之为依赖配置 你也可以借由此声明外部依赖.后面我们会了解到,这也可用用来声明项目的发布

In Gradle dependencies are grouped into configurations. A configuration is simply a named set of dependencies. We will refer to them as dependency configurations. You can use them to declare the external dependencies of your project. As we will see later, they are also used to declare the publications of your project.

Java插件定义了许多标准配置项.这些配置项形成了插件本身的classpath. 比如下面罗列的这一些.并且你可以从Table 23.5, “Java 插件 - 依赖配置”了解到更多详细内容. 

The Java plugin defines a number of standard configurations. These configurations represent the classpaths that the Java plugin uses. Some are listed below, and you can find more details in Table 23.5, “Java plugin - dependency configurations”.

* compile

编译范围依赖在所有的classpath中可用，同时它们也会被打包

The dependencies required to compile the production source of the project.

*runtime

runtime依赖在运行和测试系统的时候需要,但在编译的时候不需要.比如,你可能在编译的时候只需要JDBC API JAR,而只有在运行的时候才需要JDBC驱动实现

The dependencies required by the production classes at runtime. By default, also includes the compile time dependencies.

*testCompile

测试期编译需要的附加依赖

The dependencies required to compile the test source of the project. By default, also includes the compiled production classes and the compile time dependencies.

*testRuntime

测试运行期需要 The dependencies required to run the tests. By default, also includes the compile, runtime and test compile dependencies.

不同的插件提供了不同的标准配置,你甚至也可以定义属于自己的配置项.具体可查看 章节 50.3, “依赖配置” 

Various plugins add further standard configurations. You can also define your own custom configurations to use in your build. Please see Section 50.3, “Dependency configurations” for the details of defining and customizing dependency configurations.

## **8.4. 外部依赖**

8.4. External dependencies

依赖的类型有很多种,其中有一种类型称之为外部依赖. 这种依赖由外部构建或者在不同的仓库中,例如Maven中央仓库或Ivy仓库中抑或是本地文件系统的某个目录中.

There are various types of dependencies that you can declare. One such type is an external dependency. This a dependency on some files built outside the current build, and stored in a repository of some kind, such as Maven central, or a corporate Maven or Ivy repository, or a directory in the local file system.

定义外部依赖需要像下面这样进行依赖配置

To define an external dependency, you add it to a dependency configuration:

例  8.2. 定义外部依赖.
Example 8.2. Definition of an external dependency

build.gradle

```
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
}
```

外部依赖包含 group,name和version几个属性. 根据选取仓库的不同, group和version也可能是可选的.

An external dependency is identified using group , name and version attributes. Depending on which kind of repository you are using, group and version may be optional.

当然,也有一种更加简洁的方式来声明外部依赖. 采用 : 将三个属性拼接在一起即可. "group:name:version" 

There is a shortcut form for declaring external dependencies, which uses a string of the form "group:name:version" .

例 8.3. 快速定义外部依赖

Example 8.3. Shortcut definition of an external dependency

build.gradle

```
dependencies {
    compile 'org.hibernate:hibernate-core:3.6.7.Final'
}
```

更多定义依赖的方式可以参阅 章节 50.4, “如何声明依赖”. 

To find out more about defining and working with dependencies, have a look at Section 50.4, “How to declare your dependencies”.

## **8.5. 仓库**

8.5. Repositories

Gradle是在一个被称之为仓库的地方找寻所需的外部依赖. 仓库即是一个按 group,name和version 规则进行存储的一些文件.Gradle可以支持不同的仓库存储格式,如Maven和Ivy,并且还提供多种与仓库进行通信的方式,如通过本地文件系统或HTTP. 

How does Gradle find the files for external dependencies? Gradle looks for them in a repository. A repository is really just a collection of files, organized by group , name and version . Gradle understands several different repository formats, such as Maven and Ivy, and several different ways of accessing the repository, such as using the local file system or HTTP.

默认情况下,Gradle没有定义任何仓库,你需要在使用外部依赖之前至少定义一个仓库,例如Maven中央仓库. 

By default, Gradle does not define any repositories. You need to define at least one before you can use external dependencies. One option is use the Maven central repository:

例 8.4. 使用Maven中央仓库
Example 8.4. Usage of Maven central repository

build.gradle

```
repositories {
    mavenCentral()
}
```

或者其它远程Maven仓库:

Or a remote Maven repository:

例 8.5. 使用Maven远程仓库

Example 8.5. Usage of a remote Maven repository

build.gradle

```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

或者采用Ivy远程仓库

Or a remote Ivy repository:

例 8.6. 采用Ivy远程仓库

Example 8.6. Usage of a remote Ivy directory

build.gradle

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

或者在指定本地文件系统构建的库

You can also have repositories on the local file system. This works for both Maven and Ivy repositories.

例 8.7. 采用本地Ivy目录

Example 8.7. Usage of a local Ivy directory

build.gradle

```
repositories {
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```

一个项目可以采用多个库.Gradle会按照顺序从各个库里寻找所需的依赖文件,并且一旦找到第一个便停止搜索.

A project can have multiple repositories. Gradle will look for a dependency in each repository in the order they are specified, stopping at the first repository that contains the requested module.

了解更多库相关的信息请参阅章节 50.6, “仓库”.

To find out more about defining and working with repositories, have a look at Section 50.6, “Repositories”.

## **8.6. 打包发布**

8.6. Publishing artifacts

依赖配置也被用于发布文件[3] 我们称之为打包发布,或发布

Dependency configurations are also used to publish files.[3] We call these files publication artifacts, or usually just artifacts.

插件对于打包提供了完美的支持,所以通常而言无需特别告诉Gradle需要做什么.但是你需要告诉Gradle发布到哪里. 这就需要在 uploadArchives 任务中添加一个仓库.下面的例子是如何发布到远程Ivy仓库的:

The plugins do a pretty good job of defining the artifacts of a project, so you usually don't need to do anything special to tell Gradle what needs to be published. However, you do need to tell Gradle where to publish the artifacts. You do this by attaching repositories to the uploadArchives task. Here's an example of publishing to a remote Ivy repository:

例 8.8. 发布到Ivy仓库.

Example 8.8. Publishing to an Ivy repository

build.gradle

```
uploadArchives {
    repositories {
        ivy {
            credentials {
                username "username"
                password "pw"
            }
            url "http://repo.mycompany.com"
        }
    }
}
```

执行gradleuploadArchives, Gradle便会构建并上传你的jar包,同时会生成一个 ivy.xml一起上传到目标仓库.

Now, when you run gradle uploadArchives, Gradle will build and upload your Jar. Gradle will also generate and upload an ivy.xml as well.

当然你也可以发布到Maven仓库中.语法只需稍微一换就可以了.[4]

p.s:发布到Maven仓库你需要Maven插件的支持,当然,Gradle也会同时产生 pom.xml一起上传到目标仓库.

You can also publish to Maven repositories. The syntax is slightly different.[4] Note that you also need to apply the Maven plugin in order to publish to a Maven repository. In this instance, Gradle will generate and upload a pom.xml .

例 8.9. 发布到Maven仓库

Example 8.9. Publishing to a Maven repository

build.gradle

```
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```

了解更多有关发布的内容,请参阅第 51 章, 内容发布 artifacts.

To find out more about publication, have a look at Chapter 51, Publishing artifacts.

## **8.7. 下一步目标**

8.7. Where to next?

了解更多有关依赖的问题,请参阅第 50 章, 依赖管理, 了解更多有关发布的内容,请参阅第 51 章, 内容发布 artifacts.

For all the details of dependency resolution, see Chapter 50, Dependency Management, and for artifact publication see Chapter 51, Publishing artifacts.

若对DSL感兴趣 ,请看Project.configurations{}, Project.repositories{} and Project.dependencies{}.

If you are interested in the DSL elements mentioned here, have a look at Project.configurations{}, Project.repositories{} and Project.dependencies{}.

另外,继续顺着手册学习其它章节内容吧.~

Otherwise, continue on to some of the other tutorials.

---------------------

[3] 这让人感觉有点囧,我们正在尝试逐步分离两种概念.

[3] We think this is confusing, and we are gradually teasing apart the two concepts in the Gradle DSL.

[4] 我们也在努力改进语法让发布到Maven仓库不那么费劲.

[4] We are working to make the syntax consistent for resolving from and publishing to Maven repositories.

百度搜索[无线学院](http://wirelesscollege.cn)