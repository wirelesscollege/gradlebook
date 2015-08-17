# **第38章 IDEA 插件**

Chapter 38. The IDEA Plugin

IDEA插件可以产生包括外部依赖（相关的源码和javadoc文件）和项目依赖在内的文件供Intellij IDE使用，以便可以在IDE里打开项目（通过File菜单里的Open Project 功能）。

The IDEA plugin generates files that are used by IntelliJ IDEA, thus making it possible to open the project from IDEA (File - Open Project). Both external dependencies (including associated source and Javadoc files) and project dependencies are considered.

事实上IDEA插件产生什么文件依赖于都用了其他插件：

What exactly the IDEA plugin generates depends on which other plugins are used:

表38.1.1 IDEA 插件行为

Table 38.1. IDEA plugin behavior

|插件	|描述|
|--
|None|	生成IDEA模块文件，如果工程是根工程也将生成IDEA工程和工作空间文件。|
|java |往模块和工程文件里添加java配置|

|Plugin	|Description|
|--
|None	|Generates an IDEA module file. Also generates an IDEA project and workspace file if the project is the root project.|
|Java |Adds Java configuration to the module and project files.|

IDEA插件其中的特点之一是可以根据需要定制界面，它提供了一系列标准的方法来添加或移除生成的文件。

One focus of the IDEA plugin is to be open to customization. The plugin provides a standardized set of hooks for adding and removing content from the generated files.

## **38.1 使用**

38.1. Usage

若要使用IDEA插件，需把这个添加到你的build脚本文件

To use the IDEA plugin, include this in your build script:

例子 38.1. 使用IDEA插件

Example 38.1. Using the IDEA plugin

build.gradle
```
apply plugin: 'idea'
```

IDEA插件添加了一定数量的任务到你的工程，你最经常用的任务是idea和clean idea 任务。

The IDEA plugin adds a number of tasks to your project. The main tasks that you will use are the idea and cleanIdea tasks. 

## **38.2 任务**

38.2 Tasks

IDEA给工程添加的任务如下表所示。我们注意到clean任务并不依赖于清理工作空间的任务，之所以这样是因为工作空间通常包括大量用户特意设置的临时数据并且这些数据不可能在IDEA外进行编辑。

The IDEA plugin adds the tasks shown below to a project. Notice that the clean task does not depend on the cleanIdeaWorkspace task. This is because the workspace typically contains a lot of user specific temporary data and it is not desirable to manipulate it outside IDEA. 

表38.2.    IDEA插件---任务集

Table 38.2.  IDEA plugin - Tasks

|任务名称|	依赖|	类型|	描述|
|--
|idea |ideaProject, ideaModule, ideaWorkspace|-	|生成所有的idea配置文件|
|cleanIdea |	cleanIdeaProject, cleanIdeaModule 	|删除|删除所有的idea配置文件|
|cleanIdeaProject |	- |	删除|删除idea工程文件|
|cleanIdeaModule |	- |	删除|删除idea工程模块|
|cleanIdeaWorkspace |	- |删除|删除idea工作空间|
|ideaProject |	- 	|生成idea工程|生成ipr文件，这个任务只会被添加到跟工程|
|ideaModule |	- |	生成idea模块|生成.iml文件|
|ideaWorkspace |	- 	|生成idea工作控件|生成.iws文件，只针对跟工程|


|Task name|	Depends on|	Type	|Description|
|--
|idea 	|ideaProject, ideaModule, ideaWorkspace	|-	|Generates all IDEA configuration files|
|cleanIdea 	|cleanIdeaProject, cleanIdeaModule 	|Delete|Removes all IDEA configuration files|
|cleanIdeaProject |	- 	|Delete|Removes the IDEA project file|
|cleanIdeaModule |	- |	Delete|Removes the IDEA module file|
|cleanIdeaWorkspace |	- |	Delete|Removes the IDEA workspace file|
|ideaProject |	- 	|GenerateIdeaProject|Generates the .ipr file. This task is only added to the root project.|
|ideaModule |	- 	|GenerateIdeaModule|Generates the .iml file|
|ideaWorkspace |	- |	GenerateIdeaWorkspace|Generates the .iws file. This task is only added to the root project.|

## **38.3. 配置**

38.3. Configuration

表38.3.  idea插件配置列表
Table 38.3. Configuration of the idea plugin

|模式	|参考名称	|描述|
|--
|IdeaModel |idea	|Idea插件的最高级别的配置，让配置以友好的DSL方式的生效|
|IdeaProject |idea.project|	允许配置项目基本信息|
|IdeaModule |idea.module	|允许配置项目模块信息|
|IdeaWorkspace |idea.workspace|	允许配置项目工作空间xml文件|

|Model|	Reference name	|Description|
|--
|IdeaModel |idea|	Top level element that enables configuration of the idea plugin in a DSL-friendly fashion|
|IdeaProject |idea.project|	Allows configuring project information|
|IdeaModule |idea.module	|Allows configuring module information|
|IdeaWorkspace |idea.workspace|	Allows configuring the workspace XML|

## **38.4. 定制生成的文件**

38.4. Customizing the generated files

IDEA插件提供了入口和方法来定制生成的内容，工作空间文件只有通过xml文件修改才能生效因为它对应的对象域基本上都是空的。

The IDEA plugin provides hooks and behavior for customizing the generated content. The workspace file can effectively only be manipulated via the withXml hook because its corresponding domain object is essentially empty.

任务识别出已存在的文件并把它们同已生成的进行合并。

The tasks recognize existing IDEA files, and merge them with the generated content.

## **38.4.1. 合并**

38.4.1. Merging

如果要生成的文件已经有同名文件同在则原来的文件将会被覆盖，这视特定模块而定。其余的没有同名文件的文件将会保留。

Sections of existing IDEA files that are also the target of generated content will be amended or overwritten, depending on the particular section. The remaining sections will be left as-is.

38.4.1.1 禁用完全覆盖的合并

38.4.1.1. Disabling merging with a complete overwrite

若要完全重写已存在的IDEA文件，需配合执行一下它配套的clean任务，例如Gradle里的“gradle cleanIdea idea”。如你想要把这个任务设置为默认执行，添加“tasks.idea.dependsOn(cleanIdea)”到你的build.xml文件中，以后便不用专程执行clean操作了。

To completely rewrite existing IDEA files, execute a clean task together with its corresponding generation task, like “gradle cleanIdea idea” (in that order). If you want to make this the default behavior, add “tasks.idea.dependsOn(cleanIdea)” to your build script. This makes it unnecessary to execute the clean task explicitly. 

这个方法同样适用于插件所产生的个人定制生成的文件，例如可以添加“gradle cleanIdeaModule ideaModule”到build.xml文件来完成.iml的配置。

This strategy can also be used for individual files that the plugin would generate. For instance, this can be done for the “.iml” file with “gradle cleanIdeaModule ideaModule”. 

## **38.4.2. 生命周期的入口**

38.4.2. Hooking into the generation lifecycle

插件为Gradle生成的元数据文件提供了对象数据模型。生命周期如下：

The plugin provides objects modeling the sections of the metadata files that are generated by Gradle. The generation lifecycle is as follows: 

1.文件被读取或如果文件不存在则使用Gradle默认生成的版本。
2.用代表这个文件的对象域作为参数来执行合并前方法
3.已存在的文件将会依照Gradle 环境配置或eclipse里的特定设置被合并
4.当执行合并方法时一个代表此文件的对象域将会被创建并持久化
5.withXml执行时一个只代表xml的文件将被生成
6.生成最终的xml文件

1.	The file is read; or a default version provided by Gradle is used if it does not exist
2.	The beforeMerged hook is executed with a domain object representing the existing file
3.	The existing content is merged with the configuration inferred from the Gradle build or defined explicitly in the eclipse DSL
4.	The whenMerged hook is executed with a domain object representing contents of the file to be persisted
5.	The withXml hook is executed with a raw representation of the XML that will be persisted
6.	The final XML is persisted

下表列出了每种模型使用的对象域：

The following table lists the domain object used for each of the model types: 

表38.4. idea插件方法

Table 38.4. Idea plugin hooks

|Model	|beforeMerged { arg -> } argument type	|whenMerged { arg -> } argument type	|withXml { arg -> } argument type|
|--
|IdeaProject|Project|Project|XmlProvider|
|IdeaModule|Module|Module|XmlProvider|
|IdeaWorkspace|Workspace|workspace|XmlProvider|

38.4.2.1 部分覆盖已存在的内容

38.4.2.1. Partial rewrite of existing content

完全覆盖会导致已存在的文件全部被丢弃，从而直接失去了在IDE里所做的任何修改。合并前方法允许只覆盖其中的一部分。下面的例子就是只移除了模块域对象的所有存在的依赖：

A complete rewrite causes all existing content to be discarded, thereby losing any changes made directly in the IDE. The beforeMerged hook makes it possible to overwrite just certain parts of the existing content. The following example removes all existing dependencies from the Module domain object: 

例子38.2.2 重写模块的一部分
Example 38.2. Partial Rewrite for Module

build.gradle
```
idea.project.ipr {
    beforeMerged { project ->
        project.modulePaths.clear()
    }
}
```

38.4.2.2.修改完全填充域对象
38.4.2.2. Modifying the fully populated domain objects

当合并方法允许你去操纵完全填充域对象时，通常这是定制IDEA文件的可选方式。下面例子告诉你怎样导出所有的IDEA模块依赖：

The whenMerged hook allows you to manipulate the fully populated domain objects. Often this is the preferred way to customize IDEA files. Here is how you would export all the dependencies of an IDEA module: 

例子38.4. 导出依赖
Example 38.4. Export Dependencies

build.gradle
```
idea.module.iml {
    whenMerged { module ->
        module.dependencies*.exported = true
    }
```

38.4.2.3. 修改XML表示
38.4.2.3. Modifying the XML representation

XML接口允许你在文件被写到磁盘之前修改XML中内存大小的配置。尽管Groovy 的XML支持补充设置，但这种方法远没有直接修改对象域来的方便。相应的，你可以获取已生成文件的整个的控制权，包括不是有对象域建模的部分。

The withXmlhook allows you to manipulate the in-memory XML representation just before the file gets written to disk. Although Groovy's XML support makes up for a lot, this approach is less convenient than manipulating the domain objects. In return, you get total control over the generated file, including sections not modeled by the domain objects. 

例子38.5. 定制XML 文件
Example 38.5. Customizing the XML

build.gradle
```
idea.project.ipr {
    withXml { provider ->
        provider.node.component
                .find { it.@name == 'VcsDirectoryMappings' }
                .mapping.@vcs = 'Git'
    }
}
```

## **38.5. 需进一步考虑的事情**
38.5. Further things to consider

IDEA文件产生的依赖的路径是绝对路径，如果你手动定义了一个指向Gradle缓存依赖的路径变量，那么IDEA将自动用这个变量代替绝对路径变量，你可以通过“idea.pathVariables”属性来定义路径变量，以便它可以适当的进行合并而不是建了重复的变量。

The paths of dependencies in the generated IDEA files are absolute. If you manually define a path variable pointing to the Gradle dependency cache, IDEA will automatically replace the absolute dependency paths with this path variable. you can configure this path variable via the “idea.pathVariables” property, so that it can do a proper merge without creating duplicates.

百度搜索[无线学院](http://wirelesscollege.cn)