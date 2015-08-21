# **第51章 Gradle 测试工具**

Chapter 51. The Gradle TestKit

Gradle测试工具正在完善，在后续版本中有些API和特性可能要变更。

The Gradle TestKit is currently incubating. Please be aware that its API and other characteristics may change in later Gradle versions.

TestKit是一个依赖gradle测试插件与逻辑构建的库。于此同时,它更聚焦于功能测试。有经验的测试构建逻辑作为程序动态化执行构建的一部分。随着时间的发展，TestKit将会支持更多种类的测试。

The Gradle TestKit (a.k.a. just TestKit) is a library that aids in testing Gradle plugins and build logic generally. At this time, it is focused on functional testing. That is, testing build logic by exercising it as part of a programmatically executed build. Over time, the TestKit will likely expand to facilitate other kinds of tests.

## **51.1. 使用**

51.1. Usage

使用TestKit，你需要在配置文件里加入如下代码：

To use the TestKit, include the following in your plugin's build:

例51.1. 处理TestKit依赖

Example 51.1. Declaring the TestKit dependency

build.gradle
```
dependencies {
    testCompile gradleTestKit()
}
```

gradleTestKit()方法将TestKit所有类加载进去，好比gradle工具api，但是它不包含junit、TestNG的任何版本或者任何测试执行框架。所以你需要的依赖还是需要公开声明。

The gradleTestKit() encompasses the classes of the TestKit, as well as the Gradle Tooling API client. It does not include a version ofJUnit,TestNG, or any other test execution framework. Such a dependency must be explicitly declared.

例51.2 处理Junit依赖

Example 51.2. Declaring the JUnit dependency

build.gradle
```
dependencies {
    testCompile 'junit:junit:4.12'
}
```

## **51.2. 使用Gradle runner进行功能测试**

51.2. Functionally testing with the Gradle runner

GradleRunner帮助程序动态化执行Gradle构建，并生成结果。

The GradleRunner facilitates programmatically executing Gradle builds, and inspecting the result.


A contrived build can be created (e.g. programmatically, or from a template) that exercises the “logic under test”. The build can then be executed, potentially in a variety of ways (e.g. different combinations of tasks and arguments). The correctness of the logic can then be verified by asserting the following, potentially in combination:

构建输出;

The build's output;

构建日志（例：控制台输出）;

The build's logging (i.e. console output);

任务集通过构建和结果来执行（例：FAIED,UP-TO-DATE 等）在创建和配置完runner初始化后，构建会在GradleRunner.build（）方法或者GradleRunner.buildAndFail（）后执行,依赖预先考虑的结果。

The set of tasks executed by the build and their results (e.g. FAILED, UP-TO-DATE etc.).
After creating and configuring a runner instance, the build can be executed via the GradleRunner.build() or GradleRunner.buildAndFail() methods depending on the anticipated outcome.

下面是GradleRunner在java Junit测试下执行的例子：

The following demonstrates the usage of Gradle runner in a Java JUnit test:

例：51.3 在Junit下使用GradleRunner

Example 51.3. Using GradleRunner with JUnit

BuildLogicFunctionalTest.java

```
import org.gradle.testkit.runner.BuildResult;
import org.gradle.testkit.runner.GradleRunner;
import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.TemporaryFolder;

import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Collections;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertTrue;

import static org.gradle.testkit.runner.TaskOutcome.*;

public class BuildLogicFunctionalTest {
    @Rule public final TemporaryFolder testProjectDir = new TemporaryFolder();
    private File buildFile;

    @Before
    public void setup() throws IOException {
        buildFile = testProjectDir.newFile("build.gradle");
    }

    @Test
    public void testHelloWorldTask() throws IOException {
        String buildFileContent = "task helloWorld {" +
                                  "    doLast {" +
                                  "        println 'Hello world!'" +
                                  "    }" +
                                  "}";
        writeFile(buildFile, buildFileContent);

        BuildResult result = GradleRunner.create()
            .withProjectDir(testProjectDir.getRoot())
            .withArguments("helloWorld")
            .build();

        assertTrue(result.getStandardOutput().contains("Hello world!"));
        assertEquals(result.task(":helloWorld").getOutcome(), SUCCESS);
    }

    private void writeFile(File destination, String content) throws IOException {
        BufferedWriter output = null;
        try {
            output = new BufferedWriter(new FileWriter(destination));
            output.write(content);
        } finally {
            if (output != null) {
                output.close();
            }
        }
    }
}
```

支持任何测试框架。

Any test execution framework can be used.

因为Gradle构建脚本都是使用Groovy编程语言实现的，而且很多插件也是使用Groovy来实现，所以为了更好的编写功能测试用例，我们推荐你使用Spock测试框架，该框架是基于Groovy语言来实现的，支持很多现有的测试工具特性，例如Junit。

As Gradle build scripts are written in the Groovy programming language, and as many plugins are implemented in Groovy, it is often a productive choice to write Gradle functional tests in Groovy. Furthermore, it is recommended to use the (Groovy based) Spock test execution framework as it offers many compelling features over the use of JUnit.

下面的例子告诉你如何使用Spock测试框架来写测试用例：

The following demonstrates the usage of Gradle runner in a Groovy Spock test:

例51.4 使用Spock

Example 51.4. Using GradleRunner with Spock

BuildLogicFunctionalTest.groovy

```
import org.gradle.testkit.runner.GradleRunner
import static org.gradle.testkit.runner.TaskOutcome.*
import org.junit.Rule
import org.junit.rules.TemporaryFolder
import spock.lang.Specification

class BuildLogicFunctionalTest extends Specification {
    @Rule final TemporaryFolder testProjectDir = new TemporaryFolder()
    File buildFile

    def setup() {
        buildFile = testProjectDir.newFile('build.gradle')
    }

    def "hello world task prints hello world"() {
        given:
        buildFile << """
            task helloWorld {
                doLast {
                    println 'Hello world!'
                }
            }
        """

        when:
        def result = GradleRunner.create()
            .withProjectDir(testProjectDir.root)
            .withArguments('helloWorld')
            .build()

        then:
        result.standardOutput.contains('Hello world!')
        result.task(":helloWorld").outcome == SUCCESS
    }
}
```

这是一个通用性较高得一个联系例子，支持你去实现自定义的测试逻辑，你可以使用类加载的方式，或者jar包方式来将其融入到项目里。

It is a common practice to implement any custom build logic (like plugins and task types) that is more complex in nature as external classes in a standalone project. The main driver behind this approach is bundle the compiled code into a JAR file, publish it to a binary repository and reuse it across various projects.

### *51.2.1 在测试构建中获取被测试代码*

51.2.1. Getting the code under test into the test build

runner使用工具API去执行构建，这种实现方式会使构建在一个单独的进程中执行。与测试执行进程不是相同的进程。那么测试构建不会共享相同的classpath或者classloader，所以测试进程和被测试进程在执行中暗中不能相互联系和沟通。

The runner uses the Tooling API to execute builds. An implication of this is that the builds are executed in a separate process (i.e. not the same process executing the tests). Therefore, the test build does not share the same classpath or classloaders as the test process and the code under test is not implicitly available to the test build.

当前TestKit还不支持任何方便得方式让你在测试构建中去注入被测试代码，这个功能将会在以后的版本中去实现。

At the moment the TestKit does not provide any convenient mechanism to inject the code under test into the test builds. This feature will be added to future versions.

同时，你需要额外的配置文件让测试代码的一些变量在被测程序中生效，请看下面的例子：

In the meantime, it is possible to manually make the code under test available via some extra configuration. The following example demonstrates having the build generate a file denoting the implementation classpath of the code under test, and making it available at test runtime.

例51.5，让classpath变量在被测程序中生效

Example 51.5. Making the code under test classpath available to the tests

plugin/build.gradle

```
// Write the plugin's classpath to a file to share with the tests
task createClasspathManifest {
    def outputDir = file("$buildDir/$name")

    inputs.files sourceSets.main.runtimeClasspath
    outputs.dir outputDir

    doLast {
        outputDir.mkdirs()
        file("$outputDir/plugin-classpath.txt").text = sourceSets.main.runtimeClasspath.join("\n")
    }
}

// Add the classpath file to the test runtime classpath
dependencies {
    testRuntime files(createClasspathManifest)
}

```

注：该例子代码你可以在samples/testKit/testKitSpockClasspath路径下找到，当然你需要下载Gradle发布版本。

Note: The code for this example can be found at samples/testKit/testKitSpockClasspath in the ‘-all’ distribution of Gradle.

测试能够读取classpatch的值，在测试构建中注入这个变量，下面是使用Spock setup方法来实现的该功能的一个例子：

The tests can then read this value, and inject the classpath into the test build. The following is an example (in Groovy) of doing this from within a Spock Framework setup() method, which is analogous to a JUnit @Before method.

例51.6 在测试构建中注入被测代码

Example 51.6. Injecting the code under test classes into test builds

plugin/src/test/groovy/org/gradle/sample/BuildLogicFunctionalTest.groovy

```
def setup() {
    buildFile = testProjectDir.newFile('build.gradle')

    def pluginClasspathResource = getClass().classLoader.findResource("plugin-classpath.txt")
    if (pluginClasspathResource == null) {
        throw new IllegalStateException("Did not find plugin classpath resource, run `testClasses` build task.")
    }

    def pluginClasspath = pluginClasspathResource.readLines()
        .collect { it.replace('\\', '\\\\') } // escape backslashes in Windows paths
        .collect { "'$it'" }
        .join(", ")

    // Add the logic under test to the test build
    buildFile << """
        buildscript {
            dependencies {
                classpath files($pluginClasspath)
            }
        }
    """
}
```

注：这个例子代码可以在samples/testKit/testKitSpockClasspath发现

Note: The code for this example can be found at samples/testKit/testKitSpockClasspath in the ‘-all’ distribution of Gradle.

这种工作就是把功能测试作为gradle构建的一部分，如果你是在IDE中去执行功能测试的，那么需要额外考虑。classpath manifest文件执行类文件，这个不是用IED来产生的，而是Gradle，所以如果你对被测代码做了改动一定要记得用Gradle来进行重新编译。同理，如果被测代码的classpath发生了改变，manifest文件也会被重新生成，其他方面，在执行测试任务前要保证所有的其他things也已经更新到最新。

This approach works well when executing the functional tests as part of the Gradle build. When executing the functional tests from an IDE, there are extra considerations. Namely, the classpath manifest file points to the class files etc. generated by Gradle and not the IDE. This means that after making a change to the source of the code under test, the source must be recompiled by Gradle. Similarly, if the effective classpath of the code under test changes, the manifest must be regenerated. In either case, executing the testClasses task of the build will ensure that things are up to date.

### *51.2.2 控制构建环境*

51.2.2. Controlling the build environment

runner执行测试构建都是在单独的环境内。没有好的方式去更改环境变量，在后续版本中，TestKit将提供更好的解决方案去控制。

The runner does not currently attempt to execute the test builds in an isolated environment. It also does not expose mechanism for fine grained control of environment variables etc. Future versions of the TestKit will provide better isolation and control.

如果特殊的gradle用户目录被配置了，Gradle仍然会使用它默认的.gradle配置文件，任何针对于当前用户环境变量的配置都会被继承，你可能希望通过特殊的用户来配置构建的参数，但是实际上不行。

Unless a specific Gradle user home dir is specified via the build arguments (i.e. -g or --gradle-user-home), Gradle will use its default of ~/.gradle. This may not be desirable. Any environmental configuration (e.g. ~/.gradle/gradle.properties) for the current user will be inherited. You may wish to isolate test builds by specifying an explicit Gradle user home to use via the build arguments.

### *51.2.3 那些gradle版本可以支持这种测试*

51.2.3. The Gradle version used to test

Gradle runner要求你必须使用Gradle的发布版来进行构建，TestKit不依赖所有Gradle的实现方式。

The Gradle runner requires a Gradle distribution in order to execute the build. The TestKit does not depend on all of Gradle's implementation.

当runner被创建的时候，它将自动加载gradle发布版本下的GradleRunner类，所以当使用GradleTestkit方式时，确认所需要的类已经被加载。

When a runner is created, it will attempt to find a Gradle distribution based on where the GradleRunner class was loaded from. That is, it is expected that the class was loaded from a Gradle distribution, as is the case when using the gradleTestKit() dependency declaration.

当runner作为测试部分来执行时，发布版使用runner来构建测试，当在IDE中runner被执行时，你需要导入Gradle相关的jar包。

When using the runner as part of tests being executed by Gradle (e.g. executing the test task of a plugin project), the same distribution used to execute the tests will be used by the runner. When using the runner as part of tests being executed by an IDE, the same distribution of Gradle that was used when importing the project will be used. This means that the plugin will effectively be tested with the same version of Gradle that it is being built with.

如果gradle发布包没有找到，那么构建将会失败。

If a Gradle distribution cannot be found, creation of the runner instance will fail.

以后的版本Testkit会支持更有用的发布版来支持跨版本测试

Future versions of the TestKit will support more powerful distribution discovery, facilitating cross version testing.

### *51.2.4 debug构建逻辑*

51.2.4. Debugging build logic

runner使用工具api来执行构建，构建的实现在单独的进程中，因此，执行测试在debug末实现，是不能debug你的构建逻辑的，任何IDE下的断电都不会在测试过程中被监控到。

The runner uses the Tooling API to execute builds. An implication of this is that the builds are executed in a separate process (i.e. not the same process executing the tests). Therefore, executing your tests in debug mode does not allow you to debug your build logic as you may expect. Any breakpoints set in your IDE will be not be tripped by the code being exercised by the test build.

当前，还没有任何有效的方式去debug你的测试逻辑，当然在未来的版本中我们希望debug能够有效实现。

At this time, there is no way to effectively debug the build logic under test. This, highly desirable, feature will be available in future versions of the test kit.

