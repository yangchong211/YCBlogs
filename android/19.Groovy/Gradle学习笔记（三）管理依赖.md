## 概述

依赖管理是Gradle最耀眼的特点之一。最佳情况下，你需要做的仅仅是在构建文件中添加一行代码，Gradle将会从远程仓库下载依赖，确保你的项目能够使用依赖中的类。

Gradle甚至可以做得更多。如果你的项目中有一个依赖，并且其有自己的依赖，那么Gradle将会处理并解决这些问题。这些依赖中的依赖，被称之为**传递依赖**。

## 一、依赖仓库

一个依赖仓库可以被看作是文件的集合。Gradle默认情况下没有为你的项目定义任何依赖仓库，所以你需要在repositories代码块中添加它们。如果使用AndroidStudio，那么它会为你自动完成。如下所示：

```
repositories {
        jcenter()
        maven { url "https://jitpack.io" }
        google()
 }
```

一个依赖通常是由三种元素定义的：group、name和version。group指定了创建该依赖库的组织，通常是反向域名。name是依赖库的唯一标识。version指定了需要使用依赖库的版本号。使用这三个元素，就可以在dependencies代码块中声明一个依赖了：

```
compile 'io.reactivex.rxjava2:rxjava:2.1.8，
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
```

## 二、本地依赖

### 1、文件依赖

你可以使用Gradle提供的file方法来添加JAR文件作为一个依赖，如下所示：

```
dependencies {
    compile files('libs/ domoarigato.jar ')
}
```
当你有很多JAR文件时，这种方式会变得异常烦琐，一次添加一个完整的文件夹可能会更容易些。

默认情况下，新建的Android项目会有一个libs文件夹，其会被声明为依赖使用的文件夹。一个过滤器可以保证只有JAR文件会被依赖，而不是简单地依赖文件夹中的所有文件：

```
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
}
```

### 2、NDK依赖

Android插件默认支持原生依赖库，你所需要做的就是在模块层创建一个jniLibs文件夹，然后为每个平台创建子文件夹，将.so文件放在适当的文件夹中。

```
android {
     sourceSets.main {
        jniLibs.srcDir 'src/main/libs'
     }
 }
```

### 3、依赖Library

构建Library需要Androidy Library插件：

```
apply plugin: 'com.android.library'
```

在应用中包含依赖项目的方式有两种。一种是在项目中当作一个模块，另一种是创建一个可在多个应用中复用的.aar文件。

如果在项目中创建了一个模块作为依赖项目，那么你需要在settings.gradle中添加该模块，在应用模块中将它作为依赖：

```
include ':sample', ':rximagepicker'
```

同时，你还需要在你的module的dependences模块中添加对应依赖：

```
dependencies {
    implementation project(':rximagepicker')
}
```

## 三、依赖的概念（Compile）

Gradle将多个依赖添加至配置，并将其命名为集文件。下面是一个Android应用或依赖库的标准配置：

* compile 
* apk 
* provided 
* testCompile 
* androidTestCompile

compile是默认的配置，在编译主应用时包含所有的依赖。该配置不仅会将依赖添加至类路径，还会生成对应的APK。

如果依赖使用apk配置，则该依赖只会被打包到APK，而不会添加到编译类路径。

provided配置则完全相反，其依赖不会被打包进APK。这两个配置只适用于JAR依赖。如果试图在依赖项目中添加它们，那么将会导致错误。

最后，testCompile和androidTestCompile配置会添加用于测试的额外依赖库。在运行测试相关的任务时，这些配置会被使用，并且在添加如JUnit或Espresso测试框架时，特别有用。如果你只希望在测试APK时使用这些框架，那么就不会生产APK。


## 参考

《Gradle For Android 中文版》
