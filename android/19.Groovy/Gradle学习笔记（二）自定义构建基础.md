##  一、配置Manifest文件

我们可以直接通过构建文件而不是manifest文件来配置applicationId、minSdkVersion、targetSdkVersion、versionCode和versionName。另外，下面一些属性也是我们可以操控的：

* testApplicationId： 针对 instrument测试APK的applicationId  
* testInstrumentationRunner： JUnit测试运行器的名称，被用来运行测试
* signingConfig:  
* proguardFiles  

## 二、BuildConfig类

自SDK工具版本升级到17之后，构建工具都会生成一个叫作BuildConfig的类，该类包含一个按照构建类型设置值的DEBUG常量。如果有一部分代码你只想在debugging时期运行，比如logging，那么DEBUG会非常有用。你可以通过Gradle来扩展该文件，这样在debug和release时，就可以拥有不同的常量：

```groovy
   buildTypes {
        debug {
           buildConfigField  "String",  "API_ URL", "\"http:// test. example. com/ api\""              
           buildConfigField "boolean", "LOG_ HTTP_ CALLS", "true"
           minifyEnabled false
           proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        release {
            buildConfigField  "String",  "API_ URL", "\"http:// test. example. com/ api\""              
            buildConfigField "boolean", "LOG_ HTTP_ CALLS", "false"
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```

字符串值必须用转义双引号括起来，这样才会生成实际意义上的字符串。在添加buildConfigField行后，你可以在你的Java代码中使用BuildConfig.API_URL和BuildConfig.LOG_HTTP_ CALLS。

最近，Android工具开发小组还增加了类似的方式来配置资源值：


```groovy
   buildTypes {
        debug {
            resValue "string", "app_name", "ExampleDEBUG"
        }
        release {
           resValue "string", "app_name", "Example"
        }
    }
```
这里可以不加转义双引号，因为资源值通常默认被包装成value=""。

## 三、项目配置的范围

看这段代码：

```groovy
allprojects {
      apply plugin:'com.android.application'
      android {
        compileSdkVersion  27
        buildToolsVersion  "27.0.1"
      }
}
```

该段代码只有在你的所有模块都是Androidapp项目的时候才有效，因为你需要运用Android插件来获取Android特有的设置。

实现这种行为的更好方式是**在顶层构建文件中定义值，然后将它们应用到模块中**。Gradle允许在Project对象上添加额外属性。这意味着任何build.gradle文件都能定义额外的属性，添加额外属性需要通过ext代码块。

你可以给顶层构建文件添加一个含有自定义属性的ext代码块：

```
project.ext {
    android = [
            compileSdkVersion   : 27,
            buildToolsVersion   : "27.0.1",

            minSdkVersion       : 16,
            targetSdkVersion    : 27,
            versionCode         : 1,
            versionName         : "1.0.0"
    ]
}
```

该段代码使得模块层的构建文件可以使用rootProject来获取属性：
```
android {
    compileSdkVersion rootProject.ext.android["compileSdkVersion"]
    buildToolsVersion rootProject.ext.android["buildToolsVersion"]

    defaultConfig {
        minSdkVersion rootProject.ext.android["minSdkVersion"]
        targetSdkVersion rootProject.ext.android["targetSdkVersion"]
        versionCode rootProject.ext.android["versionCode"]
        versionName rootProject.ext.android["versionName"]
    }
}
```

## 四、项目属性

前面例子的ext代码块是定义额外属性的一种方式。你可以使用属性来动态定制构建过程。定义属性的方式有很多种，这里我们只介绍三种常用的：

* ext代码块  
* gradle.properties文件  
* -P命令行参数  

```
ext {
    local='Hello from build.gradle'
}

task printProperties  {
     println local // Local extra property
     println propertiesFile // Property from file
     if (project. hasProperty(' cmd')) { 
        println cmd // Command line property
     }
 }
```

gradle.properties文件的实现（相同文件夹下）：
```groovy
propertiesFile = Hello from gradle.properties
```

> 我们可以同时在顶层构建文件和模块构建文件中定义属性。如果一个模块定义了一个在顶层文件中早已存在的属性时，那么新属性将会直接覆盖原来的属性。

## 五、默认任务

如果没有指定任何任务而运行Gradle的话，其会运行help任务，它会打印一些如何使用Gradle工作的信息。help任务被设置为默认的任务，在每次运行没有明确指定任务的Gradle时，可以覆写默认的任务，添加一个常用的任务，甚至是多个任务。

在顶层build.gradle文件中加入一行，可用来指定默认的任务：

```
defaultTasks 'clean', 'assembleDebug'
```
现在，当你运行没有任何参数的GradleWrapper时，它会运行clean和assembleDebug。通过运行tasks任务和格式化输出，可以清晰地看到哪些任务被设置为默认任务。

```
$ gradlew tasks | grep "Default tasks" 

Default tasks: clean, assembleDebug
```
