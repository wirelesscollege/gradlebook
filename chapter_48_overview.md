# **Chapter 48. Wrapper Plugin**

wrapper plugin正在开发中，DSL和其他的配置可能会在之后的Gradle版本中发生变化。

The wrapper plugin is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.

Gradle wrapper plugin是通过添加一个Wrapper任务来生成Gradle wrapper文件的，此任务可以生成所有在使用Gradle wrapper运行构建时需要的必要文件。有关Gradle wrapper详细内容请参见Chapter 64, The Gradle Wrapper。

The Gradle wrapper plugin allows the generation of Gradle wrapper files by adding a Wrapper task, that generates all files needed to run the build using the Gradle Wrapper. Details about the Gradle Wrapper can be found in Chapter 64, The Gradle Wrapper.

## **48.1. 使用**

48.1. Usage

不用修改build.gradle文件，只需通过在命令行运行“gradle wrapper”命令，wrapper插件就可以自动应用于当前构建的根项目，适用于没有名为wrapper的任务中已定义在构建中的情况。

Without modifying the build.gradle file, the wrapper plugin can be auto-applied to the root project of the current build by running “gradle wrapper” from the command line. This applies the plugin if no task named wrapper is already defined in the build.

## **48.2. 任务**

48.2. Tasks

wrapper plugin添加如下任务到项目内：

The wrapper plugin adds the following tasks to the project:

Table 48.1. Wrapper plugin - tasks

|Task name	|Depends on	|Type	|Description|
|--
|wrapper|	-	|Wrapper|Generates Gradle wrapper files.|

百度搜索[无线学院](http://wirelesscollege.cn)

