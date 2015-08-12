# **第二十二章. 标准的 Gradle 插件**
Gradle 的发行包中有大量的插件。如下列所示：

## 22.1. 语言插件

这些插件添加了让各种语言可以被编译和在JVM执行的支持。

表 22.1. 语言插件

|插件 Id|	自动应用|	与什么插件一起使用|	描述|
|-- | --| --| -- |
|java|	java-base|	-	|向一个项目添加 Java 编译、 测试和捆绑的能力。它是很多其他 Gradle 插件的基础服务。另请参阅第 7 章， Java 快速入门。|
|groovy	java| groovy-base|	-	|添加对 Groovy 项目构建的支持。另请参阅第9章，Groovy 快速入门。|
|scala	java,|scala-base|	-	|添加对 Scala 项目构建的支持。|
|antlr	|java|	-	|添加对使用Antlr作为生成解析器的支持。|

## 22.2. 正在孵化的语言插件

这些插件添加了对多种语言的支持：

表 22.2. 语言插件

|插件 Id|	自动应用|	与什么插件一起使用|	描述|
|--
|assembler|	-|	-|	向项目添加本机汇编语言的功能。|
|c|	-|	-|	向项目添加 C语言源代码编译功能。|
|cpp|	-|	-	|向项目添加 c++ 源代码编译功能。|
|objective-c|	-|	-	|向项目中添加 Objective-C 源代码编译功能。|
|objective-cpp|	-|	-	|向项目中添加 Objective-C++ 源代码编译功能。|
|windows-resources|	-|	-|	添加对在本地bin文件中包含 Windows 资源的支持。|

## 22.3. 集成插件

以下这些插件提供了一些与各种运行时技术的集成。

表 22.3. 集成插件

|插件 Id|	自动应用|	与什么插件一起使用|	描述|
|--
|application|	java|	-	|添加了一些任务，用于运行和捆绑一个Java项目作为命令行应用程序。|
|ear|	-|	java|	添加用于构建 J2EE 应用程序的支持。|
|jetty	|war|	-|	在构建中部署你的web程序到一个内嵌的Jetty web容器中。另请参阅第 10 章， Web 应用程序快速入门。
|maven	|-	|java, war	|添加用于将项目发布到 Maven 仓库的支持。|
|osgi|	java-base|	java	|添加构建 OSGi 捆绑包的支持。|
|war	|java|	-	|添加用于组装 web 应用程序的 WAR 文件的支持。另请参阅第 10 章， Web 应用程序快速入门。|

## 22.4. 孵化中的集成插件

以下这些插件提供了一些与各种运行时技术的集成。

表 22.4. 孵化中的集成插件

|插件 Id|	自动应用|	与什么插件一起使用|	描述|
|--
|distribution|	-|	-|	添加构建 ZIP 和 TAR 分发包的支持。|
|java-library-distribution	|java, distribution|	-|	添加构建一个Java类库的 ZIP 和 TAR 分发包的支持。|
|ivy-publish|	-|	java, war	|这个插件提供了新的 DSL，用于支持发布文件到 Ivy 存储库，改善了现有的 DSL。|
|maven-publish	|-	|java, war	|这个插件提供了新的 DSL，用于支持发布文件到 Maven 存储库，改善了现有的 DSL。|

## 22.5. 软件开发插件

这些插件提供一些软件开发过程上的帮助。

表 22.5. 软件开发插件

|插件 Id|	自动应用|	与什么插件一起使用|	描述|
|--
|announce|	-|	-	|将消息发布到你所喜爱的平台，如 Twitter 或 Growl。|
|build-announcements|	announce|	-	|在构建的生命周期中，把本地公告中有关你感兴趣的事件发送到你的桌面。|
|checkstyle	|java-base|	-	|使用Checkstyle对您的项目的 Java 源文件执行质量检查并生成报告。|
|codenarc|	groovy-base	|-	|使用CodeNarc对您的项目的 Groovy 源文件执行质量检查并生成报告。|
|eclipse|	-	|java,groovy,scala	|生成Eclipse IDE所用到的文件，从而使项目能够导入到 Eclipse。另请参阅第 7 章，Java 快速入门。|
|eclipse-wtp|	-|	ear, war	|与 eclipse 插件一样，但它还生成 eclipse WTP （Web 工具平台） 的配置文件。你的war/ear项目在导入eclipse 后，应配置为能在 WTP 中使用。另请参阅第 7 章， Java 快速入门。|
|findbugs	|java-base|	-	|使用FindBugs对您的项目的 Java 源文件执行质量检查并生成报告。|
|idea|	-|	java|	生成Intellij IDEA IDE所用到的文件，从而使项目能够导入到 IDEA。|
|jdepend	|java-base	|-	|使用JDepend对您的项目的源文件执行质量检查并生成报告。|
|pmd	|java-base	|-	|使用PMD对您的项目的 Java 源文件执行质量检查并生成报告。|
|project-report	|reporting-base	|-	|生成关于Gradle构建中有用的信息的报告。|
|signing|	base|	-	|添加对生成的文件或构件进行数字签名的功能。|
|sonar|	-	|java-base, java, jacoco|	提供对Sonar代码质量平台的集成。由 sonar-runner插件取代。|

## 22.6. 孵化中的软件开发插件

这些插件提供一些软件开发过程上的帮助。

表 22.6. 软件开发插件

|插件 Id	|自动应用	|与什么插件一起使用	|描述|
|--
|build-dashboard|	reporting-base	|-	|生成构建的主控面板的报表。|
|build-init	|wrapper|	-	|添加用于初始化一个新 Gradle 构建的支持。处理转换 Maven 构建为 Gradle 构建。|
|cunit|	-|	-	|添加用于运行CUnit测试的支持。|
|jacoco	|reporting-base|	java	|提供对 Java 的JaCoCo代码覆盖率库的集成。|
|sonar-runner|	-|	java-base, java, jacoco	|提供对Sonar代码质量平台的集成。由 sonar插件取代。|
|visual-studio|	-|	本机语言插件|	添加对 Visual Studio 的集成。|
|wrapper|	-|	-|	添加一个用于生成 Gradle wrapper 文件的Wrapper任务。|

## 22.7. 基本插件

这些插件组成了基本的构建块，其他插件都由此组装而来。它们可供你在你的构建文件中使用，并在此处完整列出。然而，请注意它们都不被认为是 Gradle 公共 API 的一部分。因此，这些插件都不在用户指南中记录。您可能会引用他们的 API 文档，以了解更多关于它们的信息。

表 22.7. 基本插件

插件 Id	描述
base	
添加标准的生命周期任务，并为归档任务默认进行合理的配置：

添加构建 ConfigurationName任务。这些任务组装成属于指定配置的构件。
添加上传ConfigurationName任务。这些任务组装并上传属于指定的配置的构件。
为所有归档任务配置合适的默认值（比如从AbstractArchiveTask继承来任务）。例如，以下类型的归档任务： Jar，Tar，Zip。特别是，归档任务的destinationDir、 baseName和version属性被预先配置了默认值。这是非常有用的，因为它促进了跨项目的一致性 ；完成了有关构件命名规范及构建之后的位置上的一致。

* java-base	
对项目添加source set 的概念。不会添加任何特定的soruce sets。

* groovy-base	
向项目中添加Groovy 的source set概念。

* scala-base	
向项目中添加Scala 的source set概念。

* reporting-base	
将一些共享的公约属性添加到项目中，它们与报告的生成有关。

## 22.8. 第三方插件

你可以在维基上找到外部插件的列表。

百度搜索[无线学院](http://wirelesscollege.cn)