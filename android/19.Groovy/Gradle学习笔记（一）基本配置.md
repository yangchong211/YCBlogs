## 简介

Gradle构建脚本的书写没有基于传统的XML文件，而是基于Groovy的领域专用语言（DSL）。Groovy是一种基于Java虚拟机的动态语言。Gradle团队认为，基于动态语言的DSL语言与Ant或者任何基于XML的构建系统相比，优势都十分显著。

## 一、Gradle基础

### 1、构建生命周期
一个Gradle的构建通常有如下三个阶段。

* 初始化：项目实例会在该阶段被创建。如果一个项目有多个模块，并且每一个模块都有其对应的build.gradle文件，那么就会创建多个项目实例。
* 配置：在该阶段，构建脚本会被执行，并为每个项目实例创建和配置任务。
* 执行：在该阶段，Gradle将决定哪个任务会被执行。哪些任务被执行取决于开始该次构建的参数配置和该Gradle文件的当前目录。

### 2、构建配置文件
Android的构建文件中，有一些元素是必需的：
```
buildscript {
    repositories {
        jcenter()
      }
    dependencies {
        classpath'com.android.tools.build:gradle:3.0.1'
    }
}
```

构建脚本代码块在Android构建工具上定义了一个依赖，就像Maven的artifact。这就是Android插件的来源，Android插件提供了构建和测试应用所需要的一切。每一个Android项目都应该申请该插件：

```
apply plugin: 'com.android.application'
```
> 如果你正在构建一个依赖库，那么你需要声明'com.android.library'，而不是‘com.android.application’。你不能在一个模块中同时使用它们，因为这会导致构建错误。一个模块可以是一个Android应用模块，或者是一个Android依赖模块，但不能二者都是。

### 3、运行基本构建任务

使用terminal或命令提示符，可以导航到项目根目录，运行带有tasks的GradleWrapper命令：

> $ gradlew tasks

这将打印出所有可用的任务列表。如果你添加了--all参数，那么你将获得每个任务对应依赖的详细介绍。

> $ gradlew assembleDebug

这个任务会为这个应用创建一个debug版本的APK。

除了assemble外，还有其他三个基本任务。

* Check：运行所有的检查，这通常意味着在一个连接的设备或模拟器上运行测试。
* Build：触发assemble和check。
* Clean：清除项目的输出。

## 二、认识Gradle文件
### 1、Settings.gradle文件
对于一个只包含一个Android应用的新项目来说，settings.gradle应该是这样的：

> Include ':app'

settings文件在初始化阶段被执行，并且定义了哪些模块应该包含在构建内。在本例中，app模块被包含在内。单模块项目并不一定需要setting文件，但是多模块项目必须要有setting文件，否则，Gradle不知道哪个模块应包含在构建内。

### 2、Project级别的build.gradle文件

默认情况下其包含如下两个代码块：
```
buildscript {           //实际构建配置代码块
    ext.kotlin_version = '1.2.10'
    repositories {      //依赖仓库，每个仓库意味着一系列的依赖包
        jcenter()          //JCenter是一个很有名的Maven库
        mavenCentral()
        maven { url 'https://maven.google.com' }
        google()
    }
    dependencies {    //配置构建过程中的依赖包
        classpath 'com.android.tools.build:gradle:3.0.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {          //声明那些需要被用于所有模块的属性，甚至可以在allprojects中创建task，这些任务最终被运用到所有Module
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
        google()
    }
}
```
请注意，只要你使用了allprojects，模块就会被耦合到项目。这意味着，其很可能在没有主项目构建文件的情况下，无法独立构建模块。最初，这看起来可能不是一个问题，但是如果你后面想要分离一个内部依赖库到自己的项目，那么你将需要重构你的构建文件。

### 3、Module级别的build.gradle文件

> Module层的build.gradle文件的属性只能应用在Androidapp模块，它可以覆盖Project层build.gradle文件的任何属性。

```
apply plugin: 'com.android.application'
android {
    compileSdkVersion 25
    buildToolsVersion '25.0.3'

    defaultConfig {
        applicationId "com.qingmei2.gradle.demo"
        minSdkVersion 14
        targetSdkVersion 22
        multiDexEnabled true
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    compile 'com.android.support:multidex:1.0.1'
}
```
记录以下三点：

#### 插件

第一行用到了Android应用插件，谷歌的Android工具团队负责Android插件的编写和维护，并提供构建、测试和打包Android应用以及依赖项目的所有任务。

#### Android

* compileSdkVersion(必须)：编译应用Android API的版本
* buildToolsVersion(必须):构建工具及编译器的版本号

* applicationId 
> 该属性覆盖了manifest文件中的packagename，但applicationId和packagename有一些不同。在Gradle被用作默认的Android构建系统之前，AndroidManifest.xml中的packagename有两个用途：作为一个应用的唯一标志，以及在R资源类中被用作包名。使用构建variants,Gradle可更容易地创建不同版本的应用。
* minSdkVersion  应用最小API级别
* targetSdkVersion 
> targetSdkVersion用于通知系统，该应用已经在某特定Android版本通过测试，从而操作系统不必启用任何向前兼容的行为。

#### Dependencies

定义依赖包


