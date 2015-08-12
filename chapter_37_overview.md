# **第三十六章 OSGI 插件**

Chapter 36. The OSGi Plugin


OSGi插件提供了一个工厂类得方法用于去创建一个OsgiManifest对象，OsgiManifest继承于Manifest。欲了解更多有关通用Manifest的处理，参见章节22.14.1，“Manifest”。如果应用了Java插件，OSGi插件会用OsgiManifest对象代替默认的jar包下的manifest对象，被替换的manifest会被合并到这个新的OsgiManifest。


The OSGi plugin provides a factory method to create an OsgiManifest object. OsgiManifest extends Manifest. To learn more about generic manifest handling, see Section 22.14.1, “Manifest”. If the Java plugins is applied, the OSGi plugin replaces the manifest object of the default jar with an OsgiManifest object. The replaced manifest is merged into the new one.

OSGi的插件使得Peter Kriens BND tool得到了大量的使用。

The OSGi plugin makes heavy use of Peter Kriens BND tool.

## **36.1. 用法**

36.1. Usage

要使用OSGI插件，需首先在你的script里面引用它。

To use the OSGi plugin, include the following in your build script:

例子36.1. 使用OSGI插件

Example 36.1. Using the OSGi plugin

build.gradle
```
apply plugin: 'osgi'
```

## **36.2. 运行已引用的插件**

36.2. Implicitly applied plugins

应用最基本的java 插件

Applies the Java base plugin.

## **36.3. 任务**

36.3. Tasks

这个插件不添加任何新的任务。

This plugin does not add any tasks.

## **36.4. 依赖管理**

36.4. Dependency management

TBD

TBD

## **36.5.类型转换**

36.5. Convention object

OSGI插件添加了如下的类型转换：OsgiPluginConvention 

The OSGi plugin adds the following convention object: OsgiPluginConvention

### **36.5.1 属性转换**

36.5.1. Convention properties

OSGI不给项目添加任何属性转换

The OSGi plugin does not add any convention properties to the project.

### **36.5.2 方法转换**

36.5.2. Convention methods

OSGI插件添加了如下的方法转换。欲知更多细节，参加对象转换的API文档

The OSGi plugin adds the following methods. For more details, see the API documentation of the convention object.

表36.1.1 OSGI 方法

Table 36.1. OSGi methods



|方法| 返回值类型|    描述|
|--
|osgiManifest() | OsgiManifest |返回一个OsgiManifest对象|
|osgiManifest(Closure cl)  |  OsgiManifest |返回一个配置了关闭方法的OsgiManifest对象 |


|Method | Return Type |Description|
|--
|osgiManifest()  |OsgiManifest |   Returns an OsgiManifest object.|
|osgiManifest(Closure cl)  |  OsgiManifest |   Returns an OsgiManifest object configured by the closure.|

在类文件目录中类文件将会根据他们依赖的包和他们包含的包进行解析。在导入和导出的包的基础上OSGI Manifest 将被计算。如果类路径中包含OSGI的id，此bundle信息将会用于指定导入的包的版本信息。除了OsgiManifest已经明确的对象外你也可以添加相应的说明。

The classes in the classes dir are analyzed regarding their package dependencies and the packages they expose. Based on this the Import-Package and the Export-Package values of the OSGi Manifest are calculated. If the classpath contains jars with an OSGi bundle, the bundle information is used to specify version information for the Import-Package value. Beside the explicit properties of the OsgiManifest object you can add instructions.

例子36.2. 配置 OSGI Manifest.MF文件

Example 36.2. Configuration of OSGi MANIFEST.MF file

build.gradle
```
jar {
    manifest { // the manifest of the default jar is of type OsgiManifest
        name = 'overwrittenSpecialOsgiName'
        instruction 'Private-Package',
                'org.mycomp.package1',
                'org.mycomp.package2'
        instruction 'Bundle-Vendor', 'MyCompany'
        instruction 'Bundle-Description', 'Platform2: Metrics 2 Measures Framework'
        instruction 'Bundle-DocURL', 'http://www.mycompany.com'
    }
}
task fooJar(type: Jar) {
    manifest = osgiManifest {
        instruction 'Bundle-Vendor', 'MyCompany'    
    }
}
```

这个方法的第一个参数是property的key，其他的参数组成了他的值。欲知更多细节请参见BND 工具的说明。

The first argument of the instruction call is the key of the property. The other arguments form the value. To learn more about the available instructions have a look at the BND tool.

百度搜索[无线学院](http://wirelesscollege.cn)
