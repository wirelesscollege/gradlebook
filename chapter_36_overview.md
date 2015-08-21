# **第三十六章  Sonar Runner 插件**
Chapter 36. The Sonar Runner Plugin

Sonar Runner plugin目前正在逐步完善。在以后的gradel 版本中DSL和其他的配置可能会发生变化，此插件在未来版本中替代早期的Sonar plugin 将会是一种趋势。

The Sonar Runner plugin is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.It is intended that this plugin will replace the older Sonar Plugin in a future Gradle version.

Sonar runner插件集成了sonar ，它是一个检测代码质量的web-base平台，这个平台是基于sonar runner的，它是一个sonar客户端组件用来分析源码、build输出与存储在sonar数据库中存储收集到的信息。相比使用独立的sonar runner，sonar runner plugin提供了以下好处：

The Sonar Runner plugin provides integration with Sonar, a web-based platform for monitoring code quality. It is based on theSonar Runner, a Sonar client component that analyzes source code and build outputs, and stores all collected information in the Sonar database. Compared to using the standalone Sonar Runner, the Sonar Runner plugin offers the following benefits:

自动化配置sonar Runner

通过常规的Gradle任务执行Sonar Runner的功能，使得Gradle处处可行(开发者构建、CI 服务器等），我们不在需要手动的安装、下载、设置、和维护 Sonar Runner。

Automatic provisioning of Sonar Runner
The ability to execute the Sonar Runner via a regular Gradle task makes it available anywhere Gradle is available (developer build, CI server, etc.), without the need to manually download, setup, and maintain a Sonar Runner installation.

Gradle 构建脚本的动态配置

按照我们的需求所有的Gradle's scripting功能都可以被用来配置Sonar Runner。

Dynamic configuration from Gradle build scripts
All of Gradle's scripting features can be leveraged to configure Sonar Runner as needed.

广泛的配置默认值
Extensive configuration defaults

Gradle提供了大量的有用信息帮助Sonar Runner 能成功的分析一个项目。 通过依据这些信息预先配置 Sonar Runner,明显减少了需要手动的配置。

Gradle already has much of the information needed for Sonar Runner to successfully analyze a project. By preconfiguring the Sonar Runner based on that information, the need for manual configuration is reduced significantly.

## **36.1 Sonar  Runner 版本与适配**
36.1. Sonar Runner version and compatibility

Sonar Runner 默认用的插件版本3.0适配Sonar 3.0或更高的的sonar版本.为了适配3.0以下的sonar版本，你可以使用更早之前的sonar版本（具体请查看36.4 Sonra runner 的版本说明）

The default version of the Sonar Runner used by the plugin is 2.3, which makes it compatible with Sonar 3.0 and higher. For compatibility with Sonar versions earlier than 3.0, you can configure the use of an earlier Sonar Runner version (see Section 36.4, “Specifying the Sonar Runner version”).

## **36.2开始启程**
36.2. Getting started

用一个实例项目分析一下Sonar Runner插件的使用

To get started, apply the Sonar Runner plugin to the project to be analyzed.

Example 36.1 使用sonar 插件
Example 36.1. Applying the Sonar Runner plugin

build.gradle
```
apply plugin: "sonar-runner"
```

假设你安装的Sonar server开箱能用的设置是打开并运行的，你不需要再去强制设置。

Assuming a local Sonar server with out-of-the-box settings is up and running, no further mandatory configuration is required. 

运行gradle sonarRunner 等待直到构建完成，之后打开web 页面底部Sonar Runner的指示输出文件，你就能浏览分析结果。

Execute gradle sonarRunner and wait until the build has completed, then open the web page indicated at the bottom of the Sonar Runner output. You should now be able to browse the analysis results.

在执行Sonar Runner 任务之前，所有需要被sonar 分析的产生输出任务都需要被执行，这些任务包括编译、测试、代码覆盖。为了满足这些需求，plugins从Sonar Runner 添加了一个测试任务依赖，前提是java插件已经安装。

Before executing the sonarRunner task, all tasks producing output to be analysed by Sonar need to be executed. Typically, these are compile tasks, test tasks, and code coverage tasks. To meet these needs, the plugins adds a task dependency from sonarRunner on test if the java plugin is applied. Further task dependencies can be added as needed.

## **36.3. 配置Sonar Runner**
36.3. Configuring the Sonar Runner

添加Sonar Runner插件

The Sonar Runner plugin adds 

SonarRunnerRootExtension 扩展了root project，SonarRunnerExtension 扩展了subproject，它们允许你通过键值对的形式设置Sonar Runner，这是sonar 一个众所周知的一个属性。一个典型的基准配置包括sonar server与database的连接设置。

SonarRunnerRootExtension extension to the project and a SonarRunnerExtension extension to its subprojects, which allows you to configure the Sonar Runner via key/value pairs known as Sonar properties. A typical base line configuration includes connection settings for the Sonar server and database.

Example 36.2  配置Sonar链接设置
Example 36.2. Configuring Sonar connection settings

build.gradle
```
sonarRunner {
    sonarProperties {
        property "sonar.host.url", "http://my.server.com"
        property "sonar.jdbc.url", "jdbc:mysql://my.server.com/sonar"
        property "sonar.jdbc.driverClassName", "com.mysql.jdbc.Driver"
        property "sonar.jdbc.username", "Fred Flintstone"
        property "sonar.jdbc.password", "very clever"
    }
}
```
另外，可以从命令行设置Sonar性能。查看更多信息请见第 35.6，“从命令行配置Sonar设置”

Alternatively, Sonar properties can be set from the command line. See Section 35.6, “Configuring Sonar Settings from the Command Line” for more information.

典型的Sonar属性完整列表请查阅Sonar文档。如果你碰巧使用了Sonar插件的额外功能，也请查阅他们的文档。

For a complete list of standard Sonar properties, consult the Sonar documentation. If you happen to use additional Sonar plugins, consult their documentation.

除了设置sonar的属性，SonarRunnerRootExtension允许设置Sonar Runner的版本和JavaForkOptions。

In addition to set Sonar properties,the SonarRunnerRootExtension extension allows the configuration of the Sonar Runner version and the JavaForkOptions of the forked Sonar Runner process.

在Gradle对象模式下Sonar Runner信息包含了许多典型Sonar属性。下面的表格做了一个概述。注意附加属性只为拥有java-base或者javaplugin的项目提供，另外一些属性决定了Sonar Runner相对应的默认值。

The Sonar Runner plugin leverages information contained in Gradle's object model to provide smart defaults for many of the standard Sonar properties. The defaults are summarized in the tables below. Notice that additional defaults are provided for projects that have the java-base or java plugin applied. For some properties (notably server and database connection settings), determining a suitable default is left to the Sonar Runner.

Table 36.1.  Gradle默认的标准sonar属性

Table 36.1. Gradle defaults for standard Sonar properties

|Property	|Gradle default|
|--
|sonar.projectKey	|“$project.group:$project.name” (for root project of analysed hierarchy; left to Sonar Runner otherwise)|
|sonar.projectName|	project.name|
|sonar.projectDescription	|project.description|
|sonar.projectVersion|	project.version|
|sonar.projectBaseDir|	project.projectDir|
|sonar.working.directory	|“$project.buildDir/sonar”|
|sonar.dynamicAnalysis	|“reuseReports”|

Table 36.2 使用java–base插件时的额外默认属性

Table 36.2. Additional defaults when java-base plugin is applied

|Property|	Gradle default|
|--
|sonar.java.source|	project.sourceCompatibility|
|sonar.java.target|	project.targetCompatibility|

Table 36.3. 使用java插件时额外默认属性
Table 36.3. Additional defaults when java plugin is applied

|Property	|Gradle default|
|--
|sonar.sources	|sourceSets.main.allSource.srcDirs (filtered to only include existing directories)|
|sonar.tests	|sourceSets.test.allSource.srcDirs (filtered to only include existing directories)|
|sonar.binaries	|sourceSets.main.runtimeClasspath (filtered to only include directories)|
|sonar.libraries	|sourceSets.main.runtimeClasspath (filtering to only include files; rt.jar added if necessary)|
|sonar.surefire.reportsPath	|test.testResultsDir (if the directory exists)|
|sonar.junit.reportsPath	|test.testResultsDir (if the directory exists)|

Table 36.4使用jacoco插件的额外默认信息

Table 36.4. Additional defaults when jacoco plugin is applied

|Property	|Gradle default|
|--
|sonar.jacoco.reportPath|	jacoco.destinationFile|

## **36.4. 指定Sonar Runner 版本**
36.4. Specifying the Sonar Runner version

默认情况下使用的是2.3版本，指定一个版本可以通过设置the SonarRunnerRootExtension.getToolVersion()的属性去使用自己期望的版本。这样dependencyorg.codehaus.sonar.runner:sonar-runner-dist:«toolVersion» 会被使用.

By default, version 2.3 of the Sonar Runner is used. To specify an alternative version, set the SonarRunnerRootExtension.getToolVersion() property of the sonarRunnerextension of the project the plugin was applied to to the desired version. This will result in the Sonar Runner dependencyorg.codehaus.sonar.runner:sonar-runner-dist:«toolVersion» being used as the Sonar Runner.

Example 36.3. 设置sonar runner的版本

Example 36.3. Configuring Sonar runner version

build.gradle
```
sonarRunner {
    toolVersion = '2.3' // default
}
```

## **36.5.分析Multi-Project构建**
36.5. Analyzing Multi-Project Builds

Sonar Runner 能够一次性分析整个项目层次。这能获取整个sonar的web接口、深入了解子项目。相比之下分析一个项目的层次要比单独分析每个项目的时间少。

The Sonar Runner is capable of analyzing whole project hierarchies at once. This yields a hierarchical view in the Sonar web interface, with aggregated metrics and the ability to drill down into subprojects. Analyzing a project hierarchy also takes less time than analyzing each project separately.

应用sonar插件分析根项目的层次结构。

To analyze a project hierarchy, apply the Sonar Runner plugin to the root project of the hierarchy. Typically (but not necessarily) this will be the root project of the Gradle build. Information pertaining to the analysis as a whole, like server and database connections settings, have to be configured in the sonarRunner block of this project. Any Sonar properties set on the command line also apply to this project.

Example 36.4 全局配置设置

Example 36.4. Global configuration settings

build.gradle
```
sonarRunner {
    sonarProperties {
        property "sonar.host.url", "http://my.server.com"
        property "sonar.jdbc.url", "jdbc:mysql://my.server.com/sonar"
        property "sonar.jdbc.driverClassName", "com.mysql.jdbc.Driver"
        property "sonar.jdbc.username", "Fred Flintstone"
        property "sonar.jdbc.password", "very clever"
    }
}
```

子项目之间共享配置可配置在subprojects block。

Configuration shared between subprojects can be configured in a subprojects block.

Example 36.5.分享配置设置

Example 36.5. Shared configuration settings

build.gradle
```
subprojects {
    sonarRunner {
        sonarProperties {
            property "sonar.sourceEncoding", "UTF-8"
        }
    }
}
```

Project-specific information 在相应的sonarRunner项目模块被设置

Project-specific information is configured in the sonarRunner block of the corresponding project.

Example 36.6.个别配置设置

Example 36.6. Individual configuration settings

build.gradle
```
project(":project1") {
    sonarRunner {
        sonarProperties {
            property "sonar.language", "grvy"
        }
    }
}
```

跳过特定子项目Sonar分析可设置sonarRunner.skipProject为ture即可。
To skip Sonar analysis for a particular subproject, set sonarRunner.skipProject to true.

Example 36.7. 跳过分析一个项目

Example 36.7. Skipping analysis of a project

build.gradle
```
project(":project2") {
    sonarRunner {
        skipProject = true
    }
}
```

## **36.6. 分析自定义的source sets**
36.6. Analyzing Custom Source Sets

默认情况，sonar runner插件把项目的source作为产品sources，test source作为自己的test sources。这个工作流程和产品源目录结构可以没有关系，Source sets可以需要时被指定。

By default, the Sonar Runner plugin passes on the project's main source set as production sources, and the project's test source set as test sources. This works regardless of the project's source directory layout. Additional source sets can be added as needed.

Example 36.8. 分析自定义的source sets

Example 36.8. Analyzing custom source sets

build.gradle
```
sonarRunner {
    sonarProperties {
        properties["sonar.sources"] += sourceSets.custom.allSource.srcDirs
        properties["sonar.tests"] += sourceSets.integTest.allSource.srcDirs
    }
}
```

## **36.7.分析java以外的语言**
36.7.Analyzing languages other than Java

如果项目没有使用java语言编写，你需要设置相应的sonar.project.language.需要注意的是sonar服务器需要有处理编程语言的sonar插件这样才能支持你设定的语言。

To analyze code written in a language other than Java, you'll need to set sonar.project.language accordingly. However, note that your Sonar server has to have theSonar plugin that handles that programming language.

Example 36.9 分析java以外的语言

Example 36.9. Analyzing languages other than Java

build.gradle

```
sonarRunner {
    sonarProperties {
        property "sonar.language", "grvy" // set language to Groovy
    }
}
```

在sonar 3.4版本中，每个项目只能解析一种语言。然而在多项目构建时可以分析多种语言。

As of Sonar 3.4, only one language per project can be analyzed. It is, however, possible to analyze a different language for each project in a multi-project build.

## **36.8.更多的Sonar属性设置**
36.8. More on configuring Sonar properties

我们仔细看sonarRunner.sonarProperties {} 块.就像示例说明的那样，the property()方法允许你设定新的属性或覆盖已存在的属性。

Let's take a closer look at the sonarRunner.sonarProperties {} block. As we have already seen in the examples, the property() method allows you to set new properties or override existing ones. 

此外,到目前为止所有配置的属性,包括被Gradle预配置的属性,通过属性访问器访问都是有效的。

Furthermore, all properties that have been configured up to this point, including all properties preconfigured by Gradle, are available via theproperties accessor.

可以使用Groovy对属性中的实体进行读写。为了方便执行，这些值都是“idiomatic”类型（ 例如文件， 列表等）。在sonar属性块被评估以后，数值按以下的方式被转换为字符串： 集合数值被(递归)转换为逗号分隔的字符串， 并且其他的数值使用toString()的方法完成转换。

Entries in the properties map can be read and written with the usual Groovy syntax. To facilitate their manipulation, values still have their “idiomatic” type (File, List, etc.). After the sonarProperties block has been evaluated, values are converted to Strings as follows: Collection values are (recursively) converted to comma-separated Strings, and all other values are converted by calling their toString() method.

由于sonar参数的懒加载，Gradle的对象模型属性可以从block内安全的被引用，而无需担心是否被设定。

Because the sonarProperties block is evaluated lazily, properties of Gradle's object model can be safely referenced from within the block, without having to fear that they have not yet been set.

## **36.9通过命令行的方式设置 Sonar**
36.9. Setting Sonar Properties from the Command Line

Sonar 属性可以通过命令行设置，命名和sonar属性几乎一致，对于处理一些敏感信息、系统信息、环境信息 或者设置ad-hoc都是有用的。

Sonar Properties can also be set from the command line, by setting a system property named exactly like the Sonar property in question. This can be useful when dealing with sensitive information (e.g. credentials), environment information, or for ad-hoc configuration.

gradle sonarRunner -Dsonar.host.url=http://sonar.mycompany.com -Dsonar.jdbc.password=myPassword -Dsonar.verbose=true

我们强烈建议大部分的设置放在build脚本里，最好是有版本标注的，这样对项目里得每个人都很友好。

While certainly useful at times, we do recommend to keep the bulk of the configuration in a (versioned) build script, readily available to everyone.

Sonar属性值可以通过系统属性设置，也可以在构建脚本重重写任意value（仅为相同的属性）。当分析项目层次结构时，可以通过系统属性设定values，每一个由’sonar' 开头的系统属性，都会在sonar runner初始化时被载入。

A Sonar property value set via a system property overrides any value set in a build script (for the same property). When analyzing a project hierarchy, values set via system properties apply to the root project of the analyzed hierarchy. Each system property starting with ""sonar." will taken into account for the sonar runner setup.


## **36.10. 控制Sonar Runner进程**
36.10. Controlling the Sonar Runner process

Sonar Runner在forked进程中被执行，它可以精确的控制内存、系统属性等，项目中sonarRunner扩展的TheforkOptions利用sonar-runner插件（通常是rootProject但不是必须的）允许指定进程配置。sonarrunnerextension扩展应用到子项目时，这些属性是无效的。

The Sonar Runner is executed in a forked process. This allows fine grained control over memory settings, system properties etc. just for the Sonar Runner process. TheforkOptions property of the sonarRunner extension of the project that applies the sonar-runner plugin (Usually the rootProject but not necessarily) allows the process configuration to be specified. This property is not available in the SonarRunnerExtension extension applied to the subprojects.

实例 36.10 设置自定义sonar Runner fork选项

Example 36.10. setting custom Sonar Runner fork options

build.gradle
'''
sonarRunner {
    forkOptions {
        maxHeapSize = '512m'
    }
}
'''

如果你想看关于有效选项的完整介绍，请查看JavaForkOptions.

For a complete reference about the available options, see JavaForkOptions.

## **36.11  任务**
36.11. Tasks

Sonar Runner 插件添加如下的任务到project

The Sonar Runner plugin adds the following tasks to the project.

Table 36.5. Sonar Runner plugin - tasks

|Task name	|Depends on	|Type	|Description|
|--
|sonarRunner	|-	|SonarRunner|Analyzes a project hierarchy and stores the results in the Sonar database.（分析一个项目的层级与存储结果到sonar数据库中）|

