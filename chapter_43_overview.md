# **第四十三章. Build Announcements插件**

Chapter 43. The Build Announcements Plugin

build announcements插件正在开发中，值得注意的是在以后的Gradle版本中DSL与其他配置可以会变化。

build announcements插件在重要事件上使用announce插件发送本地通知

The build announcements plugin is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions. 

The build announcements plugin uses the announce plugin to send local announcements on important events in the build.

## **42.1. 用法**

42.1. Usage

在build脚本中使用build announcements 插件，方法如下：

To use the build announcements plugin, include the following in your build script:

例42.1 使用build announcements插件

Example 42.1. Using the build announcements plugin

build.gradle
```
apply plugin: 'build-announcements'
```

如果你想在哪里调整公告，就可以配置announce 插件来改变本地通知

That's it. If you want to tweak where the announcements go, you can configure the announce plugin to change the local announcer. 

也可以在init script中使用插件

You can also apply the plugin from an init script:

例42.2.在init script中使用build announcements插件

Example 42.2. Using the build announcements plugin from an init script

init.gradle
```
rootProject {
    apply plugin: 'build-announcements'
}
```
百度搜索[无线学院](http://wirelesscollege.cn)