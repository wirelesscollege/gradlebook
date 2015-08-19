# **第53章．发布构建产物**

Chapter 53. Publishing artifacts  

本章将介绍在Gradle 1.0至Gradle 1.3中介绍过的一个新的发布机制，
但这个新的机制是孵化产品还不成熟，它引入了一些新的概念与特征，这将会使Gradle publishing 如虎添翼。

This chapter describes the original publishing mechanism available in Gradle 1.0: in Gradle 1.3 a new mechanism for publishing was introduced. While this new mechanism is incubating and not yet complete, it introduces some new concepts and features that do (and will) make Gradle publishing even more powerful. 

你可以在67章Ivy Publishing (new)和68章Maven Publishing (new)中读到关于新出版物插件的信息。请试着使用它们，并给我们反馈使用中的问题。

You can read about the new publishing plugins in Chapter 67, Ivy Publishing (new) and Chapter 68, Maven Publishing (new). Please try them out and give us feedback. 

## **53.1引言**

53.1. Introduction

本章是关于如何声明项目的构建产物，以及在工作中如何利用这些构建产物。定义项目的产物作为文件提供给外面的世界。产物有可能是个库或者是 ZIP文件或者是其他的文件，并且想发布多少产物就可以发布多少。

This chapter is about how you declare the outgoing artifacts of your project, and how to work with them (e.g. upload them). We define the artifacts of the projects as the files the project provides to the outside world. This might be a library or a ZIP distribution or any other file. A project can publish as many artifacts as it wants. 

## **53.2.产物与配置**

53.2. Artifacts and configurations

像依赖一样，构建产物是配置出来的。事实上，配置可以同时包括构建产物和依赖。

Like dependencies, artifacts are grouped by configurations. In fact, a configuration can contain both artifacts and dependencies at the same time. 

对于项目的每一个配置，Gradle提供uploadConfigurationName 与 buildConfigurationName任务。这些任务会 build并上传属于各自配置的artifacts。

For each configuration in your project, Gradle provides the tasks uploadConfigurationName and buildConfigurationName. [18] Execution of these tasks will build or upload the artifacts belonging to the respective configuration. 

表22.5,” Java plugin - dependency configurations” 通过Java plugin显示添加的配置，两个配置对artifacts的用法都有重大意义。archives configuration 是将标准配置分配到自己的artifacts 中。Java plugin自动分配默认的jar为这些配置。在51.5“More about project libraries”会介绍更多关于runtime配置，作为依赖，你可以声明许多自定义配置，将artifacts分配给自定义的配置

Table 22.5, “Java plugin - dependency configurations” shows the configurations added by the Java plugin. Two of the configurations are relevant for the usage with artifacts. The archives configuration is the standard configuration to assign your artifacts to. The Java plugin automatically assigns the default jar to this configuration. We will talk more about the runtime configuration in Section 51.5, “More about project libraries”. As with dependencies, you can declare as many custom configurations as you like and assign artifacts to them. 

## **51.3声明artifacts**

51.3. Declaring artifacts

### **51.3.1. Archive task artifacts**

可以使用archive task来定义artifact

You can use an archive task to define an artifact:

例：51.1. 用archive task定义一个artifact

Example 51.1. Defining an artifact using an archive task

build.gradle
```
task myJar(type: Jar)

artifacts {
    archives myJar
}
```

值得注意的是：自定义的archives是构建的一部分不会自动分配到任何配置中，因此必须明确指出具体的配置。

It is important to note that the custom archives you are creating as part of your build are not automatically assigned to any configuration. You have to explicitly do this assignment.

### **51.3.2. File artifacts**

也可以把artifact放到文件中。

You can also use a file to define an artifact:

例51.2 使用文件来存放artifact

Example 51.2. Defining an artifact using a file

build.gradle
```
def someFile = file('build/somefile.txt')

artifacts {
    archives someFile
}
```

Gradle根据文件的名称计算出artifact的属性，可以自己自定义这些属性：

Gradle will figure out the properties of the artifact based on the name of the file. You can customize these properties:

Example 51.3. Customizing an artifact

build.gradle
```
task myTask(type:  MyTaskType) {
    destFile = file('build/somefile.txt')
}

artifacts {
    archives(myTask.destFile) {
        name 'my-artifact'
        type 'text'
        builtBy myTask
    }
}
```

map-based syntax定义使用文件的artifact，map必须包括输入文件，也可以包括其他属性

There is a map-based syntax for defining an artifact using a file. The map must include a file entry that defines the file. The map may include other artifact properties: 

Example 51.4. Map syntax for defining an artifact using a file

build.gradle
```
task generate(type:  MyTaskType) {
    destFile = file('build/somefile.txt')
}

artifacts {
    archives file: generate.destFile, name: 'my-artifact', type: 'text', builtBy: generate
}
```

## **51.4.发布artifacts**

51.4. Publishing artifacts

每一个任务都有一个特定的upload task，在上传之前，必须配置upload task并定义要发布的路径. 您所定义的库不会自动用于上传, 事实上，这些库中的一些只允许下载artifacts，而不是上传。下面是一个示例，您可以配置一个upload task configuration：

We have said that there is a specific upload task for each configuration. Before you can do an upload, you have to configure the upload task and define where to publish the artifacts to. The repositories you have defined (as described in Section 50.6, “Repositories”) are not automatically used for uploading. In fact, some of those repositories only allow downloading artifacts, not uploading. Here is an example of how you can configure the upload task of a configuration: 

Example 51.5. Configuration of the upload task

build.gradle
```
repositories {
    flatDir {
        name "fileRepo"
        dirs "repo"
    }
}

uploadArchives {
    repositories {
        add project.repositories.fileRepo
        ivy {
            credentials {
                username "username"
                password "pw"
            }
            url "http://repo.mycompany.com"
        }
    }
}
```

如上所示，你可以使用一个引用到现在的repository或者创建一个新的repository. 如第50.6.9中“More about Ivy resolvers”描述，你就可以使用所有的Ivy resolver来用于上传。

如果upload repository被定义于多模式的，Gradle必须为每个上传文件选择一个模式。Gradle将会结合可选布书友参数默认上传url parameter定义的模式。如果没有提供url parameter，Gradle将会使用第一个定义的artifactPattern进行上传或者如果设置了ivyPattern，将会使用第一个定义的ivyPattern上传Ivy files
上传Maven repository的相关描述，详见Section 52.6, “Interacting with Maven repositories”

As you can see, you can either use a reference to an existing repository or create a new repository. As described in Section 50.6.9, “More about Ivy resolvers”, you can use all the Ivy resolvers suitable for the purpose of uploading. 

If an upload repository is defined with multiple patterns, Gradle must choose a pattern to use for uploading each file. By default, Gradle will upload to the pattern defined by the url parameter, combined with the optional layout parameter. If no url parameter is supplied, then Gradle will use the first defined artifactPattern for uploading, or the first defined ivyPattern for uploading Ivy files, if this is set. 
Uploading to a Maven repository is described in Section 52.6, “Interacting with Maven repositories”.

## **51.5. More about project libraries**

如果你的项目被用作library，你需要定义这个library的artifacts是什么并且定义这些artifacts的依赖关系。Java plugin为这个目的添了runtime configuration，隐含假设是，即runtime依赖是要发布的artifact的依赖，当然这个是完全可定制的，您可以添加自定义配置，或者让现有的配置从其他配置扩展。你可能有一个不同artifacts 组，其中一组有不同的dependencies设置。这个机制是非常强大和灵活的。

If your project is supposed to be used as a library, you need to define what are the artifacts of this library and what are the dependencies of these artifacts. The Java plugin adds a runtime configuration for this purpose, with the implicit assumption that the runtime dependencies are the dependencies of the artifact you want to publish. Of course this is fully customizable. You can add your own custom configuration or let the existing configurations extend from other configurations. You might have a different group of artifacts which have a different set of dependencies. This mechanism is very powerful and flexible. 

如果有人想使用你的项目作为library，她只需简单声明所依赖的配置。Gradle dependency提供配置属性。如果未指定配置属性，则使用默认配置（详见50.4.9 “Dependency configurations”）,使用你的项目作为一个库，无从是从multi-project build还是通过从存储库中检索项目。在后一种情况下，版本库中的ivy.xml描述符应该包含所有必要的信息。如果你使用Maven库你不工作要有灵活性，如上面所描述的。如何发布Maven repository,详见52.6, “Interacting with Maven repositories”

If someone wants to use your project as a library, she simply needs to declare which configuration of the dependency to depend on. A Gradle dependency offers the configuration property to declare this. If this is not specified, the default configuration is used (see Section 50.4.9, “Dependency configurations”). Using your project as a library can either happen from within a multi-project build or by retrieving your project from a repository. In the latter case, an ivy.xml descriptor in the repository is supposed to contain all the necessary information. If you work with Maven repositories you don't have the flexibility as described above. For how to publish to a Maven repository, see the section Section 52.6, “Interacting with Maven repositories”. 

________________________________________
[18] To be exact, the Base plugin provides those tasks. This plugin is automatically applied if you use the Java plugin.

百度搜索[无线学院](http://wirelesscollege.cn)

