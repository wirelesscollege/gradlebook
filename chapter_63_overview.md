# **Chapter 62. Embedding Gradle**

62.1. Introduction to the Tooling API

62.1.Tooling API的介绍

The 1.0 milestone 3 release brought a new API called the tooling API, which you can use for embedding Gradle into your own custom software. This API allows you to execute and monitor builds, and to query Gradle about the details of a build. The main audience for this API will be IDEs, CI servers, other UI authors, or integration testing of your Gradle plugins. However, it is open for anyone who needs to embed Gradle in their application.

1.0里程碑发布版3带来了一个新的API名叫Tooling API，它可以用作将Gradle嵌入到你自定义的软件中。此API允许您执行和监控构建，并查询Gradle关于构建的细节。这个API的主要受众将会是一些IDE,CI服务器，其他的UI作者，或者用于Gradle插件的集成测试。然而，它是开放的，只要需要，任何人都可以将Gradle嵌入到他们自己的应用里。

A fundamental characteristic of the tooling API is that it operates in a version independent way. This means that you can use the same API to work with different target versions of Gradle. The tooling API is Gradle wrapper aware and, by default, uses the same target Gradle version as that used by the wrapper-powered project.

Tooling API的一个基本特点是,它是以版本独立的方式运作。这就意味着您可以使用相同的API来处理不同目标版本的Gradle。Tooling API是Gradle包装意识,默认情况下,使用和包装驱动的项目相同的目标Gradle版本。

Some features that the tooling API provides today:

当前Tooling API提供的一些特性：

You can query Gradle for the details of a build, including the project hierarchy and the project dependencies, external dependencies (including source and Javadoc jars), source directories and tasks of each project.

你可以查询Graddle关于构建的细节，包括每个项目的项目层级和项目依赖，外部依赖（包括源码和javadoc jars文件），源码目录和任务。

You can execute a build and listen to stdout and stderr logging and progress (e.g. the stuff shown in the 'status bar' when you run on the command line).
Tooling API can download and install the appropriate Gradle version, similar to the wrapper. Bear in mind that the tooling API is wrapper aware so you should not need to configure a Gradle distribution directly.

你可以执行一个构建，监听stdout 和stderr日志和进度（比如运行命令行时候在状态栏上显示的东西）。
Tooling API可以下载和安装合适的Gradle版本，类似于包装。考虑到Tooling API是包装意识所以你没有必要直接配置Gradle分布。

The implementation is lightweight, with only a small number of dependencies. It is also a well-behaved library, and makes no assumptions about your classloader structure or logging configuration. This makes the API easy to bundle in your application.

实现是轻量级的,只有少量的依赖性。这也是一个功能良好的库,没有假设你的类加载器结构或日志配置。这使得API很容易绑定您的应用程序。

In the future we may support other interesting features:

未来我们可能会支持其他有趣的特性

Performance. The API gives us the opportunity to do lots of caching, static analysis and preemptive work, to make things faster for the user.
Better progress monitoring and build cancellation. For example, allowing test execution to be monitored.

性能。API使我们有机会做大量的缓存,静态分析和先发制人的工作,使得用户使用更快。
更好的进度监控和构建取消。例如,允许监控测试执行。

Notifications when things in the build change, so that UIs and models can be updated. For example, your Eclipse or IDEA project will update immediately, in the background.

通知由构建时的改变导致的UI和模型可能被更新。例如，Eclipse或IDEA项目将在后台立即更新。
Validating and prompting for user supplied configuration.

验证和提示用户提供的配置

Prompting for and managing user credentials.

提示和管理用户凭证。

## **62.2. Tooling API and the Gradle Build Daemon**

62chapter.2. Tooling API和Gradle构建守护进程

Please take a look at Chapter 18, The Gradle Daemon. The Tooling API uses the daemon all the time. In fact, you cannot officially use the Tooling API without the daemon. This means that subsequent calls to the Tooling API, be it model building requests or task executing requests can be executed in the same long-living process. Chapter 18, The Gradle Daemon contains more details about the daemon, specifically information on situations when new daemons are forked.

请看看第18章,Gradle守护进程。Tooling API无时无刻不使用守护进程。事实上,若没有守护进程你无法正式地使用Tooling API。这意味着后续对Tooling API的调用,无论是模型构建请求还是任务执行请求都可以在同一长期过程下执行。第18章,Gradle守护进程包含更详细的守护进程,特别当新的守护进程被forked的情况信息。

## **62.3. Quickstart**

62.3. 快速入门

As the tooling API is an interface for developers, the Javadoc is the main documentation for it. This is exactly our intention - we don't expect this chapter to grow very much. Instead we will add more code samples and improve the Javadoc documentation. The main entry point to the tooling API is the GradleConnector. You can navigate from there to find code samples and other instructions. Another very important set of resources are the samples that live in “$gradleHome/samples/toolingApi”. These samples also specify all of the required dependencies for the Tooling API, along with the suggested repositories to obtain the jars from

由于Tooling API对于开发者是个接口，Javadoc是它的主要文档。这正是我们的目的,我们不期望这一章非常长。取而代之,我们将添加更多的代码示例来提高Javadoc文档。指向tooling API的主要入口点是GradleConnector。你可以从那里导航找到代码示例和其他指令。另一个非常重要的资源集在“$gradleHome/samples/toolingApi”。这些样例还指明了Tooling API所有必需的依赖项,以及获取jar文件的建议库

百度搜索[无线学院](http://wirelesscollege.cn)