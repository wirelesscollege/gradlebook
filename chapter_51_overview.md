第51章 Gradle 测试工具

Chapter 51. The Gradle TestKit

Gradle测试工具正在完善，在后续版本中有些API和特性可能要变更。

The Gradle TestKit is currently incubating. Please be aware that its API and other characteristics may change in later Gradle versions.

Gradle 测试工具是一个依赖测试gradle 插件与构建逻辑的库。于此同时Gradle 测试工更聚焦于功能 测试。

The Gradle TestKit (a.k.a. just TestKit) is a library that aids in testing Gradle plugins and build logic generally. At this time, it is focused on functional testing. That is, testing build logic by exercising it as part of a programmatically executed build. Over time, the TestKit will likely expand to facilitate other kinds of tests.

51.1. Usage
To use the TestKit, include the following in your plugin's build:

Example 51.1. Declaring the TestKit dependency

build.gradle
dependencies {
    testCompile gradleTestKit()
}
The gradleTestKit() encompasses the classes of the TestKit, as well as the Gradle Tooling API client. It does not include a version ofJUnit,TestNG, or any other test execution framework. Such a dependency must be explicitly declared.

Example 51.2. Declaring the JUnit dependency

build.gradle
dependencies {
    testCompile 'junit:junit:4.12'
}
51.2. Functionally testing with the Gradle runner
The GradleRunner facilitates programmatically executing Gradle builds, and inspecting the result.

A contrived build can be created (e.g. programmatically, or from a template) that exercises the “logic under test”. The build can then be executed, potentially in a variety of ways (e.g. different combinations of tasks and arguments). The correctness of the logic can then be verified by asserting the following, potentially in combination:

The build's output;
The build's logging (i.e. console output);
The set of tasks executed by the build and their results (e.g. FAILED, UP-TO-DATE etc.).
After creating and configuring a runner instance, the build can be executed via the GradleRunner.build() or GradleRunner.buildAndFail() methods depending on the anticipated outcome.

The following demonstrates the usage of Gradle runner in a Java JUnit test:

Example 51.3. Using GradleRunner with JUnit

BuildLogicFunctionalTest.java
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
Any test execution framework can be used.

As Gradle build scripts are written in the Groovy programming language, and as many plugins are implemented in Groovy, it is often a productive choice to write Gradle functional tests in Groovy. Furthermore, it is recommended to use the (Groovy based) Spock test execution framework as it offers many compelling features over the use of JUnit.

The following demonstrates the usage of Gradle runner in a Groovy Spock test:

Example 51.4. Using GradleRunner with Spock

BuildLogicFunctionalTest.groovy
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
It is a common practice to implement any custom build logic (like plugins and task types) that is more complex in nature as external classes in a standalone project. The main driver behind this approach is bundle the compiled code into a JAR file, publish it to a binary repository and reuse it across various projects.

51.2.1. Getting the code under test into the test build

The runner uses the Tooling API to execute builds. An implication of this is that the builds are executed in a separate process (i.e. not the same process executing the tests). Therefore, the test build does not share the same classpath or classloaders as the test process and the code under test is not implicitly available to the test build.

At the moment the TestKit does not provide any convenient mechanism to inject the code under test into the test builds. This feature will be added to future versions.

In the meantime, it is possible to manually make the code under test available via some extra configuration. The following example demonstrates having the build generate a file denoting the implementation classpath of the code under test, and making it available at test runtime.

Example 51.5. Making the code under test classpath available to the tests

plugin/build.gradle
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
Note: The code for this example can be found at samples/testKit/testKitSpockClasspath in the ‘-all’ distribution of Gradle.
The tests can then read this value, and inject the classpath into the test build. The following is an example (in Groovy) of doing this from within a Spock Framework setup() method, which is analogous to a JUnit @Before method.

Example 51.6. Injecting the code under test classes into test builds

plugin/src/test/groovy/org/gradle/sample/BuildLogicFunctionalTest.groovy
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
Note: The code for this example can be found at samples/testKit/testKitSpockClasspath in the ‘-all’ distribution of Gradle.
This approach works well when executing the functional tests as part of the Gradle build. When executing the functional tests from an IDE, there are extra considerations. Namely, the classpath manifest file points to the class files etc. generated by Gradle and not the IDE. This means that after making a change to the source of the code under test, the source must be recompiled by Gradle. Similarly, if the effective classpath of the code under test changes, the manifest must be regenerated. In either case, executing the testClasses task of the build will ensure that things are up to date.

51.2.2. Controlling the build environment

The runner does not currently attempt to execute the test builds in an isolated environment. It also does not expose mechanism for fine grained control of environment variables etc. Future versions of the TestKit will provide better isolation and control.

Unless a specific Gradle user home dir is specified via the build arguments (i.e. -g or --gradle-user-home), Gradle will use its default of ~/.gradle. This may not be desirable. Any environmental configuration (e.g. ~/.gradle/gradle.properties) for the current user will be inherited. You may wish to isolate test builds by specifying an explicit Gradle user home to use via the build arguments.

51.2.3. The Gradle version used to test

The Gradle runner requires a Gradle distribution in order to execute the build. The TestKit does not depend on all of Gradle's implementation.

When a runner is created, it will attempt to find a Gradle distribution based on where the GradleRunner class was loaded from. That is, it is expected that the class was loaded from a Gradle distribution, as is the case when using the gradleTestKit() dependency declaration.

When using the runner as part of tests being executed by Gradle (e.g. executing the test task of a plugin project), the same distribution used to execute the tests will be used by the runner. When using the runner as part of tests being executed by an IDE, the same distribution of Gradle that was used when importing the project will be used. This means that the plugin will effectively be tested with the same version of Gradle that it is being built with.

If a Gradle distribution cannot be found, creation of the runner instance will fail.

Future versions of the TestKit will support more powerful distribution discovery, facilitating cross version testing.

51.2.4. Debugging build logic

The runner uses the Tooling API to execute builds. An implication of this is that the builds are executed in a separate process (i.e. not the same process executing the tests). Therefore, executing your tests in debug mode does not allow you to debug your build logic as you may expect. Any breakpoints set in your IDE will be not be tripped by the code being exercised by the test build.

At this time, there is no way to effectively debug the build logic under test. This, highly desirable, feature will be available in future versions of the test kit.

