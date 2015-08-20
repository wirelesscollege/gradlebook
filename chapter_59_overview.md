# **第59章 多项目构建**

Chapter 59. Multi-project Builds

强有力的支持多项目构建是Gradle的独有卖点之一,这个主题也是最有技术含量的.

The powerful support for multi-project builds is one of Gradle's unique selling points. This topic is also the most intellectually challenging.

在Gradle中一个多项目构建包含一个根项目和一个或多个子项目(也可能还有子项目)。

A multi-project build in gradle consists of one root project, and one or more subprojects that may also have subprojects.

## **59.1 跨项目配置**

59.1. Cross project configuration

虽然每个子项目能够在与其他子项目完全隔离下配置自身,这是子项目共同的特性。但通常更好的会在项目间共享配置,因为相同的配置会影响多个子项目。

While each subproject could configure itself in complete isolation of the other subprojects, it is common that subprojects share common traits. It is then usually preferable to share configurations among projects, so the same configuration affects several subprojects.

让我们开始一个非常简单的多项目构建。Gradle是一个支持通用构建的工具。因此项目并不一定要是Java项目。我们的第一个实例是关于marine life.

Let's start with a very simple multi-project build. Gradle is a general purpose build tool at its core, so the projects don't have to be Java projects. Our first examples are about marine life.

### **59.1.1. 配置和执行**

59.1.1. Configuration and execution

58.1节中，“Build phases”介绍了Gradle每次构建的过程。让我们放大一次多项目构建的配置和过程.此处的配置是指执行一个项目的build.gradle文件,这意味着如使用'apply plugin'声明下载所有插件.默认情况下,所有项目的配置会在执行任意task前执行.这意味着当从一个项目中发起一个单一任务请求时,需要先配置所有多项目构建。原因是为了支持灵活的访问和改变任何部分的Gradle项目模型

Section 55.1, “Build phases” describes the phases of every Gradle build. Let's zoom into the configuration and execution phases of a multi-project build. Configuration here means executing the build.gradle file of a project, which implies e.g. downloading all plugins that were declared using 'apply plugin'. By default, the configuration of all projects happens before any task is executed. This means that when a single task, from a single project is requested, all projects of multi-project build are configured first. The reason every project needs to be configured is to support the flexibility of accessing and changing any part of the Gradle project model.

59.1.1.1. 按需配置

59.1.1.1. Configuration on demand

由于每个项目在执行阶段之前都配置了，因此配置注入功能和访问完整项目模型都是可能的。然而，这种方法在一个非常大的多项目构建中可能不是最有效的。它们有Gradle构建层次上的子项目。庞大的多项目构建的配置时间可能会变得很明显。可扩展性是Gradle的重要要求。因此，从1.4版本开始引入了一种新的扩展“configuration on demand”.

The Configuration injection feature and access to the complete project model are possible because every project is configured before the execution phase. Yet, this approach may not be the most efficient in a very large multi-project build. There are Gradle builds with a hierarchy of hundreds of subprojects. The configuration time of huge multi-project builds may become noticeable. Scalability is an important requirement for Gradle. Hence, starting from version 1.4 a new incubating 'configuration on demand' mode is introduced.

按需配置模式尝试配置只有相关请求任务的项目配置，即只执行参与构建项目的build.gradle文件。这种方式，一个大型多项目构建的配置时间可以减少。从长远来看，该模式将变成默认模式，可能是Gradle构建执行的唯一模式。按需配置功能是扩展的所以不是每个构建都要保证是正常工作的。该功能进行有解耦的(59.9章节-“解耦项目”)多项目构建操作应该非常好。在“按需配置”模式下，项目配置如下：

Configuration on demand mode attempts to configure only projects that are relevant for requested tasks, i.e. it only executes the build.gradle file of projects that are participating in the build. This way, the configuration time of a large multi-project build can be reduced. In the long term, this mode will become the default mode, possibly the only mode for Gradle build execution. The configuration on demand feature is incubating so not every build is guaranteed to work correctly. The feature should work very well for multi-project builds that have decoupled projects (Section 59.9, “Decoupled Projects”). In “configuration on demand” mode, projects are configured as follows:

根项目总是始终配置。这种方式的典型配置支持(项目或子项目的脚本块)

The root project is always configured. This way the typical common configuration is supported (allprojects or subprojects script blocks).

在项目所在目录执行构建还是配置，但只有当Gradle是没有任何任务执行。这样项目的按需配置时，默认任务的行为方法才是正确的。

The project in the directory where the build is executed is also configured, but only when Gradle is executed without any tasks. This way the default tasks behave correctly when projects are configured on demand.

支持标准的项目依赖并使相关被项目配置。如果项目A有一个编译依赖项目B，然后在两个项目中构建A配置。

The standard project dependencies are supported and makes relevant projects configured. If project A has a compile dependency on project B then building A causes configuration of both projects。

通过任务路径声明的任务依赖的支持并导致相关的项目被配置。如：someTask.dependsOn(":someOtherProject:someOtherTask")

The task dependencies declared via task path are supported and cause relevant projects to be configured. Example: someTask.dependsOn(":someOtherProject:someOtherTask")

通过任务路径从命令行(或工具API)请求任务会导致相关的项目被配置。如构建'projectA:projectB:someTask'导致项目被配置

A task requested via task path from the command line (or Tooling API) causes the relevant project to be configured. For example, building 'projectA:projectB:someTask' causes configuration of projectB.

急于尝试一下这个新功能?每次构建都按需配置参见19.1章节，“通过gradle.properties配置编译环境”。按需配置仅对一个给定的构建参见附录D，Gradle命令行

Eager to try out this new feature? To configure on demand with every build run see Section 19.1, “Configuring the build environment via gradle.properties”. To configure on demand just for a given build please see Appendix D, Gradle Command Line.

### **56.1.2. 定义共同的行为**

56.1.2. Defining common behavior

让我们来看看以下项目树的一些例子.这是个名为water的根项目和bluewhale的子项目多项目构建

Let's look at some examples with the following project tree. This is a multi-project build with a root project named water and a subproject named bluewhale.

示例 56.1. 多项目树 - water & bluewhale项目

Example 56.1. Multi-project tree - water & bluewhale projects

构建布局
```
Build layout

water/
  build.gradle
  settings.gradle
  bluewhale/

```

注意:该例子的代码可以在Gradle目录的samples/userguide/multiproject/firstExample/water中找到.

Note: The code for this example can be found at samples/userguide/multiproject/firstExample/water in the ‘-all’ distribution of Gradle.

settings.gradle
```
  include 'bluewhale'
```

bluewhale项目的构建脚本在哪里?在Gradle构建脚本是可选的.显然对于一个项目的构建,一个项目没有构建脚本是没有太大意义的.对于多项目构建的情况则有所不同.让我们来看看water项目的构建脚本并执行它:

And where is the build script for the bluewhale project? In Gradle build scripts are optional. Obviously for a single project build, a project without a build script doesn't make much sense. For multiproject builds the situation is different. Let's look at the build script for the water project and execute it:

示例 56.2. water(parent)项目构建脚本

Example 56.2. Build script of water (parent) project

build.gradle
```
Closure cl = { task -> println "I'm $task.project.name" }
task hello << cl
project(':bluewhale') {
    task hello << cl
}
```

Output of gradle -q hello

```
> gradle -q hello

I'm water
I'm bluewhale
```

Gradle允许你从任意构建脚本访问多项目构建的任何项目.项目API提供一个叫project()的方法,需要一个路径做为参数并返回该路径对应的项目.从任意构建脚本配置一个项目构建的能力我们称之为跨项目配置.Gradle通过配置注入来实现.

Gradle allows you to access any project of the multi-project build from any build script. The Project API provides a method called project(), which takes a path as an argument and returns the Project object for this path. The capability to configure a project build from any build script we call cross project configuration. Gradle implements this via configuration injection.

我们是不满意water项目构建脚本的.它为每个项目添加明确的任务是不方便的.我们可以做到更好.让我们首先添加另一个叫krill的项目到我们的多项目构建.

We are not that happy with the build script of the water project. It is inconvenient to add the task explicitly for every project. We can do better. Let's first add another project called krill to our multi-project build.

示例 56.3. 多项目树 - water, bluewhale & krill projects

Example 56.3. Multi-project tree - water, bluewhale & krill projects

构建布局
```
Build layout

water/
  build.gradle
  settings.gradle
  bluewhale/
  krill/
```

注意:该例子的代码可以在Gradle目录的samples/userguide/multiproject/addKrill/water 中找到.

Note: The code for this example can be found at samples/userguide/multiproject/addKrill/water in the ‘-all’ distribution of Gradle.

settings.gradle
```
  include 'bluewhale', 'krill'
```

现在我们重写water构建脚本并把它合并成一行

Now we rewrite the water build script and boil it down to a single line.

示例 56.4. Water项目构建脚本

Example 56.4. Water project build script

build.gradle
```
allprojects {
    task hello << { task -> println "I'm $task.project.name" }
}
```

Output of gradle -q hello

```
> gradle -q hello
I'm water
I'm bluewhale
I'm krill
```

如何工作的呢?该项目API提供一个返回当前项目与其所有子项目列表的allprojects属性.如果你调用的allprojects是封闭的，封闭的状态是通过allprojects授权到与其相关联的项目的.你也可以通过allprojects.each做一个迭代，这样将更加详细.

Is this cool or is this cool? And how does this work? The Project API provides a property allprojects which returns a list with the current project and all its subprojects underneath it. If you call allprojects with a closure, the statements of the closure are delegated to the projects associated with allprojects. You could also do an iteration via allprojects.each, but that would be more verbose.

其他构建系统使用继承作为其确定共同行为的主要手段.我们提供的项目继承你将会在后面看到.但是Gradle使用配置注入来定义共同行为作为常用的方式.我们认为它提供了一个非常强大和灵活配置的多项目构建.

Other build systems use inheritance as the primary means for defining common behavior. We also offer inheritance for projects as you will see later. But Gradle uses configuration injection as the usual way of defining common behavior. We think it provides a very powerful and flexible way of configuring multiproject builds.

另一个possibilty用于使用一个共同的外部脚本来配置共享,更多信息参见59.3章节,"Configuring the project using an external build script"

Another possibilty for sharing configuration is to use a common external script. See Section 59.3, “Configuring the project using an external build script” for more information.

## **56.2. 子项目配置**

56.2. Subproject configuration

该项目API也提供一个只能访问子项目的属性

The Project API also provides a property for accessing the subprojects only.

### **56.2.1. 定义共同行为**

56.2.1. Defining common behavior

示例 56.5. 定义所有项目及其子项目的共同行为

Example 56.5. Defining common behavior of all projects and subprojects

build.gradle
```
allprojects {
    task hello << {task -> println "I'm $task.project.name" }
}
subprojects {
    hello << {println "- I depend on water"}
}
```

Output of gradle -q hello
```
> gradle -q hello
I'm water
I'm bluewhale
- I depend on water
I'm krill
- I depend on water
```

你可能会注意到有两段代码引用了"hello"任务.第一个使用"task"关键字来构建任务并提供基本配置.第二个地方不使用"task"关键字,因为它进一步配置现有的"hello"任务.你只能在一个项目中构建任务一次,但你可以用任意数量的代码块来提供额外的配置.

You may notice that there are two code snippets referencing the “hello” task. The first one, which uses the “task” keyword, constructs the task and provides it's base configuration. The second piece doesn't use the “task” keyword, as it is further configuring the existing “hello” task. You may only construct a task once in a project, but you may any number of code blocks providing additional configuration.

## **56.2.2. 添加指定的行为**

56.2.2. Adding specific behavior

你可以在共同的行为上添加指定行为.通常我们把项目指定的行为放在我们想应用该指定行为的项目构建脚本中.但正如我们已看到的,我们不必这样做.我们可以像这样添加bluewhale项目的指定行为:

You can add specific behavior on top of the common behavior. Usually we put the project specific behavior in the build script of the project where we want to apply this specific behavior. But as we have already seen, we don't have to do it this way. We could add project specific behavior for the bluewhale project like this:

示例 56.6. 为特定项目定义指定行为

Example 56.6. Defining specific behaviour for particular project

build.gradle
```
allprojects {
    task hello << {task -> println "I'm $task.project.name" }
}
subprojects {
    hello << {println "- I depend on water"}
}
project(':bluewhale').hello << {
    println "- I'm the largest animal that has ever lived on this planet."
}
```

Output of gradle -q hello

```
> gradle -q hello
I'm water
I'm bluewhale
- I depend on water
- I'm the largest animal that has ever lived on this planet.
I'm krill
- I depend on water
```

正在我们所说,我们通常喜欢把项目指定行为写入到该项目的构建脚本中.让我们对krill项目重构并添加一些项目的指定行为.

As we have said, we usually prefer to put project specific behavior into the build script of this project. Let's refactor and also add some project specific behavior to the krill project.

示例 56.7. 为krill项目定义指定行为

Example 56.7. Defining specific behaviour for project krill

```
Build layout
water/
  build.gradle
  settings.gradle
  bluewhale/
    build.gradle
  krill/
    build.gradle
```

注意:该例子的代码可以在Gradle目录的samples/userguide/multiproject/spreadSpecifics/water 中找到.

Note: The code for this example can be found at samples/userguide/multiproject/spreadSpecifics/water in the ‘-all’ distribution of Gradle.

settings.gradle
```
  include 'bluewhale', 'krill'
```

bluewhale/build.gradle

```
hello.doLast {
  println "- I'm the largest animal that has ever lived on this planet."
}
krill/build.gradle
hello.doLast {
  println "- The weight of my species in summer is twice as heavy as all human beings."
}
build.gradle
allprojects {
    task hello << {task -> println "I'm $task.project.name" }
}
subprojects {
    hello << {println "- I depend on water"}
}
```

Output of gradle -q hello

```
> gradle -q hello
I'm water
I'm bluewhale
- I depend on water
- I'm the largest animal that has ever lived on this planet.
I'm krill
- I depend on water
- The weight of my species in summer is twice as heavy as all human beings.
```

## **56.2.3. 项目过滤**

56.2.3. Project filtering

为了显示更多更强大的注入配置,让我们添加另外一个叫tropicalFish的项目和通过water项目的构建脚本添加更多的行为.

To show more of the power of configuration injection, let's add another project called tropicalFish and add more behavior to the build via the build script of the water project.

56.2.3.1. 通过名字过滤

56.2.3.1. Filtering by name

示例 56.8. 为一些项目添加自定义行为(通过项目名过滤)

Example 56.8. Adding custom behaviour to some projects (filtered by project name)

构建布局

Build layout

```
water/
  build.gradle
  settings.gradle
  bluewhale/
    build.gradle
  krill/
    build.gradle
  tropicalFish/
```

注意:该例子的代码可以在Gradle目录的samples/userguide/multiproject/addTropical/water 中找到.  

Note: The code for this example can be found at samples/userguide/multiproject/addTropical/water in the ‘-all’ distribution of Gradle.

settings.gradle
```
  include 'bluewhale', 'krill', 'tropicalFish'
```

build.gradle
```
allprojects {
    task hello << {task -> println "I'm $task.project.name" }
}
subprojects {
    hello << {println "- I depend on water"}
}
configure(subprojects.findAll {it.name != 'tropicalFish'}) {
    hello << {println '- I love to spend time in the arctic waters.'}
}
```

Output of gradle -q hello

```
> gradle -q hello
I'm water
I'm bluewhale
- I depend on water
- I love to spend time in the arctic waters.
- I'm the largest animal that has ever lived on this planet.
I'm krill
- I depend on water
- I love to spend time in the arctic waters.
- The weight of my species in summer is twice as heavy as all human beings.
I'm tropicalFish
- I depend on water
```

该configure()方法接收一个列表作为参数和在该列表中为该项目配置应用.

The configure() method takes a list as an argument and applies the configuration to the projects in this list.

56.2.3.2. 通过属性过滤

56.2.3.2. Filtering by properties

使用项目名过滤是一种选择.使用额外的项目属性是另外一回事.(关于extra properties更多信息请参见13.4.2章节,"Extra properties")

Using the project name for filtering is one option. Using extra project properties is another. (See Section 13.4.2, “Extra properties” for more information on extra properties.)

示例 56.9. 为一些项目添加自定义行为(通过项目属性过滤)

Example 56.9. Adding custom behaviour to some projects (filtered by project properties)

构建布局

Build layout

```
water/
  build.gradle
  settings.gradle
  bluewhale/
    build.gradle
  krill/
    build.gradle
  tropicalFish/
    build.gradle
```

注意:该例子的代码可以在Gradle目录的samples/userguide/multiproject/tropicalWithProperties/water中找到.

Note: The code for this example can be found at samples/userguide/multiproject/tropicalWithProperties/water in the ‘-all’ distribution of Gradle.

settings.gradle
```
  include 'bluewhale', 'krill', 'tropicalFish'
```

bluewhale/build.gradle

```
ext.arctic = true
hello.doLast {
  println "- I'm the largest animal that has ever lived on this planet."
}

krill/build.gradle

ext.arctic = true
hello.doLast {
    println "- The weight of my species in summer is twice as heavy as all human beings."
}

tropicalFish/build.gradle

ext.arctic = false
```

build.gradle

```
allprojects {
    task hello << {task -> println "I'm $task.project.name" }
}
subprojects {
    hello {
        doLast {println "- I depend on water"}
        afterEvaluate { Project project ->
            if (project.arctic) { doLast {
                println '- I love to spend time in the arctic waters.' }
            }
        }
    }
}
```

Output of gradle -q hello

```
> gradle -q hello
I'm water
I'm bluewhale
- I depend on water
- I'm the largest animal that has ever lived on this planet.
- I love to spend time in the arctic waters.
I'm krill
- I depend on water
- The weight of my species in summer is twice as heavy as all human beings.
- I love to spend time in the arctic waters.
I'm tropicalFish
- I depend on water
```


在water项目的构建文件中我们使用一个afterEvaluate通知.这意味着经过我们评估后,子项目的构建脚本才会进行评估.在这些构建脚本中设置属性,我们需要这样做.你将会在56.6章节这个主题上发现更多“Dependencies - Which dependencies?”

In the build file of the water project we use an afterEvaluate notification. This means that the closure we are passing gets evaluated after the build scripts of the subproject are evaluated. As the property arctic is set in those build scripts, we have to do it this way. You will find more on this topic in Section 56.6, “Dependencies - Which dependencies?”

56.3. 多项目构建的执行规则

56.3. Execution rules for multi-project builds

当我们从根项目目录下执行了hello任务,都会很直接的表现出来.所有hello任务在不同项目中被执行.让我们切换到bluewhale目录并看看我们从那里执行Gradle会发生什么.

When we executed the hello task from the root project dir, things behaved in an intuitive way. All the hello tasks of the different projects were executed. Let's switch to the bluewhale dir and see what happens if we execute Gradle from there.

示例 56.10. 从子项目运行构建

Example 56.10. Running build from subproject

Output of gradle -q hello

> gradle -q hello

```
I'm bluewhale
- I depend on water
- I'm the largest animal that has ever lived on this planet.
- I love to spend time in the arctic waters.
```

Gradle行为后的基本规则是很简单的.Gradle看下面的层次,首先是找到当前目录的hello任务并执行它.有一点非常重要的要注意,Gradle总是会评估多项目构建的每个项目和创建所有现有任务对性爱那个.然后,根据任务名称参数和当前路径,Gradle过滤应该被执行的任务.因为Gradle跨项目配置的每一个项目在执行前都有进行评估.我们在下一章节中将仔细看这个.现在让我们做最后一个例子.让我们为bluewhale和krill添加一个任务.

The basic rule behind Gradle's behavior is simple. Gradle looks down the hierarchy, starting with the current dir, for tasks with the name hello and executes them. One thing is very important to note. Gradle always evaluates every project of the multi-project build and creates all existing task objects. Then, according to the task name arguments and the current dir, Gradle filters the tasks which should be executed. Because of Gradle's cross project configuration every project has to be evaluated before any task gets executed. We will have a closer look at this in the next section. Let's now have our last marine example. Let's add a task to bluewhale and krill.

示例 56.11. 评估和执行项目

Example 56.11. Evaluation and execution of projects

bluewhale/build.gradle

```
  ext.arctic = true

hello << { println "- I'm the largest animal that has ever lived on this planet." }

task distanceToIceberg << {
    println '20 nautical miles'
}
```

krill/build.gradle

```
ext.arctic = true
hello << {
    println "- The weight of my species in summer is twice as heavy as all human beings."
}

task distanceToIceberg << {
    println '5 nautical miles'
}
```

Output of gradle -q distanceToIceberg
```
> gradle -q distanceToIceberg
20 nautical miles
5 nautical miles
```

这里没有输出-q选项

Here's the output without the -q option:

示例 56.12. 评估和执行项目

Example 56.12. Evaluation and execution of projects

输出 gradle distanceToIceberg

Output of gradle distanceToIceberg

```
> gradle distanceToIceberg
:bluewhale:distanceToIceberg
20 nautical miles
:krill:distanceToIceberg
5 nautical miles

BUILD SUCCESSFUL

Total time: 1 secs
```

该构建从water项目中执行.无论是water还是tropicalFish都一个名称为distanceToIceberg的任务.Gradle并不关心.上面已经提到的简单规则:在项目层次所有任务下执行对应名称的任务.只是抱怨没有这样的任务!

The build is executed from the water project. Neither water nor tropicalFish have a task with the name distanceToIceberg. Gradle does not care. The simple rule mentioned already above is: Execute all tasks down the hierarchy which have this name. Only complain if there is no such task!

## **56.4. 通过绝对路径运行任务**

56.4. Running tasks by their absolute path

正如我们所见,你可以通过输入任何子项目的路径和从该路径执行构建来运行一个多项目构建.从当前路径的项目层次的所有匹配的任务将被执行.但是Gradle也提供通过绝对路径来执行任务(参见56.5章节,“Project and task paths”)

As we have seen, you can run a multi-project build by entering any subproject dir and execute the build from there. All matching task names of the project hierarchy starting with the current dir are executed. But Gradle also offers to execute tasks by their absolute path (see also Section 56.5, “Project and task paths”):

示例 56.13. 通过它们的绝对路径运行任务

Example 56.13. Running tasks by their absolute path

Output of gradle -q :hello :krill:hello hello

```
> gradle -q :hello :krill:hello hello
I'm water
I'm krill
- I depend on water
- The weight of my species in summer is twice as heavy as all human beings.
- I love to spend time in the arctic waters.
I'm tropicalFish
- I depend on water
```

该构建从tropicalFish项目中执行.我们执行water,krill和tropicalFish项目的hello任务.前两项都指定任务的绝对路径,最后任务使用上述名称匹配机制来执行

The build is executed from the tropicalFish project. We execute the hello tasks of the water, the krill and the tropicalFish project. The first two tasks are specified by their absolute path, the last task is executed using the name matching mechanism described above.

## **56.5. 项目和任务路径**

56.5. Project and task paths

一个项目路径有以下模式:它首先是一个可选的冒号,它表示根项目.根项目是没有通过它名称指定路径的唯一项目.项目路径的其余部分是项目名称,这里的下一个项目是以冒号分割的子项目.

A project path has the following pattern: It starts with an optional colon, which denotes the root project. The root project is the only project in a path that is not specified by its name. The rest of a project path is a colon-separated sequence of project names, where the next project is a subproject of the previous project.

任务的路径就是其项目路径加上任务的名称,像":bluewhale:hello".你可以用它的名字来处理同一个项目的任务.这被解释为相对路径。
The path of a task is simply its project path plus the task name, like “:bluewhale:hello”. Within a project you can address a task of the same project just by its name. This is interpreted as a relative path.

## **56.6. 依赖 - 哪个依赖?**

56.6. Dependencies - Which dependencies?

最后一节的例子是特殊的,因为这个项目没有执行依赖.他们只有配置依赖.下面的部分说明了这2种类型的依赖关系。

The examples from the last section were special, as the projects had no Execution Dependencies. They had only Configuration Dependencies. The following sections illustrate the differences between these two types of dependencies.

### **56.6.1. 执行依赖**

56.6.1. Execution dependencies

56.6.1.1. 依赖关系和执行顺序

56.6.1.1. Dependencies and execution order

示例 56.14. 依赖关系和执行顺序

Example 56.14. Dependencies and execution order

构建布局

Build layout
```
messages/
  settings.gradle
  consumer/
    build.gradle
  producer/
    build.gradle
```

注意:该例子的代码可以在Gradle目录的samples/userguide/multiproject/dependencies/firstMessages/中找到.    

Note: The code for this example can be found at samples/userguide/multiproject/dependencies/firstMessages/messages in the ‘-all’ distribution of Gradle.

settings.gradle
```
  include 'consumer', 'producer'
```

consumer/build.gradle
```
task action << {
    println("Consuming message: ${rootProject.producerMessage}")
}
```

producer/build.gradle
```
task action << {
    println "Producing message:"
    rootProject.producerMessage = 'Watch the order of execution.'
}
```

Output of gradle -q action

```
> gradle -q action
Consuming message: null
Producing message:
```

这并没有完全做到我们想要的.如果没有其他的定义,Gradle按字母顺序执行任务.因此,Gradle将在执行“:aproducer:action”前执行“:consumer:action”.让我们尝试破解这个并重命名producer为"aproducer"

This didn't quite do what we want. If nothing else is defined, Gradle executes the task in alphanumeric order. Therefore, Gradle will execute “:consumer:action” before “:aproducer:action”. Let's try to solve this with a hack and rename the producer project to “aProducer”.

示例 56.15. 依赖关系和执行顺序

Example 56.15. Dependencies and execution order

构建布局
```
Build layout

messages/
  settings.gradle

  aProducer/
    build.gradle

  consumer/
    build.gradle
```

settings.gradle
```
  include 'consumer', 'aProducer'
```

aProducer/build.gradle
```
task action << {
    println "Producing message:"
    rootProject.producerMessage = 'Watch the order of execution.'
}
```

consumer/build.gradle
```
task action << {
    println("Consuming message: ${rootProject.producerMessage}")
}
```

Output of gradle -q action
```
> gradle -q action
Producing message:
Consuming message: Watch the order of execution.
```

如果我们现在切换到consumer目录下执行构建,我们可以看到该破解是不工作的.

We can show where this hack doesn't work if we now switch to the consumer dir and execute the build.

示例 56.16. 依赖关系和执行顺序

Example 56.16. Dependencies and execution order

Output of gradle -q action

```
> gradle -q action
Consuming message: null
```

问题是两者的"action"任务是无关的.如果你从"message"项目执行这个构建,Gradle执行它们两个因为它们有相同的名字和它们的层次.在最后的例子中只有一个"action"任务是在这个层次下的,因此它是已执行的唯一的任务.我们需要一些比破解更好的东西.

The problem is that the two “action” tasks are unrelated. If you execute the build from the “messages” project Gradle executes them both because they have the same name and they are down the hierarchy. In the last example only one “action” task was down the hierarchy and therefore it was the only task that was executed. We need something better than this hack.

56.6.1.2. 声明依赖

56.6.1.2. Declaring dependencies

示例56.17. 声明依赖

Example 56.17. Declaring dependencies

构建布局

Build layout
```
messages/
  settings.gradle
  consumer/
    build.gradle
  producer/
    build.gradle
```

注意:该例子的代码可以在Gradle目录的samples/userguide/multiproject/dependencies/messagesWithDependencies/messages中找到. 

Note: The code for this example can be found at samples/userguide/multiproject/dependencies/messagesWithDependencies/messages in the ‘-all’ distribution of Gradle.

settings.gradle
```
  include 'consumer', 'producer'
```

consumer/build.gradle
```
task action(dependsOn: ":producer:action") << {
    println("Consuming message: ${rootProject.producerMessage}")
}
```

producer/build.gradle
```
task action << {
    println "Producing message:"
  
    rootProject.producerMessage = 'Watch the order of execution.'
}
```

Output of gradle -q action

```
> gradle -q action
Producing message:
Consuming message: Watch the order of execution.
```

从consumer目录运行得到:

Running this from the consumer directory gives:

示例 56.18. 声明依赖

Example 56.18. Declaring dependencies

Output of gradle -q action
```
> gradle -q action
Producing message:
Consuming message: Watch the order of execution.
```

这是现在更好的工作方式,因为在“consumer”项目中声明的“action”的任务有一个执行依赖于“producer”项目中的“action”的任务.

This is now working better because we have declared that the “action” task in the “consumer” project has an execution dependency on the “action” task in the “producer” project.

56.6.1.3. 跨项目任务依赖的性质

56.6.1.3. The nature of cross project task dependencies

当然,不同项目的任务依赖并不局限于同一个名称的任务.让我们改变我们的任务的名称并执行构建.

Of course, task dependencies across different projects are not limited to tasks with the same name. Let's change the naming of our tasks and execute the build.

示例 56.19. 跨项目任务的依赖

Example 56.19. Cross project task dependencies

consumer/build.gradle
```
task consume(dependsOn: ':producer:produce') << {
    println("Consuming message: ${rootProject.producerMessage}")
}
```

producer/build.gradle
```
task produce << {
    println "Producing message:"
    rootProject.producerMessage = 'Watch the order of execution.'
}
```

Output of gradle -q consume
```
> gradle -q consume
Producing message:
Consuming message: Watch the order of execution.
```

### **56.6.2. 配置时间依赖**

56.6.2. Configuration time dependencies

在我们进入Java部分时让我们先再看一个producer-consumer构建的例子.我们增加一个属性到“producer”项目并从“consumer”到“producer”创建一个时间依赖配置.

Let's see one more example with our producer-consumer build before we enter Java land. We add a property to the “producer” project and create a configuration time dependency from “consumer” to “producer”.

示例 56.20. 配置时间依赖

Example 56.20. Configuration time dependencies

consumer/build.gradle

```
def message = rootProject.producerMessage

task consume << {
    println("Consuming message: " + message)
}
```

producer/build.gradle

```
rootProject.producerMessage = 'Watch the order of evaluation.'
```

输出gradle -q consume

Output of gradle -q consume

```
> gradle -q consume
Consuming message: null
```

项目默认的评价顺序是字母数字(相同级别).因此“consumer”项目是在“producer”项目前和“producerMessage”值设置后，它是通过“consumer”来读取的。Gradle提供了一个解决方案。

The default evaluation order of projects is alphanumeric (for the same nesting level). Therefore the “consumer” project is evaluated before the “producer” project and the “producerMessage” value is set after it is read by the “consumer” project. Gradle offers a solution for this.

示例 56.21. 配置时间依赖 - evaluationDependsOn

Example 56.21. Configuration time dependencies - evaluationDependsOn

consumer/build.gradle
```
evaluationDependsOn(':producer')

def message = rootProject.producerMessage

task consume << {
    println("Consuming message: " + message)
}
```

输出gradle -q consume 

Output of gradle -q consume
```
> gradle -q consume
Consuming message: Watch the order of evaluation.
```

使用“evaluationDependsOn”命令的结果是在“producer”项目前“consumer”项目已被评估。该例子是一个做作的显示机制。在这种情况下，在执行时读取关键属性更容易解决。

The use of the “evaluationDependsOn” command results in the evaluation of the “producer” project before the “consumer” project is evaluated. This example is a bit contrived to show the mechanism. In this case there would be an easier solution by reading the key property at execution time.

示例 56.22. 配置时间依赖

Example 56.22. Configuration time dependencies

consumer/build.gradle
```
task consume << {
    println("Consuming message: ${rootProject.producerMessage}")
}
```

输出 gradle -q consume

Output of gradle -q consume
```
> gradle -q consume
Consuming message: Watch the order of evaluation.
```

执行依赖和配置依赖是不一样的。配置依赖是项目之间的关系，而执行依赖总是解决任务的依赖关系的。还要主要所有的项目都配置，甚至当你从一个子项目开始构建。默认配置顺序是自顶向下的，这是通常的需要。

Configuration dependencies are very different from execution dependencies. Configuration dependencies are between projects whereas execution dependencies are always resolved to task dependencies. Also note that all projects are always configured, even when you start the build from a subproject. The default configuration order is top down, which is usually what is needed.

更改默认配置为“自下而上”，使用“evaluationDependsOnChildren()”代替。

To change the default configuration order to “bottom up”, use the “evaluationDependsOnChildren()” method instead.

在相同的嵌套中配置顺序依赖字符的位置。最常见的例子是在多项目构建中共享一个生命周期(如.所有项目使用java插件)。如果你声明依赖取决于不同项目的执行依赖，该方法的默认行为是创建在两个项目之间的配置依赖。因此它很可能是你没有明确定义配置的依赖关系。

On the same nesting level the configuration order depends on the alphanumeric position. The most common use case is to have multi-project builds that share a common lifecycle (e.g. all projects use the Java plugin). If you declare with dependsOn a execution dependency between different projects, the default behavior of this method is to also create a configuration dependency between the two projects. Therefore it is likely that you don't have to define configuration dependencies explicitly.

### **56.6.3. 现实生活示例**

56.6.3. Real life examples

Gradle的多项目特点是通过现实生活中的用例驱动的。一个很好的例子包括两个web应用项目和一个创建分布包括两个web应用的父项目。[21]例如我们只使用一个构建脚本和进行跨项目配置。

Gradle's multi-project features are driven by real life use cases. One good example consists of two web application projects and a parent project that creates a distribution including the two web applications. [21] For the example we use only one build script and do cross project configuration.

示例 56.23. 依赖 - 现实生活例子 - 跨项目配置

Example 56.23. Dependencies - real life example - crossproject configuration

构建布局

Build layout
```
webDist/
  settings.gradle
  build.gradle
  date/
    src/main/java/
      org/gradle/sample/
        DateServlet.java
  hello/
    src/main/java/
      org/gradle/sample/
        HelloServlet.java
```

注意： 该示例代码可以在Gradle根目录下的samples/userguide/multiproject/dependencies/webDist中找到。        

Note: The code for this example can be found at samples/userguide/multiproject/dependencies/webDist in the ‘-all’ distribution of Gradle.

settings.gradle
```
include 'date', 'hello'
```

build.gradle
```
allprojects {
    apply plugin: 'java'
    group = 'org.gradle.sample'
    version = '1.0'
}

subprojects {
    apply plugin: 'war'
    repositories {
        mavenCentral()
    }
    dependencies {
        compile "javax.servlet:servlet-api:2.5"
    }
}

task explodedDist(type: Copy) {
    into "$buildDir/explodedDist"
    subprojects {
        from tasks.withType(War)
    }
}
```

我们有一个有趣的依赖集。显然date和hello项目对webDist有一个配置依赖，因为webapp项目所有的构建逻辑是通过webDist注入的。执行依赖是另一个方向，为webDist依赖date和hello的构建产物。甚至有三分之一的依赖。webDist在date和hello有一个配置依赖因为它需要知道archivePath。但它要求执行时间的信息。因此我们没有循环依赖。

We have an interesting set of dependencies. Obviously the date and hello projects have a configuration dependency on webDist, as all the build logic for the webapp projects is injected by webDist. The execution dependency is in the other direction, as webDist depends on the build artifacts of date and hello. There is even a third dependency. webDist has a configuration dependency on date and hello because it needs to know the archivePath. But it asks for this information at execution time. Therefore we have no circular dependency.

这种依赖模式是多项目构建的问题空间。如果一个构建系统不支持这些模式，你要么无法解决你的问题，要么需要你做丑陋的黑客，这样作为构建主都难以维持和会大规模影响的工作效率。

Such dependency patterns are daily bread in the problem space of multi-project builds. If a build system does not support these patterns, you either can't solve your problem or you need to do ugly hacks which are hard to maintain and massively impair your productivity as a build master.

## **56.7. 项目库依赖**

56.7. Project lib dependencies

如果一个项目需要通过另一个项目在其编译路径生成的jar,且不仅仅是jar,也要该jar的传递依赖?显然,这是java多项目构建的一种非常常见用例.前面在50.4.3章节-“Project dependencies”已经提到,Gradle提供项目库依赖这一点.

What if one project needs the jar produced by another project in its compile path, and not just the jar but also the transitive dependencies of this jar? Obviously this is a very common use case for Java multi-project builds. As already mentioned in Section 50.4.3, “Project dependencies”, Gradle offers project lib dependencies for this.

示例 56.24. 项目库依赖

Example 56.24. Project lib dependencies

构建布局

Build layout

```
java/
  settings.gradle
  build.gradle
  api/
    src/main/java/
      org/gradle/sample/
        api/
          Person.java
        apiImpl/
          PersonImpl.java
  services/personService/
    src/
      main/java/
        org/gradle/sample/services/
          PersonService.java
      test/java/
        org/gradle/sample/services/
          PersonServiceTest.java
  shared/
    src/main/java/
      org/gradle/sample/shared/
        Helper.java
```

注意： 该示例代码可以在Gradle根目录下的samples/userguide/multiproject/dependencies/java中找到.我们有项目"shared","api"和"personService".“personService”项目在其他两个项目上有一个库依赖,“api”项目在“shared”项目上有一个库依赖. [22]

Note: The code for this example can be found at samples/userguide/multiproject/dependencies/java in the ‘-all’ distribution of Gradle.
We have the projects “shared”, “api” and “personService”. The “personService” project has a lib dependency on the other two projects. The “api” project has a lib dependency on the “shared” project. [22]

示例 56.25. 项目库依赖

Example 56.25. Project lib dependencies

settings.gradle
```
	include 'api', 'shared', 'services:personService'
```

build.gradle
```
subprojects {
    apply plugin: 'java'
    group = 'org.gradle.sample'
    version = '1.0'
    repositories {
        mavenCentral()
    }
    dependencies {
        testCompile "junit:junit:4.12"
    }
}

project(':api') {
    dependencies {
        compile project(':shared')
    }
}

project(':services:personService') {
    dependencies {
        compile project(':shared'), project(':api')
    }
}
```

所有的构建逻辑是在根项目的“build.gradle”文件中. [23] 一个"库"依赖是一个执行依赖的特殊形式.它会导致第一个构建的其他项目增加与其他项目类路径上的jar.它还增加了其他项目类路径的依赖.所以,你可以进入“api”路径并引发“gradle编译”.首先是“shared”项目构建后“api”项目再构建.项目依赖启动部分多项目构建.

All the build logic is in the “build.gradle” file of the root project. [23] A “lib” dependency is a special form of an execution dependency. It causes the other project to be built first and adds the jar with the classes of the other project to the classpath. It also adds the dependencies of the other project to the classpath. So you can enter the “api” directory and trigger a “gradle compile”. First the “shared” project is built and then the “api” project is built. Project dependencies enable partial multi-project builds.

如果你来自Maven的地方这样你可能很快乐.如果你来自Ivy的地方,你可能会想到更多更精细的控制.Gradle提供这个给你：

If you come from Maven land you might be perfectly happy with this. If you come from Ivy land, you might expect some more fine grained control. Gradle offers this to you:

示例 56.26. 细粒度的控制依赖

Example 56.26. Fine grained control over dependencies

build.gradle

```
subprojects {
    apply plugin: 'java'
    group = 'org.gradle.sample'
    version = '1.0'
}

project(':api') {
    configurations {
        spi
    }
    dependencies {
        compile project(':shared')
    }
    task spiJar(type: Jar) {
        baseName = 'api-spi'
        dependsOn classes
        from sourceSets.main.output
        include('org/gradle/sample/api/**')
    }
    artifacts {
        spi spiJar
    }
}

project(':services:personService') {
    dependencies {
        compile project(':shared')
        compile project(path: ':api', configuration: 'spi')
        testCompile "junit:junit:4.12", project(':api')
    }
}
```

默认情况下Java插件每增加一个jar到你的项目库中包含所有的类.在该示例中我们创建一个额外的仅包含接口的“api”项目。我们这个库分配到一个新的依赖配置中。对于person的服务，我们声明该项目应只对“api”接口进行编译但会测试“api”中所有的类。

The Java plugin adds per default a jar to your project libraries which contains all the classes. In this example we create an additional library containing only the interfaces of the “api” project. We assign this library to a new dependency configuration. For the person service we declare that the project should be compiled only against the “api” interfaces but tested with all classes from “api”.

### **56.7.1. 禁用依赖项目的构建**

56.7.1. Disabling the build of dependency projects

有时候你做一个部分构建时不想依赖项目进行构建。要禁用项目依赖的构建你可以运行gradle -a 选项。

Sometimes you don't want depended on projects to be built when doing a partial build. To disable the build of the depended on projects you can run Gradle with the -a option.

## **56.8. 并行项目执行**

56.8. Parallel project execution

随着台式机和CI服务器越来越多的cpu内核，Gradle能够充分利用这些资源是很重要的。更具体的说，并行执行的尝试：

With more and more CPU cores available on developer desktops and CI servers, it is important that Gradle is able to fully utilise these processing resources. More specifically, the parallel execution attempts to:

减少多项目构建的总构建时间，其中是执行IO还是用其他不占用所有可用CPU资源的方式。

Reduce total build time for a multi-project build where execution is IO bound or otherwise does not consume all available CPU resources.

为执行小项目提供更快的反馈意见不必等待其他项目的完成。

Provide faster feedback for execution of small projects without awaiting completion of other projects.

虽然Gradle已提供并行测试执行如Test.setMaxParallelForks() 在该章节中描述的功能是并行执行项目的级别.并行执行是一个孵化功能。请使用它并让我们知道它是如何为你工作的。

Although Gradle already offers parallel test execution via Test.setMaxParallelForks() the feature described in this section is parallel execution at a project level. Parallel execution is an incubating feature. Please use it and let us know how it works for you.

并行项目执行允许在解耦的多项目构建独立项目(参见 56.9章节- “Decoupled Projects)。同时并行执行并不严格要求在配置时解耦,它的长期目标是提供一套强大的功能，将可用于完全解耦项目。这种功能包括：

Parallel project execution allows the separate projects in a decoupled multi-project build to be executed in parallel (see also: Section 56.9, “Decoupled Projects”). While parallel execution does not strictly require decoupling at configuration time, the long-term goal is to provide a powerful set of features that will be available for fully decoupled projects. Such features include:

56.1.1.1章节，“需求配置” 。

Section 56.1.1.1, “Configuration on demand”.

并行项目配置。

Configuration of projects in parallel.

为不变的项目重新使用配置。

Re-use of configuration for unchanged projects.

项目级别的最新检查。

Project-level up-to-date checks.

在构建相关项目的地方使用预构建。

Using pre-built artifacts in the place of building dependent projects.

如何并行执行工作的？首先，你需要告诉Gradle使用并行模式。你可以使用命令行参数(附录D，Gradle命令行)或配置你的构建环境变量(19.1章节，“通过 gradle.properties配置构建环境变量”)。除非你提供一些指定数量并行线程，Gradle试图选择基于可用的CPU的正确数量。每个平行的工作仅拥有一个给定的项目。这意味着从同一个项目中的2个任务从来没有并行执行。因此只有多项目构建能够使用并行执行。任务的依赖是完全支持的并将开始并行执行前面的任务。请记住，对分离的任务，按字母排序的顺序执行并不是在并行模式下工作。你需要确保任务依赖正确声明以避免排序问题。

How does parallel execution work? First, you need to tell Gradle to use the parallel mode. You can use the command line argument (Appendix D, Gradle Command Line) or configure your build environment (Section 19.1, “Configuring the build environment via gradle.properties”). Unless you provide a specific number of parallel threads Gradle attempts to choose the right number based on available CPU cores. Every parallel worker exclusively owns a given project while executing a task. This means that 2 tasks from the same project are never executed in parallel. Therefore only multi-project builds can take advantage of parallel execution. Task dependencies are fully supported and parallel workers will start executing upstream tasks first. Bear in mind that the alphabetical scheduling of decoupled tasks, known from the sequential execution, does not really work in parallel mode. You need to make sure the task dependencies are declared correctly to avoid ordering issues.

## **56.9. 解耦项目**

56.9. Decoupled Projects

Gradle允许任何项目在配置和执行阶段访问其他项目。虽然这提供一个强大的处理能力和灵活的构建者，这也限制了Gradle构建这些项目的灵活性。例如，有效的防止Gradle从正确的多项目中构建并行，仅配置项目的一个子集或在项目依赖的部分地方替代预构建。

Gradle allows any project to access any other project during both the configuration and execution phases. While this provides a great deal of power and flexibility to the build author, it also limits the flexibility that Gradle has when building those projects. For instance, this effectively prevents Gradle from correctly building multiple projects in parallel, configuring only a subset of projects, or from substituting a pre-built artifact in place of a project dependency.

如果不能直接访问对方的项目模型,则这两个项目则称为接口的项目。接口的项目可以只声明依赖关系：项目依赖(50.4.3章节-“项目依赖”)和/或任务依赖(6.5章节-“任务依赖”)。任何其他形式的项目相互作用(即通过修改另一个项目对象or通过从另一个项目对象中读取某个值)导致项目被解耦。配置过程中耦合的后果是如果Gradle调用‘configuration on demand’选项，生成的结果可能在几个方面存在缺陷。在执行截断耦合的结果是，如果Gradle调用并行选项，一个项目的任务运行来不及影响并联在项目的构建任务。

Two projects are said to be decoupled if they do not directly access each other's project model. Decoupled projects may only interact in terms of declared dependencies: project dependencies (Section 50.4.3, “Project dependencies”) and/or task dependencies (Section 6.5, “Task dependencies”). Any other form of project interaction (i.e. by modifying another project object or by reading a value from another project object) causes the projects to be coupled. The consequence of coupling during the configuration phase is that if gradle is invoked with the 'configuration on demand' option, the result of the build can be flawed in several ways. The consequence of coupling during execution phase is that if gradle is invoked with the parallel option, one project task runs too late to influence a task of a project building in parallel. Gradle does not attempt to detect coupling and warn the user, as there are too many possibilities to introduce coupling.

对于要加上项目的一个非常常见的方式是通过使用配置注入(56.1章节-"跨项目配置")。它可能不会立即显现出来，但使用的关键Gradle功能，如allprojects和subprojects的关键字会自动使你的项目被耦合。这是因为这些关键字用在build.gradle文件中来定义一个项目。通常不定义常见的配置的是个“根目录”，但作为Gradle而言这个根目录仍然是一个完成成熟的项目，并通过allprojects项目有效的耦合到所有的其他项目。根目录的子项目不影响‘configuration on demand’，但在任务项目build.gradle文件中使用一个allprojects和subprojects将会产生影响。

A very common way for projects to be coupled is by using configuration injection (Section 56.1, “Cross project configuration”). It may not be immediately apparent, but using key Gradle features like the allprojects and subprojects keywords automatically cause your projects to be coupled. This is because these keywords are used in a build.gradle file, which defines a project. Often this is a “root project” that does nothing more than define common configuration, but as far as Gradle is concerned this root project is still a fully-fledged project, and by using allprojects that project is effectively coupled to all other projects. Coupling of the root project to subprojects does not impact 'configuration on demand', but using the allprojects and subprojects in any subproject's build.gradle file will have an impact.

这意味着使用任务形式的共享构建脚本逻辑或配置注入(allprojects, subprojects,等)将导致你的项目被耦合。作为作为我们扩展项目的解耦和提供的功能以帮助你解耦项目，我们将同时介绍新功能以帮助你解决常见的方法(像配置注入)不会影响被你耦合的项目。

This means that using any form of shared build script logic or configuration injection (allprojects, subprojects, etc.) will cause your projects to be coupled. As we extend the concept of project decoupling and provide features that take advantage of decoupled projects, we will also introduce new features to help you to solve common use cases (like configuration injection) without causing your projects to be coupled.

为了充分利用好跨项目配置而不需运行并行和‘configuration on demand’选项，请遵循以下建议：

In order to make good use of cross project configuration without running into issues for parallel and 'configuration on demand' options, follow these recommendations:

避免子项目的build.gradle引用其他子项目；从根目录中提出跨项目配置。

Avoid a subproject's build.gradle referencing other subprojects; prefering cross configuration from the root project.

避免在执行时改变其他项目的配置

Avoid changing the configuration of other projects at execution time.

## **56.10. 多项目构建和测试**

56.10. Multi-Project Building and Testing

Java插件的构建任务通常用于编译，测试，和执行单个项目的代码样式检查(如果CodeQuality插件被使用)。在多项目构建中你可能经常想在一个项目范围内完成所有的任务。与此有关的buildNeeded 和buildDependents任务可以帮助你。

The build task of the Java plugin is typically used to compile, test, and perform code style checks (if the CodeQuality plugin is used) of a single project. In multi-project builds you may often want to do all of these tasks across a range of projects. The buildNeeded and buildDependents tasks can help with this.

看示例 56.25，“项目库依赖”，在示例中，“:services:personservice”项目同时依赖“:api” 和 “:shared” 项目。“:api” 项目也依赖 “:shared” 项目.

Look at Example 56.25, “Project lib dependencies”. In this example, the “:services:personservice” project depends on both the “:api” and “:shared” projects. The “:api” project also depends on the “:shared” project.

假如你正在一个项目工作，“：api”项目，你一直在做改变，但是没有执行一个clean和构建整个项目。你想构建任何必要的jar，但只有在你已经改变了项目执行的代码质量和单元测试。在构建任务做到这一点。

Assume you are working on a single project, the “:api” project. You have been making changes, but have not built the entire project since performing a clean. You want to build any necessary supporting jars, but only perform code quality and unit tests on the project you have changed. The build task does this.

示例 56.27. 构建和测试单个项目

Example 56.27. Build and Test Single Project

输出 gradle :api:build

Output of gradle :api:build
```
> gradle :api:build
:shared:compileJava
:shared:processResources
:shared:classes
:shared:jar
:api:compileJava
:api:processResources
:api:classes
:api:jar
:api:assemble
:api:compileTestJava
:api:processTestResources
:api:testClasses
:api:test
:api:check
:api:build

BUILD SUCCESSFUL

Total time: 1 secs
```

当你在一个典型开发周期中对“:api”项目进行反复的构建和更改测试工作(知道你只改变这一项目的文件)，你可能在“:shared”项目中看到改变而不想“:shared:compile”构建受影响。增加"-a"选项将导致Gradle使用cached jar解决任务项目库的依赖而不是试图重新构建依赖项目。

While you are working in a typical development cycle repeatedly building and testing changes to the “:api” project (knowing that you are only changing files in this one project), you may not want to even suffer the expense of building “:shared:compile” to see what has changed in the “:shared” project. Adding the “-a” option will cause Gradle to use cached jars to resolve any project lib dependencies and not try to re-build the depended on projects.

示例 56.28. 部分构建和单项目测试

Example 56.28. Partial Build and Test Single Project

输出 gradle -a :api:build

Output of gradle -a :api:build
```
> gradle -a :api:build
:api:compileJava
:api:processResources
:api:classes
:api:jar
:api:assemble
:api:compileTestJava
:api:processTestResources
:api:testClasses
:api:test
:api:check
:api:build

BUILD SUCCESSFUL

Total time: 1 secs
```

如果你刚刚从你的版本管理系统中得到最新的版本源码，其中包括依赖“.:api”的其他项目的改变，你可能不仅要建立依赖的所有项目，而且要测试它们。buildNeeded任务也会测试从testRuntime配置的项目库依赖所有项目。

If you have just gotten the latest version of source from your version control system which included changes in other projects that “:api” depends on, you might want to not only build all the projects you depend on, but test them as well. The buildNeeded task also tests all the projects from the project lib dependencies of the testRuntime configuration.

示例 56.29. 构建和测试依赖项目

Example 56.29. Build and Test Depended On Projects

输出 gradle :api:buildNeeded

Output of gradle :api:buildNeeded

```
> gradle :api:buildNeeded
:shared:compileJava
:shared:processResources
:shared:classes
:shared:jar
:api:compileJava
:api:processResources
:api:classes
:api:jar
:api:assemble
:api:compileTestJava
:api:processTestResources
:api:testClasses
:api:test
:api:check
:api:build
:shared:assemble
:shared:compileTestJava
:shared:processTestResources
:shared:testClasses
:shared:test
:shared:check
:shared:build
:shared:buildNeeded
:api:buildNeeded

BUILD SUCCESSFUL

Total time: 1 secs
```

你也可能想重构“:api”项目用于其他项目的那一部分。如果你把这些类型的更改,那它就不够只测试":api"项目了，你还需要测试依赖“.api”项目的所有项目。buildDependents任务还测试了在指定项目中有一个项目库依赖(在testRuntime配置中)的所有项目。

You also might want to refactor some part of the “:api” project that is used in other projects. If you make these types of changes, it is not sufficient to test just the “:api” project, you also need to test all projects that depend on the “:api” project. The buildDependents task also tests all the projects that have a project lib dependency (in the testRuntime configuration) on the specified project.

示例 56.30. 构建和测试相关项目

Example 56.30. Build and Test Dependent Projects

输出 gradle :api:buildDependents

Output of gradle :api:buildDependents

```
> gradle :api:buildDependents
:shared:compileJava
:shared:processResources
:shared:classes
:shared:jar
:api:compileJava
:api:processResources
:api:classes
:api:jar
:api:assemble
:api:compileTestJava
:api:processTestResources
:api:testClasses
:api:test
:api:check
:api:build
:services:personService:compileJava
:services:personService:processResources
:services:personService:classes
:services:personService:jar
:services:personService:assemble
:services:personService:compileTestJava
:services:personService:processTestResources
:services:personService:testClasses
:services:personService:test
:services:personService:check
:services:personService:build
:services:personService:buildDependents
:api:buildDependents

BUILD SUCCESSFUL

Total time: 1 secs
```

最后，你可能想构建和测试所有项目中的一切事物。在根目录中运行任何的任务将导致所有子项目相同名称的任务一起运行。因此你可以只运行“gradle build”来构建和测试所有项目。

Finally, you may want to build and test everything in all projects. Any task you run in the root project folder will cause that same named task to be run on all the children. So you can just run “gradle build” to build and test all projects.

## **56.11. 多项目与buildSrc**

56.11. Multi Project and buildSrc

59.4章节，“Build sources in the buildSrc project” 告诉我们我们可以在指定的buildSrc目录下构建逻辑和进行编译测试。在一个多项目构建中，它们只能有一个buildSrc目录且必须位于根目录。 

Section 59.4, “Build sources in the buildSrc project” tells us that we can place build logic to be compiled and tested in the special buildSrc directory. In a multi project build, there can only be one buildSrc directory which must be located in the root directory.

## **56.12. 属性和方法的继承**

56.12. Property and method inheritance

在一个项目中声明的属性和方法会被它所有的子项目继承。这是一种替代的配置注入。但是我们认为继承模式并不能很好的反映多项目构建的控件问题。在本用户指南的后续版本中，关于这点我们可能会写更多。

Properties and methods declared in a project are inherited to all its subprojects. This is an alternative to configuration injection. But we think that the model of inheritance does not reflect the problem space of multi-project builds very well. In a future edition of this user guide we might write more about this.

方法继承可能会集成到使用Gradle的配置注入但目前还不支持这方法(但是将在后续版本中发布)

Method inheritance might be interesting to use as Gradle's Configuration Injection does not support methods yet (but will in a future release).

你可能会奇怪为什么我们已经实现了一个我们不喜欢那么多的功能。其中一个原因是它是由其他工具提供且我们希望有一个比较的选择:)。同时我们原因为用户提供一个选择。

You might be wondering why we have implemented a feature we obviously don't like that much. One reason is that it is offered by other tools and we want to have the check mark in a feature comparison :). And we like to offer our users a choice.

## **56.13. 总结**

56.13. Summary

编写该章节是相当费力的同时读它可以也有同样的效果.我们在该章节最终的启示是使用Gradle进行多项目构建并不难.有5个元素你需要记住:allprojects,subprojects,evaluationDependsOn,evaluationDependsOnChildren 和project lib dependencies.[24]有了这些元素,并牢记Gradle有独特的配置和执行过程,这样你就具备有很大的灵活性.但是当你进入高端领域时Gradle不成为障碍,通常伴随你到顶端。

Writing this chapter was pretty exhausting and reading it might have a similar effect. Our final message for this chapter is that multi-project builds with Gradle are usually not difficult. There are five elements you need to remember: allprojects, subprojects, evaluationDependsOn, evaluationDependsOnChildren and project lib dependencies. [24] With those elements, and keeping in mind that Gradle has a distinct configuration and execution phase, you already have a lot of flexibility. But when you enter steep territory Gradle does not become an obstacle and usually accompanies and carries you to the top of the mountain.

[21] 我们的实际使用情况是采用http://lucene.apache.org/solr,你需要为你访问每个索引指定一个war。这是一个原因，我们已经创建了一个分配程序。Resin servlet容器允许我们让这样的一个分布点在servlet容器进行基本的安装。

[21] The real use case we had, was using http://lucene.apache.org/solr, where you need a separate war for each index you are accessing. That was one reason why we have created a distribution of webapps. The Resin servlet container allows us, to let such a distribution point to a base installation of the servlet container.

[22]“services”也是一个项目，但是我们只使用它作为一个容器。它没有生成构建脚本也没有通过其他构建脚本得到其他注入。

[22] “services” is also a project, but we use it just as a container. It has no build script and gets nothing injected by another build script.

[23]在这里我们这样做，因为它使布局更容易。我们通常把这个项目具体的东西放到各自项目的构建脚本中。

[23] We do this here, as it makes the layout a bit easier. We usually put the project specific stuff into the build script of the respective projects.

[24] 因此我们在7加2的规则范围内很好

[24] So we are well in the range of the 7 plus 2 Rule :)