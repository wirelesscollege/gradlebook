# *第57章 play插件*

Chapter 57. The Play Plugin

gradle当前已经支持play插件，一些配置可能在gradle以后的版本中发生改变。

The Gradle support for building Play applications is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.

play是一个比较现代的引用开发框架。play插件添加了构建支持，测试以及运行执行。

Play is a modern web application framework. The Play plugin adds support for building, testing and running Play applications with Gradle.

## *57.1使用*

57.1. Usage

为了使用play插件你需要在build.gradle内添加如下代码：

To use the Play plugin, include the following in your build script to apply the play plugin and add the Typesafe repositories:

例：57.1 使用play插件

Example 57.1. Using the Play plugin

build.gradle

```
plugins {
    id 'play'
}

repositories {
    jcenter()
    maven {
        name "typesafe-maven-release"
        url "https://repo.typesafe.com/typesafe/maven-releases"
    }
    ivy {
        name "typesafe-ivy-release"
        url "https://repo.typesafe.com/typesafe/ivy-releases"
        layout "ivy"
    }
}
```

注意尽可能的定义typesafe 资源库，在gradle后续版本中，我们可能使用其他方式来替代当前这种方式。

Note that defining the Typesafe repositories is necessary. In future versions of Gradle, this will be replaced with a more convenient syntax.

## *57.2 限制*

57.2. Limitations

Play插件当前有一些限制。

The Play plugin currently has a few limitations.

仅全面支持Play2.3.x应用，限制性支持play2.4.x应用。Gradle当前不能支持2.4的一些新特性，例如gradle不支持反转路由使用“injected”路由生成器，未来版本gradle可能会支持play2.4.x以及3.0.x。

Full support is limited to Play 2.3.x applications. Limited support is available for Play 2.4.x applications. Gradle does not include support for a few new build-related features in 2.4. Specifically, Gradle does not yet support aggregate reverse routes or the use of "injected" routes generators. Future Gradle versions will add more support for Play 2.4.x and 3.0.x.

已知项目可能只定义了单独的play应用，这意味着单项目不能构建多个play应用，然后多样化构建支持你去配置多个play项目去构建。

A given project may only define a single Play application. This means that a single project cannot build more than one Play application. However, a multi-project build can have many projects that each define their own Play application.

play应用是一个平台集合体，那意味着gradle当前不能控制play当中引用的各软件的版本号。

Play applications can only target a single “platform” (combination of Play, Scala and Java version) at a time. This means that it is currently not possible to define multiple variants of a Play application that, for example, produce jars for both Scala 2.10 and 2.11. This limitation may be lifted in future Gradle versions.

## *57.3软件模型*

57.3. Software Model

play插件使用软件模型去描述play应用，软件模型包含组件以及库文件。

The Play plugin uses a software model to describe a Play application. The software model is comprised of "components" and "binaries". Components are abstract software elements that are built to produce binaries and a Play application is represented by a PlayApplicationSpec component type. The plugin automatically creates a single PlayApplicationBinarySpec when it is applied. Additional Play components cannot be added.

Figure 57.1. Play plugin - software model

![](https://docs.gradle.org/current/userguide/img/playPluginModel.png)

Play plugin - software model

### *play应用组件*

57.3.1. The Play application component

A Play application component describes the application to be built and consists of several configuration elements. One type of element that describes the application are the source sets that define where the application controller, route, template and model class source files should be found. These source sets are logical groupings of files of a particular type and a default source set for each type is created when the play plugin is applied.

Table 57.1. Default Play source sets

|Source Set	|Type	|Directory	|Filters|
|--
|java|	JavaSourceSet|	app|	**/*.java|
|scala|	ScalaLanguageSourceSet|	app|	**/*.scala|
|routes|	RoutesSourceSet|	conf|	routes, *.routes|
|twirlTemplates|	TwirlSourceSet|	app|	**/*.html|
|javaScript|	JavaScriptSourceSet|	app/assets|	**/*.js|

These source sets can be configured or additional source sets can be added to the Play component. See Configuring Play for further information.

Another element of configuring a Play application is the platform. To build a Play application, Gradle needs to understand which versions of Play, Scala and Java to use. The Play component specifies this requirement as a PlayPlatform. If these values are not configured, a default version of Play, Scala and Java will be used. See Targeting a certain version of Play for information on configuring the Play platform.

Note that only a single platform can be specified for a given Play component. This means that only a single version of Play, Scala and Java can be used to build a Play component. In other words, a Play component can only produce one set of outputs, and those outputs will be built using the versions specified by the platform configured on the component.

### *57.3.2 play应用库*

57.3.2. The Play application binary

A Play application component is compiled and packaged to produce a set of outputs which are represented by a PlayApplicationBinarySpec. The Play binary specifies the jar files produced by building the component as well as providing elements by which additional content can be added to those jar files. It also exposes the tasks involved in building the component and creating the binary.

See Configuring Play for examples of configuring the Play binary.

#57.4# *项目结构*

57.4. Project Layout
The Play plugin follows the typical Play application layout. You can configure source sets to include additional directories or change the defaults.

├── app                 → Application source code.
│   ├── assets          → Assets that require compilation.
│   │   └── javascripts → JavaScript source code to be minified.
│   ├── controllers     → Application controller source code.
│   ├── models          → Application business source code.
│   └── views           → Application UI templates.
├── build.gradle        → Your project's build script.
├── conf                → Main application configuration file and routes files.
├── public              → Public assets.
│   ├── images          → Application image files.
│   ├── javascripts     → Typically JavaScript source code.
│   └── stylesheets     → Typically CSS source code.
└── test                → Test source code.

## *57.5 任务*

57.5. Tasks
The Play plugin hooks into the normal Gradle lifecycle tasks such as assemble, check and build, but it also adds several additional tasks which form the lifecycle of a Play project:

Table 57.2. Play plugin - lifecycle tasks

|Task name	|Depends on|	Type|	Description|
|--
|playBinary|	All compile tasks for source sets added to the Play application.|	Task|	Performs a build of just the Play application.|
|dist	|createPlayBinaryDist	|Task	|Assembles the Play distribution.|
|stage	|stagePlayBinaryDist	|Task	|Stages the Play distribution.|

The plugin also provides tasks for running, testing and packaging your Play application:

Table 57.3. Play plugin - running and testing tasks

|Task name	|Depends on	|Type	|Description|
|--
|runPlayBinary	|playBinary to build Play application.|	PlayRun|	Runs the Play application for local development. See how this works with continuous build.|
|testPlayBinary	|playBinary to build Play application and compilePlayBinaryTests.|	Test|	Runs JUnit/TestNG tests for the Play application.|

For the different types of sources in a Play application, the plugin adds the following compilation tasks:

Table 57.4. Play plugin - source set tasks

|Task name|	Source Type	|Type|	Description|
|--
|compilePlayBinaryScala	|Scala and Java	|PlatformScalaCompile	|Compiles all Scala and Java sources defined by the Play application.|
|compilePlayBinaryTwirlTemplates	|Twirl HTML templates	|TwirlCompile|	Compiles HTML templates with the Twirl compiler.|
|compilePlayBinaryRoutes	|Play Route files	|RoutesCompile	|Compiles routes files into Scala sources.|
|minifyPlayBinaryJavaScript|	JavaScript files	|JavaScriptMinify|	Minifies JavaScript files with the Google Closure compiler.|

## *57.6 查明更多*

57.6. Finding out more about your project

Gradle provides a report that you can run from the command-line that shows some details about the components and binaries that your project produces. To use this report, just run gradle components. Below is an example of running this report for one of the sample projects:

Example 57.2. The components report

Output of gradle components

```
> gradle components
:components

------------------------------------------------------------
Root project
------------------------------------------------------------

Play Application 'play'
-----------------------

Source sets
    Java source 'play:java'
        srcDir: app
        includes: **/*.java
    JavaScript source 'play:javaScript'
        srcDir: app/assets
        includes: **/*.js
    JVM resources 'play:resources'
        srcDir: conf
    Routes source 'play:routes'
        srcDir: conf
        includes: routes, *.routes
    Scala source 'play:scala'
        srcDir: app
        includes: **/*.scala
    Twirl template source 'play:twirlTemplates'
        srcDir: app
        includes: **/*.html

Binaries
    Play Application Jar 'playBinary'
        build using task: :playBinary
        platform: Play Platform (Play 2.3.9, Scala: 2.11, Java: Java SE 8)

Note: currently not all plugins register their components, so some components may not be visible here.

BUILD SUCCESSFUL

Total time: 1 secs
```

## *57.7 运行play应用*

57.7. Running a Play application

The runPlayBinary task starts the Play application under development. During development it is beneficial to execute this task as a continuous build. Continuous build is a generic feature that supports automatically re-running a build when inputs change. The runPlayBinary task is “continuous build aware” in that it behaves differently when run as part of a continuous build.

When not run as part of a continuous build, the runPlayBinary task will block the build. That is, the task will not complete as long as the application is running. When running as part of a continuous build, the task will start the application if not running and otherwise propagate any changes to the code of the application to the running instance. This is useful for quickly iterating on your Play application with an edit->rebuild->refresh cycle.

To enable continuous build, run Gradle with -t runPlayBinary or --continuous runPlayBinary.

Users of Play used to such a workflow with Play's default build system should note that compile errors are handled differently. If a build failure occurs before the runPlayBinary during a continuous build, the Play application itself will not reflect this. The Play application will remain unchanged, with the build failure details being present in Gradle's output.

## *57.8 配置play应用*

57.8. Configuring a Play application

### *57.8.1 确认play目标版本*

57.8.1. Targeting a certain version of Play

By default, Gradle uses Play 2.3.9, Scala 2.11 and the version of Java used to start the build. A Play application can select a different version by specifying a target PlayApplicationSpec.platform() on the Play application component.

Example 57.3. Selecting a version of the Play Framework

build.gradle
```
model {
    components {
        play {
            platform play: '2.3.6', scala: '2.10'
        }
    }
}
```

### *57.8.2 添加依赖*

57.8.2. Adding dependencies

You can add compile, test and runtime dependencies to a Play application through Configuration created by the Play plugin.

play is used for compile time dependencies.
playTest is used for test compile time dependencies.
playRun is used for run time dependencies.
Example 57.4. Adding dependencies to a Play application

build.gradle
```
dependencies {
    play "commons-lang:commons-lang:2.6"
}
```

### *57.8.3 配置默认的source sets*

57.8.3. Configuring the default source sets

You can further configure the default source sets to do things like add new directories, add filters, etc.

Example 57.5. Adding extra source sets to a Play application

build.gradle
```
model {
    components {
        play {
            sources {
                java {
                    source.srcDir "additional/java"
                }
                javaScript {
                    source {
                        srcDir "additional/javascript"
                        exclude "**/old_*.js"
                    }
                }
            }
        }
    }
}
```

### *57.8.4 添加额外的source sets*

57.8.4. Adding extra source sets

If your Play application has additional sources that exist in non-standard directories, you can add extra source sets that Gradle will automatically add to the appropriate compile tasks.

Example 57.6. Adding extra source sets to a Play application

build.gradle

```
model {
    components {
        play {
            sources {
                extraJava(JavaSourceSet) {
                    source.srcDir "extra/java"
                }
                extraTwirl(TwirlSourceSet) {
                    source.srcDir "extra/twirl"
                }
                extraRoutes(RoutesSourceSet) {
                    source.srcDir "extra/routes"
                }
            }
        }
    }
}
```

### *57.8.5 配置编译选项*

57.8.5. Configuring compiler options

If your Play application requires additional Scala compiler flags, you can add these arguments directly to the Scala compiler task.

Example 57.7. Configuring Scala compiler options

build.gradle
```
model {
    components {
        play {
            binaries.all {
                tasks.withType(PlatformScalaCompile) {
                    scalaCompileOptions.additionalParameters = ["-feature", "-language:implicitConversions"]
                }
            }
        }
    }
}
```

### 57.8.6 注入自定义asset pipeline

57.8.6. Injecting a custom asset pipeline

Gradle Play support comes with a simplistic asset processing pipeline that minifies JavaScript assets. However, many organizations have their own custom pipeline for processing assets. You can easily hook the results of your pipeline into the Play binary by utilizing the PublicAssets property on the binary.

Example 57.8. Configuring a custom asset pipeline

build.gradle
```
model {
    components {
        play {
            binaries.all { binary ->
                tasks.create("addCopyrightTo${binary.name.capitalize()}Assets", AddCopyrights) { copyrightTask ->
                    source "raw-assets"
                    copyrightFile = project.file('copyright.txt')
                    destinationDir = project.file("${buildDir}/${binary.name}/addCopyRights")

                    // Hook this task into the binary
                    binary.assets.addAssetDir destinationDir
                    binary.assets.builtBy copyrightTask
                }
            }
        }
    }
}

class AddCopyrights extends SourceTask {
    @InputFile
    File copyrightFile

    @OutputDirectory
    File destinationDir

    @TaskAction
    void generateAssets() {
        String copyright = copyrightFile.text
        getSource().files.each { File file ->
            File outputFile = new File(destinationDir, file.name)
            outputFile.text = "${copyright}\n${file.text}"
        }
    }
}
```
## *57.9 多样化Play应用*

57.9. Multi-project Play applications

Play applications can be built in multi-project builds as well. Simply apply the play plugin in the appropriate subprojects and create any project dependencies on the play configuration.

Example 57.9. Configuring dependencies on Play subprojects

build.gradle

```
dependencies {
    play project(":admin")
    play project(":user")
    play project(":util")
}
```

See the play/multiproject sample provided in the Gradle distribution for a working example.

## *57.10 打包Play应用发布*

57.10. Packaging a Play application for distribution


Gradle provides the capability to package your Play application so that it can easily be distributed and run in a target environment. The distribution package (zip file) contains the Play binary jars, all dependencies, and generated scripts that set up the classpath and run the application in a Play-specific Netty container.

The distribution can be created by running the dist lifecycle task and places the distribution in the $buildDir/distributions directory. Alternatively, one can validate the contents by running the stage lifecycle task which copies the files to the $buildDir/stage directory using the layout of the distribution package.

Table 57.5. Play distribution tasks

|Task name	|Depends on|	Type	|Description|
|--
|createPlayBinaryStartScripts	|-	|CreateStartScripts	|Generates scripts to run the Play application distribution.|
|stagePlayBinaryDist	|playBinary, createPlayBinaryStartScripts|	Copy|	Copies all jar files, dependencies and scripts into a staging directory.|
|createPlayBinaryDist|	stagePlayBinaryDist	|Zip|	Bundles the Play application as a distribution.|
|stage	|stagePlayBinaryDist|	Task	|Lifecycle task for staging a Play distribution.|
|dist	|createPlayBinaryDist|	Task	|Lifecycle task for creating a Play distribution.|

### *57.10.1 为你play应用添加额外文件*

57.10.1. Adding additional files to your Play application distribution

You can add additional files to the distribution package using the Distribution API.

Example 57.10. Add extra files to a Play application distribution

build.gradle
```
model {
    distributions {
        playBinary {
            contents {
                from("README.md")
                from("scripts") {
                    into "bin"
                }
            }
        }
    }
}
```
## *57.11 资源 *

57.11. Resources

For additional information about developing Play applications:

Play types in the Gradle DSL Guide:
PlayApplicationBinarySpec
PlayApplicationSpec
PlayPlatform
JvmClasses
PublicAssets
PlayDistributionContainer
JavaScriptMinify
PlayRun
RoutesCompile
TwirlCompile
Play Framework Documentation.