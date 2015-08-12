# **第六十一章 初始化脚本**

Chapter 61. Initialization Scripts

Gradle提供了强大的机制来支持基于当前环境的自定义构建，这种机制也支持那些将要和Gradle结合的工具。

Gradle provides a powerful mechanism to allow customizing the build based on the current environment. This mechanism also supports tools that wish to integrate with Gradle.

注意：这是由“build-init”插件提供的截然不同的“init”任务（参见第47章，构建初始化插件）。

Note that this is completely different from the “init” task provided by the “build-init” incubating plugin (see Chapter 47, Build Init Plugin).

## **61.1 基本用法**

61.1. Basic usage

初始化脚本（又叫init脚本），它类似于Gradle中的其他脚本，但是这些脚本都是在构建之前运行。以下是几个可能的用法：

Initialization scripts (a.k.a. init scripts) are similar to other scripts in Gradle. These scripts, however, are run before the build starts. Here are several possible uses:

•	设置enterprise-wide配置，如去哪里查找自定义插件。

•	设置基于当前环境的属性，如开发人员的机器和持续集成服务器。

•	提供所需的用户信息的构建，如资源库或数据库身份验证凭据的个人信息。

•	定义特定机器的细节，如JDK的安装位置。

•	注册构建监听，听听Gradle事件的外部工具，你会发现这很有用。

•	注册构建记录，你可以自定义Gradle事件生成的日志。

•	Set up enterprise-wide configuration, such as where to find custom plugins.

•	Set up properties based on the current environment, such as a developer's machine vs. a continuous integration server.

•	Supply personal information about the user that is required by the build, such as repository or database authentication credentials.

•	Define machine specific details, such as where JDKs are installed.

•	Register build listeners. External tools that wish to listen to Gradle events might find this useful.

•	Register build loggers. You might wish to customize how Gradle logs the events that it generates.

init脚本的一个主要限制是它不能访问buildSrc 项目中的类（见59.4节，“buildSrc 项目的资源构建”有该功能的详细信息）。

One main limitation of init scripts is that they cannot access classes in the buildSrc project (see Section 59.4, “Build sources in the buildSrc project” for details of this feature).

## **61.2 使用初始化脚本**

61.2. Using an init script

这是几种init脚本的使用方法：

There are several ways to use an init script:

•	命令行上的文件使用，命令行选项是 -I 或 --init-script后接路径脚本。命令行选项可以出现多次，每次增加一个init脚本。

•	把一个名叫init.gradle的文件放到USER_HOME/.gradle/目录下。

•	把一个后缀为.gradle的文件放到USER_HOME/.gradle/init.d/目录下。

•	把一个后缀为.gradle的文件放到GRADLE_HOME/init.d/目录下，在Gradle分布中，它允许你打包包含一些自定义构建逻辑和插件。你可以把这些和Gradle包装结合起来，以此进行自定义逻辑，适用于所有的企业构建。

•	Specify a file on the command line. The command line option is -I or --init-script followed by the path to the script. The command line option can appear more than once, each time adding another init script.

•	Put a file called init.gradle in the USER_HOME/.gradle/ directory.

•	Put a file that ends with .gradle in the USER_HOME/.gradle/init.d/ directory.

•	Put a file that ends with .gradle in the GRADLE_HOME/init.d/ directory, in the Gradle distribution. This allows you to package up a custom Gradle distribution containing some custom build logic and plugins. You can combine this with the Gradle wrapper as a way to make custom logic available to all builds in your enterprise.

如果找到一个以上的init脚本，它们都将按照上面指定的顺序被执行。在特定目录的脚本是按照字母顺序来执行。例如，当Gradle执行时，它将允许一个工具在命令行上指定一个init脚本以和放置一个到自己的主目录中定义环境以及两个脚本的运行。

If more than one init script is found they will all be executed, in the order specified above. Scripts in a given directory are executed in alphabetical order. This allows, for example, a tool to specify an init script on the command line and the user to put one in their home directory for defining the environment and both scripts will run when Gradle is executed.

## **61.3 编写一个init脚本**

61.3. Writing an init script

init脚本是一个Groovy脚本，它类似于Gradle构建脚本。每一个init脚本都有一个与之关联的Gradle实例，任何属性的参考和方法的调用在init脚本中将委托给这个Gradle实例。

Similar to a Gradle build script, an init script is a Groovy script. Each init script has a Gradle instance associated with it. Any property reference and method call in the init script will delegate to this Gradle instance.

每个init脚本也实现了脚本接口。

Each init script also implements the Script interface.

### **61.3.1 init脚本配置项目**

61.3.1. Configuring projects from an init script

你可以使用init脚本配置项目的构建，在多项目构建时可用类似的方式去配置项目。下面的示例展示了一个init脚本在项目评估前如何去执行额外的配置，这个示例使用此功能去配置一个仅适用于特定环境的额外库。

You can use an init script to configure the projects in the build. This works in a similar way to configuring projects in a multi-project build. The following sample shows how to perform extra configuration from an init script before the projects are evaluated. This sample uses this feature to configure an extra repository to be used only for certain environments.

例61.1 在项目评估前，使用init脚本去执行额外配置。

Example 60.1. Using init script to perform extra configuration before projects are evaluated

build.gradle
```
repositories {
    mavenCentral()
}

task showRepos << {
    println "All repos:"
    println repositories.collect { it.name }
}
init.gradle
allprojects {
    repositories {
        mavenLocal()
    }
}
Output of gradle --init-script init.gradle -q showRepos
> gradle --init-script init.gradle -q showRepos
All repos:
[MavenLocal, MavenRepo]
```

## **61.4 init脚本的外部依赖**

61.4. External dependencies for the init script

在59.6节 “构建脚本的外部依赖”中解释了如何将外部依赖关系添加到构建脚本。init脚本也可以声明依赖，你可以通过initscript() 方法传递一个封闭该声明的init脚本类路径。

In Section 59.6, “External dependencies for the build script” it was explained how to add external dependencies to a build script. Init scripts can also declare dependencies. You do this with the initscript() method, passing in a closure which declares the init script classpath.

例60.2声明外部依赖的init脚本

Example 60.2. Declaring external dependencies for an init script

init.gradle
```
initscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.apache.commons', name: 'commons-math', version: '2.0'
    }
}
```

通过initscript()方法关闭一个ScriptHandler 实例，你可以通过添加依赖的classpath来声明init脚本的classpath。例如，Java以同样的方式来编译classpath，见50.4节“如何声明你的依赖”，除了项目的依赖。

The closure passed to the initscript() method configures a ScriptHandler instance. You declare the init script classpath by adding dependencies to the classpath configuration. This is the same way you declare, for example, the Java compilation classpath. You can use any of the dependency types described in Section 50.4, “How to declare your dependencies”, except project dependencies.

所有的init脚本classpath声明，你都可以使用类的init脚本classpath上的任何其他类。下面是之前的示例添加并使用类的init脚本classpath。

Having declared the init script classpath, you can use the classes in your init script as you would any other classes on the classpath. The following example adds to the previous example, and uses classes from the init script classpath.

例60.3 init脚本与外部依赖

Example 60.3. An init script with external dependencies

init.gradle
```
import org.apache.commons.math.fraction.Fraction

initscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.apache.commons', name: 'commons-math', version: '2.0'
    }
}

println Fraction.ONE_FIFTH.multiply(2)
Output of gradle --init-script init.gradle -q doNothing
> gradle --init-script init.gradle -q doNothing
2 / 5
```

## **60.5 init脚本插件**

60.5. Init script plugins

类似于Gradle构建脚本或者Gradle设置文件，插件也同样可以应用在init脚本中。

Similar to a Gradle build script or a Gradle settings file, plugins can be applied on init scripts.

例 60.4  init脚本的插件使用

Example 60.4. Using plugins in init scripts

init.gradle
```
apply plugin:EnterpriseRepositoryPlugin

class EnterpriseRepositoryPlugin implements Plugin<Gradle> {

    private static String ENTERPRISE_REPOSITORY_URL = "https://repo.gradle.org/gradle/repo"

    void apply(Gradle gradle) {
        // ONLY USE ENTERPRISE REPO FOR DEPENDENCIES
        gradle.allprojects{ project ->
            project.repositories {

                // Remove all repositories not pointing to the enterprise repository url
                all { ArtifactRepository repo ->
                    if (!(repo instanceof MavenArtifactRepository) ||
                          repo.url.toString() != ENTERPRISE_REPOSITORY_URL) {
                        project.logger.lifecycle "Repository ${repo.url} removed. Only $ENTERPRISE_REPOSITORY_URL is allowed"
                        remove repo
                    }
                }

                // add the enterprise repository
                maven {
                    name "STANDARD_ENTERPRISE_REPO"
                    url ENTERPRISE_REPOSITORY_URL
                }
            }
        }
    }
}
build.gradle
repositories{
    mavenCentral()
}

 task showRepositories << {
    repositories.each{
        println "repository: ${it.name} ('${it.url}')"
    }
}
Output of gradle -q -I init.gradle showRepositories
> gradle -q -I init.gradle showRepositories
repository: STANDARD_ENTERPRISE_REPO ('https://repo.gradle.org/gradle/repo')
```

在init脚本插件中，要确保运行版本时，只有一个指定的存储库在使用。

The plugin in the init script ensures that only a specified repository is used when running the build.

当init脚本中应用插件时，Gradle会实例化插件并调用插件实例的Plugin.apply()方法。该Gradle对象被作为参数传递，其可以被用来配置一个构建的所有方面。当然，这个应用插件可以解析为一个外部的依赖，详情参见60.4节“init脚本的外部依赖”。

When applying plugins within the init script, Gradle instantiates the plugin and calls the plugin instance's Plugin.apply() method. The gradle object is passed as a parameter, which can be used to configure all aspects of a build. Of course, the applied plugin can be resolved as an external dependency as described in Section 60.4, “External dependencies for the init script”

