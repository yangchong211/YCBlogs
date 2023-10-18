#### 目录介绍
- 01.整体概述
    - 1.1 项目背景说明
    - 1.2 遇到问题记录
    - 1.3 基础概念介绍
    - 1.4 开发设计目标
- 02.状态栏基础介绍
    - 2.1 状态栏的发展过程
    - 2.2 透明状态栏和沉浸式
    - 2.3 theme主题对状态栏影响
    - 2.4 理解页面Window
- 03.状态栏常用API
    - 3.1 setSystemUiVisibility
    - 3.2 fitsSystemWindows
- 04.状态栏设置操作
    - 4.1 如何给状态栏着色
    - 4.2 设置全屏幕展示
    - 4.3 如何设置状态栏透明
    - 4.4 隐藏底部导航栏
    - 4.5 隐藏状态栏
- 05.状态栏封装设计
    - 5.1 封装库聚合的能力
- 06.状态栏遇到的坑
    - 6.1 状态栏不占位的问题
    - 6.2 状态栏文字颜色bug



### 01.整体概述
#### 1.1 项目背景说明


#### 1.2 遇到问题记录


#### 1.3 基础概念介绍


#### 1.4 开发设计目标



### 02.状态栏基础介绍
#### 2.1 状态栏的发展过程
- `android`的状态栏大致经历了以下几个阶段：
    - 在android4.4以下就不要想着对状态栏做什么文章了，现在app的适配一般也是在android4.4以上了。
    - 在android4.4—android5.0可以实现状态栏的变色，但是效果还不是很好，主要实现方式是通过FLAG_TRANSLUCENT_STATUS这个属性设置状态栏为透明并且为全屏模式，然后通过添加一个与StatusBar 一样大小的View，将View 设置为我们想要的颜色，从而来实现状态栏变色。
    - 在android5.0—android6.0系统才真正的支持状态栏变色，系统加入了一个重要的属性和方法 android:statusBarColor （对应方法为 setStatusBarColor），通过这个属性可以直接设置状态栏的颜色。
    - 在android6.0上主要就是添加了一个功能可以修改状态栏上内容和图标的颜色（黑色和白色）


#### 2.2 透明状态栏和沉浸式
- 什么是透明状态栏
    - 透明状态栏是指将状态栏设置为透明或者半透明。如果这时候将状态栏设置为透明，那么状态栏和手机页面看以来是一个整体，增大了屏幕的利用空间。
- 什么是沉浸式
    - 沉浸式是指，将页面的布局“沉浸”在状态栏下面，如果设置了沉浸式状态栏不透明，毫无疑问布局将会被状态栏遮挡。



#### 2.3 theme主题对状态栏影响
- 对于现在开发项目我们一般使用的是 `Theme.AppCompat.xxx`兼容包里面的主题
    ``` java
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorPrimary</item>
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:windowTranslucentNavigation">true</item>
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
    ```
- 那么这些属性是什么意思呢？
    - 分别代表toolbar的颜色、状态栏的颜色、输入法和`radioButton`等的颜色，但是需要注意的是：在5.0以下的版本，状态栏的颜色通过主题设置是没有用的，用以上的主题属性来设置状态栏颜色，在5.x以下是黑色，自己可以试试。
    - 后三个属性是来设置透明状态栏的，`android:windowTranslucentStatus`、`android:windowTranslucentNavigation`是4.4以后才有的属性，`android:statusBarColor`是5.0以后才有的属性


#### 2.4 理解页面Window
- 一般情况下，我们会对Activity，Fragment等设置状态栏。就拿常见的Activity来说，Activity是控制器，Window是实际窗口(包含状态栏 + decorView)



### 03.状态栏常用API
#### 3.1 setSystemUiVisibility
- 对于状态栏的控制是非常重要的，这个方法主要是用来设置系统ui的可见性以及和用户布局的位置关系。
    - 使用方式如下：getWindow().getDecorView().setSystemUiVisibility(flag);
- 下面着重介绍一下flag：
    - `SYSTEM_UI_FLAG_FULLSCREEN`(4.1+)：隐藏状态栏，手指在屏幕顶部往下拖动，状态栏会再次出现且不会消失，另外activity界面会重新调整大小，直观感觉就是activity高度有个变小的过程。
    - `SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`(4.1+):状态栏一直存在并且不会挤压activity高度，状态栏会覆盖在activity之上
    - `SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`(4.1+):配合SYSTEM_UI_FLAG_FULLSCREEN一起使用，效果使得状态栏出现的时候不会挤压activity高度，状态栏会覆盖在activity之上
    - `SYSTEM_UI_FLAG_HIDE_NAVIGATION`(4.0+):会使得虚拟导航栏隐藏，但是由于NavigationBar是非常重要的，因此只要有用户交互（例如点击一个 button），系统就会清除这个flag使NavigationBar就会再次出现，同时activity界面会被挤压 。
    - `SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`(4.1+)：效果使得导航栏不会挤压activity高度，导航栏会覆盖在activity之上。
    - `SYSTEM_UI_FLAG_IMMERSIVE`配合`SYSTEM_UI_FLAG_HIDE_NAVIGATION`一起使用，还记得之前使用SYSTEM_UI_FLAG_HIDE_NAVIGATION之后只要有用户交互，系统就会清除这个flag使NavigationBar就会再次出现，和SYSTEM_UI_FLAG_IMMERSIVE一起使用之后就会使SYSTEM_UI_FLAG_HIDE_NAVIGATION必须手指在屏幕底部往上拖动NavigationBar才会出现。
    - `SYSTEM_UI_FLAG_IMMERSIVE_STICKY`配合`View.SYSTEM_UI_FLAG_FULLSCREEN`和`View.SYSTEM_UI_FLAG_HIDE_NAVIGATION`一起使用，会使状态栏和导航栏以透明的形式出现，并且一段时间后自动消失。
- 这里注意一下，`setSystemUiVisibility`只能给DecorView设置吗？
    - 对的，只能给根布局设置该属性？为什么，这个方法的作用是：请求更改状态栏或其他屏幕/窗口装饰的可见性



#### 3.2 fitsSystemWindows
- 一些事情需要注意：
    - `fitsSystemWindows` 需要设置给根View——这个属性可以被设置给任意View，但是只有根View（content部分的根）外面才是SystemWindow，所以只有设置给根View才有用。
    - 其它padding将通通被覆盖。需要注意，如果你对一个View设置了android:fitsSystemWindows=“true”，那么你对该View设置的其他padding将通通无效。
- 看一句有疑惑的话
    - 使用系统提供的一些控件时`fitsSystemWindows`这个属性有时候并不仅仅是设置给最外层的根布局，他的子view也会添加这个属性，不是说这个属性只对根view才有效吗？
- 如何理解这句话
    - 这句话本身没问题，但是他的的前提是没有对view进行重写，在KITKAT及以下的版本，你的自定义View能够通过覆盖`fitsSystemWindows() : boolean`函数，来增加自定义行为。如果返回true，意味着你已经占据了整个Insets，如果返回false，意味着给其他的View依旧留有机会。所以当我们返回`false`时子`view`的`fitsSystemWindows` 就有效。
- 举个例子加深理解
    - `CoordinatorLayout`嵌套`AppBarLayout`嵌套`CollapsingToolbarLayout`嵌套`ImageView`，这是一个常见的效果，具体是什么我就不多介绍了，这里的每一个view都需要设置`fitsSystemWindows`为true，这样事件才能传到 `ImageView`中



### 04.状态栏设置操作
#### 4.1 如何给状态栏着色
- 给状态栏着色有两个方式
    - 第一种方式通过样式设置状态栏颜色；第二种方式通过代码设置状态栏颜色
- 通过代码设置状态栏颜色
    - 5.x以上的版本，直接通过`setStatusBarColor`设置状态栏颜色
    - 4.4版本唯一一个和5.x版本不一样的地方就是4.4没有改变状态栏颜色的api，只有改变透明度的api，那怎么解决这个问题呢？
        - 都知道，整个app展现在我们眼前的最外层的视图为DecorView，实质上是一个FrameLayout，状态栏是系统的窗口，覆盖在DecorView之上的，DecorView里面有我们的content，既然是这样，我们可以在DecorView里面加一个布局，覆盖在content之上，通过设置状态栏透明，改变这个View的颜色，这样就造成了改变状态栏颜色的假象。
- 通过样式设置状态栏颜色
    - 5.x以上的版本
    ``` java
    <style name="AppThemeTranslucent" parent="AppTheme">
        <item name="android:windowTranslucentStatus">false</item>
        <item name="android:windowTranslucentNavigation">true</item>
        <item name="android:statusBarColor">@color/colorAccent</item>
    </style>
    ```



#### 4.2 设置全屏幕展示
- 背景：有些内容在全屏模式下会让用户获得最佳体验，例如视频。既然是全屏幕展示，那么则需要隐藏状态栏，隐藏底部导航栏。
- 第一种方式：向后倾斜
    - 向后倾斜模式适用于用户不会与屏幕进行大量互动的全屏体验，例如在观看视频时。当用户希望调出系统栏时，只需点按屏幕上的任意位置即可。
- 第二种方式：沉浸模式
    - 沉浸模式适用于用户将与屏幕进行大量互动的应用。当用户需要调出系统栏时，他们可从隐藏系统栏的任一边滑动。
- 第三种方式：粘性沉浸模式
    - 在普通的沉浸模式中，只要用户从边缘滑动，系统就会负责显示系统栏。在粘性沉浸模式下，如果系统界面可见性发生变化，您无法收到回调。
    - 在粘性沉浸模式下，如果用户从隐藏了系统栏的边缘滑动，系统栏会显示出来，是半透明的，并且轻触手势会传递给应用，因此应用也会响应该手势。
- 关于什么时候设置全屏幕展示
    - 第一种，在onCreate设置，网络上大部分方案是这样的。但是隐藏状态栏后，用户手指滑动让状态栏出现，则可能会有视觉上bug
    - 第二种，监听页面状态栏变化，实现状态之间的无缝切换，界面控件的可见性与系统栏保持同步。当应用进入沉浸模式后，任何界面控件也应随系统栏一起隐藏，然后在系统界面重新出现时也再次显示出来。为此，可以在 View.OnSystemUiVisibilityChangeListener 以接收回调
    - 第三种，实现 onWindowFocusChanged()。 如果您获得窗口焦点，则可能需要再次隐藏系统栏。 



#### 4.3 如何设置状态栏透明
- 有两种方式设置状态栏透明
    - 第一种是通过改变主题的方式；第二种是通过代码方式的方式
- 通过改变主题的方式设置状态栏透明
    - 贴出value-v19、v21的主题参数
    ```
    //v19
    <style name="AppTheme1" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:windowTranslucentNavigation">true</item>
    </style>
    
    //v21
    <style name="AppTheme1" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:windowTranslucentNavigation">true</item>
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
    ```
- 通过代码方式设置状态栏透明
    - 针对4.4到5.0之间，只是window为透明，移除已经存在假状态栏则，并且取消它的Margin间距
    - 针对5.0以上，直接调用`setStatusBarColor`设置透明颜色。
- 注意一些问题
    - windowTranslucentStatus属性改变状态栏成半透明，并且布局网上顶，充满全屏
    - windowTranslucentNavigation属性改变底部导航栏颜色为半透明
    - statusBarColor 5.x才有设置状态栏颜色，4.x不是不能设置颜色的。



### 05.状态栏封装设计
#### 5.1 设置状态栏颜色





### 06.状态栏遇到的坑
#### 6.1 状态栏不占位的问题
- 思考一下为什么setContentView的布局会顶上去？
    - 设置了状态栏透明，那么内容 view 就占据状态栏的位置了。
- 先看一下activity的层级布局
    - statusBar状态栏是系统窗口，DecorView是视图的顶级窗口，继承自FrameLayout，他的子View一个是actionbar，和一个id为content的FrameLyout，这个content就是setContentView的那个布局
- 为什么设置了透明主题属性就会把contentView顶上去呢？
    - 如果设置了透明状态栏，decorView和content之间的margin为0，这时候content布局的大小就是整个屏幕的大小；如果没有设置，则margin为状态栏的高度加上actionbar的高度(如果有）。
- 有什么方法可以设置顶上去
    - 布局添加：android:fitsSystemWindows="true|false"，有没有觉得很熟悉。
    - 代码设置：window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
    - 或者theme主题设置：在theme的style中加入<item name="android:fitsSystemWindows">true</item>
- fitsSystemWindows 见名知意，意思是表示系统还是要占据状态栏的位置了，这样我们的状态栏和 ui 才能没有冲突



#### 6.2 状态栏文字颜色bug
- 这个是在 6.0 之后才提供的功能，说是可以修改颜色，其实也只是能把颜色改为暗颜色和亮颜色，说白了就是白色和黑色。




### 参考博客
- Android 刘海屏适配全攻略：https://www.jianshu.com/p/561f7241153b




### 02.android6.0状态栏内容不见
- Android 系统状态栏的字色和图标颜色为白色，当我的主题色或者图片接近白色或者为浅色的时候，状态栏上的内容就看不清了。 
- 这个问题在Android 6.0的时候得到了解决。Android 6.0 新添加了一个属性`SYSTEM_UI_FLAG_LIGHT_STATUS_BAR`，这个属性是干什么用的呢？
- 注意问题：
    - 为setSystemUiVisibility(int)方法添加的Flag,请求status bar 绘制模式，它可以兼容亮色背景的status bar 。
    - 要在设置了FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS flag ,同时清除了FLAG_TRANSLUCENT_STATUS flag 才会生效。
- 代码如下所示
    ```
    //先清除
    window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        //相当于在布局中设置android:fitsSystemWindows="true"，让contentView顶上去
        window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
    }
    ```





#### 参考案例
- https://github.com/gyf-dev/ImmersionBar












