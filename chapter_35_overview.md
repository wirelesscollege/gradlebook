# **第35章 Sonar插件**

Chapter 35. The Sonar Plugin

你可能希望使用新的Sonar Runner插件来代替该插件. 特别是Sonar Runner插件仅支持Sonar3.4及以上版本。

You may wish to use the new Sonar Runner Plugin instead of this plugin. In particular， only the Sonar Runner plugin supports Sonar 3.4 and higher.

Sonar插件提供了监控代码质量的web站，该插件增加了一个sonarAnalyze任务，用于分析项目及其子项目的插件应用.并将结果保存在Sonar数据库中.Sonar插件基于Sonar Runner且Sonar的版本需要在2.11以上。

The Sonar plugin provides integration with Sonar， a web-based platform for monitoring code quality. The plugin adds a sonarAnalyze task that analyzes the project to which the plugin is applied， as well as its subprojects. The results are stored in the Sonar database. The plugin is based on the Sonar Runner and requires Sonar 2.11 or higher.

sonarAnalyze任务是一个孤立的任务，执行它不需要依赖其他的task.除了源码，该任务同样可以分析class文件和测试结果文件(如果可用)，为达到最佳效果，推荐在analysis前运行一个完整的构建，一个典型的配置为在构建服务上一天执行一次分析。

The sonarAnalyze task is a standalone task that needs to be executed explicitly and doesn't depend on any other tasks. Apart from source code， the task also analyzes class files and test result files (if available). For best results， it is therefore recommended to run a full build before the analysis. In a typical setup， analysis would be performed once per day on a build server.

## **35.1. 用法**

35.1. Usage

当使用Sonar插件的时候，如下代码需要配置到build.gradle项目内。

At a minimum， the Sonar plugin has to be applied to the project.

示例 35.1. 使用Sonar插件

Example 35.1. Applying the Sonar plugin

build.gradle

```
apply plugin: "sonar"
```

除非Sonar使用默认配置在本地运行，否则都需要配置Sonar服务器和数据库的连接.

Unless Sonar is run locally and with default settings， it is necessary to configure connection settings for the Sonar server and database.

示例 35.2. 配置Sonar连接设置

Example 35.2. Configuring Sonar connection settings

build.gradle

```
sonar {
    server {
        url = "http://my.server.com"
    }
    database {
        url = "jdbc:mysql://my.server.com/sonar"
        driverClassName = "com.mysql.jdbc.Driver"
        username = "Fred Flintstone"
        password = "very clever"
    }
}
```

另外，可以通过命令行来设置部分或所有连接(参见35.6节:"命令行配置Sonar设置").

Alternatively， some or all connection settings can be set from the command line (see Section 35.6， “Configuring Sonar Settings from the Command Line”).

项目设置决定这个项目将会如何进行分析，默认配置适用于分析标准的Java项目，并且你可以在许多方面进行自定义。

Project settings determine how the project is going to be analyzed. The default configuration works well for analyzing standard Java projects and can be customized in many ways.

示例 35.3. 配置Sonar项目设置

Example 35.3. Configuring Sonar project settings

build.gradle

```
sonar {
    project {
        coberturaReportPath = file("$buildDir/cobertura.xml")
    }
}
```

sonar在SonarRootModel类型的对象上配置服务器、数据库和项目块上面的示例，分别对应SonarServer， SonarDatabase， and SonarProject.可参见它们的API文档了解更多信息。

The sonar， server， database， and project blocks in the examples above configure objects of type SonarRootModel， SonarServer， SonarDatabase， and SonarProject， respectively. See their API documentation for further information.

## **35.2. 分析多项目构建**

35.2. Analyzing Multi-Project Builds

Sonar插件能够分析整个项目的层次结构.且能在Sonar web界面的综合指标里生成了项目及其子项目的层次视图。这也比每个项目单独分析要快。

The Sonar plugin is capable of analyzing a whole project hierarchy at once. This yields a hierarchical view in the Sonar web interface with aggregated metrics and the ability to drill down into subprojects. It is also faster than analyzing each project separately.

分析项目的层次结构时，Sonar插件需要被应用到该层次结构的最上层的项目。典型的(但不一定)这将是项目的根目录.Sonar在项目配置一个SonarRootModel类型的对象.它拥有所有全局配置，最重要的是服务器和数据库连接设置。

To analyze a project hierarchy， the Sonar plugin needs to be applied to the top-most project of the hierarchy. Typically (but not necessarily) this will be the root project. The sonar block in that project configures an object of type SonarRootModel. It holds all global configuration， most importantly server and database connection settings.

示例 35.4. 多项目构建的全局配置

Example 35.4. Global configuration in a multi-project build

build.gradle

```
apply plugin: "sonar"

sonar {
    server {
        url = "http://my.server.com"
    }
    database {
        url = "jdbc:mysql://my.server.com/sonar"
        driverClassName = "com.mysql.jdbc.Driver"
        username = "Fred Flintstone"
        password = "very clever"
    }
}
```

在层次结构中每个项目都有自己的项目配置，公共的部分可以设置在父级构建脚本中。

Each project in the hierarchy has its own project configuration. Common values can be set from a parent build script.

示例 35.5. 在多项目构建中配置共同的项目

Example 35.5. Common project configuration in a multi-project build
build.gradle
```
subprojects {
    sonar {
        project {
            sourceEncoding = "UTF-8"
        }
    }
}
```

在子项目中的sonar块,配置一个SonarProjectModel类型对象.
The sonar block in a subproject configures an object of type SonarProjectModel.

项目也可以单独配置，例如，skip属性设置为true将防止项目(及其子项目)被分析，设置skip的项目将不会显示在Sonar web界面中。

Projects can also be configured individually. For example， setting the skip property to true prevents a project (and its subprojects) from being analyzed. Skipped projects will not be displayed in the Sonar web interface.

示例 35.6. 在多项目构建中配置单个项目 

Example 35.6. Individual project configuration in a multi-project build

build.gradle

```
project(":project1") {
    sonar {
        project {
            skip = true
        }
    }
}
```

另一种典型通用项目配置是要对编程语言进行分析的。需要注意的是Sonar只能分析每个项目的一种语言。

Another typical per-project configuration is the programming language to be analyzed. Note that Sonar can only analyze one language per project.

示例 35.7. 配置被分析的语言

Example 35.7. Configuring the language to be analyzed

build.gradle
```
project(":project2") {
    sonar {
        project {
            language = "groovy"
        }
    }
}
```

当仅设置一个属性时，这个属性设置语法会更加简洁。

When setting only a single property at a time， the equivalent property syntax is more succinct:

示例 35.8. 使用属性语法

Example 35.8. Using property syntax

build.gradle

```
project(":project2").sonar.project.language = "groovy"
```

## **35.3 分析自定义源集合**

35.3. Analyzing Custom Source Sets

默认情况下，Sonar插件将分析main和test这两源集合，这不依赖于项目源目录布局.可根据需要添加附加源集合.

By default， the Sonar plugin will analyze the production sources in the main source set and the test sources in the test source set. This works independent of the project's source directory layout. Additional source sets can be added as needed.

示例 35.9 分析自定义源集合   
Example 35.9. Analyzing custom source sets

build.gradle
```
sonar.project {
    sourceDirs += sourceSets.custom.allSource.srcDirs
    testDirs += sourceSets.integTest.allSource.srcDirs
}
```

## **35.4. 分析非Java语言**

35.4. Analyzing languages other than Java

安装相应的Sonar插件并设置相应的sonar.project.language，分析非Java语言编写的代码：

To analyze code written in a language other than Java， install the corresponding Sonar plugin， and set sonar.project.language accordingly:

示例 35.10. 分析非Java语言

Example 35.10. Analyzing languages other than Java

build.gradle

```
sonar.project {
    language = "grvy" // set language to Groovy
}
```

从Sonar3.4开始，每个项目只能使用一种语言进行分析，但是，你可以在多项目构建中为每个项目设置不同的语言。

As of Sonar 3.4， only one language per project can be analyzed. You can， however， set a different language for each project in a multi-project build.

## **35.5. 设置自定义Sonar属性**

35.5. Setting Custom Sonar Properties

最终，大部分配置以键-值对的形式被传递到Sonar代码分析器作为Sonar属性，API文档内SonarProperty 注解展示了插件对象模型与sonar属性之间的映射关系。Sonar插件可以hook一些属性，防止它们被code analyzer，同样的hooks可以被使用到那些没有被插件对象模型转化过的属性上。

Eventually， most configuration is passed to the Sonar code analyzer in the form of key-value pairs known as Sonar properties. The SonarProperty annotations in the API documentation show how properties of the plugin's object model get mapped to the corresponding Sonar properties. The Sonar plugin offers hooks to post-process Sonar properties before they get passed to the code analyzer. The same hooks can be used to add additional properties which aren't covered by the plugin's object model

对于Sonar全局属性，我们使用SonarRootModel上的withGlobalProperties勾子方法进行配置。

For global Sonar properties， use the withGlobalProperties hook on SonarRootModel:

示例 34.11. 设置自定义全局属性

Example 34.11. Setting custom global properties

build.gradle

```
sonar.withGlobalProperties { props ->
    props["some.global.property"] = "some value"
    // non-String values are automatically converted to Strings
    props["other.global.property"] = ["foo"， "bar"， "baz"]
}
```

对于预置项目的Sonar属性，我们可以在sonar项目中使用withProjectProperties hook：
For per-project Sonar properties， use the withProjectProperties hook on SonarProject:

示例 34.12. 设置自定义项目属性

Example 34.12. Setting custom project properties

build.gradle

```
sonar.project.withProjectProperties { props ->
    props["some.project.property"] = "some value"
    // non-String values are automatically converted to Strings
    props["other.project.property"] = ["foo"， "bar"， "baz"]
}
```

可用的Sonar属性列表可以在Sonar文档中找到，注意这些属性，Sonar插件对象模型具有等同的属性，且没有必要使用withGlobalProperties或withProjectProperties勾子方法.对于配置第三方Sonar插件，请查阅插件相关文档。

A list of available Sonar properties can be found in the Sonar documentation. Note that for most of these properties， the Sonar plugin's object model has an equivalent property， and it isn't necessary to use a withGlobalProperties or withProjectProperties hook. For configuring a third-party Sonar plugin， consult the plugin's documentation.

## 34.6. 通过命令行配置Sonar设置

34.6. Configuring Sonar Settings from the Command Line

以下属性通过命令行来设置也是一种选择，可把这些属性作为sonarAnalyze任务的参数，该任务参数将覆盖任何在构建脚本中所设置的对应值。

The following properties can alternatively be set from the command line， as task parameters of the sonarAnalyze task. A task parameter will override any corresponding value set in the build script.

•	server.url
•	database.url
•	database.driverClassName
•	database.username
•	database.password
•	showSql
•	showSqlResults
•	verbose
•	forceAnalysis


下面是一个完整的例子：

Here is a complete example:

-----
"
**gradle sonarAnalyze --server.url=http://sonar.mycompany.com --database.password=myPassword –verbose** "


假如你需要通过命令行设置其他属性，你可以使用系统属性这样做：

If you need to set other properties from the command line， you can use system properties to do so:
示例 34.13.实现自定义命令行属性

Example 34.13. Implementing custom command line properties

build.gradle

```
sonar.project {
    language = System.getProperty("sonar.language"， "java")
}
```

但是，请记住它通常最好在构建脚本和源码控制之下保持配置。

However， keep in mind that it is usually best to keep configuration in the build script and under source control.

34.7. 任务

34.7. Tasks

Sonar插件添加了以下任务到项目中.

The Sonar plugin adds the following tasks to the project.

表 34.1. Sonar插件 – 任务

Table 34.1. Sonar plugin – tasks

|Task name|	Depends on|	Type|	Description|
|--
|sonarAnalyze|	-	|SonarAnalyze|Analyzes a project hierarchy and stores the results in the Sonar database.|

分析项目层次结构和保存结果到Sonar数据库

百度搜索[无线学院](http://wirelesscollege.cn)
