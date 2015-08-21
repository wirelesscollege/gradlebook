# **第44章  Distribution插件**

Chapter 44. The Distribution Plugin

Distribution插件目前还在研发阶段，请注意在以后的Gradle版本中DSL和其他配置可能会有变化。

The distribution plugin is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.

Distribution插件可以帮助生成发布项目的压缩包，发布包通常包括可执行程序和其他支持文件例如文档。

The distribution plugin facilitates building archives that serve as distributions of the project. Distribution archives typically contain the executable application and other supporting files, such as documentation. 

## **44.1. 使用**

44.1. Usage

要使用distribution插件，在你的构建脚本中需包含下面的代码：

To use the distribution plugin, include the following in your build script:

例子44.1. 使用distribution插件

Example 44.1. Using the distribution plugin

build.gradle
```
apply plugin: 'distribution'
```


该插件给项目添加了属于DistributionContainer类型的名为“distributions”的扩展名。它还在distribution扩展容器中创建了一个名为“main”的distribution。如果你的build只产生了一个distribution你只需要配置这个distribution（或使用默认配置）

The plugin adds an extension named “distributions” of type DistributionContainer to the project. It also creates a single distribution in the distributions container extension named “main”. If your build only produces one distribution you only need to configure this distribution (or use the defaults). 


你可以运行 “gradle distZip”把要发布的程序打包成一个Zip，或者运行“gradle distTar”去创建一个TAR文件，运行assembleDist去构建前面两种形式的文件，文件将被创建到“$buildDir/distributions/$project.name-$project.version.«ext»”路径下.你可以运行“gradle installDist” 来组装为压缩的发布到“$buildDir/install/main”中。

You can run “gradle distZip” to package the main distribution as a ZIP, or “gradle distTar” to create a TAR file. To build both types of archives just run gradle assembleDist. The files will be created at “$buildDir/distributions/$project.name-$project.version.«ext»”.You can run “gradle installDist” to assemble the uncompressed distribution into “$buildDir/install/main”. 

## **44.2 任务**

44.2. Tasks

表44.1  Distribution插件--任务

Table 44.1. Distribution plugin - tasks

|Task name|	Depends on	|Type|	Description|
|--
|distZip |	- 	|Zip |创建一个发布文件的Zip包 |
|distTar |	- 	|Tar |创建一个发布文件的TAR包 |
|assembleDist |	distTar, distZip |	Task |创建一个发布文件的ZIP和Tar包 |
|installDist 	|- |	Sync |组装发布文件并安装到当前机器 |

对于每一个你设置到程序中的额外的发布，Distribution插件都会添加如下的任务：

For each extra distribution set you add to the project, the distribution plugin adds the following tasks:

表44.2.   多重发布---任务

Table 44.2. Multiple distributions - tasks

|Task name|	Depends on|	Type|	Description|
|--
|${distribution.name}DistZip 	|-| 	Zip| 创建一个发布文件的Zip包 |
|${distribution.name}DistTar |	- |	Tar |创建一个发布文件的TAR包|
|assemble${distribution.name.capitalize()}Dist |	${distribution.name}DistTar, ${distribution.name}DistZip 	|Task |创建一个发布文件的ZIP和Tar包 |
|install${distribution.name.capitalize()}Dist |	- |	Sync |组装发布文件并安装到当前机器 |

例子44.2. 添加额外的发布

Example 44.2. Adding extra distributions

build.gradle
```
apply plugin: 'distribution'

version = '1.2'
distributions {
    custom {}
}
```

这会添加下面的任务到项目中：

This will add following tasks to the project: 

*•	打包成zip文件
*•	customDistZip
*•	创建Tar文件
*•	customDistTar
*•	组装发布文件
*•	assembleCustomDist
*•	安装发布文件
*•	installCustomDist

假设项目名称是“myproject”并且版本号是1.2，运行“gradle customDistZip” 命令将会产生一个名称为“myproject-custom-1.2.zip”的ZIP文件。

Given that the project name is “myproject” and version “1.2”, running “gradle customDistZip” will produce a ZIP file named “myproject-custom-1.2.zip”. 

运行“gradle installCustomDist”命令将会安装发布文件到“$buildDir/install/custom”目录下。

Running “gradle installCustomDist” will install the distribution contents into “$buildDir/install/custom”. 

## **44.3. Distribution布内容**

44.3. Distribution contents

“src/$distribution.name/dist”目录下的所有文件将会自动的包含到distribution中，你可以通过配置容器中的发布对象来添加另外的文件。

All of the files in the “src/$distribution.name/dist” directory will automatically be included in the distribution. You can add additional files by configuring the Distribution object that is part of the container. 

例子44.3.  配置主要的发布

Example 44.3. Configuring the main distribution

build.gradle
```
apply plugin: 'distribution'

distributions {
    main {
        baseName = 'someName'
        contents {
            from { 'src/readme' }
        }
    }
}

apply plugin:'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://some/repo")
        }
    }
}
```

在上面的例子中，“src/readme”目录下的内容都会被包含在发布中（包括“src/dist/main”目录下的默认添加的内容）

In the example above, the content of the “src/readme” directory will be included in the distribution (along with the files in the “src/dist/main” directory which are added by default). 

“baseName”属性也会被改变，这会致使发布打包的文件会被已不同的名字创建。

The “baseName” property has also been changed. This will cause the distribution archives to be created with a different name. 

## **44.4. 发布**

44.4. Publishing distributions

Distribution插件会把打包好的文件作为默认的发布目标文件之一。如果没有其他的目标文件配置的话，当执行uploadArchives 时maven插件会执行发布这个打包好的这个文件。

The distribution plugin adds the distribution archives as candidate for default publishing artifacts. With the maven plugin applied the distribution zip file will be published when running uploadArchives if no other default artifact is configured 

例子44.4.    发布主文件

Example 44.4. publish main distribution

build.gradle
```
apply plugin:'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://some/repo")
        }
    }
}
```


