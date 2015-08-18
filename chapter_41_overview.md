# **第41章 项目报告插件**

Chapter 41. The Project Report Plugin

项目报告插件将一些任务添加到你的项目中，去生成一个对于构建有用的信息报告。这些任务通过命令行执行来生成依赖关系 和属性任务相同的内容  （见章节11.6，“获取构建信息”）。相比命令行的报告，此报告插件将报告生成在一个文件中。  另外，聚合任务依赖于所有报告任务插件的添加。

The Project report plugin adds some tasks to your project which generate reports containing useful information about your build. These tasks generate the same content that you get by executing the tasks, dependencies, and properties tasks from the command line (see Section 11.6, “Obtaining information about your build”). In contrast to the command line reports, the report plugin generates the reports into a file. There is also an aggregating task that depends on all report tasks added by the plugin.

我们计划在未来的Gradle版本中添加更多已存在报告并创建一些其他的报告。

We plan to add much more to the existing reports and create additional ones in future releases of Gradle.

## **41.1 使用**

41.1. Usage

要使用项目报告插件，需包含以下构建脚本：

To use the Project report plugin, include the following in your build script:

```
apply plugin: 'project-report'
```

## **41.2 任务**

41.2. Tasks

项目报告插件定义了以下任务：

The project report plugin defines the following tasks:

表41.1  项目报告插件--任务

Table 41.1. Project report plugin – tasks

|任务名称	|依赖于|类型	|描述|
|--
|dependencyReport|-	|DependencyReportTask|生成项目依赖报告|
|htmlDependencyReport	|-|	HtmlDependencyReportTask|生成一个或一组HTML项目依赖报告|
|propertyReport	|-	|PropertyReportTask|生成项目属性报告|
|taskReport	|-	|TaskReportTask|生成项目任务报告|
|projectReport	|dependencyReport, propertyReport, taskReport,htmlDependencyReport	|Task|生产所有的项目报告|


|Task name|	Depends on	|Type|	Description|
|--
|dependencyReport|	-	|DependencyReportTask|Generates the project dependency report.|
|htmlDependencyReport|	-	|HtmlDependencyReportTask|Generates an HTML dependency and dependency insight report for the project or a set of projects.|
|propertyReport	|-	|PropertyReportTask|Generates the project property report.|
|taskReport|	-	|TaskReportTask|Generates the project task report.|
|projectReport	|dependencyReport, propertyReport, taskReport,htmlDependencyReport	|Task|Generates all project reports.|



## **41.3 项目布局**

41.3. Project layout

项目报告插件不需要任何特定的项目布局。

The project report plugin does not require any particular project layout.

## **41.4  依赖管理**

41.4. Dependency management

项目报告插件没有定义任何依赖配置。

The project report plugin does not define any dependency configurations.

## **41.5 常规属性**

41.5. Convention properties

项目报告定义了以下常规属性：

The project report defines the following convention properties:

表41.2 项目报告插件—常规属性

Table 41.2. Project report plugin - convention properties

|属性名称	|类型	|默认值	|描述|
|--
|reportsDirName	|String	|reports	|生成报告的目录名称|
|reportsDir|	File (read-only)	|buildDir/reportsDirName	|生成报告的目录|
|projects	|Set<Project>	|A one element set with the project the plugin was applied to.|	生成报告的项目|
|projectReportDirName|	String	|project|	生成项目报告的目录名称|
|projectReportDir|	File (read-only)	|reportsDir/projectReportDirName|	生成项目报告的目录|



|Property name|	Type	|Default value|	Description|
|--
|reportsDirName|	String	|reports	|The name of the directory to generate reports into, relative to the build directory.|
|reportsDir|	File (read-only)	|buildDir/reportsDirName|	The directory to generate reports into.|
|projects	|Set<Project>|	A one element set with the project the plugin was applied to.|	The projects to generate the reports for.|
|projectReportDirName	|String	|project	|The name of the directory to generate the project report into, relative to the reports directory.|
|projectReportDir	|File (read-only)|reportsDir/projectReportDirName|	The directory to generate the project report into.|



这些常规属性提供了一个常规ProjectReportsPluginConvention类型的对象。

These convention properties are provided by a convention object of type ProjectReportsPluginConvention.

百度搜索[无线学院](http://wirelesscollege.cn)
