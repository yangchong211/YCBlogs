#### 目录介绍
- 1.基础使用介绍
    - 1.1 常量和变量
    - 1.2 函数[相当于java方法]
    - 1.3 构造方法
    - 1.4 空安全
    - 1.5 类的定义
    - 1.6 类继承
    - 1.7 数据类
    - 1.8 接口定义
    - 1.9 扩展函数
- 2.Object 表达式
    - 2.1 object 对象声明
- 3.对象表达式和声明
    - 3.1 匿名内部类
    - 3.2 单利模式
    - 3.3 静态属性、常量或者函数
    - 3.4 在kotlin中没有new关键字
- 4.Lambda表达式
    - 4.1 Lambda作用
    - 4.2 Lambda的使用场景
- 5.常用表达式
    - 5.1 When表达式
    - 5.2 for表达式
    - 5.3 while表达式
- 6.函数的操作
    - 6.1 with函数
    - 6.2 内联函数
- 7.


### 0.问题答疑部分
- 0.0.1 汇总Kotlin相对于Java有哪些优势？



### 1.基础使用介绍
#### 1.1 常量和变量
##### 1.1.1 常量
- 在 kotlin 中一切皆为对象，没有像 Java 中的原始基本类型。在 kotlin 中使用 var 修饰的为变量。例如我们定义一个 Int 类型的变量并赋值为1：
- 由于 kotlin 编译器可以自动推断出变量的类型，所以我们通常不需要指定变量的类型：
```
private var index: Int = 0      //定义具体的类型
private var a = 0           //自动识别是int类型，通常不需要指定变量的类型
```


##### 1.1.2 变量
- 在 kotlin 中使用 val 修饰的为常量。这和 java 中的 final 很相似。在 kotlin 中有一个重要的概念是：尽可能地使用 val。
```
val s = "String"    //类型为String
val ll = 22L        //类型为Long
val d = 2.5         //类型为Double
val f = 5.5F        //类型为Float
```


#### 1.2 函数[相当于java方法]
##### 1.2.1 无返回值的函数
- 没有返回值，对应java中的void

```
Unit 表示无返回值，对应 java 中 void：
fun yc(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}

Unit 的返回类型可以省略：
fun yc(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```


##### 1.2.2 有返回值的函数

```
override fun getContentView(): Int {
    return R.layout.activity_wan_android
}
```


#### 1.3 构造方法




#### 1.4 空安全
##### 1.4.1 关于空安全
- 在 kotlin 中，默认定义的变量不能为 null 的，这可以避免很多的 NullPointerException。
```
var b: String? = "abc"
b = null
指定一个变量可null是通过在类型的最后增加一个问号：

var b: String? = "abc"
val l = b.length //编译错误
当变量声明为可空时，在调用它的属性时无法通过编译

var b: String? = "abc"
val l = b?.length 
可以使用安全操作符 ?.
```

##### 1.4.2 ?.和!!.和?=各自的含义
- ?:    使用 ?: 操作符，当前面的值不为空取前面的值，否则取后面的值，这和java中三目运算符类似
- !!.   使用 !! 操作符可以跳过限制检查通过编译，此时如果变量为空会抛出空指针异常。
- ?.    使用 ?. 操作符，就先判空，如果不为空则赋值

```
//?.
//kotlin:
a?.foo()
//相当于java:
if(a!=null){
 a.foo();
}

//!!.
//kotlin:
a!!.foo()
//相当于java: 
if(a!=null){
 a.foo();
}else{
 throw new KotlinNullPointException();
}
```


#### 1.5 类的定义
- 使用 class 定义一个类。类的声明包含类名，类头(指定类型参数，主构造函数等等)，以及类主体，用大括号包裹。类头和类体是可选的；如果没有类体可以省略大括号。

##### 1.5.1 没有构造参数的类
- 无构造函数
```
class MainActivity{

}
```

##### 1.5.2 有构造参数的类
- 在 Kotlin 中类可以有一个主构造函数以及多个二级构造函数。主构造函数是类头的一部分：跟在类名后面(可以有可选的类型参数)。
- 如果主构造函数没有注解或可见性说明，则 constructor 关键字是可以省略
```
class Person constructor(name: String) {

}

//constructor 关键字是可以省略
class BannerPagerAdapter (private val ctx: Activity?){

}
```

##### 1.5.3 构造函数的函数体可以写在 init 块中
```
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```


#### 1.6 类继承
- Kotlin 中所有的类都有共同的父类 Any，它是一个没有父类声明的类的默认父类：
    - class Example //　隐式继承于 Any
    - Any 不是 java.lang.Object ；事实上它除了 equals() , hashCode() 以及 toString() 外没有任何成员了。
    - **默认情况下，kotlin 中所有的类都是不可继承 (final) 的**，所以我们只能继承那些明确声明为 open 或 abstract 的类，当我们只有单个构造器时，我们需要在从父类继承下来的构造器中指定需要的参数。这是用来替换Java中的 super 调用的。



#### 1.7 数据类
- 



#### 1.8 接口定义
- Kotlin 的接口很像 java8。它们都可以包含抽象方法，以及方法的实现。和抽象类不同的是，接口不能保存状态。可以有属性但必须是抽象的，或者提供访问器的实现。
接口用关键字 interface 来定义：
```
interface YcBar {
    fun bar()
    fun foo() {
        //函数体是可选的
    }
}
```


#### 1.9 扩展函数





### 2.Object 表达式
#### 2.1 object 对象声明
- 在Java中，单例的声明可能具有多种方式：如懒汉式、饿汉式、静态内部类、枚举等；
- 在Kotlin中，单例模式的实现只需要一个 object 关键字即可；

```
// Kt文件中的声明方式： object 关键字声明,其内部不允许声明构造方法
object SingleObject {
    fun test() {
        //...
    }
}
// 调用方式：类名.方法名()
class Main {
    fun test() {
        SingleObject.test() //在class文件中，会自动编译成SingleObject.INSTANCE.test();调用方式
    }
}

// ----------------源码和字节码分界线 ---------------
//Kotlin文件编译后的class代码如下：
public final class SingleObject {
   public static final SingleObject INSTANCE;

   public final void test() {
   }

   private SingleObject() {
      INSTANCE = (SingleObject)this;
   }

   static {
      new SingleObject();
   }
}
```


### 3.对象表达式和声明
#### 3.1 匿名内部类
- 有时候我们想要创建一个对当前类有一点小修改的对象，但不想重新声明一个子类。java 用匿名内部类的概念解决这个问题。Kotlin 用对象表达式和对象声明巧妙的实现了这一概念。

```
window.addMouseListener(object: MouseAdapter () {
    override fun mouseClicked(e: MouseEvent) {
        //...
    }
})
```

- 像 java 的匿名内部类一样，对象表达式可以访问闭合范围内的变量 (和 java 不一样的是，这些变量不用是 final 修饰的)

```
fun countClicks(window: JComponent) { 
    var clickCount = 0 
    var enterCount = 0 
    window.addMouseListener(object : MouseAdapter() { 
        override fun mouseClicked(e: MouseEvent) { 
            clickCount++ 
        }
        override fun mouseEntered(e: MouseEvent){ 
            enterCount++ 
        } 
    }) 
}
```

#### 3.2 单利模式
- 单例模式是一种很有用的模式，Kotln 中声明它很方便，其中init代码块对应java中static代码块。
- 这叫做对象声明，跟在 object关键字后面是对象名。和变量声明一样，对象声明并不是表达式，而且不能作为右值用在赋值语句。想要访问这个类，直接通过名字来使用这个类：

```
object DataProviderManager {
    init {
        //对应java中static代码块
        LogUtils.e("DataProviderManager"+"init")
    }
    fun registerDataProvider(provider: String) {
        // ...
        LogUtils.e("DataProviderManager$provider")
    }
}

//使用
// in kotlin
DataProviderManager.registerDataProvider(...)
// in java
DataProviderManager.INSTANCE.registerDataProvider(...)

注意：对象声明不可以是局部的(比如不可以直接在函数内部声明)，但可以在其它对象的声明或非内部类中进行内嵌入。
```

#### 3.3 静态属性、常量或者函数
- 我们需要一个类里面有一些静态的属性、常量或者函数，我们可以使用伴随对象。这个对象被这个类的所有对象所共享，就像java中的静态属性或者方法。在类声明内部可以用companion关键字标记对象声明：


```
class AndroidUtils{
    companion object {
        const val name = "yc"
        fun create(): AndroidUtils = AndroidUtils()
        fun getVersionCode(mContext: Context): Int {
            var versionCode = 0
            try {
                //获取软件版本号，对应AndroidManifest.xml下android:versionCode
                versionCode = mContext.packageManager.getPackageInfo(mContext.packageName, 0).versionCode
            } catch (e: PackageManager.NameNotFoundException) {
                e.printStackTrace()
            }
            return versionCode
        }
    }
}


//伴随对象的成员可以通过类名做限定词直接使用：
val create = AndroidUtils.create()
val versionCode = AndroidUtils.getVersionCode(activity)
LogUtils.e("Android$versionCode")
```


#### 3.4 在kotlin中没有new关键字
- 注意：在kotlin中没有 new 关键字。
- 对象表达式和声明的区别：
    - 对象表达式在我们使用的地方立即初始化并执行的
    - 对象声明是懒加载的，是在我们第一次访问时初始化的。
    - 伴随对象是在对应的类加载时初始化的，和 Java 的静态初始是对应的。


### 4.Lambda表达式
#### 4.1 Lambda作用
- Lambda表达式是一种很简单的方法，去定义一个匿名函数。Lambda是非常有用的，因为它们避免我们去写一些包含了某些函数的抽象类或者接口，然后在类中去实现它们。在Kotlin，我们把一个函数作为另一个函数的参数。


#### 4.2 Lambda的使用场景
- Android中非常典型的例子去解释它是怎么工作的： View.setOnClickListener() 方法。如果我们想用Java的方式去增加点击事件的回调，我首先要编写一个 OnClickListener 接口：

```
public interface OnClickListener {
    void onClick(View v);
}

//编写一个匿名内部类去实现这个接口
view.setOnClickListener(new OnClickListener(){ 
    @Override 
    public void onClick(View v) { 
        Toast.makeText(v.getContext(), "Click", Toast.LENGTH_SHORT).show(); 
    } 
});
```

- 把上面的代码转换成Kotlin

```
view.setOnClickListener(object : OnClickListener {
    override fun onClick(v: View) {
        toast("Click")
    }
}
```

- 一个lambda表达式通过参数的形式被定义在箭头的左边（普通圆括号包围），然后在箭头的右边返回结果值。当我们定义了一个方法，我们必须使用大括号包围。如果左边的参数没有用到，我们甚至可以省略左边的参数。

```
view.setOnClickListener({ view -> toast("Click")})
//或者
view.setOnClickListener({ toast("Click") })
```


### 5.常用表达式
#### 5.1 When表达式
- when 表达式与Java中的switch/case类似，但是要强大得多。这个表达式会去试图匹配所有可能的分支直到找到满意的一项。然后它会运行右边的表达式。与Java的switch/case不同之处是参数可以是任何类型，并且分支也可以是一个条件。
- 对于默认的选项，我们可以增加一个else分支，它会在前面没有任何条件匹配时再执行。条件匹配成功后执行的代码也可以是代码块：

```
/**
 * 使用when表达式
 * when表达式就相当于Java的switch表达式，省去了case和break，并且支持各种类型。
 *
 * 控制流(Control Flow)：Kotlin的控制流有if``when``for``while四种
 */
private fun selectByIndex(position: Int) {
    when (position){
        0 ->{

        }
        1 ->{

        }
        2 ->{

        }
        3 ->{

        }
        else -> {
            // 默认，相当于switch中default
        }
    }
}
```

- 因为它是一个表达式，它也可以返回一个值。我们需要考虑什么时候作为一个表达式使用，它必须要覆盖所有分支的可能性或者实现 else 分支。否则它不会被编译成功：

```
val result = when (x) {
    0, 1 -> "binary"
    else -> "error"
}
```


#### 5.2 for表达式




#### 5.3 while表达式


### 6.函数的操作
#### 6.1 with函数
- with是一个非常有用的函数，包含在Kotlin的标准库中。它接收一个对象和一个扩展函数作为它的参数，然后使这个对象扩展这个函数。这表示所有我们在括号中编写的代码都是作为对象（第一个参数）的一个扩展函数，我们可以就像作为this一样使用所有它的public方法和属性。当我们针对同一个对象做很多操作的时候这个非常有利于简化代码。

```
data class Person(val name: String, val age: Int)
val p = Person("growth",25)
with(p){
    var info = “$name - $age” 
}
```


#### 6.2 内联函数
- 下面是with函数的定义：
```
inline fun <T> with(t: T, body: T.() -> Unit) { t.body() }
```

- 这个函数接收一个 T 类型的对象和一个被作为扩展函数的函数。它的实现仅仅是让这个对象去执行这个函数。因为第二个参数是一个函数，所以我们可以把它放在圆括号外面，所以我们可以创建一个代码块，在这这个代码块中我们可以使用 this 和直接访问所有的public的方法和属性。
- 内联函数与普通的函数有点不同。一个内联函数会在编译的时候被替换掉，而不是真正的方法调用。这在一些情况下可以减少内存分配和运行时开销。举个例子，如果我们有一个函数，只接收一个函数作为它的参数。如果是一个普通的函数，内部会创建一个含有那个函数的对象。另一方面，内联函数会把我们调用这个函数的地方替换掉，所以它不需要为此生成一个内部的对象。

```
inline fun supportsLollipop(code: () -> Unit) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        code()
    }
}
```

- 它只是检查版本，然后如果满足条件则去执行。现在我们可以这么做：
```
supportsLollipop {
    window.setStatusBarColor(Color.BLACK)
}
```






























#### 参考目录
- object 对象声明：https://blog.csdn.net/love667767/article/details/79426543
- 












