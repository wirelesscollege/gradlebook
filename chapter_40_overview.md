# **第四十章. ANTLR插件**

Chapter 40. The ANTLR Plugin

ANTLR插件对Java 插件进行了扩展，并支持生成解析器。

The ANTLR plugin extends the Java plugin to add support for generating parsers using ANTLR.

ANTLR插件支持ANTLR版本 2,3,4

The ANTLR plugin supports ANTLR version 2, 3 and 4. 

## **40.1. 用法**

40.1. Usage

在build脚本中使用ANTLR插件，如下：

To use the ANTLR plugin, include the following in your build script:

例40.1 使用ANTLR 插件

Example 40.1. Using the ANTLR plugin

build.gradle
```
apply plugin: 'antlr'
```

## **40.2. Tasks**

40.2. Tasks

ANTLR 插件为项目添加了许多任务，如下图所示：

The ANTLR plugin adds a number of tasks to your project, as shown below.

表40.1. ANTLR 插件 -tasks

Table 40.1. ANTLR plugin - tasks

|Task name	|Depends on	|Type |Description|
|--
|generateGrammarSource| 	-	|AntlrTask|Generates the source files for all production ANTLR grammars.|
|generateTestGrammarSource |	-|	AntlrTask|Generates the source files for all test ANTLR grammars.|
|generateSourceSetGrammarSource 	|-	|AntlrTask|Generates the source files for all ANTLR grammars for the given source set.|

ANTLR插件需要添加如下的依赖任务

The ANTLR plugin adds the following dependencies to tasks added by the Java plugin.

表40.2. ANTLR插件-额外的任务依赖关系

Table 40.2. ANTLR plugin - additional task dependencies

|Task name|	Depends on|
|--
|compileJava|	generateGrammarSource|
|compileTestJava	|generateTestGrammarSource|
|compileSourceSetJava|	generateSourceSetGrammarSource|

## **40.3. 项目布局**

40.3. Project layout

表40.3 ANTLR 插件-项目布局

Table 40.3. ANTLR plugin - project layout

|Directory	|Meaning|
|--
|src/main/antlr 	|Production ANTLR grammar files.|
|src/test/antlr |	Test ANTLR grammar files.|
|src/sourceSet/antlr 	|ANTLR grammar files for the given source set.|

## **40.4. 依赖管理**

40.4. Dependency management

ANTLR插件添加了一个使用ANTLR提供ANTLR实现的依赖结构。下面的示例演示了如何使用ANTLR版本 3。

The ANTLR plugin adds an antlr dependency configuration which provides the ANTLR implementation to use. The following example shows how to use ANTLR version 3. 

例40.2 声明ANTLR版本

Example 40.2. Declare ANTLR version

build.gradle
```
repositories {
    mavenCentral()
}

dependencies {
    antlr "org.antlr:antlr:3.5.2" // use ANTLR version 3
}
```

如果没有依赖声明，ANTLR：ANTLR：2.7.7将作为默认。使用不同的ANTLR版本添加适当的依赖为ANTLR依赖配置以上。

If no dependency is declared, antlr:antlr:2.7.7 will be used as the default. To use a different ANTLR version add the appropriate dependency to the antlr dependency configuration as above. 

## **40.5. Convention 属性**

40.5. Convention properties

ANTLR插件没有添加任何convention属性

The ANTLR plugin does not add any convention properties.

## **40.6. Source set 属性**

40.6. Source set properties

ANTLR插件为项目中每个源文件添加以下属性：

The ANTLR plugin adds the following properties to each source set in the project.

Table 40.4. ANTLR plugin - source set properties

|Property name|	Type	|Default value|	Description|
|--
|antlr 	|SourceDirectorySet (read-only) |Not null 	|The ANTLR grammar files of this source set. Contains all .g files found in the ANTLR source directories, and excludes all other types of files. |
|antlr.srcDirs 	|Set<File>. Can set using anything described in Section 15.5, “Specifying a set of input files”.| [projectDir/src/name/antlr] 	|The source directories containing the ANTLR grammar files of this source set. |

## **40.7. 控制ANTLR进程**

40.7. Controlling the ANTLR generator process

ANTLR 工具在交叉进程中被执行，为了防止内存溢出，你需要为ANTLR进程设置heap size，这个值就是AntlrTask能够使用的最大heapsize值。

The ANTLR tool is executed in a forked process. This allows fine grained control over memory settings for the ANTLR process. To set the heap size of a ANTLR process, the maxHeapSize property of AntlrTask can be used. 

Example 40.3. setting custom max heap size for ANTLR

build.gradle

```
generateGrammarSource {
    maxHeapSize = "64m"
}
```
百度搜索[无线学院](http://wirelesscollege.cn)
