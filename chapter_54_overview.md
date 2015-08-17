# **第五十四章  Maven 插件**

Chapter 54. The Maven Plugin

本章还在编写中

This chapter is a work in progress.

maven插件可以添加项目依赖到maven仓库.

The Maven plugin adds support for deploying artifacts to Maven repositories.

## **54.1.  使用**

54.1.  Usage

要使用maven插件，在项目的build 脚本中加入如下代码：

To use the Maven plugin, include the following in your build script:

例子54.1.  使用maven插件

Example 54.1. Using the Maven plugin

build.gradle
```
apply plugin: 'maven'
```

## **54.2.  任务**

54.2.  Tasks

maven插件定义了如下的任务：

The Maven plugin defines the following tasks:

表54.1.   maven 插件---任务

Table 54.1. Maven plugin - tasks

|Task name|	Depends on|	Type|	Description|
|--
|install 	|All |tasks that build the associated archives. |	Upload|

安装相应的产物到maven缓存包括maven产生的元数据。安装任务默认的会伴随着产物配置。这个配置默认的只有一个jar包作为元素。要了解更多请参阅Section 54.6.3, “Installing to the local repository”

## **54.3. 依赖管理**

54.3. Dependency management

maven插件没有定义任何的依赖配置

The Maven plugin does not define any dependency configurations.

## **54.4. 约定属性**

54.4. Convention properties

maven插件定义了下面一些属性：

The Maven plugin defines the following convention properties:

表54.2. Maven插件---属性

Table 53.2. Maven plugin - properties

|Property name	|Type	|Default value	|Description|
|--
|pomDirName |	String |	poms |	写产生的pom文件的路径，相对于build目录|
|pomDir 	|File (read-only) 	|buildDir/pomDirName |	已产生的pom文件被写入的路径 |
|conf2ScopeMappings |	Conf2ScopeMappingContainer |n/a |	|

Grade配置与maven范围的介绍。请查看 Section 53.6.4.2, “Dependency mapping”. 

这些属性是由MavenPluginConvention 条约对象提供的。

These properties are provided by a MavenPluginConvention convention object.

## **54.5. 约定方法**

54.5. Convention methods

maven插件提供了一个创建POM文件的工厂类方法，这种方法当你需要一个不依赖于maven 仓库上下文的pom文件时是有用的。

The maven plugin provides a factory method for creating a POM. This is useful if you need a POM without the context of uploading to a Maven repo.

例子54.2. 创建一个单独的pom文件

Example 54.2. Creating a stand alone pom.
build.gradle
```
task writeNewPom << {
    pom {
        project {
            inceptionYear '2008'
            licenses {
                license {
                    name 'The Apache Software License, Version 2.0'
                    url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    distribution 'repo'
                }
            }
        }
    }.writeTo("$buildDir/newpom.xml")
}
```

除其他事情外，gradle支持maven通用的相同的构建语法。要了解更多的Gradle Maven POM 对象，请查看 MavenPom。也可以看看：MavenPluginConvention

Amongst other things, Gradle supports the same builder syntax as polyglot Maven. To learn more about the Gradle Maven POM object, see MavenPom. See also: MavenPluginConvention 

## **54.6. 与Maven 仓库交互**

54.6. Interacting with Maven repositories

### **54.6.1. 介绍***

54.6.1. Introduction

使用Gradle你可以部署到远程maven仓库或者安装到你本地的maven仓库。这包括maven所有的元数据以及快照。事实上，Gradle的部署是百分之百和maven兼容的，因为Gradle的底层实质上就是在执行maven ant 的任务。

With Gradle you can deploy to remote Maven repositories or install to your local Maven repository. This includes all Maven metadata manipulation and works also for Maven snapshots. In fact, Gradle's deployment is 100 percent Maven compatible as we use the native Maven Ant tasks under the hood. 

如果你没有一个pom文件那么部署到一个maven仓库只相当于做了一半的工作，幸运的是Gradle可以依据依赖信息自动为你产生pom文件。

Deploying to a Maven repository is only half the fun if you don't have a POM. Fortunately Gradle can generate this POM for you using the dependency information it has. 

## ***54.6.2. 部署一个maven仓库***

54.6.2. Deploying to a Maven repository

假设你的工程只产生了一个默认的jar包，现在你想把这个jar包部署到一个远程的maven仓库。

Let's assume your project produces just the default jar file. Now you want to deploy this jar file to a remote Maven repository. 

例子54.3.  上传一个文件到远程maven仓库

Example 54.3. Upload of file to remote Maven repository

build.gradle
```
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```

以上就是全部。调用uploadArchives方法将会产生一个pom文件并且上传这个pom文件和jar文件到指定的maven仓库。

That is all. Calling the uploadArchives task will generate the POM and deploys the artifact and the POM to the specified repository. 

如果你需要协议的支持那你需要做更多的工作。在这种情况下，那我们所举的例子本地的maven代码将需要更多的依赖的包，根据你依赖的协议的不同将决定你需要什么样的包。可用的协议以及相应的依赖详见表54.3 “Protocol jars for Maven deployment”（这些库具有传递依赖关系）例如，要使用ssh协议你可以这样做：

There is more work to do if you need support for protocols other than file. In this case the native Maven code we delegate to needs additional libraries. Which libraries are needed depends on what protocol you plan to use. The available protocols and the corresponding libraries are listed in Table 54.3, “Protocol jars for Maven deployment” (those libraries have transitive dependencies which have transitive dependencies). [19] For example, to use the ssh protocol you can do: 

例子54.4.    通过ssh上传一个文件

Example 54.4. Upload of file via SSH

build.gradle
```
configurations {
    deployerJars
}

repositories {
    mavenCentral()
}

dependencies {
    deployerJars "org.apache.maven.wagon:wagon-ssh:2.2"
}

uploadArchives {
    repositories.mavenDeployer {
        configuration = configurations.deployerJars
        repository(url: "scp://repos.mycompany.com/releases") {
            authentication(userName: "me", password: "myPassword")
        }
    }
}
```

对于部署人员来说有好多配置的选择。这些配置由Groovy 编译器来执行完成。这个树上的所有节点都是java 对象。要配置一个简单的属性你可以传递一个map给java对象。要把对象添加到父对象，你可以使用闭包。上面的例子中的repository和authentication就是这样的对象。表54.4“Configuration elements of the MavenDeployer”列出了所有可用的对象元素以及它对应的javadoc文档的链接。在javadoc文档中你可以查看某个对象的某个特定的属性的设置方法。

There are many configuration options for the Maven deployer. The configuration is done via a Groovy builder. All the elements of this tree are Java beans. To configure the simple attributes you pass a map to the bean elements. To add bean elements to its parent, you use a closure. In the example above repository and authentication are such bean elements. Table 54.4, “Configuration elements of the MavenDeployer” lists the available bean elements and a link to the Javadoc of the corresponding class. In the Javadoc you can see the possible attributes you can set for a particular element. 

在maven中你可以定义仓库地址以及可选的快照地址。如果没有快照被定义，发布包和快照包都将会被部署到定义的仓库中。否则快照就会被部署到快照仓库中。

In Maven you can define repositories and optionally snapshot repositories. If no snapshot repository is defined, releases and snapshots are both deployed to the repository element. Otherwise snapshots are deployed to the snapshotRepository element. 

表54.3.  maven部署可用的协议jar包

Table 54.3. Protocol jars for Maven deployment

|Protocol	|Library|
|--
|http	|org.apache.maven.wagon:wagon-http:2.2|
|ssh	|org.apache.maven.wagon:wagon-ssh:2.2|
|ssh-external	|org.apache.maven.wagon:wagon-ssh-external:2.2|
|ftp	|org.apache.maven.wagon:wagon-ftp:2.2|
|webdav	|org.apache.maven.wagon:wagon-webdav:1.0-beta-2|
|file	|-|

表54.4.  maven部署的配置方法

Table 54.4. Configuration elements of the MavenDeployer

|Element	|Javadoc|
|--
|root	|MavenDeployer |
|repository |	org.apache.maven.artifact.ant.RemoteRepository |
|authentication	|org.apache.maven.artifact.ant.Authentication |
|releases	|org.apache.maven.artifact.ant.RepositoryPolicy |
|snapshots|	org.apache.maven.artifact.ant.RepositoryPolicy |
|proxy	|org.apache.maven.artifact.ant.Proxy |
|snapshotRepository	|org.apache.maven.artifact.ant.RemoteRepository |

## ***54.6.3    安装到本地仓库***

54.6.3   Installing to the local repository

maven插件添加了一个安装任务到你的项目。这个任务依赖于打包配置中所有的打包任务。它安装这些打好的包到你的本地maven仓库。如果本地仓库默认的地址在maven settings.xml文件中被重定义了，这个任务会考虑这个配置。

The Maven plugin adds an install task to your project. This task depends on all the archives task of the archives configuration. It installs those archives to your local Maven repository. If the default location for the local repository is redefined in a Maven settings.xml, this is considered by this task. 

## ***54.6.4. maven pom 的生成***

54.6.4.  Maven POM generation

当部署一个产物到maven仓库的时候，Gradle自动的会为它生成一个pom文件。groupId，artifactId，version 以及 packaging 元素在pom中默认属性和值展示如下表。依赖元素是从项目的依赖声明中产生的。

When deploying an artifact to a Maven repository, Gradle automatically generates a POM for it. The groupId, artifactId, version and packaging elements used for the POM default to the values shown in the table below. The dependency elements are created from the project's dependency declarations. 

表54.5. maven pom 生成的默认职

Table 54.5. Default Values for Maven POM generation

|Maven Element	|Default Value|
|--
|groupId	|project.group|
|artifactId	|uploadTask.repositories.mavenDeployer.pom.artifactId (if set) or archiveTask.baseName.|
|version	|project.version|
|packaging	|archiveTask.extension|

这里，uploadTask 和 archiveTask 分别指的是用来上传和产生构建产物的方法。相应的archiveTask.baseName也是基于project.archivesBaseName生成的。

Here, uploadTask and archiveTask refer to the tasks used for uploading and generating the archive, respectively (for example uploadArchives and jar). archiveTask.baseName defaults to project.archivesBaseName which in turn defaults to project.name. 

当你设置archiveTask.baseName 属性的值不是默认的时候，你将也不得不设置uploadTask.repositories.mavenDeployer.pom.artifactId成相同的值。否则，你手上的项目可能会依赖错误的artifact ID，因为这个artifact ID是由pom文件为build其他项目产生的。

When you set the “archiveTask.baseName” property to a value other than the default, you'll also have to set uploadTask.repositories.mavenDeployer.pom.artifactId to the same value. Otherwise, the project at hand may be referenced with the wrong artifact ID from generated POMs for other projects in the same build. 

产生的pom文件可以在 build目录下找到。它可以由用户根据MavenPom API编辑自定义。例如，你也许希望部署到maven仓库的产物是另外的版本或名称而不是Gradle产生的artifact。要编辑它你可以这样做：

Generated POMs can be found in <buildDir>/poms. They can be further customized via the MavenPom API. For example, you might want the artifact deployed to the Maven repository to have a different version or name than the artifact generated by Gradle. To customize these you can do: 

例子 54.5.   编辑一个pom文件
Example 54.5. Customization of pom

build.gradle
```
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
            pom.version = '1.0Maven'
            pom.artifactId = 'myMavenName'
        }
    }
```

可以使用pom.project 编译器添加另外的内容到pom文件，有了这个编辑器，pom依赖文件中罗列出的任意的元素都可以被添加。

To add additional content to the POM, the pom.project builder can be used. With this builder, any element listed in the Maven POM reference can be added. 

例子54.6.  编辑pom文件的格式

Example 54.6. Builder style customization of pom

build.gradle
```
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
            pom.project {
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                        distribution 'repo'
                    }
                }
            }
        }
    }
}
```

请注意：groupId，artifactId，version以及package必须直接在pom文件中设置。

Note: groupId, artifactId, version, and packaging should always be set directly on the pom object. 

例子54.7. 编辑自动产生的内容

Example 54.7. Modifying auto-generated content

build.gradle
```
def installer = install.repositories.mavenInstaller
def deployer = uploadArchives.repositories.mavenDeployer

[installer, deployer]*.pom*.whenConfigured {pom ->
    pom.dependencies.find {dep -> dep.groupId == 'group3' && dep.artifactId == 'runtime' }.optional = true
}
```

如果你需要发布不止一个产物，那么配置的方式将会稍微不同。参加54.6.4.1.小节“Multiple artifacts per project”

If you have more than one artifact to publish, things work a little bit differently. See Section 54.6.4.1, “Multiple artifacts per project”. 

要自定义maven 安装器的设置（见54.6.3." Installing to the local repository "）你可以这样做：

To customize the settings for the Maven installer (see Section 54.6.3, “Installing to the local repository”), you can do: 

例子 54.8.  编辑maven 安装器

Example 54.8. Customization of Maven installer

build.gradle
```
install {
    repositories.mavenInstaller {
        pom.version = '1.0Maven'
        pom.artifactId = 'myName'
    }
}
```

## ***54.6.4.1 一个项目多个产物***

54.6.4.1. Multiple artifacts per project

Maven只能处理一个项目一个产物的情况。这个在maven的pom文件中也有体现。我们认为有很多方法可以让一个项目有多于一个的产物。在这样的情况下你需要生成多个pom文件并且你必须精确的声明你要部署到maven仓库地址的每个产物。MavenDeployer和MavenInstaller都为此提供了相应的API：

Maven can only deal with one artifact per project. This is reflected in the structure of the Maven POM. We think there are many situations where it makes sense to have more than one artifact per project. In such a case you need to generate multiple POMs. In such a case you have to explicitly declare each artifact you want to publish to a Maven repository. The MavenDeployer and the MavenInstaller both provide an API for this: 

例子54.9.  产生多个pom文件

Example 54.9. Generation of multiple poms

build.gradle
```
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
            addFilter('api') {artifact, file ->
                artifact.name == 'api'
            }
            addFilter('service') {artifact, file ->
                artifact.name == 'service'
            }
            pom('api').version = 'mySpecialMavenVersion'
        }
    }
}
```

你需要为每个要发布的项目建立一个过滤器，该过滤器定义了一个布尔表达式，用来判断什么样的Gradle产物是maven接受的。每个过滤器关联着一个你可以配置的pom文件。想要了解更多细节你可以查阅 PomFilterContainer 以及它相关联的类。

You need to declare a filter for each artifact you want to publish. This filter defines a boolean expression for which Gradle artifact it accepts. Each filter has a POM associated with it which you can configure. To learn more about this have a look at PomFilterContainer and its associated classes. 

## ***54.6.4.2. 依赖映射***

54.6.4.2. Dependency mapping

maven插件配置了由java和war插件添加的Gradle配置与maven范围之间的默认映射关系。大多数情况下你不需要去触及这部分也就是你可以安全的跳过这部分。该映射关系的工作方式如下。你只能让一个配置与一个范围相对应。多个配置可以被映射到一个或多个范围。你也可以给一个配置--范围映射赋予一个优先级。想要了解更多请参阅Conf2ScopeMappingContainer。要访问映射配置你可以这样做：

The Maven plugin configures the default mapping between the Gradle configurations added by the Java and War plugin and the Maven scopes. Most of the time you don't need to touch this and you can safely skip this section. The mapping works like the following. You can map a configuration to one and only one scope. Different configurations can be mapped to one or different scopes. You can also assign a priority to a particular configuration-to-scope mapping. Have a look at Conf2ScopeMappingContainer to learn more. To access the mapping configuration you can say: 

例子54.10. 访问一个映射配置

Example 54.10. Accessing a mapping configuration

build.gradle
```
task mappings << {
    println conf2ScopeMappings.mappings
```
如果可能的话Gradle排除规则会转换成 Maven排除规则。这种转化只有在Gradle排除规则里组和模块名称被指定时才可行（如同Maven相对Ivy两者都需要一样）。如果他们是可转换的，每个配置排除规则也会包含在Maven POM中。
Gradle exclude rules are converted to Maven excludes if possible. Such a conversion is possible if in the Gradle exclude rule the group as well as the module name is specified (as Maven needs both in contrast to Ivy). Per-configuration excludes are also included in the Maven POM, if they are convertible. 
