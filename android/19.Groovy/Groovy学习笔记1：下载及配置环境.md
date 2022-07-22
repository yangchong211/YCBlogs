## 本文简单讲述了
* Mac/Linux系统Groovy的环境配置
* Windows系统Groovy的环境配置

我们首先进入Groovy的官网：

http://www.groovy-lang.org/download.html

## Mac下配置Groovy

看到官网提示，我们可以直接下载groovy sdk的压缩包，但是对于Mac或者Linux用户，官方更推荐我们使用
SDKMAN（这是什么奇怪的名字）开发者安装工具。

![website.png](https://upload-images.jianshu.io/upload_images/7293029-84cd3365aafe8d26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[SDKMAN的Github链接](https://github.com/sdkman/sdkman-cli)

> Install Software Development Kits for the JVM such as Java, Groovy, Scala, Kotlin and Ceylon. Ant, Gradle, Grails, Maven, SBT, Spark, Spring Boot, Vert.x and many others also supported.

SDKMan还是提供了很强大的功能，为Java，Groovy，Scala，Kotlin和Ceylon等JVM安装软件开发工具包。 Ant，Gradle，Grails，Maven，SBT，Spark，Spring Boot，Vert.x和许多其他支持。

然后我们往下拉，看到提供给Mac或者Linux开发者的步骤：

![sdkman.png](https://upload-images.jianshu.io/upload_images/7293029-5e19951ea5e27149.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们直接按照他给的命令行，进行安装即可，首先输入先下载sdkman：

> $ curl -s get.sdkman.io | bash

> $ source "$HOME/.sdkman/bin/sdkman-init.sh"

很快我们安装成功，个人原因忘记截图，sdkman下载好后，我们安装groovy：

> $ sdk install groovy

![image.png](https://upload-images.jianshu.io/upload_images/7293029-2a079b5906fa29e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装好后，我们直接运行以下命令确认Groovy安装成功：

> $ groovy -version

这之后，我们可以直接通过命令行打开Groovy的console，开启我们的Groovy之旅：

> $ groovyConsole

![helloworld!.png](https://upload-images.jianshu.io/upload_images/7293029-bf5d60ab95e6b563.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Windows配置Groovy

直接进入官网，点击【Windows installer】,下载对应的exe可执行文件，下载完毕后，一键傻瓜式安装即可

![win.png](https://upload-images.jianshu.io/upload_images/7293029-eff070aa1500b9a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

WindowsInstaller 会自动帮你配置好环境变量，因此我们只需要确认环境变量是否配置好后，打开cmd命令行，进行测试即可：

![确认环境变量是否正确配置](https://upload-images.jianshu.io/upload_images/7293029-b9513f74d0680af5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![测试groovy命令以及打开GroovyConsole](https://upload-images.jianshu.io/upload_images/7293029-0dece2ad30da76da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## Mac系统配置Groovy到Idea

在Idea中使用groovy很简单，直接新建一个groovy后缀的文件即可，但是必须先把Groovy的SDK配置给Idea：

![configure1.png](https://upload-images.jianshu.io/upload_images/7293029-51398533603da89d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击Configure Groovy SDK,找到Groovy的根目录并进行配置即可：


![configure2.png](https://upload-images.jianshu.io/upload_images/7293029-271641f0a127c298.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 SDKMan虽然下载安装Groovy比较简单，但是可能导致Groovy的根目录不太好找，实际目录在
>**Path: /Users/(用户名)/.sdkman/candidates/gradle/[(版本号)|current]**

隐藏目录需要 Cmd+Shift+. 控制显示。

配置好后，运行groovy文件即可：

![](https://upload-images.jianshu.io/upload_images/7293029-4ea6d9325a1081fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
