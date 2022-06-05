#### 问题汇总目录介绍
- 4.3.1 Android与js传递数据大小有限制
- 4.3.2 多次调用callHandler部分回调函数未被调用
- 4.3.3 字符串转义bug探讨
- 4.3.8 Javascript调用原生方法会偶现失败
- 4.3.9 dispatchMessage运行主线程问题
- 4.4.0 怎么实现WebView免流方案
- 4.4.1 Channel is unrecoverably broken and will be disposed!
- 4.4.2 定制js的alert,confirm和prompt对话框
- 4.4.3 x5长按图片如何操作
- 4.4 4 x5长按文字内容如何自定义弹窗
- 4.4.5 webView.goBack()会刷新页面吗





### 4.3.1 Android与js传递数据大小有限制
- android向H5传输图片，原生获取图片之后，最终转为base64后，通过js桥传送给H5
    ```
    mWebView.post(new Runnable() {
        @Override
        public void run() {
            mWebView.loadUrl("javascript:jsFunc('" + msg + "')");
         }
    });
    ```
- 问题：有时候图片过大，又想高质量的传送，可能遇到下面问题
    ```
    [WARNING:navigator_impl.cc(315)] Refusing to load URL as it exceeds 2097152 characters.
    //拒绝加载URL超过2097152个字符
    ```
- 解决方案
    ```
    if (script.length()>=URL_MAX_CHARACTER_NUM){
        BridgeWebView.super.evaluateJavascript(script, new ValueCallback<String>(){
            @Override
            public void onReceiveValue(String s) {
                X5LogUtils.i("---evaluateJavascript-2--"+s);
            }
        });
    } else {
        loadUrl(script);
    }
    ```
- 参考资料：[拒绝加载URL超过2097152个字符](https://stackoverflow.com/questions/38066503/android-webview-send-base64-url-to-javascript-refusing-to-load-url-as-it-exceed)


### 4.3.2 多次调用callHandler部分回调函数未被调用
- https://github.com/lzyzsd/JsBridge/issues/178
- https://github.com/lzyzsd/JsBridge/issues/170


### 4.3.3 字符串转义bug探讨
- 字符串转义bug。很多时候侨接不成功，这些问题本质上的原因都是通过js bridge传递数据转义有误导致。 
- 此bug会导致严重问题，如果传递的数据转义发生错误时，将导致不可用，像WebViewJavascriptBridge: WARNING: javascript handler threw.", source: (1) 这种错误很多时候都是因为js收到的数据和期望不符导致的异常（当然有些也有可能是js hanlder 处理不当抛出的）。这是一个偶现的致命bug 。要彻底解决这个问题最根本的方法就是不应该去转义，因为在传递数据格式未限定的情况下，只要转义，正常的数据字符串中都有可能匹配到转义规则（而这些字符串本身是不需要转义），这将会导致对于一部分数据能够正常转义，而一部分数据不能，这样的bug很难测试。 如果非要转义，就必须得限定jsbridge数据传递的格式，比如必须以json形式传递（不能直接传递string、bool等基础类型），这样才可以应用固定的转义规则解析。



### 4.3.8 Javascript调用原生方法会偶现失败
- 在测试过程中发现，失败的时机往往是webview调用 onPageFinished 前后，具体的表现是js调用native方法时 shouldOverrideUrlLoading（包括两种重载）没有被触发，所以端上没有去刷新js调用的message queue. 至于为什么没有就调用shouldOverrideUrlLoading，这是因为js和webview通信机制有问题，通过改变iframe src属性的这种方式并不能保证shouldOverrideUrlLoading每次都会被调用，这也是一些其它android jsbridge 会出现此问题的原因。



### 4.3.9 dispatchMessage运行主线程问题
- 先看一下案例代码
    ```
    // 必须要找主线程才会将数据传递出去 --- 划重点
    if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
        X5LogUtils.d("分发message--------------"+javascriptCommand);
        //this.loadUrl(javascriptCommand);
        //开始执行js中_handleMessageFromNative方法
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT &&
                javascriptCommand.length()>=URL_MAX_CHARACTER_NUM) {
            this.evaluateJavascript(javascriptCommand,null);
        }else {
            this.loadUrl(javascriptCommand);
        }
    }
    ```
- 可以看到，只有当是主线程时，才会执行分发的js，那么如果不是主线程呢？那自然就不会被分发，如果要确保这段代码没问题，那么就必须保证dispatchMessage是在主线程中被调用，而callHandler最终会调用dispatchMessage，所以就得保证callHandler必须在主线程调用。如果强制让用户去遵守这个规则是不靠谱的，况且有些时候，用户也不知道自己是否在主线程，示例代码中有在 onPageFinished时调用callHandler，不错onPageFinished在大多数情况下都会在主线程中被回调，但是我查了一圈，从没看到任何官方文档有说onPageFinished会在主线程中被回调。所以，那些出现的callHanlder不能调用bug的魅族手机，最好先去检查一下是否在主线程调用的。 所以，库中是应该保证callHandler无论是在哪个线程发起的调用, 最终的js都能在主线程被执行（因为webview要执行 s代码只能在主线程）。



### 4.4.0 怎么实现WebView免流方案
- 市面上常见的免流应用，原理无非就是走“特殊通道”，让这一部分的流量不计入运营商的流量统计平台中。Android中要实现这种“特殊通道”，有几种方案。
    - vpn。目前运营商貌似没有采用这种方案，但确实是可行的。
    - 全局代理。把所有的流量中转到代理服务器中，代理服务器再根据流量判断是否属于免流流量。
    - IP直连。走这个IP的所有流量，服务器判断是否免流。
- 对于上面提到的几种方案，native页面所有的请求都是应用层发起的，实际上都比较好实现，但WebView的页面和资源请求是通过JNI发起的，想要拦截请求的话，需要一些功夫。
    - 网罗网上的所有方案，目前觉得可行的有两种，分别是全局代理和拦截WebViewClient.shouldInterceptRequest()。
    - 更多参考[如何设计一个优雅健壮的Android WebView](https://blog.klmobile.app/2018/02/27/design-an-elegant-and-powerful-android-webview-part-two/)



### 4.4.1 Channel is unrecoverably broken and will be disposed!
- 程序崩溃，但是不报错原因，在网上翻了好久，网上好多说是jni错误，但是有种情况也会出现这种错误，就是bitmap.recycle()调用不对
- 把bitmap注释掉，不报错。



### 4.4.2 定制js的alert,confirm和prompt对话框
- https://www.iteye.com/blog/gundumw100-1158719
- https://blog.csdn.net/u012246458/article/details/53665597


### 4.4.3 x5长按图片如何操作
- x5支持长按事件监听，代码如下所示：
    ```
    webView.setOnLongClickListener(new View.OnLongClickListener() {
        @Override
        public boolean onLongClick(View v) {
            return handleLongImage();
        }
    });
    
    /**
     * 长按图片事件处理
     */
    private boolean handleLongImage() {
        final WebView.HitTestResult hitTestResult = webView.getHitTestResult();
        // 如果是图片类型或者是带有图片链接的类型
        if (hitTestResult.getType() == WebView.HitTestResult.IMAGE_TYPE ||
                hitTestResult.getType() == WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE) {
            // 弹出保存图片的对话框
            new AlertDialog.Builder(EightActivity.this)
                    .setItems(new String[]{"查看大图", "保存图片到相册"},
                            new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            String picUrl = hitTestResult.getExtra();
                            //获取图片
                            Log.e("picUrl", picUrl);
                            switch (which) {
                                case 0:
    
                                    break;
                                case 1:
                                    break;
                                default:
                                    break;
                            }
                        }
                    })
                    .show();
            return true;
        }
        return false;
    }
    ```


### 4.4 4 x5长按文字内容如何自定义弹窗
- ActionMode的使用特别的简单，主要用到两个方法，startActionMode和ActionMode.Callback()，startActionMode:开启我们的菜单
    ```
    ActionMode.Callback mCallback=new ActionMode.Callback(){
        /**
         * 创建菜单的样式，返回true说明创建成功
         * @param actionMode
         * @param menu
         * @return
         */
        @Override
        public boolean onCreateActionMode(ActionMode actionMode, Menu menu) {
            MenuInflater menuInflater = actionMode.getMenuInflater();
            menuInflater.inflate(R.menu.action_mode,menu);
            return true;
        }
    
        @Override
        public boolean onPrepareActionMode(ActionMode actionMode, Menu menu) {
            return false;
        }
    
        /**
         * 当ActionMode的条目被点击的时候，调用这个方法
         * @param actionMode
         * @param menuItem
         * @return
         */
        @Override
        public boolean onActionItemClicked(ActionMode actionMode, MenuItem menuItem) {
            return false;
        }
    
        /**
         * 当ActionMode被销毁的时候调用
         * @param actionMode
         */
        @Override
        public void onDestroyActionMode(ActionMode actionMode) {
            if(actionMode!=null){
                actionMode.finish();
            }
        }
    };
    ```
- 实现自定义ActionMode，重写startActionMode方法，拦截我们的ActionMode对象，然后对此进行一些处理就可以了，直接上代码
    ```
    @Override
    public ActionMode startActionMode(ActionMode.Callback callback) {
        ActionMode actionMode = super.startActionMode(callback);
        return resolveMode(actionMode);
    }
    
    @Override
    public ActionMode startActionMode(ActionMode.Callback callback, int type) {
        ActionMode actionMode = super.startActionMode(callback, type);
        return resolveMode(actionMode);
    }
    
    public ActionMode resolveMode(ActionMode actionMode) {
        if(actionMode!=null){
            final Menu menu = actionMode.getMenu();
            menu.clear();
            for (int i = 0; i < title.length; i++) {
                menu.add(title[i]);
            }
            for (int i = 0; i < title.length; i++) {
                MenuItem item = menu.getItem(i);
                item.setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
                    @Override
                    public boolean onMenuItemClick(MenuItem menuItem) {
    
                        String title = menuItem.getTitle().toString();
                        getSelectedData(title); //获取选中的h5页面的文本
                        releaseActionMode();
                        return true;
                    }
                });
            }
            this.mActionMode = actionMode;
        }
        return actionMode;
    }
    ```
- 当点击ActionMode的item的之后，将我们的actionMode finish掉
    ```
    public void releaseActionMode() {
        if (mActionMode != null) {
            mActionMode.finish();
            mActionMode = null;
        }
    }
    ```
- 获取h5页面的文本信息，需要使用到js方法来帮助我们实现这些功能，然后在通过js和java交互回传我们的文本内容（js和java如何交互，这里就不多说了......）
    ```
    /**
     * 点击的时候，获取网页中选择的文本，回掉到原生中的js接口
     * @param title 传入点击的item文本，一起通过js返回给原生接口
     */
    private void getSelectedData(String title) {
    
        String js = "(function getSelectedText() {" +
                "var txt;" +
                "var title = \"" + title + "\";" +
                "if (window.getSelection) {" +
                "txt = window.getSelection().toString();" +
                "} else if (window.document.getSelection) {" +
                "txt = window.document.getSelection().toString();" +
                "} else if (window.document.selection) {" +
                "txt = window.document.selection.createRange().text;" +
                "}" +
                "ActionModeJavaScript.callback(txt,title);" +           //回调java方法将js获取的结果传递过去
                "})()";
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {  //android系统4.4以上的时候调用js方法用这个
            evaluateJavascript("javascript:" + js, null);
        } else {
            loadUrl("javascript:" + js);
        }
    }
    ```


### 4.4.5 webView.goBack()会刷新页面吗









