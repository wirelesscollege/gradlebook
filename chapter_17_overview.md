# **第十七章. 日志**
日志是构建工具的主要界面。如果日志太多，真正的警告和问题容易被隐藏。另一方面，如果出了错，你需要找出相关的信息。Gradle 定义了6个日志级别，如表 17.1，“日志级别”所示。除了那些您通过可能会看到的日志级别之外，有两个 Gradle 特定日志级别。这两个级别分别是 QUIET 和 LIFECYCLE. 默认使用后面的这个日志级别，用于报告构建进度。

表 17.1. 日志级别

|Level |	用于 |
|--
|ERROR	|错误消息|
|QUIET|	重要的信息消息|
|WARNING	|警告消息|
|LIFECYCLE	|进度信息消息|
|INFO	|信息性消息|
|DEBUG|	调试消息|
## **17.1. 选择一个日志级别**
您可以使用表 17.2，“日志级别的命令行选项”中所示的命令行开关来选择不同的日志级别。在表 17.3，“栈跟踪的命令行选项”中，你可以看到影响栈跟踪日志的命令行开关。

表 17.2. 日志级别的命令行选项

|选项	|输出日志级别|
|--
|没有日志选项	|LIFECYCLE 及更高|
|-q or --quiet	|QUIET 及更高|
|-i or --info	|INFO 及更高|
|-d or --debug	|DEBUG 及更高|
表 17.3. 栈跟踪的命令行选项

|选项	|意义|
|--
|没有栈跟踪选项	|构建错误（如编译错误）时没有栈跟踪打印到控制台。只有在内部异常的情况下才打印栈跟踪。如果选择 DEBUG 日志级别，则总是输出截取后的栈跟踪信息。|
|-s or --stacktrace	|输出截断的栈跟踪。我们推荐使用这一个选项而不是打印全栈的跟踪信息。Groovy 的全栈跟踪非常冗长 （由于其潜在的动态调用机制，然而他们通常不包含你的的代码中哪里错了的相关信息。）|
|-S or--full-stacktrace	|打印全栈的跟踪信息。|
## **17.2. 编写自己的日志消息**
在构建文件，打印日志的一个简单方法是把消息写到标准输出中。Gradle 会把写到标准输出的所有内容重定向到它的日志系统的 QUIET 级别中。

示例17.1. 使用标准输出写日志

build.gradle
```
println 'A message which is logged at QUIET level'
```
Gradle 还提供了一个 logger属性给构建脚本，它是一个 Logger 实例。该接口扩展自 SLF4J的 Logger接口，并添加了几个 Gradle 的特有方法。下面是关于如何在构建脚本中使用它的示例：

示例 17.2. 编写自己的日志消息

build.gradle
```
logger.quiet('An info log message which is always logged.')
logger.error('An error log message.')
logger.warn('A warning log message.')
logger.lifecycle('A lifecycle info log message.')
logger.info('An info log message.')
logger.debug('A debug log message.')
logger.trace('A trace log message.')
```
您也可以在构建脚本中通过其他使用的类挂钩到 Gradle 的日志系统中（例如 buildSrc 目录中的类）。只需使用一个 SLF4J 的logger对象。你可以在构建脚本中，用与内置的logger同样的方式使用这个logger。

示例 17.3. 使用 SLF4J 编写日志消息

build.gradle
```
import org.slf4j.Logger
import org.slf4j.LoggerFactory

Logger slf4jLogger = LoggerFactory.getLogger('some-logger')
slf4jLogger.info('An info log message logged using SLF4j')
```
## **17.3. 使用外部工具和库记录日志**
Gradle 内部使用 Ant 和 Ivy。它们都有自己的日志系统。Gradle 将其日志输出重定向到 Gradle 的日志系统。从 Ant/Ivy 的日志级别到 Gradle 的日志级别是一对一的映射，除了 Ant/Ivy 的 TRACE级别之外，它是映射到Gradle 的 DEBUG 级别的。这意味着默认情况下， Gradle 日志级别将不会显示任何 Ant/Ivy 的输出，除非是错误或警告信息。

有很多的工具仍然在使用标准输出日志记录。默认情况下，Gradle 将标准输出重定向到QUIET日志级别，把标准错误输出重写向到 ERROR 级别。这种行为是可配置的。Project对象提供了一个LoggingManager，它允许您在计算构建脚本时，修改标准输出和错误重定向的日志级别。

示例 17.4. 配置标准输出捕获

build.gradle
```
logging.captureStandardOutput LogLevel.INFO
println 'A message which is logged at INFO level'
```
为能在任务执行过程中更改标准输出或错误的日志级别，task也提供了一个 LoggingManager。

示例 17.5. 对任务配置标准输出捕获

build.gradle
```
task logInfo {
    logging.captureStandardOutput LogLevel.INFO
    doFirst {
        println 'A task message which is logged at INFO level'
    }
}
```
Gradle 还提供了对 Java Util Logging，Jakarta Commons Logging 和 Log4j 的日志工具的集成。你生成的类使用这些日志记录工具输出的任何日志消息，都将被重定向到 Gradle 的日志系统。

17.4. 改变 Gradle 日志
您可以用您自己的 logging UI大量地替换 Gradle 的。你可以这样做，例如，如果您想要以某种方式自定义 UI ——以输出更多或更少的信息，或修改日志格式您可以使用 Gradle.useLogger() 方法替换这个logging。它可以在构建脚本，或 init 脚本，或通过内嵌的 API 访问。请注意它完全禁用 Gradle 的默认输出。下面是一个示例，在 init 脚本中修改任务执行和构建完成的日志打印。

示例 17.6 的例子. 自定义 Gradle 日志
```
init.gradle
useLogger(new CustomEventLogger())

class CustomEventLogger extends BuildAdapter implements TaskExecutionListener {

    public void beforeExecute(Task task) {
        println "[$task.name]"
    }

    public void afterExecute(Task task, TaskState state) {
        println()
    }
    
    public void buildFinished(BuildResult result) {
        println 'build completed'
        if (result.failure != null) {
            result.failure.printStackTrace()
        }
    }
}
```
gradle -I init.gradle build 的输出结果
```
> gradle -I init.gradle build
[compile]
compiling source

[testCompile]
compiling test source

[test]
running unit tests

[build]

build completed
```
你的logger可以实现下面列出的任何监听器接口。当你注册一个logger时，只能替换它实现的接口的日志记录。其他接口的日志记录是不变的。你可以在 55.6章节，“在构建脚本中响应生命周期”参阅更多有关监听器接口的信息。

BuildListener
ProjectEvaluationListener
TaskExecutionGraphListener
TaskExecutionListener
TaskActionListener

百度搜索[无线学院](http://wirelesscollege.cn)