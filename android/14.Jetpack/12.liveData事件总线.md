# liveData实现事件总线
#### 目录介绍
- 01.先提出问题思考
- 02.该liveDataBus优势
- 03.EventBus使用原理
- 04.RxBus使用原理
- 05.三者之间优缺点对比
- 06.LiveDataBus的组成
- 07.LiveDataBus原理图
- 08.该库使用api方法
- 09.事件消息系列博客
- 10.遇到问题思考汇总


### 00.该库的功能
- 该库代码量少，利用LiveData实现事件总线，替代EventBus。充分利用了生命周期感知功能，可以在activities, fragments, 或者 services生命周期是活跃状态时更新这些组件。支持发送普通事件，也可以发送粘性事件；还可以发送延迟消息，以及轮训延迟消息等等。



### 01.先提出问题思考
- 能否利用LiveData的特有功能实现类似EventBus事件总线的功能？
- 针对EventBus，RxBus，为何要弄LiveDataBus，它具有那些优势？
- 如何利用LiveData生命周期感知这个功能，在实现事件总线上注意那些问题？
- 怎样实现粘性和非粘性的事件消息，能否弄个延迟发送事件的功能，以及轮训发送事件功能？
- 事件总线如果在订阅或者发送中某个环节抛出异常，如何才能不影响后续的功能？



### 02.该liveDataBus优势
- 1.该LiveDataBus的实现比较简单，支持发送普通事件，也支持发送粘性事件；
- 2.该LiveDataBus支持发送延迟事件消息，也可以用作轮训延迟事件(比如商城类项目某活动页面5秒钟刷一次接口数据)，支持stop轮训操作
- 3.该LiveDataBus可以减小APK包的大小，由于LiveDataBus只依赖Android官方Android Architecture Components组件的LiveData；
- 4.该LiveDataBus具有生命周期感知，这个是一个很大的优势。不需要反注册，避免了内存泄漏等问题；



### 03.EventBus使用原理
- 框架的核心思想，就是消息的发布和订阅，使用订阅者模式实现，其原理图大概如下所示，摘自网络。
    - ![image](https://img-blog.csdnimg.cn/20200305174827897.png)
- 发布和订阅之间的依赖关系，其原理图大概如下所示，摘自网络。
    - ![image](https://img-blog.csdnimg.cn/20200305174749401.png)
- 订阅/发布模式和观察者模式之间有着微弱的区别，个人觉得订阅/发布模式是观察者模式的一种增强版。两者区别如下所示，摘自网络。
    - ![image摘自网络](https://img-blog.csdnimg.cn/20200305174854437.png)
- LiveDataBus的组成
    - 消息： 消息可以是任何的 Object，可以定义不同类型的消息，如 Boolean、String。也可以定义自定义类型的消息。
    - 消息通道： LiveData 扮演了消息通道的角色，不同的消息通道用不同的名字区分，名字是 String 类型的，可以通过名字获取到一个 LiveData 消息通道。
    - 消息总线： 消息总线通过单例实现，不同的消息通道存放在一个 HashMap 中。
    - 订阅： 订阅者通过 with() 获取消息通道，然后调用 observe() 订阅这个通道的消息。
    - 发布： 发布者通过 with() 获取消息通道，然后调用 setValue() 或者 postValue() 发布消息。



### 04.RxBus使用原理
- RxBus不是一个库，而是一个文件，实现只有短短30行代码。RxBus本身不需要过多分析，它的强大完全来自于它基于的RxJava技术。
- 在RxJava中有个Subject类，它继承Observable类，同时实现了Observer接口，因此Subject可以同时担当订阅者和被订阅者的角色，我们使用Subject的子类PublishSubject来创建一个Subject对象（PublishSubject只有被订阅后才会把接收到的事件立刻发送给订阅者），在需要接收事件的地方，订阅该Subject对象，之后如果Subject对象接收到事件，则会发射给该订阅者，此时Subject对象充当被订阅者的角色。
- 完成了订阅，在需要发送事件的地方将事件发送给之前被订阅的Subject对象，则此时Subject对象作为订阅者接收事件，然后会立刻将事件转发给订阅该Subject对象的订阅者，以便订阅者处理相应事件，到这里就完成了事件的发送与处理。
- 最后就是取消订阅的操作了，RxJava中，订阅操作会返回一个Subscription对象，以便在合适的时机取消订阅，防止内存泄漏，如果一个类产生多个Subscription对象，我们可以用一个CompositeSubscription存储起来，以进行批量的取消订阅。



### 05.三者之间优缺点对比
- 对比结果如下所示
    | 事件总线 | 发送粘性事件 | 是否有序接收消息 | 延迟发送 | 组建生命周期感知 | 跨线程发事件 |
    | :------ | :--------- | :------------- | :------ | :-------------- | :-------- | 
    | LiveDataBus | true  | true         | true         |true         |false         |
    | EventBus | true     | true          | false       |false        |true         |
    | RxBus | true        | false       | false         |false        |true         |
- EventBus 是业界知名的通信类总线库，存在许多被人诟病的缺点：
    - 需要手动的注册和反注册，稍不小心可能会造成内存泄露。
    - 使用 EventBus 出错时难以跟踪出错的事件源。
    - 每个事件都要定义一个事件类，容易造成类膨胀。
    - 无法感知activity或者fragment生命周期，页面不可见时就处理收到消息逻辑
- 通过 LiveData 实现的 LiveDataBus 的具有以下优点：
    - 具有生命周期感知能力，不用手动注册和反注册。
    - 具有唯一的可信事件源。
    - 以字符串区分每一个事件，避免类膨胀。
    - LiveData 为 Android 官方库，更加可靠。



### 06.LiveDataBus的组成
- **消息**： 消息可以是任何的 Object，可以定义不同类型的消息，如 Boolean、String。也可以定义自定义类型的消息。
- **消息通道**： LiveData 扮演了消息通道的角色，不同的消息通道用不同的名字区分，名字是 String 类型的，可以通过名字获取到一个 LiveData 消息通道。
- **消息总线**： 消息总线通过单例实现，不同的消息通道存放在一个 HashMap 中。
- **订阅**： 订阅者通过 with() 获取消息通道，然后调用 observe() 订阅这个通道的消息。
- **发布**： 发布者通过 with() 获取消息通道，然后调用 setValue() 或者 postValue() 发布消息。



### 07.LiveDataBus原理图
#### 7.1 订阅和注册的流程图
- ![image](https://img-blog.csdnimg.cn/20200305173032750.jpg)


#### 7.1 订阅注册原理图
![image](https://img-blog.csdnimg.cn/2020030517313021.jpg)




### 08.该库使用api方法
#### 8.0 添加依赖库
- 依赖代码如下所示
    ```
    implementation 'cn.yc:LiveBusLib:1.0.2'
    ```


#### 8.1 最简单常见的发布/订阅事件消息
- 订阅事件，该种使用方式不需要取消订阅
    ```
    LiveDataBus.get()
            .with(Constant.YC_BUS, String.class)
            .observe(this, new Observer<String>() {
                @Override
                public void onChanged(@Nullable String newText) {
                    // 更新数据
                    BusLogUtils.d("接收消息--------yc_bus---2-"+newText);
                }
            });
    ```
- 那么如何发布事件消息呢，代码如下
    ```
    LiveDataBus.get().with(Constant.YC_BUS).setValue("test_yc_data");
    LiveDataBus.get().with(Constant.YC_BUS).postValue("test_yc_data");
    ```


#### 8.2 Forever模式订阅和取消订阅消息【一直会收到通知】
- 订阅事件，该种使用方式不需要取消订阅
    ```
    private Observer<String> observer = new Observer<String>() {
        @Override
        public void onChanged(@Nullable String s) {
            Toast.makeText(ThirdActivity.this, s, Toast.LENGTH_SHORT).show();
        }
    };
    
    LiveDataBus.get()
            .with(Constant.LIVE_BUS, String.class)
            .observeForever(observer);
    ```
- 这个时候需要手动移除observer观察者
    ```
    LiveDataBus.get()
           .with(Constant.LIVE_BUS, String.class)
           .removeObserver(observer);
    ```
- 发送消息同上一样，需要注意，发送这种消息，需要手动移除观察者，它表示不管在什么状态下都会收到发出的消息事件。



#### 8.3 发送粘性事件消息
- 订阅事件，该种使用方式不需要取消订阅
    ```
    LiveDataBus.get()
            .with(Constant.YC_BUS, String.class)
            .observeSticky(this, new Observer<String>() {
                @Override
                public void onChanged(@Nullable String s) {
                    // 更新数据
                    BusLogUtils.d("接收消息--------yc_bus---8----"+s);
                }
            });
    ```
- 那么如何发布事件消息呢，代码如下
    ```
    LiveDataBus.get().with(Constant.YC_BUS).setValue("test_yc_data");
    LiveDataBus.get().with(Constant.YC_BUS).postValue("test_yc_data");
    ```
- observeForever模式订阅消息，需要调用removeObserver取消订阅
    ```
    LiveDataBus.get()
            .with(Constant.YC_BUS, String.class)
            .observeStickyForever(observer);
    ```


#### 8.4 如何发送延迟消息
- 该lib拥有延迟发送消息事件的功能，发送事件消息代码如下
    ```
    //延迟5秒发送事件消息
    LiveDataBus.get().with(Constant.LIVE_BUS).postValueDelay("test_data",5000);
    ```


#### 8.5 如何发送轮训延迟消息
- 这种场景主要应用在购物类的需求中，比如一个活动页面，每个5秒刷新一下接口数据更新页面活动
    ```
    //开始轮训
    LiveDataBus.get().with(Constant.LIVE_BUS5).postValueInterval("test_data",3000, "doubi");
    
    //停止轮训
    LiveDataBus.get().with(Constant.LIVE_BUS5).stopPostInterval("doubi");
    ```



### 09.事件消息系列博客
- [01.EventBus](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/01.EventBus.md)
    - 01.EventBus简单介绍
    - 02.EventBus简单使用
    - 03.EventBus优缺点
    - 04.什么是发布/订阅模式
    - 05.EventBus实现原理
    - 06.EventBus重大问题
- [02.RxBus](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/02.RxBus.md)
    - 01.RxBus是什么
    - 02.RxBus原理是什么
    - 03.RxBus简单实现
    - 04.RxBus优质库
    - 05.简单使用代码案例
- [03.LiveData简单介绍](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/03.LiveData简单介绍.md)
    - 01.LiveData是什么东西
    - 02.为何要使用LiveData
    - 03.使用LiveData的优势
    - 04.使用LiveData的步骤
    - 05.简单使用LiveData
    - 06.observe()和observerForever()
    - 07.理解活跃状态更新数据
    - 08.setValue和postValue
- [04.LiveDataBus](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/04.LiveDataBus.md)
    - 01.为何使用liveData
    - 02.LiveDataBus的组成
    - 03.LiveDataBus原理图
    - 04.简单的实现案例代码
    - 05.遇到的问题和分析思路
    - 06.使用反射解决遇到问题
    - 07.使用postValue的bug
    - 08.如何发送延迟事件消息
    - 09.如何发送轮训延迟事件
- [05.EventBus源码分析](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/05.EventBus源码分析.md)
    - 01.EventBus注册源码解析
    - 02.EventBus事件分发解析
    - 03.EventBus取消注册解析
    - 04.总结一下EventBus的工作原理
- [06.RxBus源码分析](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/06.RxBus源码分析.md)
    - 01.后续更新
- [07.LiveData源码分析](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/07.LiveData源码分析.md)
    - 01.LiveData的原理介绍
    - 02.然后思考一些问题
    - 03.observe订阅源码分析
    - 04.setValue发送源码分析
    - 05.observeForever源码
    - 06.LiveData源码总结
- [08.Lifecycle源码分析](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/08.Lifecycle源码分析.md)
    - 01.Lifecycle的作用是什么
    - 02.Lifecycle的简单使用
    - 03.Lifecycle的使用场景
    - 04.如何实现生命周期感知
    - 05.注解方法如何被调用
- [09.观察者模式](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/09.观察者模式.md)
- [10.事件总线封装库](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/10.事件总线封装库.md)
- [11.问题思考大汇总](https://github.com/yangchong211/YCLiveDataBus/blob/master/read/11.问题思考大汇总.md)



### 10.遇到问题思考汇总


### 11.其他内容介绍
#### 11.1 参考内容
- https://github.com/bennidi/mbassador
- https://github.com/zalando/nakadi
- https://github.com/JeremyLiao/SmartEventBus
- https://github.com/pwittchen/NetworkEvents
- https://github.com/sunyatas/NetStatusBus


#### 11.2 开源项目推荐
- [1.开源博客汇总](https://github.com/yangchong211/YCBlogs)
- [2.组件化实践项目](https://github.com/yangchong211/LifeHelper)
- [3.视频播放器封装库](https://github.com/yangchong211/YCVideoPlayer)
- [4.状态切换管理器封装库](https://github.com/yangchong211/YCStateLayout)
- [5.复杂RecyclerView封装库](https://github.com/yangchong211/YCRefreshView)
- [6.弹窗封装库](https://github.com/yangchong211/YCDialog)
- [7.版本更新封装库](https://github.com/yangchong211/YCUpdateApp)
- [8.状态栏封装库](https://github.com/yangchong211/YCStatusBar)
- [9.轻量级线程池封装库](https://github.com/yangchong211/YCThreadPool)
- [10.轮播图封装库](https://github.com/yangchong211/YCBanner)
- [11.音频播放器](https://github.com/yangchong211/YCAudioPlayer)
- [12.画廊与图片缩放控件](https://github.com/yangchong211/YCGallery)
- [13.Python多渠道打包](https://github.com/yangchong211/YCWalleHelper)
- [14.整体侧滑动画封装库](https://github.com/yangchong211/YCSlideView)
- [15.Python爬虫妹子图](https://github.com/yangchong211/YCMeiZiTu)
- [17.自定义进度条](https://github.com/yangchong211/YCProgress)
- [18.自定义折叠和展开布局](https://github.com/yangchong211/YCExpandView)
- [19.商品详情页分页加载](https://github.com/yangchong211/YCShopDetailLayout)
- [20.在任意View控件上设置红点控件](https://github.com/yangchong211/YCRedDotView)
- [21.仿抖音一次滑动一个页面播放视频库](https://github.com/yangchong211/YCScrollPager)




#### 11.3 其他推荐
- 博客笔记大汇总【15年10月到至今】，包括Java基础及深入知识点，Android技术博客，Python学习笔记等等，还包括平时开发中遇到的bug汇总，当然也在工作之余收集了大量的面试题，长期更新维护并且修正，持续完善……开源的文件是markdown格式的！同时也开源了生活博客，从12年起，积累共计47篇[近100万字]，转载请注明出处，谢谢！
- 链接地址：https://github.com/yangchong211/YCBlogs
- 如果觉得好，可以star一下，谢谢！当然也欢迎提出建议，万事起于忽微，量变引起质变！



#### 11.4 关于LICENSE
```
Copyright 2017 yangchong211（github.com/yangchong211）

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```





















