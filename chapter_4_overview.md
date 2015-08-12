# **第四章. 安装**

Chapter 4. Installing Gradle

## **4.1. 先决条件**

4.1. Prerequisites

1.5或更高版本的JDK,需要配置环境变量 JAVA_HOME 并将它指向你的JDK安装目录.Gradle会采用你环境变量中设置的JDK目录,可以用java −version进行检查. p.s:Gradle自带了Groovy库,所以无需事先安装Grvoovy,所有已经安装的Grvooy也将被Gradle忽略. 

Gradle requires a Java JDK to be installed. Gradle requires a JDK 1.5 or higher. Gradle ships with its own Groovy library, therefore no Groovy needs to be installed. Any existing Groovy installation is ignored by Gradle.Gradle uses whichever JDK it finds in your path (to check, use java -version). Alternatively, you can set the JAVA_HOME environment variable to point to the install directory of the desired JDK.

## **4.2. 下载**

4.2. Download

* 从Gralde官方网站下载Gradle的最新发行包

can download one of the Gradle distributions from the Gradle web site.

## **4.3. 解压**

4.3. Unpacking

下载的ZIP发行包包括如下内容:

The Gradle distribution comes packaged as a ZIP. The full distribution contains:

* 二进制Gradle文件

The Gradle binaries.

* 用户手册 (PDF 和 HTML 两种版本)

The user guide (HTML and PDF).

* DSL参考指南

DSL reference guide.

* API手册(javadoc 和 groovy doc)

The API documentation (Javadoc and Groovydoc).

* 样例，包括用户手册中的例子,一些完整的构建样例和更加复杂的构建脚本 

Extensive samples, including the examples referenced in the user guide, along with some complete and more complex builds you can use the starting point for your own build.

* 源代码.仅供参考使用,如果你想要自己来构建Gradle你需要从源代码仓库中检出发行版本源码,具体请查看Gradle官方主页.

The binary sources. This is for reference only. If you want to build Gradle you need to download the source distribution or checkout the sources from the source repository. See the Gradle web site for details.

## **4.4. 配置环境变量**

4.4. Environment variables

运行gradle必须将 GRADLE_HOME/bin 加入到你的 PATH 环境变量中.

For running Gradle, add GRADLE_HOME/bin to your PATH environment variable. Usually, this is sufficient to run Gradle.

## **4.5. 测试安装**

4.5. Running and testing your installation

运行如下命令来检查是否安装成功.该命令会显示当前的JVM版本和Gradle版本. 

**gradle -v** 
You run Gradle via the gradle command. To check if Gradle is properly installed just type **gradle -v**. The output shows Gradle version and also local environment configuration (groovy and jvm version, etc.). The displayed gradle version should match the distribution you have downloaded.

## **4.6. JVM 参数配置**

4.6. JVM options

Gradle运行时的JVM参数可以通过GRADLE_OPTS或JAVA_OPTS来设置.这些参数将会同时生效. JAVA_OPTS设置的参数将会同其它JAVA应用共享,一个典型的例子是可以在JAVA_OPTS中设置代理和GRADLE_OPTS设置内存参数. 同时这些参数也可以在Gradle或者Gradlew脚本的开头进行设置.

JVM options for running Gradle can be set via environment variables. You can use GRADLE_OPTS or JAVA_OPTS . Those variables can be used together. JAVA_OPTS is by convention an environment variable shared by many Java applications. A typical use case would be to set the HTTP proxy in JAVA_OPTS and the memory options in GRADLE_OPTS . Those variables can also be set at the beginning of the gradle or gradlew script.

百度搜索[无线学院](http://wirelesscollege.cn) 

