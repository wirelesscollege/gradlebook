
# **Chapter 67. Ivy Publishing (new)**

Chapter 67. Ivy 发布

This chapter describes the new incubating Ivy publishing support provided by the “ivy-publish” plugin. Eventually this new publishing support will replace publishing via the Upload task.

本章描述了“ivy-publish” 插件，该插件将替代Upload任务。

If you are looking for documentation on the original Ivy publishing support using the Upload task please see Chapter 53, Publishing artifacts.

如果你正在查看Ivy的相关文档，请查看53章，发布构建产物。

This chapter describes how to publish build artifacts in the Apache Ivy format, usually to a repository for consumption by other builds or projects. What is published is one or more artifacts created by the build, and an Ivy module descriptor (normally ivy.xml) that describes the artifacts and the dependencies of the artifacts, if any.

这章描述了如何以Apache Ivy格式发布构建产物，通常发布到其他构建或项目消耗的存储库中。发布的内容是一个或多个构建产物，以及一个Ivy组件描述文件（通常是ivy.xml），它描述产物和产物依赖，如果存在的话。

A published Ivy module can be consumed by Gradle (see Chapter 51, Dependency Management) and other tools that understand the Ivy format.

一个发布的Ivy组件可以被Gradle（看52章依赖管理）和其他理解Ivy格式的工具使用。

## **67.1. The “ivy-publish” Plugin**

67.1.“ivy-publish”插件

The ability to publish in the Ivy format is provided by the “ivy-publish” plugin.

用Ivy格式发布的功能是由插件“ivy-publish” 提供。

The “publishing” plugin creates an extension on the project named “publishing” of type PublishingExtension. This extension provides a container of named publications and a container of named repositories. The “ivy-publish” plugin works with IvyPublication publications and IvyArtifactRepository repositories.

“publishing” 插件在项目上创建一个类型为PublishingExtension命名为“publishing”的扩展。这个扩展提供了一个名字是publications和名字是repositories的容器。“ivy-publish” 插件与IvyPublication publications和IvyArtifactRepository repositories一块工作。

Example 67.1. Applying the “ivy-publish” plugin

例67.1. 应用“ivy-publish”插件

build.gradle
```
apply plugin: 'ivy-publish'
```

Applying the “ivy-publish” plugin does the following:

应用“ivy-publish”插件做到以下这些：

Applies the “publishing” plugin

应用“publishing”插件

Establishes a rule to automatically create a GenerateIvyDescriptor task for each IvyPublication added (see Section 65.2, “Publications”).

建立一个规则来自动为每个添加的IvyPublication创建一个GenerateIvyDescriptor任务(参见65.2节,“发布”)。

Establishes a rule to automatically create a PublishToIvyRepository task for the combination of each IvyPublication added (see Section 65.2, “Publications”), with each IvyArtifactRepository added (see Section 
65.3, “Repositories”).

建立一个规则来自动为每个IvyPublication添加的组合创建一个PublishToIvyRepository任务（参见65.3节,“存储库”)。

## **67.2. Publications**

67.2.发布

If you are not familiar with project artifacts and configurations, you should read Chapter 52, Publishing artifacts, which introduces these concepts. This chapter also describes “publishing artifacts” using a different mechanism than what is described in this chapter. The publishing functionality described here will eventually supersede that functionality.

Publication objects describe the structure/configuration of a publication to be created. Publications are published to repositories via tasks, and the configuration of the publication object determines exactly what is published. All of the publications of a project are defined in the PublishingExtension.getPublications() container. Each publication has a unique name within the project.

如果您不熟悉项目构建产物和配置,你应该阅读51章,发布构建产物,这里介绍这些概念。这一章还增加描述了使用不同的机制“发布工件”。这里描述的发布功能将最终取代那种功能。

发布对象描述了要创建发布的结构/配置。发布品通过任务发布到存储库,发布的配置对象决定究竟发布什么。项目中所有的发布品由PublishingExtension.getPublications()容器决定。在项目中每个发布品都有一个唯一的名称。

For the “ivy-publish” plugin to have any effect, an IvyPublication must be added to the set of publications. This publication determines which artifacts are actually published as well as the details included in the associated Ivy module descriptor file. A publication can be configured by adding components, customizing artifacts, and by modifying the generated module descriptor file directly.

要使“ivy-publish”插件有所作用,必须添加一个IvyPublication到发布集。这个发布决定实际要发布哪些产物以及包含在相关Ivy组件描述文件中的详细信息。一次发布可以通过添加组件,定制产物,以及直接修改生成的组件描述符文件来配置。

## **67.2.1. Publishing a Software Component**

The simplest way to publish a Gradle project to an Ivy repository is to specify a SoftwareComponent to publish. The components presently available for publication are:

发布Gradle项目到一个Ivy存储库最简单的方式是指明一个要发布的SoftwareComponent，当前支持发布的的构成有：

Table 67.1. Software Components

|Name	|Provided By	|Artifacts	|Dependencies|
|--
|java	|Java Plugin	|Generated jar file|	Dependencies from 'runtime' configuration|
|web	|War Plugin	|Generated war file	|No dependencies|

In the following example, artifacts and runtime dependencies are taken from the `java` component, which is added by the Java Plugin.

下面的例子中，构建产物和运行时依赖从‘java’构成中获取，而‘java’构成是由Java Plugin插件添加的。

Example 67.2. Publishing a Java module to Ivy

build.gradle
```
publications {
    ivyJava(IvyPublication) {
        from components.java
    }
}
```

### **67.2.2. Publishing custom artifacts**

67.2.2. 发布自定义产物

It is also possible to explicitly configure artifacts to be included in the publication. Artifacts are commonly supplied as raw files, or as instances of AbstractArchiveTask (e.g. Jar, Zip).

我们也可以明确地配置发布的产物。产物通常是作为源文件，或者作为AbstractArchiveTasks的实例提供 (比如 Jar, Zip).

For each custom artifact, it is possible to specify the name, extension, type, classifier and conf values to use for publication. Note that each artifacts must have a unique name/classifier/extension combination.
Configure custom artifacts as follows:

对于每个自定义的产物，可以指定发布的name, extension, type, classifier 和 conf的值，要注意每个产物必须有唯一的name/classifier/extension组合。

配置自定义产物如下：

Example 67.3. Publishing additional artifact to Ivy

build.gradle
```
task sourceJar(type: Jar) {
    from sourceSets.main.java
    classifier "source"
}
publishing {
    publications {
        ivy(IvyPublication) {
            from components.java
            artifact(sourceJar) {
                type "source"
                conf "runtime"
            }
        }
    }
}
```

See the IvyPublication class in the API documentation for more detailed information on how artifacts can be customized.

查看API文档中IvyPublication类可以了解关于产物如何被自定义的详细信息。

## **67.2.3. Identity values for the published project**

67.2.3.  已发布项目的Identity值

The generated Ivy module descriptor file contains an <info> element that identifies the module. The default identity values are derived from the following:

生成的Ivy组件descriptor文件包含一个可以识别组件的<info>元素，这个默认的identity值来源如下：

organisation - Project.getGroup()
module - Project.getName()
revision - Project.getVersion()
status - Project.getStatus()
branch - (not set)

Overriding the default identity values is easy: simply specify the organisation, module or revision attributes when configuring the IvyPublication. The status and branch attributes can be set via the descriptor property (see IvyModuleDescriptorSpec). The descriptor property can also be used to add additional custom elements as children of the <info> element.

覆盖默认的identity值很简单：仅需在配置IvyPublication指定organisation, module 或 revision的属性。status和branch属性可以通过descriptor属性来设置。descriptor属性也可以用于添加其他的自定义元素，比如 <info>元素的子元素。

Example 67.4. customizing the publication identity

例67.4. 自定义发布identity

build.gradle
```
publishing {
    publications {
        ivy(IvyPublication) {
            organisation 'org.gradle.sample'
            module 'project1-sample'
            revision '1.1'
            descriptor.status = 'milestone'
            descriptor.branch = 'testing'
            descriptor.extraInfo 'http://my.namespace', 'myElement', 'Some value'

            from components.java
        }
    }
}
```

Certain repositories are not able to handle all supported characters. For example, the ':' character cannot be used as an identifier when publishing to a filesystem-backed repository on Windows.

某些repositories不能处理所有支持的字符。例如，':' 字符在Windows上的系统备份文件存储库中就不能作为标识符。

Gradle will handle any valid Unicode character for organisation, module and revision (as well as artifact name, extension and classifier). The only values that are explicitly prohibited are '\', '/' and any ISO control character. The supplied values are validated early during publication.

Gradle会处理在organisation,module和revision (以及产物的name, extension 和classifier)中任何有效的Unicode字符。唯一被明确禁止的是'\', '/'和任何ISO控制字符。所有提供的值会在发布前期校验。

## **67.2.4. Modifying the generated module descriptor**

67.2.4. 修改生成的组件描述符文件

At times, the module descriptor file generated from the project information will need to be tweaked before publishing. The “ivy-publish” plugin provides a hook to allow such modification.

有时可能需要在发布前调整由项目信息生成的组件描述符文件。“ivy-publish”插件就提供了一个允许修改的hook。

Example 67.5. Customizing the module descriptor file

例67.5.  自定义组件descriptor文件

build.gradle
```
publications {
    ivyCustom(IvyPublication) {
        descriptor.withXml {
            asNode().info[0].appendNode('description',
                                        'A demonstration of ivy descriptor customization')
        }
    }
}
```

In this example we are simply adding a 'description' element to the generated Ivy dependency descriptor, but this hook allows you to modify any aspect of the generated descriptor. For example, you could replace the version range for a dependency with the actual version used to produce the build.

在这个例子中我们仅仅添加一个元素'description'到生成的Ivy依赖descriptor中。但是这个hook允许你修改已生成descriptor的任何方面。例如，你可以替换版本范围为一个相关的实际版本来生成构建。

See IvyModuleDescriptorSpec.withXml() in the API documentation for more information.
可查看API文档中的IvyModuleDescriptorSpec.withXml()了解更多信息

It is possible to modify virtually any aspect of the created descriptor should you need to. This means that it is also possible to modify the descriptor in such a way that it is no longer a valid Ivy module descriptor, so care must be taken when using this feature.

实际上只要需要它可以用来修改已创建的descriptor的任何方面。这意味着它也可以修改descriptor导致它不再是个有效的组件描述符，所以采取这一功能时必须要小心。

The identifier (organisation, module, revision) of the published module is an exception; these values cannot be modified in the descriptor using the `withXML` hook.

已发布的组件标识符 (organisation, module, revision)是个特例；这个值在descriptor中不能使用 `withXML`修改。

## **67.2.5. Publishing multiple modules***

67.2.5. 发布多个组件

Sometimes it's useful to publish multiple modules from your Gradle build, without creating a separate Gradle subproject. An example is publishing a separate API and implementation jar for your library. With Gradle this is simple:

有时候在Gradle构建时发布多个组件是很有用的，而不需创建一个单独的Gradle子项目。有个例子是 为你的库发布一个单独的API和实现jar。使用Gradle很简单：

Example 67.6. Publishing multiple modules from a single project

build.gradle
```
task apiJar(type: Jar) {
    baseName "publishing-api"
    from sourceSets.main.output
    exclude '**/impl/**'
}
publishing {
    publications {
        impl(IvyPublication) {
            organisation 'org.gradle.sample.impl'
            module 'project2-impl'
            revision '2.3'

            from components.java
        }
        api(IvyPublication) {
            organisation 'org.gradle.sample'
            module 'project2-api'
            revision '2'
        }
    }
}
```

If a project defines multiple publications then Gradle will publish each of these to the defined repositories. Each publication must be given a unique identity as described above.

如果一个项目定义了多个发布产品那么Gradle将发布每个产品到定义的库中。每个发布品必须要指明一个唯一的标识符，如上面描述的那样。

## **67.3. Repositories**

Publications are published to repositories. The repositories to publish to are defined by the PublishingExtension.getRepositories() container.

发布品被发布到存储库中。要发布的库是由PublishingExtension.getRepositories()容器定义的。
Example 67.7. Declaring repositories to publish to

build.gradle
```
repositories {
    ivy {
        // change to point to your repo, e.g. http://my.org/repo
        url "$buildDir/repo"
    }
}
```

The DSL used to declare repositories for publishing is the same DSL that is used to declare repositories for dependencies (RepositoryHandler). However, in the context of Ivy publication only the repositories created by the ivy() methods can be used as publication destinations. You cannot publish an IvyPublication to a Maven repository for example.

用来声明发布库的DSL和用来声明依赖 (RepositoryHandler)的库是一样的。然而，在Ivy发布内容里只有被ivy()方法创建的库才会被用作发布目标库。例如你不能发布一个IvyPublication到一个Maven库中。

## **67.4. Performing a publish**

The “ivy-publish” plugin automatically creates a PublishToIvyRepository task for each IvyPublication and IvyArtifactRepository combination in the publishing.publications and publishing.repositories containers respectively.

“ivy-publish”插件自动的为每个在publishing.publications 和 publishing.repositories容器中的IvyPublication 和IvyArtifactRepository的结合创建了一个PublishToIvyRepository任务。

The created task is named “publish«PUBNAME»PublicationTo«REPONAME»Repository”, which is “publishIvyJavaPublicationToIvyRepository” for this example. This task is of type PublishToIvyRepository.

创建的任务名字为“publish«PUBNAME»PublicationTo«REPONAME»Repository”，对于这个例子也就是“publishIvyJavaPublicationToIvyRepository”。这个任务是PublishToIvyRepository类型。

Example 67.8. Choosing a particular publication to publish

build.gradle
```
apply plugin: 'java'
apply plugin: 'ivy-publish'

group = 'org.gradle.sample'
version = '1.0'

publishing {
    publications {
        ivyJava(IvyPublication) {
            from components.java
        }
    }
    repositories {
        ivy {
            // change to point to your repo, e.g. http://my.org/repo
            url "$buildDir/repo"
        }
    }
}
```

Output of gradle publishIvyJavaPublicationToIvyRepository
```
> gradle publishIvyJavaPublicationToIvyRepository
:generateDescriptorFileForIvyJavaPublication
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar
:publishIvyJavaPublicationToIvyRepository

BUILD SUCCESSFUL

Total time: 1 secs
```

### **67.4.1. The “publish” lifecycle task**

The “publish” plugin (that the “ivy-publish” plugin implicitly applies) adds a lifecycle task that can be used to publish all publications to all applicable repositories named “publish”.

“publish” 插件（“ivy-publish”隐含使用的）添加了一个声明周期任务，使用它可以发布所有发布产物到所有名字为“publish”的库中。
In more concrete terms, executing this task will execute all PublishToIvyRepository tasks in the project. This is usually the most convenient way to perform a publish.

更具体而言，执行这项任务将会执行项目中的所有PublishToIvyRepository任务。通常这个是执行发布最方便的方式。

Example 67.9. Publishing all publications via the “publish” lifecycle task

Output of gradle publish
```
> gradle publish
:generateDescriptorFileForIvyJavaPublication
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar
:publishIvyJavaPublicationToIvyRepository
:publish

BUILD SUCCESSFUL

Total time: 1 secs
```

## **67.5. Generating the Ivy module descriptor file without publishing**

67.5.不用发布生成Ivy组件描述文件

At times it is useful to generate the Ivy module descriptor file (normally ivy.xml) without publishing your module to an Ivy repository. Since descriptor file generation is performed by a separate task, this is very easy to do.

有时候在没有发布组件到Ivy库时生成Ivy组件描述文件（通常是ivy.xml）很有用，因为描述文件生成是由一个单独的任务执行的，这很容易做到。

The “ivy-publish” plugin creates one GenerateIvyDescriptor task for each registered IvyPublication, named “generateDescriptorFileFor«PUBNAME»Publication”, which will be “generateDescriptorFileForIvyJavaPublication” for the previous example of the “ivyJava” publication.

“ivy-publish” 插件为每个注册的IvyPublication创建了一个GenerateIvyDescriptor任务，名字类如“generateDescriptorFileFor«PUBNAME»Publication”, 对于之前“ivyJava” 的例子它的名字会是“generateDescriptorFileForIvyJavaPublication”
You can specify where the generated Ivy file will be located by setting the destination property on the generated task. By default this file is written to “build/publications/«PUBNAME»/ivy.xml”.

你可以通过设置任务中的目的属性指明Ivy文件的生成路径。默认情况文件会被写入到“build/publications/«PUBNAME»/ivy.xml”。

Example 67.10. Generating the Ivy module descriptor file

build.gradle
```
model {
    tasks.generateDescriptorFileForIvyCustomPublication {
        destination = file("$buildDir/generated-ivy.xml")
    }
}
Output of gradle generateDescriptorFileForIvyCustomPublication
> gradle generateDescriptorFileForIvyCustomPublication
:generateDescriptorFileForIvyCustomPublication

BUILD SUCCESSFUL

Total time: 1 secs
```

The “ivy-publish” plugin leverages some experimental support for late plugin configuration, and the GenerateIvyDescriptor task will not be constructed until the publishing extension is configured. The simplest way to ensure that the publishing plugin is configured when you attempt to access the GenerateIvyDescriptor task is to place the access inside a model block, as the example above demonstrates.

“ivy-publish”插件利用一些实验支持插件配置，GenerateIvyDescriptor任务不会构建直到发布扩展被配置好。当你试图访问GenerateIvyDescriptor任务时候，确认发布插件配置好的最简单方法是将访问放在一个模型块中，就像上面例子演示的一样。

The same applies to any attempt to access publication-specific tasks like PublishToIvyRepository. These tasks should be referenced from within a model block.

同样适用于任何像PublishToIvyRepository一样企图访问publication-specific的任务，这些任务应在模型块中被引用。

## **67.6. Complete example**

The following example demonstrates publishing with a multi-project build. Each project publishes a Java component and a configured additional source artifact. The descriptor file is customized to include the project description for each project.

下面的例子演示了多项目构建的演示发布。每个项目发布了一个java组件和一个配置好的另外的源产物。项目描述文件中包含了每个项目的项目描述。

Example 67.11. Publishing a Java module

build.gradle
```
subprojects {
    apply plugin: 'java'
    apply plugin: 'ivy-publish'

    version = '1.0'
    group = 'org.gradle.sample'

    repositories {
        mavenCentral()
    }
    task sourceJar(type: Jar) {
        from sourceSets.main.java
        classifier "source"
    }
}

project(":project1") {
    description = "The first project"

    dependencies {
       compile 'junit:junit:4.12', project(':project2')
    }
}

project(":project2") {
    description = "The second project"

    dependencies {
       compile 'commons-collections:commons-collections:3.1'
    }
}

subprojects {
    publishing {
        repositories {
            ivy {
                // change to point to your repo, e.g. http://my.org/repo
                url "${rootProject.buildDir}/repo"
            }
        }
        publications {
            ivy(IvyPublication) {
                from components.java
                artifact(sourceJar) {
                    type "source"
                    conf "runtime"
                }
                descriptor.withXml {
                    asNode().info[0].appendNode('description', description)
                }
            }
        }
    }
}
```

The result is that the following artifacts will be published for each project:

结果是每个项目将会发布以下的产物

The Ivy module descriptor file: “ivy-1.0.xml”.

Ivy 组件描述文件：“ivy-1.0.xml”

The primary “jar” artifact for the Java component: “project1-1.0.jar”.

java组件主要的jar产物: “project1-1.0.jar”.

The source “jar” artifact that has been explicitly configured: “project1-1.0-source.jar”.

明确配置的源码jar产物： “project1-1.0-source.jar”.

When project1 is published, the module descriptor (i.e. the ivy.xml file) that is produced will look like:

当项目发布后，组件描述（比如the ivy.xml文件）生成像这样：

Note that «PUBLICATION-TIME-STAMP» in this example Ivy module descriptor will be the timestamp of when the descriptor was generated.

Example 67.12. Example generated ivy.xml

output-ivy.xml

```

<?xml version="1.0" encoding="UTF-8"?>
<ivy-module version="2.0">
  <info organisation="org.gradle.sample" module="project1" revision="1.0" status="integration" publication="«PUBLICATION-TIME-STAMP»">
    <description>The first project</description>
  </info>
  <configurations>
    <conf name="default" visibility="public" extends="runtime"/>
    <conf name="runtime" visibility="public"/>
  </configurations>
  <publications>
    <artifact name="project1" type="jar" ext="jar" conf="runtime"/>
    <artifact name="project1" type="source" ext="jar" conf="runtime" m:classifier="source" xmlns:m="http://ant.apache.org/ivy/maven"/>
  </publications>
  <dependencies>
    <dependency org="junit" name="junit" rev="4.12" conf="runtime-&gt;default"/>
    <dependency org="org.gradle.sample" name="project2" rev="1.0" conf="runtime-&gt;default"/>
  </dependencies>
</ivy-module>

```

## **67.7. Future features**

The “ivy-publish” plugin functionality as described above is incomplete, as the feature is still incubating. In upcoming Gradle releases, the functionality will be expanded to include (but not limited to):

上面所述的“ivy-publish”插件功能上是不完整的，因为还在开发优化中。在未来的Gradle版本中，功能会扩展包含（但不限制在）：
Convenient customization of module attributes (module, organisation etc.)

Convenient customization of dependencies reported in module descriptor.
Multiple discrete publications per project

模块属性（模块，组织等）方便的定制

模块描述符中依赖的方便定制。

每个项目中发布多个离散发布品
