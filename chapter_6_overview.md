# **第六章.构建基础**

Chapter 6. Build Script Basics

## **6.1. Projects 和 tasks**

6.1. Projects and tasks

projects 和 tasks是Gradle中最重要的两个概念

Everything in Gradle sits on top of two basic concepts: projects and tasks.

任何一个Gradle构建都是由一个或多个 projects 组成.每个project包括许多可构建组成部分. 这完全取决于你要构建些什么.举个栗子.每个project或许是一个jar包或者一个web应用, 它也可以是一个由许多其他项目中产生的jar构成的zip压缩包.一个project不必描述它只能进行构建操作. 它也可以部署你的应用或搭建你的环境.不要担心它像听上去的那样庞大. Gradle的build-by-convention可以让您来具体定义一个project到底该做什么.

Every Gradle build is made up of one or more projects. What a project represents depends on what it is that you are doing with Gradle. For example, a project might represent a library JAR or a web application. It might represent a distribution ZIP assembled from the JARs produced by other projects. A project does not necessarily represent a thing to be built. It might represent a thing to be done, such as deploying your application to staging or production environments. Don't worry if this seems a little vague for now. Gradle's build-by-convention support adds a more concrete definition for what a project is.

每个project都由多个tasks组成.每个task都代表了构建执行过程中的一个原子性操作.如编译,打包,生成javadoc,发布到某个仓库等操作

Each project is made up of one or more tasks. A task represents some atomic piece of work which a build performs. This might be compiling some classes, creating a JAR, generating javadoc, or publishing some archives to a repository.

到目前为止,可以发现我们可以在一个project中定义一些简单任务,后续章节将会阐述多项目构建和多项目多任务的内容.

For now, we will look at defining some simple tasks in a build with one project. Later chapters will look at working with multiple projects and more about working with projects and tasks.

## **6.2. Hello world**

6.2. Hello world

你可以通过在命令行运行gradle命令来执行构建, gradle命令会从当前目录下寻找 build.gradle 文件来执行构建.我们称 build.gradle 文件为构建脚本. 严格来说这其实是一个构建配置脚本,后面你会了解到这个构建脚本定义了一个project和一些默认的task

You run a Gradle build using the gradle command. The gradle command looks for a file called build.gradle in the current directory. [2] We call this build.gradle file a build script, although strictly speaking it is a build configuration script, as we will see later. The build script defines a project and its tasks.

你可以创建如下脚本到 build.gradle 中 To try this out, create the following build script named build.gradle .

例6.1. 第一个构建脚本
Example 6.1. The first build script
```
build.gradle
task hello {
    doLast {
        println 'Hello world!'
    }
}
```

然后在该文件所在目录执行 

```gradle -q hello```

In a command-line shell, enter into the containing directory and execute the build script by running gradle -q hello:

-q 参数的作用是什么?

What does -q do?

该文档的示例中很多地方在调用gradle命令时都加了 -q 参数. 
该参数用来控制gradle的日志级别,可以保证只输出我们需要的内容. 具体可参阅本文档 第十八章 日志 来了解更多参数和信息.

Most of the examples in this user guide are run with the -q command-line option. This suppresses Gradle's log messages, so that only the output of the tasks is shown. This keeps the example output in this user guide a little clearer. You don't need to use this option if you don't want. See Chapter 18, Logging for more details about the command-line options which affect Gradle's output.

例6.2. 执行脚本

Example 6.2. Execution of a build script

Output of gradle -q hello

```
> gradle -q hello
Hello world!
```

上面的脚本定义了一个叫做 hello 的 task,并且给它添加了一个动作. 当执行gradle hello的时候, Gralde便会去调用 hello 这个任务来执行给定操作.这些操作其实就是一个用groovy书写的闭包

What's going on here? This build script defines a single task, called hello , and adds an action to it. When you run gradle hello, Gradle executes the hello task, which in turn executes the action you've provided. The action is simply a closure containing some Groovy code to execute.

如果你觉得它看上去跟Ant中的targets很像,那么是对的. Gradle的tasks就相当于Ant中的targets.不过你会发现他功能更加强大. 我们只是换了一个比target更形象的另外一个术语.不幸的是这恰巧与Ant中的术语有些冲突. ant 命令中有诸如javac、copy、tasks.所以当该文档中提及tasks时,除非特别指明 ant task. 否则指的均是指Gradle中的tasks.

If you think this looks similar to Ant's targets, well, you are right. Gradle tasks are the equivalent to Ant targets. But as you will see, they are much more powerful. We have used a different terminology than Ant as we think the word task is more expressive than the word target. Unfortunately this introduces a terminology clash with Ant, as Ant calls its commands, such as javac or copy , tasks. So when we talk about tasks, we always mean Gradle tasks, which are the equivalent to Ant's targets. If we talk about Ant tasks (Ant commands), we explicitly say ant task.

## **6.3. 快速定义任务**

6.3. A shortcut task definition

用一种更简洁的方式来定义 上面的 hello 任务.

There is a shorthand way to define a task like our hello task above, which is more concise.

例6.3.快速定义任务

Example 6.3. A task definition shortcut

```
build.gradle
task hello << {
    println 'Hello world!'
}
```

上面的脚本又一次采用闭包的方式来定义了一个叫做hello的任务,本文档后续章节中我们将会更多的采用这种风格来定义任务.

Again, this defines a task called hello with a single closure to execute. We will use this task definition style throughout the user guide.

## **6.4. 代码即脚本**

6.4. Build scripts are code

Gradle脚本采用Groovy书写,作为开胃菜,看下下面这个栗子. 

Gradle's build scripts expose to you the full power of Groovy. As an appetizer, have a look at this:

•例6.4 在gradle任务中采用groovy

Example 6.4. Using Groovy in Gradle's tasks

build.gradle

```
task upper << {
    String someString = 'mY_nAmE'
    println "Original: " + someString 
    println "Upper case: " + someString.toUpperCase()
}
```

Output of gradle -q upper

```
> gradle -q upper
Original: mY_nAmE
Upper case: MY_NAME

```

或者

or

•例6.5 在gradle任务中采用groovy

Example 6.5. Using Groovy in Gradle's tasks

build.gradle

```
task count << {
    4.times { print "$it " }
}
```

Output of gradle -q count

```
> gradle -q count
0 1 2 3
```

## **6.5. 任务依赖**

6.5. Task dependencies

你可以按如下方式创建任务间的依赖关系

As you probably have guessed, you can declare dependencies between your tasks.

在两个任务之间指明依赖关系

Example 6.6. Declaration of dependencies between tasks

build.gradle

```
task hello << {
    println 'Hello world!'
}
task intro(dependsOn: hello) << {
    println "I'm Gradle"
}
```

gradle -q intro的输出结果

Output of gradle -q intro

```
> gradle -q intro
Hello world!
I'm Gradle
```

添加依赖task也可以不必首先声明被依赖的task.

To add a dependency, the corresponding task does not need to exist.

Example 6.7. 延迟依赖 .Lazy dependsOn - the other task does not exist (yet)

build.gradle

```
task taskX(dependsOn: 'taskY') << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}
```

Output of gradle -q taskX

```
> gradle -q taskX
taskY
taskX
```

可以看到, taskX 是 在 taskY 之前定义的,这在多项目构建中非常有用 关于任务依赖的更多信息可以查看章节 15.4, “给任务添加依赖”..

The dependency of taskX to taskY is declared before taskY is defined. This is very important for multi-project builds. Task dependencies are discussed in more detail in Section 15.4, “Adding dependencies to a task”.

注意:当引用的任务尚未定义的时候不可使用短标记法(see 章节 6.8, “短标记法”) 来运行任务.

Please notice that you can't use shortcut notation (see Section 6.8, “Shortcut notations”) when referring to a task that is not yet defined.

## **6.6. 动态任务**

6.6. Dynamic tasks

借助Groovy的强大不仅可以定义简单任务还能做更多的事.例如,可以动态定义任务.

The power of Groovy can be used for more than defining what a task does. For example, you can also use it to dynamically create tasks.

例 6.8. 创建动态任务

Example 6.8. Dynamic creation of a task

build.gradle

```
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
```

gradle -q task1的输出结果.

Output of gradle -q task1

```
> gradle -q task1
I'm task number 1
```

## **6.7. 任务操纵**

6.7. Manipulating existing tasks

一旦任务被创建后,任务之间可以通过API进行相互访问.这也是与Ant的不同之处.比如可以增加一些依赖.

Once tasks are created they can be accessed via an API. This is different to Ant. For example you can create additional dependencies.

例 6.9. 通过API进行任务之间的通信 - 增加依赖

Example 6.9. Accessing a task via API - adding a dependency

build.gradle

```
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
task0.dependsOn task2, task3
```

gradle -q task0的输出结果.

Output of gradle -q task0

```
> gradle -q task0
I'm task number 2
I'm task number 3
I'm task number 0
```

为已存在的任务增加行为.

Or you can add behavior to an existing task.

例 6.10. 通过API进行任务之间的通信 - 增加任务行为

Example 6.10. Accessing a task via API - adding behaviour

build.gradle

```
task hello << {
    println 'Hello Earth'
}
hello.doFirst {
    println 'Hello Venus'
}
hello.doLast {
    println 'Hello Mars'
}
hello << {
    println 'Hello Jupiter'
}
```

Output of gradle -q hello

```
> gradle -q hello
Hello Venus
Hello Earth
Hello Mars
Hello Jupiter
```

doFirst 和 doLast 可以进行多次调用. 他们分别被添加在任务的开头和结尾.当任务开始执行时这些动作会按照既定顺序进行.其中 << 操作符 是 doLast 的简写方式.

The calls doFirst and doLast can be executed multiple times. They add an action to the beginning or the end of the task's actions list. When the task executes, the actions in the action list are executed in order. The << operator is simply an alias for doLast .

## **6.8. 短标记法**

6.8. Shortcut notations

你早就注意到了吧，没错，每个任务都是一个脚本的属性，你可以访问它:

As you might have noticed in the previous examples, there is a convenient notation for accessing an existing task. Each task is available as a property of the build script:

例 6.11. 以属性的方式访问任务

Example 6.11. Accessing task as a property of the build script

build.gradle

```
task hello << {
    println 'Hello world!'
}
hello.doLast {
    println "Greetings from the $hello.name task."
}
```

gradle -q hello的输出结果

Output of gradle -q hello

```
> gradle -q hello
Hello world!
Greetings from the hello task.
```

对于插件提供的内置任务。这尤其方便(例如:complie)

This enables very readable code, especially when using the out of the box tasks provided by the plugins (e.g. compile ).

## **6.9. 增加自定义属性**

6.9. Extra task properties

你可以为一个任务添加额外的属性.例如,新增一个叫做 myProperty 的属性,用 ext.myProperty 的方式给他一个初始值.这样便增加了一个自定义属性.

You can add your own properties to a task. To add a property named myProperty , set ext.myProperty to an initial value. From that point on, the property can be read and set like a predefined task property.

例 6.12. 为任务增加自定义属性
Example 6.12. Adding extra properties to a task

build.gradle

```
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties << {
    println myTask.myProperty
}
```

gradle -q printTaskProperties的输出结果

Output of gradle -q printTaskProperties

```
> gradle -q printTaskProperties
myValue
```

自定义属性不仅仅局限于任务上,还可以做更多事情.你可以阅读章节 13.4.2, “自定义属性”来了解更多内容.

Extra properties aren't limited to tasks. You can read more about them in Section 13.4.2, “Extra properties”.

## **6.10. 调用Ant任务**

6.10. Using Ant Tasks

Ant任务是Gradle中的一等公民.Gradle借助Groovy对Ant任务进行了优秀的整合. 
Gradle自带了一个AntBuilder,在Gradle中调用Ant任务比在 build.xml中调用更加的方便和强大. 
通过下面的例子你可以学到如何调用一个Ant任务以及如何与Ant中的属性进行通信.

Ant tasks are first-class citizens in Gradle. Gradle provides excellent integration for Ant tasks by simply relying on Groovy. Groovy is shipped with the fantastic AntBuilder . Using Ant tasks from Gradle is as convenient and more powerful than using Ant tasks from a build.xml file. From the example below, you can learn how to execute ant tasks and how to access ant properties:

例 6.13. 利用AntBuilder执行 ant.loadfile 

Example 6.13. Using AntBuilder to execute ant.loadfile target

build.gradle

```
task loadfile << {
    def files = file('../antLoadfileResources').listFiles().sort()
    files.each { File file ->
        if (file.isFile()) {
            ant.loadfile(srcFile: file, property: file.name)
            println " *** $file.name ***"
            println "${ant.properties[file.name]}"
        }
    }
}
```

gradle -q loadfile的输出结果 

Output of gradle -q loadfile

```
> gradle -q loadfile
*** agile.manifesto.txt ***
Individuals and interactions over processes and tools
Working software over comprehensive documentation
Customer collaboration  over contract negotiation
Responding to change over following a plan
 *** gradle.manifesto.txt ***
Make the impossible possible, make the possible easy and make the easy elegant.
(inspired by Moshe Feldenkrais)
```

在你脚本里还可以利用Ant做更多的事情.想了解更多请参阅 章节 17, 在Gradle中调用Ant

There is lots more you can do with Ant in your build scripts. You can find out more in Chapter 17, Using Ant from Gradle .

6.11. 方法抽取

6.11. Using methods

Gradle的强大要看你如何编写脚本逻辑.针对上面的例子,首先要做的就是要抽取方法. 

Gradle scales in how you can organize your build logic. The first level of organizing your build logic for the example above, is extracting a method.

例 6.14 利用方法组织脚本逻辑

Example 6.14. Using methods to organize your build logic

build.gradle

```
task checksum << {
    fileList('../antLoadfileResources').each {File file ->
        ant.checksum(file: file, property: "cs_$file.name")
        println "$file.name Checksum: ${ant.properties["cs_$file.name"]}"
    }
}

task loadfile << {
    fileList('../antLoadfileResources').each {File file ->
        ant.loadfile(srcFile: file, property: file.name)
        println "I'm fond of $file.name"
    }
}

File[] fileList(String dir) {
    file(dir).listFiles({file -> file.isFile() } as FileFilter).sort()
}
```

gradle -q loadfile的输出结果

Output of gradle -q loadfile

```
> gradle -q loadfile
I'm fond of agile.manifesto.txt
I'm fond of gradle.manifesto.txt
```

在后面的章节你会看到类似出去出来的方法可以在多项目构建中的子项目中调用. 无论构建逻辑多复杂,Gradle都可以提供给你一种简便的方式来组织它们.想了解更多请参阅 章节 59, 组织构建逻辑. 

Later you will see that such methods can be shared among subprojects in multi-project builds. If your build logic becomes more complex, Gradle offers you other very convenient ways to organize it. We have devoted a whole chapter to this. See Chapter 59, Organizing Build Logic.

## **6.12. 定义默认任务**

6.12. Default tasks

Gradle允许在脚本中定义多个默认任务. 

Gradle allows you to define one or more default tasks for your build.

例 6.15. 定义默认任务

Example 6.15. Defining a default tasks

build.gradle

```
defaultTasks 'clean', 'run'

task clean << {
    println 'Default Cleaning!'
}

task run << {
    println 'Default Running!'
}

task other << {
    println "I'm not a default task!"
}
```

gradle -q的输出结果.

Output of gradle -q

```
> gradle -q
Default Cleaning!
Default Running!
```

这与直接调用gradle clean run效果是一样的. 在多项目构建中,每个子项目都可以指定单独的默认任务.如果子项目未进行指定将会调用父项目指定的的默认任务. 

This is equivalent to running gradle clean run. In a multi-project build every subproject can have its own specific default tasks. If a subproject does not specify default tasks, the default tasks of the parent project are used (if defined).

## **6.13.  Configure by DAG**

稍后会对Gradle的配置阶段和运行阶段进行详细说明 (详见 章节 55, 构建的生命周期 )
配置阶段后,Gradle会了解所有要执行的任务 Gradle提供了一个钩子来捕获这些信息.一个例子就是可以检查已经执行的任务中有没有被释放.借由此,你可以为一些变量赋予不同的值. 

As we later describe in full detail (see Chapter 55, The Build Lifecycle), Gradle has a configuration phase and an execution phase. After the configuration phase, Gradle knows all tasks that should be executed. Gradle offers you a hook to make use of this information. A use-case for this would be to check if the release task is among the tasks to be executed. Depending on this, you can assign different values to some variables.

在下面的例子中,为distribution和release任务赋予了不同的 version值.

In the following example, execution of the distribution and release tasks results in different value of the version variable.

例 6.16. 依赖任务的不同输出

Example 6.16. Different outcomes of build depending on chosen tasks

build.gradle

```
task distribution << {
    println "We build the zip with version=$version"
}

task release(dependsOn: 'distribution') << {
    println 'We release now'
}

gradle.taskGraph.whenReady {taskGraph ->
    if (taskGraph.hasTask(release)) {
        version = '1.0'
    } else {
        version = '1.0-SNAPSHOT'
    }
}
```

gradle -q distribution的输出结果

Output of gradle -q distribution

```
> gradle -q distribution
We build the zip with version=1.0-SNAPSHOT
gradle -q release的输出结果
Output of gradle -q release
> gradle -q release
We build the zip with version=1.0
We release now

```

whenReady会在已发布的任务之前影响到已发布任务的执行. 即使已发布的任务不是 主要任务(也就是说,即使这个任务不是通过命令行直接调用)

The important thing is that whenReady affects the release task before the release task is executed. This works even when the release task is not the primary task (i.e., the task passed to the gradle command).

## **6.14. 下一步目标**

6.14. Where to next?

在本章中,我们了解了什么是task,但这还不够详细.欲知更多请参阅 章节 15, 任务进阶

In this chapter, we have had a first look at tasks. But this is not the end of the story for tasks. If you want to jump into more of the details, have a look at Chapter 15, More about Tasks .

另外,可以目录继续学习 第七章 , Java 快速入门 和 第八章 , 依赖管理基础.

Otherwise, continue on to the tutorials in Chapter 7, Java Quickstart and Chapter 8, Dependency Management Basics.


[2] 附录中有命令可以更改这种默认行为. 请参阅 附录 D, Gradle 命令行 

There are command line switches to change this behavior. See Appendix D, Gradle Command Line )
