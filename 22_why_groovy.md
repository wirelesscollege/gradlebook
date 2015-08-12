## **2.2. Why Groovy?**

    Why Groovy?
    首先我们认为内部的DSL语言比XML结构化语言要高级，另外Groovy对Java项目支持的非常好。
    
    We think the advantages of an internal DSL (based on a dynamic language) over XML are tremendous in case of build scripts. There are a couple of dynamic languages out there. Why Groovy? The answer lies in the context Gradle is operating in. Although Gradle is a general purpose build tool at its core, its main focus are Java projects. In such projects obviously the team members know Java. We think a build should be as transparent as possible to all team members.

    你可以会质疑为什么不使用java语言作为构建脚本语言？我们认为这是一个比较有价值的问题，因为Groovy和其他脚本一样比Java更加高效，另外就是Groovy与Java语言基本一致，大家都比较容易接受。

    You might argue why not using Java then as the language for build scripts. We think this is a valid question. It would have the highest transparency for your team and the lowest learning curve. But due to limitations of Java such a build language would not be as nice, expressive and powerful as it could be. [1] Languages like Python, Groovy or Ruby do a much better job here. We have chosen Groovy as it offers by far the greatest transparency for Java people. Its base syntax is the same as Java's as well as its type system, its package structure and other things. Groovy builds a lot on top of that. But on a common ground with Java.

    如果你们使用Python和Ruby的话，Gradle也支持Jpython与Jruby但是我们开发它的优先级将不会特别高。
    
    For Java teams which share also Python or Ruby knowledge or are happy to learn it, the above arguments don't apply. The Gradle design is well-suited for creating another build script engine in JRuby or Jython. It just doesn't have the highest priority for us at the moment. We happily support any community effort to create additional build script engines.

    百度搜索[无线学院](http://wirelesscollege.cn)
