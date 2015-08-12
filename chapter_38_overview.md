# **第38章． Eclipse插件**

Chapter 38. The Eclipse Plugins

Eclipse插件生成的文件被Eclipse IDE使用,使之能导入到Eclipse的项目中(File - Import... - Existing Projects into Workspace),包括外部依赖(包含相关的源码和Javadoc文件)以及项目依赖.

The Eclipse plugins generate files that are used by the Eclipse IDE, thus making it possible to import the project into Eclipse (File - Import... - Existing Projects into Workspace). Both external dependencies (including associated source and Javadoc files) and project dependencies are considered.

自1.0版本到gradle里程碑4,WTP-generating代码被重构到一个称为eclipse-wtp的插件中,因此假如你对WTP集成有兴趣则可以应用eclipse-wtp插件.否则应用eclipse插件就足够了.这种让Eclipse用户要求的变化有利于获取war或者ear插件，如此来代替使用Eclipse WTP。实质上Eclipse-wtp插件也应用了Eclipse插件,因此你不需要同时应用这两个插件.

Since version 1.0-milestone-4 of Gradle, the WTP-generating code was refactored into a separate plugin called eclipse-wtp. So if you are interested in WTP integration then only apply the eclipse-wtp plugin. Otherwise applying the eclipse plugin is enough. This change was requested by Eclipse users who take advantage of the war or ear plugins, but who don't use Eclipse WTP. Internally, the eclipse-wtp plugin also applies the eclipse plugin so you don't need to apply both of those plugins.

eclipse插件到底生成什么,依赖于其他插件的使用:

What exactly the eclipse plugin generates depends on which other plugins are used:

表38.1. Eclipse插件行为

Table 38.1. Eclipse plugin behavior

|Plugin	|Description|
|--
|None	|Generates minimal .project file.|
|Java	|Adds Java configuration to .project. Generates .classpath and JDT settings file.|
|Groovy	|Adds Groovy configuration to .project file.|
|Scala	|Adds Scala support to .project and .classpath files.|
|War	|Adds web application support to .project file.|
|Ear	|Adds ear application support to .project file.|

但是,eclipse-wtp插件总会生成所有WTP设置文件和增强性的.project文件.如果Java或war被应用的话，.classpath将会被扩展以获得适应与工具包或者web应用项目的包体结构。

However, the eclipse-wtp plugin always generates all WTP settings files and enhances the .project file. If a Java or War is applied, .classpath will be extended to get a proper packaging structure for this utility library or web application project.

Eclipse插件都是开放的定制并提供一套标准的钩子用于从生成的文件中添加或删除内容

Both Eclipse plugins are open to customization and provide a standardized set of hooks for adding and removing content from the generated files.

## **38.1.用法**

38.1. Usage

要使用Eclipse或Eclipse WTP插件,需在你的构建脚本中应增加一行:

To use either the Eclipse or the Eclipse WTP plugin, include one of the lines in your build script:

示例38.1. 使用Eclipse插件

Example 38.1. Using the Eclipse plugin

build.gradle
```
apply plugin: 'eclipse'
```

示例38.2. 使用Eclipse WTP插件

Example 38.2. Using the Eclipse WTP plugin

build.gradle
```
apply plugin: 'eclipse-wtp'
```

注意: 实际上,Eclipse-wtp插件也适用于eclipse插件,因此你不需要同时应用两者.

Note: Internally, the eclipse-wtp plugin also applies the eclipse plugin so you don't need to apply both.

这两个Eclipse插件添加一系列任务到你的项目,你将使用的主要任务是eclipse和cleanEclipse任务.

Both Eclipse plugins add a number of tasks to your projects. The main tasks that you will use are the eclipse and cleanEclipse tasks.

## **38.2.任务**

38.2. Tasks

Eclipse插件添加如下所示的任务到一个项目中:

The Eclipse plugins add the tasks shown below to a project.

Table38.2.Elipse 插件任务

Table 38.2. Eclipse plugin – tasks

|Task name	|Depends on|	Type|	Description|
|--
|eclipse	|all Eclipse configuration file generation tasks|	Task	|Generates all Eclipse configuration files|
|cleanEclipse	|all Eclipse configuration file clean tasks	|Delete	|Removes all Eclipse configuration files|
|cleanEclipseProject	|-	|Delete|	Removes the .project file.|
|cleanEclipseClasspath	|-	|Delete	|Removes the .classpath file.|
|cleanEclipseJdt	|-	|Delete	|Removes the .settings/org.eclipse.jdt.core.prefs file.|
|eclipseProject	|-	|GenerateEclipseProject	|Generates the .project file.|
|eclipseClasspath	|-	|GenerateEclipseClasspath|	Generates the .classpath file.|
|eclipseJdt	|-	|GenerateEclipseJdt	|Generates the .settings/org.eclipse.jdt.core.prefs file.|


表38.3. Eclipse WTP插件 - 添加任务

Table 38.3. Eclipse WTP plugin - additional tasks

|Task name|	Depends on	|Type	|Description|
|--
|cleanEclipseWtpComponent|	-	|Delete|	Removes the .settings/org.eclipse.wst.common.component file.|
|cleanEclipseWtpFacet|	-	|Delete|	Removes the .settings/org.eclipse.wst.common.project.facet.core.xml file.|
|eclipseWtpComponent|	-	|GenerateEclipseWtpComponent|	Generates the .settings/org.eclipse.wst.common.component file.|
|eclipseWtpFacet	|-	|GenerateEclipseWtpFacet|	Generates the .settings/org.eclipse.wst.common.project.facet.core.xml file.|

## **38.3. 配置**

38.3. Configuration

表38.4. Eclipse插件配置

Table 38.4. Configuration of the Eclipse plugins

|Model	|Reference name|	Description|
|--
|EclipseModel	|eclipse|	Top level element that enables configuration of the Eclipse plugin in a DSL-friendly fashion.|
|EclipseProject|	eclipse.project	|Allows configuring project information|
|EclipseClasspath|	eclipse.classpath|	Allows configuring classpath information.|
|EclipseJdt|	eclipse.jdt|	Allows configuring jdt information (source/target Java compatibility).|
|EclipseWtpComponent	|eclipse.wtp.component|	Allows configuring wtp component information only if eclipse-wtp plugin was applied.|
|EclipseWtpFacet	|eclipse.wtp.facet	|Allows configuring wtp facet information only if eclipse-wtp plugin was applied.|

## **38.4.自定义生成文件**

38.4. Customizing the generated files

Eclipse插件允许你自定义生成元数据文件.该插件为项目的Eclipse视图提供一个DSL配置模型对象.然后这些模型对象与现有的Eclipse XML数据合并,最终生成新的元数据.该模型对象提供较低级的钩子来处理表示域对象的前后合并的模型配置文件内容.他们还提供了一个非常低的钩子来直接处理在持久化之前的原始XML,微调和配置Eclipse和Eclipse WTP插件模型.

The Eclipse plugins allow you to customize the generated metadata files. The plugins provide a DSL for configuring model objects that model the Eclipse view of the project. These model objects are then merged with the existing Eclipse XML metadata to ultimately generate new metadata. The model objects provide lower level hooks for working with domain objects representing the file content before and after merging with the model configuration. They also provide a very low level hook for working directly with the raw XML for adjustment before it is persisted, for fine tuning and configuration that the Eclipse and Eclipse WTP plugins do not model.

## ***38.4.1.合并***

38.4.1. Merging

部分现有的Eclipse生成的目标文件将会被修正或重写,这依赖于特定的部分,剩下的部分将保持不变.

Sections of existing Eclipse files that are also the target of generated content will be amended or overwritten, depending on the particular section. The remaining sections will be left as-is.

38.4.1.1.用一个完整的重写禁用合并

38.4.1.1. Disabling merging with a complete rewrite

要完全重写现有的Eclipse文件,则需执行一次clean任务与corresponding generation任务,像"gradle cleanEclipse eclipse"（依次）.如果你想让这种行为变成默认行为,则需添加"tasks.eclipse.dependsOn(cleanEclipse)"到你的构建脚本.这样就不必要显示执行clean任务.

To completely rewrite existing Eclipse files, execute a clean task together with its corresponding generation task, like “gradle cleanEclipse eclipse” (in that order). If you want to make this the default behavior, add “tasks.eclipse.dependsOn(cleanEclipse)” to your build script. This makes it unnecessary to execute the clean task explicitly.

这种策略也可以用于该插件将生成的单个文件中,例如:".classpath"文件用"radle cleanEclipseClasspath eclipseClasspath"是可以做到的.

This strategy can also be used for individual files that the plugins would generate. For instance, this can be done for the “.classpath” file with “gradle cleanEclipseClasspath eclipseClasspath”.

## ***38.4.2.挂钩到生命周期***

38.4.2. Hooking into the generation lifecycle

该Eclipse插件提供的对象模型 由Gradle生成的部分Eclipse文件

The Eclipse plugins provide objects modeling the sections of the Eclipse files that are generated by Gradle. The generation lifecycle is as follows:

1.  读取文件,或由Gradle提供一个默认版本使用(如果它不存在)

1.	The file is read; or a default version provided by Gradle is used if it does not exist

2.  执行beforeMerged钩子用于代表现有文件域对象被执行

2.	The beforeMerged hook is executed with a domain object representing the existing file

3.  现有的内容被合并到Gradle构建配置或在Eclipse DSL中明确定义

3.	The existing content is merged with the configuration inferred from the Gradle build or defined explicitly in the eclipse DSL

4.  执行whenMerged钩子用于代表文件内容被持久化

4.	The whenMerged hook is executed with a domain object representing contents of the file to be persisted

5.  执行withXml钩子用于表示原始的XML将被持续执行

5.	The withXml hook is executed with a raw representation of the XML that will be persisted

6.  最终的XML持久化

6.	The final XML is persisted

以下列出了每个Eclipse模型类型的域对象：
The following table lists the domain object used for each of the Eclipse model types:

表38.5. 
Table 38.5. 钩子高级配置

|Model	|beforeMerged { arg -> } argument type	|whenMerged { arg -> } argument type|	withXml { arg -> } argument type	|withProperties { arg -> } argument type|
|--
|EclipseProject|Project	|Project|	XmlProvider|	-|
|EclipseClasspath	|Classpath|	Classpath|	XmlProvider	|-|
|EclipseJdt	|Jdt	|Jdt|	-	|java.util.Properties|
|EclipseWtpComponent	|WtpComponent	|WtpComponent	|XmlProvider|	-|
|EclipseWtpFacet	|WtpFacet	|WtpFacet	|XmlProvider|	-|

38.4.2.1. 重写部分现有内容

38.4.2.1. Partial overwrite of existing content

一个完整的重写将导致全部现有的内容被丢弃,从而失去了直接在IDE中所做的任意更改.作为一种选择,beforeMerged钩子使得可以直接覆盖现有内容的特定部分.下面的例子将从Classpath域对象中删除所有现有的依赖:

A complete overwrite causes all existing content to be discarded, thereby losing any changes made directly in the IDE. Alternatively, the beforeMerged hook makes it possible to overwrite just certain parts of the existing content. The following example removes all existing dependencies from the Classpath domain object:

示例38.3. Classpath的部分重写

Example 38.3. Partial Overwrite for Classpath

build.gradle
```
eclipse.classpath.file {
    beforeMerged { classpath ->
        classpath.entries.removeAll { entry -> entry.kind == 'lib' || entry.kind == 'var' }
    }
}
``

所得的.classpath文件将只包含Gradle生成的依赖项,但不是任何其他依赖项都有可能呈现在原始的文件中(在依赖表中,这也是默认的行为.) 其他.classpath文件的部分将会被保持原样或被合并,同样可以在.project文件中执行:

The resulting .classpath file will only contain Gradle-generated dependency entries, but not any other dependency entries that may have been present in the original file. (In the case of dependency entries, this is also the default behavior.) Other sections of the .classpath file will be either left as-is or merged. The same could be done for the natures in the .project file:

示例38.4. 项目部分重写

Example 38.4. Partial Overwrite for Project

build.gradle

```
eclipse.project.file.beforeMerged { project ->
    project.natures.clear()
}
```

38.4.2.2.修改完全填充的域对象

38.4.2.2. Modifying the fully populated domain objects

whenMerged钩子允许操作完全填充的域对象。通常这是定制Eclipse文件首选的方法.这里告诉你如何导出Eclipse项目的所有依赖：

The whenMerged hook allows to manipulate the fully populated domain objects. Often this is the preferred way to customize Eclipse files. Here is how you would export all the dependencies of an Eclipse project:

示例38.5. 依赖导出

Example 38.5. Export Dependencies

build.gradle
```
eclipse.classpath.file {
    whenMerged { classpath ->
        classpath.entries.findAll { entry -> entry.kind == 'lib' }*.exported = false
    }
}
```

38.4.2.3. 修改XML表示方法

38.4.2.3. Modifying the XML representation

withXmlhook允许操作之前的文件以XML表示方法写入到磁盘的内存中.虽然弥补了Groovy对XML支持,但这种方法比操作域对象还不太方便.作为回报你会得到完全控制生成的文件,包括不是域对象的部分模型。

The withXmlhook allows to manipulate the in-memory XML representation just before the file gets written to disk. Although Groovy's XML support makes up for a lot, this approach is less convenient than manipulating the domain objects. In return, you get total control over the generated file, including sections not modeled by the domain objects.

示例38.6.自定义XML

Example 38.6. Customizing the XML

build.gradle
```
apply plugin: 'eclipse-wtp'

eclipse.wtp.facet.file.withXml { provider ->
    provider.asNode().fixed.find { it.@facet == 'jst.java' }.@facet = 'jst2.java'
}
```

百度搜索[无线学院](http://wirelesscollege.cn)