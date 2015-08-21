# **第 11 章.Gradle命令行的基本使用**

Chapter 11. Using the Gradle Command-Line

本章介绍了命令行的基本使用.正如在前面的章节里你所见到的调用 gradle命令来完成一些功能

This chapter introduces the basics of the Gradle command-line. You run a build using the gradle command, which you have already seen in action in previous chapters.

# **11.1. 多任务调用**

11.1. Executing multiple tasks

你可以以列表的形式在命令行中一次调用多个任务.例如 gradle compile test命令会依次调用,并且每个任务仅会被调用一次. compile和test任务以及它们的依赖任务. 无论它们是否被包含在脚本中:即无论是以命令行的形式定义的任务还是依赖于其它任务都会被调用执行.来看下面的例子.

You can execute multiple tasks in a single build by listing each of the tasks on the command-line. For example, the command gradle compile test will execute the compile and test tasks. Gradle will execute the tasks in the order that they are listed on the command-line, and will also execute the dependencies for each task. Each task is executed once only, regardless of how it came to be included in the build: whether it was specified on the command-line, or it a dependency of another task, or both. Let's look at an example.

下面定义了四个任务.dist和test 都 依赖于compile,只用当 compile被调用之后才会调用gradle dist test任务

Below four tasks are defined. Both dist and test depend on the compile task. Running gradle dist test for this build script results in the compile task being executed only once.

示例图 11.1. 任务依赖

Figure 11.1. Task dependencies

Task dependencies
![](http://pkaq.github.io/gradledoc/docs/userguide/img/commandLineTutorialTasks.png)


例 11.1. 多任务调用

Example 11.1. Executing multiple tasks

build.gradle

```
task compile << {
    println 'compiling source'
}

task compileTest(dependsOn: compile) << {
    println 'compiling unit tests'
}

task test(dependsOn: [compile, compileTest]) << {
    println 'running unit tests'
}

task dist(dependsOn: [compile, test]) << {
    println 'building the distribution'
}

```

gradle dist test的输出结果.

Output of gradle dist test

```
> gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs

```

由于每个任务仅会被调用一次,所以调用gradle test test与调用gradle test效果是相同的.

Because each task is executed once only, executing gradle test test is exactly the same as executing gradle test.

# **11.2. 排除任务**

11.2. Excluding tasks

你可以用命令行选项-x来排除某些任务,让我们用上面的例子来示范一下.

You can exclude a task from being executed using the -x command-line option and providing the name of the task to exclude. Let's try this with the sample build file above.

例 11.2. 排除任务.

Example 11.2. Excluding tasks

gradle dist -x test的输出结果.

Output of gradle dist -x test

```
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs

```

可以看到,test任务并没有被调用,即使他是dist任务的依赖. 同时test任务的依赖任务compileTest也没有被调用,而像 compile被test和其它任务同时依赖的任务仍然会被调用.

You can see from the output of this example, that the test task is not executed, even though it is a dependency of the dist task. You will also notice that the test task's dependencies, such as compileTest are not executed either. Those dependencies of test that are required by another task, such as compile , are still executed.

## **11.3. 失败后继续执行**

11.3. Continuing the build when a failure occurs

默认情况下只要有任务调用失败Gradle就是中断执行.这可能会使调用过程更快,但那些后面隐藏的错误不会被发现. 所以你可以使用--continue在一次调用中尽可能多的发现所有问题.

By default, Gradle will abort execution and fail the build as soon as any task fails. This allows the build to complete sooner, but hides other failures that would have occurred. In order to discover as many failures as possible in a single build execution, you can use the --continue option.

采用了--continue选项,Gralde会调用每一个任务以及它们依赖的任务. 而不是一旦出现错误就会中断执行.所有错误信息都会在最后被列出来.

When executed with --continue , Gradle will execute every task to be executed where all of the dependencies for that task completed without failure, instead of stopping as soon as the first failure is encountered. Each of the encountered failures will be reported at the end of the build.

一旦某个任务执行失败,那么所有依赖于该任务的子任务都不会被调用.例如由于test任务依赖于complie任务,所以如果compile调用出错,test便不会被直接或间接调用.

If a task fails, any subsequent tasks that were depending on it will not be executed, as it is not safe to do so. For example, tests will not run if there is a compilation failure in the code under test; because the test task will depend on the compilation task (either directly or indirectly).

## **11.4. 简化任务名**

11.4. Task name abbreviation

当你试图调用某个任务的时候,无需输入任务的全名.只需提供足够的可以唯一区分出该任务的字符即可.例如,上面的例子你也可以这么写. 用gradle di来直接调用dist任务

When you specify tasks on the command-line, you don't have to provide the full name of the task. You only need to provide enough of the task name to uniquely identify the task. For example, in the sample build above, you can execute task dist by running gradle di:

例 11.3. 简化任务名

Example 11.3. Abbreviated task name

gradle di的输出结果

Output of gradle di

```
> gradle di
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```

你也可以用驼峰命名的任务中每个单词的首字母进行调用.例如,可以执行gradle compTest or even gradle cT来调用 compileTest任务

You can also abbreviate each word in a camel case task name. For example, you can execute task compileTest by running gradle compTest or even gradle cT

例 11.4. 简化驼峰任务名

Example 11.4. Abbreviated camel case task name

gradle cT的输出结果.

Output of gradle cT

```
> gradle cT
:compile
compiling source
:compileTest
compiling unit tests

BUILD SUCCESSFUL

Total time: 1 secs
```

简化后你仍然可以使用-x参数.

You can also use these abbreviations with the -x command-line option.

## **11.5. 选择构建位置**

11.5. Selecting which build to execute

调用gradle时,默认情况下总是会构建当前目录下的文件, 可以使用-b参数选择构建的文件,并且当你使用此参数时settings.gradle将不会生效,看下面的例子:

When you run the gradle command, it looks for a build file in the current directory. You can use the -b option to select another build file. If you use -b option then settings.gradle file is not used. Example:

例 11.5. 选择文件构建

Example 11.5. Selecting the project using a build file

subdir/myproject.gradle

```
task hello << {
    println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
}
```

gradle -q -b subdir/myproject.gradle hello的输出结果

Output of gradle -q -b subdir/myproject.gradle hello

```
> gradle -q -b subdir/myproject.gradle hello

using build file 'myproject.gradle' in 'subdir'.

```

另外,你可以使用-p参数来指定构建的目录,例如在多项目构建中你可以用-p来替代-b参数

Alternatively, you can use the -p option to specify the project directory to use. For multi-project builds you should use -p option instead of -b option.

例 11.6. 选择构建目录

Example 11.6. Selecting the project using project directory

gradle -q -p subdir hello的输出结果

Output of gradle -q -p subdir hello

```
> gradle -q -p subdir hello
using build file 'build.gradle' in 'subdir'.
```

备注: -b参数用以指定脚本具体所在位置,格式为 dirpwd/build.gradle.-p参数用以指定脚本目录即可.

# **11.6. 获取构建信息**

11.6. Obtaining information about your build

Gradle提供了许多内置任务来收集构建信息.这些内置任务对于了解依赖结构以及解决问题都是很有帮助的.

Gradle provides several built-in tasks which show particular details of your build. This can be useful for understanding the structure and dependencies of your build, and for debugging problems.

了解更多,可以参阅项目报告插件以为你的项目添加构建报告.

In addition to the built-in tasks shown below, you can also use the project report plugin to add tasks to your project which will generate these reports.

# **11.6.1. 项目列表**

11.6.1. Listing projects

执行gradle projects会为你列出子项目名称列表.如下例.

Running gradle projects gives you a list of the sub-projects of the selected project, displayed in a hierarchy. Here is an example:

例 11.7. 收集项目信息

Example 11.7. Obtaining information about projects

gradle -q projects的输出结果

Output of gradle -q projects

```
> gradle -q projects
------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'projectReports'
+--- Project ':api' - The shared API for the application
\--- Project ':webapp' - The Web application implementation

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :api:tasks
```

这份报告展示了每个项目的描述信息.当然你可以在项目中用description属性来指定这些描述信息.

The report shows the description of each project, if specified. You can provide a description for a project by setting the description property:

例 11.8. 为项目添加描述信息.

Example 11.8. Providing a description for a project

build.gradle

```
description = 'The shared API for the application'
```
						
11.6.2. 任务列表

11.6.2. Listing tasks

执行gradle tasks会列出项目中所有任务. 这会显示项目中所有的默认任务以及每个任务的描述.如下例

Running gradle tasks gives you a list of the main tasks of the selected project. This report shows the default tasks for the project, if any, and a description for each task. Below is an example of this report:

例 11.9. 获取任务信息

Example 11.9. Obtaining information about tasks

Output of gradle -q tasks

```
> gradle -q tasks
------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
dists - Builds the distribution
libs - Builds the JAR

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
dependencies - Displays all dependencies declared in root project 'projectReports'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
help - Displays a help message
projects - Displays the sub-projects of root project 'projectReports'.
properties - Displays the properties of root project 'projectReports'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).

To see all tasks and more detail, run with --all.

```

默认情况下,这只会显示那些被分组的任务.你可以通过为任务设置group属性和description来把 这些信息展示到结果中.

By default, this report shows only those tasks which have been assigned to a task group. You can do this by setting the group property for the task. You can also set the description property, to provide a description to be included in the report.

例 11.10. 更改任务报告内容

Example 11.10. Changing the content of the task report

build.gradle

```
dists {
    description = 'Builds the distribution'
    group = 'build'
}
```

当然你也可以用--all参数来收集更多任务信息.这会列出项目中所有任务以及任务之间的依赖关系.

You can obtain more information in the task listing using the --all option. With this option, the task report lists all tasks in the project, grouped by main task, and the dependencies for each task. Here is an example:

Example 11.11. Obtaining more information about tasks

Output of gradle -q tasks --all

```
> gradle -q tasks --all
------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
api:clean - Deletes the build directory (build)
webapp:clean - Deletes the build directory (build)
dists - Builds the distribution [api:libs, webapp:libs]
    docs - Builds the documentation
api:libs - Builds the JAR
    api:compile - Compiles the source files
webapp:libs - Builds the JAR [api:libs]
    webapp:compile - Compiles the source files

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
dependencies - Displays all dependencies declared in root project 'projectReports'.
api:dependencies - Displays all dependencies declared in project ':api'.
webapp:dependencies - Displays all dependencies declared in project ':webapp'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
api:dependencyInsight - Displays the insight into a specific dependency in project ':api'.
webapp:dependencyInsight - Displays the insight into a specific dependency in project ':webapp'.
help - Displays a help message
api:help - Displays a help message
webapp:help - Displays a help message
projects - Displays the sub-projects of root project 'projectReports'.
api:projects - Displays the sub-projects of project ':api'.
webapp:projects - Displays the sub-projects of project ':webapp'.
properties - Displays the properties of root project 'projectReports'.
api:properties - Displays the properties of project ':api'.
webapp:properties - Displays the properties of project ':webapp'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
api:tasks - Displays the tasks runnable from project ':api'.
webapp:tasks - Displays the tasks runnable from project ':webapp'.
```

# **11.6.3. 获取任务帮助信息**

11.6.3. Show task usage details

执行gradle help --task someTask可以显示指定任务的详细信息. 或者多项目构建中相同任务名称的所有任务的信息.如下例.

Running gradle help --task someTask gives you detailed information about a specific task or multiple tasks matching the given task name in your multiproject build. Below is an example of this detailed information:

例 11.12. 获取任务帮助

Example 11.12. Obtaining detailed help for tasks

gradle -q help --task libs的输出结果

Output of gradle -q help --task libs

```
> gradle -q help --task libs
Detailed task information for libs

Paths
     :api:libs
     :webapp:libs

Type
     Task (org.gradle.api.Task)

Description
     Builds the JAR
```

这些结果包含了任务的路径、类型以及描述信息等.

This information includes the full task path, the task type, possible commandline options and the description of the given task.

## **11.6.4. 获取依赖列表**

11.6.4. Listing project dependencies

执行 gradle dependencies 会列出项目的依赖列表,所有依赖会根据任务区分,以树型结构展示出来.如下例.

Running gradle dependencies gives you a list of the dependencies of the selected project, broken down by configuration. For each configuration, the direct and transitive dependencies of that configuration are shown in a tree. Below is an example of this report:

例 11.13. 获取依赖信息

Example 11.13. Obtaining information about dependencies

gradle -q dependencies api:dependencies webapp:dependencies 的输出结果

Output of gradle -q dependencies api:dependencies webapp:dependencies

```
> gradle -q dependencies api:dependencies webapp:dependencies
------------------------------------------------------------
Root project
------------------------------------------------------------

No configurations

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

compile
\--- org.codehaus.groovy:groovy-all:2.3.3

testCompile
\--- junit:junit:4.11
     \--- org.hamcrest:hamcrest-core:1.3

------------------------------------------------------------
Project :webapp - The Web application implementation
------------------------------------------------------------

compile
+--- project :api
|    \--- org.codehaus.groovy:groovy-all:2.3.3
\--- commons-io:commons-io:1.2

testCompile
No dependencies

```

虽然输出结果很多,但这对于了解构建任务十分有用,当然你可以通过	--configuration参数来查看 指定构建任务的依赖情况.

Since a dependency report can get large, it can be useful to restrict the report to a particular configuration. This is achieved with the optional --configuration parameter:

例 11.14. 过滤依赖信息

Example 11.14. Filtering dependency report by configuration

gradle -q api:dependencies --configuration testCompile的输出结果

Output of gradle -q api:dependencies --configuration testCompile

```
> gradle -q api:dependencies --configuration testCompile
------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

testCompile
\--- junit:junit:4.11
     \--- org.hamcrest:hamcrest-core:1.3
```

## **11.6.5. 查看特定依赖**

11.6.5. Getting the insight into a particular dependency

执行Running gradle dependencyInsight 可以查看指定的依赖情况.如下例.

Running gradle dependencyInsight gives you an insight into a particular dependency (or dependencies) that match specified input. Below is an example of this report:

例 11.15. 获取特定依赖

Example 11.15. Getting the insight into a particular dependency

gradle -q webapp:dependencyInsight --dependency groovy --configuration compile的输出结果

Output of gradle -q webapp:dependencyInsight --dependency groovy --configuration compile

```
> gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
org.codehaus.groovy:groovy-all:2.3.3
\--- project :api
     \--- compile
```

这对于了解依赖关系、了解为何选择此版本作为依赖十分有用.了解更多请参阅 依赖检查报告.

This task is extremely useful for investigating the dependency resolution, finding out where certain dependencies are coming from and why certain versions are selected. For more information please see DependencyInsightReportTask.


dependencyInsight任务是'Help'任务组中的一个.这项任务需要进行配置才可以. 如果用了Java相关的插件,那么dependencyInsight任务已经预先被配置到'Compile'下了. 你只需要通过'--dependency'参数来制定所需查看的依赖即可.如果你不想用默认配置的参数项你可以通过 '--configuration' 参数来进行指定.了解更多请参阅 依赖检查报告.


The built-in dependencyInsight task is a part of the 'Help' tasks group. The task needs to configured with the dependency and the configuration. The report looks for the dependencies that match the specified dependency spec in the specified configuration. If java related plugin is applied, the dependencyInsight task is pre-configured with 'compile' configuration because typically it's the compile dependencies we are interested in. You should specify the dependency you are interested in via the command line '--dependency' option. If you don't like the defaults you may select the configuration via '--configuration' option. For more information see DependencyInsightReportTask.

## **11.6.6. 获取项目属性列表**

11.6.6. Listing project properties

执行gradle properties可以获取项目所有属性列表.如下例.

Running gradle properties gives you a list of the properties of the selected project. This is a snippet from the output:

例 11.16. 属性信息

Example 11.16. Information about properties

gradle -q api:properties的输出结果

Output of gradle -q api:properties

```
> gradle -q api:properties
------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

allprojects: [project ':api']
ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler@12345
asDynamicObject: org.gradle.api.internal.ExtensibleDynamicObject@12345
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle

```

## **11.6.7. 构建日志**

11.6.7. Profiling a build

--profile参数可以收集一些构建期间的信息并保存到 build/reports/profile目录下并且以构建时间命名这些文件.

The --profile command line option will record some useful timing information while your build is running and write a report to the build/reports/profile directory. The report will be named using the time when the build was run.

下面这份日志记录了总体花费时间以及各过程花费的时间.并以时间大小倒序排列. 并且记录了任务的执行情况.

This report lists summary times and details for both the configuration phase and task execution. The times for configuration and task execution are sorted with the most expensive operations first. The task execution results also indicate if any tasks were skipped (and the reason) or if tasks that were not skipped did no work.

如果采用了buildSrc,那么在buildSrc/build下同时也会生成一份日志记录记录

Builds which utilize a buildSrc directory will generate a second profile report for buildSrc in the buildSrc/build directory.

![](http://pkaq.github.io/gradledoc/docs/userguide/img/profile.png)

## **11.7. Dry Run**

11.7. Dry Run

有时可能你只想知道某个任务在一个任务集中按顺序执行的结果,但并不想实际执行这些任务.那么你可以用-m参数 例如gradle -m clean compile会调用clean 和 compile,这与tasks可以形成互补,让你知道哪些任务可以用于执行.

Sometimes you are interested in which tasks are executed in which order for a given set of tasks specified on the command line, but you don't want the tasks to be executed. You can use the -m option for this. For example gradle -m clean compile shows you all tasks to be executed as part of the clean and compile tasks. This is complementary to the tasks task, which shows you the tasks which are available for execution.

## **11.8. 本章小结**

11.8. Summary

在本章中你学到很多用命令行可以做的事情,了解更多你可以参阅gradle command in 附录 D, Gradle 命令行.

In this chapter, you have seen some of the things you can do with Gradle from the command-line. You can find out more about the gradle command in Appendix D, Gradle Command Line.

