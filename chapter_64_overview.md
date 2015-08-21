# **第64章. The Gradle Wrapper**

# **Chapter 64. The Gradle Wrapper**

The Gradle Wrapper (henceforth referred to as the “wrapper”) is the preferred way of starting a Gradle build. The wrapper is a batch script on Windows, and a shell script for other operating systems. When you start a Gradle build via the wrapper, Gradle will be automatically downloaded and used to run the build.

Gradle Wrapper(此后称为“Wrapper”)是启动Gradle构建的首选方式。Wrapper在Windows上是个批处理命令，在其他操作系统中是个shell
脚本。当你使用Wrapper来启动Gradle构建时，Gradle会自动被下载并运行构建。

The wrapper is something you should check into version control. By distributing the wrapper with your project, anyone can work with it without needing to install Gradle beforehand. Even better, users of the build are guaranteed to use the version of Gradle that the build was designed to work with. Of course, this is also great for continuous integration servers (i.e. servers that regularly build your project) as it requires no configuration on the server.

Wrapper可以检查版本控制。通过给您的项目分布wrapper，任何人都可以使用它而无需事先安装Gradle。甚至更好的保证了构建者构建时使用设计好的Gradle版本。当然，这对于持续集成服务器也是极好的，因为它不需要在服务器上做配置。

You install the wrapper into your project by running the wrapper task. (This task is always available, even if you don't add it to your build). To specify a Gradle version use --gradle-version on the command-line. You can also set the URL to download Gradle from directly via --gradle-distribution-url. If no version or distribution URL is specified, the wrapper will be configured to use the gradle version the wrapper task is executed with. So if you run the wrapper task with Gradle 2.4 and the wrapper configuration will default to Gradle 2.4.

通过运行wrapper任务，将wrapper安装到项目中。（这项任务是随时可用的，即使你不添加到您的构建）。指定一个Gradle使用版本——-在命令行中输入-gradle-version。你也可以设置url直接通过--gradle-distribution-url下载Gradle。如果没有指定版本或分布URL，wrapper将被配置为使用wrapper任务执行的版本。所以，如果你运行的包装任务版本是Gradle 2.4，那么wrapper将默认配置版本2.4。

Example 64.1. Running the wrapper task

例64.1  运行wrapper任务

Output of gradle wrapper --gradle-version 2.0
```
> gradle wrapper --gradle-version 2.0
:wrapper

BUILD SUCCESSFUL

Total time: 1 secs
```

The wrapper can be further customized by adding and configuring a Wrapper task in your build script, and then executing it.

wrapper可以通过添加和配置你的构建脚本中的wrapper任务来进一步定制，然后执行。

Example 64.2. Wrapper task

例64.2  Wrapper任务

build.gradle
```
task wrapper(type: Wrapper) {
    gradleVersion = '2.0'
}
```

After such an execution you find the following new or updated files in your project directory (in case the default configuration of the wrapper task is used).

Example 64.3. Wrapper generated files

例64.3  Wrapper生成文件
```
Build layout
simple/
  gradlew
  gradlew.bat
  gradle/wrapper/
    gradle-wrapper.jar
    gradle-wrapper.properties
```

All of these files should be submitted to your version control system. This only needs to be done once. After these files have been added to the project, the project should then be built with the added gradlew command. The gradlew command can be used exactly the same way as the gradle command.

所有这些文件都应该提交给你的版本控制系统。这仅需要做一次。在这些文件被添加到项目中后，项目该使用增加的gradlew命令进行构建了。gradlew命令和gradle命令使用方式完全相同。

If you want to switch to a new version of Gradle you don't need to rerun the wrapper task. It is good enough to change the respective entry in the gradle-wrapper.properties file, but if you want to take advantage of new functionality in the Gradle wrapper, then you would need to regenerate the wrapper files.

如果想要改变Gradle的版本，你不必重新执行wrapper任务。在gradle-wrapper.properties文件中可以很好的更改各自入口。但是如果你想要利用Gradle wrapper中心的功能，那么你就需要重新封装文件。

## **64.1. Configuration**

64.1. 配置

If you run Gradle with gradlew, the wrapper checks if a Gradle distribution for the wrapper is available. If so, it delegates to the gradle command of this distribution with all the arguments passed originally to the gradlew command. If it didn't find a Gradle distribution, it will download it first.

如果你使用gradlew运行Gradle，wrapper会检查是否有Gradle分布可用。如果有，它会将这个分布的gradle命令所有参数传递给最初的gradlew命令.如果没有发现有Gradle分布，就会先进行下载。

When you configure the Wrapper task, you can specify the Gradle version you wish to use. The gradlew command will download the appropriate distribution from the Gradle repository. Alternatively, you can specify the download URL of the Gradle distribution. The gradlew command will use this URL to download the distribution. If you specified neither a Gradle version nor download URL, the gradlew command will download whichever version of Gradle was used to generate the wrapper files.

当你配置Wrapper任务的时候，你可以指定你希望使用的Gradle版本。gradlew命令将会从Gradle库中下载合适的分布。或者，你可以指定Gradle分布的下载URL。gradlew命令将会使用这个URL来下载分布。如果你既没有指定Gradle版本也没有指定下载URL，那么gradlew会下载任意哪个版本用来生成wrapper文件。

For the details on how to configure the wrapper, see the Wrapper class in the API documentation.

对于如何配置wrapper的具体细节，请看API文档中的wrapper类。

If you don't want any download to happen when your project is built via gradlew, simply add the Gradle distribution zip to your version control at the location specified by your wrapper configuration. A relative URL is supported - you can specify a distribution file relative to the location of gradle-wrapper.properties file.

如果你不想在你的项目构建中通过gradlew产生任何下载，仅需在wrapper配置路径中添加Gradle分布zip包到你的版本控制中。相关的网址是支持的-你可以指明一个分布文件关联在gradle-wrapper.properties文件里。

If you build via the wrapper, any existing Gradle distribution installed on the machine is ignored.

如果你通过wrapper构建，那么任何已有的安装好的Gradle分布都会被忽略。

## **64.2. Unix file permissions**

64.2. Unix 文件权限

The Wrapper task adds appropriate file permissions to allow the execution of the gradlew *NIX command. Subversion preserves this file permission. We are not sure how other version control systems deal with this. What should always work is to execute “sh gradlew”.

Wrapper任务添加了合适的权限用来运行gradlew *NIX 
命令的执行。svn保存了这个文件权限。我们不确定其他版本控制是如何处理的。总是起作用的就是执行 “sh gradlew”.


