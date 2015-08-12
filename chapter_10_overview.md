# **第 10 章.Web构建快速入门**

Chapter 10. Web Application Quickstart

-----

本章正在完善中

This chapter is a work in progress.

-----

本章介绍了Gradle对Web工程的相关支持.Gradle为Web开发提供了两个主要插件,War plugin 和 Jetty plugin. 其中War plugin继承自Java plugin,可以用来打war包.jetty plugin继承自War plugin作为工程部署的容器.

This chapter introduces some of the Gradle's support for web applications. Gradle provides two plugins for web application development: the War plugin and the Jetty plugin. The War plugin extends the Java plugin to build a WAR file for your project. The Jetty plugin extends the War plugin to allow you to deploy your web application to an embedded Jetty web container.

# **10.1. 打War包**

10.1. Building a WAR file

需要打包War文件,需要在脚本中使用War plugin:

To build a WAR file, you apply the War plugin to your project:

例 10.1. War plugin

Example 10.1. War plugin

build.gradle

```
apply plugin: 'war'
```
					
备注: 本示例代码可以在Gradle发行包中的 samples/webApplication/quickstart 路径下找到

Note: The code for this example can be found at samples/webApplication/quickstart which is in both the binary and source distributions of Gradle.

由于继承自Java插件,当你执行 gradle build时,将会编译、测试、打包你的工程. Gradle会在 src/main/webapp下寻找Web工程文件.编译后的classes文件以及运行时依赖也都会被包含在War包中.

This also applies the Java plugin to your project. Running gradle build will compile, test and WAR your project. Gradle will look for the source files to include in the WAR file in src/main/webapp . Your compiled classes, and their runtime dependencies are also included in the WAR file.

* Groovy web构建

Groovy web applications

在一个工程中你可以采用多个插件.比如你可以在web工程中同时使用War plugin和Groovy plugin. 插件会将Gradle依赖添加到你的War包中.

You can combine multiple plugins in a single project, so you can use the War and Groovy plugins together to build a Groovy based web application. The appropriate groovy libraries will be added to the WAR file for you.

# **10.2. Web工程启动**

10.2. Running your web application

要启动Web工程,只需使用Jetty plugin即可:

To run your web application, you apply the Jetty plugin to your project:

例 10.2. 采用Jetty plugin启动web工程

Example 10.2. Running web application with Jetty plugin

build.gradle

```
apply plugin: 'jetty'
```
					
由于Jetty plugin继承自War plugin.调用gradle jettyRun 将会把你的工程启动部署到jetty容器中. 调用gradle jettyRunWar会打包并启动部署到jetty容器中.

This also applies the War plugin to your project. Running gradle jettyRun will run your web application in an embedded Jetty web container. Running gradle jettyRunWar will build the WAR file, and then run it in an embedded web container.

TODO:url,端口,以及源文件位置都可以在脚本中进行指定.

TODO: which url, configure port, uses source files in place and can edit your files and reload.

10.3. 本章汇总

10.3. Summary

了解更多关于War plugin和Jetty plugin的应用请参阅第 26 章, War Plugin以及 第 28 章, Jetty Plugin . 你可以在发行包的samples/webApplication下找到更多示例.

You can find out more about the War plugin in Chapter 26, The War Plugin and the Jetty plugin in Chapter 28, The Jetty Plugin . You can find more sample Java projects in the samples/webApplication directory in the Gradle distribution.

百度搜索[无线学院](http://wirelesscollege.cn)