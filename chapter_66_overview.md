# **第66章.构建比较**

Chapter 66. Comparing Builds

支持构建比较是一个待优化的功能.这意味着它是不完整且并没有在Gradle产品质量中形成规则.这也意味着该Gradle用户指南章节是一个正在进行中的工作.

Build comparison support is an incubating feature. This means that it is incomplete and not yet at regular Gradle production quality. This also means that this Gradle User Guide chapter is a work in progress.

Gradle提供支持比较两次构建的结果(如.生成的二进制文件).可能有很多你想要比较两次构建结果的原因.你可能想比较:

Gradle provides support for comparing the outcomes (e.g. the produced binary archives) of two builds. There are several reasons why you may want to compare the outcomes of two builds. You may want to compare:

使用新Gradle版本的构建与使用当前版本的构建比较(即升级Gradle版本)

A build with a newer version of Gradle than it's currently using (i.e. upgrading the Gradle version).

Gradle构建与其他工具如ant,Maven或其他工具执行构建的比较(即迁移到Gradle).

A Gradle build with a build executed by another tool such as Apache Ant, Apache Maven or something else (i.e. migrating to Gradle).

同样的Gradle构建在相同版本前后变化的构建(即测试构建改变)

The same Gradle build, with the same version, before and after a change to the build (i.e. testing build changes).

在这些情况下通过比较构建你能够做出比较明智的有关Gradle升级,迁移到Gradle或变更构建的决定.该比较过程中会生成一项HTML报告，它列出了相同部分及指出不同的地方。

By comparing builds in these scenarios you can make an informed decision about the Gradle upgrade, migration to Gradle or build change by understanding the differences in the outcomes. The comparison process produces a HTML report outlining which outcomes were found to be identical and identifying the differences between non-identical outcomes.

## **66.1.术语定义**

66.1. Definition of terms

以下是用于构建比较及其定义的术语.

The following are the terms used for build comparison and their definitions.

“Build”
“构建”

在构建比较的环境里,一项构建未必是Gradle构建。它可以是任何生成可见“结果”的调用“过程”。不过比较中至少有一个构建会是Gradle构建。
In the context of build comparison, a build is not necessarily a Gradle build. It can be any invokable “process” that produces observable “outcomes”. At least one of the builds in a comparison will be a Gradle build.

“Build Outcome”
“构建结果”

构建过程中有些事情会以可见的方式发生,如创建一个zip文件或测试执行。这些都是比较的东西。
Something that happens in an observable manner during a build, such as the creation of a zip file or test execution. These are the things that are compared.

“Source Build”
“源码构建”
构建针对做比较的对象,通常在“当前”状态的构建。换句话说,比较的左手边。
The build that comparisons are being made against, typically the build in its “current” state. In other words, the left hand side of the comparison.

“Target Build”
“目标构建”
与源构建相比较的构建,通常“提议”的构建。换句话说,比较的右手边。
The build that is being compared to the source build, typically the “proposed” build. In other words, the right hand side of the comparison.

“Host Build”
“主机构建”
执行比较进程的Gradle构建。它可能是相同项目中的目标构建或是源码构建或者可能是个完全不同的项目。它没必要和源码构建或目标构建有相同的Gradle版本。主机构建必须运行在Gradle1.2或更高版本。
The Gradle build that executes the comparison process. It may be the same project as either the “target” or “source” build or may be a completely separate project. It does not need to be the same Gradle version as the “source” or “target” builds. The host build must be run with Gradle 1.2 or newer.


“Compared Build Outcome”
“比较构建结果”

构建结果,目的是逻辑上等效的“源”和“目标”的构建,因此是意义上的可比性。
Build outcomes that are intended to be logically equivalent in the “source” and “target” builds, and are therefore meaningfully comparable.

“Uncompared Build Outcome”
“非比较构建结果”
A build outcome is uncompared if a logical equivalent from the other build cannot be found (e.g. a build produces a zip file that the other build does not).
在另外的比较构建里没有找到逻辑上等同物的构建结果。（例如一个构建生成一个zip文件而另外的构建没有）
“Unknown Build Outcome”
“未知构建结果”

无法被主机理解的构建结果。这可以发生在源或目标构建的Gradle版本比主机高，而新的Gradle版本公开了新的结果类型。未知的构建结果可以比较一旦他们被识别逻辑等同于一个来自另外构建的未知的构建结果,且没有实际比较意义的比较可以被执行。主机构建使用最新的Gradle版本将避免遇到未知的构建结果。
A build outcome that cannot be understood by the host build. This can occur when the source or target build is a newer Gradle version than the host build and that Gradle version exposes new outcome types. Unknown build outcomes can be compared in so far as they can be identified to be logically equivalent to an unknown build outcome in the other build, but no meaningful comparison of what the build outcome actually is can be performed. Using the latest Gradle version for the host build will avoid encountering unknown build outcomes.

## **66.2. 当前功能**

66.2. Current Capabilities

由于这是个正在逐渐完善的功能,目前得到实现的功能是有限的.

As this is an incubating feature, a limited set of the eventual functionality has been implemented at this time.

### **66.2.1. 支持构建**

66.2.1. Supported builds

目前仅支持Gradle构建的比较.源码和目标构建必须在Gradle更新或等于1.0的版本上执行.

Only support for comparing Gradle builds is available at this time. Both the source and target build must execute with Gradle newer or equal to version 1.0. The host build must be at least version 1.2.

将来的版本将提供支持在其他构建系统执行构建,如Apache Ant的或Apache Maven,以及支持用于执行任意程序（如shell脚本基础版本）

Future versions will provide support for executing builds from other build systems such as Apache Ant or Apache Maven, as well as support for executing arbitrary processes (e.g. shell script based builds)

### **66.2.2. 支持构建输出**

66.2.2. Supported build outcomes

仅支持zip文件比较构建结果,包括jar,war和ear文件.

Only support for comparing build outcomes that are zip archives is supported at this time. This includes jar, war and ear archives.

将来的版本将提供支持比较结果如测试执行的结果(如.哪个测试被执行,哪个测试失败等)

Future versions will provide support for comparing outcomes such as test execution (i.e. which tests were executed, which tests failed, etc.)

## **66.3. 比较Gradle构建**

66.3. Comparing Gradle Builds

compare-gradle-builds插件能够用来促进比较两个Gradle构建.该插件增加了一个名为“compareGradleBuilds”的CompareGradleBuilds任务到项目中.该任务配置指定的比较.默认情况下,它被配置为当前的构建与使用当前Gradle版本通过执行任务:“clean assemble”.

The compare-gradle-builds plugin can be used to facilitate a comparison between two Gradle builds. The plugin adds a CompareGradleBuilds task named “compareGradleBuilds” to the project. The configuration of this task specifies what is to be compared. By default, it is configured to compare the current build with itself using the current Gradle version by executing the tasks: “clean assemble”.

apply plugin: 'compare-gradle-builds'

该任务可以改变配置进行对比.

This task can be configured to change what is compared.
```
compareGradleBuilds {
    sourceBuild {
        projectDir "/projects/project-a"
        gradleVersion "1.1"
    }
    targetBuild {
        projectDir "/projects/project-b"
        gradleVersion "1.2"
    }
}
```

上面的例子指定使用两个不同版本的Gradle进行两个不同的项目的比较.      

The example above specifies a comparison between two different projects using two different Gradle versions.

### **66.3.1. 尝试Gradle升级**

66.3.1. Trying Gradle upgrades

你可以使用构建比较功能快速的尝试新Gradle版本来支持你的构建.

You can use the build comparison functionality to very quickly try a new Gradle version with your build.

尝试试用不同Gradle版本来进行你当前的构建,简单添加以下内容到项目根目录下的build.gradle下

To try your current build with a different Gradle version, simply add the following to the build.gradle of the root project.

```
apply plugin: 'compare-gradle-builds'

compareGradleBuilds {
    targetBuild.gradleVersion = "«gradle version»"
}
```
            
然后只需执行compareGradleBuilds任务.你将看到正在执行的“source” 和 “target” 构建在控制台输出.

Then simply execute the compareGradleBuilds task. You will see the console output of the “source” and “target” builds as they are executing.

### **66.3.2. 比较"结果"**

66.3.2. The comparison “result”

如果在比较结果输出中有任何差异,该任务将失败.本地的HTML报告将提供洞察比较.如果所有比较结果发现都是相同的,且没有未比较的结果,也没有未知的构建结果,该任务将成功.

If there are any differences between the compared outcomes, the task will fail. The location of the HTML report providing insight into the comparison will be given. If all compared outcomes are found to be identical, and there are no uncompared outcomes, and there are no unknown build outcomes, the task will succeed.

您可以通过配置任务设置ignoreFailures属性为true来忽略差异比较失败的结果.

You can configure the task to not fail on compared outcome differences by setting the ignoreFailures property to true.

```
compareGradleBuilds {
    ignoreFailures = true
}
```    

### **66.3.3. 比较哪些产物?**

66.3.3. Which archives are compared?

对于要进行比较的产物,必须将其添加作为archives配置的一个产物,看看第53章Publishing artifacts关于如何配置和增加artifacts的详细信息.

For an archive to be a candidate for comparison, it must be added as an artifact of the archives configuration. Take a look at Chapter 53, Publishing artifacts for more information on how to configure and add artifacts.

该产物必须必须由zip,Jar,War,Ear任务生成.Gradle将来的版本将在这部分更加灵活.

The archive must also have been produced by a Zip, Jar, War, Ear task. Future versions of Gradle will support increased flexibility in this area.
