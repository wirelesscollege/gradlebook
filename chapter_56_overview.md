# **Chapter 54. Building native binaries(构建本地二进制文件)**

Gradle对构建本地二进制文件的支持正在开发中。请注意，DSL和其他配置在以后的Gradle版本中可能会改变。 

The Gradle support for building native binaries is currently incubating. Please be aware that the DSL and other configuration may change in later Gradle versions.

各种本地二进制插件通过使用c++\c以及其他语言添加对创建本地软件组件的支持，如可执行文件和共享库。Gradle提供给开发者商标权利和灵活性，以及依赖于更多能在传统的jvm开发空间中发现的管理实践。

The various native binary plugins add support for building native software components, such as executables or shared libraries, from code written in C++, C and other languages. While many excellent build tools exist for this space of software development, Gradle offers developers its trademark power and flexibility together with dependency management practices more traditionally found in the JVM development space.

## **54.1. Supported languages**

目前支持以下源语言:

The following source languages are currently supported:
•	C
•	C++
•	Objective-C
•	Objective-C++
•	Assembly
•	Windows resources

## **54.2. Tool chain support（工具链支持）**

Gradle提供了使用不同工具链执行相同构建的功能。当你建立一个本地二进制文件时,它将试图找到一个安装在你的设备里可以构建二进制的工具链。你可以准确地调整它如何工作,详细请参见54.15节,“Tool chains”。

Gradle offers the ability to execute the same build using different tool chains. When you build a native binary, Gradle will attempt to locate a tool chain installed on your machine that can build the binary. You can fine tune exactly how this works, see Section 54.15, “Tool chains” for details.

以下工具链支持：

The following tool chains are supported:

|Operating System	|Tool Chain	|Notes|
|--
|Linux	|GCC||

|Linux	|Clang||

|Mac OS X|	XCode|	Uses the Clang tool chain bundled with XCode.|
|Windows	|Visual C++|Windows XP and later, Visual C++ 2010 and later.|
|Windows	|GCC with Cywin 32|Windows XP and later.|
|Windows	|GCC with MinGW|Windows XP and later. Mingw-w64 is currently not supported.|

以下工具链非正式支持。他们通常工作很好,但未连续测试:

The following tool chains are unofficially supported. They generally work fine, but are not tested continuously:

|Operating System	|Tool Chain|	Notes|
|--
|Mac OS X|	GCC |from Macports	|
|Mac OS X	|Clang |from Macports	|
|Windows	|GCC with Cywin 64|Windows XP and later.|
|UNIX-like	|GCC||
|UNIX-like	|Clang||

## **54.3.安装工具链**

54.3. Tool chain installation

请注意,如果你正在使用GCC那么你目前需要安装支持c++,即使你不是从c++源构建。这个警告将在未来Gradle版本删除。

Note that if you are using GCC then you currently need to install support for C++, even if you are not building from C++ source. This caveat will be removed in a future Gradle version.

为了构建本地二进制文件，你需要安装一个兼容的工具链。

To build native binaries, you will need to have a compatible tool chain installed:

### **54.3.1. Windows**

为了在windows上构建，需要安装一个兼容的Visual Studio版本。本地插件将会检测到Visual Studio的安装，并选择最新的版本。没有必要浪费时间纠结在环境变量和批处理脚本上。通过Cygwin shell 和Windows命令行处理的很好。
或者你可以安装携带GCC或者MinGW环境的Gygwin工具.Clang目前还不支持

To build on Windows, install a compatible version of Visual Studio. The native plugins will discover the Visual Studio installations and select the latest version. There is no need to mess around with environment variables or batch scripts. This works fine from a Cygwin shell or the Windows command-line.
Alternatively, you can install Cygwin with GCC or MinGW. Clang is currently not supported.

### **54.3.2. OS X**

为了在OS X上构建，你需要安装XCode.本地插件通过系统路径会检测到Code的安装。

To build on OS X, you should install XCode. The native plugins will discover the XCode installation using the system PATH.

The native plugins also work with GCC and Clang bundled with Macports. To use one of the Macports tool chains, you will need to make the tool chain the default using theport select command and add Macports to the system PATH.

### **54.3.3. Linux**

为了能在Linuc上构建，需安装一个兼容的GCC或Clang版本。本地插件通过系统了路径会发现CC或Clang。

To build on Linux, install a compatible version of GCC or Clang. The native plugins will discover GCC or Clang using the system PATH.

## **54.4.组件模型**

54.4. Component model

为使用Gradle构建本地二进制文件,你的项目应该定义一个或多个本地组件。每个组件代表Gradle应该构建的一个可执行文件或者是库文件。一个项目可以定义任何数量的组件。Gradle默认不定义任何组件。

To build native binaries using Gradle, your project should define one or more native components. Each component represents either an executable or a library that Gradle should build. A project can define any number of components. Gradle does not define any components by default.

对于每个组件，Gradle定义了一个组件构建使用的每种语言的源码集。源码集基本是一组包含源码文件的目录的集合。例如，当你使用c插件，定义一个名为helloworld的库，Gradle就会默认定义一个源码集，包含了c的源文件在src/helloworld/c目录中。它将使用这些源文件来构建helloworld库。下面内容将会更加详细描述。

For each component, Gradle defines a source set for each language that the component can be built from. A source set is essentially just a set of source directories containing source files. For example, when you apply the c plugin and define a library called helloworld, Gradle will define, by default, a source set containing the C source files in thesrc/helloworld/c directory. It will use these source files to build the helloworld library. This is described in more detail below.

对于每一个组件,它定义了一个或多个二进制文件作为输出。为了构建二进制文件，Gradle会为组件定义源文件，编译成适用于源语言并将结果链接到二进制文件中。对于一个可执行组件，Gradle可以产生可执行的二进制文件。例如，当你定义一个名为helloworld的库并且在Linux系统中构建，Gradle会默认的生成库文件helloworld.so和helloworld.a。

For each component, Gradle defines one or more binaries as output. To build a binary, Gradle will take the source files defined for the component, compile them as appropriate for the source language, and link the result into a binary file. For an executable component, Gradle can produce executable binary files. For a library component, Gradle can produce both static and shared library binary files. For example, when you define a library called helloworld and build on Linux, Gradle will, by default, producelibhelloworld.so and libhelloworld.a binaries.

在许多情况下，一个组件可能产生多个二进制文件。这些二进制文件可能非常依赖于用来构建使用的工具链，这些工具链由编译器/链接器、依赖物或额外的源码文件提供。每个本地产生的二进制组件称为变异。二进制变量将在下面详细讨论。

In many cases, more than one binary can be produced for a component. These binaries may vary based on the tool chain used to build, the compiler/linker flags supplied, the dependencies provided, or additional source files provided. Each native binary produced for a component is referred to as variant. Binary variants are discussed in detail below.

## **54.5.并行编译**

54.5. Parallel Compilation

默认情况下Gradle使用单一构建职工池同时编译和链接本地组件。不需要任何特殊配置启用并行构建。

Gradle uses the single build worker pool to concurrently compile and link native components, by default. No special configuration is required to enable concurrent building.

默认情况下,工池的大小是由构建机器上可用的处理器数量决定的（如同构建JVM报道的那样）。为了明确地设置工人数量可使用--max-workers命令选项或者 org.gradle.workers.max系统属性，一般没有必要更改它的默认设置。

By default, the worker pool size is determined by the number of available processors on the build machine (as reported to the build JVM). To explicitly set the number of workers use the --max-workers command-line option or org.gradle.workers.max system property. There is generally no need to change this setting from its default.

构建工人池被所有构建任务共享。这就意味着当使用并行项目执行的时候，并行个体编译操作的最大数量不会增加。例如，如果构建机器有4个处理器内核10个项目并行编译，Gradle将会仅使用共4个工人，而不是40.

The build worker pool is shared across all build tasks. This means that when using parallel project execution, the maximum number of concurrent individual compilation operations does not increase. For example, if the build machine has 4 processing cores and 10 projects are compiling in parallel, Gradle will only use 4 total workers, not 40.

## **54.6. 构建库**

54.6. Building a library

为构建一个静态或共享库，需在组件容器中定义一个库组件。以下的例子定义了一个名叫hello的库：

To build either a static or shared native library, you define a library component in the components container. The following sample defines a library called hello:

例54.1 定义一个库组件

Example 54.1. Defining a library component
build.gradle
```
model {
    components {
        hello(NativeLibrarySpec)
    }
}
```

库组件通过使用NativeLibrarySpec标识。每个库组件可以产生至少一个共享库二进制(SharedLibraryBinarySpec)和至少一个静态库二进制(StaticLibraryBinarySpec)

A library component is represented using NativeLibrarySpec. Each library component can produce at least one shared library binary (SharedLibraryBinarySpec) and at least one static library binary (StaticLibraryBinarySpec).

## **54.7. 构建可执行程序**

54.7. Building an executable

为构建本地可执行程序，你在组件容器中定义一个可执行的组件。以下的例子定义了一个名为main的可执行程序

To build a native executable, you define an executable component in the components container. The following sample defines an executable called main:

例54.2 定义可执行组件

Example 54.2. Defining executable components

build.gradle

```
model {
    components {
        main(NativeExecutableSpec) {
            sources {
               c.lib library: "hello"
            }
        }
    }
}
```

一个可执行组件使用NativeExecutableSpec标识，每个可执行组件可产生至少一个可执行二进制文件(NativeExecutableBinarySpec)
An executable component is represented using NativeExecutableSpec. Each executable component can produce at least one executable binary (NativeExecutableBinarySpec).

对于每个定义的组件，Gradle都添加了一个具有相同名称的FunctionalSourceSet。每种这样的功能性源码集包含针对于项目支持的每种语言的特定源码集合。

For each component defined, Gradle adds a FunctionalSourceSet with the same name. Each of these functional source sets will contain a language-specific source set for each of the languages supported by the project.

## **54.8. Tasks**

对于每个由构建产生的NativeBinarySpec，一个生命周期任务被构建可用来创建二进制以及其他做编译、链接、装配二进制实际工作的任务。

For each NativeBinarySpec that can be produced by a build, a single lifecycle task is constructed that can be used to create that binary, together with a set of other tasks that do the actual work of compiling, linking or assembling the binary.

|Component Type	|Native Binary Type	|Lifecycle task|	Location of created binary|
|--
|NativeExecutableSpec| NativeExecutableBinarySpec|${component.name}Executable|	${project.buildDir}/binaries/${component.name}Executable/${component.name}|
|NativeLibrarySpec| SharedLibraryBinarySpec${component.name}SharedLibrary	|${project.buildDir}/binaries/${component.name}SharedLibrary/lib${component.name}.so|
|NativeLibrarySpec| StaticLibraryBinarySpec |${component.name}StaticLibrary	 |${project.buildDir}/binaries/${component.name}StaticLibrary/${component.name}.a|

### **54.8.1. 使用共享库**

54.8.1. Working with shared libraries

对于每一个可执行的二进制,cpp插件提供了一个安装$ { binary.name }任务,它开发了安装装可执行文件程序,连同它所需要的共享库。这让你运行可执行文件时没必要非在最后的位置安装共享库。

For each executable binary produced, the cpp plugin provides an install${binary.name} task, which creates a development install of the executable, along with the shared libraries it requires. This allows you to run the executable without needing to install the shared libraries in their final locations.

## **54.9. 找出更多关于您的项目的信息**

54.9. Finding out more about your project

Gradle提供了可以展示你项目产生的组件和二进制文件更详细的信息报告，可以通过命令行运行得到。要使用这个报告,只需运行gradle组件。下面是一个运行样例项目报告的例子:

Gradle provides a report that you can run from the command-line that shows some details about the components and binaries that your project produces. To use this report, just run gradle components. Below is an example of running this report for one of the sample projects:

例54.3 组件报告
Example 54.3. The components report

Output of gradle components
````
> gradle components
:components

------------------------------------------------------------
Root project
------------------------------------------------------------

Native library 'hello'
----------------------

Source sets
    C++ source 'hello:cpp'
        src/hello/cpp

Binaries
    Shared library 'hello:sharedLibrary'
        build using task: :helloSharedLibrary
        platform: current
        build type: debug
        flavor: default
        tool chain: Tool chain 'clang' (Clang)
        shared library file: build/binaries/helloSharedLibrary/libhello.dylib
    Static library 'hello:staticLibrary'
        build using task: :helloStaticLibrary
        platform: current
        build type: debug
        flavor: default
        tool chain: Tool chain 'clang' (Clang)
        static library file: build/binaries/helloStaticLibrary/libhello.a

Native executable 'main'
------------------------

Source sets
    C++ source 'main:cpp'
        src/main/cpp

Binaries
    Executable 'main:executable'
        build using task: :mainExecutable
        install using task: :installMainExecutable
        platform: current
        build type: debug
        flavor: default
        tool chain: Tool chain 'clang' (Clang)
        executable file: build/binaries/mainExecutable/main

Note: currently not all plugins register their components, so some components may not be visible here.

BUILD SUCCESSFUL

Total time: 1 secs
```

## **54.10  语言支持**

54.10. Language support

目前，Gradle支持由下列语言任意结合构建本地二进制文件。一个本地二进制文件包含一个或多个名为FunctionalSourceSet的实例（如'main', 'test'等），每个实例都包含LanguageSourceSets，它针对每种语言含有一个源码文件。

Presently, Gradle supports building native binaries from any combination of source languages listed below. A native binary project will contain one or more named FunctionalSourceSet instances (eg 'main', 'test', etc), each of which can contain LanguageSourceSets containing source files, one for each language.
•	C
•	C++
•	Objective-C
•	Objective-C++
•	Assembly
•	Windows resources

### **54.10.1. C++ sources**

C++语言支持依赖于cpp插件。

C++ language support is provided by means of the 'cpp' plugin.

例54.4 cpp插件

Example 54.4. The 'cpp' plugin
build.gradle
```
apply plugin: 'cpp'C++源码
```

C++源码列入在本地二进制文件里是通过CppSourceSet来支持的。它定义了一组c++源码文件和有选择的定义一组输出头文件（对一个库而言）。对于任何指定组件CppSourceSet默认包含.cpp源码文件在src/${name}/cpp中，头文件在src/${name}/headers里。

C++ sources to be included in a native binary are provided via a CppSourceSet, which defines a set of C++ source files and optionally a set of exported header files (for a library). By default, for any named component the CppSourceSet contains .cpp source files in src/${name}/cpp, and header files in src/${name}/headers.

app插件为每个CppSourceSet定义默认的位置，然而为了不同的项目布局也可以延伸或覆盖这些默认项。

While the cpp plugin defines these default locations for each CppSourceSet, it is possible to extend or override these defaults to allow for a different project layout.

Example 54.5. C++ source set
build.gradle
```
sources {
    cpp {
        source {
            srcDir "src/source"
            include "**/*.cpp"
        }
    }
}
```

对于命名为'main'的库，在src/main/headers中的头文件被认为是共有的或者是可输出的头。那种不能被输出的头文件应该被放在目录src/main/cpp 里。（但是需要注意的事这样的头文件相对于包含他们的文件应该总是被引用的方式）

For a library named 'main', header files in src/main/headers are considered the “public” or “exported” headers. Header files that should not be exported should be placed inside the src/main/cpp directory (though be aware that such header files should always be referenced in a manner relative to the file including them).

### **54.10.2. C sources**

c语言支持依赖于‘c’插件

C language support is provided by means of the 'c' plugin.

例54.6 c插件

Example 54.6. The 'c' plugin

build.gradle
```
apply plugin: 'c'
```

C源码列入在本地二进制文件里是通过CSourceSet来支持的。它定义了一组c源码文件和有选择的定义一组输出头文件（对一个库而言）。对于任何指定组件CSourceSet默认包含.c源码文件在src/${name}/c中，头文件在src/${name}/headers里。

C sources to be included in a native binary are provided via a CSourceSet, which defines a set of C source files and optionally a set of exported header files (for a library). By default, for any named component the CSourceSet contains .c source files in src/${name}/c, and header files in src/${name}/headers.

app插件为每个CppSourceSet定义默认的位置，然而为了不同的项目布局也可以延伸或覆盖这些默认项。

While the c plugin defines these default locations for each CSourceSet, it is possible to extend or override these defaults to allow for a different project layout.

例54.7 C源码集

Example 54.7. C source set
build.gradle
```
sources {
    c {
        source {
            srcDir "src/source"
            include "**/*.c"
        }
        exportedHeaders {
            srcDir "src/include"
        }
    }
}
```

对于命名为'main'的库，在src/main/headers中的头文件被认为是共有的或者是可输出的头。那种不能被输出的头文件应该被放在目录src/main/c里。（但是需要注意的事这样的头文件相对于包含他们的文件应该总是被引用的方式）

For a library named 'main', header files in src/main/headers are considered the “public” or “exported” headers. Header files that should not be exported should be placed inside the src/main/c directory (though be aware that such header files should always be referenced in a manner relative to the file including them).

### **54.10.3. Assembler sources**

汇编语言支持依赖于'assembler'插件

Assembly language support is provided by means of the 'assembler' plugin.

例54.8  'assembler'插件

Example 54.8. The 'assembler' plugin
build.gradle

```
apply plugin: 'assembler'
```
汇编源码列入在本地二进制文件里是通过AssemblerSourceSet来支持的。它定义了一组汇编源码文件和有选择的定义一组输出头文件（对一个库而言）。对于任何指定组件AssemblerSourceSet默认包含.s源码文件在src/${name}/asm里。

Assembler sources to be included in a native binary are provided via a AssemblerSourceSet, which defines a set of Assembler source files. By default, for any named component the AssemblerSourceSet contains .s source files under src/${name}/asm.

### **54.10.4. Objective-C sources**

Objective-C语言依赖于'objective-c'插件支持。

Objective-C language support is provided by means of the 'objective-c' plugin.

例54.9 'objective-c'插件

Example 54.9. The 'objective-c' plugin

build.gradle
```
apply plugin: 'objective-c'
```

Objective-C源码列入在本地二进制文件里是通过ObjectiveCSourceSet来支持的。它定义了一组Objective-C源码文件和有选择的定义一组输出头文件（对一个库而言）。对于任何指定组件   ObjectiveCSourceSet默认包含.m源码文件在src/${name}/objectiveC里。

Objective-C sources to be included in a native binary are provided via a ObjectiveCSourceSet, which defines a set of Objective-C source files. By default, for any named component the ObjectiveCSourceSet contains .m source files under src/${name}/objectiveC.

### **54.10.5. Objective-C++ sources***

Objective-C++语言支持依赖于'objective-cpp'插件。

Objective-C++ language support is provided by means of the 'objective-cpp' plugin.

例54.10 'objective-cpp'插件

Example 54.10. The 'objective-cpp' plugin

build.gradle
```
apply plugin: 'objective-cpp'
```

Objective-C++源码列入在本地二进制文件里是通过AssemblerSourceSet来支持的。它定义了一组汇编源码文件和有选择的定义一组输出头文件（对一个库而言）。对于任何指定组件ObjectiveCppSourceSet默认包含.mm源码文件在src/${name}/ObjectiveCppSourceSet里。

Objective-C++ sources to be included in a native binary are provided via a ObjectiveCppSourceSet, which defines a set of Objective-C++ source files. By default, for any named component the ObjectiveCppSourceSet contains .mm source files under src/${name}/ObjectiveCppSourceSet.

## **54.11. Configuring the compiler, assembler and linker(配置编译器、汇编和连接器)**

每个生成的为禁止文件都和一套编译器和链接器设置密切相关,其中包括命令行参数以及宏定义。这些设置可以应用于所有的二进制文件,单个二进制或选择性基于某些标准的一组二进制文件。

Each binary to be produced is associated with a set of compiler and linker settings, which include command-line arguments as well as macro definitions. These settings can be applied to all binaries, an individual binary, or selectively to a group of binaries based on some criteria.

例54.11  应用于所有二进制文件的设置

Example 54.11. Settings that apply to all binaries

build.gradle
```
binaries.all {
    // Define a preprocessor macro for every binary
    cppCompiler.define "NDEBUG"

    // Define toolchain-specific compiler and linker options
    if (toolChain in Gcc) {
        cppCompiler.args "-O2", "-fno-access-control"
        linker.args "-Xlinker", "-S"
    }
    if (toolChain in VisualCpp) {
        cppCompiler.args "/Zi"
        linker.args "/DEBUG"
    }
}
```

每个二进制与特定NativeToolChain相关联,使得设置基于这个值更具有针对性。

Each binary is associated with a particular NativeToolChain, allowing settings to be targeted based on this value.

It is easy to apply settings to all binaries of a particular type:

例54.12  应用于所有共享库的设置

Example 54.12. Settings that apply to all shared libraries

build.gradle
```
// For any shared library binaries built with Visual C++,
// define the DLL_EXPORT macro
binaries.withType(SharedLibraryBinarySpec) {
    if (toolChain in VisualCpp) {
        cCompiler.args "/Zi"
        cCompiler.define "DLL_EXPORT"
    }
}
```

另外，也可以指定设置应用于某个特定可执行文件或库组件所生成的全部二进制文件。

Furthermore, it is possible to specify settings that apply to all binaries produced for a particular executable or library component:

例54.13 设置应用于main生成的所有二进制文件

Example 54.13. Settings that apply to all binaries produced for the 'main' executable component

build.gradle
```
model {
    components {
        main(NativeExecutableSpec) {
            targetPlatform "x86"
            binaries.all {
                if (toolChain in VisualCpp) {
                    sources {
                        platformAsm(AssemblerSourceSet) {
                            source.srcDir "src/main/asm_i386_masm"
                        }
                    }
                    assembler.args "/Zi"
                } else {
                    sources {
                        platformAsm(AssemblerSourceSet) {
                            source.srcDir "src/main/asm_i386_gcc"
                        }
                    }
                    assembler.args "-g"
                }
            }
        }
    }
}
```

上面的示例将提供的配置适用于所有可执行的二进制文件构建。

The example above will apply the supplied configuration to all executable binaries built.

同样，设置可以为组件目标二进制文件指定一个特殊的类型：如主库组件的所有共享二进制文件。

Similarly, settings can be specified to target binaries for a component that are of a particular type: eg all shared libraries for the main library component.

例54.14  设置仅应用于main库组件生成的共享二进制文件

Example 54.14. Settings that apply only to shared libraries produced for the 'main' library component

build.gradle
```
model {
    components {
        main(NativeLibrarySpec) {
            binaries.withType(SharedLibraryBinarySpec) {
                // Define a preprocessor macro that only applies to shared libraries
                cppCompiler.define "DLL_EXPORT"
            }
        }
    }
}
```

54.12. Windows Resources

在使用VisualCpp工具链时,Gradle能够编译窗口资源(rc)文件和链接成一个本地二进制。此功能由'windows-resources插件支持。

When using the VisualCpp tool chain, Gradle is able to compile Window Resource (rc) files and link them into a native binary. This functionality is provided by the'windows-resources' plugin.

例54.15 windows-resources 插件

Example 54.15. The 'windows-resources' plugin

build.gradle
```
apply plugin: 'windows-resources'
```

Windows资源列入在本地二进制文件里是通过WindowsResourceSet来支持的。它定义了一组Windows资源源文件。对于任何指定组件WindowsResourceSet默认包含.rc源码文件在src/${name}/rc里。

Windows resources to be included in a native binary are provided via a WindowsResourceSet, which defines a set of Windows Resource source files. By default, for any named component the WindowsResourceSet contains .rc source files under src/${name}/rc.

和其他源码类型一样，你可以配置windows资源包含在二进制文件中的位置。

As with other source types, you can configure the location of the windows resources that should be included in the binary.

Example 54.16. Configuring the location of Windows resource sources

build-resource-only-dll.gradle
```
sources {
    rc {
        source {
            srcDirs "src/hello/rc"
        }
        exportedHeaders {
            srcDirs "src/hello/headers"
        }
    }
}
```

你可以通过提供windows资源且没有其他语言源码来构建一个仅资源的库，并配置适当的链接。

You are able to construct a resource-only library by providing Windows Resource sources with no other language sources, and configure the linker as appropriate:


例54.17 构建一个仅资源的dll

Example 54.17. Building a resource-only dll

build-resource-only-dll.gradle
```
model {
    components {
        helloRes(NativeLibrarySpec) {
            binaries.all {
                rcCompiler.args "/v"
                linker.args "/noentry", "/machine:x86"
            }
            sources {
                rc {
                    source {
                        srcDirs "src/hello/rc"
                    }
                    exportedHeaders {
                        srcDirs "src/hello/headers"
                    }
                }
            }
        }
    }
}
```

以上的例子也演示了传递额外命令行参数给资源编译器的机制。rcCompiler 延伸是 PreprocessingTool的一种类型。

The example above also demonstrates the mechanism of passing extra command-line arguments to the resource compiler. The rcCompiler extension is of type PreprocessingTool.

## **54.13. Library Dependencies**

本机组件的依赖关系是二进制库文件输出头文件。头文件在编译过程中被使用，和已编译的二进制依赖在链接和执行中被使用。

Dependencies for native components are binary libraries that export header files. The header files are used during compilation, with the compiled binary dependency being used during linking and execution.

### **54.13.1. Dependencies within the same project**

一组源码可能依赖来自同一个项目的另一个二进制组件提供的头文件。一个常见的例子是一个本地可执行组件,使用一个单独的本地库组件提供的功能。这样一个库的依赖可以添加到一个与可执行组件相关联的源码集:

A set of sources may depend on header files provided by another binary component within the same project. A common example is a native executable component that uses functions provided by a separate native library component.

Such a library dependency can be added to a source set associated with the executable component:

Example 54.18. Providing a library dependency to the source set

build.gradle
```
sources {
    cpp {
        lib library: "hello"
    }
}
```
或者,一个库的依赖可以直接提供给可执行文件的NativeExecutableBinary。

Alternatively, a library dependency can be provided directly to the NativeExecutableBinary for the executable.

Example 54.19. Providing a library dependency to the binary
build.gradle
```
model {
    components {
        hello(NativeLibrarySpec) {
            sources {
                c {
                    source {
                        srcDir "src/source"
                        include "**/*.c"
                    }
                    exportedHeaders {
                        srcDir "src/include"
                    }
                }
            }
        }
        main(NativeExecutableSpec) {
            sources {
                cpp {
                    source {
                        srcDir "src/source"
                        include "**/*.cpp"
                    }
                }
            }
            binaries.all {
                // Each executable binary produced uses the 'hello' static library binary
                lib library: 'hello', linkage: 'static'
            }
        }
    }
}
```

### **54.13.2. Project Dependencies**

对于一个在不同Gradle项目生成的组件，符号是相同的。

For a component produced in a different Gradle project, the notation is similar.

Example 54.20. Declaring project dependencies

build.gradle

```
project(":lib") {
    apply plugin: "cpp"
    model {
        components {
            main(NativeLibrarySpec)
        }
    }
    // For any shared library binaries built with Visual C++,
    // define the DLL_EXPORT macro
    binaries.withType(SharedLibraryBinarySpec) {
        if (toolChain in VisualCpp) {
            cppCompiler.define "DLL_EXPORT"
        }
    }
}

project(":exe") {
    apply plugin: "cpp"

    model {
        components {
            main(NativeExecutableSpec) {
                sources {
                    cpp {
                        lib project: ':lib', library: 'main'
                    }
                }
            }
        }
    }
}
```

## **54.14. Native Binary Variants**

对于每个定义的可执行文件或库文件,Gradle可以建立若干不同的本地二进制变异。不同的变体的例子包括调试的和发布的二进制文件,32位与64位的二进制文件,不同定义处理器标志生成的二进制文件。

For each executable or library defined, Gradle is able to build a number of different native binary variants. Examples of different variants include debug vs release binaries, 32-bit vs 64-bit binaries, and binaries produced with different custom preprocessor flags.

Gradle产生的二进制文件可以在构建类型,平台,和特性下差异化。对于这些变体维度,可以指定给每个组件一个，一些或者所有这些可用值以及目标。例如,插件可以定义一系列的支持平台,但您可以选择只针对windows x86作为特定组件。

Binaries produced by Gradle can be differentiated on build type, platform, and flavor. For each of these 'variant dimensions', it is possible to specify a set of available values as well as target each component at one, some or all of these. For example, a plugin may define a range of support platforms, but you may choose to only target Windows-x86 for a particular component.

### **54.14.1. Build types**

构建一个二进制的类型决定了各种非功能性方面因素,如是否包含调试信息,或者二进制编译优化水平。典型的构建类型是“调试”和“发布”,但一个项目可以自由定义任何组构建类型。

A build type determines various non-functional aspects of a binary, such as whether debug information is included, or what optimisation level the binary is compiled with. Typical build types are 'debug' and 'release', but a project is free to define any set of build types.

Example 54.21. Defining build types

build.gradle
```
model {
    buildTypes {
        debug
        release
    }
}
```

如果一项目没有定义构建类型，那么会有一个单独的，默认的类型名为debug。

If no build types are defined in a project, then a single, default build type called 'debug' is added.

对于构建类型,Gradle项目通常会为每个工具链定义一组编译器和链接器的标志。

For a build type, a Gradle project will typically define a set of compiler/linker flags per tool chain.

Example 54.22. Configuring debug binaries

build.gradle
```
binaries.all {
    if (toolChain in Gcc && buildType == buildTypes.debug) {
        cppCompiler.args "-g"
    }
    if (toolChain in VisualCpp && buildType == buildTypes.debug) {
        cppCompiler.args '/Zi'
        cppCompiler.define 'DEBUG'
        linker.args '/DEBUG'
    }
}
```

在这个阶段,它是完全取决于构建脚本配置为每个构建类型配置相关的编译器和链接器的标志。Gradle的未来版本将自动为任何“调试”构建类型包含适当的调试标记,同时注意各种级别的优化。

At this stage, it is completely up to the build script to configure the relevant compiler/linker flags for each build type. Future versions of Gradle will automatically include the appropriate debug flags for any 'debug' build type, and may be aware of various levels of optimisation as well.

### **54.14.2. Platform**

一个可执行或库可以被构建并运行在不同的操作系统和cpu架构,并在每个平台生成不同的产物。Gradle定义每个OS /架构结合作为NativePlatform,并且一个项目可能定义任意数量的平台。如果一个项目没有定义任何平台,那么“当前”平台将会添加作为默认平台。

An executable or library can be built to run on different operating systems and cpu architectures, with a variant being produced for each platform. Gradle defines each OS/architecture combination as a NativePlatform, and a project may define any number of platforms. If no platforms are defined in a project, then a single, default platform 'current' is added.

目前,平台由一个定义的操作系统和架构构成。随着我们继续开发Gradle本地二进制支持,平台的概念将扩展到包括c运行时版本，Windows SDK,ABI,等等。复杂的构建可能使用Gradle的可扩展性将额外的属性应用到每个平台,然后可以查询二进制文件指定的特定内容,包括预处理器宏或编译器参数。

Presently, a Platform consists of a defined operating system and architecture. As we continue to develop the native binary support in Gradle, the concept of Platform will be extended to include things like C-runtime version, Windows SDK, ABI, etc. Sophisticated builds may use the extensibility of Gradle to apply additional attributes to each platform, which can then be queried to specify particular includes, preprocessor macros or compiler arguments for a native binary.

Example 54.23. Defining platforms

build.gradle
```
model {
    platforms {
        x86 {
            architecture "x86"
        }
        x64 {
            architecture "x86_64"
        }
        itanium {
            architecture "ia-64"
        }
    }
}
```

对于给定的变体,它将试图找到一个可以构建目标平台的NativeToolChain。可用的工具链在定义的顺序可被搜索到。更多细节请见下面的工具链节。

For a given variant, Gradle will attempt to find a NativeToolChain that is able to build for the target platform. Available tool chains are searched in the order defined. See the tool chains section below for more details.

### **54.14.3. Flavor**

每个组件可以有一组特性,每一个特性由一个单独的二进制变体生成。而构建类型和目标平台变体维度有个定义的意图,每个项目可以定义任意数量的特性且可以以任何形式把意图应用于它们。

Each component can have a set of named flavors, and a separate binary variant can be produced for each flavor. While the build type and target platform variant dimensions have a defined meaning in Gradle, each project is free to define any number of flavors and apply meaning to them in any way.

组件特性的一个例子可能在“演示”,“支付”和“企业”组件版本中有所区别,相同的一组源码会生成功能不同的二进制文件。

An example of component flavors might differentiate between 'demo', 'paid' and 'enterprise' editions of the component, where the same set of sources is used to produce binaries with different functions.

Example 54.24. Defining flavors

build.gradle
```
model {
    flavors {
        english
        french
    }
    components {
        hello(NativeLibrarySpec) {
            binaries.all {
                if (flavor == flavors.french) {
                    cppCompiler.define "FRENCH"
                }
            }
        }
    }
}
```

在上面的示例中,一个库被定义了一个“英语”和“法国”的特性。编译法国这个变量时,一个单独的宏定义导致生成不同的二进制。

In the example above, a library is defined with a 'english' and 'french' flavor. When compiling the 'french' variant, a separate macro is defined which leads to a different binary being produced.

如果一个组件没有被定义特性，那么默认将会有一个名字为‘default’的特性。

If no flavor is defined for a component, then a single default flavor named 'default' is used.

### **54.14.4. Selecting the build types, platforms and flavors for a component**

对于一个默认的组件,Gradle将试图为每一个项目定义的构建类型、平台和特性以及他们之间的结合体创建一个本地二进制变量。可以在每个组件的基础上通过指定的一组targetBuildTypes,targetPlatform和/或targetFlavors覆盖这点。

For a default component, Gradle will attempt to create a native binary variant for each and every combination of buildType, platform and flavor defined for the project. It is possible to override this on a per-component basis, by specifying the set of targetBuildTypes, targetPlatform and/or targetFlavors.

Example 54.25. Targeting a component at particular platforms

build.gradle
```
model {
    components {
        hello(NativeLibrarySpec) {
            targetPlatform "x86"
            targetPlatform "x64"
        }
        main(NativeExecutableSpec) {
            targetPlatform "x86"
            targetPlatform "x64"
            sources {
                cpp.lib library: 'hello', linkage: 'static'
            }
        }
    }
}
```

在这里你可以看到TargetedNativeComponent.targetPlatform()方法用于指定一个平台,名字为named的NativeExecutableSpec应该为此平台被构建。

Here you can see that the TargetedNativeComponent.targetPlatform() method is used to specify a platform that the NativeExecutableSpec named main should be built for.

类似的机制存在选择TargetedNativeComponent.targetBuildTypes()和TargetedNativeComponent.targetFlavors()。

A similar mechanism exists for selecting TargetedNativeComponent.targetBuildTypes() and TargetedNativeComponent.targetFlavors().

### **54.14.5. Building all possible variants**

当一组构建类型、目标平台和特性组件定义后，每一个可能的组合的NativeBinarySpec模型元素就创建好了。然而,在很多情况下是不可能建立一个特定的变量,也许是因为没有可用工具链构建为一个特定的平台。

When a set of build types, target platforms, and flavors is defined for a component, a NativeBinarySpec model element is created for every possible combination of these. However, in many cases it is not possible to build a particular variant, perhaps because no tool chain is available to build for a particular platform.

如果一个二进制变量由于某种原因不能被构建,那么和那变量相关联的NativeBinarySpec是不可以被构建的。可以使用这个属性来创建一个任务用来在特定机器上生成一个所有可能的变体。

If a binary variant cannot be built for any reason, then the NativeBinarySpec associated with that variant will not be buildable. It is possible to use this property to create a task to generate all possible variants on a particular machine.

Example 54.26. Building all possible variants

build.gradle
```
task buildAllExecutables {
    dependsOn binaries.withType(NativeExecutableBinary).matching {
        it.buildable
    }
}
```
## **54.15. Tool chains**

一个构建可能使用不同的工具链为不同的平台构建变量。为此,核心的二进制的插件将试图定位和提供支持的工具链。然而,项目的设置工具链也可以显式地定义,允许额外的交叉编译器进行配置以及允许指定安装目录。

A single build may utilize different tool chains to build variants for different platforms. To this end, the core 'native-binary' plugins will attempt to locate and make available supported tool chains. However, the set of tool chains for a project may also be explicitly defined, allowing additional cross-compilers to be configured as well as allowing the install directories to be specified.

### **54.15.1. Defining tool chains**

The supported tool chain types are:
•	Gcc
•	Clang
•	VisualCpp

Example 54.27. Defining tool chains

build.gradle
```
model {
    toolChains {
        visualCpp(VisualCpp) {
            // Specify the installDir if Visual Studio cannot be located
            // installDir "C:/Apps/Microsoft Visual Studio 10.0"
        }
        gcc(Gcc) {
            // Uncomment to use a GCC install that is not in the PATH
            // path "/usr/bin/gcc"
        }
        clang(Clang)
    }
}
```

每个工具链实现允许一定程度的配置(了解更多信息请见API文档)。

Each tool chain implementation allows for a certain degree of configuration (see the API documentation for more details).

### **54.15.2. Using tool chains**

没有必要或可能指定应该用于构建的工具链。对于给定的变量,Gradle将试图找到能够构建目标平台的NativeToolChain。可用的工具链可以在定义的顺序搜索到。

It is not necessary or possible to specify the tool chain that should be used to build. For a given variant, Gradle will attempt to locate a NativeToolChain that is able to build for the target platform. Available tool chains are searched in the order defined.

当没有定义一个架构平台或操作系统的情况下，默认工具链的目标是假设的。所以如果一个平台没有为操作系统定义一个值,Gradle会找到可以为指定的架构构建的可用的第一个工具链。

When a platform does not define an architecture or operating system, the default target of the tool chain is assumed. So if a platform does not define a value foroperatingSystem, Gradle will find the first available tool chain that can build for the specified architecture.

核心Gradle工具链能够在以下架构中指定目标。在每种情况下,工具链将当前的操作系统作为目标。详细信息,请参阅下一节在其他操作系统交叉编译。

The core Gradle tool chains are able to target the following architectures out of the box. In each case, the tool chain will target the current operating system. See the next section for information on cross-compiling for other operating systems.

|Tool Chain	|Architectures|
|--
|GCC	|x86, x86_64|
|Clang|	x86, x86_64|
|Visual C++	|x86, x86_64, ia-64|

所以对于GCC在linux上运行,支持的目标平台为linux / x86和“linux / x86_64”。对于通过Cygwin运行在Windows上的GCC,支持的平台为Windows / x86和Windows / x86_64。(Cygwin POSIX运行时还没有塑造作为平台的一部分,但可能将来会)。

So for GCC running on linux, the supported target platforms are 'linux/x86' and 'linux/x86_64'. For GCC running on Windows via Cygwin, platforms 'windows/x86' and 'windows/x86_64' are supported. (The Cygwin POSIX runtime is not yet modelled as part of the platform, but will be in the future.)

如果没有为一个项目定义目标平台,那么所有构建的二进制文件将把名字为“当前”的平台作为目标平台。这个默认平台没有指定架构和操作系统值,因此使用第一个可用的工具链的缺省值。

If no target platforms are defined for a project, then all binaries are built to target a default platform named 'current'. This default platform does not specify anyarchitecture or operatingSystem value, hence using the default values of the first available tool chain.

Gradle提供一个钩子,允许构建作者控制具体参数设置传递给可执行的工具链。这使得构建作者在Gradle解决任何限制,或Gradle做出的假设。钩的参数应被视为一个“杀手锏”机制,以优先考虑真正塑造底层域。

Gradle provides a hook that allows the build author to control the exact set of arguments passed to a tool chain executable. This enables the build author to work around any limitations in Gradle, or assumptions that Gradle makes. The arguments hook should be seen as a 'last-resort' mechanism, with preference given to truly modelling the underlying domain.

Example 54.28. Reconfigure tool arguments

build.gradle
```
model {
    toolChains {
        visualCpp(VisualCpp) {
            eachPlatform {
                cppCompiler.withArguments { args ->
                    args << "-DFRENCH"
                }
            }
        }
        clang(Clang) {
            eachPlatform {
                cCompiler.withArguments { args ->
                    Collections.replaceAll(args, "CUSTOM", "-DFRENCH")
                }
                linker.withArguments { args ->
                    args.remove "CUSTOM"
                }
                staticLibArchiver.withArguments { args ->
                    args.remove "CUSTOM"
                }
            }
        }
    }
}
```

### **54.15.3. Cross-compiling with GCC**

GCC和Clang工具链的交叉编译通过添加额外的目标平台的支持是可行的。这样做是通过为工具链指定一个目标平台。为每个目标平台可以指定一个自定义配置。每个目标平台都应有指定的配置。

Cross-compiling is possible with the Gcc and Clang tool chains, by adding support for additional target platforms. This is done by specifying a target platform for a toolchain. For each target platform a custom configuration can be specified.

Example 54.29. Defining target platforms

build.gradle
```
model {
    toolChains {
        gcc(Gcc) {
            target("arm"){
                cppCompiler.withArguments { args ->
                    args << "-m32"
                }
                linker.withArguments { args ->
                    args << "-m32"
                }
            }
            target("sparc")
        }
    }
    platforms {
        arm {
            architecture "arm"
        }
        sparc {
            architecture "sparc"
        }
    }
    components {
        main(NativeExecutableSpec) {
            targetPlatform "arm"
            targetPlatform "sparc"
        }
    }
}
```

## **54.16. Visual Studio IDE integration**

Gradle可以为你构建中定义的本地组件生成Visual Studio项目和解决方案文件。这种功能被添加到visual studio插件。对于多项目构建,所有拥有本地组件的项目都该应用此插件。

Gradle has the ability to generate Visual Studio project and solution files for the native components defined in your build. This ability is added by the visual-studio plugin. For a multi-project build, all projects with native components should have this plugin applied.

visual-studio插件应用的时候，将会为每个定义的组件创建任务名${component.name}VisualStudio。这个文物会为明明的组件生成一个 Visual Studio解决方案文件。这个解决方案包含了那个组件的一个 Visual Studio项目，以及链接到每个被依赖的二进制文件的项目文件。

When the visual-studio plugin is applied, a task name ${component.name}VisualStudio is created for each defined component. This task will generate a Visual Studio Solution file for the named component. This solution will include a Visual Studio Project for that component, as well as linking to project files for each depended-on binary.

visual studio生成的文件的内容可以通过由visualStudio的扩展提供的API hook进行修改,。看看visual studio的样本,或看看VisualStudioExtension.getProjects()和VisualStudioExtension.getSolutions()API文档的更多细节。

The content of the generated visual studio files can be modified via API hooks, provided by the visualStudio extension. Take a look at the 'visual-studio' sample, or seeVisualStudioExtension.getProjects() and VisualStudioExtension.getSolutions() in the API documentation for more details.

## **54.17. CUnit support**

Gradle cunit插件支持本地二进制项目编译和执行cunit测试。对于你的项目中每个NativeExecutableSpec和NativeLibrarySpecdefined,Gradle将创建一个匹配CUnitTestSuiteSpec组件,名为$ { component.name }Test。

The Gradle cunit plugin provides support for compiling and executing CUnit tests in your native-binary project. For each NativeExecutableSpec and NativeLibrarySpecdefined in your project, Gradle will create a matching CUnitTestSuiteSpec component, named ${component.name}Test.

### **54.17.1. CUnit sources**

Gradle将为项目中的每个CUnitTestSuiteSpec组件创建一个名为“cunit”的CSourceSet项。这个源码集应该包含组件源码的cunit测试文件。源文件可以位于常规位置(src/${component.name}Test/cunit)或者可以像任何其源码集一样配置。

Gradle will create a CSourceSet named 'cunit' for each CUnitTestSuiteSpec component in the project. This source set should contain the cunit test files for the component sources. Source files can be located in the conventional location (src/${component.name}Test/cunit) or can be configured like any other source set.

Gradle通过利用一些生成的CUnit启动资源初始化CUnit test注册表并执行测试。Gradle将期望和调用一个函数签名void gradle_cunit_register(),您可以使用它配置实际的CUnit套件和执行测试。

Gradle initialises the CUnit test registry and executes the tests, utilising some generated CUnit launcher sources. Gradle will expect and call a function with the signature void gradle_cunit_register() that you can use to configure the actual CUnit suites and tests to execute.

Example 54.30. Registering CUnit tests
```
suite_operators.c
#include <CUnit/Basic.h>
#include "gradle_cunit_register.h"
#include "test_operators.h"

int suite_init(void) {
    return 0;
}

int suite_clean(void) {
    return 0;
}

void gradle_cunit_register() {
    CU_pSuite pSuiteMath = CU_add_suite("operator tests", suite_init, suite_clean);
    CU_add_test(pSuiteMath, "test_plus", test_plus);
    CU_add_test(pSuiteMath, "test_minus", test_minus);
}
```

基于这种机制，你的CUnit源码可能不会包含一个主函数，因为它与Gradle提供的方法相冲突。

Due to this mechanism, your CUnit sources may not contain a main method since this will clash with the method provided by Gradle.

### **54.17.2. Building CUnit executables**

CUnitTestSuiteSpec组件都有一个关联的NativeExecutableSpec或NativeLibrarySpec组件。为主要组件配置的每个NativeBinarySpec，都有一个匹配的CUnitTestSuiteBinarySpec将配置在测试套件组件中。这些测试套件二进制文件可以以类似的方式配置其他二进制实例:

A CUnitTestSuiteSpec component has an associated NativeExecutableSpec or NativeLibrarySpec component. For each NativeBinarySpec configured for the main component, a matching CUnitTestSuiteBinarySpec will be configured on the test suite component. These test suite binaries can be configured in a similar way to any other binary instance:

Example 54.31. Registering CUnit tests

build.gradle
```
binaries.withType(CUnitTestSuiteBinarySpec) {
    lib library: "cunit", linkage: "static"

    if (flavor == flavors.failing) {
        cCompiler.define "PLUS_BROKEN"
    }
}
```
由项目提供的CUnit源码和启动器需要核心CUnit头文件和库文件。目前,项目必须为每个CUnitTestSuiteBinarySpec提供这个库依赖。

Both the CUnit sources provided by your project and the generated launcher require the core CUnit headers and libraries. Presently, this library dependency must be provided by your project for each CUnitTestSuiteBinarySpec.

### **54.17.3. Running CUnit tests**

对于每个CUnitTestSuiteBinarySpec,Gradle将创建一个任务来执行这个二进制文件,它将运行所有的注册CUnit测试。测试结果将在${build.dir}/test-results 目录中。

For each CUnitTestSuiteBinarySpec, Gradle will create a task to execute this binary, which will run all of the registered CUnit tests. Test results will be found in the${build.dir}/test-results directory.

Example 54.32. Running CUnit tests

build.gradle
```
apply plugin: "c"
apply plugin: "cunit"

model {
    flavors {
        passing
        failing
    }
    platforms {
        x86 {
            architecture "x86"
        }
    }
    repositories {
        libs(PrebuiltLibraries) {
            cunit {
                headers.srcDir "libs/cunit/2.1-2/include"
                binaries.withType(StaticLibraryBinary) {
                    staticLibraryFile =
                        file("libs/cunit/2.1-2/lib/" +
                             findCUnitLibForPlatform(targetPlatform))
                }
            }
        }
    }
    components {
        operators(NativeLibrarySpec) {
            targetPlatform "x86"
        }
    }
}
binaries.withType(CUnitTestSuiteBinarySpec) {
    lib library: "cunit", linkage: "static"

    if (flavor == flavors.failing) {
        cCompiler.define "PLUS_BROKEN"
    }
}
```

注意:这个例子的代码可以samples/native-binaries/cunit找到，在Gradle“-all”的分布中。 

Note: The code for this example can be found at samples/native-binaries/cunit in the ‘-all’ distribution of Gradle.

Output of gradle -q runFailingOperatorsTestCUnitExe
```
> gradle -q runFailingOperatorsTestCUnitExe
```

There were test failures:
  1. /home/user/gradle/samples/native-binaries/cunit/src/operatorsTest/c/test_plus.c:6  - plus(0, -2) == -2
  2. /home/user/gradle/samples/native-binaries/cunit/src/operatorsTest/c/test_plus.c:7  - plus(2, 2) == 4

当前对CUnit的支持非常基础，未来集成计划包括：
•  允许测试声明使用Javadoc-style注释
•  改进HTML报告,类似于JUnit
•  测试执行的实时反馈
•  额外测试框架支持

The current support for CUnit is quite rudimentary. Plans for future integration include:
•	Allow tests to be declared with Javadoc-style annotations.
•	Improved HTML reporting, similar to that available for JUnit.
•	Real-time feedback for test execution.
•	Support for additional test frameworks.

## **54.18. GoogleTest support**

Gradle谷歌测试插件支持在本地二进制项目中编译和执行测试。为每个在你的项目中定义的NativeExecutableSpec andNativeLibrarySpec,它将创建一个匹配GoogleTestTestSuiteSpec组件,名为${component.name}Test。

The Gradle google-test plugin provides support for compiling and executing GoogleTest tests in your native-binary project. For each NativeExecutableSpec andNativeLibrarySpec defined in your project, Gradle will create a matching GoogleTestTestSuiteSpec component, named ${component.name}Test.

### **54.18.1. GoogleTest sources**

Gradle将会为项目中每个GoogleTestTestSuiteSpec组件创建一个名为的cpp的CppSourceSet。这个源码集应该包含该组件源码的GoogleTest测试文件。源文件可以位于常规位置 (src/${component.name}Test/cpp)或者可以像任何其他源码集一样配置。

Gradle will create a CppSourceSet named 'cpp' for each GoogleTestTestSuiteSpec component in the project. This source set should contain the GoogleTest test files for the component sources. Source files can be located in the conventional location (src/${component.name}Test/cpp) or can be configured like any other source set.

### **54.18.2. Building GoogleTest executables**

GoogleTestTestSuiteSpec组件都有一个关联的NativeExecutableSpec或NativeLibrarySpec组件。对于主要组件配置的每个NativeBinarySpec,都将在测试套件组件中配置一个匹配的GoogleTestTestSuiteBinarySpec。这些测试套件二进制文件可以以类似的方式配置其他二进制实例:

A GoogleTestTestSuiteSpec component has an associated NativeExecutableSpec or NativeLibrarySpec component. For each NativeBinarySpec configured for the main component, a matching GoogleTestTestSuiteBinarySpec will be configured on the test suite component. These test suite binaries can be configured in a similar way to any other binary instance:

Example 54.33. Registering GoogleTest tests

build.gradle
```
binaries.withType(GoogleTestTestSuiteBinarySpec) {
    lib library: "googleTest", linkage: "static"

    if (flavor == flavors.failing) {
        cppCompiler.define "PLUS_BROKEN"
    }
}
```

由项目提供的GoogleTest源码需要核心GoogleTest头文件和库文件。目前，项目必须为每个GoogleTestTestSuiteBinarySpec提供库依赖。

The GoogleTest sources provided by your project require the core GoogleTest headers and libraries. Presently, this library dependency must be provided by your project for each GoogleTestTestSuiteBinarySpec.

### **54.18.3. Running GoogleTest tests**

对于每个GoogleTestTestSuiteBinarySpec，Gradle将会创建一个任务去执行这个二进制文件，它将会运行所有注册的GoogleTest测试，测试结果将会在目录 ${build.dir}/test-results下找到。

For each GoogleTestTestSuiteBinarySpec, Gradle will create a task to execute this binary, which will run all of the registered GoogleTest tests. Test results will be found in the ${build.dir}/test-results directory.

当前对GoogleTest的支持还是很基本的。未来集成计划包括：
•改进HTML报告,类似于JUnit
•测试执行的实时反馈
•额外测试框架支持

The current support for GoogleTest is quite rudimentary. Plans for future integration include:
•	Improved HTML reporting, similar to that available for JUnit.
•	Real-time feedback for test execution.
•	Support for additional test frameworks.

百度搜索[无线学院](http://wirelesscollege.cn)

