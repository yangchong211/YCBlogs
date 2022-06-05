#### 优化汇总目录介绍
- 5.0.1 视频全屏播放按返回页面被放大
- 5.0.2 加快加载webView中的图片资源
- 5.0.3 自定义加载异常error的状态页面
- 5.0.4 WebView硬件加速导致页面渲染闪烁
- 5.0.6 web音频播放销毁后还有声音
- 5.0.9 后台无法释放js导致发热耗电
- 5.1.0 可以提前显示加载进度条
- 5.1.1 WebView密码明文存储漏洞优化
- 5.1.2 页面关闭后不要执行web中js
- 5.1.3 WebView + HttpDns优化
- 5.1.4 如何禁止WebView返回时刷新
- 5.1.5 WebView处理404、500逻辑





### 5.0.4 WebView硬件加速导致页面渲染闪烁
- 4.0以上的系统我们开启硬件加速后，WebView渲染页面更加快速，拖动也更加顺滑。但有个副作用就是，当WebView视图被整体遮住一块，然后突然恢复时（比如使用SlideMenu将WebView从侧边滑出来时），这个过渡期会出现白块同时界面闪烁。解决这个问题的方法是在过渡期前将WebView的硬件加速临时关闭，过渡期后再开启
    ```
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        webview.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
    }
    ```
- 关于硬件加速的原理是什么？






### 5.1.2 页面关闭后不要执行web中js
- 页面关闭后，直接返回，不要执行网络请求和js方法。代码如下所示：
    ```
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
        X5LogUtils.i("-------shouldOverrideUrlLoading----2---"+request.getUrl().toString());
        //页面关闭后，直接返回，不要执行网络请求和js方法
        boolean activityAlive = X5WebUtils.isActivityAlive(context);
        if (!activityAlive){
            return false;
        }
    }
    ```



### 5.1.4 如何禁止WebView返回时刷新
- webView在内部跳转的新的链接的时候，发现总会在返回的时候reload()一遍，但有时候我们希望保持上个状态。两种解决办法
- 第一种方法
    - 如果仅仅是简单的不更新数据，可以设置： mX5WebView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ONLY);
    - 如果你在网上搜索，你会发现很多都是这个回答，我也不知道别人究竟有没有试过这种方法，这个方案是不可行的。
- 第二种方法
    - new一个WebView，有人说部分浏览器都是从新new的webView，借鉴这种方法亲测可用： 
    - 布局里添加一个容器：
        ```
        <FrameLayout
            android:id="@+id/webContentLayout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>
        ```
    - 然后动态生成WebView，并且添加进去就可以
        ```
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_web);
            webContentLayout = (FrameLayout)findViewBy(R.id.webContentLayout);
            addWeb(url);
        }
        
        private void addWeb(String url) {
            WebView mWeb = new WebView(MainActivity.this);
            FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT,
                    ViewGroup.LayoutParams.MATCH_PARENT
            );
            mWeb.setLayoutParams(params);
            mWeb.setWebChromeClient(new WebChromeClient());
            mWeb.setWebViewClient(new MyWebViewClient());
            mWeb.getSettings().setJavaScriptEnabled(true);
            mWeb.loadUrl(url);
            webContentLayout.addView(mWeb);
        }
        
        //截获跳转
        private class MyWebViewClient extends WebViewClient{
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                Log.i(TAG, "shouldOverrideUrlLoading: " + url);
                if (!urlList.contains(url)) {
                    addWeb(url);
                    urlList.add(url);
                    return true;
                } else {
                    return super.shouldOverrideUrlLoading(view, url);
                }
            }
        }
        
        //返回处理，和传统的mWeb.canGoBack()不一样了，而是直接remove
        @Override
        public void onBackPressed() {
            int childCount = webContentLayout.getChildCount();
            if (childCount > 1) {
                webContentLayout.removeViewAt(childCount - 1);
            } else {
                super.onBackPressed();
            }
        }
        ```


### 5.1.5 WebView处理404、500逻辑
- 在Android6.0以下的系统我们如何处理404这样的问题呢？
    - 通过获取网页的title，判断是否为系统默认的404页面。在WebChromeClient()子类中可以重写他的onReceivedTitle()方法
    ```
    @Override
    public void onReceivedTitle(WebView view, String title) {
        super.onReceivedTitle(view, title);
        // android 6.0 以下通过title获取
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            if (title.contains("404") || title.contains("500") || title.contains("Error")) {
                // 避免出现默认的错误界面
                view.loadUrl("about:blank");
                //view.loadUrl(mErrorUrl);
            }
        }
    }
    ```
- Android6.0以上判断404或者500
    ```
    @TargetApi(android.os.Build.VERSION_CODES.M)
    @Override
    public void onReceivedHttpError(WebView view, WebResourceRequest request, WebResourceResponse errorResponse) {
        super.onReceivedHttpError(view, request, errorResponse);
        // 这个方法在6.0才出现
        int statusCode = errorResponse.getStatusCode();
        System.out.println("onReceivedHttpError code = " + statusCode);
        if (404 == statusCode || 500 == statusCode) {
            //避免出现默认的错误界面
            view.loadUrl("about:blank");
            //view.loadUrl(mErrorUrl);
        }
    }
    ```



