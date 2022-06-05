#### 目录介绍
- 01.ServiceLocator是什么
- 02.如何快速使用
- 03.get_it不同的注册方式
- 04.覆盖注册会出现什么
- 08.获取服务的性能分析






### 01.ServiceLocator是什么
- 核心原理
    - Service Locator可以将接口（抽象基类）与具体实现分离和解耦合，同时允许通过接口从App中的任何位置访问具体实现。从而提升其隔离性和可测试性。
- get_it
    - 它是一个轻量级 ServiceLocator 库，仅仅用到了 99 行代码（包括注释），看最开始的代码。建议有时间都去阅读一下。
    - 服务不再由使用者创建，而是通过容器注入。这样我们可以不再依赖于具体的实现，而是依赖于一层薄薄的的接口。这样调用者不再知道服务具体实现细节，可以很轻松的使用 mock 数据进行替换。ServiceLocator 其实就是一种特殊的控制反转。


### 02.如何快速使用
- get_it 非常简单，使用就分两步。
    - 注册服务
    - 依赖注入
- 注册服务
    ```
      GetIt getIt = new GetIt();
    
      void register(){
        //注册模式状态管理service
        getIt.registerSingleton<BusinessService>(new BusinessServiceImpl());
        getIt.registerLazySingleton<BusinessService>(() =>new BusinessServiceImpl());
      }
    ```
- 依赖注入
    - 在需要使用到这个依赖的地方我们还是通过这个容器来获取依赖。
    ```
    var businessService = getIt<BusinessService>();
    businessService.noneBusinessPattern();
    ```
- 这样我们的服务就是在容器中创建的，在实际依赖的时候，我们可以只依赖于接口，然后通过容器注入（DI）实现了该接口的实际对象，达到了解耦的效果。


### 03.get_it不同的注册方式
- GetIt 提供了多种注册方式，这将会影响这些对象的生命周期。目前有三种：
    - 工厂模式：void registerFactory<T>(FactoryFunc<T> func) 每次都会返回新的实例。
    - 单例模式：void registerSingleton<T>(T instance) 每次返回同一实例。 这种模式需要手动初始化，就像我们上面例子中那样。
    - 单例模式（懒加载）： void registerLazySingleton<T>(FactoryFunc<T> func) 这种方式只有第一次注入依赖的时候，才会初始化服务，并且每次返回相同实例。
- Factory工厂模式
    - 传入参数是一个函数func，这个func函数要返回一个实现了类型T的对象，每次调用获取对象的方式时，都会返回一个新对象。
- 单例模式&懒加载单例
    - 单例注册时，传入一个范型T的对象或其子类的对象，如果创建这个单例时是耗时的，那么可以不在app启动的时候注册，而放到第一次使用到该服务的时候，即使用懒加载的方式。


### 04.覆盖注册会出现什么
- 如果你在容器中注册了两次同一服务的话，默认情况下会在调试模式中得到一个断言，就像下面这样。
    ```
    void setupLocator(){
      getIt.registerSingleton(NavigateService());
      getIt.registerSingleton(NavigateService());
    }
    ```
    - Failed assertion: line 53 pos 12: 'allowReassignment || !_factories.containsKey(T)': Type NavigateService is already registered
- get_it 会认为你可能是写错了，所以提醒你这里注册了两次相同服务。如果你真的必须覆盖注册，那么你可以通过设置属性 allowReassignment == true 来关闭此断言。



### 08.获取服务的性能分析
- 我们可以从 get_it 的源码中看到，这个 ServiceLocator 就是用一个 map 在储存数据。
    - final _factories = new Map<Type, _ServiceFactory<dynamic>>();
    - 所以获取服务的性能是 O(1)。

https://blog.csdn.net/win7583362/article/details/106849312