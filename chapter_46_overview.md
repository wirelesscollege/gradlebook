# **第46章 Java lib发布插件**

Chapter 45. The Java Library Distribution Plugin

Java lib发布插件目前正在开发中。请注意，在以后的Gradleb版本中DSL和其他配置可能会改变。

The Java library distribution plugin is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.

该插件支持zip格式发布lib包，且发行的jar包包含所需要的依赖。

The Java library distribution plugin adds support for building a distribution ZIP for a Java library. The distribution contains the JAR file for the library and its dependencies.

## **45.1  运用**

45.1. Usage

要使用java lib发布插件，需要包含以下构建脚本：

To use the Java library distribution plugin, include the following in your build script:

例45.1 使用java lib 发布插件

Example 45.1. Using the Java library distribution plugin

build.gradle
```
apply plugin: 'java-library-distribution'
```

定义名称时你必须设置baseName属性，如下所示：

To define the name for the distribution you have to set the baseName property as shown below:

例 45.2 配置发布名称

Example 45.2. Configure the distribution name

build.gradle
```
distributions {
    main{
        baseName = 'my-name'
    }
}
```

该插件库构建一个发布，而这个发布包运行时将依赖于这个库。所有文件将被添加存储到根目录下的src/main/dist目录中。你可以执行“gradle distZip”去创建一个包含发布信息的zip文件。

The plugin builds a distribution for your library. The distribution will package up the runtime dependencies of the library. All files stored in src/main/dist will be added to the root of the archive distribution. You can run “gradle distZip” to create a ZIP file containing the distribution.

## **45.2 任务**

45.2. Tasks

Java lib 发布插件增加了以下任务到你的项目中。

The Java library distribution plugin adds the following tasks to the project.

表45.1 java lib 发布插件--任务

Table 45.1. Java library distribution plugin - tasks

|任务名称|	依赖于|	类型	|描述|
|--
|distZip	|jar|	Zip|创建一个完整的zip文件并包含运行库|

|Task name	|Depends on	|Type	|Description|
|--
|distZip	|jar	|Zip|Creates a full distribution ZIP archive including runtime libraries.|

## **45.3 发布包含的其他资源**

45.3. Including other resources in the distribution

所有的文件都拷贝至src/dist目录中。其中包含的任何静态文件，只需将他们放置在src/dist目录中，或将他们添加到发布的目录中。

All of the files from the src/dist directory are copied. To include any static files in the distribution, simply arrange them in the src/dist directory, or add them to the content of the distribution.

例 45.3 发布包含的文件

Example 45.3. Include files in the distribution

build.gradle
```
distributions {
    main {
        baseName = 'my-name'
        contents {
            from { 'src/dist' }
        }
    }
}
```

百度搜索[无线学院](http://wirelesscollege.cn)



