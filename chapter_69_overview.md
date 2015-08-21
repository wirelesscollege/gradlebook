# **第69章 基于模型配置的规则**

Chapter 69. Rule based model configuration

本章描述的是Gradle3.0版本下一代Gradle构建体系。我们会在Gradle2.X版本的技术上进行开发，使其支持本地Binaries构建。

This chapter describes and documents what is essentially the foundation for the Gradle 3.0 and the next generation of Gradle builds. It is being incrementally developed during the Gradle 2.x stream and is in use for Gradle's support for building native binaries.

之前讨论的所有技术，包括DSL,API都会被进一步开发（例如：not considered stable and subject to change - see Appendix C, The Feature Lifecycle）。早期我们提及的一些新功能，在孵化期间会让大家更早得了解到，测试到这些新功能，以至于我们得到一个更好用的Gradle。

All of the mechanisms, DSL, API, and techniques discussed here are incubating (i.e. not considered stable and subject to change - see Appendix C, The Feature Lifecycle). Exposing new features early, during incubation, allows early testing and the incorporation of real world feedback ultimately resulting in a better Gradle.

下面的构建脚本是基于构建规则的一个例子

The following build script is an example of a rule based build.

Example 69.1. an example of a simple rule based build

build.gradle

```
@Managed
interface Person {
  void setFirstName(String n); String getFirstName()
  void setLastName(String n); String getLastName()
}

class PersonRules extends RuleSource {
  @Model void person(Person p) {}

  @Mutate void setFirstName(Person p) {
    p.firstName = "John"
  }

 @Mutate void createHelloTask(CollectionBuilder<Task> tasks, Person p) {
    tasks.create("hello") {
      doLast {
        println "Hello $p.firstName $p.lastName!"
      }
    }
  }
}

apply plugin: PersonRules

model {
  person {
    lastName = "Smith"
  }
}
```
注意：例子中的代码在Gradle正式发行版的samples/modelRules/basicRuleSourcePlugin 目录下能够找到。

Note: The code for this example can be found at samples/modelRules/basicRuleSourcePlugin in the ‘-all’ distribution of Gradle.

Gradle hello的输出结果

Output of gradle hello

```
> gradle hello
:hello
Hello John Smith!

BUILD SUCCESSFUL

Total time: 1 secs
```

本章致力于解释Gradle以后会往哪个方向发展，为什么要这么发展。

The rest of this chapter is dedicated to explaining what is going on in this build script, and why Gradle is moving in this direction.

## **69.1背景**

69.1. Background

Gradle以为领域模式为核心主旨。聚焦领域模式比执行模式会有更多得优势（例如前一代的Ant构建工具）。强大得领域模型引导结构上的意图。（从怎么做到做什么）.它会让人们更加理解Gradle的构建，因为这个构建将会更有意义。

Gradle embraces domain modelling as core tenet. Focusing on the domain model as opposed to the execution model (like prior generation build tools such as Apache Ant) has many advantages. A strong domain model communicates the intent (i.e. the what) over the mechanics (i.e. the how). This allows humans to understand builds at a level that is meaningful to them.

与帮助其他人一样，强大得领域模型也会帮助尽职尽责的机器们。围绕领域模型插件能够更好的协作。（例如：插件可以按照约定表达一些关于java应用的事情）。模型将会代替Gradle如何做更明智的选择。

As well as helping humans, a strong domain model also helps the dutiful machines. Plugins can more effectively collaborate around a strong domain model (e.g. plugins can say something about Java applications, such as providing conventions). Very importantly, by having a model of the what instead of the how Gradle can make intelligent choices on just how to do the how.

“基于模型配置的规则”趋势可以被概括为可以让Gradle能力更加丰富，拥有更强大更有效的方式方法。也会让今天的Gradle模型更健壮更简单。

The move towards “Rule based model configuration” can be summarised as improving Gradle's ability to model richer domains in a more effective way. It also makes expressing the kinds of models present in today's Gradle more robust and simpler.

## **69.2 为什么要改变**

69.2. Motivations for change

Domain modelling 在Gradle里并不新鲜。java插件的SourceSet concept就是一个Domain modelling的例子，就好像本地插件套装内的NativeBinary模型一样。

Domain modelling in Gradle is not new. The Java plugin's SourceSet concept is an example of domain modelling, as is the modelling of NativeBinary in the Native plugin suite.

Gradle与其他构件工具的一个明显区别就是Gradle的embrace modelling比其他模型更加开放也更容易协作。从根本上讲Gradle是一个模型化软件构建工具，通过task来实行model例如编译任务。不同领域插件提供了其他插件能够与之交流的模型们。例如（java，c++，android插件）。

One distinguishing characteristic of Gradle compared to other build tools that also embrace modelling is that Gradle's model is open and collaborative. Gradle is fundamentally a tool for modelling software construction and then realizing the model, via tasks such as compilation etc.. Different domain plugins (e.g. Java, C++, Android) provide models that other plugins can collaborate with and build upon.

Gradle采用先进的技术来实现它的模型。（例如：我们知道构建中的一些事情），下一代Gradle将会采用相同的技术，让它自己来构建这个模型。通过定义构建Task可以有效的了解依赖关系以及输入输出，Gradle将能够在cache，并发，应用其他的优化方式去工作。使用任务图谱是一个长期的idea，必要的解决了软件的复杂程度。任务图谱定义了gradle必须执行的规则，而Rule based model configuration与任务图谱采用了相同的思想。

While Gradle has long employed sophisticated techniques when it comes to realizing the model (i.e. what we know as building things), the next generation of Gradle builds will employ some of the same techniques to building of the model itself. By defining build tasks as effectively a graph of dependent functions with explicit inputs and outputs, Gradle is able to order, cache, parallelize and apply other optimizations to the work. Using a “graph of tasks” for the production of software is a long established idea, and necessary given the complexity of software production. The task graph effectively defines the rules of execution the Gradle must follow. The term “Rule based model configuration” refers to applying the same concepts to building the model that builds the task graph.

另外一个关键动机就是性能，当前Gradle对限制了并行式构建，新的模型将会实现并行构建，这样Gradle在不管构建大项目还是小项目上都会更加有效。

Another key motivation is performance and scale. Aspects of the current approach that Gradle takes to modelling the build prevent pervasive parallelism and limit scalability. The new model is being designed with the requirements of modern software delivery in mind, where immediate responsiveness is critical for projects large and small.

## **69.3 概念**

69.3. Concepts

本章需要讲解rule based model configuration的关键概念，下述子章节讲解具体实施时的理念。

This section outlines the key concepts of rule based model configuration. Subsequent sections in this chapter will show the concepts in action.


### **69.3.1. The “model space”**

model space这个术语的意思是地址化规则的正式模型。

The term “model space” is used to refer to the formal model, addressable by rules.

model space是模式已经存在的模型，项目对象就是项目根目录下地一些资源描述文件，例如project.repositories,project.task等等.构建脚本有效的添加配置这些对象。在大部分时候project space对于gradle是不透明的，它仅仅是Gradle能够识别的一种对象图谱罢了。

An analog with existing model is effectively the “project space”. The Project object is effectively the root of a graph of objects (e.g project.repositories, project.tasks etc.). A build script is effectively adding and configuring objects of this graph. For the most part, the “project space” is opaque to Gradle. It is an arbitrary graph of objects that Gradle only partially understands.

每一个工程都有自己的模型，这与project space是不同得。model space 的关键特点是Gradle更加了解他，也能更好的使用他。

Each project also has its own model space, which is distinct from the project space. A key characteristic of the “model space” is that Gradle knows much more about it (which is knowledge that can be put to good use). 

模型中的对象是managed，Gradle将会定义多种相关规则，让我们更好的理解他们的关系。

The objects in the model space are “managed”, to a greater extent than objects in the project space. The origin, structure, state, collaborators and relationships of objects in the model space are first class constructs. This is effectively the characteristic that functionally distinguishes the model space from the project space: the objects of the model space are defined in ways that Gradle can understand them intimately, as opposed to an object that is the result of running relatively opaque code. A “rule” is effectively a building block of this definition.

model space将会代替project space，未来只会有space，然而这中间的过渡还是对我们有帮助的。

The model space will eventually replace the project space, in so far as it will be the only “space”. However, during the transition the distinction is helpful.

### **69.3.2. Model paths**

model path是通过model space来定义的，通常情况下task就代表这是个任务，但是如果任务得名字是hello，那么她得model path就会被定义为task.hello

A model path identifies a path through a model space, to an element. A common representation is a period-delimited set of names. The model path "tasks" is the path to the element that is the task container. Assuming a task who's name is hello, the path "tasks.hello" is the path to this task.

未完持续（官网就这么写的）
TBD - more needed here.

### **69.3.3. Rules**

model space也定义了一系列的规则，规则就相当于抽象场景里得功能，他们产生模型元素，也取决于模型元素。每个规则都有一个单独的主题，以及0或者更多得输入。当输入是有效不变的情况下，只有主题能够被规则所改变。

The model space is defined in terms of “rules”. A rule is just a function (in the abstract sense) that either produces a model element, or acts upon a model element. Every rule has a single subject and zero or more inputs. Only the subject can be changed by a rule, while the inputs are effectively immutable.

Gradle担保所有的输入都能够在规则执行之前被全部识别。在这个过程中，识别到这个元素并且有效的执行它。并且能够保证你所有的依赖关系。（原文说的好啰嗦，使用任务图谱，执行模型啥得，最后就是这一句话，保证所有依赖关系）

Gradle guarantees that all inputs are fully “realized“ before the rule executes. The process of “realizing” a model element is effectively executing all the rules for which it is the subject, transitioning it to its final state. There is a strong analogy here to Gradle's task graph and task execution model. Just as tasks depend on each other and Gradle ensures that dependencies are satisfied before executing a task, rules effectively depend on each other (i.e. a rule depends on all rules who's subject is one of the inputs) and Gradle ensures that all dependencies are satisfied before executing the rule.

模型元素通常被其他的模型元素们所定义，例如编译任务配置为source set配置所定义。在这个场景下，编译任务就是规则的主题，source set就是输入。如此机遇source set 输入内一个规则就能够配置一个任务的主题，从而我们不用关心它是如何被配置的，谁配置的，什么时候配置的。

Model elements are very often defined in terms of other model elements. For example, a compile task's configuration can be defined in terms of the configuration of the source set that it is compiling. In this scenario, the compile task would be the subject of a rule and the source set an input. Such a rule could configure the task subject based on the source set input without concern for how it was configured, who it was configured by or when the configuration was specified.

有不同的方法和形式来处理规则。围绕着本章接下来的具体例子来解释不同机制和规则。

There are several ways to declare rules, and in several forms. An explanation of the different forms and mechanisms along with concrete examples is forthcoming in this chapter.

### **69.3.4. Managed model elements**

当前任何java对象都是modelspace的一部分，然而在“managed”与“unmanaged”对象之间是有区别的。

Currently, any kind of Java object can be part of the model space. However, there is a difference between “managed” and “unmanaged” objects.

managed对象在初始化的时候就是透明的不变的。透明就是结构能够通过规则容易理解，包括model space中的属性。请查看Managed annotation获取创建managed model objects的更多得信息。

A “managed” object is transparent and enforces immutability once realized. Being transparent means that its structure is understood by the rule infrastructure and as such each of its properties are also individual elements in the model space. Please see the Managed annotation for more information on creating managed model objects.

unmanaged对象在model space中就是不透明的。也不执行不变性。随着时间的推移，更多得机制将会对定义managed模型元素有效，最终会以某种方式被managed。

An “unmanaged” object is opaque to the the model space and does not enforce immutability. Over time, more mechanisms will be available for defining managed model elements culminating in all model elements being managed in some way.

### **69.3.5. References, binding and scopes**

前面提到，一个规则有一个主题，一个0个或者多个输入。在Gradle执行前规则的主题和输入被作为引用reference和绑定bound。
每条规则都可以有效的宣布主题和input作为引用，如何实现取决于规则的形式，例如，规则可以由RuleSource作为引用参数。

As previously mentioned, a rule has a subject and zero or more inputs. The rule's subject and inputs are declared as “references” and are “bound” to model elements before execution by Gradle. Each rule must effectively forward declare the subject and inputs as references. Precisely how this is done depends on the form of the rule. For example, the rules provided by a RuleSource declare references as method parameters.

引用要么是“by-path” 要么是 “by-type”
A reference is either “by-path” or “by-type”.

“by-type” reference就是通过类型来识别模型元素，例如，一个指向TaskContainer的引用在project space内可以有效的识别为task元素。通过类型绑定可以找到它。

A “by-type” reference identifies a particular model element by its type. For example, a reference to the TaskContainer effectively identifies the "tasks" element in the project model space. The model space is not exhaustively searched for candidates for by-type binding. The search space for a by-type binding is determined by the “scope” of the rule (discussed later).

“by-path” reference就是通过在model space内的path来识别一个元素，By-path references 通常对于规则目标来说是相对的，但这不影响它绑定什么引用，元素通过路径被识别，但是通过类型来引用，否则就会“binding failure”

A “by-path” reference identifies a particular model element by its path in model space. By-path references are always relative to the rule scope; there is currently no way to path “out” of the scope. All by-path references also have an associated type, but this does not influence what the reference binds to. The element identified by the path must however by type compatible with the reference, or a fatal “binding failure” will occur.

#### **69.3.5.1. Binding scope**

规则是通过“scope”来绑定的，它决定了引用绑定。大多数的规则在项目目标上绑定，然而，规则也可以被图标内被目标化。CollectionBuilder.named()方法就是一个例子。一个被目标化规则的机制。规则在构建脚本中用{}来宣布，或者通过根空间RuleSource插件作为目标。在默认的scope中被考虑。

Rules are bound within a “scope”, which determines how references bind. Most rules are bound at the project scope (i.e. the root of the model graph for the project). However, rules can be scoped to a node within the graph. The CollectionBuilder.named() method is an example, of a mechanism for applying scoped rules. Rules declared in the build script using the model {} block, or via a RuleSource applied as a plugin use the root of the model space as the scope. This can be considered the default scope.

By-path references对于rule scope总是相对的，当scope是root的时候，影响绑定到图谱上的任何元素，当不是root得时候，scope的孩子被引用给by-path。

By-path references are always relative to the rule scope. When the scope is the root, this effectively allows binding to any element in the graph. When it is not, the children of the scope can be referred to by-path.

当绑定by-type引用时，下面元素你需要考虑。

When binding by-type references, the following elements are considered:

目标元素自己

The scope element itself.

目标元素直属元素

The immediate children of the scope element.

modle space直属元素

The immediate children of the model space (i.e. project space) root.

通常规则都会在root下被定义，仅仅root直属的孩子受到规则影响。

For the common case, where the rule is effectively scoped to the root, only the immediate children of the root need to be considered.

## **69.4. Rule sources**

一种定义规则的方式是通过RuleSource的子类。类型作为插件实现以同样得方式被应用。

One way to define rules, is via a RuleSource subclass. Such types can be applied in the same manner (to project objects) as Plugin implementations (i.e. via Project.apply()).

Example 69.2. applying a rule source plugin

build.gradle
```
@Managed
interface Person {
  void setFirstName(String n); String getFirstName()
  void setLastName(String n); String getLastName()
}

class PersonRules extends RuleSource {
  @Model void person(Person p) {}

  @Mutate void setFirstName(Person p) {
    p.firstName = "John"
  }

 @Mutate void createHelloTask(CollectionBuilder<Task> tasks, Person p) {
    tasks.create("hello") {
      doLast {
        println "Hello $p.firstName $p.lastName!"
      }
    }
  }
}
```

apply plugin: PersonRules

Rule source插件可以以同样的方式与其他类型的插件打包和分发(见58章,编写自定义插件)。
规则的不同的方法来源是离散的,独立的规则。他们的订单,或者the fact属于同一类,无关紧要。

Rule source plugins can be packaged and distributed in the same manner as other types of plugins (see Chapter 61, Writing Custom Plugins).

The different methods of the rule source are discrete, independent rules. Their order, or the fact that they belong to the same class, are irrelevant.

Example 69.3. a model creation rule

build.gradle
```
@Model void person(Person p) {}
```

这条规则声明有一个模型元素为“person”路径(方法名定义的)“persion”类型。这是管理模型类型规则的形式类型。在这里,person对象是规则主题。方法可能会有方法体,实例。它也可能有更多的参数,也会规则的输入。

This rule declares the there is a model element at path "person" (defined by the method name), of type Person. This is the form of the Model type rule for Managed types. Here, the person object is the rule subject. The method could potentially have a body, that mutated the person instance. It could also potentially have more parameters, that would be the rule inputs.

Example 69.4. a model mutation rule

build.gradle
```
@Mutate void setFirstName(Person p) {
  p.firstName = "John"
}
```

这个变异规则变异了persion对象。该方法的第一个参数是主题。在这里,按类型引用作为参数没有路径注释。它也可能有更多的参数,将规则的输入。

This Mutate rule mutates the person object. The first parameter to the method is the subject. Here, a by-type reference is used as no Path annotation is present on the parameter. It could also potentially have more parameters, that would be the rule inputs.

Example 69.5. creating a task

build.gradle
```
@Mutate void createHelloTask(CollectionBuilder<Task> tasks, Person p) {
   tasks.create("hello") {
     doLast {
       println "Hello $p.firstName $p.lastName!"
     }
   }
 }
 ```

这种变异规则通过变异的任务集合有效地增加了一个任务。这里的主题是“任务”节点,这是可用的CollectionBuilder任务。唯一的输入元素是我们的人。作为人被用作输入在这里,它将一直在执行该规则被识别到。也就是说,任务容器有效取决于元素的人。如果有其他配置规则元素的人,可能在构建脚本中指定或其他插件,也将保证执行。

This Mutate rule effectively adds a task, by mutating the tasks collection. The subject here is the "tasks" node, which is available as a CollectionBuilder of Task. The lone input is our person element. As the person is being used as an input here, it will have been realised before executing this rule. That is, the task container effectively depends on the person element. If there are other configuration rules for the person element, potentially specified in a build script or other plugin, the will also be guaranteed to have been executed.

在这个例子中persion是托管类型,任何试图修改参数的人在这个方法会导致一个异常被抛出。在他们的生命周期在适当的点管理对象实施不变性。

As Person is a Managed type in this example, any attempt to modify the person parameter in this method would result in an exception being thrown. Managed objects enforce immutability at the appropriate point in their lifecycle.

更多信息,请参见RuleSource文档约束规则来源如何必须实现和更多类型的规则。

Please see the documentation for RuleSource for more information on constraints on how rule sources must be implemented and for more types of rules.

69.5. The “model DSL”
It is also possible to declare rules directly in the build script using the “model DSL”.

Example 69.6. the model dsl

build.gradle
```
model {
  person {
    lastName = "Smith"
  }
}
```

Continuing with the example so far of the model element "person" of type Person being present, the above DSL snippet effectively adds a mutation rule for the person that sets its lastName property.

The general form of the model DSL is:
```
model {
  «model-path-to-subject» {
    «imperative code»
  }
}
```
Where there may be multiple blocks.

It is also possible to create Managed type elements at the root level.

The general form of a creation rule is:
```
model {
  «element-name»(«element-type») {
    «imperative code»
  }
}
```       
The following model rule is creating the person element:

Example 69.7. a DSL creation rule

build.gradle
```
person(Person) {
  firstName = "John"
}
```
Note: The code for this example can be found at samples/modelRules/modelDsl in the ‘-all’ distribution of Gradle.

模型DSL当前是被限制的，只可以申报创建和一般突变规则。也只能通过by-path来引用主题,DSL是不可能为规则有输入的。这些限制将在未来Gradle版本解决。

The model DSL is currently quite limited. It is only possible to declare creation and general mutation rules. It is also only possible to refer to the subject by-path and it is not possible for the rule to have inputs. These are all limitations that will be addressed in future Gradle versions.

## **69.6. The model report**

构建中的模型任务按照树形结构显示model space。他可在查看元素绑定的时候被使用。

The built-in model task displays the model space as a tree. It can be used to see what the potential elements to bind to are.

Example 66.8. model task output

Output of gradle model
```
> gradle model
:model

------------------------------------------------------------
Root project
------------------------------------------------------------

model
    person
        firstName
        lastName
```

当前报告仅仅展示了model space的结构，未来Gradle版本将显示节点的类型和值。未来版本讲提供更丰富更互动性的探索model space的方法。

Currently the report only shows the structure of the model space. In future Gradle versions it will also display the types and values of the nodes. Future versions will also provide richer and more interactive ways of exploring the model space.

## **限制与未来方向**
69.7. Limitations and future direction

Rule based model configuration是Gradle的未来，这个领域才刚刚开始，但是也在积极的发展。早期的实验已经证明，这种方法更有效，能够提供更丰富的诊断和援助并且更具可扩展性。然而，还是有很多得限制。

Rule based model configuration is the future of Gradle. This area is fledgling, but under very active development. Early experiments have demonstrated that this approach is more efficient, able to provide richer diagnostics and authoring assistance and is more extensible. However, there are currently many limitations.


大多数的发展到目前为止都集中在证明该方法的有效性，以及构建的内部规则执行引擎和力学模型图.用户面临(e.g the DSL, rule source classes)这些尚未优化的简洁性和易用性问题。同样地,许多必要的配置模式和结构还不能够通过API表达。

The majority of the development to date has been focused on proving the efficacy of the approach, and building the internal rule execution engine and model graph mechanics. The user facing aspects (e.g the DSL, rule source classes) are yet to be optimized for conciseness and general usability. Likewise, many necessary configuration patterns and constructs are not yet able to be expressed via the API.

结合的更好的语法,更丰富的配置结构工具和更多的表达能力,将使更多的工具构建工程师和用户在新的方式中都理解,修改和扩展构建。

In conjunction with the addition of better syntax, a richer toolkit of configuration constructs and generally more expressive power, more tooling will be added that will enable build engineers and users alike to comprehend, modify and extend builds in new ways.

基于固有的自然规则为基础的方法,它与今天的Gradle建设构建模型相比更有效。然而,在未来它还将利用并行性,来节省配置和执行时间。此外,由于增加透明度模型，Gradle将能够进一步通过缓存和预处理机制减少构建时间。超出一般工具的构建性能,这将大大提高当使用Gradle等工具作为ide的体验性。

Due to the inherent nature of the rule based approach, it is more efficient at constructing the build model than today's Gradle. However, in the future Gradle will also leverage the parallelism that this approach enables both at configuration and execution time. Moreover, due to increased transparency of the model Gradle will be able to further reduce build times by caching and pre-computing the build model. Beyond improved general build performance, this will greatly improve the experience when using Gradle from tools such as IDEs.

Gradle的这个方面正在积极开发中,它将迅速被应用。请务必查看Gradle对应版本的文档以便您使用和观看的Gradle在未来版本的发行说明中的变化。

As this area of Gradle is under active development, it will be changing rapidly. Please be sure to consult the documentation of Gradle corresponding to the version you are using and to watch for changes announced in the release notes for future versions.
