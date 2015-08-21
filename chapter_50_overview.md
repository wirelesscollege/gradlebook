# **第50章. Java Gradle插件开发的插件**
# **Chapter 50. The Java Gradle Plugin Development Plugin**

Java Gradle插件开发的插件

The Java Gradle plugin development plugin

目前正在开发中。请注意，在以后的Gradle版本中DSL和其他配置可能会改变。 

The Java Gradle plugin development plugin is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.

The Java Gradle plugin development plugin可用于协助Gradle插件的开发。它会自动适用 Java插件,添加依赖编译配置的gradleApi()，并在jartask执行期间进行插件元数据的验证。

The Java Gradle Plugin development plugin can be used to assist in the development of Gradle plugins. It automatically applies theJava plugin, adds the gradleApi() dependency to the compile configuration and performs validation of plugin metadata during jartask execution.

## **50.1. 用法**
## **50.1. Usage**

使用the Java Gradle Plugin Development plugin时需在你的构建脚本中添加如下内容:

To use the Java Gradle Plugin Development plugin, include the following in your build script:

例50.1 使用Java Gradle Plugin Development plugin 

Example 50.1. Using the Java Gradle Plugin Development plugin

build.gradle
```
apply plugin: 'java-gradle-plugin'
```

为了使应用此插件时可以自动适用于java插件并且添加依赖于编译配置的 gradleApi() ，还需要用校验来修饰下jar任务

Applying the plugin automatically applies the Java plugin and adds the gradleApi() dependency to the compile configuration. It also decorates the jar task with validations.

需要执行如下校验

The following validations are performed:
•	需要有个为插件定义的插件描述符
•	插件描述符需包含一个实现类属性
•	实现类属性引用一个jar包中有效的类文件

任何一个失败的校验都会导致一个警告消息

•	There is a plugin descriptor defined for the plugin.
•	The plugin descriptor contains an implementation-class property.
•	The implementation-class property references a valid class file in the jar.

Any failed validations will result in a warning message.



