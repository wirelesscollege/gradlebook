# **第52章.依赖管理**

Chapter 52. Dependency Management

# **52.1.介绍**

52.1. Introduction

依赖管理是每次构建的一个关键功能,并且Gradle强调提供一类既易于理解又能兼容多种方法的依赖管理.如果你熟练使用Maven或Ivy的方法,你会很高兴地得知,Gradle除完全兼容这两种方法之外还支持足够灵活的完全自定义方法.

Dependency management is a critical feature of every build, and Gradle has placed an emphasis on offering first-class dependency management that is both easy to understand and compatible with a wide variety of approaches. If you are familiar with the approach used by either Maven or Ivy you will be delighted to learn that Gradle is fully compatible with both approaches in addition to being flexible enough to support fully-customized approaches.

这里主要强调Gradle支持的依赖管理：

Here are the major highlights of Gradle's support for dependency management:

传递依赖管理：Gradle让你完全控制你项目的依赖关系树。

Transitive dependency management: Gradle gives you full control of your project's dependency tree.

支持不受管理的依赖：如果你的依赖只是在版本控制或者共享驱动器文件,Gradle提供了强大的功能以支持这一点.

Support for non-managed dependencies: If your dependencies are simply files in version control or a shared drive, Gradle provides powerful functionality to support this.

支持自定义依赖项的定义：Gradle的模块依赖让你在构建脚本中具备描述依赖层次结构的能力.

Support for custom dependency definitions: Gradle's Module Dependencies give you the ability to describe the dependency hierarchy in the build script.

完全可定制的方式来解决依赖：Gradle为你提供定制解决依赖的能力使依赖替代更容易.

A fully customizable approach to Dependency Resolution: Gradle provides you with the ability to customize resolution rules making dependency substitution easy.

完全兼容Maven和Ivy：如果你有在一个Maven POM或Ivy文件中定义的依赖关系,Gradle提供了一系列流行的构建工具与之无缝集成.

Full Compatibility with Maven and Ivy: If you have defined dependencies in a Maven POM or an Ivy file, Gradle provides seamless integration with a range of popular build tools.

与现有的依赖管理基础实施集成：Gradle能够兼容Maven和Ivy库.如果你使用Archiva,Nexus,或Artifactory,Gradle能100％兼容所有格式的库.

Integration with existing dependency management infrastructure: Gradle is compatible with both Maven and Ivy repositories. If you use Archiva, Nexus, or Artifactory, Gradle is 100% compatible with all repository formats.

成千上万的相互依赖的开源组件都有一系列的版本不兼容,依赖管理造成的问题的建立越来越复杂的习惯.当构建的依赖关系树变得笨拙,构建工具应该不会强迫你采用单一,不灵活的方法来依赖管理.一个适当的构建系统必须设计灵活,和Gradle可以处理任何情况.

With hundreds of thousands of interdependent open source components each with a range of versions and incompatibilities, dependency management has a habit of causing problems as builds grow in complexity. When a build's dependency tree becomes unwieldy, your build tool shouldn't force you to adopt a single, inflexible approach to dependency management. A proper build system has to be designed to be flexible, and Gradle can handle any situation.

### **52.1.1.灵活的依赖管理的迁移**

52.1.1. Flexible dependency management for migrations

依赖管理能将一个构建系统迁移到另一个上.如果你是从像Ant或Maven的工具来迁移Gradle,你可能会面临一些困难的情况.例如,一种常用的模式是与存储在文件系统版本少jar文件的Ant项目.其他构建系统需要更换批发这种方法的迁移之前.随着Gradle,你能适应新的构建,以相关性或依赖性的元数据中的任何现有的来源.这使得以Gradle增量迁移比其他容易得多.在大多数大型项目,打造迁移和任何变化发展过程是渐进,因为大多数企业不能停止一切并迁移到依赖管理的构建工具的想法.

Dependency management can be particularly challenging during a migration from one build system to another. If you are migrating from a tool like Ant or Maven to Gradle, you may be faced with some difficult situations. For example, one common pattern is an Ant project with version-less jar files stored in the filesystem. Other build systems require a whhas no concept ofolesale replacement of this approach before migrating. With Gradle, you can adapt your new build to any existing source of dependencies or dependency metadata. This makes incremental migration to Gradle much easier than the alternative. On most large projects, build migrations and any change to development process is incremental because most organizations can't afford to stop everything and migrate to a build tool's idea of dependency management.

即使你的项目是使用自定义的依赖关系管理系统或类似Eclipse的.classpath文件作为主数据的依赖管理,这是很容易写一个Gradle插件来用这个数据Gradle.对于移民而言,这是一种常见的技术与Gradle.（但是,一旦你迁移,这可能是一个好主意,从.classpath文件搬走,并用Gradle的依赖管理功能直接.）

Even if your project is using a custom dependency management system or something like an Eclipse .classpath file as master data for dependency management, it is very easy to write a Gradle plugin to use this data in Gradle. For migration purposes this is a common technique with Gradle. (But, once you've migrated, it might be a good idea to move away from a .classpath file and use Gradle's dependency management features directly.)

### **52.1.2.依赖管理和Java**

52.1.2. Dependency management and Java

讽刺的是,对于拥有丰富开源组件库的一种语言,Java没有库或版本的概念。在Java中,没有标准的方式告诉JVM,您使用的Hibernate版本是3.0.5 ,以及foo-1.0.jar取决于bar-2.0.jar。这导致对外的解决方案通常基于构建工具。目前最受欢迎的是Maven和Ivy。虽然Maven提供了一个完整的构建系统,Ivy仅仅关注依赖关系管理。

It is ironic that in a language known for its rich library of open source components that Java has no concept of libraries or versions. In Java, there is no standard way to tell the JVM that you are using version 3.0.5 of Hibernate, and there is no standard way to say that foo-1.0.jar depends on bar-2.0.jar. This has led to external solutions often based on build tools. The most popular ones at the moment are Maven and Ivy. While Maven provides a complete build system, Ivy focuses solely on dependency management.

这两个工具都依赖于XML文件描述符,其包含了有关某个jar依赖项的信息。都使用存储库放置实际的jar包连同他们的描述符文件,都提供解决一种或其他种方式的jar版本冲突方案。都已出现解决依赖冲突的标准,虽然Gradle最初使用Ivy进行依赖关系管理。Gradle使用本地Gradle依赖解析引擎已经取代了这样直接依赖Ivy的方式，它支持一系列依赖解析方法包括POM和Ivy描述符文件。

Both tools rely on descriptor XML files, which contain information about the dependencies of a particular jar. Both also use repositories where the actual jars are placed together with their descriptor files, and both offer resolution for conflicting jar versions in one form or the other. Both have emerged as standards for solving dependency conflicts, and while Gradle originally used Ivy under the hood for its dependency management. Gradle has replaced this direct dependency on Ivy with a native Gradle dependency resolution engine which supports a range of approaches to dependency resolution including both POM and Ivy descriptor files.

## **52.2. 依赖管理最佳实践**

52.2. Dependency Management Best Practices

由于Gradle有很强的依赖关系管理的想法,该工具为您提供了两个选择:按照推荐的最佳实践或支持任何类型你能想到的模式。本节概述了Gradle项
目推荐的管理依赖性最佳实践。

While Gradle has strong opinions on dependency management, the tool gives you a choice between two options: follow recommended best practices or support any kind of pattern you can think of. This section outlines the Gradle project's recommended best practices for managing dependencies.

不管什么语言,合适的依赖关系管理对于每一个项目都是很重要的。从一个复杂的用Java编写的企业应用程序依赖于数以百计的开源库到最简单的取决于少数库的Clojure应用、依赖管理方法差别很大,它可以依赖于目标技术,应用程序部署方法,和项目的性质。比起企业应用程序集成到更大的系统的软件和基础设施，项目打包为可重用的库可能有不同的需求。尽管如此宽的变化要求,Gradle项目建议,所有的项目都遵循这一套核心规则:

No matter what the language, proper dependency management is important for every project. From a complex enterprise application written in Java depending on hundreds of open source libraries to the simplest Clojure application depending on a handful of libraries, approaches to dependency management vary widely and can depend on the target technology, the method of application deployment, and the nature of the project. Projects bundled as reusable libraries may have different requirements than enterprise applications integrated into much larger systems of software and infrastructure. Despite this wide variation of requirements, the Gradle project recommends that all projects follow this set of core rules:

### **52.2.1. 把版本放在文件名中（jar文件版本化）**

52.2.1. Put the Version in the Filename (Version the jar)

一个库的版本必须是文件名的一部分。然而版本的jar通常是在Manifest文件中,当你检查项目时它不是显而易见的。如果有人让你看20个jar文件,你更喜欢哪种?文件名称像commons-beanutils-1.3.jar还是文件名像spring.jar的文件?如果依赖文件名使用版本号可以使您快速识别依赖版本。

The version of a library must be part of the filename. While the version of a jar is usually in the Manifest file, it isn't readily apparent when you are inspecting a project. If someone asks you to look at a collection of 20 jar files, which would you prefer? A collection of files with names like commons-beanutils-1.3.jar or a collection of files with names like spring.jar? If dependencies have file names with version numbers you can quickly identify the versions of your dependencies.

如果版本不清楚你可能会引出微妙非常难找的Bug。例如可能有一个项目使用Hibernate 2.5。试想一个开发人员决定在她的机器上安装版本3.0.5的Hibernate修复一个关键安全Bug但是忘了通知其他团队的这种变化。她可能成功解决安全问题,但她也引出了微妙的bug到代码库那就是使用了Hibernate当前已弃用的特性。几周后集成机器上发生了一个异常,且不能在别人的机器上复现。多个开发人员在这个问题上花费了很长时间最终意识到如果他们知道Hibernate已经从2.5升级到3.0.5，这个错误会很容易发现。

If versions are unclear you can introduce subtle bugs which are very hard to find. For example there might be a project which uses Hibernate 2.5. Think about a developer who decides to install version 3.0.5 of Hibernate on her machine to fix a critical security bug but forgets to notify others in the team of this change. She may address the security bug successfully, but she also may have introduced subtle bugs into a codebase that was using a now-deprecated feature from Hibernate. Weeks later there is an exception on the integration machine which can't be reproduced on anyone's machine. Multiple developers then spend days on this issue only finally realising that the error would have easy to uncover if they knew that Hibernate had been upgraded from 2.5 to 3.0.5.

jar包以版本命名增加了项目的表现力且使得更加容易维护。这种做法也会减少潜在的错误。

Versions in jar names increase the expressiveness of your project and make them easier to maintain. This practice also reduces the potential for error.

### **52.2.2.管理传递依赖关系**

52.2.2. Manage transitive dependencies

传递依赖管理技术,可使您的项目依赖于库,反过来这个库,依赖于其他库。这个递归传递依赖模式形成一个依赖关系树,其包括项目的一级依赖性,二级依赖性,等等。如果你不塑造依赖模型作为有层次包含一级和二级依赖的树，那样很容易很快就失去控制成为一个组装的非结构化的依赖树。考虑Gradle项目本身,它只有几个直接,一级的依赖关系,当它编译时它需要一百多依赖类路径。规模更大、使用Spring,Hibernate,和其他库的企业项目,以及成百上千的内部项目,可以导致非常大的依赖关系树。

Transitive dependency management is a technique that enables your project to depend on libraries which, in turn, depend on other libraries. This recursive pattern of transitive dependencies results in a tree of dependencies including your project's first-level dependencies, second-level dependencies, and so on. If you don't model your dependencies as a hierarchical tree of first-level and second-level dependencies it is very easy to quickly lose control over an assembled mess of unstructured dependencies. Consider the Gradle project itself, while Gradle only has a few direct, first-level dependencies, when Gradle is compiled it needs more than one hundred dependencies on the classpath. On a far larger scale, Enterprise projects using Spring, Hibernate, and other libraries, alongside hundreds or thousands of internal projects, can result in very large dependency trees.

When these large dependency trees need to change, you'll often have to solve some dependency version conflicts. Say one open source library needs one version of a logging library and a another uses an alternative version. Gradle and other build tools all have the ability to resolve conflicts, but what differentiates Gradle is the control it gives you over transitive dependencies and conflict resolution.

While you could try to manage this problem manually, you will quickly find that this approach doesn't scale. If you want to get rid of a first level dependency you really can't be sure which other jars you should remove. A dependency of a first level dependency might also be a first level dependency itself, or it might be a transitive dependency of yet another first level dependency. If you try to manage transitive dependencies yourself, the end of the story is that your build becomes brittle: no one dares to change your dependencies because the risk of breaking the build is too high. The project classpath becomes a complete mess, and, if a classpath problem arises, hell on earth invites you for a ride.

NOTE:In one project, we found a mystery LDAP related jar in the classpath. No code referenced this jar and there was no connection to the project. No one could figure out what the jar was for, until it was removed from the build and the application suffered massive performance problems whenever it attempted to authenticate to LDAP. This mystery jar was a necessary transitive, fourth-level dependency that was easy to miss because no one had bothered to use managed transitive dependencies.
Gradle offers you different ways to express first-level and transitive dependencies. With Gradle you can mix and match approaches; for example, you could store your jars in an SCM without XML descriptor files and still use transitive dependency management.

52.2.3. Resolve version conflicts

Conflicting versions of the same jar should be detected and either resolved or cause an exception. If you don't use transitive dependency management, version conflicts are undetected and the often accidental order of the classpath will determine what version of a dependency will win. On a large project with many developers changing dependencies, successful builds will be few and far between as the order of dependencies may directly affect whether a build succeeds or fails (or whether a bug appears or disappears in production).

If you haven't had to deal with the curse of conflicting versions of jars on a classpath, here is a small anecdote of the fun that awaits you. In a large project with 30 submodules, adding a dependency to a subproject changed the order of a classpath, swapping Spring 2.5 for an older 2.4 version. While the build continued to work, developers were starting to notice all sorts of surprising (and surprisingly awful) bugs in production. Worse yet, this unintentional downgrade of Spring introduced several security vulnerabilities into the system, which now required a full security audit throughout the organization.

In short, version conflicts are bad, and you should manage your transitive dependencies to avoid them. You might also want to learn where conflicting versions are used and consolidate on a particular version of a dependency across your organization. With a good conflict reporting tool like Gradle, that information can be used to communicate with the entire organization and standardize on a single version. If you think version conflicts don't happen to you, think again. It is very common for different first-level dependencies to rely on a range of different overlapping versions for other dependencies, and the JVM doesn't yet offer an easy way to have different versions of the same jar in the classpath (see Section 50.1.2, “Dependency management and Java”).

Gradle offers the following conflict resolution strategies:

Newest: The newest version of the dependency is used. This is Gradle's default strategy, and is often an appropriate choice as long as versions are backwards-compatible.
Fail: A version conflict results in a build failure. This strategy requires all version conflicts to be resolved explicitly in the build script. See ResolutionStrategy for details on how to explicitly choose a particular version.
While the strategies introduced above are usually enough to solve most conflicts, Gradle provides more fine-grained mechanisms to resolve version conflicts:

Configuring a first level dependency as forced. This approach is useful if the dependency in conflict is already a first level dependency. See examples in DependencyHandler.
Configuring any dependency (transitive or not) as forced. This approach is useful if the dependency in conflict is a transitive dependency. It also can be used to force versions of first level dependencies. See examples in ResolutionStrategy
Dependency resolve rules are an incubating feature introduced in Gradle 1.4 which give you fine-grained control over the version selected for a particular dependency.
To deal with problems due to version conflicts, reports with dependency graphs are also very helpful. Such reports are another feature of dependency management.

52.2.4. Use Dynamic Versions and Changing Modules

There are many situations when you want to use the latest version of a particular dependency, or the latest in a range of versions. This can be a requirement during development, or you may be developing a library that is designed to work with a range of dependency versions. You can easily depend on these constantly changing dependencies by using a dynamic version. A dynamic version can be either a version range (e.g. 2.+) or it can be a placeholder for the latest version available (e.g. latest.integration).

Alternatively, sometimes the module you request can change over time, even for the same version. An example of this type of changing module is a Maven SNAPSHOT module, which always points at the latest artifact published. In other words, a standard Maven snapshot is a module that never stands still so to speak, it is a “changing module”.

The main difference between a dynamic version and a changing module is that when you resolve a dynamic version, you'll get the real, static version as the module name. When you resolve a changing module, the artifacts are named using the version you requested, but the underlying artifacts may change over time.

By default, Gradle caches dynamic versions and changing modules for 24 hours. You can override the default cache modes using command line options. You can change the cache expiry times in your build using the resolution strategy (see Section 50.9.3, “Fine-tuned control over dependency caching”).

## **52.3. 依赖配置**

52.3. Dependency configurations

Gradle依赖被分组到配置.配置有一个名称,其他一些特性,并且它们可以彼此扩展.许多Gradle插件添加预定义的配置到你的项目.Java插件,例如,增加了一些配置来表示它需要的各种classpaths.详细信息参见第22.5节-“依赖管理”.当然,你可以在此之上添加自定义配置.有许多用例为自定义配置.这是非常方便的,例如添加不需要的依赖进行构建,需要或测试软件（如随你的发行版发布额外的JDBC驱动程序）.

In Gradle dependencies are grouped into configurations. Configurations have a name, a number of other properties, and they can extend each other. Many Gradle plugins add pre-defined configurations to your project. The Java plugin, for example, adds some configurations to represent the various classpaths it needs. see Section 23.5, “Dependency management” for details. Of course you can add custom configurations on top of that. There are many use cases for custom configurations. This is very handy for example for adding dependencies not needed for building or testing your software (e.g. additional JDBC drivers to be shipped with your distribution).

一个项目的配置是通过一个配置对象来管理的.将你传递给配置对象的闭包应用于它的API.更多的了解这个API请看ConfigurationContainer。

A project's configurations are managed by a configurations object. The closure you pass to the configurations object is applied against its API. To learn more about this API have a look at ConfigurationContainer.

定义一个配置:

To define a configuration:

示例 52.1. 定义一个配置

Example 52.1. Definition of a configuration

build.gradle

```
configurations {
    compile
}
```

访问一个配置:
To access a configuration:

示例 52.2. 访问一个配置
Example 52.2. Accessing a configuration

build.gradle

```
println configurations.compile.name
println configurations['compile'].name
```

配置一个配置:

To configure a configuration:

示例 52.3. 配置一个配置

Example 52.3. Configuration of a configuration

build.gradle
```
configurations {
    compile {
        description = 'compile classpath'
        transitive = true
    }
    runtime {
        extendsFrom compile
    }
}

configurations.compile {
    description = 'compile classpath'
}
```

## **52.4. 如何声明你的依赖**

52.4. How to declare your dependencies

你可以声明几种不同类型的依赖

There are several different types of dependencies that you can declare:

表 52.1. 依赖类型

Table 52.1. Dependency types

|Type	|Description|
|--
|External module dependency|	A dependency on an external module in some repository.|
|Project dependency|	A dependency on another project in the same build.|
|File dependency	|A dependency on a set of files on the local filesystem.|
|Client module dependency	|A dependency on an external module, where the artifacts are located in some repository but the module meta-data is specified by the local build. You use this kind of dependency when you want to override the meta-data for the module.|
|Gradle API dependency	|A dependency on the API of the current Gradle version. You use this kind of dependency when you are developing custom Gradle plugins and task types.|
|Local Groovy dependency|	A dependency on the Groovy version used by the current Gradle version. You use this kind of dependency when you are developing custom Gradle plugins and task types.|

### **52.4.1. 外部模块依赖**

52.4.1. External module dependencies

外部模块依赖关系是最常见的依赖关系.它们是指外部库中的一个模块.

External module dependencies are the most common dependencies. They refer to a module in an external repository.

示例 52.4. 模块依赖

Example 52.4. Module dependencies

build.gradle

```
dependencies {
    runtime group: 'org.springframework', name: 'spring-core', version: '2.5'
    runtime 'org.springframework:spring-core:2.5',
            'org.springframework:spring-aop:2.5'
    runtime(
        [group: 'org.springframework', name: 'spring-core', version: '2.5'],
        [group: 'org.springframework', name: 'spring-aop', version: '2.5']
    )
    runtime('org.hibernate:hibernate:3.0.5') {
        transitive = true
    }
    runtime group: 'org.hibernate', name: 'hibernate', version: '3.0.5', transitive: true
    runtime(group: 'org.hibernate', name: 'hibernate', version: '3.0.5') {
        transitive = true
    }
}
```

更多示例和一个完整的参考参见API文档中的DependencyHandler类.

See the DependencyHandler class in the API documentation for more examples and a complete reference.

Gradle为模块依赖提供不同的表示法.这有一个string表示法和一个map表示法.一个模块依赖有一个允许进一步配置的API.关于更多的了解参见API中的ExternalModuleDependency.该API提供了一些属性和配置方法.通过string表示法你可以定义一个subset属性.利用map表示你可以定义所有属性.无论是map表示法还是string表示法只要获得完整的API,你就可以可以指定一个单一依赖配置一起关闭.

Gradle provides different notations for module dependencies. There is a string notation and a map notation. A module dependency has an API which allows further configuration. Have a look at ExternalModuleDependency to learn all about the API. This API provides properties and configuration methods. Via the string notation you can define a subset of the properties. With the map notation you can define all properties. To have access to the complete API, either with the map or with the string notation, you can assign a single dependency to a configuration together with a closure.

如果你声明一个依赖,Gradle会在repositories查找一个模块的描述文件(pom.xml或ivy.xml).如果存在这样的模块描述文件,则解析这个模块和依赖产物(如. hibernate-3.0.5.jar)以及其下载其依赖项(如. cglib). 如果不存在这样的模块描述文件,Gradle会查找一个叫hibernate-3.0.5.jar的文件来检索.在Maven中,一个模块能够一个且仅有一个依赖产物. 在Gradle和Ivy中,一个模块能有多个依赖产物,每个依赖产物可以有一组不同的依赖.

If you declare a module dependency, Gradle looks for a module descriptor file (pom.xml or ivy.xml) in the repositories. If such a module descriptor file exists, it is parsed and the artifacts of this module (e.g. hibernate-3.0.5.jar) as well as its dependencies (e.g. cglib) are downloaded. If no such module descriptor file exists, Gradle looks for a file called hibernate-3.0.5.jar to retrieve. In Maven, a module can have one and only one artifact. In Gradle and Ivy, a module can have multiple artifacts. Each artifact can have a different set of dependencies.

#### **52.4.1.1. 依赖模块与多依赖产物**

52.4.1.1. Depending on modules with multiple artifacts

如前所述,一个Maven模块只有一个依赖产物.因此,当你的项目依赖一个Maven模块,它的依赖产物是很明显的.用Gradle或Ivy,情况就不同了.Ivy的依赖描述文件（ivy.xml）可声明多个依赖产物.关于了解更多信息,请参见Ivy reference的ivy.xml.在Gradle中,当你在一个Ivy模块上声明一个依赖,你实际上就声明了一个默认配置模块的依赖.所以实际的依赖产物(通常是jar)是你依赖的产物与缺省配置相关联的模块.这里有一些情况是很重要的:

As mentioned earlier, a Maven module has only one artifact. Hence, when your project depends on a Maven module, it's obvious what its artifact is. With Gradle or Ivy, the case is different. Ivy's dependency descriptor (ivy.xml) can declare multiple artifacts. For more information, see the Ivy reference for ivy.xml. In Gradle, when you declare a dependency on an Ivy module, you actually declare a dependency on the default configuration of that module. So the actual set of artifacts (typically jars) you depend on is the set of artifacts that are associated with the default configuration of that module. Here are some situations where this matters:

模块的默认配置包含不需要的依赖产物.仅仅依赖声明所需的依赖产物,而不是取决于整个配置.

The default configuration of a module contains undesired artifacts. Rather than depending on the whole configuration, a dependency on just the desired artifacts is declared.

所需的依赖产物属于一个其他的默认配置.该配置被显式命名为依赖声明的一部分。

The desired artifact belongs to a configuration other than default. That configuration is explicitly named as part of the dependency declaration.

在其他情况是它是需要调整声明依赖的.请参见API文档中DependencyHandler类的例子和参考完整的依赖声明.

There are other situations where it is necessary to fine-tune dependency declarations. Please see the DependencyHandler class in the API documentation for examples and a complete reference for declaring dependencies.

#### **52.4.1.2. 依赖产物的表示法**

52.4.1.2. Artifact only notation

如上所说,如果没有找到模块描述文件,Gradle默认下载的jar名称的模块.但有时,即使库中包含模块描述文件,你仅要下载依赖产物的jar文件,不要其他依赖.[14]且有时你想要从一个库中下载没有模块属性的zip文件,Gradle提供一个关于那些用例-简单的前缀要与下载的扩展名“@”符号的构建产物表示法:

As said above, if no module descriptor file can be found, Gradle by default downloads a jar with the name of the module. But sometimes, even if the repository contains module descriptors, you want to download only the artifact jar, without the dependencies. [14] And sometimes you want to download a zip from a repository, that does not have module descriptors. Gradle provides an artifact only notation for those use cases - simply prefix the extension that you want to be downloaded with '@' sign:

示例 52.5. 依赖产物的表示法

Example 52.5. Artifact only notation

build.gradle
```
dependencies {
    runtime "org.groovy:groovy:2.2.0@jar"
    runtime group: 'org.groovy', name: 'groovy', version: '2.2.0', ext: 'jar'
}
```

一个依赖产物表示法创建一个用指定扩展名仅下载依赖产物文件的模块依赖.忽略现有模块描述文件.

An artifact only notation creates a module dependency which downloads only the artifact file with the specified extension. Existing module descriptors are ignored.

#### **52.4.1.3. 分类**

52.4.1.3. Classifiers

Maven依赖管理有分类的概念.[15]Gradle也支持.从Maven库检索分类依赖你可以这样写:

The Maven dependency management has the notion of classifiers. [15] Gradle supports this. To retrieve classified dependencies from a Maven repository you can write:

示例 52.6. 依赖分类

Example 52.6. Dependency with classifier

build.gradle
```
compile "org.gradle.test.classifiers:service:1.0:jdk15@jar"
otherConf group: 'org.gradle.test.classifiers', name: 'service', version: '1.0', classifier: 'jdk14'
```

我们可以看到在上面的第一行中,分类可以与依赖产物表示法一起使用.

As can be seen in the first line above, classifiers can be used together with the artifact only notation.

这很容易遍历一个配置的依赖产物：

It is easy to iterate over the dependency artifacts of a configuration:

示例 52.7. 遍历一个配置

Example 52.7. Iterating over a configuration

build.gradle
```
task listJars << {
    configurations.compile.each { File file -> println file.name }
}
```

Output of gradle -q listJars
```
> gradle -q listJars
hibernate-core-3.6.7.Final.jar
antlr-2.7.6.jar
commons-collections-3.1.jar
dom4j-1.6.1.jar
hibernate-commons-annotations-3.2.0.Final.jar
hibernate-jpa-2.0-api-1.0.1.Final.jar
jta-1.1.jar
slf4j-api-1.6.1.jar
```

### **52.4.2. 客户端模块依赖**

52.4.2. Client module dependencies

客户端模块的依赖关系允许你在构建脚本中直接声明可传递的依赖.它们是一个外部库中的替换模块描述。

Client module dependencies allow you to declare transitive dependencies directly in the build script. They are a replacement for a module descriptor in an external repository.

示例 52.8. 客户端模块依赖 - 可传递依赖

Example 52.8. Client module dependencies - transitive dependencies

build.gradle
```
dependencies {
    runtime module("org.codehaus.groovy:groovy:2.3.10") {
        dependency("commons-cli:commons-cli:1.0") {
            transitive = false
        }
        module(group: 'org.apache.ant', name: 'ant', version: '1.9.4') {
            dependencies "org.apache.ant:ant-launcher:1.9.4@jar",
                         "org.apache.ant:ant-junit:1.9.4"
        }
    }
}
```

在Groovy上声明该依赖.Groovy本身就具有依赖性.但是Gradle不会从构建文件信息中找XML描述文件出来.客户端模块的依赖可以是标准模块依赖或依赖产物或另一个客户端模块.也可以看看API文档的ClientModule类.

This declares a dependency on Groovy. Groovy itself has dependencies. But Gradle does not look for an XML descriptor to figure them out but gets the information from the build file. The dependencies of a client module can be normal module dependencies or artifact dependencies or another client module. Also look at the API documentation for the ClientModule class.

在当前版本的客户端模块中有一个限制.比方说你的项目是一个库并且你想这个库上传到贵公司的Maven或Ivy库中.Gradle上传你项目的Jar文件与依赖的XML描述文件到公司资源库中.如果你使用客户端模块在XML中描述符文件声明依赖是不正确的.我们将在Gradle的未来版本中改进这一点.

In the current release client modules have one limitation. Let's say your project is a library and you want this library to be uploaded to your company's Maven or Ivy repository. Gradle uploads the jars of your project to the company repository together with the XML descriptor file of the dependencies. If you use client modules the dependency declaration in the XML descriptor file is not correct. We will improve this in a future release of Gradle.

### **50.4.3. 项目依赖**

50.4.3. Project dependencies

Gradle在类似多项目构建的一部分项目中区分外部依赖和依赖.对于后者你可以声明为项目依赖.

Gradle distinguishes between external dependencies and dependencies on projects which are part of the same multi-project build. For the latter you can declare Project Dependencies.

示例 50.9. 项目依赖

Example 50.9. Project dependencies

build.gradle
```
dependencies {
    compile project(':shared')
}
```

更多信息参见API文档的ProjectDependency.

For more information see the API documentation for ProjectDependency.

多项目构建在第56章-Multi-project Builds中已讨论过.

Multi-project builds are discussed in Chapter 56, Multi-project Builds.

### **50.4.4. 文件依赖**

50.4.4. File dependencies

文件依赖允许你直接添加一组文件到配置中,而不需先将它们添加到库中.如果你不能或不想这么做-在库中存放一些文件,这可能是有用的.或者你不希望使用任何库存储你的依赖关系.

File dependencies allow you to directly add a set of files to a configuration, without first adding them to a repository. This can be useful if you cannot, or do not want to, place certain files in a repository. Or if you do not want to use any repositories at all for storing your dependencies.

要添加一些文件作为依赖的配置,你只需通过一个文件集合的依赖：

To add some files as a dependency for a configuration, you simply pass a file collection as a dependency:

示例 50.10. 文件依赖

Example 50.10. File dependencies

build.gradle
```
dependencies {
    runtime files('libs/a.jar', 'libs/b.jar')
    runtime fileTree(dir: 'libs', include: '*.jar')
}
```

文件依赖不包括在你项目已发布的依赖描述中.但是,文件依赖是包含在传递项目依赖内的.这意味着它们不能被当前构建的外部使用,但是他们可以使用相同的构建

File dependencies are not included in the published dependency descriptor for your project. However, file dependencies are included in transitive project dependencies within the same build. This means they cannot be used outside the current build, but they can be used with the same build.

你可以声明任务生成文件的文件依赖关系.你可以这样做,如当通过构建生成文件时.

You can declare which tasks produce the files for a file dependency. You might do this when, for example, the files are generated by the build.

示例 50.11. 生成文件依赖

Example 50.11. Generated file dependencies

build.gradle
```
dependencies {
    compile files("$buildDir/classes") {
        builtBy 'compile'
    }
}

task compile << {
    println 'compiling classes'
}

task list(dependsOn: configurations.compile) << {
    println "classpath = ${configurations.compile.collect {File file -> file.name}}"
}
```

Output of gradle -q list
```
> gradle -q list
compiling classes
classpath = [classes]
```

### **50.4.5. Gradle API依赖**

50.4.5. Gradle API Dependency

你可以通过使用DependencyHandler.gradleApi()方法在Gradle当前版本的API中声明一个依赖,当你开发自定义的Gradle任务或插件时是非常有用的.

You can declare a dependency on the API of the current version of Gradle by using the DependencyHandler.gradleApi() method. This is useful when you are developing custom Gradle tasks or plugins.

示例 50.12. Gradle API依赖

Example 50.12. Gradle API dependencies

build.gradle
```
dependencies {
    compile gradleApi()
}
```

### **50.4.6. 本地Groovy依赖**

50.4.6. Local Groovy Dependency

你可以通过使用DependencyHandler.localGroovy()方法在Gradle分布式的Groovy上声明一个依赖.当你在Groovy上开发自定义Gradle任务或插件时这是非常有用的.

You can declare a dependency on the Groovy that is distributed with Gradle by using the DependencyHandler.localGroovy() method. This is useful when you are developing custom Gradle tasks or plugins in Groovy.

示例 50.13. Gradle的Groovy依赖

Example 50.13. Gradle's Groovy dependencies

build.gradle
```
dependencies {
    compile localGroovy()
}
```

### **50.4.7. 排除可传递依赖**

50.4.7. Excluding transitive dependencies

通过配置或依赖你可以排除可传递依赖:

You can exclude a transitive dependency either by configuration or by dependency:

示例 50.14. 排除可传递依赖

Example 50.14. Excluding transitive dependencies

build.gradle
```
configurations {
    compile.exclude module: 'commons'
    all*.exclude group: 'org.gradle.test.excludes', module: 'reports'
}

dependencies {
    compile("org.gradle.test.excludes:api:1.0") {
        exclude module: 'shared'
    }
}
```

如果你定义了一个特定配置的排除,当解决此配置或任意继承配置时该排除可传递依赖将过滤所有依赖.如果你想从你的所有配置排除一个可传递依赖,你可以试用Groovy的spread-dot运算符来表达这种以简洁的方式.如图所示.当定义一个排除时,你可以指定任何该组织或模块名或同时指定两者.请参见API文档中的Dependency和Configuration类.

If you define an exclude for a particular configuration, the excluded transitive dependency will be filtered for all dependencies when resolving this configuration or any inheriting configuration. If you want to exclude a transitive dependency from all your configurations you can use the Groovy spread-dot operator to express this in a concise way, as shown in the example. When defining an exclude, you can specify either only the organization or only the module name or both. Also look at the API documentation of the Dependency and Configuration classes.

不是所有的可传递依赖都能排除的 - 一些可传递依赖可能是应用程序的正确运行时行为所必需的.可排除的依赖通常是运行时非必须或在目标环境/平台已是可用的.

Not every transitive dependency can be excluded - some transitive dependencies might be essential for correct runtime behavior of the application. Generally, one can exclude transitive dependencies that are either not required by runtime or that are guaranteed to be available on the target environment/platform.

你应该排除全部依赖或配置?事实证明大多数情况下你要使用全部配置排除.这里是我们希望排除可传递依赖的一些典型原因.请记住,对于一些我们使用的用例有比排除更好的解决方案！

Should you exclude per-dependency or per-configuration? It turns out that in the majority of cases you want to use the per-configuration exclusion. Here are some typical reasons why one might want to exclude a transitive dependency. Bear in mind that for some of these use cases there are better solutions than exclusions!

依赖不希望是licensing原因引起的.

The dependency is undesired due to licensing reasons.

依赖是在任意远程库中没有价值的.

The dependency is not available in any remote repositories.

依赖是运行时非必须的.

The dependency is not needed for runtime.

依赖的版本与期望版本冲突.对于使用情况,请参见第50.2.3,“Resolve version conflicts” 并在文件ResolutionStrategy对于这个问题可能有更好的解决方案.

The dependency has a version that conflicts with a desired version. For that use case please refer to Section 50.2.3, “Resolve version conflicts” and the documentation on ResolutionStrategy for a potentially better solution to the problem.

基本上,在大多数情况下排除可传递依赖应该在每个配置中完成.这样依赖声明会更明确.它更准确是因为每个依赖排除规则不保证给定的传递依赖不在配置中显示.例如,一些没有任何排除规则,可能留下可传递的依赖.

Basically, in most of the cases excluding the transitive dependency should be done per configuration. This way the dependency declaration is more explicit. It is also more accurate because a per-dependency exclude rule does not guarantee the given transitive dependency does not show up in the configuration. For example, some other dependency, which does not have any exclude rules, might pull in that unwanted transitive dependency.

其他依赖排除的例子在reference中的ModuleDependency和DependencyHandler类里找到.

Other examples of dependency exclusions can be found in the reference for the ModuleDependency or DependencyHandler classes.

### **50.4.8. 可选属性**

50.4.8. Optional attributes

依赖的所有属性是可选的,除了名字.在库中真正查找所需的依赖取决于库类型.参见50.6章节,“Repositories”.例如,你使用Maven库,你需要定义一个组,名字和版本.如果你使用文件系统库你可能只需要名字或名字与版本.

All attributes for a dependency are optional, except the name. Which attributes are required for actually finding dependencies in the repository will depend on the repository type. See Section 50.6, “Repositories”. For example, if you work with Maven repositories, you need to define the group, name and version. If you work with filesystem repositories you might only need the name or the name and the version.

示例 50.15. 依赖可选属性

Example 50.15. Optional attributes of dependencies

build.gradle
```
dependencies {
    runtime ":junit:4.12", ":testng"
    runtime name: 'testng' 
}
```

你还可以将依赖表示法的集合或数组分配到一个配置:

You can also assign collections or arrays of dependency notations to a configuration:

示例 50.16. 依赖集合和数组

Example 50.16. Collections and arrays of dependencies

build.gradle
```
List groovy = ["org.codehaus.groovy:groovy-all:2.3.10@jar",
               "commons-cli:commons-cli:1.0@jar",
               "org.apache.ant:ant:1.9.4@jar"]
List hibernate = ['org.hibernate:hibernate:3.0.5@jar',
                  'somegroup:someorg:1.0@jar']
dependencies {
    runtime groovy, hibernate
}
```

### **50.4.9. 依赖配置**

50.4.9. Dependency configurations

在Gradle中依赖可以有不同的配置(如你的项目有不同的配置).如果你不明确指定任务东西,Gradle会使用默认依赖配置.Maven库的依赖,默认配置是唯一的可能性.如果你使用Ivy库和要声明一个非默认配置为你的依赖,你必须使用map表示法来声明：

In Gradle a dependency can have different configurations (as your project can have different configurations). If you don't specify anything explicitly, Gradle uses the default configuration of the dependency. For dependencies from a Maven repository, the default configuration is the only possibility anyway. If you work with Ivy repositories and want to declare a non-default configuration for your dependency you have to use the map notation and declare:

示例 50.17. 依赖配置

Example 50.17. Dependency configurations

build.gradle
```
dependencies {
    runtime group: 'org.somegroup', name: 'somedependency', version: '1.0', configuration: 'someConfiguration'
}
```

做同样的项目依赖你需要声明:

To do the same for project dependencies you need to declare:

示例 50.18. 项目依赖配置

Example 50.18. Dependency configurations for project

build.gradle
```
dependencies {
    compile project(path: ':api', configuration: 'spi')
}
```

### **50.4.10. 依赖报告**

50.4.10. Dependency reports

你能够通过命令行生成依赖报告(参见11.6.4章节,"Listing project dependencies").在项目报告插件(参见第41章,The Project Report Plugin)的帮助下通过你的构建能够创建一个报告.

You can generate dependency reports from the command line (see Section 11.6.4, “Listing project dependencies”). With the help of the Project report plugin (see Chapter 41, The Project Report Plugin) such a report can be created by your build.

由于Gradle 1.2还有一个新的编程API可以访问分解的依赖信息。依赖报告(见前一个段落)正是使用这个API。API允许您按照分解的依赖图走,并提供依赖信息。在未来的版本中此API将会提供更多关于解析结果的信息。关于API的更多信息请参阅Javadocs ResolvableDependencies.getResolutionResult()。潜在的ResolutionResult API的用法:

Since Gradle 1.2 there is also a new programmatic API to access the resolved dependency information. The dependency reports (see the previous paragraph) are using this API under the covers. The API lets you walk the resolved dependency graph and provides information about the dependencies. In future releases the API will grow to provide more information about the resolution result. For more information about the API please refer to the Javadocs on ResolvableDependencies.getResolutionResult(). Potential usages of the ResolutionResult API:

根据你的用例创建高级的依赖报告。　　

使构建逻辑决策基于依赖图的内容。

Creation of advanced dependency reports tailored to your use case.
Enabling the build logic to make decisions based on the content of the dependency graph.

## **50.5. 使用依赖工作**

50.5. Working with dependencies

下面的例子,我们有以下依赖设置:

For the examples below we have the following dependencies setup:

Example 50.19. Configuration.copy

build.gradle
```
configurations {
    sealife
    alllife
}

dependencies {
    sealife "sea.mammals:orca:1.0", "sea.fish:shark:1.0", "sea.fish:tuna:1.0"
    alllife configurations.sealife
    alllife "air.birds:albatross:1.0"
}
```

这个依赖有如下传递的依赖关系：

The dependencies have the following transitive dependencies:

shark-1.0 -> seal-2.0, tuna-1.0

orca-1.0 -> seal-1.0

tuna-1.0 -> herring-1.0

你可以使用此配置去访问声明的依赖关系或者他们的子集：

You can use the configuration to access the declared dependencies or a subset of those:

Example 50.20. Accessing declared dependencies

build.gradle
```
task dependencies << {
    configurations.alllife.dependencies.each { dep -> println dep.name }
    println()
    configurations.alllife.allDependencies.each { dep -> println dep.name }
    println()
    configurations.alllife.allDependencies.findAll { dep -> dep.name != 'orca' }
        .each { dep -> println dep.name }
}
```

Output of gradle -q dependencies
```
> gradle -q dependencies
albatross

albatross
orca
shark
tuna

albatross
shark
tuna
```

依赖关系的任务只返回属于显式配置的依赖关系。allDependencies任务包括扩展配置的依赖关系。

The dependencies task returns only the dependencies belonging explicitly to the configuration. The allDependencies task includes the dependencies from extended configurations.

要获取配置依赖的库文件，你可以这样做：

To get the library files of the configuration dependencies you can do:

Example 50.21. Configuration.files

build.gradle
```
task allFiles << {
    configurations.sealife.files.each { file ->
        println file.name
    }
}
```

Output of gradle -q allFiles
```
> gradle -q allFiles
orca-1.0.jar
shark-1.0.jar
tuna-1.0.jar
herring-1.0.jar
seal-2.0.jar
```

有时你想要一个配置依赖子集的库文件(例如一个单独的依赖)。

Sometimes you want the library files of a subset of the configuration dependencies (e.g. of a single dependency).

Example 50.22. Configuration.files with spec

build.gradle
```
task files << {
    configurations.sealife.files { dep -> dep.name == 'orca' }.each { file ->
        println file.name
    }
}
```

Output of gradle -q files
```
> gradle -q files
orca-1.0.jar
seal-2.0.jar
```

Configuration.files方法总是检索整体配置的所有产物。然后过滤检索文件的指定依赖项。正如你在这个例子中所看到的,包括传递依赖关系。

The Configuration.files method always retrieves all artifacts of the whole configuration. It then filters the retrieved files by specified dependencies. As you can see in the example, transitive dependencies are included.

您也可以复制一个配置。您可以选择性地指定,只有一个子集的依赖关系可以从原始配置复制。有两种不同的复制方法。只复制方法复制属于显式配置的依赖。copyRecursive方法则复制所有的依赖关系,包括扩展配置的依赖关系。

You can also copy a configuration. You can optionally specify that only a subset of dependencies from the original configuration should be copied. The copying methods come in two flavors. The copy method copies only the dependencies belonging explicitly to the configuration. The copyRecursive method copies all the dependencies, including the dependencies from extended configurations.

Example 50.23. Configuration.copy

build.gradle
```
task copy << {
    configurations.alllife.copyRecursive { dep -> dep.name != 'orca' }
        .allDependencies.each { dep -> println dep.name }
    println()
    configurations.alllife.copy().allDependencies
        .each { dep -> println dep.name }
}
```

Output of gradle -q copy
```
> gradle -q copy
albatross
shark
tuna

albatross
```

重要的是要注意,复制配置的返回文件通常与原始配置依赖子集的返回文件不总是相同的。以防在子集依赖和不属于自己的依赖之间发生版本冲突，解析的结果可能有所不同。

It is important to note that the returned files of the copied configuration are often but not always the same than the returned files of the dependency subset of the original configuration. In case of version conflicts between dependencies of the subset and dependencies not belonging to the subset the resolve result might be different.

Example 50.24. Configuration.copy vs. Configuration.files

build.gradle
```
task copyVsFiles << {
    configurations.sealife.copyRecursive { dep -> dep.name == 'orca' }
        .each { file -> println file.name }
    println()
    configurations.sealife.files { dep -> dep.name == 'orca' }
        .each { file -> println file.name }
}
```
Output of gradle -q copyVsFiles
```
> gradle -q copyVsFiles
orca-1.0.jar
seal-1.0.jar

orca-1.0.jar
seal-2.0.jar
```

在上面的示例中,orca依赖于seal-1.0而shark依赖onseal-2.0。因此原始配置有版本冲突，解决办法是升级新的seal-2.0版本。于是文件方法返回seal-2.0作为orca的传递依赖。复制配置只有orca作为依赖项,因此没有版本冲突且seal-1.0作为传递依赖返回。

In the example above, orca has a dependency on seal-1.0 whereas shark has a dependency onseal-2.0. The original configuration has therefore a version conflict which is resolved to the newer seal-2.0 version. The files method therefore returns seal-2.0 as a transitive dependency of orca. The copied configuration only has orca as a dependency and therefore there is no version conflict and seal-1.0 is returned as a transitive dependency.

一旦配置被解析它是不可变的。改变其状态或它的一个依赖项的状态会导致一个异常。你可以复制一个解析配置。复制的配置处于未解析的状态,可刷新解决。

Once a configuration is resolved it is immutable. Changing its state or the state of one of its dependencies will cause an exception. You can always copy a resolved configuration. The copied configuration is in the unresolved state and can be freshly resolved.

想要了解更多关于配置类的API请查看API文档：Configuration。

To learn more about the API of the configuration class see the API documentation: Configuration.

## **50.6.存储库**

50.6. Repositories

Gradle库管理、基于Apache Ivy,给你很多仓库布局和检索策略的处理空间。此外Gradle提供了多种便利的方法添加预配置存储库。

Gradle repository management, based on Apache Ivy, gives you a lot of freedom regarding repository layout and retrieval policies. Additionally Gradle provides various convenience method to add pre-configured repositories.

你可以配置任何数量的存储库,每个都是由Gradle独立处理。如果Gradle在一个特定的存储库发现一个模块描述符,它将尝试下载的所有构件,从相同的存储库模块。尽管模块元数据和模块构件必须位于相同的存储库,但可以构建一个单独的存储库由多个url,给多个地址去搜索元数据文件和jar文件实现。

You may configure any number of repositories, each of which is treated independently by Gradle. If Gradle finds a module descriptor in a particular repository, it will attempt to download all of the artifacts for that module from the same repository. Although module meta-data and module artifacts must be located in the same repository, it is possible to compose a single repository of multiple URLs, giving multiple locations to search for meta-data files and jar files.

有很多不同类型可以生命的库：

There are several different types of repositories you can declare:

Table 50.2. Repository types

|Type|	Description|
|--
|Maven central repository	|A pre-configured repository that looks for dependencies in Maven Central.|
|Maven JCenter repository	|A pre-configured repository that looks for dependencies in Bintray's JCenter.|
|Maven local repository	|A pre-configured repository that looks for dependencies in the local Maven repository.|
|Maven repository	|A Maven repository. Can be located on the local filesystem or at some remote location.|
|Ivy repository	|An Ivy repository. Can be located on the local filesystem or at some remote location.|
|Flat directory repository|	A simple repository on the local filesystem. Does not support any meta-data formats.|

### **50.6.1. Maven中央存储库**

50.6.1. Maven central repository

欲添加Maven 2中央存储库(http://repo1.maven.org/maven2)仅需添加这个到您的构建脚本:

To add the central Maven 2 repository (http://repo1.maven.org/maven2) simply add this to your build script:

Example 50.25. Adding central Maven repository

build.gradle
```
repositories {
    mavenCentral()
}
```

Now Gradle will look for your dependencies in this repository.

警告：请注意,Maven 2中央存储库只支持HTTP,不支持HTTPS。如果你需要一个公共HTTPS中央存储库,您可以使用JCenter公共存储库(请参见50.6.2“Maven JCenter库”)。

Warning: Be aware that the central Maven 2 repository is HTTP only and HTTPS is not supported. If you need a public HTTPS enabled central repository, you can use the JCenter public repository (see Section 50.6.2, “Maven JCenter repository”).

### **50.6.2. Maven JCenter库**

50.6.2. Maven JCenter repository

Bintray JCenter是一个最新的所有流行的Maven OSS构件的集合,包括直接向Bintray发布的产物。

Bintray's JCenter is an up-to-date collection of all popular Maven OSS artifacts, including artifacts published directly to Bintray.

欲添加JCenter Maven存储库(http://repo1.maven.org/maven2)仅需添加这个到您的构建脚本:

To add the JCenter Maven repository (https://jcenter.bintray.com) simply add this to your build script:

Example 50.26. Adding Bintray's JCenter Maven repository

build.gradle
```
repositories {
    jcenter()
}
```

Now Gradle will look for your dependencies in the JCenter repository. jcenter() uses HTTPS to connect to the repository. If you want to use HTTP you can configure jcenter():

Example 50.27. Using Bintrays's JCenter with HTTP

build.gradle
```
repositories {
    jcenter {
        url "http://jcenter.bintray.com/"
    }
}
```

### **50.6.3. 本地Maven存储库**

50.6.3. Local Maven repository

为使用本地Maven缓存作为一个存储库你可以做：

To use the local Maven cache as a repository you can do:

Example 50.28. Adding the local Maven cache as a repository

build.gradle
```
repositories {
    mavenLocal()
}
```

Gradle使用Maven相同的逻辑来识别您的本地Maven缓存的位置。如果本地存储库位置在settings.xml中定义。那么这个地址将会被使用。settings.xml中USER_HOME/.m2的优先级高于M2_HOME/conf。如果没有可用的settings.xml,Gradle会使用默认位置USER_HOME/.m2/repository。

Gradle uses the same logic as Maven to identify the location of your local Maven cache. If a local repository location is defined in a settings.xml, this location will be used. The settings.xml in USER_HOME/.m2 takes precedence over the settings.xml in M2_HOME/conf. If no settings.xml is available, Gradle uses the default location USER_HOME/.m2/repository.

### **50.6.4. Maven 存储库**

50.6.4. Maven repositories

为了添加一个定制的Maven存储库你可以这样做：

For adding a custom Maven repository you can do:

Example 50.29. Adding custom Maven repository

build.gradle
```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

有时一个存储库会将POMs发布到一个位置，把jar文件和其他的产物发布在另一个位置。定义这样的库,你这样做:

Sometimes a repository will have the POMs published to one location, and the JARs and other artifacts published at another location. To define such a repository, you can do:

Example 50.30. Adding additional Maven repositories for JAR files

build.gradle
```
repositories {
    maven {
        // Look for POMs and artifacts, such as JARs, here
        url "http://repo2.mycompany.com/maven2"
        // Look for artifacts here if not found at the above location
        artifactUrls "http://repo.mycompany.com/jars"
        artifactUrls "http://repo.mycompany.com/jars2"
    }
}
```

Gradle首先看pom和jar文件的url。如果jar文件没有找到，产物的url用来寻找jar文件。

Gradle will look at the first URL for the POM and the JAR. If the JAR can't be found there, the artifact URLs are used to look for JARs.

### **50.6.4.1. 访问密码保护Maven存储库**

50.6.4.1. Accessing password protected Maven repositories

访问一个Maven存储库要使用基本身份验证,在您定义存储库时指定要使用的用户名和密码

To access a Maven repository which uses basic authentication, you specify the username and password to use when you define the repository:

Example 50.31. Accessing password protected Maven repository

build.gradle

```
repositories {
    maven {
        credentials {
            username 'user'
            password 'password'
        }
        url "http://repo.mycompany.com/maven2"
    }
}
```

建议将你的用户名和密码放在 gradle.properties文件中而不是直接放在build文件中。

It is advisable to keep your username and password in gradle.properties rather than directly in the build file.

### **50.6.5.平面目录库**

50.6.5. Flat directory repository

如果你想使用一个(平)文件系统目录作为存储库,只需输入:

If you want to use a (flat) filesystem directory as a repository, simply type:

Example 50.32. Flat repository resolver

build.gradle
```
repositories {
    flatDir {
        dirs 'lib'
    }
    flatDir {
        dirs 'lib1', 'lib2'
    }
}
```

这样增加存储库用来浏览一个或多个目录寻找依赖。注意,这种类型的库不支持任何meta-data格式像Ivy XML或Maven POM文件。取而代之,Gradle会基于构件的存在动态生成一个模块描述符(没有任何依赖信息)。然而,由于Gradle更喜欢使用实际元数据创建的模块描述符,而不是生成的，平面目录库不能用于覆盖其他存储库中的实际元数据。例如,如果它发现只有jmxri-1.2.1.jar在一个平面目录中,但jmxri-1.2.1.pom在另一个支持元数据的存储库里,它将使用第二个存储库提供模块。处于使用本地产物覆盖远程产物使用例子考虑使用Ivy或Maven代替。他们的url指向一个本地目录。如果你仅使用平面目录库工作不必设置一个依赖的所有属性。看50.4.8节“Optional attributes”。

This adds repositories which look into one or more directories for finding dependencies. Note that this type of repository does not support any meta-data formats like Ivy XML or Maven POM files. Instead, Gradle will dynamically generate a module descriptor (without any dependency information) based on the presence of artifacts. However, as Gradle prefers to use modules whose descriptor has been created from real meta-data rather than being generated, flat directory repositories cannot be used to override artifacts with real meta-data from other repositories. So, for example, if Gradle finds only jmxri-1.2.1.jar in a flat directory repository, but jmxri-1.2.1.pom in another repository that supports meta-data, it will use the second repository to provide the module. For the use case of overriding remote artifacts with local ones consider using an Ivy or Maven repository instead whose URL points to a local directory. If you only work with flat directory repositories you don't need to set all attributes of a dependency. See Section 50.4.8, “Optional attributes”.

### **50.6.6. Ivy 存储库**

50.6.6. Ivy repositories

#### **50.6.6.1. 使用标准布局定义一个Ivy存储库**

50.6.6.1. Defining an Ivy repository with a standard layout

Example 50.33. Ivy repository

build.gradle
```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

50.6.6.2. 为Ivy存储库定义一个指定的布局

50.6.6.2. Defining a named layout for an Ivy repository

您可以通过使用一个指定布局来指定符合Ivy或Maven默认布局的存储库。

You can specify that your repository conforms to the Ivy or Maven default layout by using a named layout.

Example 50.34. Ivy repository with named layout

build.gradle
```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        layout "maven"
    }
}
```

有效的指定布局值是'gradle'（默认），'maven' and 'ivy'.看API文档中IvyArtifactRepository.layout()获取这些指定布局的详细信息。

Valid named layout values are 'gradle' (the default), 'maven' and 'ivy'. See IvyArtifactRepository.layout() in the API documentation for details of these named layouts.

#### **50.6.6.3. 为Ivy存储库定义指定模式布局**

50.6.6.3. Defining custom pattern layout for an Ivy repository

To define an Ivy repository with a non-standard layout, you can define a 'pattern' layout for the repository:

Example 50.35. Ivy repository with pattern layout

build.gradle
```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        layout "pattern", {
            artifact "[module]/[revision]/[type]/[artifact].[ext]"
        }
    }
}
```

为了定义一个Ivy存储库从不同地址获取Ivy文件和产物，你可以定义不同的模式用来定位Ivy文件和产物。

To define an Ivy repository which fetches Ivy files and artifacts from different locations, you can define separate patterns to use to locate the Ivy files and artifacts:

每个指定的产物或Ivy在存储库添加一个额外的使用模式。这种模式的使用顺序同他们被定义时的顺序。

Each artifact or ivy specified for a repository adds an additional pattern to use. The patterns are used in the order that they are defined.

Example 50.36. Ivy repository with multiple custom patterns

build.gradle
````
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        layout "pattern", {
            artifact "3rd-party-artifacts/[organisation]/[module]/[revision]/[artifact]-[revision].[ext]"
            artifact "company-artifacts/[organisation]/[module]/[revision]/[artifact]-[revision].[ext]"
            ivy "ivy-files/[organisation]/[module]/[revision]/ivy.xml"
        }
    }
}
```

可选地,模式库布局可以有它的 'organisation'作为Maven风格的一部分,用正斜杠取代点作为分隔符。例如,组织my.company可被表示为my/company。

Optionally, a repository with pattern layout can have its 'organisation' part laid out in Maven style, with forward slashes replacing dots as separators. For example, the organisation my.company would then be represented as my/company.

Example 50.37. Ivy repository with Maven compatible layout

build.gradle
```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        layout "pattern", {
            artifact "[organisation]/[module]/[revision]/[artifact]-[revision].[ext]"
            m2compatible = true
        }
    }
}
```

#### **50.6.6.4.  访问密码保护ivy存储库**

50.6.6.4. Accessing password protected Ivy repositories

访问一个ivy存储库需使用基本权限。当你在定义存储库时就要指定用户名和密码

To access an Ivy repository which uses basic authentication, you specify the username and password to use when you define the repository:

Example 50.38. Ivy repository

build.gradle
```
repositories {
    ivy {
        url 'http://repo.mycompany.com'
        credentials {
            username 'user'
            password 'password'
        }
    }
}
```
### **50.6.7. 支持的库传输协议**

50.6.7. Supported repository transport protocols

Maven和Maven存储库支持多种传输协议的使用。目前支持以下协议:

Maven and Maven repositories support the use of various transport protocols. At the moment the following protocols are supported:

Table 50.3. Repository transport protocols

|Type|	Authentication schemes|
|--
|file	|none|
|http	|username/password|
|https	|username/password|
|sftp	|username/password|
|s3	|access key/secret key|

定义一个存储库使用存储库配置块。使用库闭包,Maven存储库声明为Maven。Ivy存储库声明为Ivy。传输协议是存储库URL定义的一部分。下面的构建脚本演示如何创建一个基于http的Maven和Ivy存储库:

To define a repository use the repositories configuration block. Within the repositories closure, a Maven repository is declared with maven. An Ivy repository is declared with ivy. The transport protocol is part of the URL definition for a repository. The following build script demonstrates how to create a HTTP-based Maven and Ivy repository:

Example 50.39. Declaring a Maven and Ivy repository

build.gradle
```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }

    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```
如果需要身份验证存储库,可以提供相关的凭证。下面的例子展示了如何为SFTP库提供的用户名/密码的身份验证:

If authentication is required for a repository, the relevant credentials can be provided. The following example shows how to provide username/password-based authentication for SFTP repositories:

Example 50.40. Providing credentials to a Maven and Ivy repository

build.gradle
```
repositories {
    maven {
        url "sftp://repo.mycompany.com:22/maven2"
        credentials {
            username 'user'
            password 'password'
        }
    }

    ivy {
        url "sftp://repo.mycompany.com:22/repo"
        credentials {
            username 'user'
            password 'password'
        }
    }
}
```

当使用一个AWS S3支持库你需要使用AwsCredentials进行身份验证,提供访问密钥和密钥。下面的例子显示了如何声明一个S3支持库和提供AWS凭证

When using an AWS S3 backed repository you need to authenticate using AwsCredentials, providing access-key and a private-key. The following example shows how to declare a S3 backed repository and providing AWS credentials:

Example 50.41. Declaring a S3 backed Maven and Ivy repository


build.gradle
```
repositories {
    maven {
        url "s3://myCompanyBucket/maven2"
        credentials(AwsCredentials) {
            accessKey "someKey"
            secretKey "someSecret"
        }
    }

    ivy {
        url "s3://myCompanyBucket/ivyrepo"
        credentials(AwsCredentials) {
            accessKey "someKey"
            secretKey "someSecret"
        }
    }
}
```

#### **50.6.7.1. S3配置属性**

50.6.7.1. S3 configuration properties

以下系统属性可以用来配置与s3存储库的交互:

The following system properties can be used to configure the interactions with s3 repositories:

Table 50.4. S3 Configuration Properties

|Property	|Description|
|--
|org.gradle.s3.endpoint |	Used to override the AWS S3 endpoint when using a non AWS, S3 API compatible, storage service.|
|org.gradle.s3.maxErrorRetry|	Specifies the maximum number of times to retry a request in the event that the S3 server responds with a HTTP 5xx status code. When not specified a default value of 3 is used.|

#### **50.6.7.2.S3 URL格式**

50.6.7.2. S3 URL formats

S3 URL是'virtual-hosted-style' 必须是如下格式s3://<bucketName>[.<regionSpecificEndpoint>]/<s3Key>

S3 URL's are 'virtual-hosted-style' and must be in the following format s3://<bucketName>[.<regionSpecificEndpoint>]/<s3Key>

比如s3://myBucket.s3.eu-central-1.amazonaws.com/maven/release

e.g. s3://myBucket.s3.eu-central-1.amazonaws.com/maven/release

myBucket是 AWS S3 bucket名字

myBucket is the AWS S3 bucket name.

s3.eu-central-1.amazonaws.com 是可选的指定域名

s3.eu-central-1.amazonaws.com is the optional region specific endpoint.

/maven/release是 AWS S3要是(bucket对象的唯一标识符)

/maven/release is the AWS S3 key (unique identifier for an object within a bucket)

#### **50.6.7.3. S3代理设置**

50.6.7.3. S3 proxy settings

S3的代理可以使用以下系统属性配置:

A proxy for S3 can be configured using the following system properties:

https.proxyHost
https.proxyPort
https.proxyUser
https.proxyPassword
https.nonProxyHosts

如果'org.gradle.s3.endpoint'属性已被指定URI和http(不是https)URI可以使用系统代理设置如下：
If the 'org.gradle.s3.endpoint' property has been specified with a http (not https) URI the following system proxy settings can be used:

http.proxyHost
http.proxyPort
http.proxyUser
http.proxyPassword
http.nonProxyHosts

#### **50.6.7.4.AWS S3 V4签名**

50.6.7.4. AWS S3 V4 Signatures (AWS4-HMAC-SHA256)

某些AWS S3 区域 (eu-central-1 - Frankfurt) 要求所有HTTP请求签订AWS的签名版本4。建议当使用要求V4签名的buckets时指定S3 url包含区域特定端点。比如s3://somebucket.s3.eu-central-1.amazonaws.com/maven/release

Some of the AWS S3 regions (eu-central-1 - Frankfurt) require that all HTTP requests are signed in accordance with AWS's signature version 4. It is recommended to specify S3 URL's containing the region specific endpoint when using buckets that require V4 signatures. e.g. s3://somebucket.s3.eu-central-1.amazonaws.com/maven/release

注意:当没有为要求V4签名的buckets指定区域端点,Gradle将使用默认的AWS区域(us-east-1)以下警告会出现在控制台:

NOTE: When a region-specific endpoint is not specified for buckets requiring V4 Signatures, Gradle will 

use the default AWS region (us-east-1) and the following warning will appear on the console:
Attempting to re-send the request to .... with AWS V4 authentication. To avoid this warning in the future, please use region-specific endpoint to access buckets located in regions that require V4 signing.

未能为要求V4签名的buckets指定特定区域端点意味着:

Failing to specify the region-specific endpoint for buckets requiring V4 signatures means:

3 round-trips to AWS, as opposed to one, for every file upload and download.
Depending on location - increased network latencies and slower builds.
Increased likelihood of transmission failures.

### **50.6.8. Working with repositories**

To access a repository:

Example 50.42. Accessing a repository

build.gradle
```
println repositories.localRepository.name
println repositories['localRepository'].name
To configure a repository:

Example 50.43. Configuration of a repository

build.gradle
repositories {
    flatDir {
        name 'localRepository'
    }
}
repositories {
    localRepository {
        dirs 'lib'
    }
}
repositories.localRepository {
    dirs 'lib'
}
```

### **50.6.9.更多关于Ivy解析器**

50.6.9. More about Ivy resolvers

关于存储库:Gradle是非常灵活的

Gradle is extremely flexible regarding repositories:

有很多种协议与存储库通信的选择(如文件系统、http、ssh sftp…) 　　
协议sftp目前仅支持用户名/密码的身份验证。　　每个存储库都有它自己的布局。　　
比方说,你声明一个依赖在junit:junit:3.8.2库。现在Gradle如何在库里找到它?在某种程度上依赖信息必须映射到一个路径。与Maven相比,这条路径是固定的,用Gradle您可以定义一个模式定义路径的样子。这里有一些例子:[16]

There are many options for the protocol to communicate with the repository (e.g. filesystem, http, ssh, sftp ...)
The protocol sftp currently only supports username/password-based authentication.
Each repository can have its own layout.
Let's say, you declare a dependency on the junit:junit:3.8.2 library. Now how does Gradle find it in the repositories? Somehow the dependency information has to be mapped to a path. In contrast to Maven, where this path is fixed, with Gradle you can define a pattern that defines what the path will look like. Here are some examples: [16]

// Maven2 layout (if a repository is marked as Maven2 compatible, the organization (group) is split into subfolders according to the dots.)
someroot/[organisation]/[module]/[revision]/[module]-[revision].[ext]

// Typical layout for an Ivy repository (the organization is not split into subfolder)
someroot/[organisation]/[module]/[revision]/[type]s/[artifact].[ext]

// Simple layout (the organization is not used, no nested folders.)
someroot/[artifact]-[revision].[ext]
To add any kind of repository (you can pretty easy write your own ones) you can do:

Example 50.44. Definition of a custom repository

build.gradle
```
repositories {
    ivy {
        ivyPattern "$projectDir/repo/[organisation]/[module]-ivy-[revision].xml"
        artifactPattern "$projectDir/repo/[organisation]/[module]-[revision](-[classifier]).[ext]"
    }
}
```

由Ivy当然也是Gradle提供的解析概况可以在这里找到。使用Gradle你不必通过XML配置而可以直接通过API。

An overview of which Resolvers are offered by Ivy and thus also by Gradle can be found here. With Gradle you just don't configure them via XML but directly via their API.

## **50.7.依赖性解析是如何工作的**

50.7. How dependency resolution works

Gradle取得依赖声明和存储库定义 试图下载所有的依赖关系通过一个名为依赖解析的进程。下面是一个简短的概述关于这一进程是如何工作的。

Gradle takes your dependency declarations and repository definitions and attempts to download all of your dependencies by a process called dependency resolution. Below is a brief outline of how this process works.

给定一个所需依赖,Gradle首先试图解析该依赖的模块。顺序检查每个存储库,首先寻找到一个模块描述符文件(POM或Ivy文件)其可以表明该模块的存在。如果没有找到模块描述符,Gradle将搜索主要模块产物的存在，其表明该模块存在于存储库。

Given a required dependency, Gradle first attempts to resolve the module for that dependency. Each repository is inspected in order, searching first for a module descriptor file (POM or Ivy file) that indicates the presence of that module. If no module descriptor is found, Gradle will search for the presence of the primary module artifact file indicating that the module exists in the repository.

如果依赖声明为一个动态的版本(比如1.+)，Gradle将解析这个到存储库中最新的可用静态版本(如1.2)。对于Maven存储库,这是使用maven-metadata.xml文件完成的。然而对于Ivy库是通过目录清单完成的。

If the dependency is declared as a dynamic version (like 1.+), Gradle will resolve this to the newest available static version (like 1.2) in the repository. For Maven repositories, this is done using the maven-metadata.xml file, while for Ivy repositories this is done by directory listing.

如果模块描述符是一个POM文件,有一个父亲POM已声明,Gradle将递归地企图解析每个POM的父模块。　　一旦检查模块每个库,它将选择要使用的“最好的”一个。这样做使用以下标准: 　　动态版本,“更高”的静态版本是优先于“低”版本。

If the module descriptor is a POM file that has a parent POM declared, Gradle will recursively attempt to resolve each of the parent modules for the POM.
Once each repository has been inspected for the module, Gradle will choose the 'best' one to use. This is done using the following criteria:

对于一个动态版本，高静态版本优于较低版本。

For a dynamic version, a 'higher' static version is preferred over a 'lower' version.

用模块描述文件（Ivy或POM文件）声明的模块优于仅有产物文件的模块

Modules declared by a module descriptor file (Ivy or POM file) are preferred over modules that have an artifact file only.

早些时候库模块优于晚些时候的库的模块

Modules from earlier repositories are preferred over modules in later repositories.

当依赖由一个静态版本声明并且库中存在模块描述文件时，没有必要继续搜索后面的库且进程剩下的部分将终止。

When the dependency is declared by a static version and a module descriptor file is found in a repository, there is no need to continue searching later repositories and the remainder of the process is short-circuited.

模块中所有的产物之后将会从上面进程中选择的库中被请求。

All of the artifacts for the module are then requested from the same repository that was chosen in the process above.

## **50.8.  微调相关性解析过程**

50.8. Fine-tuning the dependency resolution process

在大多数情况下,Gradle的默认依赖关系管理将解析你构建中想要的依赖关系。在某些情况下,然而,有必要调整解析以确保你的构建接受正确的依赖关系。

In most cases, Gradle's default dependency management will resolve the dependencies that you want in your build. In some cases, however, it can be necessary to tweak dependency resolution to ensure that your build receives exactly the right dependencies.

有许多方式可以使你影响Gradle解析依赖项。

There are a number of ways that you can influence how Gradle resolves dependencies.

### **50.8.1.强制一个特定模块的版本**

50.8.1. Forcing a particular module version

强制一个模块版本告诉Gradle总是使用一个特定的版本依赖(传递),覆盖一个发布模块描述符中指定的任何版本。这可以在解决版本冲突时非常有用,更多信息参见50.2.3节“解决版本冲突”。

Forcing a module version tells Gradle to always use a specific version for given dependency (transitive or not), overriding any version specified in a published module descriptor. This can be very useful when tackling version conflicts - for more information see Section 50.2.3, “Resolve version conflicts”.

强制版本也可以用来对付传递依赖项的流氓元数据。如果传递依赖元数据质量差导致在依赖解析时间出现问题,这时可以使用一个新的,此依赖的固定版本。例如,查看ResolutionStrategy类的API文档。注意,“依赖解决规则”(下面概述的)提供一个更强大的机制来取代一个破碎的模块依赖项。请参见50.8.2.3“黑名单特定版本替换”。

Force versions can also be used to deal with rogue metadata of transitive dependencies. If a transitive dependency has poor quality metadata that leads to problems at dependency resolution time, you can force Gradle to use a newer, fixed version of this dependency. For an example, see the ResolutionStrategy class in the API documentation. Note that 'dependency resolve rules' (outlined below) provide a more powerful mechanism for replacing a broken module dependency. See Section 50.8.2.3, “Blacklisting a particular version with a replacement”.


### **50.8.2 使用依赖项解析规则**

50.8.2. Using dependency resolve rules

每个解决依赖关系都执行依赖解决规则,它提供了一个强大的api在依赖被解析之前操纵一个请求的依赖。这个特性在开发优化中,但目前提供的能力可以改变组,名称和/或请求依赖的版本,以及允许依赖在解析期间被一个完全不同的模块替换。

A dependency resolve rule is executed for each resolved dependency, and offers a powerful api for manipulating a requested dependency prior to that dependency being resolved. This feature is incubating, but currently offers the ability to change the group, name and/or version of a requested dependency, allowing a dependency to be substituted with a completely different module during resolution.

依赖性解析规则提供一个非常强大的方式来控制依赖项解析过程,并可用于实现各种先进模式依赖关系管理。其中一些模式下面概述。更多信息和代码示例见ResolutionStrategy类的API文档。

Dependency resolve rules provide a very powerful way to control the dependency resolution process, and can be used to implement all sorts of advanced patterns in dependency management. Some of these patterns are outlined below. For more information and code samples see the ResolutionStrategy class in the API documentation.

#### **50.8.2.1.塑造发布单元**

50.8.2.1. Modelling releaseable units

通常一个组织使用一个单独的版本发布一系列库,在这里库一起被构建、测试和发布。这些库形成一个可发布的单元,设计旨在作为一个整体。使用来自不同的发布单元的库是没有意义的。

Often an organisation publishes a set of libraries with a single version; where the libraries are built, tested and published together. These libraries form a 'releasable unit', designed and intended to be used as a whole. It does not make sense to use libraries from different releasable units together.

但是对于传递性依赖解析很容易违背这个约定，比如

But it is easy for transitive dependency resolution to violate this contract. For example:

module-a depends on releasable-unit:part-one:1.0
module-b depends on releasable-unit:part-two:1.1

在发布单元里既依赖于module-a 又依赖于module-b的构建会得到不同的库版本

A build depending on both module-a and module-b will obtain different versions of libraries within the releasable unit.

依赖性解析规则给你强制执行构建中可发布单元的权力。想象一个由所有拥有'org.gradle'组的库定义的可发布单元。我们可以强制所有这样库使用一致的版本

Dependency resolve rules give you the power to enforce releasable units in your build. Imagine a releasable unit defined by all libraries that have 'org.gradle' group. We can force all of these libraries to use a consistent version:

Example 50.45. Forcing consistent version for a group of libraries

build.gradle
```
configurations.all {
    resolutionStrategy.eachDependency { DependencyResolveDetails details ->
        if (details.requested.group == 'org.gradle') {
            details.useVersion '1.4'
        }
    }
}
```

#### **50.8.2.2.实现一个自定义版本控制方案**

50.8.2.2. Implement a custom versioning scheme

在一些团体环境中,Gradle构建中可声明的模块版本的列表是在外部维护和审计的。依赖性解析规则提供了一个整洁的实现模式:

In some corporate environments, the list of module versions that can be declared in Gradle builds is maintained and audited externally. Dependency resolve rules provide a neat implementation of this pattern:

在构建脚本中,开发人员用模块组和名称声明依赖性,但使用占位符版本,例如:“default”。　默认的版本通过依赖解决规则被解析成一个特定的版本,它会在公共批准模块的目录中查找版本。
这个规则实现可以整齐地封装在一个企业插件里,组织内部所有建立可以共享。

In the build script, the developer declares dependencies with the module group and name, but uses a placeholder version, for example: 'default'.
The 'default' version is resolved to a specific version via a dependency resolve rule, which looks up the version in a corporate catalog of approved modules.
This rule implementation can be neatly encapsulated in a corporate plugin, and shared across all builds within the organisation.

Example 50.46. Using a custom versioning scheme

build.gradle
```
configurations.all {
    resolutionStrategy.eachDependency { DependencyResolveDetails details ->
        if (details.requested.version == 'default') {
            def version = findDefaultVersionInCatalog(details.requested.group, details.requested.name)
            details.useVersion version
        }
    }
}

def findDefaultVersionInCatalog(String group, String name) {
    //some custom logic that resolves the default version into a specific version
    "1.0"
}
```

### **50.8.2.3. 使用替代黑名单一个特定版本**

50.8.2.3. Blacklisting a particular version with a replacement

依赖性解析规则提供这样的机制：黑名单一个依赖的特定版本以及提供一个替代版本。这可能是有用的,当某种依赖版本破碎而不应该被使用时,依赖解析规则可使得这个版本被替换为一个已知的好版本。对于损伤模块的一个例子它声明的依赖在一个任何公共存储库无法找到的库里,但还有许多其他原因导致一个特定的模块版本是不想要的以及更想要不同的版本。

Dependency resolve rules provide a mechanism for blacklisting a particular version of a dependency and providing a replacement version. This can be useful if a certain dependency version is broken and should not be used, where a dependency resolve rule causes this version to be replaced with a known good version. One example of a broken module is one that declares a dependency on a library that cannot be found in any of the public repositories, but there are many other reasons why a particular module version is unwanted and a different version is preferred.

下面的例子,假设1.2.1版本包含重要的补丁且相对于1.2应该优先使用。提供的规则将强制执行:任何时候遇到1.2版本将被替换为1.2.1。注意,这是不同于如上所述的强制版本,这个模块任何其他版本的将不会受到影响。这意味着“最新”冲突解决策略仍将选择1.3版本当会这个版本过时的时候。

In example below, imagine that version 1.2.1 contains important fixes and should always be used in preference to 1.2. The rule provided will enforce just this: any time version 1.2 is encountered it will be replaced with 1.2.1. Note that this is different from a forced version as described above, in that any other versions of this module would not be affected. This means that the 'newest' conflict resolution strategy would still select version 1.3 if this version was also pulled transitively.

Example 50.47. Blacklisting a version with a replacement

build.gradle
```
configurations.all {
    resolutionStrategy.eachDependency { DependencyResolveDetails details ->
        if (details.requested.group == 'org.software' && details.requested.name == 'some-library' && details.requested.version == '1.2') {
            //prefer different version which contains some necessary fixes
            details.useVersion '1.2.1'
        }
    }
}
```

#### **50.8.2.4.兼容替换一个依赖模块**

50.8.2.4. Substituting a dependency module with a compatible replacement

有时一个完全不同的模块可以作为一个请求模块依赖关系的替代品。例子包括使用“groovy”代替“groovy-all”,或使用“log4j-over-slf4j”替代“log4j”。从Gradle 1.5可以使用依赖解析规则来做这些替换:

At times a completely different module can serve as a replacement for a requested module dependency. Examples include using 'groovy' in place of 'groovy-all', or using 'log4j-over-slf4j' instead of 'log4j'. Starting with Gradle 1.5 you can make these substitutions using dependency resolve rules:

Example 50.48. Changing dependency group and/or name at the resolution

build.gradle
```
configurations.all {
    resolutionStrategy.eachDependency { DependencyResolveDetails details ->
        if (details.requested.name == 'groovy-all') {
            //prefer 'groovy' over 'groovy-all':
            details.useTarget group: details.requested.group, name: 'groovy', version: details.requested.version
        }
        if (details.requested.name == 'log4j') {
            //prefer 'log4j-over-slf4j' over 'log4j', with fixed version:
            details.useTarget "org.slf4j:log4j-over-slf4j:1.7.10"
        }
    }
}
```

### **50.8.3.依赖替代规则**

50.8.3. Dependency Substitution Rules

依赖替换规则同依赖解析规则工作类似。事实上,许多依赖解析功能与依赖的规则可以用依赖替换规则实现。他们允许项目和模块依赖关系被透明地替换为指定的替代品。与依赖解析规则不同,依赖替代规则允许项目和模块依赖关系可交换替代。

Dependency substitution rules work similarly to dependency resolve rules. In fact, many capabilities of dependency resolve rules can be implemented with dependency substitution rules. They allow project and module dependencies to be transparently substituted with specified replacements. Unlike dependency resolve rules, dependency substitution rules allow project and module dependencies to be substituted interchangeably.

注意:添加一个依赖项替换规则到配置更改了配置解析时的时机。配置不是第一次使用时解析,而是在任务图构建的时候被解析。这可能产生意想不到的后果,如果配置任务执行期间被进一步修改,或如果配置依赖于其他任务执行期间发布的模块。

NOTE: Adding a dependency substitution rule to a configuration changes the timing of when that configuration is resolved. Instead of being resolved on first use, the configuration is instead resolved when the task graph is being constructed. This can have unexpected consequences if the configuration is being further modified during task execution, or if the configuration relies on modules that are published during execution of another task.

To explain:

配置可以声明作为一个到任何任务的输入,并且此配置可以在解析时包括项目依赖
如果一个项目的依赖是一个任务的输入(通过配置),那么构建项目工件的任务必须被添加到任务依赖关系。
为了确定项目输入一个任务的依赖关系,Gradle需要解析配置的输入。
因为Gradle任务图固定，一旦任务执行开始后,Gradle需要执行这解析优先于执行任何任务。

A Configuration can be declared as an input to any Task, and that configuration can include project dependencies when it is resolved.
If a project dependency is an input to a Task (via a configuration), then tasks to built the project artifacts must be added to the task dependencies.
In order to determine the project dependencies that are inputs to a task, Gradle needs to resolve the Configuration inputs.
Because the Gradle task graph is fixed once task execution has commenced, Gradle needs to perform this resolution prior to executing any tasks.

没有依赖替换规则,它永远不会知道外部模块依赖会引用一个项目依赖。这使得很容易通过简单的图遍历确定项目依赖项的全套配置。使用此功能,Gradle可以不再做这样的假设,并为了确定项目依赖关系必须执行一个完整的解析。

In the absence of dependency substitution rules, Gradle knows that an external module dependency will never transitively reference a project dependency. This makes it easy to determine the full set of project dependencies for a configuration through simple graph traversal. With this functionality, Gradle can no longer make this assumption, and must perform a full resolve in order to determine the project dependencies.

#### **50.8.3.1.使用项目依赖替代外部模块依赖**

50.8.3.1. Substituting an external module dependency with a project dependency

依赖替代的一个用例是使用一个本地开发版本的模块代替一个从外部存储库下载的版本。这可能对于测试一个本地、补丁版依赖版本很有用。

One use case for dependency substitution is to use a locally developed version of a module in place of one that is downloaded from an external repository. This could be useful for testing a local, patched version of a dependency.

要替换的模块可以声明有或没有指定的版本。

The module to be replaced can be declared with or without a version specified.

Example 50.49. Substituting a module with a project

build.gradle
```
configurations.all {
    resolutionStrategy.dependencySubstitution {
        substitute module("org.utils:api") with project(":api")
        substitute module("org.utils:util:2.5") with project(":util")
    }
}
```

Note that a project that is substituted must be included in the multi-project build (via settings.gradle). Dependency substitution rules take care of replacing the module dependency with the project dependency and wiring up any task dependencies, but do not implicitly include the project in the build.

#### **50.8.3.2. 使用模块替换一个项目依赖**

50.8.3.2. Substituting a project dependency with a module replacement

使用替换规则的另一种方法是多项目构建时更换一个项目与一个模块的依赖。这可能对于大型多项目建设加快发展是游泳的,其通过允许项目依赖项的子集从存储库下载,而不是构建。

Another way to use substitution rules is to replace a project dependency with a module in a multi-project build. This can be useful to speed up development with a large multi-project build, by allowing a subset of the project dependencies to be downloaded from a repository rather than being built.
用作替代品的模块必须声明指定的版本。
The module to be used as a replacement must be declared with a version specified.

Example 50.50. Substituting a project with a module

build.gradle
```
configurations.all {
    resolutionStrategy.dependencySubstitution {
        substitute project(":api") with module("org.utils:api:1.3")
    }
}
```

当一个项目依赖被替换为一个模块依赖,该项目还包括在整个多项目建设里。然而,构建取代依赖任务不会为了建立解析执行依赖配置。

When a project dependency has been replaced with a module dependency, that project is still included in the overall multi-project build. However, tasks to build the replaced dependency will not be executed in order to build the resolve the depending Configuration.

#### **50.8.3.3.有条件地代替依赖**

50.8.3.3. Conditionally substituting a dependency

依赖替代常见的使用例子是允许在多项目构建中更灵活的组装子项目。这对于在一个大型多项目开发一个本地、外部依赖项补丁版本或构建子模块会很有用。

A common use case for dependency substitution is to allow more flexible assembly of sub-projects within a multi-project build. This can be useful for developing a local, patched version of an external dependency or for building a subset of the modules within a large multi-project build.

下面的示例使用一个依赖替代规则通过组 "org.example"取代任何模块依赖,但前提是本地的项目匹配依赖名称才可以被定位。

The following example uses a dependency substitution rule to replace any module dependency with the group "org.example", but only if a local project matching the dependency name can be located.

Example 50.50. Conditionally substituting a dependency

build.gradle
```
configurations.all {
    resolutionStrategy.dependencySubstitution.all { DependencySubstitution dependency ->
        if (dependency.requested instanceof ModuleComponentSelector && dependency.requested.group == "org.example") {
            def targetProject = findProject(":${dependency.requested.module}")
            if (targetProject != null) {
                dependency.useTarget targetProject
            }
        }
    }
}
```

注意,一个被替换的项目必须包含在多项目建设里(通过settings.gradle)。依赖替代规则照顾更换模块依赖与项目的依赖,但不隐式地包含项目的构建。

Note that a project that is substituted must be included in the multi-project build (via settings.gradle). Dependency substitution rules take care of replacing the module dependency with the project dependency, but do not implicitly include the project in the build.

### **50.8.4. 指定默认的配置依赖关系**

50.8.4. Specifying default dependencies for a configuration

配置可以使用默认配置依赖如果没有显式地设置配置依赖项。此功能的主要用例是使用版本控制工具的开发插件用户可能会覆盖。通过指定默认依赖性,只要用户没有指定使用一个特定版本插件可以使用一个默认的工具版本。

A configuration can be configured with default dependencies to be used if no dependencies are explicitly set for the configuration. A primary use case of this functionality is for developing plugins that make use of versioned tools that the user might override. By specifying default dependencies, the plugin can use a default version of the tool only if the user has not specified a particular version to use.

Example 50.52. Specifying default dependencies on a configuration

build.gradle
```
configurations {
    pluginTool {
        defaultDependencies { dependencies ->
            dependencies.add(this.project.dependencies.create("org.gradle:my-util:1.0"))
        }
    }
}
```

### **50.8.5.Ivy动态解析模式**

50.8.5. Enabling Ivy dynamic resolve mode

Gradle的Ivy库实现支持等同于Ivy的动态解析模式。通常情况下,它将为每个依赖定义使用rev属性包含在ivy.xml文件。在动态解决模式,对于一个给定的依赖定义Gradle将更喜欢revConstraint属性相对rev属性。如果revConstraint属性不存在,则使用rev属性。

Gradle's Ivy repository implementations support the equivalent to Ivy's dynamic resolve mode. Normally, Gradle will use the rev attribute for each dependency definition included in an ivy.xml file. In dynamic resolve mode, Gradle will instead prefer the revConstraint attribute over the rev attribute for a given dependency definition. If the revConstraint attribute is not present, the rev attribute is used instead.

启用动态解决模式,您需要设置适当的选择存储库定义。如下所示几个例子。注意动态解析模式只有Gradle的Ivy的存储库。它不适用于Maven存储库,或自定义Ivy DependencyResolver实现。

To enable dynamic resolve mode, you need to set the appropriate option on the repository definition. A couple of examples are shown below. Note that dynamic resolve mode is only available for Gradle's Ivy repositories. It is not available for Maven repositories, or custom Ivy DependencyResolver implementations.

Example 50.53. Enabling dynamic resolve mode

build.gradle
```
// Can enable dynamic resolve mode when you define the repository
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        resolve.dynamicMode = true
    }
}

// Can use a rule instead to enable (or disable) dynamic resolve mode for all repositories
repositories.withType(IvyArtifactRepository) {
    resolve.dynamicMode = true
}
```

### **50.8.6.组件的元数据规则**

50.8.6. Component metadata rules

每个模块(也称为组件)具有与之相关的元数据,比如组名称,版本,依赖关系等等。元数据通常来源于模块的描述符。模块的元数据的元数据规则允许某些部分从构建脚本中操纵。他们在一个模块的描述符被下载后生效,但在这之前已经在所有候选版本中被选择了。这使得元数据规则成为自定义依赖项解析的另一个工具。

Each module (also called component) has metadata associated with it, such as its group, name, version, dependencies, and so on. This metadata typically originates in the module's descriptor. Metadata rules allow certain parts of a module's metadata to be manipulated from within the build script. They take effect after a module's descriptor has been downloaded, but before it has been selected among all candidate versions. This makes metadata rules another instrument for customizing dependency resolution.

一个模块的元数据,Gradle理解是一个模块的状态方案。这个概念,也是从Ivy,塑造了不同级别的模块转换通过成熟度。默认状态计划,命令从最小最成熟的状态,是集成、里程碑,释放。除了状态方案,模块也有一个(当前)状态,必须在其状态值之一。如果在描述符中未指定,默认状态为Ivy模块和Maven快照模块集成,并释放没有快照的Maven模块。

One piece of module metadata that Gradle understands is a module's status scheme. This concept, also known from Ivy, models the different levels of maturity that a module transitions through over time. The default status scheme, ordered from least to most mature status, is integration, milestone, release. Apart from a status scheme, a module also has a (current) status, which must be one of the values in its status scheme. If not specified in the (Ivy) descriptor, the status defaults to integration for Ivy modules and Maven snapshot modules, and release for Maven modules that aren't snapshots.

模块的状态和状态计划再最新版本选择符被解析时考虑。具体来说,latest.someStatus将解析成最高的模块版本状态即有状态someStatus或更成熟的状态。例如,使用默认状态计划到位,latest.integration将选择最高的模块版本无论它的状态(因为集成是最不成熟的状态),而latest.release发布将选择最高的模块版本释放状态。下面是代码中的样子:

A module's status and status scheme are taken into consideration when a latest version selector is resolved. Specifically, latest.someStatus will resolve to the highest module version that has status someStatus or a more mature status. For example, with the default status scheme in place, latest.integration will select the highest module version regardless of its status (because integration is the least mature status), whereas latest.release will select the highest module version with status release. Here is what this looks like in code:

Example 50.54. 'Latest' version selector

build.gradle
```
dependencies {
    config1 "org.sample:client:latest.integration"
    config2 "org.sample:client:latest.release"
}

task listConfigs << {
    configurations.config1.each { println it.name }
    println()
    configurations.config2.each { println it.name}
}
Output of gradle -q listConfigs
> gradle -q listConfigs
client-1.5.jar

client-1.4.jar
```

The next example demonstrates latest selectors based on a custom status scheme declared in a component metadata rule that applies to all modules:

Example 50.55. Custom status scheme

build.gradle
```
dependencies {
    config3 "org.sample:api:latest.silver"
    components {
        all { ComponentMetadataDetails details ->
            if (details.id.group == "org.sample" && details.id.name == "api") {
                details.statusScheme = ["bronze", "silver", "gold", "platinum"]
            }
        }
    }
}
```

组件的元数据规则可以应用到指定的模块。必须指定模块的形式为"group:module"。

Component metadata rules can be applied to a specified module. Modules must be specified in the form of "group:module".

Example 50.56. Custom status scheme by module

build.gradle
```
dependencies {
    config4 "org.sample:lib:latest.prod"
    components {
        withModule('org.sample:lib') { ComponentMetadataDetails details ->
            details.statusScheme = ["int", "rc", "prod"]
        }
    }
}
```

Gradle还可以创建组件的元数据规则通过使用Ivy特有的元数据模块从Ivy 库解决。Ivy描述符的值可以通过IvyModuleDescriptor接口获得。

Gradle can also create component metadata rules utilizing Ivy-specific metadata for modules resolved from an Ivy repository. Values from the Ivy descriptor are made available via the IvyModuleDescriptor interface.

Example 50.57. Ivy component metadata rule

build.gradle
```
dependencies {
    config6 "org.sample:lib:latest.rc"
    components {
        withModule("org.sample:lib") { ComponentMetadataDetails details, IvyModuleDescriptor ivyModule ->
            if (ivyModule.branch == 'testing') {
                details.status = "rc"
            }
        }
    }
}
```

注意,任何声明特定的参数的规则必须包括ComponentMetadataDetails参数作为第一个参数。第二个Ivy元数据参数是可选的。

Note that any rule that declares specific arguments must always include a ComponentMetadataDetails argument as the first argument. The second Ivy metadata argument is optional.

组件的元数据规则也可以使用一个规则源对象定义。一个规则源对象是任何对象包含一个定义了规则操作并用@Mutate注释的方法。

Component metadata rules can also be defined using a rule source object. A rule source object is any object that contains exactly one method that defines the rule action and is annotated with @Mutate.

This method:

must return void.
must have ComponentMetadataDetails as the first argument.
may have an additional parameter of type IvyModuleDescriptor.
Example 50.58. Rule source component metadata rule

build.gradle
```
dependencies {
    config5 "org.sample:api:latest.gold"
    components {
        withModule('org.sample:api', new CustomStatusRule())
    }
}

class CustomStatusRule {
    @Mutate
    void setStatusScheme(ComponentMetadataDetails details) {
        details.statusScheme = ["bronze", "silver", "gold", "platinum"]
    }
}
```

### **50.8.7.组件选择规则***

50.8.7. Component Selection Rules

组件选择规则可能影响哪个组件实例被选择当多个版本与选择器匹配时。规则应用到每一个可用的版本并且允许使用该规则显式拒绝版本。这允许Gradle忽略任何不满足设定的条件规则的组件实例。例子包括:

Component selection rules may influence which component instance should be selected when multiple versions are available that match a version selector. Rules are applied against every available version and allow the version to be explicitly rejected by rule. This allows Gradle to ignore any component instance that does not satisfy conditions set by the rule. Examples include:

对于一个像'1.+'的特定版本可能会在选择时被显式拒绝。

For a dynamic version like '1.+' certain versions may be explicitly rejected from selection

对于一个静态版本如 '1.4' 一个实例可能被拒绝基于额外的组件元数据,如Ivy分支属性,允许从随后的存储库使用一个实例。

For a static version like '1.4' an instance may be rejected based on extra component metadata such as the Ivy branch attribute, allowing an instance from a subsequent repository to be used.

通过ComponentSelectionRules对象配置规则。每个配置的规则将被ComponentSelection对象作为参数来调用,其中包含考虑的候选版本信息。调用ComponentSelection.reject()将导致给定候选版本被显式拒绝,在这种情况下,选择器将不考虑该候选。

Rules are configured via the ComponentSelectionRules object. Each rule configured will be called with a ComponentSelection object as an argument which contains information about the candidate version being considered. Calling ComponentSelection.reject() causes the given candidate version to be explicitly rejected, in which case the candidate will not be considered for the selector.

下面的示例显示了一个规则,它不允许一个模块特定版本但允许动态版本选择下一个最佳候选。

The following example shows a rule that disallows a particular version of a module but allows the dynamic version to choose the next best candidate.

Example 50.59. Component selection rule

build.gradle
```
configurations {
    rejectConfig {
        resolutionStrategy {
            componentSelection {
                // Accept the highest version matching the requested version that isn't '1.5'
                all { ComponentSelection selection ->
                    if (selection.candidate.group == 'org.sample' && selection.candidate.module == 'api' && selection.candidate.version == '1.5') {
                        selection.reject("version 1.5 is broken for 'org.sample:api'")
                    }
                }
            }
        }
    }
}

dependencies {
    rejectConfig "org.sample:api:1.+"
}
```

注意,版本选择应用首先从最高的版本。被选择的版本可能是所有组件选择规则接受的第一个版本。当一个版本没有被规则明确拒绝则认为此版本被接受。

Note that version selection is applied starting with the highest version first. The version selected will be the first version found that all component selection rules accept. A version is considered accepted no rule explicitly rejects it.

同样,规则可以针对特定模块。必须指定模块的形式"group:module"。

Similarly, rules can be targeted at specific modules. Modules must be specified in the form of "group:module".

Example 50.60. Component selection rule with module target

build.gradle
```
configurations {
    targetConfig {
        resolutionStrategy {
            componentSelection {
                withModule("org.sample:api") { ComponentSelection selection ->
                    if (selection.candidate.version == "1.5") {
                        selection.reject("version 1.5 is broken for 'org.sample:api'")
                    }
                }
            }
        }
    }
}
```

组件选择规则也可在选择一个版本时考虑组件的元数据,被考虑的元数据参数可能是ComponentMetadata和IvyModuleDescriptor。

Component selection rules can also consider component metadata when selecting a version. Possible metadata arguments that can be considered are ComponentMetadata and IvyModuleDescriptor.

Example 50.61. Component selection rule with metadata

build.gradle
```
configurations {
    metadataRulesConfig {
        resolutionStrategy {
            componentSelection {
                // Reject any versions with a status of 'experimental'
                all { ComponentSelection selection, ComponentMetadata metadata ->
                    if (selection.candidate.group == 'org.sample' && metadata.status == 'experimental') {
                        selection.reject("don't use experimental candidates from 'org.sample'")
                    }
                }
                // Accept the highest version with either a "release" branch or a status of 'milestone'
                withModule('org.sample:api') { ComponentSelection selection, IvyModuleDescriptor descriptor, ComponentMetadata metadata ->
                    if (descriptor.branch != "release" && metadata.status != 'milestone') {
                        selection.reject("'org.sample:api' must have testing branch or milestone status")
                    }
                }
            }
        }
    }
}
```
注意ComponentSelection参数总是需要作为第一个参数当使用其他Ivy元数据声明组件选择规则时,但元数据参数可以在任何顺序声明。

Note that a ComponentSelection argument is always required as the first parameter when declaring a component selection rule with additional Ivy metadata parameters, but the metadata parametesrs can be declared in any order.

最后,组件选择规则也可以使用一个规则源对象定义。一个规则源对象是任何对象包含一个定义了规则操作并用@Mutate注释的方法。

Lastly, component selection rules can also be defined using a rule source object. A rule source object is any object that contains exactly one method that defines the rule action and is annotated with @Mutate.

This method:

must return void.
must have ComponentSelection as the first argument.
may have additional parameters of type ComponentMetadata and/or IvyModuleDescriptor.
Example 50.62. Component selection rule using a rule source object

build.gradle
```
class RejectTestBranch {
    @Mutate
    void evaluateRule(ComponentSelection selection, IvyModuleDescriptor ivy) {
        if (ivy.branch == "test") {
            selection.reject("reject test branch")
        }
    }
}

configurations {
    ruleSourceConfig {
        resolutionStrategy {
            componentSelection {
                all new RejectTestBranch()
            }
        }
    }
}
```

### **50.8.8. 模块替换规则**

50.8.8. Module replacement rules

模块替换规则允许构建声明遗产库已经被新的取代。一个很好的例子是,当一个新库取代遗留的一个是"google-collections" -> "guava" 迁移。创建google-collections的团队决定改变模块名称从"com.google.collections:google-collections" 到 "com.google.guava:guava"。这是合法的行业场景:团队需要能够改变他们维护的产品名字,包括模块的坐标。重命名模块的坐标对解决冲突有一定的影响。

Module replacement rules allow a build to declare that a legacy library has been replaced by a new one. A good example when a new library replaced a legacy one is the "google-collections" -> "guava" migration. The team that created google-collections decided to change the module name from "com.google.collections:google-collections" into "com.google.guava:guava". This a legal scenario in the industry: teams need to be able to change the names of products they maintain, including the module coordinates. Renaming of the module coordinates has impact on conflict resolution.

为了解释对解决冲突的影响,让我们考虑一下"google-collections" -> "guava"场景。可能会发生两个库被拉到相同的依赖图中。例如,“我们”项目取决于guava但我们的一些依赖进入google-collections遗留的版本。这可能会导致运行时错误,例如在测试或应用程序执行。

To explain the impact on conflict resolution, let's consider the "google-collections" -> "guava" scenario. It may happen that both libraries are pulled into the same dependency graph. For example, "our" project depends on guava but some of our dependencies pull in a legacy version of google-collections. This can cause runtime errors, for example during test or application execution. 

Gradle不会自动解决google-collections VS guava 
冲突,因为它不被认为是“版本冲突”。这是因为模块坐标库是完全不同的和冲突解析被激活当“组”和“名称”坐标是相同的但依赖图有不同的版本(更多信息,请参阅冲突解决部分)。这个问题传统的补救措施是:

Gradle does not automatically resolve the google-collections VS guava conflict because it is not considered as a "version conflict". It's because the module coordinates for both libraries are completely different and conflict resolution is activated when "group" and "name" coordinates are the same but there are different versions available in the dependency graph (for more info, please refer to the section on conflict resolution). Traditional remedies to this problem are:

声明排除规则来避免把“google-collections”放到图里。这可能是最受欢迎的方法。

Declare exclusion rule to avoid pulling in "google-collections" to graph. It is probably the most popular approach.

避免依赖项放入遗产库。

Avoid dependencies that pull in legacy libraries.

如果新版本不再遗留库升级依赖版本。

Upgrade the dependency version if the new version no longer pulls in a legacy library.

降级到"google-collections"。不推荐，仅提及完整性

Downgrade to "google-collections". It's not recommended, just mentioned for completeness.

传统方法工作但不够普遍。例如,一个组织想要解决在所有项目中的google-collections VS guava冲突问题。从Gradle 2.2可以声明某些模块被其他所取代。这使得组织在公司插件套件中包含组件替代信息和整体解决企业所有Gradle-powered项目问题。

Traditional approaches work but they are not general enough. For example, an organisation wants to resolve the google-collections VS guava conflict resolution problem in all projects. Starting from Gradle 2.2 it is possible to declare that certain module was replaced by other. This enables organisations to include the information about module replacement in the corporate plugin suite and resolve the problem holistically for all Gradle-powered projects in the enterprise.

Example 50.63. Declaring module replacement

build.gradle
```
dependencies {
    modules {
        module("com.google.collections:google-collections") {
            replacedBy("com.google.guava:guava")
        }
    }
}
```

更多的例子和详细的API,请参阅DSL ComponentMetadataHandler的参考。

For more examples and detailed API, please refer to the DSL reference for ComponentMetadataHandler.

当我们声明“google-collections”被"guava"取代会发生什么?Gradle可以使用此信息来解决冲突。Gradle将认为"guava"的每一个版本都比“google-collections”的任何版本更新更好。此外,Gradle将确保只有guavajar在类路径/解决文件列表中存在。请注意,如果只有“google-collections”出现在依赖图(如没有"guava")Gradle不会急切地用"guava"取而代之。模块替换是一种Gradle用于解决冲突的信息。如果没有冲突(如只有“google-collections”或"guava"在图中)替代信息不会被使用。

What happens when we declare that "google-collections" are replaced by "guava"? Gradle can use this information for conflict resolution. Gradle will consider every version of "guava" newer/better than any version of "google-collections". Also, Gradle will ensure that only guava jar is present in the classpath / resolved file list. Please note that if only "google-collections" appears in the dependency graph (e.g. no "guava") Gradle will not eagerly replace it with "guava". Module replacement is an information that Gradle uses for resolving conflicts. If there is no conflict (e.g. only "google-collections" or only "guava" in the graph) the replacement information is not used.

目前是不可能声明某些模块由一组模块替换。然而,它可以声明多个模块被单个模块替换。

Currently it is not possible to declare that certain modules is replaced by a set of modules. However, it is possible to declare that multiple modules are replaced by a single module.

## **50.9.依赖缓存**

50.9. The dependency cache

Gradle包含一个高度复杂的依赖缓存机制,寻求减少依赖解析生成的远程请求的数量,同时努力保证依赖解析的结果是正确的和可再生的。

Gradle contains a highly sophisticated dependency caching mechanism, which seeks to minimise the number of remote requests made in dependency resolution, while striving to guarantee that the results of dependency resolution are correct and reproducible.

Gradle依赖缓存包含2个关键类型的存储:

The Gradle dependency cache consists of 2 key types of storage:

基于文件的存储下载的工件,包括二进制jar以及原始下载像POM文件和Ivy文件元数据。下载的存储路径工件包括SHA1校验和,这意味着2个具有相同名称但不同的内容的产物很容易被缓存。

A file-based store of downloaded artifacts, including binaries like jars as well as raw downloaded meta-data like POM files and Ivy files. The storage path for a downloaded artifact includes the SHA1 checksum, meaning that 2 artifacts with the same name but different content can easily be cached.

　二进制存储解析模块的元数据,包括解决动态版本结果,模块描述符,和产物。　　分离从缓存中存储下载的构件元数据允许我们做一些非常强大的东西通过我们的缓存,若是一个透明的,仅文件缓存布局它将会很难用。

A binary store of resolved module meta-data, including the results of resolving dynamic versions, module descriptors, and artifacts.
Separating the storage of downloaded artifacts from the cache metadata permits us to do some very powerful things with our cache that would be difficult with a transparent, file-only cache layout.

Gradle缓存不允许本地缓存中隐藏问题和创建其他神秘并难以调试的对许多构建工具一直是种挑战的行为。这种新的行为是带宽和存储高效的方式实现的。在这样做的时候,它使企业建立可靠,重现性好。

The Gradle cache does not allow the local cache to hide problems and create other mysterious and difficult to debug behavior that has been a challenge with many build tools. This new behavior is implemented in a bandwidth and storage efficient way. In doing so, Gradle enables reliable and reproducible enterprise builds.

### **50.9.1.Gradle依赖缓存的关键特性**

50.9.1. Key features of the Gradle dependency cache

50.9.1.1. 单独的元数据缓存器

50.9.1.1. Separate metadata cache

Gradle至今在元数据缓存以二进制格式记录依赖解析的各个方面。元数据缓存中存储的信息包括:

Gradle keeps a record of various aspects of dependency resolution in binary format in the metadata cache. The information stored in the metadata cache includes:

解析动态版本的结果(例如1.+)到一个具体的版本(如1.2)。

The result of resolving a dynamic version (e.g. 1.+) to a concrete version (e.g. 1.2).

为一个特定的模块解析的模块元数据,包括模块的产物和模块依赖关系。

The resolved module metadata for a particular module, including module artifacts and module dependencies.

为一个特定的模块解析的产物元数据，包括到下载产物文件的指针

The resolved artifact metadata for a particular artifact, including a pointer to the downloaded artifact file.

一个特定的存储库中缺少一个特定的模块或产物,消除重复到一个不存在资源的访问。

The absence of a particular module or artifact in a particular repository, eliminating repeated attempts to access a resource that does not exist.

元数据缓存器中的每个条目包含一个存储库记录用于提供的信息以及时间戳,可用于缓存到期。

Every entry in the metadata cache includes a record of the repository that provided the information as well as a timestamp that can be used for cache expiry.

50.9.1.2.库缓存是独立的

50.9.1.2. Repository caches are independent

如上所述,对于每个库都有一个单独的元数据缓存。存储库是由其URL,类型和布局识别的。如果一个模块或工件还没有从这个存储库之中被解析,Gradle将试图解决库中的模块。这会涉及一个远程存储库查找,然而在很多情况下不要求下载(seeSection 50.9.1.3,“构件重用”,下文)。
如上所述,对于每个库都有一个单独的元数据缓存。存储库是由其URL,类型和布局。如果一个模块或工件还没有从这个存储库之前解决,它将试图解决模块库。这总是涉及到一个远程存储库中查找,然而在很多情况下不需要下载(seeSection 50.9.1.3,“构件重用”,下文)。

As described above, for each repository there is a separate metadata cache. A repository is identified by its URL, type and layout. If a module or artifact has not been previously resolved from this repository, Gradle will attempt to resolve the module against the repository. This will always involve a remote lookup on the repository, however in many cases no download will be required (seeSection 50.9.1.3, “Artifact reuse”, below).

如果在任何指定的库构建中无法获得所需的工件，依赖项解析将会失败,即使本地缓存有一个来自不同的检索库这个产物的副本。独立存储库允许将构建以先进的方式互相孤立，这是一个关键的特性来在任何环境中创建可靠的和可再生的构建。

Dependency resolution will fail if the required artifacts are not available in any repository specified by the build, even if the local cache has a copy of this artifact which was retrieved from a different repository. Repository independence allows builds to be isolated from each other in an advanced way that no build tool has done before. This is a key feature to create builds that are reliable and reproducible in any environment.

50.9.1.3.产物重用

50.9.1.3. Artifact reuse

下载一个工件之前,它试图确定所需的工件的校验和通过下载与该产物关联的沙文件。如果校验和可以检索得到,那么如果已经存在相同的id和校验此产物不会下载。如果不能从远程服务器获取校验和,产物将急哦下载(如果它匹配现有的工件则忽略)。

Before downloading an artifact, Gradle tries to determine the checksum of the required artifact by downloading the sha file associated with that artifact. If the checksum can be retrieved, an artifact is not downloaded if an artifact already exists with the same id and checksum. If the checksum cannot be retrieved from the remote server, the artifact will be downloaded (and ignored if it matches an existing artifact).

不仅考虑从一个不同的存储库下载的产物,Gradle还将试图重用在本地Maven资源库中找到的产物。如果一个候选产物已经通过Maven下载,并且可以匹配远程服务器声明的校验和，那么Gradle将使用此产物。

As well as considering artifacts downloaded from a different repository, Gradle will also attempt to reuse artifacts found in the local Maven Repository. If a candidate artifact has been downloaded by Maven, Gradle will use this artifact if it can be verified to match the checksum declared by the remote server.

50.9.1.4.基于存储的校验和

50.9.1.4. Checksum based storage

有可能在响应相同的工件标识符时不同的存储库提供一个不同的二进制构件。,对于Maven快照工件这是常有的事,但是也可以适用于任何发布时没有改变标识符的产物。通过基于SHA1校验和的缓存工件,Gradle能够维护相同产物的多个版本。这意味着,当对一个存储库解析时Gradle将永远不会覆盖从一个不同的库文件中缓存的产物。这样做不需要每一个存储库单独的工件文件。

It is possible for different repositories to provide a different binary artifact in response to the same artifact identifier. This is often the case with Maven SNAPSHOT artifacts, but can also be true for any artifact which is republished without changing it's identifier. By caching artifacts based on their SHA1 checksum, Gradle is able to maintain multiple versions of the same artifact. This means that when resolving against one repository Gradle will never overwrite the cached artifact file from a different repository. This is done without requiring a separate artifact file store per repository.

50.9.1.5.缓存锁

50.9.1.5. Cache Locking

Gradle依赖缓存使用基于文件的锁定来确保它可以被多个Gradle并发进程安全地使用。当二进制元数据存储被读或写的时候锁被持有,但缓慢操作时,如下载远程工件被释放。

The Gradle dependency cache uses file-based locking to ensure that it can safely be used by multiple Gradle processes concurrently. The lock is held whenever the binary meta-data store is being read or written, but is released for slow operations such as downloading remote artifacts.

### **50.9.2.覆盖缓存的命令行选项**

50.9.2. Command line options to override caching

50.9.2.1. 离线

50.9.2.1. Offline

——离线命令行开关告诉Gradle总是从缓存中使用依赖模块,无论他们是否又检查了一遍。当运行离线时,Gradle将永远不会试图访问网络执行依赖解析。如果需要的模块在依赖缓存中不存在,构建执行将会失败。

The --offline command line switch tells Gradle to always use dependency modules from the cache, regardless if they are due to be checked again. When running with offline, Gradle will never attempt to access the network to perform dependency resolution. If required modules are not present in the dependency cache, build execution will fail.

50.9.2.2. 刷新

50.9.2.2. Refresh

有时,Gradle依赖缓存可以与实际状态配置存储库不同步。也许是存储库配置初始化错误,或者一个“不变”模块发表错误。在依赖缓存刷新所有依赖项,在命令行上使用——refresh-dependencies选项。

At times, the Gradle Dependency Cache can be out of sync with the actual state of the configured repositories. Perhaps a repository was initially misconfigured, or perhaps a “non-changing” module was published incorrectly. To refresh all dependencies in the dependency cache, use the --refresh-dependencies option on the command line.

——refresh-dependencies选项告诉Gradle忽略所有解析模块和工件的缓存条目。一个全新的将针对所有配置存储库解析,进行动态版本重新计算,模块刷新和工件下载。然而,在可能的情况下Gradle将在再次下载之前检查以前下载的产物是否有效。它是通过比较存储库发布的SHA1值与现有的下载产物的SHA1值实现的。

The --refresh-dependencies option tells Gradle to ignore all cached entries for resolved modules and artifacts. A fresh resolve will be performed against all configured repositories, with dynamic versions recalculated, modules refreshed, and artifacts downloaded. However, where possible Gradle will check if the previously downloaded artifacts are valid before downloading again. This is done by comparing published SHA1 values in the repository with the SHA1 values for existing downloaded artifacts.

### **50.9.3.微调控制依赖缓存**

50.9.3. Fine-tuned control over dependency caching

你可以通过使用配置的ResolutionStrateg来微调缓存的某个方面。

You can fine-tune certain aspects of caching using the ResolutionStrategy for a configuration.

默认，Gradle缓存24小时动态版本。要更改Gradle缓存动态版本的解析版本的时间，使用：

By default, Gradle caches dynamic versions for 24 hours. To change how long Gradle will cache the resolved version for a dynamic version, use:

Example 50.64. Dynamic version cache control

build.gradle
```
configurations.all {
    resolutionStrategy.cacheDynamicVersionsFor 10, 'minutes'
}
```

默认，Gradle缓存24小时变更模块。要更改Gradle缓存一个变更模块的元数据和产物的时间，使用：

By default, Gradle caches changing modules for 24 hours. To change how long Gradle will cache the meta-data and artifacts for a changing module, use:

Example 50.65. Changing module cache control

build.gradle
```
configurations.all {
    resolutionStrategy.cacheChangingModulesFor 4, 'hours'
}
```

更多细节,看看ResolutionStrategy的API文档。

For more details, take a look at the API documentation for ResolutionStrategy.

50.10.传递依赖关系管理的策略

50.10. Strategies for transitive dependency management

Many projects rely on the Maven Central repository. This is not without problems.
　
Maven中央存储库反应迟钝。　　许多流行的项目的POM文件指定依赖关系或其他错误的配置(例如,POM文件的“commons-httpclient 3.0”模块声明JUnit运行时依赖)。　　对许多项目没有一个正确的依赖关系(或多或少地强加POM格式)。

The Maven Central repository can be down or can be slow to respond.
The POM files of many popular projects specify dependencies or other configuration that are just plain wrong (for instance, the POM file of the “commons-httpclient-3.0” module declares JUnit as a runtime dependency).

For many projects there is not one right set of dependencies (as more or less imposed by the POM format).

如果你的项目依赖于Maven中央存储库,你可能会需要一个额外的自定义库,因为:

If your project relies on the Maven Central repository you are likely to need an additional custom repository, because:

您可能需要还没有上传到Maven中央的依赖关系。
你想要在Maven POM文件中妥善处理无效的元数据。　　
你不想让人们知道停工期以及Maven中央反应迟缓,如果他们只是想构建您的项目。

You might need dependencies that are not uploaded to Maven Central yet.
You want to deal properly with invalid metadata in a Maven Central POM file.
You don't want to expose people to the downtimes or slow response of Maven Central, if they just want to build your project.

这没什么大不了的可以通过设置一个自定义存储库,[17]但它可以乏味的使其更新。对于一个新版本,您必须创建新的XML描述符和目录。您的自定义库是另一个基础设施元素它可能有停工,需要更新。为了有历史构建,您需要保留所有过去的库,更不用说一个备份。这是另一个间接层。另一个你要查找的信息。这不是一个非常大的事,但是在其总和有影响。

It is not a big deal to set-up a custom repository, [17] but it can be tedious to keep it up to date. For a new version, you always have to create the new XML descriptor and the directories. Your custom repository is another infrastructure element which might have downtimes and needs to be updated. To enable historical builds, you need to keep all the past libraries, not to mention a backup of these. It is another layer of indirection. Another source of information you have to lookup. All this is not really a big deal but in its sum it has an impact. 

库管理者喜欢Artifactory或Nexus可更方便,但大多数开源项目通常不会有很多这些产品的host。这是改变与Bintray等新服务,让开发者主机和分发他们释放二进制文件使用自助服务存储库平台。Bintray还支持共享工件虽然JCenter公共存储库提供所有流行的开源Java工件一个解决地址(请参见50.6.2“Maven JCenter库”)。

Repository managers like Artifactory or Nexus make this easier, but most open source projects don't usually have a host for those products. This is changing with new services like Bintray that let developers host and distribute their release binaries using a self-service repository platform. Bintray also supports sharing approved artifacts though the JCenter public repository to provide a single resolution address for all popular OSS Java artifacts (see Section 50.6.2, “Maven JCenter repository”).

这是一个常见的原因为什么许多项目倾向于在版本控制系统存储库。这种方法Gradle是完全支持的。库可以存储在一个平面目录没有任何XML模块描述符文件。但它提供完整的传递依赖关系管理。您可以使用客户模块依赖关系来表达的依赖关系,或工件依赖性,以防第一水平依赖没有传递依赖关系。人们可以从你的源代码控制系统看看这样一个项目,有必要构建它的一切。

This is a common reason why many projects prefer to store their libraries in their version control system. This approach is fully supported by Gradle. The libraries can be stored in a flat directory without any XML module descriptor files. Yet Gradle offers complete transitive dependency management. You can use either client module dependencies to express the dependency relations, or artifact dependencies in case a first level dependency has no transitive dependencies. People can check out such a project from your source code control system and have everything necessary to build it.

如果你正使用一个分布式版本控制系统比如Git您可能不希望使用版本控制系统来存储库,因为人们可以查看整个历史。但即使在这里Gradle的灵活性可以使你的生活更容易。例如,您可以使用一个共享没有XML描述符的平面目录,但您可以充分传递依赖管理,如上所述。

If you are working with a distributed version control system like Git you probably don't want to use the version control system to store libraries as people check out the whole history. But even here the flexibility of Gradle can make your life easier. For example, you can use a shared flat directory without XML descriptors and yet you can have full transitive dependency management, as described above.

你也可以有一个混合策略。如果你的主要关心的问题是POM文件的坏元数据和维护的自定义XML描述符,然后客户端模块提供了一个选择。然而,你仍然可以使用Maven2库或您的自定义库的库仍然享受传递依赖关系管理。或者你可以只提供pom客户端模块坏元数据。对于jar文件和正确的pom你仍然使用远程存储库。

You could also have a mixed strategy. If your main concern is bad metadata in the POM file and maintaining custom XML descriptors, then Client Modules offer an alternative. However, you can still use a Maven2 repo or your custom repository as a repository for jars only and still enjoy transitive dependency management. Or you can only provide client modules for POMs with bad metadata. For the jars and the correct POMs you still use the remote repository.

### **50.10.1.隐式传递依赖关系**

50.10.1. Implicit transitive dependencies

还有来处理没有XML描述符文件传递依赖关系的另一种方法。你可以用Gradle来做这件事情,但是我们不推荐它。我们提到它是为了完整性和与其他构建工具和比较。

There is another way to deal with transitive dependencies without XML descriptor files. You can do this with Gradle, but we don't recommend it. We mention it for the sake of completeness and comparison with other build tools.

诀窍是只使用工件依赖性和组列表。这将直接表达你的第一级依赖性和传递依赖关系(参见50.4.8节,“可选属性”)。问题在于Gradle依赖关系管理将认为这是指定所有依赖项作为一级依赖性。依赖的报告将不会显示你真正的依赖图且编译任务使用所有依赖项,不仅第一级依赖性。总之,构建不易于维护和可靠的相比它使用客户端模块时,并且你获得不到任何东西。

The trick is to use only artifact dependencies and group them in lists. This will directly express your first level dependencies and your transitive dependencies (see Section 50.4.8, “Optional attributes”). The problem with this is that Gradle dependency management will see this as specifying all dependencies as first level dependencies. The dependency reports won't show your real dependency graph and the compile task uses all dependencies, not just the first level dependencies. All in all, your build is less maintainable and reliable than it could be when using client modules, and you don't gain anything.

[14]Gradle支持部分multiproject构建(见57章,多项目构建)。

[14] Gradle supports partial multiproject builds (see Chapter 57, Multi-project Builds).

[15] http://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-project-relationships.html

[16] At http://ant.apache.org/ivy/history/latest-milestone/concept.html you can learn more about ivy patterns.

[17]如果你想保护你的项目在Maven中央不停工事情变得更加复杂。您可能想要设置存储库代理。在企业环境中,这是很常用的。一个开源项目,它看起来像是多余的。

[17] If you want to shield your project from the downtimes of Maven Central things get more complicated. You probably want to set-up a repository proxy for this. In an enterprise environment this is rather common. For an open source project it looks like overkill.