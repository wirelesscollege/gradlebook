# **Chapter 57. Writing Custom Task Classes**

Gradle支持两种类型的任务。一种类型是一种你可以使用闭包动作定义的简单任务。我们在第六章 构建脚本基础中看到了这些。对于这种任务类型，闭包动作判断任务的动作。这种任务类型有利于实现构建脚本中一次性的任务

Gradle supports two types of task. One such type is the simple task, where you define the task with an action closure. We have seen these in Chapter 6, Build Script Basics. For this type of task, the action closure determines the behaviour of the task. This type of task is good for implementing one-off tasks in your build script.

另一种类型任务是增强任务,其中行为内置在任务里,该任务提供了一些属性,您可以使用它配置行为。我们已经在第14章看到了这些, More about Tasks。大多数Gradle插件使用增强任务。使用增强任务,你不需要实现和用简单任务一样的任务行为。只需声明任务和使用其属性配置任务。通过这种方式,增强任务让你在许多不同的地方可重用一个行为,可能会交叉不同的构建。

The other type of task is the enhanced task, where the behaviour is built into the task, and the task provides some properties which you can use to configure the behaviour. We have seen these in Chapter 14, More about Tasks. Most Gradle plugins use enhanced tasks. With enhanced tasks, you don't need to implement the task behaviour as you do with simple tasks. You simply declare the task and configure the task using its properties. In this way, enhanced tasks let you reuse a piece of behaviour in many different places, possibly across different builds.

增强型任务的行为和属性是由任务的类来定义的。当你声明增强型任务的时候，你就明确了其类型和任务类。

The behaviour and properties of an enhanced task is defined by the task's class. When you declare an enhanced task, you specify the type, or class of the task.

在Gradle中实现你自己定义的任务类很容易。您可以实现一个自定义的任务类几乎可以使用任何你喜欢的语言,最终是以编译成字节码的形式提供。在我们的示例中,我们将使用Groovy作为实现语言,但是你可以使用,例如,Java或Scala语言。一般来说,使用Groovy语言是最简单的选择,因为Gradle API与Groovy运行的很好。

Implementing your own custom task class in Gradle is easy. You can implement a custom task class in pretty much any language you like, provided it ends up compiled to bytecode. In our examples, we are going to use Groovy as the implementation language, but you could use, for example, Java or Scala. In general, using Groovy is the easiest option, because the Gradle API is designed to work well with Groovy.

## **57.1. Packaging a task class**

你可以把任务类的源代码放在很多地方。

There are several places where you can put the source for the task class.
Build script

你可以直接在构建脚本中包含任务类。这样的好处是:任务是自动编译和包含在构建脚本类路径里,你无须做任何事。然而,构建脚本外的任务类是不可见的,所以你不能重用定义在构建脚本外的任务类。

You can include the task class directly in the build script. This has the benefit that the task class is automatically compiled and included in the classpath of the build script without you having to do anything. However, the task class is not visible outside the build script, and so you cannot reuse the task class outside the build script it is defined in.

buildSrc project

你可以把任务类的源代码放在目录rootProjectDir/buildSrc/src/main/groovy中。Gradle将注意编译和测试任务类，使其可以在构建脚本类路径中执行。任务类对于构建使用的每个构建脚本都是可见的。然而,在构建外它是不可见的。所以你不能重用构建外的任务类。使用buildSrc项目方法分离任务声明——也就是说,这个任务应该做什么——从任务的实现来看就是,任务是如何做的。

You can put the source for the task class in the rootProjectDir/buildSrc/src/main/groovy directory. Gradle will take care of compiling and testing the task class and making it available on the classpath of the build script. The task class is visible to every build script used by the build. However, it is not visible outside the build, and so you cannot reuse the task class outside the build it is defined in. Using the buildSrc project approach separates the task declaration - that is, what the task should do - from the task implementation - that is, how the task does it.

见59章、组织构建逻辑 可了解更多关于buildSrc project的详细信息.

See Chapter 59, Organizing Build Logic for more details about the buildSrc project.

Standalone project

你可以为你的任务类创建一个单独的项目。这个项目可以生成和发布一个jar文件，你可以在多种构建中使用和共享。一般来说，这个jar文件可能会包含一些定义的插件、绑定若干相关联的任务类，或一些这两者的结合到一个单独的库文件中。

You can create a separate project for your task class. This project produces and publishes a JAR which you can then use in multiple builds and share with others. Generally, this JAR might include some custom plugins, or bundle several related task classes into a single library. Or some combination of the two.

在我们的实例中，为了简单化我们将会以构建脚本中的插件开始。之后我们将看下创建一个独立的项目。

In our examples, we will start with the task class in the build script, to keep things simple. Then we will look at creating a standalone project.

## **57.2. Writing a simple task class**

为实现一个定义的任务类，你要扩展DefaultTask

To implement a custom task class, you extend DefaultTask.

例57.1 定义一个定制任务

Example 57.1. Defining a custom task

build.gradle
```
class GreetingTask extends DefaultTask {
}
```

这个任务没做任何有用的事情，所以我们要加些行为。为了这样做，我们给任务添加了个方法，并标注了TaskAction注释。当任务执行的时候，Gradle会调用这个方法。你没有必要使用一个方法为此任务定义行为。例如你可以使用任务函数的闭包调用doFirst() 或 doLast()来添加行为。

This task doesn't do anything useful, so let's add some behaviour. To do so, we add a method to the task and mark it with the TaskAction annotation. Gradle will call the method when the task executes. You don't have to use a method to define the behaviour for the task. You could, for instance, call doFirst() or doLast() with a closure in the task constructor to add behaviour.

例57.2  一个hello world 任务

Example 57.2. A hello world task

build.gradle
```
task hello(type: GreetingTask)

class GreetingTask extends DefaultTask {
    @TaskAction
    def greet() {
        println 'hello from GreetingTask'
    }
}
```
Output of gradle -q hello
```
> gradle -q hello
hello from GreetingTask
```

我们给一个任务添加个属性，那么我们就可以定义它。任务是简单的POGOs，当你声明一个任务，你就可以设置属性或者调用此任务对象的方法。这里添加一个greeting的属性，并且当我们声明greeting任务的时候就设置该属性的值。

Let's add a property to the task, so we can customize it. Tasks are simply POGOs, and when you declare a task, you can set the properties or call methods on the task object. Here we add a greeting property, and set the value when we declare the greeting task.

例57.3  一个自定义的hello world任务

Example 57.3. A customizable hello world task

build.gradle
```
// Use the default greeting
task hello(type: GreetingTask)

// Customize the greeting
task greeting(type: GreetingTask) {
    greeting = 'greetings from GreetingTask'
}

class GreetingTask extends DefaultTask {
    String greeting = 'hello from GreetingTask'

    @TaskAction
    def greet() {
        println greeting
    }
}
```

Output of gradle -q hello greeting
```
> gradle -q hello greeting
hello from GreetingTask
greetings from GreetingTask
```

## **57.3. A standalone project**

现在我们将我们的任务转移到一个独立的项目,所以我们可以发布它,并分享。这个项目是一个简单的Groovy项目,生成一个包含任务类的JAR。这是一个简单的项目构建脚本。它适用于Groovy插件,并添加Gradle API作为编译时依赖项。

Now we will move our task to a standalone project, so we can publish it and share it with others. This project is simply a Groovy project that produces a JAR containing the task class. Here is a simple build script for the project. It applies the Groovy plugin, and adds the Gradle API as a compile-time dependency.

例57.4  一个自定义任务的构建

Example 57.4. A build for a custom task

build.gradle
```
apply plugin: 'groovy'

dependencies {
    compile gradleApi()
    compile localGroovy()
}
```

Note: The code for this example can be found at samples/customPlugin/plugin in the ‘-all’ distribution of Gradle.

对于任务类的源码应该在哪里，我们仅仅是依据惯例。

We just follow the convention for where the source for the task class should go.

例57.5   一个自定义任务

Example 57.5. A custom task

src/main/groovy/org/gradle/GreetingTask.groovy
```
package org.gradle

import org.gradle.api.DefaultTask
import org.gradle.api.tasks.TaskAction

class GreetingTask extends DefaultTask {
    String greeting = 'hello from GreetingTask'

    @TaskAction
    def greet() {
        println greeting
    }
}
```

## **57.3.1. Using your task class in another project**

为在构建脚本时使用某任务类,您需要添加类到构建脚本的类路径中。要做到这一点,你需要用一个“buildscript { }”块,如59.6节所述,“构建脚本的外部依赖”。下面的例子显示当载有任务类的JAR包已经发布到一个本地存储库时，你该如何做到这一点:

To use a task class in a build script, you need to add the class to the build script's classpath. To do this, you use a buildscript { } block, as described in Section 59.6, “External dependencies for the build script”. The following example shows how you might do this when the JAR containing the task class has been published to a local repository:

例57.6 在其他的项目使用自定义任务

Example 57.6. Using a custom task in another project

build.gradle
```
buildscript {
    repositories {
        maven {
            url uri('../repo')
        }
    }
    dependencies {
        classpath group: 'org.gradle', name: 'customPlugin',
                  version: '1.0-SNAPSHOT'
    }
}

task greeting(type: org.gradle.GreetingTask) {
    greeting = 'howdy!'
}
```

## **57.3.2. Writing tests for your task class**

当你测试任务类时候，可以使用ProjectBuilder类来创建Project实例使用。

You can use the ProjectBuilder class to create Project instances to use when you test your task class.

例57.7 测试一个自定义任务

Example 57.7. Testing a custom task

src/test/groovy/org/gradle/GreetingTaskTest.groovy
```
class GreetingTaskTest {
    @Test
    public void canAddTaskToProject() {
        Project project = ProjectBuilder.builder().build()
        def task = project.task('greeting', type: GreetingTask)
        assertTrue(task instanceof GreetingTask)
    }
}
```

## **57.4. Incremental tasks**

增量任务是个待开发的特性

Incremental tasks are an incubating feature.

由于之前描述的实现的介绍（早在Gradle 1.6 的发布周期），在Gradle社区中的讨论已经生成了优越的想法，就是如同下面的描述一样公开更改的信息给任务实现者。如同这样，这个特性的API将会在未来的版本中几乎一定会变更。然而，请与当前的实现做实验并且在论坛中分享经验。

Since the introduction of the implementation described above (early in the Gradle 1.6 release cycle), discussions within the Gradle community have produced superior ideas for exposing the information about changes to task implementors to what is described below. As such, the API for this feature will almost certainly change in upcoming releases. However, please do experiment with the current implementation and share your experiences with the Gradle community.

孵化过程的特性,它是Gradle特性生命周期的一部分(请参阅附录C,生命周期),为此它的存在性在于通过结合早期用户的反馈确保高质量的最终实现。

The feature incubation process, which is part of the Gradle feature lifecycle (see Appendix C, The Feature Lifecycle), exists for this purpose of ensuring high quality final implementations through incorporation of early user feedback.

使用Gradle,很容易实现一个任务，即当它所有的输入和输出是最新的的时候跳过此任务(参见14.9节,“跳过最新的任务”)。然而,有些时候只有几个输入文件相对上次执行改变了,那么你就想要避免再次加工所有那些没改变的输入。这对一个转换器的任务特别有用,它会将输入文件以一对一的比例转换为输出文件。

With Gradle, it's very simple to implement a task that gets skipped when all of it's inputs and outputs are up to date (see Section 14.9, “Skipping tasks that are up-to-date”). However, there are times when only a few input files have changed since the last execution, and you'd like to avoid reprocessing all of the unchanged inputs. This can be particularly useful for a transformer task, that converts input files to output files on a 1:1 basis.

如果你想优化你的构建使得处理仅过期的输出，那么你可以使用增强任务。

If you'd like to optimise your build so that only out-of-date inputs are processed, you can do so with an incremental task.

### **57.4.1. Implementing an incremental task**

对于一个处理输出不断增值的任务,它必须包含增量任务动作。这个任务动作方法包含一个IncrementalTaskInputs参数,这表明对于Gradle的此动作将仅处理改变的输入。

For a task to process inputs incrementally, that task must contain an incremental task action. This is a task action method that contains a single IncrementalTaskInputs parameter, which indicates to Gradle that the action will process the changed inputs only.

增量任务动作可能会提供一个IncrementalTaskInputs.outOfDate()动作来处理任何过时的输入文件,还有一个IncrementalTaskInputs.removed()动作  用来执行在之前执行过程中被删除的任何输入文件。

The incremental task action may supply an IncrementalTaskInputs.outOfDate() action for processing any input file that is out-of-date, and a IncrementalTaskInputs.removed() action that executes for any input file that has been removed since the previous execution.

例57.8 定义一个增量任务行为

Example 57.8. Defining an incremental task action

build.gradle
```
class IncrementalReverseTask extends DefaultTask {
    @InputDirectory
    def File inputDir

    @OutputDirectory
    def File outputDir

    @Input
    def inputProperty

    @TaskAction
    void execute(IncrementalTaskInputs inputs) {
        println inputs.incremental ? "CHANGED inputs considered out of date"
                                   : "ALL inputs considered out of date"
        inputs.outOfDate { change ->
            println "out of date: ${change.file.name}"
            def targetFile = new File(outputDir, change.file.name)
            targetFile.text = change.file.text.reverse()
        }

        inputs.removed { change ->
            println "removed: ${change.file.name}"
            def targetFile = new File(outputDir, change.file.name)
            targetFile.delete()
        }
    }
}
```

Note: The code for this example can be found at samples/userguide/tasks/incrementalTask in the ‘-all’ distribution of Gradle.

对于这样一个简单的转换器任务,任务动作只需要为过时的输入文件生成输出文件，且为任何移除的输入删除输出文件。

For a simple transformer task like this, the task action simply needs to generate output files for any out-of-date inputs, and delete output files for any removed inputs.

一个任务可能仅包含一个单独的增量任务动作。

A task may only contain a single incremental task action.

### **57.4.2. Which inputs are considered out of date?**

当Gradle有之前任务执行历史记录，且仅相对之前任务执行改变的内容写进了输入文件，那么Gradle能够判定哪个输入文件需要被任务重新处理。这种情况下，IncrementalTaskInputs.outOfDate()动作可以执行应用于任何添加和修改的输入文件。IncrementalTaskInputs.removed()动作可以执行用于任何移除的输入文件。

When Gradle has history of a previous task execution, and the only changes to the task execution context since that execution are to input files, then Gradle is able to determine which input files need to be reprocessed by the task. In this case, the IncrementalTaskInputs.outOfDate() action will be executed for any input file that was added or modified, and the IncrementalTaskInputs.removed() action will be executed for any removed input file.

然而，很多情况下Gradle是不能判定哪个输入文件需要被再次处理。例子包括：

However, there are many cases where Gradle is unable to determine which input files need to be reprocessed. Examples include:

没有从前执行可用的历史记录。

There is no history available from a previous execution.

你正在使用一个不同的Gradle版本来进行构建。目前，Gradle不能使用来自不同版本的任务历史。

You are building with a different version of Gradle. Currently, Gradle does not use task history from a different version.

添加到任务中的upToDateWhen条件返回为false.

An upToDateWhen criteria added to the task returns false.

相对之前的执行现在的输入属性已经改变了。

An input property has changed since the previous execution.

相对之前的执行 一个或多个输出文件已经改变了。

One or more output files have changed since the previous execution.

在这些情况下，Gradle会认为所有的输入文件都是过时的。每个输入文件都会执行IncrementalTaskInputs.outOfDate()
动作，而IncrementalTaskInputs.removed()动作根本不会被执行。

In any of these cases, Gradle will consider all of the input files to be outOfDate. The IncrementalTaskInputs.outOfDate() action will be executed for every input file, and the IncrementalTaskInputs.removed() action will not be executed at all.

你可以验证下Gradle是否可以通过IncrementalTaskInputs.isIncremental()来判断输入文件新增的变化。

You can check if Gradle was able to determine the incremental changes to input files with IncrementalTaskInputs.isIncremental().

### **57.4.3. An incremental task in action**

鉴于上述增量任务的实现,我们可以通过例子探索各种变化场景。注意各种突变任务(“updateInputs”、“removeInput”等)只是出于当前演示目的:这些通常不会是你构建脚本的一部分。

Given the incremental task implementation above, we can explore the various change scenarios by example. Note that the various mutation tasks ('updateInputs', 'removeInput', etc) are only present for demonstration purposes: these would not normally be part of your build script.

首先,考虑IncrementalReverseTask首次对一组输入执行。在这种情况下,所有的输入都将被认为是“过时的”:

First, consider the IncrementalReverseTask executed against a set of inputs for the first time. In this case, all inputs will be considered “out of date”:

例57.9  首次运行增量任务

Example 57.9. Running the incremental task for the first time

build.gradle
```
task incrementalReverse(type: IncrementalReverseTask) {
    inputDir = file('inputs')
    outputDir = file("$buildDir/outputs")
    inputProperty = project.properties['taskInputProperty'] ?: "original"
}
Build layout
incrementalTask/
  build.gradle
  inputs/
    1.txt
    2.txt
    3.txt
```
Output of gradle -q incrementalReverse
```
> gradle -q incrementalReverse
ALL inputs considered out of date
out of date: 1.txt
out of date: 2.txt
out of date: 3.txt
```

自然地若没有变化再次执行任务,那么整个任务是最新的没有文件传送给任务动作:

Naturally when the task is executed again with no changes, then the entire task is up to date and no files are reported to the task action:

例57.10  使用没有改变的输入文件运行增量任务

Example 57.10. Running the incremental task with unchanged inputs

Output of gradle -q incrementalReverse
```
> gradle -q incrementalReverse
When an input file is modified in some way or a new input file is added, then re-executing the task results in those files being reported to IncrementalTaskInputs.outOfDate():
```
例57.11  使用过时的输入文件运行增量任务

Example 57.11. Running the incremental task with updated input files

build.gradle
```
task updateInputs() << {
    file('inputs/1.txt').text = "Changed content for existing file 1."
    file('inputs/4.txt').text = "Content for new file 4."
}
Output of gradle -q updateInputs incrementalReverse
> gradle -q updateInputs incrementalReverse
CHANGED inputs considered out of date
out of date: 1.txt
out of date: 4.txt
```

当一个存在的输入文件被移除，那么再次执行这项任务将会导致那个文件被传送给IncrementalTaskInputs.removed()。

When an existing input file is removed, then re-executing the task results in that file being reported to IncrementalTaskInputs.removed():

例57.12 使用已移除的输入文件运行增量任务

Example 57.12. Running the incremental task with an input file removed

build.gradle
```
task removeInput() << {
    file('inputs/3.txt').delete()
}
Output of gradle -q removeInput incrementalReverse
> gradle -q removeInput incrementalReverse
CHANGED inputs considered out of date
removed: 3.txt
```

当一个输出文件被删除（或被修改），此时Gradle不能判定哪个输入文件过时。在这种情况下，所有的输入文件被报给ncrementalTaskInputs.outOfDate()行为，而没有输出文件被报给IncrementalTaskInputs.removed()。

When an output file is deleted (or modified), then Gradle is unable to determine which input files are out of date. In this case, all input files are reported to the IncrementalTaskInputs.outOfDate() action, and no input files are reported to the IncrementalTaskInputs.removed() action:

例57.13  使用已移除的输出文件运行增量任务

Example 57.13. Running the incremental task with an output file removed

build.gradle
```
task removeOutput() << {
    file("$buildDir/outputs/1.txt").delete()
}
Output of gradle -q removeOutput incrementalReverse
> gradle -q removeOutput incrementalReverse
ALL inputs considered out of date
out of date: 1.txt
out of date: 2.txt
out of date: 3.txt
```

当任务输入属性被修改时,Gradle无法判断这个属性如何影响任务输出,所以所有输入文件被认为是过时了。因此类似于改变的输出文件的例子,所有输入文件传送给IncrementalTaskInputs.outOfDate()操作,并没有输入文件传送给IncrementalTaskInputs.removed():

When a task input property is modified, Gradle is unable to determine how this property impacted the task outputs, so all input files are assumed to be out of date. So similar to the changed output file example, all input files are reported to the IncrementalTaskInputs.outOfDate() action, and no input files are reported to the IncrementalTaskInputs.removed() action:

例57.14  使用改变的输入属性运行增量任务

Example 57.14. Running the incremental task with an input property changed

```
Output of gradle -q -PtaskInputProperty=changed incrementalReverse
> gradle -q -PtaskInputProperty=changed incrementalReverse
ALL inputs considered out of date
out of date: 1.txt
out of date: 2.txt
out of date: 3.txt
```
百度搜索[无线学院](http://wirelesscollege.cn)
