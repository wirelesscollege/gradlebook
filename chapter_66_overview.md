# **第66章 Maven发布**

Chapter 66. Maven Publishing (new)

本章描述的新孵化的Maven发布功能是由“maven-publish”插件提供支持的。最终通过上传任务，新发布支持将取代你的发布。

This chapter describes the new incubating Maven publishing support provided by the “maven-publish” plugin. Eventually this new publishing support will replace publishing via the Upload task.

如果你正在查找原始的maven发布文档去支持使用上传任务，那么请参阅章Chapter 52, Publishing artifacts。
本章描述了如何发布构建产物到一个Apache Maven仓库。模块可以使用理解maven仓库格式的maven、gradle和其他工具发布到Maven仓库,(见依赖管理51章)。

If you are looking for documentation on the original Maven publishing support using the Upload task please see Chapter 52, Publishing artifacts.
This chapter describes how to publish build artifacts to an Apache Maven Repository. A module published to a Maven repository can be consumed by Maven, Gradle (see Chapter 51, Dependency Management) and other tools that understand the Maven repository format.

## **66.1.  “maven-publish” 插件**

66.1. The “maven-publish” Plugin

“maven-publish”插件提供了发布为maven格式的能力。

The ability to publish in the Maven format is provided by the “maven-publish” plugin.

“publishing”插件在项目中创建一个扩展名为“publishing”的PublishingExtension类型。这个扩展提供了一个发布容器以及一个资源仓库。“maven-publish”插件与MavenPublication plublications 以及MavenArtifactRepository仓库协同工作。

The “publishing” plugin creates an extension on the project named “publishing” of type PublishingExtension. This extension provides a container of named publications and a container of named repositories. The “maven-publish” plugin works with MavenPublication publications and MavenArtifactRepository repositories.

Example 66.1. Applying the 'maven-publish' plugin

build.gradle
```
apply plugin: 'maven-publish'
```

应用“maven-publish”插件

Applying the “maven-publish” plugin does the following:

"publish"插件可以自动创建pom。(see Section 66.2, “Publications”).

Applies the “publishing” plugin
Establishes a rule to automatically create a GenerateMavenPom task for each MavenPublication added (see Section 66.2, “Publications”).

自动发布以及自动创建maven仓库。(see Section 66.2, “Publications”)(see Section 66.3, “Repositories”).

Establishes a rule to automatically create a PublishToMavenRepository task for the combination of each MavenPublication added (see Section 66.2, “Publications”), with each MavenArtifactRepository added (see Section 66.3, “Repositories”).

自动发布到maven本地仓库。(seeSection 66.2, “Publications”).

Establishes a rule to automatically create a PublishToMavenLocal task for each MavenPublication added (seeSection 66.2, “Publications”).

## **发布**

66.2. Publications

如果您不熟悉项目产物和配置，您应该阅读52章，52章介绍了这些发布产物的概念。这一章使用不同的方式描述了“发布产物”是什么。这里描述的发布功能最终将会被取代。发布对象描述的结构/配置会被创建。

If you are not familiar with project artifacts and configurations, you should read the Chapter 52, Publishing artifacts that introduces these concepts. This chapter also describes “publishing artifacts” using a different mechanism than what is described in this chapter. The publishing functionality described here will eventually supersede that functionality.
Publication objects describe the structure/configuration of a publication to be created. 

通过任务我们可以把产物发布到仓库,发布对象配置决定了究竟要发布什么。所有项目的发布都是在PublishingExtension.getPublications()容器中定义的。在项目中每个发布都有一个唯一的名字。

Publications are published to repositories via tasks, and the configuration of the publication object determines exactly what is published. All of the publications of a project are defined in the PublishingExtension.getPublications() container. Each publication has a unique name within the project.

如果让“maven-publish”插件产生效果，MavenPublication必须添加到发布集中。这个发布决定了那些在POM文件中定义的附件将会被发布出去。可以通过添加组件，自定义附件以及修改POM文件来决定发布配置。

For the “maven-publish” plugin to have any effect, a MavenPublication must be added to the set of publications. This publication determines which artifacts are actually published as well as the details included in the associated POM file. A publication can be configured by adding components, customizing artifacts, and by modifying the generated POM file directly.

### **66.2.1. 发布组件**

66.2.1. Publishing a Software Component

组件从gradle项目发布到maven仓库其实很简单，我们来看看那些支持组件发布的一些插件：

The simplest way to publish a Gradle project to a Maven repository is to specify a SoftwareComponent to publish. The components presently available for publication are:

Table 66.1. Software Components

|Name	|Provided By	|Artifacts	|Dependencies|
|--
|java	|Chapter 23, The Java Plugin|	Generated jar file	|Dependencies from 'runtime' configuration|
|web	|Chapter 26, The War Plugin|	Generated war file	|No dependencies|

在下面的例子里，演示了附件以及运行时依赖是如何在java组件中配置的

In the following example, artifacts and runtime dependencies are taken from the `java` component, which is added by the Java Plugin.

Example 66.2. Adding a MavenPublication for a Java component

build.gradle

```
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
}
```

### **66.2.2 发布自定义产物**

66.2.2. Publishing custom artifacts

可以在发布中显式的配置产物。附件通常为原始文件,或者为AbstractArchiveTask实例(例如Jar,Zip)。

It is also possible to explicitly configure artifacts to be included in the publication. Artifacts are commonly supplied as raw files, or as instances of AbstractArchiveTask (e.g. Jar, Zip).

对于每一个定制的产物,可以指定要使用的扩展和类型来发布。注意,构建产物可以为空，但是必须要有扩展名。

For each custom artifact, it is possible to specify the extension and classifier values to use for publication. Note that only one of the published artifacts can have an empty classifier, and all other artifacts must have a unique classifier/extension combination.

可以按照如下来配置自定义产物：

Configure custom artifacts as follows:

Example 66.3. Adding additional artifact to a MavenPublication

build.gradle

```
task sourceJar(type: Jar) {
    from sourceSets.main.allJava
}

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java

            artifact sourceJar {
                classifier "sources"
            }
        }
    }
}
```

如果想了解更多关于如何自定义产物可以查看MavenPublicationAPI文档。

See the MavenPublication class in the API documentation for more information about how artifacts can be customized.

### **66.2.3. 在POM文件里标注值**

66.2.3. Identity values in the generated POM

生成的POM文件的属性将包含来源于以下项目属性的标注值:

The attributes of the generated POM file will contain identity values derived from the following project properties:

```
groupId - Project.getGroup()
artifactId - Project.getName()
version - Project.getVersion()
```

覆盖默认值很简单:当配置MavenPublication artifactId或版本属性，简单地指定groupId即可。

Overriding the default identity values is easy: simply specify the groupId, artifactId or version attributes when configuring the MavenPublication.

Example 66.4. customizing the publication identity

build.gradle

```
publishing {
    publications {
        maven(MavenPublication) {
            groupId 'org.gradle.sample'
            artifactId 'project1-sample'
            version '1.1'

            from components.java
        }
    }
}
```

某些仓库无法处理所有的字符。例如,当在Windows上发布filesystem-backed库时，':'字符不能用作标识符。

Certain repositories will not be able to handle all supported characters. For example, the ':' character cannot be used as an identifier when publishing to a filesystem-backed repository on Windows.

Maven限制‘groupId’和‘artifactId必须在有限字符集([A-Za-z0-9_ \ \ -。]+)内，Gradle与maven一样实施这一限制。“版本”(与“扩展”和“分类”一样),Gradle将处理任何有效的Unicode字符。

Maven restricts 'groupId' and 'artifactId' to a limited character set ([A-Za-z0-9_\\-.]+) and Gradle enforces this restriction. For 'version' (as well as artifact 'extension' and 'classifier'), Gradle will handle any valid Unicode character.

唯有Unicode值明确禁止“\”,“/”和任何ISO控制字符。提供的值也是在早期发布时已验证的。

The only Unicode values that are explicitly prohibited are '\', '/' and any ISO control character. Supplied values are validated early in publication.

## **66.2.4. 修改生成的POM文件**

66.2.4. Modifying the generated POM

生成的POM文件需要在发布前调整，“maven-publish”提供了hook极致允许如下的修改。

The generated POM file may need to be tweaked before publishing. The “maven-publish” plugin provides a hook to allow such modification.

Example 66.5. Modifying the POM file

build.gradle

```
publications {
    mavenCustom(MavenPublication) {
        pom.withXml {
            asNode().appendNode('description',
                                'A demonstration of maven POM customization')
        }
    }
}
```

这个例子我们为POM添加了‘description’元素，按照这种hook方式我们在修改任何POM文件。例如，你能在构建过程中替换版本号。

In this example we are adding a 'description' element for the generated POM. With this hook, you can modify any aspect of the POM. For example, you could replace the version range for a dependency with the actual version used to produce the build.

更多信息请查看API文档中的MavenPom.withXml()方法

See MavenPom.withXml() in the API documentation for more information.

你可以修改并创建任意的POM文件。这意味这你可以修改一个无效的POM文件，所以你在使用的时候一定要注意这点。

It is possible to modify virtually any aspect of the created POM should you need to. This means that it is also possible to modify the POM in such a way that it is no longer a valid Maven Pom, so care must be taken when using this feature.

(groupId, artifactId, version)这些标示符是一个例外，在POM文件里这些值不能使用withXML来修改。

The identifier (groupId, artifactId, version) of the published module is an exception; these values cannot be modified in the POM using the `withXML` hook.

### **66.2.5. 发布多样化模块**

66.2.5. Publishing multiple modules

有时从我们Gradle构建中发布多样化模块相当有用，不用去创建单独的Gradle子工程。下面是一个发布独立API和Jar包的例子：

Sometimes it's useful to publish multiple modules from your Gradle build, without creating a separate Gradle subproject. An example is publishing a separate API and implementation jar for your library. With Gradle this is simple:

Example 66.6. Publishing multiple modules from a single project

build.gradle

```
task apiJar(type: Jar) {
    baseName "publishing-api"
    from sourceSets.main.output
    exclude '**/impl/**'
}

publishing {
    publications {
        impl(MavenPublication) {
            groupId 'org.gradle.sample.impl'
            artifactId 'project2-impl'
            version '2.3'

            from components.java
        }
        api(MavenPublication) {
            groupId 'org.gradle.sample'
            artifactId 'project2-api'
            version '2'

            artifact apiJar
        }
    }
}
```
如果一个项目定义了多个发布那么Gradle将发布他们到已定义的仓库。如上所述每个发布必须被给予一个唯一的身份。

If a project defines multiple publications then Gradle will publish each of these to the defined repositories. Each publication must be given a unique identity as described above.

## **66.3. 仓库**

66.3. Repositories

发布产物将会被发布到仓库中，发布仓库通过PublishingExtension.getRepositories()容器定义。

Publications are published to repositories. The repositories to publish to are defined by the PublishingExtension.getRepositories() container.

Example 66.7. Declaring repositories to publish to

build.gradle

```
publishing {
    repositories {
        maven {
            // change to point to your repo, e.g. http://my.org/repo
            url "$buildDir/repo"
        }
    }
}
```

DSL描述发布仓库的依赖性,RepositoryHandler。然而,仅有MavenArtifactRepository仓库可以用于发布。

The DSL used to declare repositories for publication is the same DSL that is used to declare repositories to consume dependencies from, RepositoryHandler. However, in the context of Maven publication only MavenArtifactRepository repositories can be used for publication.

## **66.4. 执行发布**

66.4. Performing a publish

maven-publish插件为在 publishing.publications与publishing中每个MavenPublication与MavenArtifactRepository的结合自动创建PublishToMavenRepository任务。

The “maven-publish” plugin automatically creates a PublishToMavenRepository task for each MavenPublication and MavenArtifactRepository combination in the publishing.publications and publishing.repositories containers respectively.

被创建的任务命名为“publish«PUBNAME»PublicationTo«REPONAME»Repository”

The created task is named “publish«PUBNAME»PublicationTo«REPONAME»Repository”.

Example 66.8. Publishing a project to a Maven repository

build.gradle

```
apply plugin: 'java'
apply plugin: 'maven-publish'

group = 'org.gradle.sample'
version = '1.0'

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
}
publishing {
    repositories {
        maven {
            // change to point to your repo, e.g. http://my.org/repo
            url "$buildDir/repo"
        }
    }
}

```

Output of gradle publish

```
> gradle publish
:generatePomFileForMavenJavaPublication
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:publishMavenJavaPublicationToMavenRepository
:publish

BUILD SUCCESSFUL

Total time: 1 secs
```
在这个例子里：“publishMavenJavaPublicationToMavenRepository”任务被创建，类型为PublishToMavenRepository。该任务会连接到发布生命周期中。执行“gradle publish” 构建POM文件，所有的构建产物将会发布，把他们传到仓库中去。

In this example, a task named “publishMavenJavaPublicationToMavenRepository” is created, which is of type PublishToMavenRepository. This task is wired into the publish lifecycle task. Executing “gradle publish” builds the POM file and all of the artifacts to be published, and transfers them to the repository.

## **66.5. 发布到Maven本地**

66.5. Publishing to Maven Local

为了与本地maven集成，发布模块到本地就比较有用了。在maven的规则里，所有的模块都是被安装进去得，maven-publish插件将自动创建PublishToMavenLocal任务为每个在publishing.publications容器内的MavenPublication。这些任务与publishToMavenLocal的生命周期相关联。你不需要在publishing.repositories里配置mavenLocal。

For integration with a local Maven installation, it is sometimes useful to publish the module into the local .m2 repository. In Maven parlance, this is referred to as 'installing' the module. The “maven-publish” plugin makes this easy to do by automatically creating a PublishToMavenLocal task for each MavenPublication in the publishing.publications container. Each of these tasks is wired into the publishToMavenLocal lifecycle task. You do not need to have `mavenLocal` in your `publishing.repositories` section.

被创建的任务将被命名为“publish«PUBNAME»PublicationToMavenLocal”.

The created task is named “publish«PUBNAME»PublicationToMavenLocal”.

Example 66.9. Publish a project to the Maven local repository

Output of gradle publishToMavenLocal

```
> gradle publishToMavenLocal
:generatePomFileForMavenJavaPublication
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:publishMavenJavaPublicationToMavenLocal
:publishToMavenLocal

BUILD SUCCESSFUL

Total time: 1 secs
```
例子里得任务结果命名为publishMavenJavaPublicationToMavenLocal，任务与publishToMavenLocal生命周期联系。执行gradle publishToMavenLocal构建POM文件，所有的产物都会被发布，也会安装到maven本地仓库中。

The resulting task in this example is named “publishMavenJavaPublicationToMavenLocal”. This task is wired into the publishToMavenLocal lifecycle task. Executing “gradle publishToMavenLocal” builds the POM file and all of the artifacts to be published, and “installs” them into the local Maven repository.

## **66.6. 不发布生成POM文件**

66.6. Generating the POM file without publishing

有时不需要实际发布而未一个模块生成POM文件很有必要，自从POM可以被单独的任务生成后，这样所就变得相当的简单。

At times it is useful to generate a Maven POM file for a module without actually publishing. Since POM generation is performed by a separate task, it is very easy to do so.

生成POM的任务是GenerateMavenPom类型，基于发布 “generatePomFileFor«PUBNAME»Publication”的一个名字。所以在下面的例子里，发布被命名为“mavenCustom”，任务被命名为“generatePomFileForMavenCustomPublication”.

The task for generating the POM file is of type GenerateMavenPom, and it is given a name based on the name of the publication: “generatePomFileFor«PUBNAME»Publication”. So in the example below, where the publication is named “mavenCustom”, the task will be named “generatePomFileForMavenCustomPublication”.

Example 66.10. Generate a POM file without publishing

build.gradle
```
model {
    tasks.generatePomFileForMavenCustomPublication {
        destination = file("$buildDir/generated-pom.xml")
    }
}
```

Output of gradle generatePomFileForMavenCustomPublication

```
> gradle generatePomFileForMavenCustomPublication
:generatePomFileForMavenCustomPublication

BUILD SUCCESSFUL

Total time: 1 secs
```
所有发布模式的细节都在考虑范围内,包括组件的,定制的构件,以及pom.withXml下的任何修改。

All details of the publishing model are still considered in POM generation, including components`, custom artifacts, and any modifications made via pom.withXml.

“maven-publish”插件为了后期插件的配置做一些实验,与任何GenerateMavenPom任务才能构建出版扩展配置。最简单的方式,当您试图访问GenerateMavenPom任务时，以确保发布插件配置是在一个block里访问,就好像上面的例子一样。

The “maven-publish” plugin leverages some experimental support for late plugin configuration, and any GenerateMavenPom tasks will not be constructed until the publishing extension is configured. The simplest way to ensure that the publishing plugin is configured when you attempt to access the GenerateMavenPom task is to place the access inside a model block, as the example above demonstrates.

这同样适用于任何试图访问publication-specific任务，例如PublishToMavenRepository。这些任务应该一个block中被引用。

The same applies to any attempt to access publication-specific tasks like PublishToMavenRepository. These tasks should be referenced from within a model block.

百度搜索[无线学院](http://wirelesscollege.cn)
