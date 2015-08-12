# Chapter 2. Overview
这里是Gradle的一些特性
## **2.1Features**


* **动态构建**

    Declarative builds and build-by-convention
    
    Gradle使用动态说明性语言DSL，这种语言陈述是基于Groovy脚本的，你可以按照喜欢的方式去定义Gradle配置脚本，它支持Java，Groovy，OSGi，Web以及Scala projects.不仅如此，该语言也可以进行扩展，可以添加你自己的元素，这样使你的构建程序更加简洁，可持续，也可维护。

    At the heart of Gradle lies a rich extensible Domain Specific Language (DSL) based on Groovy. Gradle pushes declarative builds to the next level by providing declarative language elements that you can assemble as you like. Those elements also provide build-by-convention support for Java, Groovy, OSGi, Web and Scala projects. Even more, this declarative language is extensible. Add your own new language elements or enhance the existing ones. Thus providing concise, maintainable and comprehensible builds.

    完全依赖于普通得编程语言，如果你有唯一独特性的需求，Gradle可以提供给你最合适的任务图表。

    Language for dependency based programming
The declarative language lies on top of a general purpose task graph, which you can fully leverage in your builds. It provides utmost flexibility to adapt Gradle to your unique needs.

* **结构化构建**

    Structure your build
    
    Gradle支持你设计自己的构建程序，你的配置层级可以内联其他配置等，只需要符合本地构建逻辑即可，当然也支持晚上构建程序，最后你可以创建一个比较好的结构配置，简单易用。

    The suppleness and richness of Gradle finally allows you to apply common design principles to your build. For example, it is very easy to compose your build from reusable pieces of build logic. Inline stuff where unnecessary indirections would be inappropriate. Don't be forced to tear apart what belongs together (e.g. in your project hierarchy). Thus avoiding smells like shotgun changes or divergent change that turn your build into a maintenance nightmare. At last you can create a well structured, easily maintained, comprehensible build.

* **深度API**

    Deep API
    Gradle允许你hooks它生命周期内的任何一个环节，并且允许你复制或者自定义它的核心配置。

    From being a pleasure to be used embedded to its many hooks over the whole lifecycle of build execution, Gradle allows you to monitor and customize its configuration and execution behavior to its very core.

* **Gradle评价**

    Gradle scales
    
    Gradle的评价非常好，支持单项目构建也支持多样化项目构建，让你的构建更有艺术性，为很多企业解决了头痛的构建性能问题。
    
    Gradle scales very well. It significantly increases your productivity, from simple single project builds up to huge enterprise multi-project builds. This is true for structuring the build. With the state-of-art incremental build function, this is also true for tackling the performance pain many large enterprise builds suffer from.

* **多样化项目构建**

    Multi-project builds
    
    Gradle是杰出的多样化项目构建工具，你可以配置项目中的依赖关系。
    
    Gradle's support for multi-project build is outstanding. Project dependencies are first class citizens. We allow you to model the project relationships in a multi-project build as they really are for your problem domain. Gradle follows your layout not vice versa.

    Gradle也支持部分构建，如果你构建一个子项目的时候，可以不去构建其他依赖的子项目，你可以选择重新构建部分项目依赖的子项目，这在比较大的项目构建中，这种灵活性的配置会让你的项目构建更加省时。
    
    Gradle provides partial builds. If you build a single subproject Gradle takes care of building all the subprojects that subproject depends on. You can also choose to rebuild the subprojects that depend on a particular subproject. Together with incremental builds this is a big time saver for larger builds.

* **多方式管理你的依赖**

    Many ways to manage your dependencies

    不同的Team会有不同得管理依赖的方式，Gradle提供了市场上大部分的构建工具，例如Maven和Ivy，都可以无缝的融合。

    Different teams prefer different ways to manage their external dependencies. Gradle provides convenient support for any strategy. From transitive dependency management with remote Maven and Ivy repositories to jars or dirs on the local file system.

    Gradle是第一个集成化构建工具，支持Ant task，并且可以对Ant项目做深度支持，它可以直接引用指定你的build.xml文件，也可以使用你的文件内的各种属性和变量，例如properties，paths等等...
    
    Gradle is the first build integration tool
Ant tasks are first class citizens. Even more interesting, Ant projects are first class citizens as well. Gradle provides a deep import for any Ant project, turning Ant targets into native Gradle tasks at runtime. You can depend on them from Gradle, you can enhance them from Gradle, you can even declare dependencies on Gradle tasks in your build.xml. The same integration is provided for properties, paths, etc ...

    Gradle完全支持已经存在的Maven与Ivy项目，Gradle也提供了pom文件转化为gradle脚本，Mavne项目将在运行时被导入。

    Gradle fully supports your existing Maven or Ivy repository infrastructure for publishing and retrieving dependencies. Gradle also provides a converter for turning a Maven pom.xml into a Gradle script. Runtime imports of Maven projects will come soon.

* 可移植性

    Ease of migration
    
    Gradle适应任何你已有的结构，不管你的产品开发与发布是否在一个分支上，我们始终推荐你在产品构建时写TestCase，后面会有一些比较好的例子。
    
    Gradle can adapt to any structure you have. Therefore you can always develop your Gradle build in the same branch where your production build lives and both can evolve in parallel. We usually recommend to write tests that make sure that the produced artifacts are similar. That way migration is as less disruptive and as reliable as possible. This is following the best-practices for refactoring by applying baby steps.

* Groovy

    Groovy
    
    Gradle的构建配置也就是脚本是使用Groovy语言去写的，而不是xml。不像其他的工具，他的配置完全是动态的脚本语言。
    
    Gradle's build scripts are written in Groovy, not XML. But unlike other approaches this is not for simply exposing the raw scripting power of a dynamic language. That would just lead to a very difficult to maintain build. The whole design of Gradle is oriented towards being used as a language, not as a rigid framework. And Groovy is our glue that allows you to tell your individual story with the abstractions Gradle (or you) provide. Gradle provides some standard stories but they are not privileged in any form. This is for us a major distinguishing features compared to other declarative build systems. Our Groovy support is also not just some simple coating sugar layer. The whole Gradle API is fully groovynized. Only by that using Groovy is the fun and productivity gain it can be.

* The Gradle wrapper

    The Gradle wrapper
    
    Gradle wrapper允许你在不安装Gradle的情况下使用Gradle进行构建和持续集成，降低了你使用Gradle的门槛。Wrapper这个特性很有意思，相当于0成本使用Gradle。
    
    The Gradle Wrapper allows you to execute Gradle builds on machines where Gradle is not installed. This is useful for example for some continuous integration servers. It is also useful for an open source project to keep the barrier low for building it. The wrapper is also very interesting for the enterprise. It is a zero administration approach for the client machines. It also enforces the usage of a particular Gradle version thus minimizing support issues.

* Free and open source

    Gradle is an open source project, and is licensed under the ASL.

    百度搜索[无线学院](http://wirelesscollege.cn)

