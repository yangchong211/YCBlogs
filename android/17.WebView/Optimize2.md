#### 优化汇总目录介绍
- 5.1.6 WebView判断断网和链接超时
- 5.1.7 @JavascriptInterface注解方法注意点
- 5.1.8 使用onJsPrompt实现js通信注意点
- 5.1.9 Cookie同步场景和具体操作
- 5.2.0 shouldOverrideUrlLoading处理多类型
- 5.2.1 WebView独立进程解决方案
- 5.2.2 截取WebView屏幕的整个可视区域
- 5.2.3 截取WebView屏幕长图效果
- 5.2.4 Android P加载X5内核失败优化
- 5.2.5 WebView流量轻量优化
- 5.2.6 onReceivedTitle执行多次
- 5.2.7 WebView自定义长按选择弹窗
- 5.2.8 关于下拉刷新重新加载web页面优化




### 5.1.6 WebView判断断网和链接超时
- 第一种方法，可以直接使用，在WebViewClient中的onReceivedError方法中捕获
    ```
    @Override
    public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
        super.onReceivedError(view, errorCode, description, failingUrl);
        // 断网或者网络连接超时
        if (errorCode == ERROR_HOST_LOOKUP || errorCode == ERROR_CONNECT || errorCode == ERROR_TIMEOUT) {
            // 避免出现默认的错误界面
            view.loadUrl("about:blank"); 
            //view.loadUrl(mErrorUrl);
        }
    }
    ```
- 第二种方法，只是提供思路，但是不建议使用，既然是请求url，那么可不可以通过HttpURLConnection来获取状态码，是不是回到了最初学习的时候，哈哈
    - 注意一定是要开启子线程，因为网络请求是耗时操作，就不解释这多呢
    ```
    new Thread(new Runnable() {
        @Override
        public void run() {
            int responseCode = getResponseCode(mUrl);
            if (responseCode == 404 || responseCode == 500) {
                Message message = mHandler.obtainMessage();
                message.what = responseCode;
                mHandler.sendMessage(message);
            }
        }
    }).start();
    ```
    - 在主线程中处理消息
    ```
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            int code = msg.what;
            if (code == 404 || code == 500) {
                System.out.println("handler = " + code);
                mWebView.loadUrl(mErrorUrl);
            }
        }
    };
    ```
    - 那么看看getResponseCode(mUrl)是如何操作的
    ```
    /**
     * 获取请求状态码
     * @param url
     * @return 请求状态码
     */
    private int getResponseCode(String url) {
        try {
            URL getUrl = new URL(url);
            HttpURLConnection connection = (HttpURLConnection) getUrl.openConnection();
            connection.setRequestMethod("GET");
            connection.setReadTimeout(5000);
            connection.setConnectTimeout(5000);
            return connection.getResponseCode();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return -1;
    }
    ```



### 5.1.9 Cookie同步场景和具体操作
- 场景：C/S->B/S Cookie同步
    - 在Android混合应用开发中，一般来说，有些页面通过Native方式实现，有些通过WebView实现。对于登录而已，假设我们通过Native登录，需要把SessionID传递给WebView，这种情况需要同步。
- 场景：不同域名之间的Cookie同步
    - 对于分布式应用，或者灰度发布的应用，他的缓存服务器是同一个，那么域名的切换时会导致获取不到sessionID，因此，不同域名也需要特别处理。
    ```
    public static void synCookies(Context context, String url) {
        try {
            if (!UserManager.hasLogin()) {  
                //判断用户是否已经登录
                setCookie(context, url, "SESSIONID", "invalid");
            } else {
                setCookie(context, url, "SESSIONID", SharePref.getSessionId());
    
            }
        } catch (Exception e) {
        }
    }
    
    private static void setCookie( Context context,String url, String key, String value) {
        if (Build.VERSION.SDK_INT < 21) {
            CookieSyncManager.createInstance(context);
        }
        CookieManager cookieManager = CookieManager.getInstance();
        cookieManager.setAcceptCookie(true);  //同样允许接受cookie
        URL pathInfo = new URL(url);
        String[] whitelist = new String{".abc.com","abc.cn","abc.com.cn"}; //白名单
        String domain = null;
    
        for(int i=0;i<whitelist.length;i++){
            if(pathInfo.getHost().endsWith(whitelist[i])){
                domain  = whitelist[i];
                break;
            }
        }
        if(TextUtils.isEmpty(domain)) return; //不符合白名单的不同步cookie
        StringBuilder sbCookie = new StringBuilder();
        sbCookie.append(String.format("%s=%s", key, value));
        sbCookie.append(String.format(";Domain=%s", domain));
        sbCookie.append(String.format(";path=%s", "/"));
        String cookieValue = sbCookie.toString();
        cookieManager.setCookie(url, cookieValue);
        if (Build.VERSION.SDK_INT < 21) {
            CookieSyncManager.getInstance().sync();
        } else {
            CookieManager.getInstance().flush();
        }
    }
    ```
    - 调用位置：shouldOverrideUrlLoading中调用
    ```
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        //同步，当然还可以优化，如果前一个域名和后一个域名不同时我们再同步
        syncCookie(view.getContext(),url);   
        //省略代码……
    }
    ```
- 场景三：从B/S->C/S
    - 这种情况多发生于第三方登录，但是获取cookie通过CookieManager即可，但是的好时机是页面加载完成。
    - 注意：在页面加载之前一定要调用cookieManager.setAcceptCookie(true);  允许接受cookie，否则可能产生问题。
    ```
    public void onPageFinished(WebView view, String url) {  
        CookieManager cookieManager = CookieManager.getInstance();  
        String CookieStr = cookieManager.getCookie(url);  
        LogUtil.i("Cookies", "Cookies = " + CookieStr);  
        super.onPageFinished(view, url);  
    } 
    ```


### 5.2.0 shouldOverrideUrlLoading处理多类型
- 这个方法很强大，平时一般很少涉及处理多类型，比如电子邮件类型，地图类型，选中的文字类型等等。我在X5WebViewClient中的shouldOverrideUrlLoading方法中只是处理了未知类型
    - 还是很可惜，没遇到过复杂的h5页面，需要处理各种不同类型。这里只是简单介绍一下，知道就可以呢。
    ```
    /**
     * 这个方法中可以做拦截
     * 主要的作用是处理各种通知和请求事件
     * 返回值是true的时候控制去WebView打开，为false调用系统浏览器或第三方浏览器
     * @param view                              view
     * @param url                               链接
     * @return                                  是否自己处理，true表示自己处理
     */
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        //页面关闭后，直接返回，不要执行网络请求和js方法
        boolean activityAlive = X5WebUtils.isActivityAlive(context);
        if (!activityAlive){
            return false;
        }
        if (TextUtils.isEmpty(url)) {
            return false;
        }
        try {
            url = URLDecoder.decode(url, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        WebView.HitTestResult hitTestResult = null;
        if (url.startsWith("http:") || url.startsWith("https:")){
            hitTestResult = view.getHitTestResult();
        }
        if (hitTestResult == null) {
            return false;
        }
        //HitTestResult 描述
        //WebView.HitTestResult.UNKNOWN_TYPE 未知类型
        //WebView.HitTestResult.PHONE_TYPE 电话类型
        //WebView.HitTestResult.EMAIL_TYPE 电子邮件类型
        //WebView.HitTestResult.GEO_TYPE 地图类型
        //WebView.HitTestResult.SRC_ANCHOR_TYPE 超链接类型
        //WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE 带有链接的图片类型
        //WebView.HitTestResult.IMAGE_TYPE 单纯的图片类型
        //WebView.HitTestResult.EDIT_TEXT_TYPE 选中的文字类型
        if (hitTestResult.getType() == WebView.HitTestResult.UNKNOWN_TYPE) {
            return false;
        }
        return super.shouldOverrideUrlLoading(view, url);
    }
    ```


### 5.2.1 WebView独立进程解决方案
- https://www.jianshu.com/p/b66c225c19e2
- 待验证



### 5.2.2 截取WebView屏幕的整个可视区域
- 有两种方法，第一种是截取activity的可见区域的视图；第二种是根据View的宽高去draw视图
- 第一种是截取activity的可见区域的视图
    ```
    /**
     * 截屏，截取activity的可见区域的视图
     * @param activity
     * @return
     */
    public static Bitmap activityShot(Activity activity) {
        /*获取windows中最顶层的view*/
        View view = activity.getWindow().getDecorView();
        //允许当前窗口保存缓存信息
        view.setDrawingCacheEnabled(true);
        view.buildDrawingCache();
        //获取状态栏高度
        Rect rect = new Rect();
        view.getWindowVisibleDisplayFrame(rect);
        int statusBarHeight = rect.top;
        WindowManager windowManager = activity.getWindowManager();
        //获取屏幕宽和高
        DisplayMetrics outMetrics = new DisplayMetrics();
        windowManager.getDefaultDisplay().getMetrics(outMetrics);
        int width = outMetrics.widthPixels;
        int height = outMetrics.heightPixels;
        //去掉状态栏
        Bitmap bitmap = Bitmap.createBitmap(view.getDrawingCache(), 0, statusBarHeight, width, height - statusBarHeight);
        //销毁缓存信息
        view.destroyDrawingCache();
        view.setDrawingCacheEnabled(false);
        return bitmap;
    }
    ```
- 第二种是根据View的宽高去draw视图，这里提供截取RelativeLayout代码。其他的可以看项目demo。
    ```
    /**
     * 截取RelativeLayout
     **/
    public static Bitmap getRelativeLayoutBitmap(RelativeLayout relativeLayout) {
        int h = 0;
        Bitmap bitmap;
        for (int i = 0; i < relativeLayout.getChildCount(); i++) {
            h += relativeLayout.getChildAt(i).getHeight();
        }
        // 创建对应大小的bitmap
        bitmap = Bitmap.createBitmap(relativeLayout.getWidth(), h, Bitmap.Config.ARGB_8888);
        final Canvas canvas = new Canvas(bitmap);
        relativeLayout.draw(canvas);
        return bitmap;
    }
    ```


### 5.2.3 截取WebView屏幕长图效果
- 直接提供代码，如下所示：实际开发中建议用这种去截取长图，不仅仅使用webView，还适用LinearLayout，RelativeLayout等。
    ```
    /**
     * 计算view的大小
     */
    public static Bitmap measureSize(Activity activity, View view) {
        //将布局转化成view对象
        View viewBitmap = view;
        WindowManager manager = activity.getWindowManager();
        DisplayMetrics outMetrics = new DisplayMetrics();
        manager.getDefaultDisplay().getMetrics(outMetrics);
        int width = outMetrics.widthPixels;
        int height = outMetrics.heightPixels;
        //然后View和其内部的子View都具有了实际大小，也就是完成了布局，相当与添加到了界面上。
        //接着就可以创建位图并在上面绘制
        return layoutView(viewBitmap, width, height);
    }
    
    /**
     * 填充布局内容
     */
    private static  Bitmap layoutView(final View viewBitmap, int width, int height) {
        // 整个View的大小 参数是左上角 和右下角的坐标
        viewBitmap.layout(0, 0, width, height);
        int measuredWidth = View.MeasureSpec.makeMeasureSpec(width, View.MeasureSpec.EXACTLY);
        int measuredHeight = View.MeasureSpec.makeMeasureSpec(height, View.MeasureSpec.UNSPECIFIED);
        viewBitmap.measure(measuredWidth, measuredHeight);
        viewBitmap.layout(0, 0, viewBitmap.getMeasuredWidth(), viewBitmap.getMeasuredHeight());
        viewBitmap.setDrawingCacheEnabled(true);
        viewBitmap.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH);
        viewBitmap.setDrawingCacheBackgroundColor(Color.WHITE);
        // 把一个View转换成图片
        Bitmap cachebmp = viewConversionBitmap(viewBitmap);
        viewBitmap.destroyDrawingCache();
        return cachebmp;
    }
    
    /**
     * view转bitmap
     */
    private static Bitmap viewConversionBitmap(View v) {
        int w = v.getWidth();
        int h = 0;
        if (v instanceof LinearLayout){
            LinearLayout linearLayout = (LinearLayout) v;
            for (int i = 0; i < linearLayout.getChildCount(); i++) {
                h += linearLayout.getChildAt(i).getHeight();
            }
        } else if (v instanceof RelativeLayout){
            RelativeLayout relativeLayout = (RelativeLayout) v;
            for (int i = 0; i < relativeLayout.getChildCount(); i++) {
                h += relativeLayout.getChildAt(i).getHeight();
            }
        } else {
            h = v.getHeight();
        }
        Bitmap bmp = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);
        Canvas c = new Canvas(bmp);
        //如果不设置canvas画布为白色，则生成透明
        c.drawColor(Color.WHITE);
        v.layout(0, 0, w, h);
        v.draw(c);
        return bmp;
    }
    ```


### 5.2.4 Android P加载X5内核失败优化
- 关于Android 9：绝不多数手机是可以直接初始化成功的，但是到了Android 9，有人反馈部分手机初始化直接失败。
- 具体原因呢是因为从Android 6.0开始引入了对Https的推荐支持，与以往不同，Android P的系统上面默认所有Http的请求都被阻止了。
- 如何修改，代码如下所示。
    ```
    <application
        ...
        android:usesCleartextTraffic="true"
        ...>
        ...
    </application>
    ```


### 5.2.5 WebView流量轻量优化
- https://www.jianshu.com/p/39a9832847a6



### 5.2.6 onReceivedTitle执行多次
- onReceivedTitle(WebView view, String title)拿标题
    - 这个方法在网页回退时是无法拿到正确的上一级标题的，网上的处理方法是自己维护一个List去缓存标题，在执行完webView.goBack()后，移除List的最后一条，再将新的最后一条设置给标题栏。
    - 而且onReceivedTitle方法在一个页面打开时并不是仅调用一次，而是多次调用。
    ```
    /**
     * 获取标题
     * @param view                  view
     * @param title                 title
     */
    private void getWebTitle(WebView view, String title){
        /*if (mBasisView!=null){
            mBasisView.setTitle(title);
        }*/
        LogUtils.i("-------onReceivedTitle-----1--"+title);
        WebBackForwardList forwardList = view.copyBackForwardList();
        WebHistoryItem item = forwardList.getCurrentItem();
        if (item != null) {
            if (mBasisView!=null){
                mBasisView.setTitle(item.getTitle());
                LogUtils.i("-------onReceivedTitle----2---"+item.getTitle());
            }
        }
    }
    ```


### 5.2.7 WebView自定义长按选择弹窗
- https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650823357&idx=1&sn=673814ecce823d1d62367dd68733ea97&chksm=80b78e23b7c0073501d7fb058bec48bd9f376b0fe9631a1b44925ee20a1d3dc5cf0782b0d9ee&scene=38#wechat_redirect


### 5.2.8 关于下拉刷新重新加载web页面优化
- webView滑动到顶部，并且web已经加载结束后才能触发下拉刷新重新加载页面
    ```
    mSwipeRefreshContainer.setOnRefreshListener(new OnRefreshListener() {
        @Override
        public void onRefresh() {
            if (NetworkUtils.isNetworkAvailable()){
                //个人觉得下拉刷新更新web页面体验不太友好，参考qq浏览器，百度浏览器等，是点击触发刷新按钮刷新web
                //先结束下拉刷新，然后在加载web页面。
                //建议下拉刷新结束后加载web，不建议下拉刷新动画中就加载web
                if (mWebView!=null && mWebView.isTop() && mWebView.getWebViewClient().isLoadFinish()){
                    mSwipeRefreshContainer.setRefreshing(false);
                    mAgentWeb.reLoad();
                }
                //mSwipeRefreshContainer.setRefreshing(false);
            } else {
                mPaddingView.setViewState(NET);
            }
        }
    });
    ```






