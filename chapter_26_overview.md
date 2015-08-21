# **第二十六章. War 插件**
War 的插件继承自 Java 插件并添加了对组装 web 应用程序的 WAR 文件的支持。它禁用了 Java 插件生成默认的 JAR archive，并添加了一个默认的 WAR archive 任务。

## **26.1. 用法**
要使用 War 的插件，请在构建脚本中包含以下语句：

示例 26.1. 使用War 插件

build.gradle
```
apply plugin: 'war'
```

## **26.2. 任务**
War 插件向project 中添加了以下任务。

表 26.1. War 插件 - 任务

|任务名称	|依赖于	|类型	|描述|
|--
|war	|compile|	War	|组装应用程序 WAR 文件。|

War 插件向 Java 插件所加入的 tasks 添加了以下的依赖。

表 26.2. War 插件 - 额外的task 依赖

|任务名称	|依赖于|
|--
|assemble|	war|

图 26.1. War 插件 - tasks
![](https://docs.gradle.org/current/userguide/img/warPluginTasks.png)

## **26.3. 项目布局**
表 26.3. War 插件 - 项目布局

|目录|	意义|
|--
|from <s1>'src/main/webapp'</s1>	|Web 应用程序源代码|

## **26.4. 依赖管理**

War 插件添加了两个依赖配置： providedCompile和providedRuntime。虽然它们有各自的compile和runtime配置，但这些配置有相同的作用域，只是它们不会添加到 WAR 文件中。要特别注意，这些provided配置的传递使用。假设你添加commons-httpclient:commons-httpclient:3.0依赖到任何一个provided配置。这个依赖又依赖于commons-codec。这意味着 httpclient 和 commons-codec 都不会添加到你的 WAR 中，即使 commons-codec 是 compile 配置上的一个显示依赖。如果你不想要这种传递行为，只是把provided依赖声明成和commons-httpclient:commons-httpclient:3.0@jar一样。

## **26.5. 公约属性**
表26.4. War插件 ​​- 目录属性

|属性名称	|类型	|默认值	|描述|
|--
|webAppDirName	|String	|from <s1>'src/main/webapp'</s1>|	web 应用程序源目录的名称，是一个相对于项目目录的目录名称。|
|webAppDir	|File (read-only)|	projectDir/webAppDirName|	Web 应用程序的源目录。
这些属性由一个WarPluginConvention公约对象提供。|

## **26.6. War**
War task 的默认行为是将src/main/webapp的内容复制到archive 的根目录下。你的webapp目录自然可能包含一个WEB-INF子目录，这个子目录可能还再包含一个web.xml文件。已编译的类被编译进WEB-INF/classes。所有runtime [13]配置的依赖被复制到WEB-INF/lib。

另请参阅 War。

26.7. 自定义
下面是一个示例，展示了最重要的自定义选项：

示例 26.2. war 插件的自定义

build.gradle
```
configurations {
   moreLibs
}

repositories {
   flatDir { dirs "lib" }
   mavenCentral()
}

dependencies {
    compile module(":compile:1.0") {
        dependency ":compile-transitive-1.0@jar"
        dependency ":providedCompile-transitive:1.0@jar"
    }
    providedCompile "javax.servlet:servlet-api:2.5"
    providedCompile module(":providedCompile:1.0") {
        dependency ":providedCompile-transitive:1.0@jar"
    }
    runtime ":runtime:1.0"
    providedRuntime ":providedRuntime:1.0@jar"
    testCompile 'junit:junit:4.11'
}
    moreLibs ":otherLib:1.0"
}

war {
    from 'src/rootContent' // adds a file-set to the root of the archive
    webInf { from 'src/additionalWebInf' } // adds a file-set to the WEB-INF dir.
    classpath fileTree('additionalLibs') // adds a file-set to the WEB-INF/lib dir.
    classpath configurations.moreLibs // adds a configuration to the WEB-INF/lib dir.
    webXml = file('src/someWeb.xml') // copies a file to WEB-INF/web.xml
}
```
当然，你可以用一个定义了excludes 和 includes 的闭包来配置不同的文件集。


[13]runtime配置继承自compile配置。


