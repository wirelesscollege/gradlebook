# **第十五章. 任务详述**
在入门教程 （[第 6 章，构建脚本基础](chapter_6_overview.md)） 中，你已经学习了如何创建简单的任务。之后您还学习了如何将其他行为添加到这些任务中。并且你已经学会了如何创建任务之间的依赖。这都是简单的任务。但 Gradle 让任务的概念更深远。Gradle 支持增强的任务，也就是，有自己的属性和方法的任务。这是真正的与你所使用的 Ant 目标（target）的不同之处。这种增强的任务可以由你提供，或由 Gradle 提供。

## **15.1. 定义任务**
在第 6 章，构建脚本基础 中我们已经看到如何通过关键字这种风格来定义任务。在某些情况中，你可能需要使用这种关键字风格的几种不同的变式。例如，在表达式中不能用这种关键字风格。

示例 15.1. 定义任务

build.gradle
```
task(hello) << {
    println "hello"
}

task(copy, type: Copy) {
    from(file('srcDir'))
    into(buildDir)
}
```
您还可以使用字符串作为任务名称：

示例 15.2. 定义任务 — — 使用字符串作为任务名称

build.gradle
```
task('hello') <<
{
    println "hello"
}

task('copy', type: Copy) {
    from(file('srcDir'))
    into(buildDir)
}
```
对于定义任务，有一种替代的语法你可能更愿意使用：

示例 15.3. 使用替代语法定义任务

build.gradle
```
tasks.create(name: 'hello') << {
    println "hello"
}

tasks.create(name: 'copy', type: Copy) {
    from(file('srcDir'))
    into(buildDir)
}
```
在这里我们将任务添加到tasks集合。关于create ()方法的更多变化可以看看TaskContainer。

15.2. 定位任务
你经常需要在构建文件中查找你所定义的任务，例如，为了去配置或是依赖它们。对这样的情况，有很多种方法。首先，每个任务都可作为项目的一个属性，并且使用任务名称作为这个属性名称：

示例 15.4. 以属性方式访问任务

build.gradle
```
task hello

println hello.name
println project.hello.name
```
任务也可以通过tasks集合来访问。

示例 15.5. 通过tasks集合访问任务

build.gradle
```
task hello

println tasks.hello.name
println tasks['hello'].name
```
您可以从任何项目中，使用tasks.getByPath()方法获取任务路径并且通过这个路径来访问任务。你可以用任务名称，相对路径或者是绝对路径作为参数调用getByPath()方法。

示例 15.6. 通过路径访问任务

build.gradle
```
project(':projectA') {
    task hello
}

task hello

println tasks.getByPath('hello').path
println tasks.getByPath(':hello').path
println tasks.getByPath('projectA:hello').path
println tasks.getByPath(':projectA:hello').path
```

gradle -q hello的输出结果
```
> gradle -q hello
:hello
:hello
:projectA:hello
:projectA:hello
```
有关查找任务的更多选项，可以看一下TaskContainer。

## **15.3. .配置任务**
作为一个例子，让我们看看由 Gradle 提供的Copy任务。若要创建Copy任务，您可以在构建脚本中声明：

示例 15.7. 创建一个复制任务

build.gradle
```
task myCopy(type: Copy)
```
上面的代码创建了一个什么都没做的复制任务。可以使用它的 API 来配置这个任务 （见Copy）。下面的示例演示了几种不同的方式来实现相同的配置。

示例 15.8. 配置任务的几种方式

build.gradle
```
Copy myCopy = task(myCopy, type: Copy)
myCopy.from 'resources'
myCopy.into 'target'
myCopy.include('**/*.txt', '**/*.xml', '**/*.properties')
```
这类似于我们通常在 Java 中配置对象的方式。您必须在每一次的配置语句重复上下文 （myCopy）。这显得很冗余并且很不好读。

还有另一种配置任务的方式。它也保留了上下文，且可以说是可读性最强的。它是我们通常最喜欢的方式。

示例 15.9. 配置任务-使用闭包

build.gradle
```
task myCopy(type: Copy)

myCopy {
   from 'resources'
   into 'target'
   include('**/*.txt', '**/*.xml', '**/*.properties')
}
```
这种方式适用于任何任务。该例子的第 3 行只是tasks.getByName()方法的简洁写法。特别要注意的是，如果您向getByName()方法传入一个闭包，这个闭包的应用是在配置这个任务的时候，而不是任务执行的时候。

您也可以在定义一个任务的时候使用一个配置闭包。

示例 15.10. 使用闭包定义任务

build.gradle
```
task copy(type: Copy) {
   from 'resources'
   into 'target'
   include('**/*.txt', '**/*.xml', '**/*.properties')
}
```
## **15.4. 对任务添加依赖**
定义任务的依赖关系有几种方法。在第 6.5 章节，"任务依赖"中，已经向你介绍了使用任务名称来定义依赖。任务的名称可以指向同一个项目中的任务，或者其他项目中的任务。要引用另一个项目中的任务，你需要把它所属的项目的路径作为前缀加到它的名字中。下面是一个示例，添加了从projectA:taskX到projectB:taskY的依赖关系：

示例 15.11. 从另一个项目的任务上添加依赖

build.gradle
```
project('projectA') {
    task taskX(dependsOn: ':projectB:taskY') << {
        println 'taskX'
    }
}

project('projectB') {
    task taskY << {
        println 'taskY'
    }
}
```
gradle -q taskX的输出结果
```
> gradle -q taskX
taskY
taskX
```
您可以使用一个Task对象而不是任务名称来定义依赖，如下：

示例 15.12. 使用 task 对象添加依赖

build.gradle
```
task taskX << {
    println 'taskX'
}

task taskY << {
    println 'taskY'
}

taskX.dependsOn taskY
```
gradle -q taskX的输出结果

```
> gradle -q taskX
taskY
taskX
```
对于更高级的用法，您可以使用闭包来定义任务依赖。在计算依赖时，闭包会被传入正在计算依赖的任务。这个闭包应该返回一个 Task 对象或是Task 对象的集合，返回值会被作为这个任务的依赖项。下面的示例是从taskX加入了项目中所有名称以lib开头的任务的依赖：

示例 15.13. 使用闭包添加依赖

build.gradle
```
task taskX << {
    println 'taskX'
}

taskX.dependsOn {
    tasks.findAll { task -> task.name.startsWith('lib') }
}

task lib1 << {
    println 'lib1'
}

task lib2 << {
    println 'lib2'
}

task notALib << {
    println 'notALib'
}
```
gradle -q taskX的输出结果

```
> gradle -q taskX
lib1
lib2
taskX
```
有关任务依赖的详细信息，请参阅Task的 API。

## **15.5. 任务排序**
任务排序还是一个孵化中的功能。请注意此功能在以后的 Gradle 版本中可能会改变。
在某些情况下，控制两个任务的执行的顺序，而不引入这些任务之间的显式依赖，是很有用的。任务排序和任务依赖之间的主要区别是，排序规则不会影响那些任务的执行，而仅将执行的顺序。

任务排序在许多情况下可能很有用：

强制任务顺序执行： 如，'build' 永远不会在 'clean' 前面执行。
在构建中尽早进行构建验证：如，验证在开始发布的工作前有一个正确的证书。
通过在长久验证前运行快速验证以得到更快的反馈：如，单元测试应在集成测试之前运行。
一个任务聚合了某一特定类型的所有任务的结果：如，测试报告任务结合了所有执行的测试任务的输出。
有两种排序规则是可用的："必须在之后运行"和"应该在之后运行"。

通过使用 “ 必须在之后运行”的排序规则，您可以指定 taskB 必须总是运行在 taskA 之后，无论taskA和taskB这两个任务在什么时候被调度执行。这被表示为 taskB.mustRunAfter(taskA) 。“应该在之后运行”的排序规则与其类似，但没有那么严格，因为它在两种情况下会被忽略。首先是如果使用这一规则引入了一个排序循环。其次，当使用并行执行，并且一个任务的所有依赖项除了任务应该在之后运行之外所有条件已满足，那么这个任务将会运行，不管它的“应该在之后运行”的依赖项是否已经运行了。当倾向于更快的反馈时，会使用“应该在之后运行”的规则，因为这种排序很有帮助但要求不严格。

目前使用这些规则仍有可能出现taskA执行而taskB 没有执行，或者taskB执行而taskA 没有执行。

示例 15.14. 添加 '必须在之后运行 ' 的任务排序

build.gradle
```
task taskX << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}
taskY.mustRunAfter taskX
```
gradle -q taskY taskX 的输出结果
```
> gradle -q taskY taskX
taskX
taskY
```
示例 15.15. 添加 '应该在之后运行 ' 的任务排序

build.gradle
```
task taskX << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}
taskY.shouldRunAfter taskX
```

gradle -q taskY taskX 的输出结果
```
> gradle -q taskY taskX
taskX
taskY
```
在上面的例子中，它仍有可能执行taskY而不会导致taskX也运行：

示例 15.16. 任务排序并不意味着任务执行

gradle -q taskY 的输出结果
```
> gradle -q taskY
taskY
```
如果想指定两个任务之间的“必须在之后运行”和“应该在之后运行”排序，可以使用Task.mustRunAfter()和Task.shouldRunAfter()方法。这些方法接受一个任务实例、 任务名称或Task.dependsOn()所接受的任何其他输入作为参数。

请注意"B.mustRunAfter(A)"或"B.shouldRunAfter(A)"并不意味着这些任务之间的任何执行上的依赖关系：

它是可以独立地执行任务A和B 的。排序规则仅在这两项任务计划执行时起作用。
当--continue参数运行时，可能会是A执行失败后B执行了。
如之前所述，如果“应该在之后运行”的排序规则引入了排序循环，那么它将会被忽略。

示例 15.17. 当引入循环时，“应该在其之后运行”的任务排序会被忽略

build.gradle
```
task taskX << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}
task taskZ << {
    println 'taskZ'
}
taskX.dependsOn taskY
taskY.dependsOn taskZ
taskZ.shouldRunAfter taskX
```
gradle -q taskX的输出结果
```
> gradle -q taskX
taskZ
taskY
taskX
```
## **15.6. 向任务添加描述**
你可以向你的任务添加描述。例如，当执行gradle tasks时显示这个描述。

示例 15.18. 向任务添加描述

build.gradle
```
task copy(type: Copy) {
   description 'Copies the resource directory to the target directory.'
   from 'resources'
   into 'target'
   include('**/*.txt', '**/*.xml', '**/*.properties')
}
```
## **15.7. 替换任务**
有时您想要替换一个任务。例如，您想要把通过 Java 插件添加的一个任务与不同类型的一个自定义任务进行交换。你可以这样实现：

示例 15.19. 重写任务

build.gradle
```
task copy(type: Copy)

task copy(overwrite: true) << {
    println('I am the new one.')
}
```
gradle -q copy 的输出结果
```
> gradle -q copy
I am the new one.
```
在这里我们用一个简单的任务替换Copy类型的任务。当创建这个简单的任务时，您必须将overwrite属性设置为 true。否则 Gradle 将抛出异常，说这种名称的任务已经存在。

## **15.8. 跳过任务**
Gradle 提供多种方式来跳过任务的执行。

### **15.8.1. 使用断言**

你可以使用onlyIf()方法将断言附加到一项任务中。如果断言结果为 true，才会执行任务的操作。你可以用一个闭包来实现断言。闭包会作为一个参数传给任务，并且任务应该执行时返回true，或任务应该跳过时返回false。断言只在任务要执行前才计算。

示例 15.20. 使用断言跳过一个任务

build.gradle
```
task hello << {
    println 'hello world'
}

hello.onlyIf { !project.hasProperty('skipHello') }
```
gradle hello -PskipHello的输出结果

```
> gradle hello -PskipHello
:hello SKIPPED


BUILD SUCCESSFUL

Total time: 1 secs
```

### **15.8.2. 使用 StopExecutionException*

如果跳过任务的规则不能与断言同时表达，您可以使用StopExecutionException。如果一个操作（action）抛出了此异常，那么这个操作（action）接下来的行为和这个任务的其他 操作（action）都会被跳过。构建会继续执行下一个任务。

示例 15.21. 使用 StopExecutionException 跳过任务

build.gradle

```
task compile << {
    println 'We are doing the compile.'
}

compile.doFirst {
    // Here you would put arbitrary conditions in real life. But we use this as an integration test, so we want defined behavior.
    if (true) { throw new StopExecutionException() }
}
task myTask(dependsOn: 'compile') << {
   println 'I am not affected'
}
```

gradle -q myTask 的输出结果

```
> gradle -q myTask
I am not affected
```
如果您使用由 Gradle 提供的任务，那么此功能将非常有用。它允许您向一个任务的内置操作中添加执行条件。[7]

### **15.8.3. 启用和禁用任务**

每一项任务有一个默认值为true的enabled标记。将它设置为false，可以不让这个任务的任何操作执行。

示例 15.22. 启用和禁用任务

build.gradle
```
task disableMe << {
    println 'This should not be printed if the task is disabled.'
}
disableMe.enabled = false
```
Gradle disableMe的输出结果
```
> gradle disableMe
:disableMe SKIPPED

BUILD SUCCESSFUL

Total time: 1 secs
```

## **15.9. 跳过处于最新状态的任务**
如果您使用 Gradle 自带的任务，如 Java 插件所添加的任务的话，你可能已经注意到 Gradle 将跳过处于最新状态的任务。这种行在您自己定义的任务上也有效，而不仅仅是内置任务。

15.9.1. 声明一个任务的输入和输出

让我们来看一个例子。在这里我们的任务从一个 XML 源文件生成多个输出文件。让我们运行它几次。

示例 15.23. 一个生成任务

build.gradle
```
task transform {
    ext.srcFile = file('mountains.xml')
    ext.destDir = new File(buildDir, 'generated')
    doLast {
        println "Transforming source file."
        destDir.mkdirs()
        def mountains = new XmlParser().parse(srcFile)
        mountains.mountain.each { mountain ->
            def name = mountain.name[0].text()
            def height = mountain.height[0].text()
            def destFile = new File(destDir, "${name}.txt")
            destFile.text = "$name -> ${height}\n"
        }
    }
}
```
gradle transform的输出结果
```
> gradle transform
:transform
Transforming source file.
```
gradle transform的输出结果
```
> gradle transform
:transform
Transforming source file.
```
请注意 Gradle 第二次执行执行这项任务时，即使什么都未作改变，也没有跳过该任务。我们的示例任务被用一个操作（action）闭包来定义。Gradle 不知道这个闭包做了什么，也无法自动判断这个任务是否为最新状态。若要使用 Gradle 的最新状态（up-to-date）检查，您需要声明这个任务的输入和输出。

每个任务都有一个inputs和outputs的属性，用来声明任务的输入和输出。下面，我们修改了我们的示例，声明它将 XML 源文件作为输入，并产生输出到一个目标目录。让我们运行它几次。

示例 15.24. 声明一个任务的输入和输出

build.gradle
```
task transform {
    ext.srcFile = file('mountains.xml')
    ext.destDir = new File(buildDir, 'generated')
    inputs.file srcFile
    outputs.dir destDir
    doLast {
        println "Transforming source file."
        destDir.mkdirs()
        def mountains = new XmlParser().parse(srcFile)
        mountains.mountain.each { mountain ->
            def name = mountain.name[0].text()
            def height = mountain.height[0].text()
            def destFile = new File(destDir, "${name}.txt")
            destFile.text = "$name -> ${height}\n"
        }
    }
}
```
gradle transform的输出结果
```
> gradle transform
:transform
Transforming source file.
```
gradle transform的输出结果
```
> gradle transform
:transform UP-TO-DATE
```
现在，Gradle 知道哪些文件要检查以确定任务是否为最新状态。

任务的 inputs 属性是 TaskInputs类型。任务的 outputs 属性是 TaskOutputs类型。

一个没有定义输出的任务将永远不会被当作是最新的。对于任务的输出并不是文件的场景，或者是更复杂的场景， TaskOutputs.upToDateWhen()方法允许您以编程方式计算任务的输出是否应该被判断为最新状态。

一个只定义了输出的任务，如果自上一次构建以来它的输出没有改变，那么它会被判定为最新状态。

### **15.9.2. 它是怎么实现的？**

在第一次执行任务之前，Gradle 对输入进行一次快照。这个快照包含了输入文件集和每个文件的内容的哈希值。然后 Gradle 执行该任务。如果任务成功完成，Gradle 将对输出进行一次快照。该快照包含输出文件集和每个文件的内容的哈希值。Gradle 会保存这两个快照，直到任务的下一次执行。

之后每一次，在执行任务之前，Gradle 会对输入和输出进行一次新的快照。如果新的快照和前一次的快照一样，Gradle 会假定这些输出是最新状态的并跳过该任务。如果它们不一则， Gradle 则会执行该任务。Gradle 会保存这两个快照，直到任务的下一次执行。

请注意，如果一个任务有一个指定的输出目录，在它上一次执行之后添加到该目录的所有文件都将被忽略，并且不会使这个任务成为过时状态。这是不相关的任务可以在不互相干扰的情况下共用一个输出目录。如果你因为一些理由而不想这样，请考虑使用TaskOutputs.upToDateWhen()

## **15.10. 任务规则**
有时你想要有这样一项任务，它的行为依赖于参数数值范围的一个大数或是无限的数字。任务规则是提供此类任务的一个很好的表达方式：

示例 15.25. 任务规则

build.gradle
```
tasks.addRule("Pattern: ping<ID>") { String taskName ->
    if (taskName.startsWith("ping")) {
        task(taskName) << {
            println "Pinging: " + (taskName - 'ping')
        }
    }
}
```
Gradle q pingServer1的输出结果
```
> gradle -q pingServer1
Pinging: Server1
```
这个字符串参数被用作这条规则的描述。当对这个例子运行 gradle tasks 的时候，这个描述会被显示。

规则不只是从命令行调用任务才起作用。你也可以对基于规则的任务创建依赖关系：

示例 15.26. 基于规则的任务依赖

build.gradle
```
tasks.addRule("Pattern: ping<ID>") { String taskName ->
    if (taskName.startsWith("ping")) {
        task(taskName) << {
            println "Pinging: " + (taskName - 'ping')
        }
    }
}

task groupPing {
    dependsOn pingServer1, pingServer2
}
```
Gradle q groupPing的输出结果
```
> gradle -q groupPing
Pinging: Server1
Pinging: Server2
```
## **15.11. 析构器任务**
析构器任务是一个 孵化中 的功能 (请参阅  C.1.2 章节， “Incubating”)。
当最终的任务准备运行时，析构器任务会自动地添加到任务图中。

示例 15.27. 添加一个析构器任务

build.gradle
```
task taskX << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}

taskX.finalizedBy taskY
```
gradle -q taskX的输出结果
```
> gradle -q taskX
taskX
taskY
```
即使最终的任务执行失败，析构器任务也会被执行。

示例 15.28. 执行失败的任务的任务析构器

build.gradle
```
task taskX << {
    println 'taskX'
    throw new RuntimeException()
}
task taskY << {
    println 'taskY'
}

taskX.finalizedBy taskY
```
gradle -q taskX的输出结果
```
> gradle -q taskX
taskX
taskY
```
另一方面，如果最终的任务什么都不做的话，比如由于失败的任务依赖项或如果它被认为是最新的状态，析构任务不会执行。

在不管构建成功或是失败，都必须清理创建的资源的情况下，析构认为是很有用的。这样的资源的一个例子是，一个 web 容器会在集成测试任务前开始，并且在之后关闭，即使有些测试失败。

你可以使用Task.finalizedBy()方法指定一个析构器任务。这个方法接受一个任务实例、 任务名称或<a4><c5>Task.dependsOn()</c5></a4>所接受的任何其他输入作为参数。

## **15.12. 总结**
如果你是从 Ant 转过来的，像Copy这种增强的 Gradle 任务，看起来就像是一个 Ant 目标（target）和一个 Ant 任务（task）之间的混合物。实际上确实是这样子。Gradle 没有像 Ant 那样对任务和目标进行分离。简单的 Gradle 任务就像 Ant 的目标，而增强的 Gradle 任务还包括 Ant 任务方面的内容。Gradle 的所有任务共享一个公共 API，您可以创建它们之间的依赖性。这样的一个任务可能会比一个 Ant 任务更好配置。它充分利用了类型系统，更具有表现力而且易于维护。


[7]你可能会想，为什么既不导入StopExecutionException也没有通过其完全限定名来访问它。原因是，Gradle 会向您的脚本添加默认的一些导入。这些导入是可自定义的 （见附录 E，现有的 IDE 支持和没有支持时如何应对）。


