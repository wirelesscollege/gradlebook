# **Chapter 19. Continuous build**

Chapter 19. 持续构建

Continuous build is an incubating feature. This means that it is incomplete and not yet at regular Gradle production quality. This also means that this Gradle User Guide chapter is a work in progress.
Typically, you ask Gradle to perform a single build by way of specifying tasks that Gradle should execute. Gradle will determine the the actual set of tasks that need to be executed to satisfy the request, execute them all, and then stop doing work until the next request. A continuous build differs in that Gradle will keep satisfying the initial build request (until instructed to stop) by executing the build when it is detected that the result of the previous build is now out of date. For example, if your build compiles Java source files to Java class files, a continuous build would automatically initiate a compile when the source files change. Continuous build is useful for many scenarios.

持续构建还是一个正在开发阶段的Feature,意味着在Gradle发布产品包中，它还没有被完全完成。也意味着在Gradle用户文档里，这个Feature也是一个进行中的事情。以前你要求gradle按照指定的任务去完成一个构建，那么gradle会按照你的要求去执行，直到下一个请求到来会停止上一次的构建，持续构建的不同就在于，Gradle会自己检查上一次的构建是否过期或者是否已经有变化或者结果了，然后会自己停止或者触发构建。例如：如果你构建编译java文件到class文件，持续构建将会在源码改动的时候自动化进行编译，在很多场景下，持续构建会非常有用。

## **19.1. How do I start and stop a continuous build?**

19.1. 如何开启或者停止一个持续构建？

A continuous build can be started by supplying either the --continuous or -t switches to Gradle, along with the list of tasks, switches and arguments that define the work to do. For example, gradle build --continuous. This will have the same effect as running gradle build, but instead of Gradle exiting when done, it will wait for changes to the build inputs. When a change occurs, gradle build will be automatically executed again and the process repeats.

持续构建实际上以--continuous或者-t参数来触发，例如：gradle build --coutinuous，构建效果和gradle build其实是一样的，但是代替Gradle不同得是，构建完成后盖任务不会终止，而是会监控你的输入变化，一旦变化发生，gradle将自动重新构建该任务。

If Gradle is attached to an interactive input source, such as a terminal, the continuous build can be exited by pressing CTRL-D (On Microsoft Windows, it is required to also press ENTER or RETURN after CTRL-D). If Gradle is not attached to an interactive input source (e.g. is running as part of a script), the build process must be terminated (e.g. using the kill command or similar). If the build is being executed via the Tooling API, the build can be cancelled using the Tooling API's cancellation mechanism.

如果gradle构建依赖于交互式输入源，例如terminal，持续构建可以使用ctrl+D来进行退出（在windows环境下你需要在按ctrl+d后再按enter或者return来进行退出）。如果gradle不依赖于交互式输入，例如：作为脚本来执行，构建进行在terminated下，例如，你需要使用kill命令来杀死gradle持续构建进行，如果构建是在工具API下执行的，那么构建可以在工具API的cacel动作下被取消掉。


## **19.2. What will cause a subsequent build?**

19.2. 如何触发后续构建？

Task file inputs

任务文件输入

Task implementations declare their file system inputs by annotating their properties with InputFiles and other similar annotations. Please see Example 14.24, “Declaring the inputs and outputs of a task” for more information.

任务得实现如果依赖于输入文件的属性或者其他注解来控制，那么可以更改文件属性或者其他方式的注解来触发。可以查看Example 14.24, “Declaring the inputs and outputs of a task” for more information.

At this time, only changes to input files to the previously executed tasks will initiate a build. Furthermore, only changes that occur after the build has completed (and Gradle has prompted that it is waiting for changes) are noticed. No other changes will initiate a build. For example, changes to build scripts and build logic will not initiate build. Likewise, changes to files that are read during the configuration of the build, not the execution, will not initiate a build. In order to incorporate such changes, the continuous build must be restarted manually.

在构建前更改输入文件可以影响构建的启动，在构建后更改文件可以触发下一次构建。例如：更改构建脚本将不会启动一个构建，在构建执行中更改文件也不会触发一个构建，如果你非要这样改变，那么持续构建只能手动去重新启动去触发你的改变。

Consider a typical build using the Java plugin, using the conventional filesystem layout. The following diagram visualizes the task graph for gradle build:

考虑到一个典型得使用java插件进行构建，使用正常的文件系统布局，下面将展示一下gradle构建过程中的图表。

Figure 19.1. Java plugin task graph

19.1 java插件任务图谱

Java plugin task graph

java 插件任务图

The following key tasks of the graph use files in the corresponding directories as inputs:

下面的关键性任务图谱都是使用对应的目录来做来输入源的

compileJava
src/main/java
processResources
src/main/resources
compileTestJava
src/test/java
processTestResources
src/test/resources


Assuming that the initial build is successful (i.e. the build task and its dependencies complete without error), changes to files in, or the addition/remove of files from, the locations listed above will initiate a new build. If a change is made to a Java source file in src/main/java, the build will fire and all tasks will be scheduled. Gradle's incremental build support ensures that only the tasks that are actually affected by the change are executed.

如果启动构建是成功的，例如：构建任务以及它的依赖都没有报错，那么改变文件输入或者添加或者删除文件，被列举以上任务将触发一个新的构建。如果是在src/main/java目录下去改变java源码，构建将会终止所有的人物将会被列入计划，gradle增加一次构建去支持你的任务改变以及改变所带来的影响。

If the change to the main Java source causes compilation to fail, subsequent changes to the test source in src/test/java will not initiate a new build. As the test source depends on the main source, there is no point building until the main source has changed, potentially fixing the compilation error. After each build, only the inputs of the tasks that actually executed will be monitored for changes.

如果改动java源码编译失败，那么测试代码将不会再次构建，测试代码依赖于源代码，只有源码改变测试代码才会构建，所以你必须修复这个编译错误。每次构建后，任务得输入执行监控于改变的发生。

Continuous build is in no way coupled to compilation. It works for all types of tasks. For example, the processResources task copies and processes the files from src/main/resources for inclusion in the built JAR. As such, a change to any file in this directory will also initiate a build.

持续构建不仅仅是于编译相联系，它为所有类型的任务工作，例如，processResources人物，COPY资源文件到jar包，类似如此，改变文件夹内的任何文件都会触发一次新的构建。

## **19.3. Limitations and quirks**

19.3 限制问题

There are several issues to be aware with the current implementation of continuous build. These are likely to be addressed in future Gradle releases.

当前持续构建还有数个问题在open状态，这些问题将会在文莱的gradle release版本中被逐步修复。

### **19.3.1. Requires Java 7 or later**

19.3.1 要求java7以上版本

Gradle uses the JDK's WatchService to receive notification of changes to files between builds. This API was introduced in Java 7. As such, Gradle's continuous build is not currently supported when building with Java 6.

Gradle使用JDK的watchservice来接受构建间得文件变化通知，这个API在java7后才支持，所以gradle 持续构建当前不仅仅支持java6，也支持7以及以上版本。

### **19.3.2. Performance and stability on Mac OS X**

19.3.2 Mac OS X系统上的性能和稳定性

The JDK file watching facility relies on inefficient file system polling on Mac OS X (see: JDK-7133447). This can significantly delay notification of changes on large projects with many source files.

JDK监控服务在Mac OS X JDK-7133447上是无效的，监控不到大项目下原文件的若干改变。

Additionally, the watching mechanism may deadlock under heavy load on Mac OS X (see: JDK-8079620). This will manifest as Gradle appearing not to notice file changes. If you suspect this is occurring, exit continuous build and start again.

在JDK-8079620上，监控无法会死锁，也会导致Gradle监控不到文件的变化，如果你想触发构建，需要手动重新启动持续构建。

### **19.3.3. Changes to symbolic links**

19.3.3. 修改为软链接方式

Creating or removing symbolic link to files will initiate a build.

创建或者移除文件软链接会引发一个构建

Modifying the target of a symbolic link will not cause a rebuild.

修改软连接目标不会触发重新构建

Creating or removing symbolic links to directories will not cause rebuilds.

创建或者移除文件夹的软链接不会引发重新构建

Creating new files in the target directory of a symbolic link will not cause a rebuild.

创建一个新的文件并且添加软链接不会引发新构建

Deleting the target directory will not cause a rebuild.

删除目标目录不会引发重新构建

### **19.3.4. Changes to build logic are not considered**

19.3.4. 修改构建逻辑不在考虑范围内

The current implementation does not recalculate the build model on subsequent builds. This means that changes to task configuration, or any other change to the build model, are effectively ignored.

改变任务配置，以及构建模型产生的效果都会被忽略，所以也不回触发自动构建。