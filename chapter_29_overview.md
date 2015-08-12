# **第二十九章. Checkstyle 插件**
Checkstyle 插件使用Checkstyle对你的项目的 Java 源文件执行质量检查，并从检查结果中生成报告。

## **29.1. 用法**
要使用 Checkstyle 插件，请在构建脚本中包含以下语句：

示例 29.1. 使用 Checkstyle 插件

build.gradle
```
apply plugin: 'checkstyle'
```

该插件向你的项目添加了大量的执行质量检查的任务。你可以通过运行gradle check执行检查。

## **29.2. Tasks**
Checkstyle 插件向project 中添加了以下tasks：

表 29.1. Checkstyle 插件 - tasks

|任务名称	|依赖于	|类型	|描述|
|--
|checkstyleMain	|classes|	checkstyle	|针对生产Java 源文件运行 Checkstyle。|
|checkstyleTest	|testClasses	|checkstyle	|针对测试 Java 源文件运行 Checkstyle。|
|checkstyleSourceSet	|sourceSetClasses|	checkstyle	|针对source set 的 Java 源文件运行 Checkstyle。|

Checkstyle 插件向 Java 插件所加入的 tasks 添加了以下的依赖。

表 29.2. Checkstyle 插件 - 额外的task 依赖

|任务名称	|依赖于|
|--
|check|	所有 Checkstyle tasks，包括checkstyleMain和checkstyleTest。|

## **29.3. 项目布局**
Checkstyle 插件预计是以下的项目布局：

表 29.3. Checkstyle 插件 - 项目布局

|File|	意义|
|--
|config/checkstyle/checkstyle.xml|	Checkstyle 配置文件|

## **29.4. 依赖管理**
Checkstyle 插件添加了下列的依赖配置：

表29.4. Checkstyle 插件 ​​- 依赖配置

|名称	|意义|
|--
|checkstyle	|用到的 Checkstyle 库|

## **29.5. 配置**
请参阅CheckstyleExtension。

百度搜索[无线学院](http://wirelesscollege.cn)
