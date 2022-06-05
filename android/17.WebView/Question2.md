#### 问题汇总目录介绍
- 4.1.6 allowFileAccess漏洞
- 4.1.7 WebView嵌套ScrollView问题
- 4.1.8 WebView中图片点击放大
- 4.1.9 页面滑动期间不渲染/执行
- 4.2.0 被运营商劫持和注入问题
- 4.2.1 解决资源加载缓慢问题
- 4.2.2 判断是否已经滚动到页面底端
- 4.2.3 使用loadData加载html乱码
- 4.2.4 WebView下载进度无法监听
- 4.2.5 webView出现302/303重定向
- 4.2.6 onPageFinished偶发不执行
- 4.2.8 onReceiveError问题
- 4.2.9 loadUrl在19以上超过2097152个字符失效
- 4.3.0 WebViewJavascriptBridge: WARNING





### 4.1.6 allowFileAccess漏洞
- 如果webView.getSettings().setAllowFileAccess(boolean)设置为true，则会面临该问题；该漏洞是通过WebView对Javascript的延时执行和html文件替换产生的。
    - 解决方案是禁止WebView页面打开本地文件，即：webView.getSettings().setAllowFileAccess(false);
    - 或者更直接的禁止使用JavaScript：webView.getSettings().setJavaScriptEnabled(false);



### 4.1.7 WebView嵌套ScrollView问题
- 问题描述
    - 当 WebView 嵌套在 ScrollView 里面的时候，如果 WebView 先加载了一个高度很高的网页，然后加载了一个高度很低的网页，就会造成 WebView 的高度无法自适应，底部出现大量空白的情况出现。
- 解决办法
    - 可以参考这篇博客：https://blog.csdn.net/self_study/article/details/54378978


### 4.1.8 WebView中图片点击放大
- 首先载入js
    ```
    //将js对象与java对象进行映射
    webView.addJavascriptInterface(new ImageJavascriptInterface(context), "imagelistener");
    ```
- html加载完成之后，添加监听图片的点击js函数，这个可以在onPageFinished方法中操作
    ```
    @Override
    public void onPageFinished(WebView view, String url) {
        X5LogUtils.i("-------onPageFinished-------"+url);
        //html加载完成之后，添加监听图片的点击js函数
        //addImageClickListener();
        addImageArrayClickListener(webView);
    }
    ```
- 具体看addImageArrayClickListener的实现方法。
    ```
    /**
     * android与js交互：
     * 首先我们拿到html中加载图片的标签img.
     * 然后取出其对应的src属性
     * 循环遍历设置图片的点击事件
     * 将src作为参数传给java代码
     * 这个循环将所图片放入数组，当js调用本地方法时传入。
     * 当然如果采用方式一获取图片的话，本地方法可以不需要传入这个数组
     * 通过js代码找到标签为img的代码块，设置点击的监听方法与本地的openImage方法进行连接
     * @param webView                       webview
     */
    private void addImageArrayClickListener(WebView webView) {
        webView.loadUrl("javascript:(function(){" +
                "var objs = document.getElementsByTagName(\"img\"); " +
                "var array=new Array(); " +
                "for(var j=0;j<objs.length;j++){" +
                "    array[j]=objs[j].src; " +
                "}"+
                "for(var i=0;i<objs.length;i++)  " +
                "{"
                + "    objs[i].onclick=function()  " +
                "    {  "
                + "        window.imagelistener.openImage(this.src,array);  " +
                "    }  " +
                "}" +
                "})()");
    }
    ```
- 最后看看js的通信接口做了什么
    ```
    public class ImageJavascriptInterface {
    
        private Context context;
        private String[] imageUrls;
    
        public ImageJavascriptInterface(Context context,String[] imageUrls) {
            this.context = context;
            this.imageUrls = imageUrls;
        }
    
        public ImageJavascriptInterface(Context context) {
            this.context = context;
        }
    
        /**
         * 接口返回的方式
         */
        @android.webkit.JavascriptInterface
        public void openImage(String img , String[] imageUrls) {
            Intent intent = new Intent();
            intent.putExtra("imageUrls", imageUrls);
            intent.putExtra("curImageUrl", img);
    //        intent.setClass(context, PhotoBrowserActivity.class);
            context.startActivity(intent);
            for (int i = 0; i < imageUrls.length; i++) {
                Log.e("图片地址"+i,imageUrls[i].toString());
            }
        }
    }
    ```


### 4.1.9 页面滑动期间不渲染/执行
- 在有些需求中会有一些吸顶的元素，例如导航条，购买按钮等；当页面滚动超出元素高度后，元素吸附在屏幕顶部。在WebView中成了难题：在页面滚动期间，Scroll Event不触发。不仅如此，WebView在滚动期间还有各种限定：
    - setTimeout和setInterval不触发。
    - GIF动画不播放。
    - 很多回调会延迟到页面停止滚动之后。
    - background-position: fixed不支持。
- 这些限制让WebView在滚动期间很难有较好的体验。这些限制大部分是不可突破的，但至少对于吸顶功能还是可以做一些支持，解决方法：
    - 在Android上，监听touchMove事件可以在滑动期间做元素的position切换（惯性运动期间就无效了）。
- 参考美团技术文章



### 4.2.0 被运营商劫持和注入问题
- 市面上出现这种情况的原因大概分为三种：
    - 1.DNS劫持（也就是运营商搞的鬼）
    - 2.http劫持(此类情况最多)
    - 3.项目中使用第三方的jar包
- 由于WebView加载的页面代码是从服务器动态获取的，这些代码将会很容易被中间环节所窃取或者修改，其中最主要的问题出自地方运营商和一些WiFi。监测到的问题包括：
    - 无视通信规则强制缓存页面。
    - header被篡改。
    - 页面被注入广告。
    - 页面被重定向。
    - 页面被重定向并重新iframe到新页面，框架嵌入广告。
    - HTTPS请求被拦截。
    - DNS劫持。
- 针对页面注入的行为，有一些解决方案：
    - 1.使用CSP（Content Security Policy）
    - 2.HTTPS。
        - HTTPS可以防止页面被劫持或者注入，然而其副作用也是明显的，网络传输的性能和成功率都会下降，而且HTTPS的页面会要求页面内所有引用的资源也是HTTPS的，对于大型网站其迁移成本并不算低。HTTPS的一个问题在于：一旦底层想要篡改或者劫持，会导致整个链接失效，页面无法展示。这会带来一个问题：本来页面只是会被注入广告，而且广告会被CSP拦截，而采用了HTTPS后，整个网页由于受到劫持完全无法展示。
        - 对于安全要求不高的静态页面，就需要权衡HTTPS带来的利与弊了。
    - 3.App使用Socket代理请求。参考美团技术方案
        - 如果HTTP请求容易被拦截，那么让App将其转换为一个Socket请求，并代理WebView的访问也是一个办法。
        - 通常不法运营商或者WiFi都只能拦截HTTP（S）请求，对于自定义的包内容则无法拦截，因此可以基本解决注入和劫持的问题。
        - Socket代理请求也存在问题：
        - 首先，使用客户端代理的页面HTML请求将丧失边下载边解析的能力；根据前面所述，浏览器在HTML收到部分内容后就立刻开始解析，并加载解析出来的外链、图片等，执行内联的脚本……而目前WebView对外并没有暴露这种流式的HTML接口，只能由客户端完全下载好HTML后，注入到WebView中。因此其性能将会受到影响。
        - 其次，其技术问题也是较多的，例如对跳转的处理，对缓存的处理，对CDN的处理等等……稍不留神就会埋下若干大坑。
        - 此外还有一些其他的办法，例如页面的MD5检测，页面静态页打包下载等等方式，具体如何选择还要根据具体的场景抉择。
- 熟悉几个概念
    - 什么是DNS劫持？
        - 把域名解析成ip就需要用到dns解析，DNS在作为域名和IP地址相互映射的一个分布式数据库，就是我们的浏览器，会将域名拿到DNS去解析出ip地址来访问，DNS劫持是指在劫持的网络范围内拦截域名解析的请求，分析请求的域名，把审查范围以外的请求放行，否则返回假的IP地址或者什么都不做使请求失去响应，其效果就是对特定的网络不能反应或访问的是假网址。
    - 什么是http劫持？
        - 通俗来说，你要去百度的首页，他会给你百度首页，然后再百度首页的某个部位＋个广告。 更具欺骗性，危害更大。




### 4.2.1 解决资源加载缓慢问题
- 在资源预加载方面，其实也有很多种方式，下面主要列举了一些：
    - 第一种方式是使用 WebView 自身的缓存机制：如果我们在 APP 里面访问一个页面，短时间内再次访问这个页面的时候，就会感觉到第二次打开的时候顺畅很多，加载速度比第一次的时间要短，这个就是因为 WebView 自身内部会做一些缓存，只要打开过的资源，他都会试着缓存到本地，第二次需要访问的时候他直接从本地读取，但是这个读取其实是不太稳定的东西，关掉之后，或者说这种缓存失效之后，系统会自动把它清除，我们没办法进行控制。基于这个 WebView 自身的缓存，有一种资源预加载的方案就是，我们在应用启动的时候可以开一个像素的 WebView ，事先去访问一下我们常用的资源，后续打开页面的时候如果再用到这些资源他就可以从本地获取到，页面加载的时间会短一些。
    - 第二种方案是，自己去构建和管理缓存：把这些需要预加载的资源放在 APP 里面，可能是预先放进去的，也可能是后续下载的，问题在于前端这些页面怎么去缓存，两个方案，第一种是前端可以在 H5 打包的时候把里面的资源 URL 进行替换，这样可以直接访问本地的地址；第二种是客户端可以拦截这些网页发出的所有请求做替换。
    - 具体可以看美团的技术文章：[美团大众点评 Hybrid 化建设](https://mp.weixin.qq.com/s/rNGD6SotKoO8frmxIU8-xw?)



### 4.2.2 判断是否已经滚动到页面底端
- getScrollY()方法返回的是当前可见区域的顶端距整个页面顶端的距离,也就是当前内容滚动的距离.
- getHeight()或者getBottom()方法都返回当前WebView 这个容器的高度
- getContentHeight 返回的是整个html的高度,但并不等同于当前整个页面的高度,因为WebView有缩放功能，所以当前整个页面的高度实际上应该是原始html 的高度再乘上缩放比例. 因此,更正后的结果,准确的判断方法应该是：
    ```
    if(WebView.getContentHeight*WebView.getScale() == (webview.getHeight()+WebView.getScrollY())){
        //已经处于底端
    }
    ```


### 4.2.3 使用loadData加载html乱码
- 可以通过使用 WebView.loadData(String data, String mimeType, String encoding)) 方法来加载一整个 HTML 页面的一小段内容，第一个就是我们需要 WebView 展示的内容，第二个是我们告诉 WebView 我们展示内容的类型，一般，第三个是字节码，但是使用的时候，这里会有一些坑
    - 明明已经指定了编码格式为 UTF-8，加载却还会出现乱码……
    ```
    String html = new String("<h3>我是loadData() 的标题</h3><p>&nbsp&nbsp我是他的内容</p>");
    webView.loadData(html, "text/html", "UTF-8");
    ```
- 使用loadData()或 loadDataWithBaseURL()加载一段HTML代码片段
    - data:是要加载的数据类型，但在数据里面不能出现英文字符：'#', '%', '\' , '?' 这四个字符，如果有的话可以用 %23, %25, %27, %3f，这些字符来替换，在平时测试时，你的数据时，你的数据里含有这些字符，但不会出问题，当出问题时，你可以替换下。
        * %，会报找不到页面错误，页面全是乱码。乱码样式见符件。
        * #，会让你的goBack失效，但canGoBAck是可以使用的。于是就会产生返回按钮生效，但不能返回的情况。
        * \ 和? 我在转换时，会报错，因为它会把\当作转义符来使用，如果用两级转义，也不生效，我是对它无语了。
    - 我们在使用loadData时，就意味着需要把所有的非法字符全部转换掉，这样就会给运行速度带来很大的影响，因为在使用时，在页面stytle中会使用很多%号。页面的数据越多，运行的速度就会越慢。
    - data中，有人会遇到中文乱码问题，解决办法：参数传"utf-8"，页面的编码格式也必须是utf-8，这样编码统一就不会乱了。别的编码我也没有试过。
- 解决办法
    ```
    String html = new String("<h3>我是loadData() 的标题</h3><p>&nbsp&nbsp我是他的内容</p>");
    webView.loadData(html, "text/html;charset=UTF-8", "null");
    ```


### 4.2.4 WebView下载进度无法监听
- https://www.jianshu.com/p/6e38e1ef203a



### 4.2.5 webView出现302/303重定向
- 专业叙述
    - 302重定向又称之为302代表暂时性转移
- 网络解释
    - 重定向是网页制作中的一个知识，几个例子跟你说明，假设你现在所处的位置是一个论坛的登录页面，你填写了帐号，密码，点击登陆，如果你的帐号密码正确，就自动跳转到论坛的首页，不正确就返回登录页；这里的自动跳转，就是重定向的意思。或者可以说，重定向就是，在网页上设置一个约束条件，条件满足，就自动转入到其它网页、网址 。比如，你输入一个网站链接，一般可以直接进入网站，如果出现错误，则又跳转到另外一个网页。
- 举个例子
    - 叙述下这种问题的情况，就是WebView首先加载A链接，然后在WebView上点击一个B链接进行加载，B链接会自动跳转到C链接，这个时候调用WebView的goback方法，会返回到加载B链接，但是B链接又会跳转到C链接，从而导致没法返回到A链接界面（当然也有朋友说快速的按两次返回键－也就是连续触发了两次goback可以返回到A链接，但并不是所有用户都懂这个，而且操作上也很恶心。），这就是重定向问题。
- 实现WebView的滑动监听和优雅处理回退栈问题
    - WebView能否知道某个url是不是301/302呢？当然知道，WebView能够拿到url的请求信息和响应信息，根据header里的code很轻松就可以实现，事实正是如此，交给WebView来处理重定向(return false)，这时候按返回键，是可以正常地回到重定向之前的那个页面的。（PS：从上面的章节可知，WebView在5.0以后是一个独立的apk，可以单独升级，新版本的WebView实现肯定处理了重定向问题）
    - 但是，业务对url拦截有需求，肯定不能把所有的情况都交给系统WebView处理。为了解决url拦截问题，本文引入了另一种思想——通过用户的touch事件来判断重定向。具体可以看项目lib中的ScrollWebView！


### 4.2.6 onPageFinished偶发不执行


### 4.2.8 onReceiveError问题
- https://www.jianshu.com/p/fcebd23cbebb



### 4.2.9 loadUrl在19以上超过2097152个字符失效
- 修改代码如下所示
    ```
    /**
    * loadUrl方法在19以上超过2097152个字符失效
    */
    private static final int URL_MAX_CHARACTER_NUM=2097152;
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT &&
            javascriptCommand.length()>=URL_MAX_CHARACTER_NUM) {
        this.evaluateJavascript(javascriptCommand,null);
    }else {
        this.loadUrl(javascriptCommand);
    }
    ```


### 4.3.0 WebViewJavascriptBridge: WARNING
- 有时候会出现这种报错日志：WebViewJavascriptBridge: WARNING: javascript handler threw.", source: (1) 
    - 发生这种错误的原因。在作者库的library下的assets中有个js文件，错误是在那里面出现的。
    - 这里总结如下：两边互传数据最好做的是严格意义上的json字符串。最后一点，也是最重要的一点。
        ```
        try {
            handler(message.data, responseCallback);
        } catch (exception) {
            if (typeof console != 'undefined') {
                console.log("WebViewJavascriptBridge: WARNING: javascript handler threw.", message, exception);
            }
        }
        ```
    - 上面的错误，是发生在WebViewJavascriptBridge.js的这里。。这里try catch发生错误。handler是js端给到的处理函数，也就是在js端的这个处理函数里发生任何错误，都会出现这个错误提示。导致大家无法获取准确的错误。
- 出现这个错误。
    - 首先检查js端的代码。js端发生错误，就报上面的错误。所以首要是去找js端的错误。





