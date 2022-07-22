## 1、Java和Groovy对比
### 1.1、Hello Groovy!
从一个简单的案例开始：
```Groovy
class A01_GroovyApp {

    public static void hello() {
        println('Hello Groovy!')
    }

    public static void main(String[] args) {
        hello()
    }
}
```
运行结果
```
Hello Groovy!
```

### 1.2、如何实现循环
简单的，可以类似Java一样这样写：

```groovy
Stream.of(1, 2, 3, 4)
        .forEach(new Consumer<Integer>() {
    @Override
    void accept(Integer integer) {
        println("Java print" + integer)
    }
})
```

运行结果
> Java print1
Java print2
Java print3
Java print4

也可以这样写
```groovy    
for (i in 0..4)  {    //方式1
    print "$i"
}

println()

0.upto(4) { print "$it" }  //方式2

println()

3.times { print("$it") }   //方式3

println()

0.step(10, 2) { print("$it") }   //方式4
```
运行结果：
>01234
01234
012
02468

### 1.3、初识GDK

Java中，假如我们想要在代码中调用SVN的help，我们应该这样实现Java代码：

![SVN_Java.png](https://upload-images.jianshu.io/upload_images/7293029-232345d66d1aa58d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在groovy中，我们只需要这样实现：

```groovy
println "svn help".execute().text

//或者执行 groovy -v命令查询groovy的版本
println "groovy -v".execute().text

//或者执行 ls -l命令
println "ls -l".execute().text
```
结果：
> java.lang.UNIXProcess

> Groovy Version: 2.4.15 JVM: 1.8.0_131 Vendor: Oracle Corporation OS: Mac OS X

> total 128
-rw-r--r--  1 qing.mei  staff    23 Mar 29 20:11 A00_Helloworld.groovy
-rw-r--r--  1 qing.mei  staff   232 Apr  4 20:04 A01_GroovyApp.groovy
-rw-r--r--  1 qing.mei  staff    50 Mar 29 20:29 A02_GroovyScript.groovy
-rw-r--r--  1 qing.mei  staff   383 Apr  4 20:09 A03_foreach.groovy
-rw-r--r--  1 qing.mei  staff   114 Mar 30 17:33 A04_GDK.groovy
-rw-r--r--  1 qing.mei  staff    93 Mar 30 19:27 A05_Exception.groovy
-rw-r--r--  1 qing.mei  staff   250 Mar 30 19:29 A06_Wizard.groovy
-rw-r--r--  1 qing.mei  staff    29 Mar 30 19:29 A06_Wizard_Script.groovy
-rw-r--r--  1 qing.mei  staff   125 Mar 30 19:36 A07_GroovyBean.groovy
-rw-r--r--  1 qing.mei  staff   188 Mar 30 19:36 A07_GroovyBean_Script.groovy
-rw-r--r--  1 qing.mei  staff   387 Mar 30 19:36 A07_JavaBean.java
-rw-r--r--  1 qing.mei  staff   299 Apr  4 19:25 A08Calendar.groovy
-rw-r--r--  1 qing.mei  staff   188 Apr  4 19:33 A09Robot.groovy
-rw-r--r--  1 qing.mei  staff   200 Apr  4 19:31 A09RobotScript.groovy
-rw-r--r--  1 qing.mei  staff   524 Apr  4 19:44 A10Params.groovy
-rw-r--r--  1 qing.mei  staff  1540 Apr  4 20:01 A11MultipleAssignments.groovy

## 2、JavaBean
相比Java的get和set方法，我们来看Groovy的JavaBean:
```groovy
class A07_GroovyBean {

    def name = 'qingmei2'
    final year

    A07_GroovyBean(year) {
        this.year = year
    }
}
```

其操作方式：
```groovy
def groovyBean = new A07_GroovyBean(10)

println "year: $groovyBean.year"
println "name: $groovyBean.name"
println 'Setting...'
groovyBean.name = "LiHua"
println "name: $groovyBean.name"
```
执行结果：
> year: 10
name: qingmei2
Setting...
name: LiHua

同时，还可以这样节省代码：
```groovy
//我们可以这样代替Java的Calendar.getInstance()
Calendar.instance

str = 'hello'

//谨慎使用class属性，类似Map/生成器等一些类对该属性有特殊处理，因此为了避免意外，一般使用getClass()
def name = str.class.name

println "the string = $str"
println "the string class name = $name"
```

## 3、灵活初始化和具名参数
Groovy中可以灵活地初始化一个JavaBean类。在构造对象时，可以简单地以逗号分隔的名值对来给出属性值。如果类有一个无参构造器，该操作会在构造器之后执行。1也可以设计自己的方法，使其接受具名参数。要利用这一特性，需要把第一个形参定义为Map。下面通过代码来实际地看一下。
```groovy
class A09Robot {

    def type, height, width

    def access(Map location, weight, fragile) {
        println "received fragile ? $fragile, weight: $weight, location: $location"
    }
}
```

```groovy
def robot = new A09Robot(type: 'arm', width: 10, height: 40)
println "$robot.type, $robot.width, $robot.height"

robot.access(x: 30, y: 20, z: 10, 50, true)
robot.access(50, true, x: 30, y: 20, z: 10)
```
我们来看结果：
> arm, 10, 40
received fragile ? true, weight: 50, location: [x:30, y:20, z:10]
received fragile ? true, weight: 50, location: [x:30, y:20, z:10]

## 4.可选形参
参考Kotlin，我们可以给函数的参数设置默认的值：
```groovy
//groovy 可以把方法和构造器形参设置为可选择的
//但是这些形参必须位于形参列表的末尾
def log(x, base = 10) {
    Math.log(x) / Math.log(base)
}

println log(1024)
println log(1024, 10)
println log(1024, 2)

//同时，groovy还会把末尾的数组形参视作可选的
//因此，可以为最后一个形参提供0个或者多个值
def task(name, String[] details) {
    println "$name - $details"
}

task 'call', '123-456-7890'
task 'call', '123-456-7890', '234,567,890'
task 'checkmail'
```

结果：
> 3.0102999566398116
3.0102999566398116
10.0
call - [123-456-7890]
call - [123-456-7890, 234,567,890]
checkmail - []

## 5.使用多赋值
向方法传递多个参数，这在很多编程语言中都司空见惯。但是从方法返回多个结果，尽管可能非常实用，却不那么常见。

要想从方法返回多个结果，并将它们一次性赋给多个变量，我们可以返回一个数组，然后将多个变量以逗号分隔，放在圆括号中，置于赋值表达式左侧即可。

后面的例子中有一个负责将全名分割为名字（FirstName）和姓氏（LastName）的方法。不出所料，split()方法就返回一个数组。可以把splitName()的结果赋给一对变量：firstName和lastName。Groovy会把结果中的两个值分别赋给这两个变量。

```groovy
def splitFullName(String fullname) {
    fullname.split ' '
}

def (firstname, lastname) = splitFullName('James Bond')

println "$lastname, $firstname $lastname"
```

结果：
> Bond, James Bond

还可以使用该特性来交换变量，无需创建中间变量来保存被交换的值，只需将欲交换的变量放在圆括号内，置于赋值表达式左侧，同时将它们以相反顺序放于方括号内，置于右侧即可。

```groovy
//交换值的方式

def name1 = 'name1'
def name2 = 'name2'

println "$name1 and $name2"

(name1, name2) = [name2, name1]

println "$name1 and $name2"
```

结果：

> name1 and name2
  name2 and name1

此外，当变量与值的数目不匹配时，如果有多余的变量，groovy会把它们设置为null，多余的值则会被丢弃：

```groovy
//仍然是上面的名字分割案例
def (names) = splitFullName('James Bond')
println "$names"

//左侧只有两个变量，因此Spike和Tyke会被丢弃
def (String cat, String mouse) = ['Tom', 'Jerry', 'Spike', 'Tyke']
println "$cat and $mouse"

def (String first, String second, String third) = ['Tom', 'Jerry']
println "$first and $second and $third"
```
结果：

> James          //多余的值则会被丢弃
Tom and Jerry
Tom and Jerry and null    //多余的变量，groovy会把它们设置为null


