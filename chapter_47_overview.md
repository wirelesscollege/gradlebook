# **第47章 构建初始化插件**

Chapter 47. Build Init Plugin

构建初始化插件正在开发中，在以后的gradle版本中可能会发生DSL或者其他配置的变化。

The Build Init plugin is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.

gradle构建初始化插件可以在引入一个新的gradle中间时使用，它支持在不同类型的项目中创建标记以更好的转化为gradle的构建任务，例如：Apache Maven build   

The Gradle Build Init plugin can be used to bootstrap the process of creating a new Gradle build. It supports creating brand new projects of different types as well as converting existing builds (e.g. An Apache Maven build) to be Gradle builds.

Gradle插件使用前需要在配置文件里声明（详见章节21.3“插件的应用”），而构建初始化插件是被自动应用的插件，那意味着你不需要在配置文件里去配置它，使用这个初始化插件其实就是在你创建Gradle 构建时就开始了，不需要创建一个“stub” build.gradle文件去应用它。

Gradle plugins typically need to be applied to a project before they can be used (see Section 21.3, “Applying plugins”). The Build Init plugin is an automatically applied plugin, which means you do not need to apply it explicitly. To use the plugin, simply execute the task named init where you would like to create the Gradle build. There is no need to create a “stub” build.gradle file in order to apply the plugin.

在wrapper类型任务里也是一样，你只需要保证Gradle Wrapper插件配置到项目里了就好了（见48章，Wrapper 插件）。

It also leverages the wrapper task from the Wrapper plugin (see Chapter 48, Wrapper Plugin), which means that the Gradle Wrapper will also be installed into the project.


## **47.1 任务**

47.1. Tasks

插件添加如下的任务到项目里：  
The plugin adds the following tasks to the project:

表 47.1 构建初始化插件--任务   
Table 47.1. Build Init plugin - tasks

|Task name|	Depends on|	Type|	Description|
|--
|init	|wrapper	|InitBuild|	Generates a Gradle project.|
|wrapper	|-	|Wrapper|	Generates Gradle wrapper files.|


## **47.2 需要设置什么**
47.2. What to set up


初始化支持不同的构建设置类型，这些类型可以使用参数来进行传递。例如：创建一个java library项目，你可以执行：gradle init --type java-library。  
The init supports different build setup types. The type is specified by supplying a --type argument value. For example, to create a Java library project simply execute: gradle init --type java-library.

如果--type参数不支持，那么gradle可以内引用环境变量。例如：我想要内引用一个pom类型得值，gradle将会寻找pom.xml完成一个gradle构建。  
If a --type parameter is not supplied, Gradle will attempt to infer the type from the environment. For example, it will infer a type value of “pom” if it finds a pom.xml to convert to a Gradle build.

如果类型没有引用，那么basic类型会被采用。   
If the type could not be inferred, the type “basic” will be used.

支持所有的构建类型包括Wrapper类型项目。    
All build setup types include the setup of the Gradle Wrapper.


## **47.3 构建初始化类型**

47.3. Build init types

因为插件正在开发中，所以仅仅只有一少部分的初始化类型支持，更多的类型将在未来的Gradle release版本中加入。

As this plugin is currently incubating, only a few build init types are currently supported. More types will be added in future Gradle releases.


### **47.3.1 “pom”（Maven标记）**
47.3.1. “pom” (Maven conversion)

pom类型可以将Apache Maven项目转化为gradle构建. 此插件可以将pom文件转化为一或多个gradle文件。但它仅仅在pom文件有效的情况下使用，如果这个文件存在，那么类型将会自动内引用。

The “pom” type can be used to convert an Apache Maven build to a Gradle build. This works by converting the POM to one or more Gradle files. It is only able to be used if there is a valid “pom.xml” file in the directory that the init task is invoked in. This type will be automatically inferred if such a file exists.

Maven conversion是由gradle community 小组成员开发，是使用maven2gradle工具实现的。  

The Maven conversion implementation was inspired by the maven2gradle tool that was originally developed by Gradle community members.

转化的过程有如下的特征：   
The conversion process has the following features:

使用有效的POM文件以及有效的设置  
Uses effective POM and effective settings (support for POM inheritance, dependency management, properties)

支持单模块项目以及多样化模块项目     
Supports both single module and multimodule projects

支持自定义模块名      
Supports custom module names (that differ from directory names)

产生大致数据-例如id，描述，和版本   
Generates general metadata - id, description and version

应用maven，java以及war插件      
Applies maven, java and war plugins (as needed)

支持打包war项目到jar包       
Supports packaging war projects as jars if needed

生成依赖，包含外置以及内置依赖     
Generates dependencies (both external and inter-module)

生成下载资源库，例如本地Maven仓库     
Generates download repositories (inc. local Maven repository)

仅支持java编译设置    
Adjusts Java compiler settings

支持打包原文件以及测试     
Supports packaging of sources and tests

支持TestNG runner     
Supports TestNG runner

依靠maven实施插件设置生成全局扩展     
Generates global exclusions from Maven enforcer plugin settings


### **47.3.2. “java-library”**

“java-library”类型初始化当前不支持推断使用，只能明确得声明为这个类型才能使用，这个和刚才的pom类型是有区别的，pom类型可以不声明type。    
The “java-library” build init type is not inferable. It must be explicitly specified.

它有以下的特征：    
It has the following features:

需要使用java插件    
Uses the “java” plugin

需要使用jcenter依赖资源库     
Uses the “jcenter” dependency repository

需要使用Junit测试    
Uses JUnit for testing

拥有本地源码转化目录    
Has directories in the conventional locations for source code

包含简单的类以及单元测试，如果项目不存在源码和测试文件的话     
Contains a sample class and unit test, if there are no existing source or test files


### **47.3.3. “scala-library”**

scala-library没有明确的初始化类型，它是需要明确指定type类型。      
The “scala-library” build init type is not inferable. It must be explicitly specified.

它有以下特征：   
It has the following features:

使用scala插件   
Uses the “scala” plugin

使用jcenter依赖仓库    
Uses the “jcenter” dependency repository

使用Scala2.10版本    
Uses Scala 2.10

使用ScalaTest测试    
Uses ScalaTest for testing

拥有本地源码转化目录    
Has directories in the conventional locations for source code

包含简单的scala类以及ScalaTest测试套件，如果项目不存在源码和测试文件的话   
Contains a sample scala class and an associated ScalaTest test suite, if there are no existing source or test files


### **47.3.4. “groovy-library”**

groovy-library没有明确的初始化类型，它也是需要明确指定type类型。     
The “groovy-library” build init type is not inferable. It must be explicitly specified.


它拥有如下特征：   
It has the following features:

使用groovy插件  
Uses the “groovy” plugin

使用jcenter仓库   
Uses the “jcenter” dependency repository

使用groovy 2.x版本    
Uses Groovy 2.x

使用Spock测试框架来测试    
Uses Spock testing framework for testing

拥有源码转化目录    
Has directories in the conventional locations for source code

包含简单的Groovy类以及associated Spock specification，如果项目不存在源码和测试文件的话    
Contains a sample Groovy class and an associated Spock specification, if there are no existing source or test files



### **47.3.5  基本类型”**

47.3.5. “basic

The “basic” build init type is useful for creating a fresh new Gradle project. It creates a sample build.gradle file, with comments and links to help get started.

基本类型在创建一个新的gradle项目时是非常有用的，它可以创建一个简单的gradle构建文件，拥有备注及连接帮助你开始使用。

This type is used when no type was explicitly specified, and no type could be inferred.

这种类型不需要你配置，也不需要你明确指定。