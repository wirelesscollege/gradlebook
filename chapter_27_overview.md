# **第二十七章. Ear 插件**

Ear 插件添加了用于组装 web 应用程序的 EAR 文件的支持。它添加了一个默认的EAR archive task。它不需要 Java 插件，但是对于使用了 Java 插件的项目，它将禁用默认的 JAR archive 的生成。

## **27.1. 用法**
要使用 Ear 的插件，请在构建脚本中包含以下语句：

示例 27.1. 使用 Ear 插件

build.gradle
```
apply plugin: 'ear'
```

## **27.2. Tasks**
Ear 插件向project 中添加了以下任务。

表 27.1. Ear 插件 - tasks

|任务名称|	依赖于|	类型|	描述|
|--
|ear	|compile（仅在也配置了使用 Java 插件的时候）|	ear|	组装应用程序 EAR 文件。|

Ear 插件向基础插件所加入的 tasks 添加了以下的依赖。

表 27.2. Ear 插件 - 额外的task 依赖

|任务名称|	依赖于|
|--
|assemble|	ear|

## **27.3. 项目布局**
表 27.3. Ear 插件 - 项目布局

|目录	|意义|
|--
|src/main/application	|Ear 资源，如 META-INF 目录|

## **27.4. 依赖管理**
Ear 插件添加了两个依赖配置：deploy和earlib。所有在deploy配置中的依赖项都放在 EAR 文件的根目录中，并且是不可传递的。所有在 earlib 配置的依赖都放在 EAR 文件的“lib”目录中，并且是 可 传递的。

## **27.5. 公约属性**
表27.4. Ear插件 ​​- 目录属性

|属性名称|	类型	|默认值	|描述|
|--
|appDirName|	String|	src/main/application|	相对于项目目录的应用程序源目录名称。|
|libDirName	|String	|into(<s2>'libs'</s2>) |{	生成的 EAR 文件里的 lib 目录名称。|
|deploymentDescriptor|	org.gradle.plugins.ear.descriptor.DeploymentDescriptor	|部署描述符，它有一个合理的名为application.xml的默认值	用于生成部署描述符文件的元数据，例如application.|xml。如果此文件已存在于appDirName/META-INF，那么会使用这个已存在的文件的内容，而ear.deploymentDescriptor中的显式配置将被忽略。|

这些属性由一个EarPluginConvention 公约对象提供。

## **27.6. Ear**
Ear task 的默认行为是将src/main/application的内容复制到archive 的根目录下。如果你的application目录没有包含META-INF/application.xml部署描述符，那么将会为你生成一个。

另请参阅 Ear。

## **27.7. 自定义**
下面是一个示例，展示了最重要的自定义选项：

示例 26.2. ear 插件的自定义

build.gradle
```
apply plugin: 'ear'
apply plugin: 'java'

repositories { mavenCentral() }

dependencies {
    //following dependencies will become the ear modules and placed in the ear root
    deploy project(':war')

    //following dependencies will become ear libs and placed in a dir configured via libDirName property
    earlib group: 'log4j', name: 'log4j', version: '1.2.15', ext: 'jar'
}

ear {
    appDirName 'src/main/app'  // use application metadata found in this folder
    libDirName 'APP-INF/lib'  // put dependency libraries into APP-INF/lib inside the generated EAR;
                                // also modify the generated deployment descriptor accordingly
    deploymentDescriptor {  // custom entries for application.xml:
//      fileName = "application.xml"  // same as the default value
//      version = "6"  // same as the default value
        applicationName = "customear"
        initializeInOrder = true
        displayName = "Custom Ear"  // defaults to project.name
        description = "My customized EAR for the Gradle documentation"  // defaults to project.description
//      libraryDirectory = "APP-INF/lib"  // not needed, because setting libDirName above did this for us
//      module("my.jar", "java")  // wouldn't deploy since my.jar isn't a deploy dependency
//      webModule("my.war", "/")  // wouldn't deploy since my.war isn't a deploy dependency
        securityRole "admin"
        securityRole "superadmin"
        withXml { provider -> // add a custom node to the XML
            provider.asNode().appendNode("data-source", "my/data/source")
        }
    }
}
```
你还可以使用Ear任务提供的自定义选项，如from和metaInf。

## **27.8. 使用自定义的描述符文件**
假设你已经有了application.xml ，并且想要使用它而不是去配置ear.deploymentDescriptor代码段。去把 META-INF/application.xml 放在你的源文件夹里的正确的位置（请查看 appDirName 属性）。这个已存在的文件的内容将会被使用，而 ear.deploymentDescriptor 里的显示配置则会被忽略。