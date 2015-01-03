目录
-----

* [1 介绍](#1-介绍)
    * [1.1 新构建系统的目标](#11-新构建系统的目标)
    * [1.2 Gradle是什么?](#12-Gradle是什么)
* [2 要求](#2-要求)
* [3 基础项目](#3-基础项目)
    * [3.1 基本的build文件](#31-基本的build文件)
    * [3.2 项目结构](#32-项目结构)
        * [3.2.1 配置结构](#321-配置结构)
    * [3.3 构建任务](#33-构建任务)
        * [3.3.1 通用任务](#331-通用任务)
        * [3.3.2 Java项目任务](#332-Java项目任务)
        * [3.3.3 Android任务](#333-Android任务)
    * [3.4 自定义构建](#34-自定义构建)
        * [3.4.1 Manifest选项](#341-Manifest选项)
        * [3.4.2 Build Types](#TOC-Build-Types)
        * [3.4.3 Signing Configurations](#TOC-Signing-Configurations)
        * [3.4.4 Running ProGuard](#TOC-Running-ProGuard)
        * [3.4.5 Shrinking Resources](#TOC-Shrinking-Resources)
* [4 Dependencies, Android Libraries and Multi-project setup](#TOC-Dependencies-Android-Libraries-and-Multi-project-setup)
    * [4.1 Dependencies on binary packages](#TOC-Dependencies-on-binary-packages)
        * [4.1.1 Local packages](#TOC-Local-packages)
        * [4.1.2 Remote artifacts](#TOC-Remote-artifacts)
    * [4.2 Multi project setup](#TOC-Multi-project-setup)
    * [4.3 Library projects](#TOC-Library-projects)
        * [4.3.1 Creating a Library Project](#TOC-Creating-a-Library-Project)
        * [4.3.2 Differences between a Project and a Library Project](#TOC-Differences-between-a-Project-and-a-Library-Project)
        * [4.3.3 Referencing a Library](#TOC-Referencing-a-Library)
        * [4.3.4 Library Publication](#TOC-Library-Publication)
* [5 Testing](#TOC-Testing)
    * [5.1 Basics and Configuration](#TOC-Basics-and-Configuration)
    * [5.2 Running tests](#TOC-Running-tests)
    * [5.3 Testing Android Libraries](#TOC-Testing-Android-Libraries)
    * [5.4 Test reports](#TOC-Test-reports)
        * [5.4.1 Single projects](#TOC-Single-projects)
        * [5.4.2 Multi-projects reports](#TOC-Multi-projects-reports)
    * [5.5 Lint support](#TOC-Lint-support)
* [6 Build Variants](#TOC-Build-Variants)
    * [6.1 Product flavors](#TOC-Product-flavors)
    * [6.2 Build Type + Product Flavor = Build Variant](#TOC-Build-Type-Product-Flavor-Build-Variant)
    * [6.3 Product Flavor Configuration](#TOC-Product-Flavor-Configuration)
    * [6.4 Sourcesets and Dependencies](#TOC-Sourcesets-and-Dependencies)
    * [6.5 Building and Tasks](#TOC-Building-and-Tasks)
    * [6.6 Testing](#TOC-Testing1)
    * [6.7 Multi-flavor variants](#TOC-Multi-flavor-variants)
* [7 Advanced Build Customization](#TOC-Advanced-Build-Customization)
    * [7.1 Build options](#TOC-Build-options)
        * [7.1.1 Java Compilation options](#TOC-Java-Compilation-options)
        * [7.1.2 aapt options](#TOC-aapt-options)
        * [7.1.3 dex options](#TOC-dex-options)
    * [7.2 Manipulating tasks](#TOC-Manipulating-tasks)
    * [7.3 BuildType and Product Flavor property reference](#TOC-BuildType-and-Product-Flavor-property-reference)
    * [7.4 Using sourceCompatibility 1.7](#TOC-Using-sourceCompatibility-1.7)

## 1 介绍

本文档适用于Gradle plugin 0.9版本，所以可能和我们1.0之前介绍的老版本有所不同。

### 1.1 新构建系统的目标

新构建系统的目标是：

* 可以很容易的重用代码和资源
* 可以很容易的创建应用的衍生版本，所以不管你是创建多个apk，还是不同功能的应用都很方便
* 可以很容易的配置、扩展以及自定义构建过程
* 和IDE无缝整合

### 1.2 Gradle是什么

Gradle是一个非常优秀的构建系统工具，允许你通过插件的方式创建自定义的构建逻辑

Gradle的以下特性让我们选择了它：

* 用过领域专用语言(DSL)描述和控制构建逻辑
* 构建文件基于Groovy，并且可以组合使用各种定义的元素，然后通过代码来控制这些DSL达到定制逻辑的目的
* 内建的基于Maven或者Ivy的依赖管理
* 使用非常灵活，Gradle不会强制实现的方式，你可以使用最佳实践
* 插件能提供DSL以及API为构建文件使用
* 良好的工具API以供IDE集成

## 2 要求

* Gradle 1.10或者1.11或者1.12，并且使用0.11.1版本的插件
* SDK with Build Tools要求19.0.0，有些功能可能需要更新的版本

## 3 基础项目

一个Gradle工程是通过名字叫 *build.gradle* 的文件描述其构建过程的，该文字位于工程的根目录下。

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

### 3.3 构建任务

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

不做任何改变两次运行 **build** ，Gradle会报告所有任务已经处于UP-TO-DATE状态，这意味着没有什么可做的。这使得任务之间可以正确的相互依赖，又不会导致其他不需要的操作执行。s

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

### 3.4 自定义构建

Android plugin提供了大量的DSL能够让你直接基于构建系统定制很多事情。

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

#### 3.4.2 Build Types

#### 3.4.3 Signing Configurations

#### 3.4.4 Running ProGuard

#### 3.4.5 Shrinking Resources

## 4 Dependencies, Android Libraries and Multi-project setup

### 4.1 Dependencies on binary packages

#### 4.1.1 Local packages

#### 4.1.2 Remote artifacts

### 4.2 Multi project setup

### 4.3 Library projects

#### 4.3.1 Creating a Library Project

#### 4.3.2 Differences between a Project and a Library Project

#### 4.3.3 Referencing a Library

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