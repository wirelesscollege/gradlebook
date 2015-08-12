# **第63章.构建比较**

Chapter 63. Comparing Builds

支持构建比较是一个潜在的特性.这意味着它是不完整且并没有在Gradle产品质量中形成规则.这也意味着该Gradle用户指南章节是一个正在进行的工作.

Build comparison support is an incubating feature. This means that it is incomplete and not yet at regular Gradle production quality. This also means that this Gradle User Guide chapter is a work in progress.

Gradle提供支持比较两次构建的结果(如.生成的二进制文件).你可能要比较两次构建的结果有几个原因.你可能想比较:

Gradle provides support for comparing the outcomes (e.g. the produced binary archives) of two builds. There are several reasons why you may want to compare the outcomes of two builds. You may want to compare:

使用新Gradle版本的构建与使用当前版本的构建比较(即升级Gradle版本)

A build with a newer version of Gradle than it's currently using (i.e. upgrading the Gradle version).

Gradle构建与其他工具如ant,Maven或其他工具执行构建的比较(即迁移到Gradle).

A Gradle build with a build executed by another tool such as Apache Ant, Apache Maven or something else (i.e. migrating to Gradle).

同样的Gradle构建具有在相同版本前后差异的构建(即测试构建更改)

The same Gradle build, with the same version, before and after a change to the build (i.e. testing build changes).

在这些情况下通过比较构建你能够做出比较明智的有关Gradle升级,迁移到Gradle或差异构建的决定.该比较过程中产生列出发现的差异结果和识别不同的结果的HTML报告。

By comparing builds in these scenarios you can make an informed decision about the Gradle upgrade, migration to Gradle or build change by understanding the differences in the outcomes. The comparison process produces a HTML report outlining which outcomes were found to be identical and identifying the differences between non-identical outcomes.

## **63.1.术语定义**

63.1. Definition of terms

以下是用于构建比较及其定义的术语.

The following are the terms used for build comparison and their definitions.

“Build”

In the context of build comparison, a build is not necessarily a Gradle build. It can be any invokable “process” that produces observable “outcomes”. At least one of the builds in a comparison will be a Gradle build.

“Build Outcome”

Something that happens in an observable manner during a build, such as the creation of a zip file or test execution. These are the things that are compared.

“Source Build”

The build that comparisons are being made against, typically the build in its “current” state. In other words, the left hand side of the comparison.

“Target Build”

The build that is being compared to the source build, typically the “proposed” build. In other words, the right hand side of the comparison.

“Host Build”

The Gradle build that executes the comparison process. It may be the same project as either the “target” or “source” build or may be a completely separate project. It does not need to be the same Gradle version as the “source” or “target” builds. The host build must be run with Gradle 1.2 or newer.

“Compared Build Outcome”
Build outcomes that are intended to be logically equivalent in the “source” and “target” builds, and are therefore meaningfully comparable.

“Uncompared Build Outcome”
A build outcome is uncompared if a logical equivalent from the other build cannot be found (e.g. a build produces a zip file that the other build does not).

“Unknown Build Outcome”

A build outcome that cannot be understood by the host build. This can occur when the source or target build is a newer Gradle version than the host build and that Gradle version exposes new outcome types. Unknown build outcomes can be compared in so far as they can be identified to be logically equivalent to an unknown build outcome in the other build, but no meaningful comparison of what the build outcome actually is can be performed. Using the latest Gradle version for the host build will avoid encountering unknown build outcomes.

## **63.2. 当前功能**

63.2. Current Capabilities

由于这是个正在逐渐完善的功能,目前得到实现的功能是有限的.

As this is an incubating feature, a limited set of the eventual functionality has been implemented at this time.

### **63.2.1. 支持构建**

63.2.1. Supported builds

目前仅支持Gradle构建的比较.源码和目标构建必须在Gradle更新或等于1.0的版本上执行.

Only support for comparing Gradle builds is available at this time. Both the source and target build must execute with Gradle newer or equal to version 1.0. The host build must be at least version 1.2.

将来的版本将提供支持从其他构建系统执行的构建,如Apache Ant的或Apache Maven的,以及支持用于执行任意程序（如shell脚本基础版本）

Future versions will provide support for executing builds from other build systems such as Apache Ant or Apache Maven, as well as support for executing arbitrary processes (e.g. shell script based builds)

### **63.2.2. 支持构建输出**

63.2.2. Supported build outcomes

仅支持zip文件比较构建结果,包括jar,war和ear文件.

Only support for comparing build outcomes that are zip archives is supported at this time. This includes jar, war and ear archives.

将来的版本将提供支持比较结果如测试执行的结果(如.哪个测试被执行,哪个测试失败等)

Future versions will provide support for comparing outcomes such as test execution (i.e. which tests were executed, which tests failed, etc.)

## **63.3. 比较Gradle构建**

63.3. Comparing Gradle Builds

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

### **63.3.1. 尝试Gradle升级**

63.3.1. Trying Gradle upgrades

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

### **63.3.2. 比较"结果"**

63.3.2. The comparison “result”

如果有任何差异比较结果输出,该任务将失败.本地的HTML报告将提供洞察比较.如果所有比较结果发现都是相同的,且没有未比较的结果,也没有未知的构建结果,该任务将成功.

If there are any differences between the compared outcomes, the task will fail. The location of the HTML report providing insight into the comparison will be given. If all compared outcomes are found to be identical, and there are no uncompared outcomes, and there are no unknown build outcomes, the task will succeed.

您可以通过配置任务设置ignoreFailures属性为true来忽略差异比较失败的结果.

You can configure the task to not fail on compared outcome differences by setting the ignoreFailures property to true.

```
compareGradleBuilds {
    ignoreFailures = true
}
```    

### **63.3.3. 比较哪个文档?**

63.3.3. Which archives are compared?

关于选择哪个文档进行比较,必须将其添加到archives配置的artifact中,看看第51章Publishing artifacts关于如何配置和增加artifacts的详细信息.

For an archive to be a candidate for comparison, it must be added as an artifact of the archives configuration. Take a look at Chapter 51, Publishing artifacts for more information on how to configure and add artifacts.

该文档必须必须通过一个zip,Jar,War,Ear任务生成.Gradle将来的版本将在这块区域更加灵活.

The archive must also have been produced by a Zip, Jar, War, Ear task. Future versions of Gradle will support increased flexibility in this area.

百度搜索[无线学院](http://wirelesscollege.cn)