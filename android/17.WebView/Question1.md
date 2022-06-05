#### 问题汇总目录介绍
- 4.0.0 WebView进化史介绍
- 4.0.1 提前初始化WebView必要性
- 4.0.2 x5加载office资源
- 4.0.3 WebView播放视频问题
- 4.0.4 无法获取webView的正确高度
- 4.0.5 使用scheme协议打开链接风险
- 4.0.6 如何处理加载错误
- 4.0.7 webView防止内存泄漏
- 4.0.8 关于js注入时机修改
- 4.0.9 视频/图片宽度超过屏幕
- 4.1.0 如何保证js安全性
- 4.1.1 如何代码开启硬件加速
- 4.1.2 WebView设置Cookie
- 4.1.3 开启硬件加速导致的闪烁问题
- 4.1.4 webView加载网页不显示图片
- 4.1.5 绕过证书校验漏洞


### 4.0.0 WebView进化史介绍
- 进化史如下所示
    - 从Android4.4系统开始，Chromium内核取代了Webkit内核。
    - 从Android5.0系统开始，WebView移植成了一个独立的apk，可以不依赖系统而独立存在和更新。
    - 从Android7.0 系统开始，如果用户手机里安装了 Chrome ， 系统优先选择 Chrome 为应用提供 WebView 渲染。
    - 从Android8.0系统开始，默认开启WebView多进程模式，即WebView运行在独立的沙盒进程中。


### 4.0.1 提前初始化WebView必要性
- 第一次打开Web面 ，使用WebView加载页面的时候特别慢，第二次打开就能明显的感觉到速度有提升，为什么？
    - 是因为在你第一次加载页面的时候 WebView 内核并没有初始化 ，所以在第一次加载页面的时候需要耗时去初始化WebView内核 。
    - 提前初始化WebView内核 ，例如如下把它放到了Application里面去初始化 , 在页面里可以直接使用该WebView，这种方法可以比较有效的减少WebView在App中的首次打开时间。当用户访问页面时，不需要初始化WebView的时间。
    - 但是这样也有不好的地方，额外的内存消耗。页面间跳转需要清空上一个页面的痕迹，更容易内存泄露。


### 4.0.2 x5加载office资源
- 关于加载word，pdf，xls等文档文件注意事项：Tbs不支持加载网络的文件，需要先把文件下载到本地，然后再加载出来
- 还有一点要注意，在onDestroy方法中调用此方法mTbsReaderView.onStop()，否则第二次打开无法浏览。更多可以看FileReaderView类代码！



### 4.0.3 WebView播放视频问题
- 1、此次的方案用到WebView，而且其中会有视频嵌套，在默认的WebView中直接播放视频会有问题， 而且不同的SDK版本情况还不一样，网上搜索了下解决方案，在此记录下. 
    ```
    webView.getSettings.setPluginState(PluginState.ON);
    webView.setWebChromeClient(new WebChromeClient());
    ```
- 2、然后在webView的Activity配置里面加上： android:hardwareAccelerated="true"
- 3、以上可以正常播放视频了，但是webView的页面都finish了居然还能听 到视频播放的声音， 于是又查了下发现webview的onResume方法可以继续播放，onPause可以暂停播放， 但是这两个方法都是在Added in API level 11添加的，所以需要用反射来完成。
- 4、停止播放：在页面的onPause方法中使用：webView.getClass().getMethod("onPause").invoke(webView, (Object[])null);
- 5、继续播放：在页面的onResume方法中使用：webView.getClass().getMethod("onResume").invoke(webView,(Object[])null);这样就可以控制视频的暂停和继续播放了。



### 4.0.4 无法获取webView的正确高度
- 偶发情况，获取不到webView的内容高度
    - 其中htmlString是一个HTML格式的字符串。
    ```
    webView.loadData(htmlString, "text/html", "utf-8");
    webView.setWebViewClient(new WebViewClient() {
        public void onPageFinished(WebView view, String url) {
            super.onPageFinished(view, url);
            Log.d("yc", view.getContentheight() + "");
        }
    });
    ```
    - 这是因为onPageFinished回调指的WebView已经完成从网络读取的字节数，这一点。在点onPageFinished被激发的页面可能还没有被解析。
- 第一种解决办法：提供onPageFinished（）一些延迟
    ```
    webView.setWebViewClient(new WebViewClient() {
        @Override
        public void onPageFinished(WebView view, String url) {
            super.onPageFinished(view, url);
            webView.postDelayed(new Runnable() {
                @Override
                public void run() {
                    int contentHeight = webView.getContentHeight();
                    int viewHeight = webView.getHeight();
                }
            }, 500);
        }
    });
    ```
- 第二种解决办法：使用js获取内容高度，具体可以看这篇文章：https://www.jianshu.com/p/ad22b2649fba



### 4.0.5 使用scheme协议打开链接风险
- 常见的用法是在APP获取到来自网页的数据后，重新生成一个intent，然后发送给别的组件使用这些数据。比如使用Webview相关的Activity来加载一个来自网页的url，如果此url来自url scheme中的参数，如：yc://ycbjie:8888/from?load_url=http://www.taobao.com。
    - 如果在APP中，没有检查获取到的load_url的值，攻击者可以构造钓鱼网站，诱导用户点击加载，就可以盗取用户信息。
    - 这个时候，别人非法篡改参数，于是将scheme协议改成yc://ycbjie:8888/from?load_url=http://www.doubi.com。这个时候点击进去即可进入钓鱼链接地址。
- 使用建议
    - APP中任何接收外部输入数据的地方都是潜在的攻击点，过滤检查来自网页的参数。
    - 不要通过网页传输敏感信息，有的网站为了引导已经登录的用户到APP上使用，会使用脚本动态的生成URL Scheme的参数，其中包括了用户名、密码或者登录态token等敏感信息，让用户打开APP直接就登录了。恶意应用也可以注册相同的URL Sechme来截取这些敏感信息。Android系统会让用户选择使用哪个应用打开链接，但是如果用户不注意，就会使用恶意应用打开，导致敏感信息泄露或者其他风险。
- 解决办法
    - 在内嵌的WebView中应该限制允许打开的WebView的域名，并设置运行访问的白名单。或者当用户打开外部链接前给用户强烈而明显的提示。具体操作可以看5.0.8 如何设置白名单操作方式。


### 4.0.6 如何处理加载错误(Http、SSL、Resource)
- 对于WebView加载一个网页过程中所产生的错误回调，大致有三种
    ```
    /**
     * 只有在主页面加载出现错误时，才会回调这个方法。这正是展示加载错误页面最合适的方法。
     * 然而，如果不管三七二十一直接展示错误页面的话，那很有可能会误判，给用户造成经常加载页面失败的错觉。
     * 由于不同的WebView实现可能不一样，所以我们首先需要排除几种误判的例子：
     *      1.加载失败的url跟WebView里的url不是同一个url，排除；
     *      2.errorCode=-1，表明是ERROR_UNKNOWN的错误，为了保证不误判，排除
     *      3failingUrl=null&errorCode=-12，由于错误的url是空而不是ERROR_BAD_URL，排除
     * @param webView                                           webView
     * @param errorCode                                         errorCode
     * @param description                                       description
     * @param failingUrl                                        failingUrl
     */
    @Override
    public void onReceivedError(WebView webView, int errorCode,
                                String description, String failingUrl) {
        super.onReceivedError(webView, errorCode, description, failingUrl);
        // -12 == EventHandle.ERROR_BAD_URL, a hide return code inside android.net.http package
        if ((failingUrl != null && !failingUrl.equals(webView.getUrl())
                && !failingUrl.equals(webView.getOriginalUrl())) /* not subresource error*/
                || (failingUrl == null && errorCode != -12) /*not bad url*/
                || errorCode == -1) { //当 errorCode = -1 且错误信息为 net::ERR_CACHE_MISS
            return;
        }
        if (!TextUtils.isEmpty(failingUrl)) {
            if (failingUrl.equals(webView.getUrl())) {
                //做自己的错误操作，比如自定义错误页面
            }
        }
    }

    /**
     * 只有在主页面加载出现错误时，才会回调这个方法。这正是展示加载错误页面最合适的方法。
     * 然而，如果不管三七二十一直接展示错误页面的话，那很有可能会误判，给用户造成经常加载页面失败的错觉。
     * 由于不同的WebView实现可能不一样，所以我们首先需要排除几种误判的例子：
     *      1.加载失败的url跟WebView里的url不是同一个url，排除；
     *      2.errorCode=-1，表明是ERROR_UNKNOWN的错误，为了保证不误判，排除
     *      3failingUrl=null&errorCode=-12，由于错误的url是空而不是ERROR_BAD_URL，排除
     * @param webView                                           webView
     * @param webResourceRequest                                webResourceRequest
     * @param webResourceError                                  webResourceError
     */
    @Override
    public void onReceivedError(WebView webView, WebResourceRequest webResourceRequest,
                                WebResourceError webResourceError) {
        super.onReceivedError(webView, webResourceRequest, webResourceError);
    }

    /**
     * 任何HTTP请求产生的错误都会回调这个方法，包括主页面的html文档请求，iframe、图片等资源请求。
     * 在这个回调中，由于混杂了很多请求，不适合用来展示加载错误的页面，而适合做监控报警。
     * 当某个URL，或者某个资源收到大量报警时，说明页面或资源可能存在问题，这时候可以让相关运营及时响应修改。
     * @param webView                                           webView
     * @param webResourceRequest                                webResourceRequest
     * @param webResourceResponse                               webResourceResponse
     */
    @Override
    public void onReceivedHttpError(WebView webView, WebResourceRequest webResourceRequest,
                                    WebResourceResponse webResourceResponse) {
        super.onReceivedHttpError(webView, webResourceRequest, webResourceResponse);
    }

    /**
     * 任何HTTPS请求，遇到SSL错误时都会回调这个方法。
     * 比较正确的做法是让用户选择是否信任这个网站，这时候可以弹出信任选择框供用户选择（大部分正规浏览器是这么做的）。
     * 有时候，针对自己的网站，可以让一些特定的网站，不管其证书是否存在问题，都让用户信任它。
     * 坑：有时候部分手机打开页面报错，绝招：让自己网站的所有二级域都是可信任的。
     * @param webView                                           webView
     * @param sslErrorHandler                                   sslErrorHandler
     * @param sslError                                          sslError
     */
    @Override
    public void onReceivedSslError(WebView webView, SslErrorHandler sslErrorHandler, SslError sslError) {
        super.onReceivedSslError(webView, sslErrorHandler, sslError);
        //判断网站是否是可信任的，与自己网站host作比较
        if (WebViewUtils.isYCHost(webView.getUrl())) {
            //如果是自己的网站，则继续使用SSL证书
            sslErrorHandler.proceed();
        } else {
            super.onReceivedSslError(webView, sslErrorHandler, sslError);
        }
    }
    ```



### 4.0.7 webView防止内存泄漏
- https://my.oschina.net/zhibuji/blog/100580


### 4.0.8 关于js注入时机修改
- **onPageFinished()或者onPageStarted()方法中注入js代码**
    - 做过WebView开发，并且需要和js交互，大部分都会认为js在WebViewClient.onPageFinished()方法中注入最合适，此时dom树已经构建完成，页面已经完全展现出来。但如果做过页面加载速度的测试，会发现WebViewClient.onPageFinished()方法通常需要等待很久才会回调（首次加载通常超过3s），这是因为WebView需要加载完一个网页里主文档和所有的资源才会回调这个方法。
    - 能不能在WebViewClient.onPageStarted()中注入呢？答案是不确定。经过测试，有些机型可以，有些机型不行。在WebViewClient.onPageStarted()中注入还有一个致命的问题——这个方法可能会回调多次，会造成js代码的多次注入。
    - 从7.0开始，WebView加载js方式发生了一些小改变，**官方建议把js注入的时机放在页面开始加载之后**。
- **WebViewClient.onProgressChanged()方法中注入js代码**
    - WebViewClient.onProgressChanged()这个方法在dom树渲染的过程中会回调多次，每次都会告诉我们当前加载的进度。
        - 在这个方法中，可以给WebView自定义进度条，类似微信加载网页时的那种进度条
        - 如果在此方法中注入js代码，则需要避免重复注入，需要增强逻辑。可以定义一个boolean值变量控制注入时机
    - 那么有人会问，加载到多少才需要处理js注入逻辑呢？
        - 正是因为这个原因，页面的进度加载到80%的时候，实际上dom树已经渲染得差不多了，表明WebView已经解析了<html>标签，这时候注入一定是成功的。在WebViewClient.onProgressChanged()实现js注入有几个需要注意的地方：
        - 1 上文提到的多次注入控制，使用了boolean值变量控制
        - 2 重新加载一个URL之前，需要重置boolean值变量，让重新加载后的页面再次注入js
        - 3 如果做过本地js，css等缓存，则先判断本地是否存在，若存在则加载本地，否则加载网络js
        - 4 注入的进度阈值可以自由定制，理论上10%-100%都是合理的，不过建议使用了75%到90%之间可以。


### 4.0.9 视频/图片宽度超过屏幕
- 视频播放宽度或者图片宽度比webView设置的宽度大，超过屏幕：这个时候可以设置ws.setLoadWithOverviewMode(true);
- 另外一种让图片不超出屏幕范围的方法，可以用的是css
    ```
    <script type="text/javascript">
       var tables = document.getElementsByTagName("img");  //找到table标签
         for(var i = 0; i<tables.length; i++){  // 逐个改变
                tables[i].style.width = "100%";  // 宽度改为100%
                 tables[i].style.height = "auto";
         }
    </script>
    ```
- 设置加载进来的页面自适应手机屏幕，通过webView的setting属性设置。
    ```
    // 网页内容的宽度是否可大于WebView控件的宽度
    settings.setUseWideViewPort(true); 
    settings.setLoadWithOverviewMode(true); 
    ```


### 4.1.0 如何保证js安全性
- Android和js如何通信
    - 为了与Web页面实现动态交互，Android应用程序允许WebView通过WebView.addJavascriptInterface接口向Web页面注入Java对象，页面Javascript脚本可直接引用该对象并调用该对象的方法。
    - 这类应用程序一般都会有类似如下的代码：
        ```
        webView.addJavascriptInterface(javaObj, "jsObj");
        ```
    - 此段代码将javaObj对象暴露给js脚本，可以通过jsObj对象对其进行引用，调用javaObj的方法。结合Java的反射机制可以通过js脚本执行任意Java代码，相关代码如下：
        - 当受影响的应用程序执行到上述脚本的时候，就会执行someCmd指定的命令。
        ```
        <script>
        　　function execute(cmdArgs) {
            　　return jsobj.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec(cmdArgs);
        　　}
        
        　　execute(someCmd);
        </script>
        ```
- addJavascriptInterface任何命令执行漏洞
    - 在webView中使用js与html进行交互是一个不错的方式，但是，在Android4.2(16，包含4.2)及以下版本中，如果使用addJavascriptInterface，则会存在被注入js接口的漏洞；在4.2之后，由于Google增加了@JavascriptInterface，该漏洞得以解决。
- @JavascriptInterface注解做了什么操作
    - 之前，任何Public的函数都可以在JS代码中访问，而Java对象继承关系会导致很多Public的函数都可以在JS中访问，其中一个重要的函数就是getClass()。然后JS可以通过反射来访问其他一些内容。通过引入 @JavascriptInterface注解，则在JS中只能访问 @JavascriptInterface注解的函数。这样就可以增强安全性。
- 在4.2之前，存在漏洞，解决方案如下所示。移除android 4.2之前的默认接口
    ```
    removeJavascriptInterface(“searchBoxJavaBridge_”)
    removeJavascriptInterface(“accessibility”)
    removeJavascriptInterface(“accessibilityTraversal”)
    ```



### 4.1.1 如何代码开启硬件加速
- 开启软硬件加速这个性能提升还是很明显的，但是会耗费更大的内存 。直接调用代码api即可完成，webView.setOpenLayerType(true);
- 硬件加速分为四个级别：
    - Application级别<application android:hardwareAccelerated="true"...>
    - Activity级别<activity android:hardwareAccelerated="true"...>
    - window级别（目前为止，Android还不支持在Window级别关闭硬件加速。）
        ```
        getWindow().setFlags(
            WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
            WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED);
        ```
    - View级别view.setLayerType(View.LAYER_TYPE_HARDWARE, null);




### 4.1.2 WebView设置Cookie
- h5页面为何要设置cookie，主要是避免网页重复登录，作用是记录用户登录信息，下次进去不需要重复登录。
- 代码里怎么设置Cookie，如下所示
    ```
    /**
     * 同步cookie
     *
     * @param url               地址
     * @param cookieList        需要添加的Cookie值,以键值对的方式:key=value
     */
    private void syncCookie (Context context , String url, ArrayList<String> cookieList) {
        //初始化
        CookieSyncManager.createInstance(context);
        //获取对象
        CookieManager cookieManager = CookieManager.getInstance();
        cookieManager.setAcceptCookie(true);
        //移除
        cookieManager.removeSessionCookie();
        //添加
        if (cookieList != null && cookieList.size() > 0) {
            for (String cookie : cookieList) {
                cookieManager.setCookie(url, cookie);
            }
        }
        String cookies = cookieManager.getCookie(url);
        X5LogUtils.d("cookies-------"+cookies);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            cookieManager.flush();
        } else {
            CookieSyncManager.getInstance().sync();
        }
    }
    ```
- 在android里面在调用webView.loadUrl(url)之前一句调用此方法就可以给WebView设置Cookie 
    - 注:这里一定要注意一点，在调用设置Cookie之后不能再设置，否则设置Cookie无效。该处需要校验，为何？？？
    ```
    webView.getSettings().setBuiltInZoomControls(true);  
    webView.getSettings().setJavaScriptEnabled(true);  
    ```
- 还有跨域问题： 域A: test1.yc.com 域B: test2.yc.com
    - 那么在域A生产一个可以使域A和域B都能访问的Cookie就需要将Cookie的domain设置为.yc.com；
    - 如果要在域A生产一个令域A不能访问而域能访问的Cookie就要将Cookie设置为test2.yc.com。
- Cookie的过期机制
    - 可以设置Cookie的生效时间字段名为： expires 或 max-age。
        - expires：过期的时间点
        - max-age：生效的持续时间，单位为秒。
    - 若将Cookie的 max-age 设置为负数，或者 expires 字段设置为过期时间点，数据库更新后这条Cookie将从数据库中被删除。如果将Cookie的 max-age 和 expires 字段设置为正常的过期日期，则到期后再数据库更新时会删除该条数据。
- 下面列出几个有用的接口：
    - 获取某个url下的所有Cookie：CookieManager.getInstance().getCookie(url)
    - 判断WebView是否接受Cookie：CookieManager.getInstance().acceptCookie()
    - 清除Session Cookie：CookieManager.getInstance().removeSessionCookies(ValueCallback<Boolean> callback)
    - 清除所有Cookie：CookieManager.getInstance().removeAllCookies(ValueCallback<Boolean> callback)
    - Cookie持久化：CookieManager.getInstance().flush()
    - 针对某个主机设置Cookie：CookieManager.getInstance().setCookie(String url, String value)



### 4.1.3 开启硬件加速导致的闪烁问题
- WebView开启硬件加速导致屏幕花屏原因分析
    - 4.0以上的系统我们开启硬件加速后，WebView渲染页面更加快速，拖动也更加顺滑。
    - 但有个副作用就是，当WebView视图被整体遮住一块，然后突然恢复时（比如使用SlideMenu将WebView从侧边滑出来时），这个过渡期会出现白块同时界面闪烁。
- 在应用开启硬件加速后，WebView可能在加载过程出现闪烁现象。解决方案：
    - 为WebView关闭硬件加速功能。在过渡期前将WebView的硬件加速临时关闭，过渡期后再开启，webView.setLayerType(View.LAYER_TYPE_SOFTWARE,null);




### 4.1.4 webView加载网页不显示图片
- webView从Lollipop(5.0)开始webView默认不允许混合模式, https当中不能加载http资源, 而开发的时候可能使用的是https的链接, 但是链接中的图片可能是http的, 所以需要设置开启。
    ```
    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.LOLLIPOP) {
            mWebView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
    }
    mWebView.getSettings().setBlockNetworkImage(false);
    ```



### 4.1.5 绕过证书校验漏洞
- webViewClient中有onReceivedError方法，当出现证书校验错误时，我们可以在该方法中使用handler.proceed()来忽略证书校验继续加载网页，或者使用默认的handler.cancel()来终端加载。
    - 因为我们使用了handler.proceed()，由此产生了该“绕过证书校验漏洞”。如果确定所有页面都能满足证书校验，则不必要使用handler.proceed()
    ```
    @SuppressLint("NewApi")
    @Override
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
        //handler.proceed();// 接受证书
        super.onReceivedSslError(view, handler, error);
    }
    ```








