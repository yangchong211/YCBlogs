# JakeWharton评价我的代码像是在打地鼠？

> 【标题党警告】本文主要内容为 **Gradle依赖替换规则详解**。

## RxJava3版本迁移的血泪史

不久前`RxJava`正式发布了`3.x`版本，作为`RxJava`的爱好者，笔者第一时间对个人项目进行了`3.x`版本的迁移。

迁移过程中遇到了一个小问题，那就是`RxAndroid`因为没有及时升级，因此内部还是依赖`2.x`版本的`RxJava`,这就导致项目的依赖发生了冲突。

笔者的解决方式非常简单，既然`RxAndroid`依赖了不合适的`RxJava`版本，我就把它的依赖排除掉就可以了:

```groovy
implementation ('io.reactivex.rxjava2:rxandroid:2.1.0')  {
  exclude group: 'io.reactivex.rxjava2', module: 'rxjava'
}
```

这样做之后，项目成功将`RxJava`迁移到了`3.x`版本，笔者还第一时间在 [这篇文章](https://juejin.im/post/5d1eeffe6fb9a07f0870b4e8#comment) 中进行了如下的评论：

![](https://raw.githubusercontent.com/qingmei2/qingmei2-blogs-art/master/android/gradle/1/image.i1uk7iv8l6n.png)

评论发出去了一段时间，并没有收到各路大神的批评，笔者便以为这就是 **正确的升级方式**，于是在 [RxAndroid](https://github.com/ReactiveX/RxAndroid) 的这个 [issue](https://github.com/ReactiveX/RxAndroid/issues/538) 中 **沾沾自喜** 地进行了分享：

![](https://raw.githubusercontent.com/qingmei2/qingmei2-blogs-art/master/android/gradle/1/image.yjajub8lb4.png)

没想到 [JakeWharton](https://github.com/JakeWharton) 竟然看到了我的回复，并且非常直接针对我提供的代码进行了点评：

![](https://raw.githubusercontent.com/qingmei2/qingmei2-blogs-art/master/android/gradle/1/image.l8dj3zojb39.png)

翻译过来的意思就是：

> 长远来看，制定一个 **替换规则** 远比通过`exclude`这种类似 **打地鼠** 的方式要好得多。

收到男神的回复令我受宠若惊，但我更迫切需要了解我的代码问题出在了哪里—— **我一直认为我的代码就是正确的处理方案，但事实却证明了我的无知**。

我翻阅了对应的`Gradle`文档，`Gradle`中提供了对应的 **依赖替换规则**，而我之前一直没有了解过它，这也正是本文的主要内容。

## 依赖替换规则

依赖替换规则的适用场景分为以下几种：

* 1.根据某些条件对依赖进行替换；
* 2.将本地依赖替换为外部依赖；
* 3.将外部依赖替换为本地依赖；

我们先解释一下 **外部依赖** 和 **本地依赖** 是什么。

### 外部依赖

**外部依赖**，顾名思义，就是从远程仓库拉取的依赖，也被称为常用的 **三方库**：

```groovy
// 从远程仓库拉取的开源代码库
implementation 'com.facebook.stetho:stetho:1.5.1'
implementation 'io.reactivex.rxjava3:rxjava:3.0.0-RC0'
```

### 本地依赖

**本地依赖**，也就是我们项目中常见的`module`,按照**《阿里Java开发手册》**中来描述，也叫做 **一方库**：

```groovy
implementation project(':library')
```

好的，现在我们了解了这两个基本概念，问题来了：

## 知道这些有什么用？

有同学肯定会有这个困惑，这些概念我虽然都了解了，但实际开发过程中我并没有用到这些， **项目依然稳定的迭代和运行**，那学习这些东西有什么用呢？

这些规则真的很有用，在实际开发过程中，我们肯定会遇到一些问题，这些问题我们通过`baidu`或者`google`的方式绕了过去，但是这真的解决了吗？

比如说 **依赖冲突**。

### 1.根据某些条件对依赖进行替换

举个例子，很多UI三方库都会依赖`RecyclerView`，但这么多的依赖库，我们不可避免遇到版本不同导致依赖冲突的情况，一般情况下，我们是这么解决的：

```groovy
// 将RecyclerView的依赖从这个三方库中排除掉
implementation "xxx.xxx:xxx:1.0.0",{
    exclude group: 'com.android.support', module: 'recyclerview-v7'
}
```

将`RecyclerView`的依赖从这个三方库中排除掉，令其使用项目本身的`RecyclerView`版本，这样项目就可以正常运行了，看起来并没有什么问题。

[JakeWharton](https://github.com/JakeWharton) 非常敏锐地点出了问题的所在——试想，如果项目的依赖比较复杂，也许我们要面对的将是这样的依赖配置：

```gradle
implementation "libraryA:xxx:1.0.0",{
    exclude group: 'com.android.support', module: 'recyclerview-v7'
}
implementation "libraryB:xxx:2.2.0",{
    exclude group: 'com.android.support', module: 'recyclerview-v7'
}
implementation "libraryC:xxx:0.0.8",{
    exclude group: 'com.android.support', module: 'recyclerview-v7'
}
```

我们需要将每个依赖了`RecyclerView`的三方库都通过`exclude`的方式移除掉本身对应的依赖，这种缝缝补补式地乱堵，不正是在 **打地鼠** 么。

针对类似这种情况，我们可以在`gradle`的构建过程中强制指定依赖的版本，以笔者的项目为例，我们针对`RxJava`的版本依赖进行了统一：

![](https://raw.githubusercontent.com/qingmei2/qingmei2-blogs-art/master/android/gradle/1/image.4xabn5zb26i.png)

现在，项目中所有`RxJava`相关的依赖，在构建过程中版本都统一使用了`3.0.0-RC0`，这样就 **避免了依赖冲突**，开发者再也不需要针对每一个有`RxJava`依赖的三方库进行额外的`exclude`了。

### 2.本地依赖替换为外部依赖

本地依赖替换为外部依赖，最经典的场景就是`SDK`的发布测试，如果您有过开源项目的经历，对此一定不会陌生。

以笔者开源的 [RxImagePicker](https://github.com/qingmei2/RxImagePicker) 为例，日常开发过程中，`sample`代码依赖本地的`module`；新版本发布后，笔者的UI测试代码便需要通过依赖`jcenter`远程仓库的最新代码。

这种情况下，通过`dependencySubstitution`便可以非常方便对这两种场景进行切换：

![](https://raw.githubusercontent.com/qingmei2/qingmei2-blogs-art/master/android/gradle/1/image.vp0v05kdr9g.png)

`useRemote`只是定义在`build.gradle`文件中的一个变量，作为切换开发-测试环境的开关：

```groovy
final boolean useRemote = true  
```

当`useRemote`值为`true`时，`sample`依赖远程仓库，当值为`false`时，`sample`依赖本地`module`。

看起来代码量反而增加了，实际上，随着项目复杂度的提升，这种全局的配置优点显而易见。

### 3.将外部依赖替换为本地依赖

该规则和2非常相似，只不过将依赖替换的双方调换了而已，下面是官方的示例代码：

```groovy
configurations.all {
    resolutionStrategy.dependencySubstitution {
        substitute module("org.utils:api") because "we work with the unreleased development version" with project(":api")
        substitute module("org.utils:util:2.5") with project(":util")
    }
}
```

## 最终的迁移方案？

故事的最后，笔者的解决方案如下：

![](https://raw.githubusercontent.com/qingmei2/qingmei2-blogs-art/master/android/gradle/1/image.eh8jzs6e7p4.png)

* 1.因为`group`不同，所以需要先将`2.x`的`rxjava`全局`exclude`掉；
* 2.将所有`3.x`的`rxjava`的依赖版本都统一（文中是`3.0.0-RC0`）；

笔者并不知道这种方式是否就是 [JakeWharton](https://github.com/JakeWharton) 描述的解决方案，但相比较之前而言效果确实更好，如果有更好的依赖管理方案，诚挚希望您能在评论区中进行分享。

## 感受

`GitHub`确实是一个神奇的东西，它让我避免固步自封，毕竟世界上最顶尖的开发者们都聚焦于此，在他们眼里，你的代码永远都有着非常广阔的进步空间。

发现自己的短板不是坏事，它可以督促我不断去尝试自我超越，就像我常年放在文章末尾的那句话一样，**万一哪天我进步了呢？**

## 关于我

Hello，我是[却把清梅嗅](https://github.com/qingmei2)，如果您觉得文章对您有价值，欢迎 ❤️，也欢迎关注我的[个人博客](https://juejin.im/user/588555ff1b69e600591e8462)或者[Github](https://github.com/qingmei2)。

如果您觉得文章还差了那么点东西，也请通过**关注**督促我写出更好的文章——万一哪天我进步了呢？

* [我的Android学习体系](https://github.com/qingmei2/android-programming-profile)
* [关于文章纠错](https://github.com/qingmei2/Programming-life/blob/master/error_collection.md)
* [关于知识付费](https://github.com/qingmei2/Programming-life/blob/master/appreciation.md)
