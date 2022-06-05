#### 问题汇总目录介绍
- 4.4.6 mWebView.scrollTo(0, 0)回顶部失效
- 4.4.7 部分手机监听滑动顶部或底部失效
- 4.4.9 webView背景设置为透明无效探索
- 4.5.0 如何屏蔽掉WebView中长按事件
- 4.5.1 WeView出现OOM影响主进程如何避免
- 4.5.2 WebView域控制不严格漏洞
- 4.5.3 下载文件时的路径穿越问题
- 4.5.4 WebView中http和https混合使用问题
- 4.5.5 调用系统EMAIL发送邮件崩溃
- 4.5.7 WebView访问部分网页崩溃问题
- 4.6.1 在web页面android软键盘覆盖问题
- 4.6.2 为什么打包之后JS调用出现失败
- 4.6.3 ViewPager里非首屏WebView点击事件不响应
- 4.6.4 怎么知道WebView是否存在滚动条
- 4.6.5 WebView被导航栏遮挡的问题




### 4.4.6 mWebView.scrollTo(0, 0)回顶部失效
- 思考一下，为何会失效



### 4.4.7 部分手机监听滑动顶部或底部失效
- 先来看一下如何监听webView滑动到顶部和底部的逻辑代码，可以说网上绝大多数的都是这样
    ```
    /**
     * 判断是否在顶部
     * @return                              true表示在顶部
     */
    private boolean isTop() {
        return mWebView.getScrollY() <= 0;
    }
    
    /**
     * 判断是否在底部
     * @return                              true表示在底部
     */
    private boolean isBottom() {
        //return  Math.abs(webContent - webNow)<1
        return mWebView.getHeight() + mWebView.getScrollY() >= mWebView.getContentHeight() * mWebView.getScale();
    }
    ```
- 出现问题，部分手机
    - 在部分设备上会有问题，这些安卓手机可以隐藏掉下面的返回键,home键和菜单键的.也可以显示.可能因为如此，webView就不知道它的底部具体在哪里了.Math.abs(webContent - webNow)的值有可能等于1甚至大于1。
- 建议添加下面逻辑
    - 要完整实现网页加载完毕，监听网页滚动到底部才触发某些事件的话，那就可以设个标识，在onPageFinished的时候设为true，然后判断if(isLoadFinish &&  scrollY !=0) 再回调自定义listener的onPageEnd，再加上加载完毕的标识为true，这样就可以实现网页加载完毕后，滚动到底部的监听了。






### 4.4.9 webView背景设置为透明无效探索
- webView是一个使用方便、功能强大的控件，但由于webView的背景颜色默认是白色，在一些场合下会显得很突兀（比如背景是黑色，app中的夜间模式）。
- 首先看一下，网上的解决方案
    ```
    android:layerType="software"（没效果）
    mWebView.setBackgroundColor(0);（没效果）
    mWebView.setBackgroundDrawable(R.color.transparent);（没效果）
    ```
- 最后解决方案如下所示
    - 首先检查配置文件里application设置android:hardwareAccelerated=”false”，自己尝试后必须这样设置才行；
    - 在loadUrl后设置mWebView.setBackgroundColor(0);mWebView.getBackground().setAlpha(1); // 设置填充透明度 范围：0-255
    - 检查xml布局文件里的WebView的父层布局，也要设置背景为透明的；（之前也因为这个问题没发现，绕了很大一个圈）



### 4.5.0 如何屏蔽掉WebView中长按事件
- webView长按时将会调用系统的复制控件，具体可以看一下案例。案例中提到，你可以自定义长按逻辑，也可以屏蔽长按事件。具体屏蔽逻辑该如何操作呢？
    ```
    mWebView.setOnLongClickListener(new OnLongClickListener() {  
        @Override 
        public boolean onLongClick(View v) {  
          return true;  
        }  
    });
    ```


### 4.5.1 WeView出现OOM影响主进程如何避免
- WeView出现OOM，我在实际开发中没有遇到，倒是有点可惜。这个是看网上的，后期有待求证……
- 问题描述：由于WebView默认运行在应用进程中，如果WebView加载的数据过大（例如加载大图片），就可能导致OOM问题，从而影响应用主进程。
- 解决方案：为了避免WebView影响主进程，可以尝试将WebView所在的Activity运行在独立进程中。这样即使WebView出现了OOM问题，应用主进程也不会受到影响。具体做法也很简单，只要在AndroidManifest文件中为相应的Activity设置process属性即可。为Activity设置了process属性，意思就是让这个Activity运行在名为:remote的私有进程中。
- 需要注意，这种方式可能会有进程通信方面的问题，因此需要根据实际情况决定是否需要使用。



### 4.5.2 WebView域控制不严格漏洞
- 由于应用中的WebView没有对file:///类型的url做限制，可能导致外部攻击者利用Javascript代码读取本地隐私数据。
- 解决方案：
    - 1.如果WebView不需要使用file协议，直接禁用所有与file协议相关的功能即可。需要注意，即使禁用了File协议，也不影响对assets和resources资源的加载。它们的url格式分别为：file:///android_asset、file:///android_res。
    ```
    webSettings.setAllowFileAccess(false);
    webSettings.setAllowFileAccessFromFileURLs(false);
    webSettings.setAllowUniversalAccessFromFileURLs(false);
    ```
    - 2.如果WebView需要使用file协议，则应该禁用file协议的Javascript功能。具体方法为：在调用loadUrl方法前，以及在shouldOverrideUrlLoading方法中判断url的scheme是否为file。如果是file协议，就禁用Javascript，否则启用Javascript。
    ```
    //WebSettings
    webSettings.setAllowFileAccess(true);
    webSettings.setAllowFileAccessFromFileURLs(false);
    webSettings.setAllowUniversalAccessFromFileURLs(false);
     
    //调用loadUrl前
    if("file".equals(Uri.parse(url).getScheme())){//判断是否为file协议
        webView.getSettings().setJavaScriptEnabled(false);
    }else{
        webView.getSettings().setJavaScriptEnabled(true);
    }
    webView.loadUrl(url);
     
    //WebViewClient中做的操作
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
        if("file".equals(request.getUrl().getScheme())){//判断是否为file协议
            view.getSettings().setJavaScriptEnabled(false);
        }else{
            view.getSettings().setJavaScriptEnabled(true);
        }
        return false;
    }
    
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if("file".equals(Uri.parse(url).getScheme())){//判断是否为file协议
            view.getSettings().setJavaScriptEnabled(false);
        }else{
            view.getSettings().setJavaScriptEnabled(true);
        }
        return false;
    }
    ```


### 4.5.3 下载文件时的路径穿越问题
- 下载文件时，如果文件名中包含“../”这样的字符，并且WebView并未对文件名进行过滤，就会出现文件路径穿越问题。攻击者可以借助这种方式将可执行文件写入一些特定的位置。
- 解决方案：在下载文件时对文件名进行判断，过滤“../”这样的字符。


### 4.5.4 WebView中http和https混合使用问题
- 在Android 5.0及以上，WebView可能在加载混合使用http和https的网页时出现异常。比如在一个https的安全网页中加载使用http协议的资源将会失败。
- 解决方案：在Android 5.0后利用WebSettings设置WebView支持http和https混合内容模式。
    ```
    if(Build.VERSION.SDK_INT>=21){
        //方式1
        webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_COMPATIBILITY_MODE);
    }
    
    //或者
    if(Build.VERSION.SDK_INT>=21){
        webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
    }
    ```
- 需要注意，MIXED_CONTENT_ALWAYS_ALLOW这个模式是不安全的，建议先使用MIXED_CONTENT_COMPATIBILITY_MODE模式。这个模式会尝试以安全的方式加载部分http资源，另一部分http资源则不会被加载。资源是否能被加载的判断依据可能会随着版本的不同而改变，因此需要根据实际情况决定是否采用这一模式。


### 4.5.5 调用系统EMAIL发送邮件崩溃
- 崩溃日志：ActivityNotFoundException: No Activity found to handle Intent { act=android.intent.action.SENDTO dat=mailto:xxxxxxxxxxxx@xxx.xxx }
- 在调用系统EMAIL发从邮件时，如果手机没有能接受SENDTO和mailto的应用，将会出现如下崩溃，特别是一些国行手机，这些手机里面没有安卓原生GMAIL。


### 4.5.7 WebView组件访问部分网页崩溃问题
- 在测试WebView组件时发现总是出现崩溃现像：提示：ERR_CLEARTEXT_NOT_PERMITTED
- 当时以为是权限问题，查找自己的AndroidManifest文件发现已经申请INTERNET权限了。看了网上的一些大佬的文章才知道，原来由于 Android P （9.0）限制了明文流量的网络请求，非加密的流量请求都会被系统禁止掉，所以如果访问没有https协议的网站默认不不可以访问的。
- 解决方案如下所示
    - 1.只访问带有https协议的网站。
    - 2.在AndroidManifest文件设置明文通信属性。
    ```
      ...
      <!--权限-->
    <uses-permission android:name="android.permission.INTERNET"/>
        <application
           ...
           <!--默认为false，即不允许未加密的网络流量通信-->
            android:cleartextTrafficPermitted="true"
          ...
    ```
    - 还有另一种操作，如下所示
    ```
    //在res文件夹下添加xml文件夹下添加network_security_config.xml文件
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config xmlns:android="http://schemas.android.com/apk/res/android">
        <debug-overrides>
            <trust-anchors>
                <!-- Trust user added CAs while debuggable only -->
                <certificates src="user" />
            </trust-anchors>
        </debug-overrides>
        <base-config cleartextTrafficPermitted="true" />
    </network-security-config>
    
    
    //在AndroidManifest.xml文件中添加属性
    //在application 中 添加android:networkSecurityConfig="@xml/network_security_config"
    
    <application
        android:name=".base.BaseApplication"
        android:networkSecurityConfig="@xml/network_security_config"
        android:theme="@style/AppTheme">
    ```




### 4.6.1 在web页面android软键盘覆盖问题
- 遇到问题
    - 软键盘是由WebView中的网页元素所触发弹出的，在web页面android软键盘覆盖问题
- 最常见的解决办法
    - 在AndroidManifest文件中对activity设置：android:windowSoftInputMode的值adjustPan或者adjustResize即可
    - adjustPan是把整个界面向上平移，使输入框露出，不会改变界面的布局；
    - adjustResize则是重新计算弹出软键盘之后的界面大小，相当于是用更少的界面区域去显示内容，输入框一般自然也就在内了。
- 最简单解决办法
    - 出现坑的条件是：带有WebView的activity使用了全屏模式或者adjustPan模式。
    - 如果activity中有WebView，就不要使用全屏模式，并且把它的windowSoftInputMode值设为adjustResize就好了
- 如果WebView是全屏模式
     * 1.找到activity的根View，也就是根view，然后获取到第一个Child，也就是setContentView这个布局
     * 2.设置一个Listener监听View树变化，
     * 3.界面变化之后，获取"可用高度"
     * 4.最后一步，重设高度
- 具体看lib中的AndroidBug5497Workaround类代码
    - [AndroidBug5497Workaround](https://github.com/yangchong211/YCWebView/blob/master/WebViewLib/src/main/java/com/ycbjie/webviewlib/tools/AndroidBug5497Workaround.java)


### 4.6.2 为什么打包之后JS调用出现失败
- 在proguard-rules.pro中添加混淆，避免被混淆
    ```
    # 该包下所有的类和类成员不混淆
    -keep class com.ycbjie.webviewlib.** {
        *;
    }
    -dontwarn com.ycbjie.webviewlib.**
    ```


### 4.6.3 ViewPager里非首屏WebView点击事件不响应
- 问题描述
    - 如果你的多个WebView是放在ViewPager里一个个加载出来的，那么就会遇到这样的问题。ViewPager首屏WebView的创建是在前台，点击时没有问题；而其他非首屏的WebView是在后台创建，滑动到它后点击页面点击事件不响应。
- 解决方案：
    - 这个问题的办法是继承WebView类，在子类覆盖onTouchEvent方法，填入如下代码：
    ```
    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onScrollChanged(getScrollX(), getScrollY(), getScrollX(), getScrollY());
        }
        return super.onTouchEvent(ev);
    }
    ```


### 4.6.4 怎么知道WebView是否存在滚动条
- 做类似上拉加载下一页这样的功能的时候，页面初始的时候需要知道当前WebView是否存在纵向滚动条，如果有则不加载下一页，如果没有则加载下一页直到其出现纵向滚动条。
- 继承WebView类，在子类添加下面的代码：
    ```
    public boolean existVerticalScrollbar () {
        return computeVerticalScrollRange() > computeVerticalScrollExtent();
    }
    ```
- computeVerticalScrollRange得到的是可滑动的最大高度，computeVerticalScrollExtent得到的是滚动把手自身的高，当不存在滚动条时，两者的值是相等的。当有滚动条时前者一定是大于后者的。



### 4.6.5 WebView被导航栏遮挡的问题
- 解决部分手机虚拟导航栏遮挡住页面webView底部
    ```
    //透明导航栏
    // getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
    ```
- 为什么会出现这种情况




### 4.9.9 掘金问题反馈记录
- 使用JsBridge遇到的坑
    - 由于JsBridge采用 json字符串，客户端传给前端数据中/进行了转义，导致前端收到数据后解析不出来。二，当前端给Native端发消息时，如果发送的消息频率过快，导致队列清空 shouldurl不回调，最终callback不回调，客户端也就收不到消息了。









