# **Chapter 32. The JDepend Plugin**

JDepend插件可以检查你项目的源码文件，使用该插件可以生成JDepend的检查报告。

The JDepend plugin performs quality checks on your project's source files using JDepend and generates reports from these checks.

## **32.1. Usage**

在你的构建脚本中使用JDepend插件需要包含以下的配置:

To use the JDepend plugin, include the following in your build script:

**Example 32.1. Using the JDepend plugin**

build.gradle
```
apply plugin: 'jdepend'
```

该插件在项目上增加了一系列的执行质量检查任务，你可以通过运行Gradle check 来执行JDepend的这些检查。

The plugin adds a number of tasks to the project that perform the quality checks. You can execute the checks by running gradle check.

## **32.2. Tasks**

JDepend插件项目增加了以下任务：

The JDepend plugin adds the following tasks to the project:

**Table 32.1. JDepend plugin - tasks**

|Task name|	Depends on	|Type|	Description|
|--
|jdependMain|	classes	|JDepend|	Runs JDepend against |the production Java source files.|
|jdependTest	|testClasses	|JDepend	|Runs JDepend against the test Java source files.|
|jdependSourceSet	|sourceSetClasses|	JDepend	Runs JDepend against the given source set's Java source files.|

JDepend插件需要通过Java插件添加以下的依赖任务
The JDepend plugin adds the following dependencies to tasks defined by the Java plugin.

**Table 32.2. JDepend plugin - additional task dependencies**

|Task name	|Depends on|
|--
|check	|All JDepend tasks, including jdependMain and jdependTest.|

## **32.3. Dependency management**

JDepend插件需要添加如下的依赖配置

The JDepend plugin adds the following dependency configurations:

Table 32.3. JDepend plugin - dependency configurations

|Name	|Meaning|
|--
|jdepend	|The JDepend libraries to use|

## **32.4. Configuration**

参见API文档中的JdependExtension类(https://docs.gradle.org/current/dsl/org.gradle.api.plugins.quality.JDependExtension.html)

See the ![JDependExtension](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.quality.JDependExtension.html class in the API documentation.


