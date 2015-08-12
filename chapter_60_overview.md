# **第六十章 组织构建逻辑**

Chapter 60. Organizing Build Logic 

Gradle提供了多种组织构建逻辑的方式。首先你可以把你的构建逻辑直接放到一个任务的动作中。如果多个任务使用共同的逻辑你可以把逻辑抽取出来成一个方法。如果多个项目共用一个逻辑你可以把这个逻辑定义成一个方法并放到父工程里边。如果构建逻辑太复杂不易于抽取出方法，你应该把你的逻辑封装成一个类来执行你的方法。Gradle可以非常简单的实现。把你的类放到一个目录里然后Gradle会自动编译他们并把他们添加到构建脚本的classpath里。

Gradle offers a variety of ways to organize your build logic. First of all you can put your build logic directly in the action closure of a task. If a couple of tasks share the same logic you can extract this logic into a method. If multiple projects of a multi-project build share some logic you can define this method in the parent project. If the build logic gets too complex for being properly modeled by methods then you likely should implement your logic with classes to encapsulate your logic. [25] Gradle makes this very easy. Just drop your classes in a certain directory and Gradle automatically compiles them and puts them in the classpath of your build script. 

下面对所有组织构建逻辑的方法做一个总结：

Here is a summary of the ways you can organise your build logic:

•   POGOs. 这种方法你可以直接在构建脚本中声明并且使用普通的旧的Groovy对象。构建脚本是用Groovy写的，毕竟Groovy提供了大量的方法去组织代码。

•   POGOs. You can declare and use plain old Groovy objects (POGOs) directly in your build script. The build script is written in Groovy, after all, and Groovy provides you with lots of excellent ways to organize code. 

•   继承属性和方法。在一些工程中，子工程继承父工程的一些属性和方法。

•   Inherited properties and methods. In a multi-project build, sub-projects inherit the properties and methods of their parent project.

•   配置注入。在一些工程里，一个工程（通常是根工程）可以注入属性和方法到另外的工程。

•   Configuration injection. In a multi-project build, a project (usually the root project) can inject properties and methods into another project.

•   源码。把你的构建逻辑的源码放到一个目录下然后Gradle就会自动编译并且把他们添加到构建脚本的classpath里。

•   buildSrc project. Drop the source for your build classes into a certain directory and Gradle automatically compiles them and includes them in the classpath of your build script. 

•   共享脚本。定义公共的配置在另外的脚本，并且应用这个脚本到多个工程，也可以跨越多个项目。

•   Shared scripts. Define common configuration in an external build, and apply the script to multiple projects, possibly across different builds. 

•   自定义任务。把构建逻辑放到一个自定义任务中并在多个项目重用这个任务。

•   Custom tasks. Put your build logic into a custom task, and reuse that task in multiple places.

•   自定义插件。把你的构建逻辑到一个自定义插件中，并且在多个项目中使用这个插件。这个插件必须在你的构建脚本的classpath中。你可以用构建源码的方法打包这个插件并把它包含在一个额外的目录里。

•   Custom plugins. Put your build logic into a custom plugin, and apply that plugin to multiple projects. The plugin must be in the classpath of your build script. You can achieve this either by using build sources or by adding an external library that contains the plugin. 

•   执行一个另外的构建。从当前项目执行一个另外Gradle的构建。

•   Execute an external build. Execute another Gradle build from the current build.

•   额外的lib目录。直接在你的构建文件中使用额外的lib目录。

•   External libraries. Use external libraries directly in your build file. 

## **60.1. 继承属性和方法**

60.1. Inherited properties and methods

一个项目构建脚本中定义的任何方法和属性也被所有的子项目可见。你可以利用这配置公共的配置，并且可以把方法抽取出来封装成一个方法并并子项目使用。

Any method or property defined in a project build script is also visible to all the sub-projects. You can use this to define common configurations, and to extract build logic into methods which can be reused by the sub-projects. 

例子60.1. 继承属性和方法

Example 60.1. Using inherited properties and methods

build.gradle
```
// Define an extra property
ext.srcDirName = 'src/java'

// Define a method
def getSrcDir(project) {
    return project.file(srcDirName)
}
child/build.gradle
task show << {
    // Use inherited property
    println 'srcDirName: ' + srcDirName

    // Use inherited method
    File srcDir = getSrcDir(project)
    println 'srcDir: ' + rootProject.relativePath(srcDir)
}

Output of gradle -q show
> gradle -q show
srcDirName: src/java
srcDir: child/src/java

```

## **60.2. 注入配置**

60.2. Injected configuration

你可以使用57.1节Section 57.1, “Cross project configuration” 和  57.2节 “Subproject configuration”中介绍的配置注入的方法把属性和方法注入到多个工程里。这相对继承来说是一个更好的选择因为其一注入是在构建脚本中执行这样你就可以注入不同的逻辑到不同的项目中，其二你可以多个配置例如仓库，插件，任务等等。下面的例子显示它是如何起作用的。

You can use the configuration injection technique discussed in Section 57.1, “Cross project configuration” and Section 57.2, “Subproject configuration” to inject properties and methods into various projects. This is generally a better option than inheritance, for a number of reasons: The injection is explicit in the build script, You can inject different logic into different projects, And you can inject any kind of configuration such as repositories, plug-ins, tasks, and so on. The following sample shows how this works. 

例子60.2 注入属性和方法

Example 60.2. Using injected properties and methods

build.gradle
```
subprojects {
    // Define a new property
    ext.srcDirName = 'src/java'

    // Define a method using a closure as the method body
    ext.srcDir = { file(srcDirName) }

    // Define a task
    task show << {
        println 'project: ' + project.path
        println 'srcDirName: ' + srcDirName
        File srcDir = srcDir()
        println 'srcDir: ' + rootProject.relativePath(srcDir)
    }
}

// Inject special case configuration into a particular project
project(':child2') {
    ext.srcDirName = "$srcDirName/legacy"
}
child1/build.gradle
// Use injected property and method. Here, we override the injected value
srcDirName = 'java'
def dir = srcDir()
Output of gradle -q show
> gradle -q show
project: :child1
srcDirName: java
srcDir: child1/java
project: :child2
srcDirName: src/java/legacy
srcDir: child2/src/java/legacy
```

## **60.3. 用额外的构建脚本配置项目**

60.3. Configuring the project using an external build script

你可以使用的额外的脚本配置当前的项目。Gradle所有的构建语言都可以在额外的脚本获得。你甚至可以在这个额外的配置文件中使用其他项目的脚本。

You can configure the current project using an external build script. All of the Gradle build language is available in the external script. You can even apply other scripts from the external script. 

例子60.3.  使用额外的构建脚本配置项目

Example 60.3. Configuring the project using an external build script

build.gradle
```
apply from: 'other.gradle'
other.gradle
println "configuring $project"
task hello << {
    println 'hello from other script'
}
Output of gradle -q hello
> gradle -q hello
configuring root project 'configureProjectUsingScript'
hello from other script
```

60.4. 在项目中添加源码

60.4. Build sources in the buildSrc project

当你运行Gradle，它会检查buildSrc目录是否存在，若存在Gradle会自动编译并测试源码并把路径添加到构建脚本的classpath里。你不需要提供另外的方法，这可以是一个你添加任务和插件的很好的地方。

When you run Gradle, it checks for the existence of a directory called buildSrc. Gradle then automatically compiles and tests this code and puts it in the classpath of your build script. You don't need to provide any further instruction. This can be a good place to add your custom tasks and plugins. 

对多个项目的构建可以只有一个buildSrc目录，但是必须是在项目的根目录下。

For multi-project builds there can be only one buildSrc directory, which has to be in the root project directory. 

下面的脚本就是Gradle默认应用到buildSrc项目的脚本。

Listed below is the default build script that Gradle applies to the buildSrc project:

图60.1. 默认的buildSrc构建脚本

Figure 60.1. Default buildSrc build script
```
apply plugin: 'groovy'

dependencies {
    compile gradleApi()
    compile localGroovy()
}
```

这意味着你可以只把你的源码放到这个目录下并且执行for a Java/Groovy project 转换（见表格23.4 “Table 23.4, “Java plugin - default project layout”）

This means that you can just put your build source code in this directory and stick to the layout convention for a Java/Groovy project (see Table 23.4, “Java plugin - default project layout”). 

如果你需要更普遍的适应性，你可以提供你自己的build.gradle文件。Gradle会不管是否有一个指定的脚本都会使用默认的构建脚本。这意味你只需要声明你需要的额外的元素。下面的例子。我们注意到这个例子不需要声明Gradle API的依赖，这个会被默认的脚本声明。

If you need more flexibility, you can provide your own build.gradle. Gradle applies the default build script regardless of whether there is one specified. This means you only need to declare the extra things you need. Below is an example. Notice that this example does not need to declare a dependency on the Gradle API, as this is done by the default build script: 

例子60.4. 自定义源码构建脚本

Example 60.4. Custom buildSrc build script

buildSrc/build.gradle
```
repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4.12'
}
```

这个buildSrc项目可以被多个项目构建，类似任何一个常规的其他项目。然而，所有实际运行的项目的classpath都必须在在该根项目的依赖里。你可以往任何你想产出的项目中添加这个配置。

The buildSrc project can be a multi-project build, just like any other regular multi-project build. However, all of the projects that should be on the classpath of the actual build must be runtime dependencies of the root project in buildSrc. You can do this by adding this to the configuration of each project you wish to export: 

例子60.5.    向根项目的buildSrc里添加子项目

Example 60.5. Adding subprojects to the root buildSrc project

buildSrc/build.gradle
```
rootProject.dependencies {
  runtime project(path)
}
```

注：该例子的源码可以在samples/multiProjectBuildSrc目录里找到。

Note: The code for this example can be found at samples/multiProjectBuildSrc in the ‘-all’ distribution of Gradle.

## **60.5. 从一个构建里运行另一个Gradle构建**

60.5. Running another Gradle build from a build

你可以使用GradleBuild任务，你既可以要么使用buildFile属性去指定执行哪个构建，也可以使用任务属性去指定执行哪些任务。

You can use the GradleBuild task. You can use either of the dir or buildFile properties to specify which build to execute, and the tasks property to specify which tasks to execute. 

例子60.6. 从一个构建执行另一个构建

Example 60.6. Running another build from a build

build.gradle
```
task build(type: GradleBuild) {
    buildFile = 'other.gradle'
    tasks = ['hello']
}
other.gradle
task hello << {
    println "hello from the other build."
}
Output of gradle -q build
> gradle -q build
hello from the other build.
```

60.6.构建脚本额外的依赖

60.6. External dependencies for the build script

如果你的构建脚本需要使用额外包，你可以自己把它们添加到构建脚本的classpath里，你使用buildscript()方法来完成这个，会把构建脚本声明的闭环当做参数传递过去。

If your build script needs to use external libraries, you can add them to the script's classpath in the build script itself. You do this using the buildscript() method, passing in a closure which declares the build script classpath. 

例子60.7. 为构建脚本声明另外的依赖

Example 60.7. Declaring external dependencies for the build script

build.gradle
```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'commons-codec', name: 'commons-codec', version: '1.2'
    }
}
```

传递到buildscript()方法闭环配置了一个ScriptHandler实例。你通过添加依赖到classpath配置来声明构建脚本的classpath。这和你声明java编译classpath是相同的方法。你可以使用章节51.4. Section 51.4, “How to declare your dependencies”中的任意一种依赖形式。

The closure passed to the buildscript() method configures a ScriptHandler instance. You declare the build script classpath by adding dependencies to the classpath configuration. This is the same way you declare, for example, the Java compilation classpath. You can use any of the dependency types described in Section 51.4, “How to declare your dependencies”, except project dependencies.

已经声明了构建脚本的classpath，你可以使用你构建脚本中的类像你使用classpath中的任何一个类一样。下面的例子是对上面例子的补充，使用构建脚本classpath中的类。

Having declared the build script classpath, you can use the classes in your build script as you would any other classes on the classpath. The following example adds to the previous example, and uses classes from the build script classpath.

例子60.8.  一个带额外的依赖的构建脚本

Example 60.8. A build script with external dependencies

build.gradle
```
import org.apache.commons.codec.binary.Base64

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'commons-codec', name: 'commons-codec', version: '1.2'
    }
}

task encode << {
    def byte[] encodedString = new Base64().encode('hello world\n'.getBytes())
    println new String(encodedString)
}
Output of gradle -q encode
> gradle -q encode
aGVsbG8gd29ybGQK
```

对多项目的构建，一个项目构建脚本中声明的依赖对其他子项目都是可见的。

For multi-project builds, the dependencies declared in the a project's build script, are available to the build scripts of all sub-projects. 

60.7. Ant可选的依赖

60.7. Ant optional dependencies

对于额外的依赖没有被ant可选的任务处理的原因我们也不是很理解。但是你可以轻易的用其他方式完成这件事情。

For reasons we don't fully understand yet, external dependencies are not picked up by Ant's optional tasks. But you can easily do it in another way. [26] 

例子60.9.  Ant 可选依赖

Example 60.9. Ant optional dependencies

build.gradle
```
configurations {
    ftpAntTask
}

dependencies {
    ftpAntTask("org.apache.ant:ant-commons-net:1.9.4") {
        module("commons-net:commons-net:1.4.1") {
            dependencies "oro:oro:2.0.8:jar"
        }
    }
}

task ftp << {
    ant {
        taskdef(name: 'ftp',
                classname: 'org.apache.tools.ant.taskdefs.optional.net.FTP',
                classpath: configurations.ftpAntTask.asPath)
        ftp(server: "ftp.apache.org", userid: "anonymous", password: "me@myorg.com") {
            fileset(dir: "htdocs/manual")
        }
    }
}
```

这也是一个客户端模块的很好的例子。Maven 中央仓库中的POM 文件没有为ant-commons-net任务这个用例提供正确的信息。

This is also a good example for the usage of client modules. The POM file in Maven Central for the ant-commons-net task does not provide the right information for this use case.

## **60.8. 总结**

60.8. Summary

Gradle提供了多种用户组织构建逻辑的方法。你可以选择适合你的需求的方式和不你不需要的方面的一个平衡点，避免冗余和难以维护。我们的经验是每个非常复杂的自定义构建在不同的构建之间很少被分享。其他的构建工具都是分割构建逻辑到单独的工程。Gradle省去了这方面你不必要的开销。

Gradle offers you a variety of ways of organizing your build logic. You can choose what is right for your domain and find the right balance between unnecessary indirections, and avoiding redundancy and a hard to maintain code base. It is our experience that even very complex custom build logic is rarely shared between different builds. Other build tools enforce a separation of this build logic into a separate project. Gradle spares you this unnecessary overhead and indirection. 

________________________________________

[25] Which might range from a single class to something very complex. 

[26] In fact, we think this is a better solution. Only if your buildscript and Ant's optional task need the same library would you have to define it twice. In such a case it would be nice if Ant's optional task would automatically pick up the classpath defined in the “gradle.settings” file. 

