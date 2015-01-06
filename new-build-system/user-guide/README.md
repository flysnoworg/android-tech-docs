目录
-----

* [1 介绍](#1)
    * [1.1 新构建系统的目标](#11)
    * [1.2 Gradle是什么?](#12)
* [2 要求](#2)
* [3 基础项目](#3)
    * [3.1 基本的build文件](#31)
    * [3.2 项目结构](#32)
        * [3.2.1 配置结构](#321)
    * [3.3 构建任务](#33)
        * [3.3.1 通用任务](#331)
        * [3.3.2 Java项目任务](#332)
        * [3.3.3 Android任务](#333)
    * [3.4 自定义构建](#34)
        * [3.4.1 Manifest选项](#341)
        * [3.4.2 构建类型](#342)
        * [3.4.3 签名配置](#343)
        * [3.4.4 使用混淆](#344)
        * [3.4.5 清理资源](#345)
* [4 依赖，Android库工程以及多工程设置](#4)
    * [4.1 依赖二进制包](#41)
        * [4.1.1 本地包](#411)
        * [4.1.2 远程artifacts](#412)
    * [4.2 Multi project setup](#42)
    * [4.3 Library projects](#43)
        * [4.3.1 Creating a Library Project](#431)
        * [4.3.2 Differences between a Project and a Library Project](#432)
        * [4.3.3 Referencing a Library](#433)
        * [4.3.4 Library Publication](#434)
* [5 Testing](#5)
    * [5.1 Basics and Configuration](#51)
    * [5.2 Running tests](#52)
    * [5.3 Testing Android Libraries](#53)
    * [5.4 Test reports](#54)
        * [5.4.1 Single projects](#541)
        * [5.4.2 Multi-projects reports](#542)
    * [5.5 Lint support](#55)
* [6 Build Variants](#6)
    * [6.1 Product flavors](#61)
    * [6.2 Build Type + Product Flavor = Build Variant](#62)
    * [6.3 Product Flavor Configuration](#63)
    * [6.4 Sourcesets and Dependencies](#64)
    * [6.5 Building and Tasks](#65)
    * [6.6 Testing](#66)
    * [6.7 Multi-flavor variants](#67)
* [7 Advanced Build Customization](#7)
    * [7.1 Build options](#71)
        * [7.1.1 Java Compilation options](#711)
        * [7.1.2 aapt options](#712)
        * [7.1.3 dex options](#713)
    * [7.2 Manipulating tasks](#72)
    * [7.3 BuildType and Product Flavor property reference](#73)
    * [7.4 Using sourceCompatibility 1.7](#74)

<a id="1" href="#1"></a>

## 1 介绍

本文档适用于Gradle plugin 0.9版本，所以可能和我们1.0之前介绍的老版本有所不同。

<a id="11" href="#11"></a>

### 1.1 新构建系统的目标

新构建系统的目标是：

* 可以很容易的重用代码和资源
* 可以很容易的创建应用的衍生版本，所以不管你是创建多个apk，还是不同功能的应用都很方便
* 可以很容易的配置、扩展以及自定义构建过程
* 和IDE无缝整合

<a id="12" href="#12"></a>

### 1.2 Gradle是什么

Gradle是一个非常优秀的构建系统工具，允许你通过插件的方式创建自定义的构建逻辑

Gradle的以下特性让我们选择了它：

* 用过领域专用语言(DSL)描述和控制构建逻辑
* 构建文件基于Groovy，并且可以组合使用各种定义的元素，然后通过代码来控制这些DSL达到定制逻辑的目的
* 内建的基于Maven或者Ivy的依赖管理
* 使用非常灵活，Gradle不会强制实现的方式，你可以使用最佳实践
* 插件能提供DSL以及API为构建文件使用
* 良好的工具API以供IDE集成

<a id="2" href="#2"></a>

## 2 要求

* Gradle 1.10或者1.11或者1.12，并且使用0.11.1版本的插件
* SDK with Build Tools要求19.0.0，有些功能可能需要更新的版本

<a id="3" href="#3"></a>

## 3 基础项目

一个Gradle工程是通过名字叫 *build.gradle* 的文件描述其构建过程的，该文字位于工程的根目录下。

<a id="31" href="#31"></a>

### 3.1 基本的build文件

最基本的Java工程，其 *build.gradle* 非常简单：

    apply plugin: 'java'

这里应用了Gradle提供的Java插件。该插件提供了构建和测试Java应用所需的一些东西。

一个最基本的Android工程的build.gradle如下：

    buildscript {
        repositories {
            mavenCentral()
        }
    
        dependencies {
            classpath 'com.android.tools.build:gradle:0.11.1'
        }
    }
    
    apply plugin: 'android'
    
    android {
        compileSdkVersion 19
        buildToolsVersion "19.0.0"
    }

在Android buid file中，有3个主要组成部分。

buildscript { ... }部分配置了驱动构建的代码。

在该部分中，定义配置使用了Maven中央仓库，并且声明依赖一个Maven artifact(构件？)。 这个artifact是一个包含0.11.1版本的Android Gradle插件库。

**注意：** 这部分的配置只会影响构建过程的代码，和你的工程没有关系。工程会定义它自己的仓库和相关依赖。稍候会详细介绍。

接下来，android插件被应用，这和上面的Java插件是一样的。

最后，android { ... }这部分配置了android构建需要的所有参数。这里也是Android DSL的入口点。

默认情况下，只有编译的目标SDK、构建工具的版本是必需的。就是 `compileSdkVersion` 和 `buildtoolsVersion` 两个配置属性。

compilation target和旧构建系统中的project.properties文件里 **target** 属性是一样的。这个新的属性和以前的 **target** 一样，可以指定一个int类型(api级别)或者string类的值。

**重要:** 你应该只应用android插件就好了，不要同时应用java插件，因为这会导致构建错误

**注意：** 你还需要在同目录下添加一个 *local.properties* 文件，并通过 `sdk.dir` 属性配置所需的SDK的路径。除此之外，你也可以设置一个名为 `ANDROID_HOME` 环境变量。这两种方法都差不多，你可以选择自己喜欢的。

<a id="32" href="#32"></a>

### 3.2 项目结构

上面说的build文件约定了一个默认的文件夹结构。Gradle遵循约定优先于配置的原则，在可能的情况下提供合理的默认值。

基本的项目始于两个名为"source sets"的部分。也就是main source code 和 test code。他们分别位于：

* src/main
* src/androidTest/

里面的每一个文件夹都对应相应的组件。

对于Java和Android这两个插件来说，他们的Java源代码和Java资源的位置是：

* java/
* resources/

对于Android插件来说，它还有以下特性文件和文件夹：

* AndroidManifest.xml
* res/
* assets/
* aidl/
* rs/
* jni/

**注意：** src/androidTest/AndroidManifest.xml是不需要的，它会被自动创建。

<a id="321" href="#321"></a>

#### 3.2.1 配置结构

当默认的项目结构不适用的时候，你可能需要配置它。根据Gradle文档说明，可以通过如下方式重新配置Java项目的 `sourceSets`:

    sourceSets {
        main {
            java {
                srcDir 'src/java'
            }
            resources {
                srcDir 'src/resources'
            }
        }
    }

**注意：** `srcDir` 会添加指定的文件夹到现有的源文件夹列表中(Gradle文档没有提到这个，但是的确是这样)。

要替换默认的源文件夹的话，可以给 `srcDirs` 指定一个路径数组 。下面使用对象调用另一种方式配置：

    sourceSets {
        main.java.srcDirs = ['src/java']
        main.resources.srcDirs = ['src/resources']
    }
    
想要了解更多的信息，请参考Gradle文档的[Java插件](http://gradle.org/docs/current/userguide/java_plugin.html)部分。

Android插件也使用相似的语法，但是它有它自己的 *sourceSets* ,这些已经内置在android对象中了。

这儿有个示例，它使用了旧项目结构的源代码，并且重新映射了androidTest *sourceSet* 到测试文件夹：

    android {
        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res.srcDirs = ['res']
                assets.srcDirs = ['assets']
            }
    
            androidTest.setRoot('tests')
        }
    }

**注意：** 因为旧结构中把所有的源文件(java, aidl, renderscript, and java resources)都放在同一文件夹下，所以我们需要重新映射这些 *sourceSet* 的新组件到同一src目录.

**注意:** setRoot()方法会移动整个 *sourceSet* (包括其下的子文件夹)到一个新文件夹。这里是移动src/androidTest/\*到tests/\*。

这些都是Android特有的，并不适用于Java *sourceSets* 。

这是一个迁移的例子(译者注：比如从旧项目结构迁移过来)。

<a id="33" href="#33"></a>

### 3.3 构建任务

<a id="331" href="#331"></a>

#### 3.3.1 通用任务

在构建文件中应用一个插件的时候会自动的创建一系列可运行的构建任务。Java plugin 和 the Android plugin都可以做到这一点。以下是约定的一些任务：

* **assemble** 这个任务会汇集项目的所有输出。
* **check** 这个任务会执行所有校验检查
* **build** 这个任务会同时执行 **assemble** 和 **check** 任务
* **clean** 这个任务会清理项目的所有输出

事实上， **assemble** ,  **check** 以及 **build** 这三个任务并没有作任何事情，他们只是插件的引导任务，引导插件添加的其他任务去完成一些工作。

这样就可以允许你调用同样的任务，而不用管它是什么类型的项目或者应用了什么插件。

比如，应用 *findbugs* 插件会创建一个任务，并且让 **check** 任务依赖它，这样当这个 **check** 任务被调用的时候，这个新创建的任务也会被调用.

在命令行中输入如下命令可以获取一些高级别的任务介绍。

```bash
gradle tasks
```

要查看所有的任务列表以及任务之间的依赖关系运行：

```bash
gradle tasks --all
```

**注意：** Gradle会自动监控任务定义的输入和输出。

不做任何改变两次运行 **build** ，Gradle会报告所有任务已经处于UP-TO-DATE状态，这意味着没有什么可做的。这使得任务之间可以正确的相互依赖，又不会导致其他不需要的操作执行。

<a id="332" href="#332"></a>

#### 3.3.2 Java项目任务

Java plugin创建了两个主要的任务，主要的引导任务都依赖他们。

* **assemble** 
    * **jar** 这个任务创建所有输出
* **check** 
    * **test** 这个任务运行所有测试

**jar** 任务直接或者间接的依赖其他任务：比如 **classes** 会编译所有Java代码.

**testClasses** 会编译所有测试，但是它很少使用，因为 **test**这个任务依赖它(和 **classes** 差不多)。

通常情况下，你可能只用到 **assemble**  或者 **check** ,其他的任务不会使用。

你可以在[这儿](http://gradle.org/docs/current/userguide/java_plugin.html)看到Java plugin的所有任务列表以及他们的依赖关系

<a id="333" href="#333"></a>

#### 3.3.3 Android任务

Android plugin使用了同样的约定规则以和其他插件保持兼容，并且又添加了一些额外的引导任务:

* **assemble** 这个任务会汇集项目的所有输出。
* **check** 这个任务会执行所有校验检查
* **connectedCheck** 运行checks需要一个连接的设备或者模拟器，这些checks将会同时运行在所有连接的设备上。
* **deviceCheck** 通过API连接远程设备运行checks。它被用于CI(译者注:持续集成)服务器上。
* **build** 这个任务会同时执行 **assemble** 和 **check** 任务
* **clean** 这个任务会清理项目的所有输出

这些新的引导任务是必须的，以便能够在没有连接的设备的情况下运行定期检查。

注意 **build** 既不依赖 **deviceCheck** ,也不依赖 **connectedCheck** 。

一个Android工程至少有两个输出:一个debug APK和一个release APK。他们每一个都有自己的引导任务以便可以单独的构建他们：

* **assemble**
    * **assembleDebug**
    * **assembleRelease**

他们两个都依赖其他任务，这些任务执行很多必须的步骤以生成一个APK。 **assemble** 任务又依赖他们两个，所以执行 **assemble** 会生成两个APK。

提示：在命令行下，Gradle支持任务名称驼峰方式的快捷调用，比如：

```bash
gradle aR
```
和
```bash
gradle assembleRelease
```
是一样的，当然前提是没有其他任务匹配'aR'。

检验引导任务也有他们自己的依赖：
* **check**
    * **lint** 
* **connectedCheck**
    * **connectedAndroidTest**
    * **connectedUiAutomatorTest** (尚未实现)
* **deviceCheck** 
    * 这个依赖其他插件创建的时候实现的测试扩展点的哪些任务。

最后，插件会为所有的构建类型( **debug, release, test** )创建install/uninstall任务，也只有他们能被安装（需要签名）。

<a id="34" href="#34"></a>

### 3.4 自定义构建

Android plugin提供了大量的DSL能够让你直接基于构建系统定制很多事情。

<a id="341" href="#341"></a>

#### 3.4.1 Manifest选项

通过DSL可以配置manifest的如下选项：

* minSdkVersion
* targetSdkVersion
* versionCode
* versionName
* applicationId (更有效的packageName -- 请看[ApplicationId 与 PackageName](../applicationid-vs-packagename.md)获取更多信息)
* Package Name for the test application
* Instrumentation test runner

示例:

    android {
        compileSdkVersion 19
        buildToolsVersion "19.0.0"
    
        defaultConfig {
            versionCode 12
            versionName "2.0"
            minSdkVersion 16
            targetSdkVersion 16
        }
    }

**android** 元素里的 **defaultConfig** 负责定义所有的配置。

Android Plugin以前的版本是使用packageName配置manifest的'packageName'属性。
从0.11.0开始，你应该在build.gradle里使用applicationId来配置manifest的'packageName'属性。
这是为了消除应用的packageName（也就是ID）和java的packages之间的歧义。

通过build文件定义的强大之处在于可以动态的被配置。
比如，可以从一个文件读取版本名字，或者使用一些其他的自定义的逻辑：

    def computeVersionName() {
        ...
    }
    
    android {
        compileSdkVersion 19
        buildToolsVersion "19.0.0"
    
        defaultConfig {
            versionCode 12
            versionName computeVersionName()
            minSdkVersion 16
            targetSdkVersion 16
        }
    }
    
**注意：**  不要使用在给定的范围内，和其他已经存在的getters方法冲突的方法名字。比如在defaultConfig { ...}中调用getVersionName()方法会自动的调用defaultConfig.getVersionName()方法，如果你也自定义一个这样的名字的方法，那么你的方法不会调用。

如果一个属性没有通过DSL设置，则会使用它们的默认值。这里有个表格说明是如何处理的。

属性铭|DSL对象的默认值|默认值
-----|-------------|-----
versionCode|-1|如有有的话从manifest中读取
versionName|null|如有有的话从manifest中读取
minSdkVersion|-1|如有有的话从manifest中读取
targetSdkVersion|-1|如有有的话从manifest中读取
applicationId|null|如有有的话从manifest中读取
testApplicationId|null|applicationId + “.test”
testInstrumentationRunner|null|android.test.InstrumentationTestRunner
signingConfig|null|null
proguardFile|N/A (只能设置)|N/A (只能设置)
proguardFiles|N/A (只能设置)|N/A (只能设置)

如果你在构建脚本中使用自定义的逻辑获取这些属性的时候，那么第二列的值尤其重要。比如，你可能这样写：

```java
if (android.defaultConfig.testInstrumentationRunner == null) {
    // assign a better default...
}
```

如果值一直为null，那么在构建的时候，它将会被从第三列中获取的实际的默认值替换，但是在DSL元素中又不包含这个默认值，所以你无法查询到它。
这是为了防止解析应用的manifest文件，除非真的需要。

<a id="342" href="#342"></a>

#### 3.4.2 构建类型

默认情况下，Android plugin会自动的设置项目，构建release和debug两个版本。
他们主要的差异主要在于是否可以在设备上调试应用以及APK如何签名。

debug版本会被使用已知的名称/密码自动生成的密钥/证书签名。release版本在构建过程中不会被签名，需要构建后再签名。

这些配置可以通过一个叫 **BuildType** 配置。默认情况下，已经创建了 **debug** 和 **release** 这两个实例。

Android plugin允许自定义这两个示例，并且可以创建其他的 *Build Types* 。这些是可以在 **buildTypes** DSL容器中配置完成:

    android {
        buildTypes {
            debug {
                applicationIdSuffix ".debug"
            }
    
            jnidebug.initWith(buildTypes.debug)
            jnidebug {
                packageNameSuffix ".jnidebug"
                jniDebuggable true
            }
        }
    }
    
以上的片段实现了以下几点：

* 配置了默认的 **debug** 构建类型：
    * 设置包名为<app appliationId>.debug以便可以在同一设备上同时安装 *debug* 和 *release* 两个版本的APK
* 创建一个叫 **jnidebug** 新的 *Build Types* ，并且配置它作为 **debug** 构建类型的一个副本
* 然后再配置 **jnidebug** ，启用JNI组件的debug构建，并且添加一个不同的包名后缀

创建一个新的 *Build Types* 很简单，只需要在 **buildTypes** 容器下添加一个元素，然后调用 **initWith()** 或者使用一个闭包配置它。

这里有一些可能用到的属性以及他们的默认值：

属性名|debug时的默认值|release或者其他类型的默认值
-----|--------------|-----------------------
**debuggable** |true|false
**jniDebuggable** |false|false
**renderscriptDebuggable** |false|false
**renderscriptOptimLevel** |3|3
**applicationIdSuffix** |null|null
**versionNameSuffix** |null|null
**signingConfig** |android.signingConfigs.debug|null
**zipAlignEnabled** |false|true
**minifyEnabled** |false|false
**proguardFile** |N/A (只能设置)|N/A (只能设置)
**proguardFiles** |N/A (只能设置)|N/A (只能设置)

除了这些属性外， 代码和资源也会影响到 *Build Types* 。
对于每一个 Build Type,都会创建一个新的匹配的 *sourceSet* ，默认位置是

    src/<buildtypename>/
    
这意味着 *Build Type* 的名字不能是 *main* 和 *androidTest* (这两个已经被插件占用)，并且他们相互之间的名字必须唯一。

和其他的source sets一样，Build Type的source set的位置可以被重定向：

    android {
        sourceSets.jnidebug.setRoot('foo/jnidebug')
    }
    
此外，对于每一个 *Build Type* ，都会新创建assemble\<BuildTypeName\>任务。

**assembleDebug** 和 **assembleRelease** 这两个任务已经讲过了，这里讲的是他们是从哪来的。是在 **debug** 和 **release** 这两个 *Build Types* 被预先创建的时候。

提示：记得你可以通过输入gradle aJ来运行assembleJnidebug任务哦。

可能使用到的情况：

* 在debug模式下需要，但是在release下不需要的权限
* 自定义debug的实现
* 微debug默认使用不同的资源（比如一个资源的值是由签名的证书决定的）

*BuildType* 的代码/资源主要通过以下方式使用：

* manifest被合并进app的manifest
* 代码仅仅是作为一个额外的source文件夹(译者注：其实和自己新建一个source文件夹，然后在这个文件夹下新建包和类一样)
* 资源会覆盖main里的资源，替换现有的值

<a id="343" href="#343"></a>

#### 3.4.3 签名配置

要对一个应用签名，要求如下：

* 一个keystore
* 一个keystore的密码
* 一个key的别名
* 一个key的密码
* 存储类型

位置、key别名、key密码以及存储类型一起组成了签名配置( *SigningConfig* 类型)

默认情况下， 已经有了一个 **debug** 的签名配置，它使用了debug keystore，该keystore有一个已知的密码和默认的带有已知密码的key。
debug keystore位于$HOME/.android/debug.keystore，如果没有会被创建。

**debug** *Build Type* 被设置为自动使用 **debug** 签名配置。

你也可以创建其他的签名配置或者自定义内置的配置。可以通过 **signingConfigs** DSL容器实现。

    android {
        signingConfigs {
            debug {
                storeFile file("debug.keystore")
            }
    
            myConfig {ss
                storeFile file("other.keystore")
                storePassword "android"
                keyAlias "androiddebugkey"
                keyPassword "android"
            }
        }
    
        buildTypes {
            foo {
                debuggable true
                jniDebuggable true
                signingConfig signingConfigs.myConfig
            }
        }
    }
    
以上片段会把debug keystore的路径改为项目的根目录。这会自动的影响任何用到它的 *Build Types* ，在这里影响到的是 **debug** *Build Type* 。

以前片段也创建了一个新的签名配置，并且被一个新的 *Build Type* 使用。

**注意：** 只有默认路径下的debug keystores才会被自动创建。如果改变了debug keystore的路径将不会在需要的时候创建。创建一个使用不同名字的 *SigningConfig* ，但是用的是默认的debug keystore路径的话是会被自动创建的。也就是说，会不会被自动创建，和keystore的路径有关，和配置的名字无关。

**说明：** 通常情况下，会使用项目根目录的相对路径作为keystores的路径，但有时候也会用绝对路径，虽然这并不推荐(被自动创建的debug keystore除外)。

**注意：如果已经你将这些文件放到版本控制中，你可能不想把密码存储在文件中。Stack Overflow上有个帖子介绍可以从控制台或者环境变量中获取这些密码等信息。**

[http://stackoverflow.com/questions/18328730/how-to-create-a-release-signed-apk-file-using-gradle](http://stackoverflow.com/questions/18328730/how-to-create-a-release-signed-apk-file-using-gradle)

**我们以后更新这个指南的时候会添加更多的信息。**

<a id="344" href="#344"></a>

#### 3.4.4 使用混淆

自从Gradle plugin for ProGuard 4.10版本以后，Gradle开始支持混淆。如果通过Build Type的 *minifyEnabled* 属性配置了使用混淆后，The ProGuard plugin会自动被应用，并且自动创建一些任务。

    android {
        buildTypes {
            release {
                minifyEnabled true
                proguardFile getDefaultProguardFile('proguard-android.txt')
            }
        }
    
        productFlavors {
            flavor1 {
            }
            flavor2 {
                proguardFile 'some-other-rules.txt'
            }
        }
    }
    
使用buildTypes以及productFlavors定义的规则文件可以轻松的生成多种版本。

有两个默认的规则文件

* proguard-android.txt
* proguard-android-optimize.txt

他们位于SDK中，使用getDefaultProguardFile()方法可以返回文件的全路经。除了是否启用优化之外，这两个文件的其他功能都是相同的。

<a id="345" href="#345"></a>

#### 3.4.5 清理资源

在构建的时候，你也可以自动的移除一些未使用的资源。更多信息，请参考[资源清理](resource-shrinking.md)文档

<a id="4" href="#4"></a>

## 4 依赖，Android库工程以及多工程设置

Gradle可以依赖其他的一些组件，这些组建可以是外部二进制包，也可以是其他Gradle工程。

<a id="41" href="#41"></a>

### 4.1 依赖二进制包

<a id="411" href="#411"></a>

#### 4.1.1 本地包

要配置依赖一个外部库jar包，你可以在 **compile** 配置里添加一个依赖。

    dependencies {
        compile files('libs/foo.jar')
    }
    
    android {
        ...
    }

注： **dependencies** DSL元素是标准Gradle API的一部分，并不属于 **android** 的元素。

**compile** 配置用来编译main application，它里面的一切都会被添加到编译的classpath中，并且也会被打包到最终的APK中。

这里还有添加依赖时其他的配置：

* **compile：** main application
* **androidTestCompile：** test application
* **debugCompile：** debug Build Type
* **releaseCompile：** release Build Type

因为要构建生成一个APK，必然会有相关联的 *Build Type* ，APK默认配置了两个(或者更多)编译配置：compile和\<buildtype\>Compile。
创建一个新的 *Build Type* 的时候会自动创建一个基于它名字的编译配置。

当一个debug版本需要一个自定义库(比如报告崩溃)，但是release版本不需要或者需要一个不同版本的库的时候，会显得非常有用。

<a id="412" href="#412"></a>

#### 4.1.2 远程 artifacts

Gradle支持从Maven和Ivy仓库获取artifacts。

首先必须把库添加到列表中，并且以Maven或者Ivy的定义方式定义需要的依赖。

    repositories {
        mavenCentral()
    }
    
    
    dependencies {
        compile 'com.google.guava:guava:11.0.2'
    }
    
    android {
        ...
    }
    
注： **mavenCentral()** 是指定仓库URL的快捷方式。Gradle同时支持远程和本地两种仓库
注：Gradle遵循依赖的传递性。这就意味着如果依赖本身会依赖其他东西，这些也会被拉取过来。

更多关于dependencies的设置信息，请参见Gradle 用户指南[这儿](http://gradle.org/docs/current/userguide/artifact_dependencies_tutorial.html)，以及DSL文档[这儿](http://gradle.org/docs/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.htmls)


<a id="42" href="#42"></a>

### 4.2 Multi project setup

<a id="43" href="#43"></a>

### 4.3 Library projects

<a id="431" href="#431"></a>

#### 4.3.1 Creating a Library Project

<a id="432" href="#432"></a>

#### 4.3.2 Differences between a Project and a Library Project

<a id="433" href="#433"></a>

#### 4.3.3 Referencing a Library

<a id="434" href="#434"></a>

#### 4.3.4 Library Publication

## 5 Testing

### 5.1 Basics and Configuration

### 5.2 Running tests

### 5.3 Testing Android Libraries

### 5.4 Test reports

#### 5.4.1 Single projects

#### .4.2 Multi-projects reports

### 5.5 Lint support

## 6 Build Variants

### 6.1 Product flavors

### 6.2 Build Type + Product Flavor = Build Variant

### 6.3 Product Flavor Configuration

### 6.4 Sourcesets and Dependencies

### 6.5 Building and Tasks

### 6.6 Testing

### 6.7 Multi-flavor variants

## 7 Advanced Build Customization

### 7.1 Build options

#### 7.1.1 Java Compilation options

#### 7.1.2 aapt options

#### 7.1.3 dex options

### 7.2 Manipulating tasks

### 7.3 BuildType and Product Flavor property reference

### 7.4 Using sourceCompatibility 1.7