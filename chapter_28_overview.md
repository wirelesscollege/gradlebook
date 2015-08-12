# **第二十八章. Jetty 插件**
Jetty 插件继承自 War 插件，并添加一些任务，这些任务可以让你在构建时部署你的 web 应用程序到一个 Jetty 的 web 嵌入式容器中。

## **28.1. 用法**
要使用 Jetty 的插件，请在构建脚本中包含以下语句：

示例 28.1. 使用 Jetty 插件

build.gradle
```
apply plugin: 'jetty'
```

## **28.2. 任务**
Jetty 插件定义了以下任务：

表 28.1. Jetty 插件 - 任务

|任务名称|	依赖于|	类型|	描述|
|--
|jettyRun|	compile|	jettyRun|	启动 Jetty 实例并将部署上 exploded web 应用程序。|
|jettyRunWar	|war	|jettyRunWar|	启动 Jetty 实例并将部署上 WAR 包。|
|jettyStop	|-	|jettyStop	|停止 Jetty 实例。|

图 28.1. Jetty 插件 - tasks
![](https://docs.gradle.org/current/userguide/img/jettyPluginTasks.png)


## **28.3. 项目布局**
Jetty 插件使用 和 War 插件相同的布局。

## **28.4. 依赖管理**
Jetty 插件并不定义任何依赖配置。

## **28.5. 公约属性**
Jetty 插件定义了下列公约属性：

表 28.2. Jetty插件 - 属性

|属性名称	|类型	|默认值	|描述|
|--
|contextPath|	String|	WAR |文件的base name	在 Jetty 容器里面的应用程序部署位置。|
|httpPort	|Integer	|8080	|Jetty 监听 HTTP 请求的 TCP 端口。|
|stopPort	|Integer	|null	|Jetty 监听 admin 请求的 TCP 端口。|
|stopKey	|String	|null	|当需要请求停止时，传递给 Jetty 的key。|
这些属性都由一个JettyPluginConvention公约对象提供。

百度搜索[无线学院](http://wirelesscollege.cn)
