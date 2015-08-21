# **Chapter 61. Writing Custom Plugins**

Gradle插件能打包可重用的构建逻辑，它能够在许多不同的项目和构建中使用。Gradle允许你实现自己定制的插件，所以你可以重用你自己的构建逻辑，并且可以与人共享。

A Gradle plugin packages up reusable pieces of build logic, which can be used across many different projects and builds. Gradle allows you to implement your own custom plugins, so you can reuse your build logic, and share it with others.

你可以用任何你喜欢的语言去实现定制插件，被提供的是最终编译成的字节码。对于这个例子，我们将要使用Groovy做为实现语言。如果你想要，你也可以使用java或者Scala替代。

You can implement a custom plugin in any language you like, provided the implementation ends up compiled as bytecode. For the examples here, we are going to use Groovy as the implementation language. You could use Java or Scala instead, if you want.

## **61.1. Packaging a plugin**

有许多你可以放插件源码的位置。

There are several places where you can put the source for the plugin.

Build script

你可以直接在构建脚本里包含插件源码。这类的好处是:插件是自动编译和包含在构建脚本类路径里,你无须做任何事。然而,在构建脚本外插件是不可见的,所以你不能在定义的构建脚本外重用插件。

You can include the source for the plugin directly in the build script. This has the benefit that the plugin is automatically compiled and included in the classpath of the build script without you having to do anything. However, the plugin is not visible outside the build script, and so you cannot reuse the plugin outside the build script it is defined in.

buildSrc project

你可以把插件的源代码放在目录rootProjectDir/buildSrc/src/main/groovy中。Gradle将注意编译和测试插件，使其可以在构建脚本类路径中执行。插件对于构建中使用的每个构建脚本都是可见的。然而,在构建外它是不可见的。所以你不能重用构建外的插件。

You can put the source for the plugin in the rootProjectDir/buildSrc/src/main/groovy directory. Gradle will take care of compiling and testing the plugin and making it available on the classpath of the build script. The plugin is visible to every build script used by the build. However, it is not visible outside the build, and so you cannot reuse the plugin outside the build it is defined in.

见62章、组织构建逻辑 可了解更多关于buildSrc project的详细信息.

See Chapter 62, Organizing Build Logic for more details about the buildSrc project.

Standalone project

你可以为你的插件创建一个单独的项目。这个项目可以生成和发布一个jar文件，你可以在多种构建中使用和共享。一般来说，这个jar文件可能会包含一些自定义的插件或绑定若干相关联的任务类或一些这两者的结合到一个单独的库文件中。

You can create a separate project for your plugin. This project produces and publishes a JAR which you can then use in multiple builds and share with others. Generally, this JAR might include some custom plugins, or bundle several related task classes into a single library. Or some combination of the two.
在我们的实例中，为了简单化我们将会以构建脚本中的插件开始。之后我们将看下创建一个独立的项目。
In our examples, we will start with the plugin in the build script, to keep things simple. Then we will look at creating a standalone project.

## **61.2. 编写一个简单的插件**
61.2. Writing a simple plugin

要创建一个自定义插件,您需要编写一个插件的实现。当项目使用插件时，Gradle实例化插件并调用插件实例的Plugin.apply()方法。项目对象作为参数传递,该插件可以使用它配置项目。下面的示例包含一个greeting插件,添加一个hello任务到项目里。

To create a custom plugin, you need to write an implementation of Plugin. Gradle instantiates the plugin and calls the plugin instance's Plugin.apply() method when the plugin is used with a project. The project object is passed as a parameter, which the plugin can use to configure the project however it needs to. The following sample contains a greeting plugin, which adds a hello task to the project.

例61.1  自定义插件

Example 58.1. A custom plugin

build.gradle
```
apply plugin: GreetingPlugin

class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        project.task('hello') << {
            println "Hello from the GreetingPlugin"
        }
    }
}
```

Output of gradle -q hello
```
> gradle -q hello
Hello from the GreetingPlugin
```

需要注意的是,一个给定插件会为应用它的每个项目创建一个新实例。还要注意,Plugin类是一个泛型类型。这个例子使他接收到Plugin类型作为类型参数。采取不同的类型参数写出特别的定制插件看似可以实现,但这将是不太可能的(除非有人找出更有创造性的事情要做)。

One thing to note is that a new instance of a given plugin is created for each project it is applied to. Also note that the Plugin class is a generic type. This example has it receiving the Plugin type as a type parameter. It's possible to write unusual custom plugins that take different type parameters, but this will be unlikely (until someone figures out more creative things to do here).

## **61.3. Getting input from the build**

大多数插件都需要从构建脚本获取一些配置。这样做的一个方法是使用扩展对象。Gradle项目都有一个关联的ExtensionContainer对象,帮助跟踪传递给插件的所有设置和属性。您可以捕获用户的输入,告诉关于你的插件扩展容器。捕获输入,只需添加一个Java Bean的使用类到扩展容器的列表。Groovy是插件很好的语言选择,因为普通老的Groovy对象包含Java Bean要求的所有getter和setter方法。

Most plugins need to obtain some configuration from the build script. One method for doing this is to use extension objects. The Gradle Project has an associated ExtensionContainer object that helps keep track of all the settings and properties being passed to plugins. You can capture user input by telling the extension container about your plugin. To capture input, simply add a Java Bean compliant class into the extension container's list of extensions. Groovy is a good language choice for a plugin because plain old Groovy objects contain all the getter and setter methods that a Java Bean requires.

让我们添加一个简单的扩展对象到项目中。这里我们添加一个greeting扩展对象到项目里，它允许你配置greeting

Let's add a simple extension object to the project. Here we add a greeting extension object to the project, which allows you to configure the greeting.

例61.2.  自定义插件扩展

Example 61.2. A custom plugin extension

build.gradle
```
apply plugin: GreetingPlugin

greeting.message = 'Hi from Gradle'

class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        // Add the 'greeting' extension object
        project.extensions.create("greeting", GreetingPluginExtension)
        // Add a task that uses the configuration
        project.task('hello') << {
            println project.greeting.message
        }
    }
}

class GreetingPluginExtension {
    def String message = 'Hello from GreetingPlugin'
}
```
```
Output of gradle -q hello
> gradle -q hello
Hi from Gradle
```

在这个例子中，GreetingPluginExtension是一个普通的老的Groovy对象，它含有一个message的字段。扩展对象被添加到插件列表中，名字为greeting。这个对象随后成为了项目的属性，名字和在扩展对象名字一样。

In this example, GreetingPluginExtension is a plain old Groovy object with a field called message. The extension object is added to the plugin list with the name greeting. This object then becomes available as a project property with the same name as the extension object.

通常,你需要给一个单独的插件指定若干个相关的属性。Gradle为每个扩展对象添加一个配置闭包块,所以你可以把设置分组。下面的例子展示了是如何工作的

Oftentimes, you have several related properties you need to specify on a single plugin. Gradle adds a configuration closure block for each extension object, so you can group settings together. The following example shows you how this works.

例61.3.  带有配置闭包的自定义插件

Example 61.3. A custom plugin with configuration closure

build.gradle
```
apply plugin: GreetingPlugin

greeting {
    message = 'Hi'
    greeter = 'Gradle'
}

class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        project.extensions.create("greeting", GreetingPluginExtension)
        project.task('hello') << {
            println "${project.greeting.message} from ${project.greeting.greeter}"
        }
    }
}

class GreetingPluginExtension {
    String message
    String greeter
}
```

Output of gradle -q hello
```
> gradle -q hello
Hi from Gradle
```

在这个例子中，很多设置使用greeting闭包被分组。构建脚本中闭包块的名字需要与扩展对象名字匹配。那么当闭包快被执行的时候，根据标准Groovy闭包委托特性，在扩展对象的字段将会被映射到闭包变量中。

In this example, several settings can be grouped together within the greeting closure. The name of the closure block in the build script (greeting) needs to match the extension object name. Then, when the closure is executed, the fields on the extension object will be mapped to the variables within the closure based on the standard Groovy closure delegate feature.

## **61.4. Working with files in custom tasks and plugins**

当开发自定义任务和插件,灵活地接受输入配置文件地址是个好办法。要做到这一点,您可以利用Project.file()方法尽可能晚地分解值到文件中。

When developing custom tasks and plugins, it's a good idea to be very flexible when accepting input configuration for file locations. To do this, you can leverage the Project.file() method to resolve values to files as late as possible.

例61.4   延迟评估文件属性

Example 61.4. Evaluating file properties lazily

build.gradle

```
class GreetingToFileTask extends DefaultTask {

    def destination

    File getDestination() {
        project.file(destination)
    }

    @TaskAction
    def greet() {
        def file = getDestination()
        file.parentFile.mkdirs()
        file.write "Hello!"
    }
}

task greet(type: GreetingToFileTask) {
    destination = { project.greetingFile }
}

task sayGreeting(dependsOn: greet) << {
    println file(greetingFile).text
}

ext.greetingFile = "$buildDir/hello.txt"
```

Output of gradle -q sayGreeting
```
> gradle -q sayGreeting
Hello!
```

在这个例子中,我们配置了greeting任务目标属性作为一个闭包,这是用Project.file()方法评估在最后一刻将闭包返回值转化为一个文件对象。您会注意到,在上面的示例中,我们在任务中配置好投入使用后才指定greetingFile的属性值。这种延迟评估的关键性好处在于当设置一个文件的属性的时候可接受任何值，然后在读取属性的时候分解值。

In this example, we configure the greet task destination property as a closure, which is evaluated with the Project.file() method to turn the return value of the closure into a file object at the last minute. You will notice that in the example above we specify the greetingFile property value after we have configured to use it for the task. This kind of lazy evaluation is a key benefit of accepting any value when setting a file property, then resolving that value when reading the property.

## **61.5. A standalone project**

现在我们将插件移到一个独立的项目,这样我们可以发布,并与他人分享。这个项目是一个简单的Groovy项目,可以产生一个包含插件类的jar包。这里是项目中一个简单的构建脚本。它适用于Groovy插件,并添加Gradle API作为编译时依赖项。

Now we will move our plugin to a standalone project, so we can publish it and share it with others. This project is simply a Groovy project that produces a JAR containing the plugin classes. Here is a simple build script for the project. It applies the Groovy plugin, and adds the Gradle API as a compile-time dependency.

例61.5  一个自定义插件的构建

Example 61.5. A build for a custom plugin

build.gradle

```
apply plugin: 'groovy'

dependencies {
    compile gradleApi()
    compile localGroovy()
}
```

注意：这个实例代码能够在gradle根目录下的samples/customPlugin/plugin找到。

Note: The code for this example can be found at samples/customPlugin/plugin in the ‘-all’ distribution of Gradle.

那么Gradle是怎样发现Plugin执行的？答案是你需要在jar中的META-INF/gradle-plugins目录中提供一个属性文件用来匹配你插件的id.

So how does Gradle find the Plugin implementation? The answer is you need to provide a properties file in the jar's META-INF/gradle-plugins directory that matches the id of your plugin.

例61.6   连接自定义插件

Example 61.6. Wiring for a custom plugin

src/main/resources/META-INF/gradle-plugins/org.samples.greeting.properties
implementation-class=org.gradle.GreetingPlugin

注意属性文件名匹配插件id且放在源文件夹中，实现类属性指明Plugin实现类。

Notice that the properties filename matches the plugin id and is placed in the resources folder, and that the implementation-class property identifies the Plugin implementation class.

### **61.5.1. Creating a plugin id**

插件id是完全受限，与java包的方式类似（即反向域名）。这有助于避免冲突且提供了使用类似所有制方式分组插件的方法。

Plugin ids are fully qualified in a manner similar to Java packages (i.e. a reverse domain name). This helps to avoid collisions and provides a way to group plugins with similar ownership.

你的插件id应该是一个组件的结合，它能反映所提供插件的命名空间及名称(合理的指向你或你的组织)。例如,如果你有一个Github账户名为“foo”和你的插件名为“bar”,一个合适的插件id名可能就是com.github.foo.bar。类似地,如果你的插件在baz组织开发,插件id可能是org.baz.bar。

Your plugin id should be a combination of components that reflect namespace (a reasonable pointer to you or your organization) and the name of the plugin it provides. For example if you had a Github account named “foo” and your plugin was named “bar”, a suitable plugin id might be com.github.foo.bar. Similarly, if the plugin was developed at the baz organization, the plugin id might be org.baz.bar.

插件id应遵从如下规则：

Plugin ids should conform to the following:

可能包含任何字母数字字符,'.', 和 '-'。

必须包含至少一个'.' 用来分隔插件名中的命名空间

命名空间按照惯例使用小字母反向域名惯例

按照惯例名字只使用小写字母

org.gradle和com.gradleware命名空间不能被使用

不能以'.'开头或结尾

不能包含连续的'.'字符（比如'..'）

May contain any alphanumeric character, '.', and '-'.

Must contain at least one '.' character separating the namespace from the name of the plugin.

Conventionally use a lowercase reverse domain name convention for the namespace.

Conventionally use only lowercase characters in the name.

org.gradle and com.gradleware namespaces may not be used.

Cannot start or end with a '.' character.

Cannot contain consecutive '.' characters (i.e. '..').

尽管插件Id和包名称存在着常规的相似性，但是包名称通常比插件id有必要更具体些。例如，添加 “gradle”作为你插件id的组件看起来是合理的，但由于插件id仅用于Gradle插件，这样做是多余的。一般情况下，一个命名空间指明了所有权，一个名字是一个好插件需要的所有。

Although there are conventional similarities between plugin ids and package names, package names are generally more detailed than is necessary for a plugin id. For instance, it might seem reasonable to add “gradle” as a component of your plugin id, but since plugin ids are only used for Gradle plugins, this would be superfluous. Generally, a namespace that identifies ownership and a name are all that are needed for a good plugin id.

### **61.5.2. Publishing your plugin**

如果在你组织发布插件内部使用，你可以像发布其他代码构建产物一样发布它。可以看下ivy和maven发布构建产物的章节

If you are publishing your plugin internally for use within your organization, you can publish it like any other code artifact. See the ivy and maven chapters on publishing artifacts.

如果你有兴趣发布插件使其能够在Gradle社区被广泛使用,你可以发布到Gradle插件门户中。这个网站提供搜索和收集Gradle社区提供的插件信息。看看这里的说明关于如何使你的插件发布到这个网站上。

If you are interested in publishing your plugin to be used by the wider Gradle community, you can publish it to the Gradle plugin portal. This site provides the ability to search for and gather information about plugins contributed by the Gradle community. See the instructions here on how to make your plugin available on this site.

### **61.5.3. Using your plugin in another project**

为在构建脚本时使用某插件,您需要添加插件类到构建脚本的类路径中。要做到这一点,你需要用一个“buildscript { }”块,如20.4节所述,“使用构建脚本块应用插件”。下面的例子显示当载有插件的JAR包已经发布到一个本地存储库时，你该如何做到这一点:

To use a plugin in a build script, you need to add the plugin classes to the build script's classpath. To do this, you use a “buildscript { }” block, as described in Section 20.4, “Applying plugins with the buildscript block”. The following example shows how you might do this when the JAR containing the plugin has been published to a local repository:

例61.7  在其他项目中使用自定义插件

Example 61.7. Using a custom plugin in another project

build.gradle
```
buildscript {
    repositories {
        maven {
            url uri('../repo')
        }
    }
    dependencies {
        classpath group: 'org.gradle', name: 'customPlugin',
                  version: '1.0-SNAPSHOT'
    }
}
apply plugin: 'org.samples.greeting'
```

或者，如果你的插件被发布到插件门户上，你可以使用孵化插件DSL（见20.5节,“使用DSL应用插件”)来应用插件
Alternatively, if your plugin is published to the plugin portal, you can use the incubating plugins DSL (see Section 20.5, “Applying plugins with the plugins DSL”) to apply the plugin:

Example 61.8. Applying a community plugin with the plugins DSL

build.gradle
```
plugins {
    id "com.jfrog.bintray" version "0.4.1"
}
```

### **61.5.4. Writing tests for your plugin**

当你测试你插件执行时候，可以使用ProjectBuilder类来创建Project实例使用

You can use the ProjectBuilder class to create Project instances to use when you test your plugin implementation.

例61.9 测试自定义插件

Example 61.9. Testing a custom plugin

```
src/test/groovy/org/gradle/GreetingPluginTest.groovy
class GreetingPluginTest {
    @Test
    public void greeterPluginAddsGreetingTaskToProject() {
        Project project = ProjectBuilder.builder().build()
        project.pluginManager.apply 'org.samples.greeting'

        assertTrue(project.tasks.hello instanceof GreetingTask)
    }
}
```
### **61.5.5. Using the Java Gradle Plugin development plugin**
61.5.5. Using the Java Gradle Plugin development plugin

您可以使用孵化Java Gradle插件开发插件来消除一些在你构建脚本中的样本声明并提供一些基本的插件元数据的验证。这个插件会自动应用java插件，添加gradleApi()依赖到编译配置,并执行插件元数据验证作为jar任务执行的一部分。

You can use the incubating Java Gradle Plugin development plugin to eliminate some of the boilerplate declarations in your build script and provide some basic validations of plugin metadata. This plugin will automatically apply the Java plugin, add the gradleApi() dependency to the compile configuration, and perform plugin metadata validations as part of the jar task execution.

例61.10  使用Java Gradle Plugin Development插件

Example 61.10. Using the Java Gradle Plugin Development plugin

build.gradle
```
apply plugin: 'java-gradle-plugin'
```
## **61.6. 维护多个domain objects**

61.6. Maintaining multiple domain objects

Gradle提供了一些实用程序类维护对象的集合,它们与Gradle构建语言融合得很好。

Gradle provides some utility classes for maintaining collections of objects, which work well with the Gradle build language.

例61.11 管理域对象

Example 61.11. Managing domain objects

build.gradle

```
apply plugin: DocumentationPlugin

books {
    quickStart {
        sourceFile = file('src/docs/quick-start')
    }
    userGuide {

    }
    developerGuide {

    }
}

task books << {
    books.each { book ->
        println "$book.name -> $book.sourceFile"
    }
}

class DocumentationPlugin implements Plugin<Project> {
    void apply(Project project) {
        def books = project.container(Book)
        books.all {
            sourceFile = project.file("src/docs/$name")
        }
        project.extensions.books = books
    }
}

class Book {
    final String name
    File sourceFile

    Book(String name) {
        this.name = name
    }
}
```

Output of gradle -q books
```
> gradle -q books
developerGuide -> /home/user/gradle/samples/userguide/organizeBuildLogic/customPluginWithDomainObjectContainer/src/docs/developerGuide
quickStart -> /home/user/gradle/samples/userguide/organizeBuildLogic/customPluginWithDomainObjectContainer/src/docs/quick-start
userGuide -> /home/user/gradle/samples/userguide/organizeBuildLogic/customPluginWithDomainObjectContainer/src/docs/userGuide
````

Project.container()方法创建NamedDomainObjectContainer实例,有很多有用管理和配置对象的方法。为了使用一种类型在任何project.container方法中,它必须公开一个名字是“name”的属性作为唯一的,且为常量的对象的名称。project.container(类)容器方法的变体创建新实例,试图调用类的构造函数，这个构造函数只有一个字符串参数并期望为对象的名字。请点击上面的链接查看project.container方法变量允许自定义实例化策略。

The Project.container() methods create instances of NamedDomainObjectContainer, that have many useful methods for managing and configuring the objects. In order to use a type with any of the project.container methods, it MUST expose a property named “name” as the unique, and constant, name for the object. The project.container(Class) variant of the container method creates new instances by attempting to invoke the constructor of the class that takes a single string argument, which is the desired name of the object. See the above link for project.container method variants that allow custom instantiation strategies.

Previous|Contents|Next
