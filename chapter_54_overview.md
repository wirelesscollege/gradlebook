# **Chapter 54. The Signing Plugin（签名插件）**

The signing plugin添加了为构建文件及产物做数字签名的功能。数字签名可以被用来证明谁构建了签名产物，以及其他的信息比如签名是什么时候生成的。

The signing plugin adds the ability to digitally sign built files and artifacts. These digital signatures can then be used to prove who built the artifact the signature is attached to as well as other information such as when the signature was generated.

The signing plugin目前只支持生成PGP签名(是一种被Maven Central Repository要求的出版所需的签名格式)。

The signing plugin currently only provides support for generating PGP signatures (which is the signature format required for publication to the Maven Central Repository).

## **54.1. Usage**

在使用the Signing plugin时需包含如下内容到你的构建脚本中

To use the Signing plugin, include the following in your build script:

例54.1 使用签名插件

Example 54.1. Using the Signing plugin

build.gradle
```
apply plugin: 'signing'
```

## **54.2. Signatory credentials**

为了创建PGP签名，你需要一个钥匙对（你可以在GnuPG HOWTOs中找到使用GnuPG tools 创建钥匙对时用到的那些指令/命令）。你需要给签名插件你的钥匙信息，这就意味着三件事：

In order to create PGP signatures, you will need a key pair (instructions on creating a key pair using the GnuPG tools can be found in the GnuPG HOWTOs). You need to provide the signing plugin with your key information, which means three things
```
•	公钥ID（一个8个字符的十六进制字符串）
•	包含私钥的秘钥环文件的绝对路径
•	用来保护你私钥的密码
•	The public key ID (an 8 character hexadecimal string).
•	The absolute path to the secret key ring file containing your private key.
•	The passphrase used to protect your private key.
```

这几项必须要提供signing.keyId, signing.secretKeyRingFile, 和 signing.password属性的值。考虑到这些值所特有和私有的属性，把他们放在用户gradle.properties文件里是个很好的做法（在“Gradle properties and system properties”19.2部分描述过）

These items must be supplied as the values of properties signing.keyId, signing.secretKeyRingFile, and signing.passwordrespectively. Given the personal and private nature of these values, a good practice is to store them in the user gradle.propertiesfile (described in Section 19.2, “Gradle properties and system properties”).
signing.keyId=24875D73
signing.password=secret
signing.secretKeyRingFile=/Users/me/.gnupg/secring.gpg

如果在用户gradle.properties文件中指定这些信息你的环境是不可行的，那么你就需要无论如何找到信息源头，手动地设置项目属性。

If specifying this information in the user gradle.properties file is not feasible for your environment, you can source the information however you need to and set the project properties manually.

```
import org.gradle.plugins.signing.Sign

gradle.taskGraph.whenReady { taskGraph ->
    if (taskGraph.allTasks.any { it instanceof Sign }) {
        // Use Java 6's console to read from the console (no good for
        // a CI environment)
        Console console = System.console()
        console.printf "\n\nWe have to sign some things in this build." +
                       "\n\nPlease enter your signing details.\n\n"

        def id = console.readLine("PGP Key Id: ")
        def file = console.readLine("PGP Secret Key Ring File (absolute path): ")
        def password = console.readPassword("PGP Private Key Password: ")

        allprojects { ext."signing.keyId" = id }
        allprojects { ext."signing.secretKeyRingFile" = file }
        allprojects { ext."signing.password" = password }

        console.printf "\nThanks.\n\n"
    }
}
```
        
## **54.3. Specifying what to sign**

和配置怎样签署（即签署配置）一样，你同样需要指明签什么。The Signing plugin提供了DSL来指明需要签署的任务和/或配置

As well as configuring how things are to be signed (i.e. the signatory configuration), you must also specify what is to be signed. The Signing plugin provides a DSL that allows you to specify the tasks and/or configurations that should be signed.

### **54.3.1. Signing Configurations**

想要签署一个配置的构建产物很常见。例如,Java插件配置一个jar来构建，进而这个jar的构建产物被添加到产物配置中。使用DSL签署,您就可以指定的所有应该签署的配置产物。

It is common to want to sign the artifacts of a configuration. For example, the Java plugin configures a jar to build and this jar artifact is added to the archives configuration. Using the Signing DSL, you can specify that all of the artifacts of this configuration should be signed.

Example 54.2. Signing a configuration

build.gradle
```
signing {
    sign configurations.archives
}
```

在你的项目中创建一个名字叫signArchives的任务（sign类型），如果有需要的话可以构建任何产物并且为它们生成签名。签名文件将会放在构建产物被签名的位置。

This will create a task (of type Sign) in your project named “signArchives”, that will build any archives artifacts (if needed) and then generate signatures for them. The signature files will be placed alongside the artifacts being signed.

Example 54.3. Signing a configuration output

Output of gradle signArchives
````
> gradle signArchives
:compileJava
:processResources
:classes
:jar
:signArchives

BUILD SUCCESSFUL

Total time: 1 secs
```

### **54.3.2. Signing Tasks**

在某些情况下您需要签署的产物可能不是配置的一部分。在这种情况下你可以直接签署产生此构建产物的任务。

In some cases the artifact that you need to sign may not be part of a configuration. In this case you can directly sign the task that produces the artifact to sign.

Example 54.4. Signing a task

build.gradle
```
task stuffZip (type: Zip) {
    baseName = "stuff"
    from "src/stuff"
}

signing {
    sign stuffZip
}
```

这将会在你的项目里创建一个名为“signStuffZip”的任务（Sign类型），它会构建输入任务的产物（如果需要）并给它签名。签名文件会放在签名产物的旁边。

This will create a task (of type Sign) in your project named “signStuffZip”, that will build the input task's archive (if needed) and then sign it. The signature file will be placed alongside the artifact being signed.

例54.5  签署一项任务输出

Example 54.5. Signing a task output

Output of gradle signStuffZip
```
> gradle signStuffZip
:stuffZip
:signStuffZip

BUILD SUCCESSFUL

Total time: 1 secs
```

对于一个要被签署的任务，她必须要生成某种类型的产物。能够做到这点的是 Tar, Zip, Jar, War 和 Ear类型的任务。

For a task to be “signable”, it must produce an archive of some type. Tasks that do this are the Tar, Zip, Jar, War and Ear tasks.

### **54.3.3. Conditional Signing**

一种常见的使用模式是仅在一定条件下签署构建产物。例如，你不希望签署非发布版本的构建产物，为了做到这点，你就可以指定签名仅在一定条件下执行。

A common usage pattern is to only sign build artifacts under certain conditions. For example, you may not wish to sign artifacts for non release versions. To achieve this, you can specify that signing is only required under certain conditions.

例54.6  有条件的签名

Example 54.6. Conditional signing

build.gradle
```
version = '1.0-SNAPSHOT'
ext.isReleaseVersion = !version.endsWith("SNAPSHOT")

signing {
    required { isReleaseVersion && gradle.taskGraph.hasTask("uploadArchives") }
    sign configurations.archives
}
```

在本例中，仅当正在创建一个发布版本并且即将发布的时候我们需要签名。因为我们是要检查任务图确定我们是否要出版,我们必须设置signing.required属性为关闭状态以延迟评估。查看 SigningExtension.setRequired()来获取更多信息。

In this example, we only want to require signing if we are building a release version and we are going to publish it. Because we are inspecting the task graph to determine if we are going to be publishing, we must set the signing.required property to a closure to defer the evaluation. See SigningExtension.setRequired() for more information.

## **54.4. Publishing the signatures**

当通过签署DSL指明签署内容后，合成的签署产物将会自动的加入到签名和产物依赖配置中。这就意味着如果你想要上传你的签名和产物到分布式存储库仅需要跟平常一样执行uploadArchives 任务即可。

When specifying what is to be signed via the Signing DSL, the resultant signature artifacts are automatically added to the signatures and archives dependency configurations. This means that if you want to upload your signatures to your distribution repository along with the artifacts you simply execute the uploadArchives task as normal.

## **54.5. Signing POM files**

当部署构建产物签名到Maven存储库中时，你也会想要将已发布的POM文件签名。将签名插件添加一个signing.signPom()（查看:SigningExtension.signPom()）方法，它用于你上传任务配置中的beforeDeployment()块中

When deploying signatures for your artifacts to a Maven repository, you will also want to sign the published POM file. The signing plugin adds a signing.signPom() (see:SigningExtension.signPom()) method that can be used in the beforeDeployment() block in your upload task configuration.

Example 54.7. Signing a POM for deployment

build.gradle
```
uploadArchives {
    repositories {
        mavenDeployer {
            beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }
        }
    }
}
```
当签名没有被要求且POM由于配置不足不能被签名的时候（即没有签名的凭证），signPom()方法将保持静止什么都不做。

When signing is not required and the POM cannot be signed due to insufficient configuration (i.e. no credentials for signing) then the signPom() method will silently do nothing.

百度搜索[无线学院](http://wirelesscollege.cn)

