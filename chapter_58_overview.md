 # **第58章  构建生命周期**

Chapter 58. The Build Lifecycle

我们早前说过，Gradle的核心就是一个基于依赖的编程语言。对于Gradle而言，这意味着你可以定义任务和任务之间的依赖关系，并保证这些任务按照他们的依赖顺序执行，且每个任务只执行一次。这些任务形成了一个有向无环图，它是一个执行任务后会创建一个依赖关系图的构建工具，Gradle构建完整的依赖图之前需要执行完所有的任务。这是Gradle的核心部分，能够使一般不可能实现的功能得以实现。

We said earlier that the core of Gradle is a language for dependency based programming. In Gradle terms this means that you can define tasks and dependencies between tasks. Gradle guarantees that these tasks are executed in the order of their dependencies, and that each task is executed only once. These tasks form a Directed Acyclic Graph. There are build tools that build up such a dependency graph as they execute their tasks. Gradle builds the complete dependency graph before any task is executed. This lies at the heart of Gradle and makes many things possible which would not be possible otherwise.

因此，你构建脚本配置的依赖图，严格意义上来说是构建配置脚本。

Your build scripts configure this dependency graph. Therefore they are strictly speaking build configuration scripts.

## **58.1. 构建阶段**

58.1. Build phases

Gradle构建有三个不同的阶段。

A Gradle build has three distinct phases.

初始化

Initialization

Gradle支持单个和多个项目构建。在初始化阶段，Gradle决定哪些项目要参与构建，并为每一个项目创建一个项目实例。

Gradle supports single and multi-project builds. During the initialization phase, Gradle determines which projects are going to take part in the build, and creates a Project instance for each of these projects.

配置

Configuration

在项目对象进行配置这个阶段，参与构建的所有项目的构建脚本都要执行。Gradle1.4中介绍了一个可选特性称为按需配置。在这种模式下，Gradle仅配置相关项目（详情见56.1.1.1，“按需配置”）

During this phase the project objects are configured. The build scripts of all projects which are part of the build are executed. Gradle 1.4 introduced an incubating opt-in feature called configuration on demand. In this mode, Gradle configures only relevant projects (see Section 56.1.1.1, “Configuration on demand”).

执行

Execution

Gradle确定的任务集，在配置的阶段创建和配置，并执行。该任务集是由任务名称的参数传递给他的命令和当前目录，然后执行每个选定的任务。

Gradle determines the subset of the tasks, created and configured during the configuration phase, to be executed. The subset is determined by the task name arguments passed to the gradle command and the current directory. Gradle then executes each of the selected tasks.

## **58.2. 设置文件**

58.2. Settings file

除了构建脚本文件外，Gradle还定义了一个设置文件。这个设置文件是由Gradle命名约定来确定的。它的默认名称是settings.gradle.在本章的后面我们将讲解Gradle是如何寻找一个配置文件的。

Beside the build script files, Gradle defines a settings file. The settings file is determined by Gradle via a naming convention. The default name for this file is settings.gradle. Later in this chapter we explain how Gradle looks for a settings file.

设置文件在初始化阶段执行。多项目构建必须有一个 settings.gradle 文件在多层级的根项目中，因为这个文件定义了哪些项目参与了多项目构建（详情见56章，多项目构建）。对于单项目构建，设置文件是可选的。除了定义包含的项目外，你可能还需要添加它的库到你的构建脚本路径中（详情见59章，组织构建逻辑）。让我们先来做一个单项目构建：

The settings file is executed during the initialization phase. A multiproject build must have a settings.gradle file in the root project of the multiproject hierarchy. It is required because the settings file defines which projects are taking part in the multi-project build (see Chapter 56, Multi-project Builds). For a single-project build, a settings file is optional. Besides defining the included projects, you might need it to add libraries to your build script classpath (see Chapter 59, Organizing Build Logic). Let's first do some introspection with a single project build:

例：58.1 单项目构建

Example 58.1. Single project build

settings.gradle
```
println 'This is executed during the initialization phase.'
build.gradle
println 'This is executed during the configuration phase.'

task configured {
    println 'This is also executed during the configuration phase.'
}

task test << {
    println 'This is executed during the execution phase.'
}

task testBoth {
    doFirst {
      println 'This is executed first during the execution phase.'
    }
    doLast {
      println 'This is executed last during the execution phase.'
    }
    println 'This is executed during the configuration phase as well.'
}
Output of gradle test testBoth
> gradle test testBoth
This is executed during the initialization phase.
This is executed during the configuration phase.
This is also executed during the configuration phase.
This is executed during the configuration phase as well.
:test
This is executed during the execution phase.
:testBoth
This is executed first during the execution phase.
This is executed last during the execution phase.

BUILD SUCCESSFUL

Total time: 1 secs
```

对于构建脚本来说，属性访问和方法调用取决于项目对象，同样属性访问和方法调用中的设置文件取决于设置对象。看看API文档中的设置类的更多信息。

For a build script, the property access and method calls are delegated to a project object. Similarly property access and method calls within the settings file is delegated to a settings object. Look at the Settings class in the API documentation for more information.

## **58.3. 多项目构建**

58.3. Multi-project builds

 多项目构建即gradle在单一执行的时候构建多个项目。你需要在设置文件中声明参与了多项目构建的项目。有更多关于多项目构建主题的相关章节（详情见56章，多项目构建）。

A multi-project build is a build where you build more than one project during a single execution of Gradle. You have to declare the projects taking part in the multiproject build in the settings file. There is much more to say about multi-project builds in the chapter dedicated to this topic (see Chapter 56, Multi-project Builds).

### **58.3.1 项目位置**

58.3.1. Project locations

多项目构建总是用一个根树表示，树中的每一个元素都代表一个项目。项目的路径表示了这个项目在多项目构建树中的位置，在大多数情况下，项目路径是与文件系统中的项目的实际位置是一致的，然而，这种行为是可配置的。项目树是在settings.gradle文件中创建的。一般默认情况下设置文件位置也是根项目位置，不过，你可以在设置文件中自定义根项目位置。

Multi-project builds are always represented by a tree with a single root. Each element in the tree represents a project. A project has a path which denotes the position of the project in the multi-project build tree. In most cases the project path is consistent with the physical location of the project in the file system. However, this behavior is configurable. The project tree is created in thesettings.gradle file. By default it is assumed that the location of the settings file is also the location of the root project. But you can redefine the location of the root project in the settings file.

### **58.3.2 构建树**

58.3.2. Building the tree

在设置文件中你可以使用一组方法来构建项目树，即分层和平面布局方法，使用这组方法可以让你得到特别的支持。

In the settings file you can use a set of methods to build the project tree. Hierarchical and flat physical layouts get special support.

58.3.2.1 分层布局

58.3.2.1. Hierarchical layouts

例 58.2 分层布局

Example 58.2. Hierarchical layout

settings.gradle

include 'project1', 'project2:child', 'project3:child1'

此方法包括以项目路径作为参数。假定项目路径等同于相对的物理文件系统路径。例如，路径 ‘services:api’默认情况下是映射到文件夹’services/api’（相对于项目的根），你只需要指定树的叶子。这意味着包含路径’services:hotels:api'’将会创建三个项目:'services', 'services:hotels' 和'services:hotels:api'.

The include method takes project paths as arguments. The project path is assumed to be equal to the relative physical file system path. For example, a path 'services:api' is mapped by default to a folder 'services/api' (relative from the project root). You only need to specify the leaves of the tree. This means that the inclusion of the path 'services:hotels:api' will result in creating 3 projects: 'services', 'services:hotels' and 'services:hotels:api'.

58.3.2.2  平面布局

58.3.2.2. Flat layouts

例 58.3 平面布局

Example 58.3. Flat layout

settings.gradle
```
includeFlat 'project3', 'project4'
```
这个includeFlat 方法将目录名字作为参数，这些目录需要像兄弟姐妹一样存在于项目的根目录下。这些目录的位置默认为多项目树的根目录下的子目录。

The includeFlat method takes directory names as an argument. These directories need to exist as siblings of the root project directory. The location of these directories are considered as child projects of the root project in the multi-project tree.

### **58.3.3  修改项目树的元素**

58.3.3. Modifying elements of the project tree

多项目树创建后在设置文件中创建一个叫项目描述的项，你可以在任意时间到设置文件中修改这些描述项。要访问一个描述符，你可以这样做：

The multi-project tree created in the settings file is made up of so called project descriptors. You can modify these descriptors in the settings file at any time. To access a descriptor you can do:

例 58.4  修改项目树的元素

Example 58.4. Modification of elements of the project tree

settings.gradle
```
println rootProject.name
println project(':projectA').name
```

使用这些描述符，你可以修改项目的名字，项目目录和构建文件。

Using this descriptor you can change the name, project directory and build file of a project.

例 58.5 修改项目树的元素

Example 58.5. Modification of elements of the project tree

settings.gradle
```
rootProject.name = 'main'
project(':projectA').projectDir = new File(settingsDir, '../my-project-a')
project(':projectA').buildFileName = 'projectA.gradle'
```
更多详情请查看api文档中的ProjectDescriptor 类。

Look at the ProjectDescriptor class in the API documentation for more information.

## **58.4 初始化**

58.4. Initialization

如何知道Gradle是单项目还是多项目构建？你可以到设置文件目录中
去触发多项目构建，这会是件很简单的事情。但是Gradle还允许任何参与构建的子项目执行构建。在Gradle中如果你要执行一个没有settings.gradle文件的项目，它会以下列方式去查找这个文件：

How does Gradle know whether to do a single or multiproject build? If you trigger a multiproject build from a directory with a settings file, things are easy. But Gradle also allows you to execute the build from within any subproject taking part in the build. [20] If you execute Gradle from within a project with no settings.gradle file, Gradle looks for a settings.gradle file in the following way:


	去查找一个和master目录具有相同嵌套级别的目录。

	如果没有找到，可以继续查找它的父目录。

	如果依然找不到，这个构建可以作为一个单项目构建执行。

	如果找到了settings.gradle文件，Gradle会检查当前项目是不是settings.

gradle文件中多项目层级定义的一部分，如果不是，此构建作为单项目构建执行，否则，多项目构建执行。

•	It looks in a directory called master which has the same nesting level as the current dir.
•	If not found yet, it searches parent directories.
•	If not found yet, the build is executed as a single project build.
•	If a settings.gradle file is found, Gradle checks if the current project is part of the multiproject hierarchy defined in the found settings.gradle file. If not, the build is executed as a single project build. Otherwise a multiproject build is executed.

此行为的目的是什么？Gradle需要确定这个项目是不是多项目构建中的子项目。当然，如果它是子项目，则只有子项目和它相关的项目构建。但是Gradle需要创建整个多项目构建的构建配置（参见第56章，多项目构建）。你可以使用-u命令行选项来告诉它不去看settings.gradle文件中的父层级，则当前项目一直进行单项目构建，如果当前项目包含settings.gradle这个文件，-u选项就没有意义了。这样的构建始终执行如下：

What is the purpose of this behavior? Gradle needs to determine whether the project you are in is a subproject of a multiproject build or not. Of course, if it is a subproject, only the subproject and its dependent projects are built, but Gradle needs to create the build configuration for the whole multiproject build (see Chapter 56, Multi-project Builds). You can use the -u command line option to tell Gradle not to look in the parent hierarchy for a settings.gradle file. The current project is then always built as a single project build. If the current project contains a settings.gradle file, the -u option has no meaning. Such a build is always executed as:


•	如果settings.gradle文件中没有定义多项目层级，则进行单项目构建。
•	如果settings.gradle文件中定义了多项目层级，则进行多项目构建。
自动搜索 settings.gradle文件仅适用于多项目构建的物理分层或平面布局。对于平面布局，你还必须参照上面（“master”）中所描述的命名公约。Gradle多项目构建支持任意的物理布局。但对于这样的任意布局，你需要在设置文件所在的目录执行构建。有关如何运行根的部分构建，见56.4章节中的“绝对路径运行任务”。

•	a single project build, if the settings.gradle file does not define a multiproject hierarchy
•	a multiproject build, if the settings.gradle file does define a multiproject hierarchy.
The automatic search for a settings.gradle file only works for multi-project builds with a physical hierarchical or flat layout. For a flat layout you must additionally follow the naming convention described above (“master”). Gradle supports arbitrary physical layouts for a multiproject build, but for such arbitrary layouts you need to execute the build from the directory where the settings file is located. For information on how to run partial builds from the root see Section 56.4, “Running tasks by their absolute path”.

Gradle会为每个参与构建的项目创建一个项目对象。对于多项目构建的这些项目都会在设置对象中指定（增加根项目）。每个项目对象都有一个默认的名字等于其顶级目录的名称，除了根项目每个项目都有一个父项目，且任何项目都有可能有子项目。

Gradle creates a Project object for every project taking part in the build. For a multi-project build these are the projects specified in the Settings object (plus the root project). Each project object has by default a name equal to the name of its top level directory, and every project except the root project has a parent project. Any project may have child projects.

## **58.5 单项目的配置和执行**

58.5. Configuration and execution of a single project build

对于单项目构建，初始化阶段后，工作流程非常简单。构建脚本执行会在初始化阶段创建项目对象，然后Gradle寻找名字等同于那些通过的传递作为命令行参数的任务。如果这些任务名字存在，它们可以按照通过的顺序独立执行构建。多项目的配置和执行在第56章（多项目构建）中进行讨论。

For a single project build, the workflow of the after initialization phases are pretty simple. The build script is executed against the project object that was created during the initialization phase. Then Gradle looks for tasks with names equal to those passed as command line arguments. If these task names exist, they are executed as a separate build in the order you have passed them. The configuration and execution for multi-project builds is discussed in Chapter 56, Multi-project Builds.

## **58.6构建脚本的生命周期响应**

58.6. Responding to the lifecycle in the build script

你的构建脚本可以接收通知作为构建过程，通过它的生命周期。这些通知一般采用两种形式：你可以实现一个特定的侦听器接口，或者你也可以提供一个闭合时通报解除的执行。下面使用的是闭合的例子，有关如何使用监听器接口的详细信息，请参阅相关的API文档。

Your build script can receive notifications as the build progresses through its lifecycle. These notifications generally take two forms: You can either implement a particular listener interface, or you can provide a closure to execute when the notification is fired. The examples below use closures. For details on how to use the listener interfaces, refer to the API documentation.

## ***58.6.1 项目评估***

58.6.1. Project evaluation

你可以在项目评估之前和之后马上收到通知 。这可以用来做的事情相同，一旦在构建脚本中的所有定义都得到了应用，或对于一些自定义日志记录或分析会进行额外的配置。

You can receive a notification immediately before and after a project is evaluated. This can be used to do things like performing additional configuration once all the definitions in a build script have been applied, or for some custom logging or profiling.

下面是一个给每个hasTests 属性值为true的项目添加一个测试任务的例子。
Below is an example which adds a test task to each project which has a hasTests property value of true.

例58.6   给每个具有一定的属性集项目添加测试任务

Example 58.6. Adding of test task to each project which has certain property set

build.gradle
```
allprojects {
    afterEvaluate { project ->
        if (project.hasTests) {
            println "Adding test task to $project"
            project.task('test') << {
                println "Running tests for $project"
            }
        }
    }
}
projectA.gradle
hasTests = true
Output of gradle -q test
> gradle -q test
Adding test task to project ':projectA'
Running tests for project ':projectA'
```

这个例子使用Project.afterEvaluate()方法添加一个执行项目评估后的闭合。另外，也可以接收任意项目评估的通知。这个例子进行项目评估的一些自定义日志记录。注意，无论项目是否成功的进行或者失败异常都能收到通知。

This example uses method Project.afterEvaluate() to add a closure which is executed after the project is evaluated.
It is also possible to receive notifications when any project is evaluated. This example performs some custom logging of project evaluation. Notice that the afterProject notification is received regardless of whether the project evaluates successfully or fails with an exception.

例58.7 通知

Example 58.7. Notifications

build.gradle
```
gradle.afterProject {project, projectState ->
    if (projectState.failure) {
        println "Evaluation of $project FAILED"
    } else {
        println "Evaluation of $project succeeded"
    }
}
Output of gradle -q test
> gradle -q test
Evaluation of root project 'buildProjectEvaluateEvents' succeeded
Evaluation of project ':projectA' succeeded
Evaluation of project ':projectB' FAILED
```

你也可以添加ProjectEvaluationListener到Gradle中去接收这些事件。

You can also add a ProjectEvaluationListener to the Gradle to receive these events.

## **58.6.2  创建任务**

58.6.2. Task creation

你可以在一个任务添加到项目后，马上接收到一个通知。在构建文件之前，这些可以被用来设置一些默认值或者添加行为任务。

You can receive a notification immediately after a task is added to a project. This can be used to set some default values or add behaviour before the task is made available in the build file.

下面这个例子是设置每个任务创建的srcDir 属性。

The following example sets the srcDir property of each task as it is created.

例 58.8 设置所有任务的某些属性

Example 58.8. Setting of certain property to all tasks

build.gradle
```
tasks.whenTaskAdded { task ->
    task.ext.srcDir = 'src/main/java'
}

task a

println "source dir is $a.srcDir"
Output of gradle -q a
> gradle -q a
source dir is src/main/java
```

你也可以添加一个Action 到TaskContainer 中去接收这些事件。

You can also add an Action to a TaskContainer to receive these events.

### **58.6.3  任务执行准备图**

58.6.3. Task execution graph ready

你在任务执行图完成后，可以马上接收到一个通知。我们已经在6.13章节，“通过DAG配置”中看到了这一点。你也可以添加一个TaskExecutionGraphListener 到 TaskExecutionGraph中去接收这些事件。

You can receive a notification immediately after the task execution graph has been populated. We have seen this already in Section 6.13, “Configure by DAG”.
You can also add a TaskExecutionGraphListener to the TaskExecutionGraph to receive these events.

### **58.6.4 任务执行**

58.6.4. Task execution

你可以在任何任务执行之前和之后马上接收到一个消息。以下示例展示了每个任务执行的开始和结束日志。注意，不管任务是否成功完成或失败的异常信息，afterTask 通知都会接收。

You can receive a notification immediately before and after any task is executed.
The following example logs the start and end of each task execution. Notice that the afterTask notification is received regardless of whether the task completes successfully or fails with an exception.

例58.9  每个任务执行开始和结束的日志
Example 58.9. Logging of start and end of each task execution

build.gradle
```
task ok

task broken(dependsOn: ok) << {
    throw new RuntimeException('broken')
}

gradle.taskGraph.beforeTask { Task task ->
    println "executing $task ..."
}

gradle.taskGraph.afterTask { Task task, TaskState state ->
    if (state.failure) {
        println "FAILED"
    }
    else {
        println "done"
    }
}
Output of gradle -q broken
> gradle -q broken
executing task ':ok' ...
done
executing task ':broken' ...
FAILED
```

你也可以使用TaskExecutionListener 到TaskExecutionGraph 中去接收这些事件。

You can also use a TaskExecutionListener to the TaskExecutionGraph to receive these events.
 
[20]Gradle支持部分多项目构建（详见56章，多项目构建）。

[20] Gradle supports partial multiproject builds (see Chapter 56, Multi-project Builds).

