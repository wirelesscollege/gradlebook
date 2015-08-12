# **第三十章. CodeNarc 插件**
CodeNarc 插件使用CodeNarc对项目的 Groovy 源文件执行质量检查并生成报告。

## **30.1. 用法**
要使用 CodeNarc 插件，请在构建脚本中包含以下语句：

示例 30.1. 使用 CodeNarc 插件

build.gradle
```
apply plugin: 'codenarc'
```
该插件向你的项目添加了大量的执行质量检查的任务。你可以通过运行gradle check执行检查。

## **30.2. 任务**
CodeNarc 插件向project 中添加了以下任务：

表 30.1. CodeNarc 插件 - 任务

|任务名称|	依赖于|	类型|	描述|
|--
|codenarcMain|	-|	codenarc|	针对生产 Groovy 源文件运行 CodeNarc。|
|codenarcTest|	-|	codenarc|	针对测试 Groovy 源文件运行 CodeNarc。|
|codenarcSourceSet|	-	|codenarc	|针对给定的source set 的 Groovy 源文件运行 CodeNarc。|
|CodeNarc |插件向 |Groovy |插件所加入的任务添加了以下的依赖。|

表 30.2. CodeNarc 插件 - 附加的任务依赖

|任务名称|	依赖于|
|--
|check	|所有的 CodeNarc 任务，包括codenarcMain和codenarcTest。|

## **30.3. 项目布局**
CodeNarc 插件预计是以下的项目布局：

表 30.3. CodeNarc 插件 - 项目布局

|File|	意义|
|--
|config/codenarc/codenarc.xml	|CodeNarc 配置文件|

## **30.4. 依赖管理**
CodeNarc 插件添加了下列的依赖配置：

表30.4. CodeNarc 插件 ​​- 依赖配置

|名称	|意义|
|--
|codenarc|	使用的 CodeNarc 库|

## **30.5. 配置**
请参阅CodeNarcExtension。

百度搜索[无线学院](http://wirelesscollege.cn)

