### 02.该库如何使用
#### 目录介绍
- 2.1 如何引入该库
- 2.2 最简单使用
- 2.3 常用api说明
- 2.4 使用建议
- 2.5 关于web页面异常状态区分类型
- 2.6 如何使用拦截缓存
- 2.7 使用Https+Dns
- 2.8 js交互操作处理
- 2.9 关于添加混淆



### 2.1 如何引入
- **如何引用，该x5的库已经更新到最新版本，引用最新稳定版，采用maven依赖**
    ```
    //第一种：以maven依赖形式
    implementation 'cn.yc:WebViewLib:1.4.7'
    api 'com.tencent.tbs.tbssdk:sdk:43967'
  
    //第二种：以jar依赖形式
    implementation 'cn.yc:WebViewLib:1.4.8'
    ```


### 2.2 最简单使用
- **项目初始化**
    ```
    X5WebUtils.init(this);
    ```
- **可以使用X5WebView，已经做了常见的setting属性设置**
    ```
    <X5WebView
        android:id="@+id/web_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbarSize="3dp" />
    ```
- **如果想有带进度的，可以使用ProgressWebView**
    ```
    <可以使用ProgressWebView
        android:id="@+id/web_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbarSize="3dp" />
    ```
- **如何使用自己的WebViewClient和WebChromeClient**
    ```
    //主要是在X5WebViewClient和X5WebChromeClient已经做了很多常见的逻辑处理，如果不满足你使用，可以如下这样写
    YcX5WebViewClient webViewClient = new YcX5WebViewClient(webView, this);
    webView.setWebViewClient(webViewClient);
    YcX5WebChromeClient webChromeClient = new YcX5WebChromeClient(webView,this);
    webView.setWebChromeClient(webChromeClient);

    private class YcX5WebViewClient extends MyX5WebViewClient {
        public YcX5WebViewClient(X5WebView webView, Context context) {
            super(webView, context);
        }

        //重写你需要的方法即可
    }

    private class YcX5WebChromeClient extends X5WebChromeClient{
        public YcX5WebChromeClient(X5WebView webView,Activity activity) {
            super(webView,activity);
            //重写你需要的方法即可
        }
    }
    ```
- **针对类似购物的商品详情页面的webView**
    - 当WebView在最顶部或者最底部的时候，不消费事件，则可以使用VerticalWebView



### 2.3 常用api
- **关于web的接口回调，包括常见状态页面切换，进度条变化等监听处理**
    ```
    mWebView.getX5WebChromeClient().setWebListener(interWebListener);
    private InterWebListener interWebListener = new InterWebListener() {
        @Override
        public void hindProgressBar() {
            pb.setVisibility(View.GONE);
        }

        @Override
        public void showErrorView(@X5WebUtils.ErrorType int type) {
            //设置自定义异常错误页面
        }

        @Override
        public void startProgress(int newProgress) {
            //该方法是是监听进度条进度变化的逻辑
            pb.setProgress(newProgress);
        }

        @Override
        public void showTitle(String title) {
            //该方法是监听h5中title
        }
    };
    ```
- **关于视频播放的时候，web的接口回调，主要是视频相关回调，比如全频，取消全频，隐藏和现实webView**
    ```
    x5WebChromeClient = x5WebView.getX5WebChromeClient();
    x5WebChromeClient.setVideoWebListener(new VideoWebListener() {
        @Override
        public void showVideoFullView() {
            //视频全频播放时监听
        }

        @Override
        public void hindVideoFullView() {
            //隐藏全频播放，也就是正常播放视频
        }

        @Override
        public void showWebView() {
            //显示webView
        }

        @Override
        public void hindWebView() {
            //隐藏webView
        }
    });
    ```
- **其他api说明**
    ```
    //X5WebView中
    //设置是否开启密码保存功能，不建议开启，默认已经做了处理，存在盗取密码的危险
    mWebView.setSavePassword(false);
    //是否开启软硬件加速
    mWebView.setOpenLayerType(false);
    //获取x5WebChromeClient对象
    x5WebChromeClient = mWebView.getX5WebChromeClient();
    //获取x5WebViewClient对象
    x5WebViewClient = mWebView.getX5WebViewClient();
    ```
- **关于如何使用仿微信加载H5页面进度条**
    - 前端页面时受到网路环境，页面内容大小的影响有时候会让用户等待很久。显示一个加载进度条可以说很大程度上提升用户的体验。
    ```
    private WebProgress pb;
    //显示进度条
    pb.show();
    //设置进度条过度颜色
    pb.setColor(Color.BLUE,Color.RED);
    //设置单色进度条
    pb.setColor(Color.BLUE);
    //为单独处理WebView进度条
    pb.setWebProgress(newProgress);
    //进度完成后消失
    pb.hide();
    ```
- 设置cookie和清除cookie操作
    ```
    //同步cookie
    WebkitCookieUtils.syncCookie(this,url,cookieList);
    //清除cookie
    WebkitCookieUtils.remove(url);
    ```
- 关于x5相关的功能
    ```
    //夜间模式，enable:true(日间模式)，enable：false（夜间模式）
    mWebView.setDayOrNight(true);
    //前进后退缓存，true表示缓存
    mWebView.setContentCacheEnable(true);
    //对于无法缩放的页面当用户双指缩放时会提示强制缩放，再次操作将触发缩放功能
    mWebView.setForcePinchScaleEnabled(true);
    //设置无痕模式
    mWebView.setShouldTrackVisitedLinks(true);
    //刘海屏适配
    mWebView.setDisplayCutoutEnable(true);
    //一次性删除所有缓存
    mWebView.clearAllWebViewCache(true);
    //缓存清除，针对性删除
    mWebView.clearCache();
    ```


### 2.4 使用建议
- **优化一下相关的操作**
    - 关于设置js支持的属性
    ```
    @Override
    public void onResume() {
        super.onResume();
        if (mWebView != null) {
            mWebView.getSettings().setJavaScriptEnabled(true);
        }
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (mWebView != null) {
            mWebView.getSettings().setJavaScriptEnabled(false);
        }
    }
    ```
    - 关于destroy销毁逻辑
    ```
    @Override
    protected void onDestroy() {
        if (webView != null) {
            webView.destroy();
        }
        super.onDestroy();
    }
    ```


### 2.5 关于web页面异常状态区分类型
- 对于web加载异常，分为多种状态，比如常见的有，没有网络；404加载异常；onReceivedError，请求网络出现error；在加载资源时通知主机应用程序发生SSL错误
    ```
    @Override
    public void showErrorView(@X5WebUtils.ErrorType int type) {
        switch (type){
            //没有网络
            case X5WebUtils.ErrorMode.NO_NET:
                break;
            //404，网页无法打开
            case X5WebUtils.ErrorMode.STATE_404:
                break;
            //onReceivedError，请求网络出现error
            case X5WebUtils.ErrorMode.RECEIVED_ERROR:
                break;
            //在加载资源时通知主机应用程序发生SSL错误
            case X5WebUtils.ErrorMode.SSL_ERROR:
                break;
            default:
                break;
        }
    }
    ```


### 2.6 如何使用拦截缓存
- 针对x5库的webView拦截使用方法如下所示，要是你想单独用这块缓存的功能，你可以直接将cache代码拷贝到你的项目中。功能独立，拿来即用！
    ```
    YcX5WebViewClient webViewClient = new YcX5WebViewClient(mWebView, this);
    mWebView.setWebViewClient(webViewClient);

    private class YcX5WebViewClient extends JsX5WebViewClient {
        public YcX5WebViewClient(X5WebView webView, Context context) {
            super(webView, context);
        }

        /**
         * 此方法废弃于API21，调用于非UI线程，拦截资源请求并返回响应数据，返回null时WebView将继续加载资源
         * 注意：API21以下的AJAX请求会走onLoadResource，无法通过此方法拦截
         *
         * 其中 WebResourceRequest 封装了请求，WebResourceResponse 封装了响应
         * 封装了一个Web资源的响应信息，包含：响应数据流，编码，MIME类型，API21后添加了响应头，状态码与状态描述
         * @param webView                           view
         * @param s                                 s
         */
        @Override
        public WebResourceResponse shouldInterceptRequest(WebView webView, String s) {
            WebResourceResponse request = WebViewCacheDelegate.getInstance().interceptRequest(s);
            return WebResponseAdapter.adapter(request);
        }

        /**
         * 此方法添加于API21，调用于非UI线程，拦截资源请求并返回数据，返回null时WebView将继续加载资源
         *
         * 其中 WebResourceRequest 封装了请求，WebResourceResponse 封装了响应
         * 封装了一个Web资源的响应信息，包含：响应数据流，编码，MIME类型，API21后添加了响应头，状态码与状态描述
         * @param webView                           view
         * @param webResourceRequest                request，添加于API21，封装了一个Web资源的请求信息，
         *                                          包含：请求地址，请求方法，请求头，是否主框架，是否用户点击，是否重定向
         * @return
         */
        @Override
        public WebResourceResponse shouldInterceptRequest(WebView webView, WebResourceRequest webResourceRequest) {
            WebResourceResponse request = WebViewCacheDelegate.getInstance().
                    interceptRequest(webResourceRequest);
            return WebResponseAdapter.adapter(request);
        }

    }
    ```
- 不过，别忘记了拦截缓存初始化
    ```
    public static void initCache(Application application){
        //1.创建委托对象
        WebViewCacheDelegate webViewCacheDelegate = WebViewCacheDelegate.getInstance();
        //2.创建调用处理器对象，实现类
        WebViewCacheWrapper.Builder builder = new WebViewCacheWrapper.Builder(application);
        //设置缓存路径，默认getCacheDir，名称CacheWebViewCache
        builder.setCachePath(new File(application.getCacheDir(),"CacheWebViewCache"))
                //设置缓存大小，默认100M
                .setCacheSize(1024*1024*100)
                //设置http请求链接超时，默认20秒
                .setConnectTimeoutSecond(20)
                //设置http请求链接读取超时，默认20秒
                .setReadTimeoutSecond(20)
                //设置缓存为正常模式，默认模式为强制缓存静态资源
                .setCacheType(WebCacheType.NORMAL);
        webViewCacheDelegate.init(builder);
    }
    ```


#### 2.7 使用Https+Dns



#### 2.8 js交互操作处理
- Java调用js的使用方法
    - 代码如下所示，下面updateAttentionStatus代表js这边的方法名称
    - webView.callHandler(“updateAttentionStatus”, …, new CallBackFunction());这是Java层主动调用Js的”updateAttentionStatus”方法。
    ```
    mWebView.callHandler("updateAttentionStatus", attention, new CallBackFunction() {
        @Override
        public void onCallBack(String data) {

        }
    });
    ```
- js调用java的使用方法
    - **代码如下所示，下面中的toPhone代表的是Android这边提供给js的方法名称**
    - webView.registerHandler(“toPhone”, …);这是Java层注册了一个叫”toPhone”的接口方法，目的是提供给Js来调用。这个”toPhone”的接口方法的回调就是BridgeHandler.handler()。
    ```
    mWebView.registerHandler("toPhone", new BridgeHandler() {
        @Override
        public void handler(String data, CallBackFunction function) {
            try {
                JSONObject jsonData = new JSONObject(data);
                String phone = jsonData.optString("phone");
                //todo 打电话
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    });
    ```
    - **如何回调数据给web那边**
    ```
    //注意，这里回传数据目前只是支持String字符串类型
    function.onCallBack("回调数据");
    ```


### 2.9 添加混淆
- 代码如下所示
    ```
    # 该包下所有的类和类成员不混淆
    -keep class com.ycbjie.webviewlib.** {
        *;
    }
    -dontwarn com.ycbjie.webviewlib.**
    ```
- 关于腾讯x5混淆
    ```
    -dontwarn dalvik.**
    -dontwarn com.tencent.smtt.**
    
    -keep class com.tencent.smtt.** {
        *;
    }
    
    -keep class com.tencent.tbs.** {
        *;
    }
    ```





