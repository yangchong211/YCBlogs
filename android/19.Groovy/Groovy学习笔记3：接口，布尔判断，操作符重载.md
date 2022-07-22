## Groovy接口

Groovy 不需要显示的通过new创建匿名内部类的实例。

```groovy
//Button对象
class Button {

    void addOnClickListener(OnClickListener listener) {
        listener.onClick()
    }

    void addOnLongClickListener(OnLongClickListener listener) {
        listener.onLongClick()
    }

}

//按钮的点击监听
interface OnClickListener {
    void onClick()
}
//长按事件监听
interface OnLongClickListener {
    void onLongClick()
}
```


调用了addOnClickListener方法，同时为该方法提供了一个代码块，借助as操作符，相当于实现了OnClickListener接口：

```groovy
def button = new Button()

listener = { println 'addListener' }

button.addOnClickListener(listener) as OnClickListener
```

输出：
> addListener

Groovy自会处理剩下的工作。它会拦截对接口中任何方法的调用，然后将调用路由到我们提供的代码块。

对于有多个方法的接口，如果打算为其所有方法提供一个相同的实现，和上面一样，不需要特殊的操作:

```groovy
def button = new Button()

listener = { println 'addListener' }

button.addOnClickListener(listener) as OnClickListener
button.addOnLongClickListener(listener) as OnLongClickListener
```

输出:
> addListener
addListener

Groovy没有强制实现接口中的所有方法：可以只定义自己关心的，而不考虑其他方法。如果剩下的方法从来不会被调用，那也就没必要去实现这些方法了。当在单元测试中通过实现接口来模拟某些行为时，这项技术非常有用。

但是在大多数实际情况下，接口中的每个方法需要不同的实现。不用担心，Groovy可以摆平。只需要创建一个映射，以每个方法的名字作为键，以方法对应的代码体作为键值，同时使用简单的Groovy风格，用冒号（:）分隔方法名和代码块即可。

不必实现所有方法，只需实现真正关心的那些即可。如果未予实现的方法从未被调用过，那么也就没有必要浪费精力去实现这些伪存根。当然，如果没提供的方法被调用了，则会出现异常：

```groovy
class Button {

    void addOnStateChangeListener(OnStateChangeListener listener) {
        listener.onCreate()
        listener.onStart()
        listener.onStop()
        listener.onDestory()
    }

}

//状态监听
interface OnStateChangeListener {

    void onCreate()

    void onStart()

    void onStop()

    void onDestroy()
}

```
```groovy
def onStateChangelistener = [
        onCreate: {
            println 'onCreate'
        },
        onStart : {
            println 'onStart'
        },
        onStop  : {
            println 'onStop'
        }  //这里只实现3个状态监听，onDestroy()并未实现
]
button.addOnStateChangeListener(onStateChangelistener as OnStateChangeListener)
```

结果：

![image.png](https://upload-images.jianshu.io/upload_images/7293029-2ffe1f4fbd6392d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 布尔值判断

Java要求if语句的条件部分必须是一个布尔表达式，比如前面例子中的if(obj!=null)和if(val>0)。

Groovy会尝试推断，如果在需要布尔值的地方放了一个对象引用，Groovy会检查该引用是否为null。它将null视作false，将非null的值视作true:

```groovy
str = 'hello'

if (str) {
    println str
}
```
结果：

![image.png](https://upload-images.jianshu.io/upload_images/7293029-47cc285014cb67a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更准确的说，表达式的结果还与对象的类型有关。例如，如果对象是一个集合（如java.util.ArrayList），那么Groovy会检查该集合是否为空。
因此，在这种情况下，只有当obj不为null，而且该集合至少包含一个元素时，表达式if(obj)才会被计算为true：

```groovy
str1 = null
println str1 ? 'str1 is not null' : 'str1 is null'

str2 = [1, 2, 3]
println str2 ? 'str2 is not null' : 'str2 is null'

str3 = []
println str3 ? 'str3 is not null' : 'str3 is null'
```

结果：

![image.png](https://upload-images.jianshu.io/upload_images/7293029-e4d9a717c63bfaa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

集合类不是唯一受到特殊对待的。那么有哪些类型将被特殊对待，Groovy又是如何计算它们的呢？

 | 类型 | 为真的条件 |
 | - | - |
 | Boolean | true |
 | Collection | 集合不为空 |
 | character | 值不为0 |
 |CharSequence|长度不为0|
 |Enumration|hasMoreElements()为true|
 |Iterator|hasNext()为true|
 |Number| Double值部位0|
 |Map|映射不为空|
 |Matcher|至少有一个匹配|
 |Object[]|长度大于0|
 |其他引用类型|引用不为null|

除了使用Groovy内建的布尔求值约定，在自己的类中，还可以通过实现asBoolean()方法来编写自己的布尔转换。

## 操作符重载

Groovy支持操作符重载，可以巧妙地应用这一点来创建DSL

比如我们可以重载++操作符，该示例映射的是String类的next()方法：

```groovy
for (ch = 'a'; ch < 'd'; ch++) {
    println ch
}
```

结果：
> a
b
c

Groovy中还可以使用简洁的for-each语法，不过两种实现都用到了String类的next()方法：

```groovy
for (ch in 'a'..'c') {
    println ch
}
```
结果同上，都是输出了abc。

要向集合中添加元素，可以使用<<操作符，该操作符会被转换为Groovy在Collection上添加的leftShift()方法:

```groovy
list = ['hello']
list << 'groovy!'
println list
```

输出：
> [hello, groovy!]

通过添加映射方法，我们可以为自己的类提供操作符，比如为+操作符添加plus()方法:

```groovy
class B03ComplexNumber {

    def real, imaginary //实部，虚部

    def plus(other) {
        new B03ComplexNumber(real: real + other.real,
                imaginary: imaginary + other.imaginary)
    }

    @Override
    String toString() {
        return "$real ${imaginary > 0 ? '+' : ''} ${imaginary}i "
    }
}
```

我们执行这段代码：
```groovy
c1 = new B03ComplexNumber(real: 1, imaginary: 4)
c2 = new B03ComplexNumber(real: 2, imaginary: 3)
println c1 + c2
```
输出：

> 3 + 7i 

ComplexNumber类重载了+操作符。对于计算涉及负数平方根的复杂方程式，复数非常有用: 复数有实部和虚部。
因为在ComplexNumber类上添加了plus()方法，所以可以使用+操作符把两个复数加到一起，得到又一个作为结果的复数。

