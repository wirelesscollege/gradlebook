# **Chapter 42. The Announce Plugin**

## 第42章 通知插件

The Gradle announce plugin allows you to send custom announcements during a build. The following notification systems are supported:

Gradle通知插件允许你在构建中发送自定义通知，它支持如下平台：

Twitter
notify-send (Ubuntu)
Snarl (Windows)
Growl (Mac OS X)

## **42.1. Usage**

42.1使用

To use the announce plugin, apply it to your build script:

如果想去使用通知插件，你应该按照如下配置来应用这个插件:

Example 42.1. Using the announce plugin

build.gradle
```
apply plugin: 'announce'
```

Next, configure your notification service(s) of choice (see table below for which configuration properties are available):

下一步，配置你可选的通知服务（查看下面列表，看看哪种配置属性是有效的，需要的）

Example 42.2. Configure the announce plugin

build.gradle
```
announce {  
  username = 'myId'
  password = 'myPassword'
}
```

Finally, send announcements with the announce method:

最后,使用通知方法来发送相关通知。

Example 42.3. Using the announce plugin

build.gradle

```
task helloWorld << {  
    println "Hello, world!"
}  

helloWorld.doLast {  
    announce.announce("helloWorld completed!", "twitter")
    announce.announce("helloWorld completed!", "local")
}
```

The announce method takes two String arguments: The message to be sent, and the notification service to be used. The following table lists supported notification services and their configuration properties.

通知方法有2种String类型参数：被发送的消息体，以及被使用的通知服务名。下面的表格列出了我们支持的通知服务以及他们的配置属性。

Table 42.1. Announce Plugin Notification Services

|Notification Service|	Operating System|	Configuration Properties|	Further Information|
|--
|twitter	|Any	|username, password	||
|snarl	|Windows	|	||
|growl|	Mac OS X	|	 ||
|notify-send	|Ubuntu	|	Requires the notify-send package to be installed. Use sudo apt-get install libnotify-bin to install it.||
|local|	Windows, Mac OS X, Ubuntu	|	Automatically chooses between snarl, growl, and notify-send depending on the current operating system.| |

## **42.2. Configuration**

42.4 配置

See the AnnouncePluginExtension class in the API documentation.

请到API文档内查看AnnouncePluginExtension类