# **第三十三章. PMD 插件**

Chapter 33. The PMD Plugin

PMD 插件使用PMD做java源代文件质量检查，并生成报告。

The PMD plugin performs quality checks on your project's Java source files using PMD and generates reports from these checks. 

## **33.1. 用法**

33.1. Usage

在build脚本中使用PMD 插件，如下：

To use the PMD plugin, include the following in your build script:

例：33.1使用PMD 插件

Example 33.1. Using the PMD plugin

build.gradle
```
apply plugin: 'pmd'
```

插件添加一些任务对项目进行质量检查，可以通过运行gradle check工具检查。

The plugin adds a number of tasks to the project that perform the quality checks. You can execute the checks by running gradle check.

## **33.2. Tasks**

33.2. Tasks

PMD 插件为这个项目添加了如下任务：

The PMD plugin adds the following tasks to the project:

Table 33.1. PMD plugin - tasks

|Task name|	Depends on|	Type|	Description|
|--
|pmdMain |	-	|Pmd|Runs PMD against the production Java source files.|
|pmdTest| 	-	|Pmd|Runs PMD against the test Java source files.|
|pmdSourceSet |	-	|Pmd|Runs PMD against the given source set's Java source files.|

PMD plugin的依赖增加了如下通过Java plugin定义的任务

The PMD plugin adds the following dependencies to tasks defined by the Java plugin.

表 33.2. PMD plugin -额外的任务依赖关系

Table 33.2. PMD plugin - additional task dependencies


|Task name	|Depends on|
|--
|check	|All PMD tasks, including pmdMain and pmdTest.|


## **33.3. 依赖管理**

33.3. Dependency management

PMD插件添加了如下依赖配置：

The PMD plugin adds the following dependency configurations:

表33.3. PMD插件－依赖配置

Table 33.3. PMD plugin - dependency configurations

|Name	|Meaning|
|--
|pmd 	|The PMD libraries to use|

## **33.4. 配置**

33.4. Configuration

见API文档中PmdExtension类

See the PmdExtension class in the API documentation.

百度搜索[无线学院](http://wirelesscollege.cn)

