## 一、构建类型

你可以在buildTypes代码块中定义构建类型。下面是AndroidStudio创建的构建文件的标准buildTypes代码块：

```groovy
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```

在你的项目中，release构建类型不是被创建的唯一构建类型。默认情况下，每个模块都有一个debug构建类型。其被设置为默认构建类型，但是你可以通过将其包含至buildTypes代码块，然后覆写任何你想改变的属性配置。

### 1、创建构建类型

当默认的设置不够用时，我们可以很容易地创建自定义构建类型。新的构建类型只需在buildTypes代码块中新增一个对象即可。

```groovy
android {
  buildTypes {
        staging {
            versionNameSuffix  "-staging"
            buildConfigFiled "String","API_URL","https://xxx.xxx.xx"
        }
    }
}
```
staging构建类型针对applicationID定义了一个新的后缀，使其和debug以及release版本的applicationID不一样。假设你的构建类型是默认的，并且添加了staging构建类型，那么不同构建类型的applicationID应该像下面这样：

* debug      com.package
* release    com.package
* staging    com.package.staging    

这意味着你将能够在相同色设备上同时安装staging版本和release版本，而不发生任何冲突。staging构建类型也有版本名后缀，其在相同设备上区分多个应用版本时非常重要。

在创建一个新的构建类型时，还可以用另一个构建类型的属性来初始化该构建类型：

```groovy
buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        staging.initWith(buildTypes.debug)
        staging {
            debuggable = false
        }
}
```
initWith()方法创建了一个新的构建类型，并且复制了一个已经存在的构建类型的所有属性到新的构建类型中。我们可以通过在新的构建类型对象中简单地定义它们来覆写属性或定义额外的属性。

### 2、依赖

Gradle自动为每个构建类型创建新的依赖配置。如果你只想在debug构建中添加一个appcompat-v7。那么，你可以这么做：

```
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    debugCompile 'com.android.support:appcompat-v7:25.3.1'
}
```
你可以以这种方式结合任何构建配置的构建类型，来让依赖变得更加具体。

## 二、 product flavor

与被用来配置相同App或library的不同构建类型相反，productflavor被用来创建不同的版本。典型的例子是一个应用有免费版和付费版。

如果你不确定是否需要一个新的构建类型或新的productflavor，那么你应该问自己，是否想要创建一个供内部使用的应用，或一个会发布到GooglePlay的新APK。如果你需要一个全新的App，独立于你已有的应用发布，那么productflavor就是你需要的，否则，你应该坚持使用构建类型。

### 1、创建product flavor

```
productFlavors { 

    red {
        manifestPlaceholders = [
                APP_NAME           : "@string/app_name"]
    }
    blue {
        manifestPlaceholders = [
                APP_NAME           : "@string/app_name_dev"]
    }
}
```
### 2、源集

和构建类型类似，productflavor也可以拥有它们自己的源集目录。为一个特殊的flavor创建一个文件夹就和创建一个有flavor名称的文件夹一样简单。你甚至可以为一个特定构建类型和flavor的结合体创建一个文件夹。该文件夹的名称将是flavor名称＋构建类型的名称。例如，如果你想让blueflavor的release版本有一个不同的应用图标，那么文件夹将会被叫作blueRelease。

合并文件夹的组成将比构建类型文件夹和productflavor文件夹拥有更高优先级。

### 3、多维度

在某些情况下，你可能想要更进一步，创建productflavor的结合体。例如，客户A和客户B在他们的App中都想要免费版和付费版，并且是基于相同的代码、不同的品牌。创建四种不同的flavor意味着需要像这样设置多个拷贝，所以这不是最佳做法。而使用flavor维度是结合flavor的有效方式，如下所示：

```
android { 
      flavorDimensions "color", "price" 
      productFlavors { 
            red { flavorDimension "color" } 
            blue { flavorDimension "color" } 
            free { flavorDimension "price" }
            paid { flavorDimension "price" }
       }
 }
```

当你为flavor添加了维度后，Gradle会希望你能够为每个flavor都添加维度。如果你忘了，那么你会收到一个带有错误接受的构建错误。

维度的顺序非常重要。当结合两个flavor时，它们可能定义了相同的属性或资源。在这种情况下，flavor维度数组的顺序就决定了哪个flavor配置将被覆盖。在上一个例子中，color维度覆盖了price维度。该顺序也决定了构建variant的名称。

假设默认的构建配置是debug和release构建类型，那么前面例子中定义的flavor将会生成下面这些构建variant：

* blueFreeDebug and blueFreeRelease
* bluePaidDebug and bluePaidRelease
* redFreeDebug and redFreeRelease
* redPaidDebug and redPaidRelease

### 3、构建 variant

如果没有productflavor，variant将只会包含构建类型。没有任何构建类型是不可能的，即便你自己不定义任何构建类型，Gradle的Android插件也会为你的App或library创建一个debug构建类型。

#### 3.1 Task

Gradle的Android插件将会为你配置的每一个构建variant创建任务。一个新的Android应用默认有debug和release两种构建类型，所以你可以用assembleDebug和assembleRelease来分别构建两个APKs，即用单个命令assemble来创建两个APKs。

一旦你开始添加flavor，那么整个全新的任务系列将会被创建，因为每个构建类型的任务会和每个productflavor相结合。

仅仅一个BuildType  + Product Flavor。你可以得到：

* assembleBlue：使用blue flavor配置和组装BlueRelease及BlueDebug。
* assembleDebug：使用debug构建配置类型，并为每个productflavor组装一个debug版本。
* assembleBlueDebug：用构建配置类型结合flavor配置，并且flavor设置将会覆盖构建类型设置。

#### 3.2 源集

构建variant，是一个构建类型和一个或多个productflavor的结合体，其可以有自己的源集文件夹。例如，由debug构建类型blueflavor和freeflavor创建的variant可以有其自己的源集src/blueFreeDebug/java/。其可以通过使用sourceSets代码块来覆盖文件夹的位置。

源集的引入额外增加了构建进程的复杂性。Gradle的Android插件在打包应用之前将main源集和构建类型源集合并在一起。此外，library项目也可以提供额外的资源，这些也需要被合并。这同样适用于manifest文件。

资源 和 manifest 的 优先顺序：

> BuildType >> Flavor >> Main >> Dependencies

如果资源在flavor和main源集中都有申明，那么flavor中的资源将被赋予更高的优先级。在这种情况下，flavor源集中的资源将会被打包。在library项目中申明的资源通常具有最低优先级。

> 这也就说明了，即使appModule和Library中同样拥有同名的资源，appModule会覆盖Library的资源


想了解更多，请阅读官方文档：

http://tools.android.com/tech-docs/new-build-system/user-guide/manifest-merger

#### 3.3 variant过滤器

在你的构建中，可以完全忽略某些variant。通过这种方式，你就可以通过assemble命令来加快构建所有variant的进程，并且你的任务列表不会被任何无须执行的任务污染。

variant过滤器也可确保在AndroidStudio的构建variant切换器中不会出现过滤的构建variant。

你可以在App或library根目录下的build.gradle文件使用以下代码来过滤variants：


```
android.variantFilter { variant -> 
    if(variant.buildType.name.equals('release')) {
        variant.getFlavors().each() { flavor -> 
            if(flavor.name.equals('blue')) { 
                variant.setIgnore(true);
            }
        }
    }
}
```

在本例中，我们首先检查了variant的构建类型中是否含有release，然后我们去除了所有含有该名称的Productflavor。当不使用flavor维度时，在flavor数组中将只含有一个Productflavor。一旦你开始使用flavor维度，那么flavor数组就将会持有和维度一样多的flavor。在示例脚本中，我们检查了blueproductflavor,并且告诉构建脚本忽略这一variant。

#### 3.4 签名配置

Android插件使用了一个通用keystore和一个已知密码自动创建了debug配置，所以就没有必要为该构建类型再创建一个签名配置了：

```
android { 
    signingConfigs { 
        staging.initWith(signingConfigs.debug)
        release { 
            storeFilefile("release.keystore")
            storePassword "secretpassword"
            keyAlias "gradleforandroid"
            keyPassword "secretpassword"
        }
    }
}
```

release配置使用storeFile来指定keystore文件的路径，之后定义了密钥别名和两个密码。

在定义签名配置之后，你需要将它们应用到你的构建类型或flavor中。构建类型和flavor都有一个叫作signingConfi的属性，如下所示：

```
buildTypes { 
        release { 
                signingConfigsigningConfigs.release
        }
}
```

该例使用了构建类型，但如果你想为每个你所创建的flavor使用不同的凭证，那么你就需要创建不同的签名配置，不过你可以以相同的方式来定义它们：


```
productFlavors {
     blue { 
            signingConfigsigningConfigs.release
     }
 }
```

使用签名配置这种方式会造成很多问题。当为flavor分配一个配置时，实际上它是覆盖了构建类型的签名配置。如果你不想这样，那么在使用flavor时，就应该为每个构建类型的每个flavor分配不同的密钥.

## 参考

本文大部分内容节选自 凯文·贝利格里姆斯(KevinPelgrims).

《GradleforAndroid中文版》电子工业出版社.Kindle版本.

仅供自己作为笔记参考，未经允许严禁转载及相关商业性用途！
