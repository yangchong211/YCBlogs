#### 使用目录介绍
- 01.WebView加载html页面
- 02.加载WebViewJavascriptBridge.js
- 03.分析WebViewJavascriptBridge.js
- 04.页面Html注册”functionInJs”方法
- 05.“functionInJs”执行结果回传Java
- 06.该库交互流程图



### 00.大概流程梳理
- Android调用js
    - 调用callHandler方法，然后将方法中的三个属性封装到WebJsMessage对象中。如果用户传递callback不为空，则添加到map集合中
    - 然后开始分发数据，把这些先都添加到list集合中。那么何时调用呢？
    - 然后看一下JsX5WebViewClient类的onPageFinished，获取list集合，然后遍历依次调用dispatchMessage处理事件。注意这里借助了handler发送到主线程队列中处理
    - 补充一下：
- js调用Android
    - 调用registerHandler方法，设置Android方法和handler存放到messageHandlers集合中。那么那个地方去获取集合信息，然后处理事件呢？
    - 然后在shouldOverrideUrlLoading方法中，拦截url是否是自己定义的scheme，则开始分发消息



### 01.WebView加载html页面
- webView.registerHandler(“submitFromWeb”, …);这是Java层注册了一个叫”submitFromWeb”的接口方法，目的是提供给Js来调用。这个”submitFromWeb”的接口方法的回调就是BridgeHandler.handler()。
- webView.callHandler(“functionInJs”, …, new CallBackFunction());这是Java层主动调用Js的”functionInJs”方法。
    ```
    public class MainActivity extends Activity implements OnClickListener {
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            webView = (BridgeWebView) findViewById(R.id.webView);
            webView.loadUrl("file:///android_asset/demo.html");
            webView.registerHandler("submitFromWeb", new BridgeHandler() {
                @Override
                public void handler(String data, CallBackFunction function) {
                    Log.i(TAG, "handler = submitFromWeb, data from web = " + data);
                    function.onCallBack("submitFromWeb exe, response data 中文 from Java");
                }
            });
    
            webView.callHandler("functionInJs", new Gson().toJson(user), new CallBackFunction() {
                @Override
                public void onCallBack(String data) {
    
                }
            });
        }
    }
    ```
- 一层层深入callHandler()方法的实现。这其中会调用到doSend()方法，这里想解释下m.setCallbackId(callbackStr)方法的作用。
    - 该方法设置的callbackId生成后不仅仅会被传到Js，而且会以key-value对的形式和responseCallback配对保存到responseCallbacks这个Map里面。
    - 它的目的，就是为了等Js把处理结果回调给Java层后，Java层能根据callbackId找到对应的responseCallback，做后续的回调处理。
    ```
    private void doSend(String handlerName, String data, CallBackFunction responseCallback) {
        Message m = new Message();
        if (!TextUtils.isEmpty(data)) {
            m.setData(data);
        }
        if (responseCallback != null) {
            String callbackStr = String.format(BridgeUtil.CALLBACK_ID_FORMAT, ++uniqueId + (BridgeUtil.UNDERLINE_STR + SystemClock.currentThreadTimeMillis()));
            responseCallbacks.put(callbackStr, responseCallback);
            m.setCallbackId(callbackStr);
        }
        if (!TextUtils.isEmpty(handlerName)) {
            m.setHandlerName(handlerName);
        }
        queueMessage(m);
    }
    ```
- 最终可以看到是BridgeWebView.dispatchMessage(Message m)方法调用的是this.loadUrl()，调用了_handleMessageFromNative这个Js方法。那这个Js的方法是哪里来的呢？
    ```
    final static String JS_HANDLE_MESSAGE_FROM_JAVA = "javascript:WebViewJavascriptBridge._handleMessageFromNative('%s');";
    
    void dispatchMessage(Message m) {
        String messageJson = m.toJson();
        //escape special characters for json string
        messageJson = messageJson.replaceAll("(\\\\)([^utrn])", "\\\\\\\\$1$2");
        messageJson = messageJson.replaceAll("(?<=[^\\\\])(\")", "\\\\\"");
        String javascriptCommand = String.format(BridgeUtil.JS_HANDLE_MESSAGE_FROM_JAVA, messageJson);
        if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
            this.loadUrl(javascriptCommand);
        }
    }
    ```

### 02.加载WebViewJavascriptBridge.js
- 在WebViewClient.onPageFinished()里面的BridgeUtil.webViewLoadLocalJs(view, BridgeWebView.toLoadJs)。正是把保存在assert/WebViewJavascriptBridge.js加载到WebView中。
    ```
    /**
     * 当页面加载完成会调用该方法
     * @param view                              view
     * @param url                               url链接
     */
    @Override
    public void onPageFinished(WebView view, String url) {
        X5LogUtils.i("-------onPageFinished-------"+url);
        if (!X5WebUtils.isConnected(webView.getContext()) && webListener!=null) {
            //隐藏进度条方法
            webListener.hindProgressBar();
            //显示异常页面
            webListener.showErrorView(X5WebUtils.ErrorMode.NO_NET);
        }
        super.onPageFinished(view, url);
        //设置网页在加载的时候暂时不加载图片
        //webView.getSettings().setBlockNetworkImage(false);
        //页面finish后再发起图片加载
        if(!webView.getSettings().getLoadsImagesAutomatically()) {
            webView.getSettings().setLoadsImagesAutomatically(true);
        }
        //这个时候添加js注入方法
        //WebViewJavascriptBridge.js
        BridgeUtil.webViewLoadLocalJs(view, BridgeWebView.TO_LOAD_JS);
        if (webView.getStartupMessage() != null) {
            for (Message m : webView.getStartupMessage()) {
                //分发message 必须在主线程才分发成功
                webView.dispatchMessage(m);
            }
            webView.setStartupMessage(null);
        }
        //html加载完成之后，添加监听图片的点击js函数
        //addImageClickListener();
        addImageArrayClickListener(webView);
    }
    ```


### 03.分析WebViewJavascriptBridge.js
- 看看WebViewJavascriptBridge.js的代码，就能找到function _handleMessageFromNative()这个Js方法了。
    - _handleMessageFromNative()方法里面会调用_dispatchMessageFromNative()方法。
    - 当处理来自Java层的主动调用时候会走“直接发送”的else分支。
    - message.callbackId会被取出来，实例化一个responseCallback，而它是用来Js处理完成后把结果数据回调给Java层代码的。
    - 接着会根据message.handleName（在这个分析例子中，handleName的值就是”functionInJs”）在messageHandlers这个Map去获取handler，最后交给handler去处理。
    ```
    function _dispatchMessageFromNative(messageJSON) {
        setTimeout(function() {
            var message = JSON.parse(messageJSON);
            var responseCallback;
            //java call finished, now need to call js callback function
            if (message.responseId) {
                ...
            } else {
                //直接发送
                if (message.callbackId) {
                    var callbackResponseId = message.callbackId;
                    responseCallback = function(responseData) {
                        _doSend({
                            responseId: callbackResponseId,
                            responseData: responseData
                        });
                    };
                }
    
                var handler = WebViewJavascriptBridge._messageHandler;
                if (message.handlerName) {
                    handler = messageHandlers[message.handlerName];
                }
                //查找指定handler
                try {
                    handler(message.data, responseCallback);
                } catch (exception) {
                    if (typeof console != 'undefined') {
                        console.log("WebViewJavascriptBridge: WARNING: javascript handler threw.", message, exception);
                    }
                }
            }
        });
    }
    ```

### 04.页面Html注册”functionInJs”方法
- 延续上面的分析，messageHandler是哪里设置的呢。答案就在当初webView.loadUrl(“file:///android_asset/demo.html”);加载的这个demo.html中。bridge.registerHandler(“functionInJs”, …)这里注册了”functionInJs”。
    ```
    <html>
        <head>
        ...
        </head>
        <body>
        ...
        </body>
        <script>
            ...
    
            connectWebViewJavascriptBridge(function(bridge) {
                bridge.init(function(message, responseCallback) {
                    console.log('JS got a message', message);
                    var data = {
                        'Javascript Responds': '测试中文!'
                    };
                    console.log('JS responding with', data);
                    responseCallback(data);
                });
    
                bridge.registerHandler("functionInJs", function(data, responseCallback) {
                    document.getElementById("show").innerHTML = ("data from Java: = " + data);
                    var responseData = "Javascript Says Right back aka!";
                    responseCallback(responseData);
                });
            })
        </script>
    </html>
    ```

### 05.“functionInJs”执行结果回传Java
- “funciontInJs”执行完毕后调用的responseCallback正是_dispatchMessageFromNative()实例化的，而它实际会调用_doSend()方法。_doSend()方法会先把Message推送到sendMessageQueue中。然后修改messagingIframe.src，这里会出发Java层的WebViewClient.shouldOverrideUrlLoading()的回调。
    ```
    function _doSend(message, responseCallback) {
        if (responseCallback) {
            var callbackId = 'cb_' + (uniqueId++) + '_' + new Date().getTime();
            responseCallbacks[callbackId] = responseCallback;
            message.callbackId = callbackId;
        }
    
        sendMessageQueue.push(message);
        messagingIframe.src = CUSTOM_PROTOCOL_SCHEME + '://' + QUEUE_HAS_MESSAGE;
    }
    ```
- 在BridgeWebViewClient.shouldOverrideUrlLoading()里面，会先执行webView.flushMessageQueue()的分支。
    ```
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        try {
            url = URLDecoder.decode(url, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    
        if (url.startsWith(BridgeUtil.YY_RETURN_DATA)) { // 如果是返回数据
            webView.handlerReturnData(url);
            return true;
        } else if (url.startsWith(BridgeUtil.YY_OVERRIDE_SCHEMA)) { //
            webView.flushMessageQueue();
            return true;
        } else {
            return super.shouldOverrideUrlLoading(view, url);
        }
    }
    ```
- webView.flushMessageQueue()首先去执行Js的_flushQueue()方法，并附带着CallBackFunction。Js的_flushQueue()方法会把sendMessageQueue中的所有message都回传给Java层。CallBackFunction就是把messageQueue解析出来后一个一个Message在for循环中处理，也正是在for循环中，”functionInJs”的Java层回调方法被执行了。
    ```
    void flushMessageQueue() {
        if (Thread.currentThread() == Looper.getMainLooper().getThread()) {
            loadUrl(BridgeUtil.JS_FETCH_QUEUE_FROM_JAVA, new CallBackFunction() {
    
                @Override
                public void onCallBack(String data) {
                    // deserializeMessage
                    List<Message> list = null;
                    try {
                        list = Message.toArrayList(data);
                    } catch (Exception e) {
                        e.printStackTrace();
                        return;
                    }
                    if (list == null || list.size() == 0) {
                        return;
                    }
                    for (int i = 0; i < list.size(); i++) {
                        ...
                    }
                }
            });
        }
    }
    ```
- 到此，JsBridge使用了MessageQueue后，分析起来有点绕。但原理是不变的，Js调用Java是通过WebViewClient.shouldOverrideUrlLoading()。当然，ava调用Js是通过WebView.loadUrl(“javascript:xxxx”)。



### 06.该库交互流程图
- java调用js的流程图
    - 第一步操作：mWebView.callHandler("functionInJs", "小杨逗比", new CallBackFunction() {//这里面是回调});
    - 第二步操作：将handlerName，data，responseCallback，封装到Message对象中，然后开始分发数据，最后webView执行_handleMessageFromNative；
    - 第三步操作：去WebViewJavascriptBridge.js类中找到_handleMessageFromNative方法，js根据"functionInJs"找到对应的js方法并且执行；
    - 第四步操作：js把运行结果保存到message对象中，然后添加到js消息队列中；
    - 第五步操作：在_dispatchMessageFromNative方法中，可以看到，js向native发送 "消息队列中有消息" 的通知；
    - 第六步操作：webView执行js的_fetchQueue（WebViewJavascriptBridge.js类）方法；
    - 第七步操作：js把消息队列中的所有消息都一起回传给webView；
    - 第八步操作：webView收到所有的消息，一个一个串行处理，注意其中包括 "functionInJs"方法运行的结果的消息；
- js调用Android的流程图
    - 第一步操作：mWebView.registerHandler("toPhone", new BridgeHandler() { //回调});
    - 第二步操作：调用messageHandlers.put(handlerName, handler)，将名称和BridgeHandler对象放到map集合中
    - 第三步操作：在shouldOverrideUrlLoading方法中拦截url，与网页约定好一个协议，匹配则执行相应操作，也就是利用WebViewClient接口回调方法拦截url
    - 第四步操作：如果是url.startsWith(BridgeUtil.YY_RETURN_DATA)则有数据返回；如果是BridgeUtil.YY_OVERRIDE_SCHEMA则刷新消息队列
    - 第五步操作：通过BridgeHandler对象，将data和callBackFunction返回交给开发者







