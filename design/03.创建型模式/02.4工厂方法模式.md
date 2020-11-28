### 一、工厂方法模式简介

##### 1.定义

工厂方法模式\(Factory Method Pattern\)又称为工厂模式，也叫虚拟构造器\(Virtual Constructor\)模式或者多态工厂\(Polymorphic Factory\)模式，它属于类创建型模式。

在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

###### **2.使用动机**

现在对该系统（上篇文章提到）进行修改，不再设计一个按钮工厂类来统一负责所有产品的创建，而是将具体按钮的创建过程交给专门的工厂子类去完成。

我们先定义一个抽象的按钮工厂类，再定义具体的工厂类来生成圆形按钮、矩形按钮、菱形按钮等，它们实现在抽象按钮工厂类中定义的方法。这种抽象化的结果使这种结构可以在不修改具体工厂类的情况下引进新的产品，如果出现新的按钮类型，只需要为这种新类型的按钮创建一个具体的工厂类就可以获得该新按钮的实例，这一特点无疑使得工厂方法模式具有超越简单工厂模式的优越性，更加符合“开闭原则”。

### 二、工厂方法模式结构

##### 1.模式结构

![](http://upload-images.jianshu.io/upload_images/3985563-8ce4534a7a872a09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  


  
工厂方法模式包含如下角色：

●Product：抽象产品，工厂方法模式所创建的对象的超类，也就是所有产品类的共同父类或共同拥有的接口。在实际的系统中，这个角色也常常使用抽象类实现。

●ConcreteProduct：具体产品，这个角色实现了抽象产品（Product）所声明的接口，工厂方法模式所创建的每一个对象都是某个具体产品的实例。

●Factory：抽象工厂，担任这个角色的是工厂方法模式的核心，任何在模式中创建对象的工厂类必须实现这个接口。在实际的系统中，这个角色也常常使用抽象类实现。

●ConcreteFactory：具体工厂，担任这个角色的是实现了抽象工厂接口的具体Java类。具体工厂角色含有与业务密切相关的逻辑，并且受到使用者的调用以创建具体产品对象。

##### 2.时序图

![](http://upload-images.jianshu.io/upload_images/3985563-bb6ceace257078ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  


  
①先调用具体工厂对象中的方法createProduct\(\)

②根据传入产品类型参数（也可以无参），获得具体的产品对象

③返回产品对象并使用

### 三、工厂方法模式的使用实例

![](http://upload-images.jianshu.io/upload_images/3985563-c5a708a3fbf93c26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  


  
上面的类图中，在灯这个品类下，有**灯泡**和**灯管**两种产品，并且都实现了灯的通用方法：关灯和开灯。在工厂类下，有各种生产具体产品的子工厂负责生产相应的两种灯具。

如果还不是太明白，那我们来假设一个情景。小明（客户端）想要买一个灯泡，他不认识工厂，只能去供销店（工厂类）买，于是和老板说“我要一个灯泡”，老板说 “没问题！您稍等”。转身到了后院，对生产灯泡的小弟（灯泡工厂子类）吆喝一声，给我造个灯泡！不一会灯泡造好了，老板拿给小明，“嘿嘿，灯泡给您作了一个，您试试？”，小明把灯泡拧在灯口上，开关了两下（灯的通用方法）“嘿！挺好，没问题！”，付了钱高高兴兴走了。

  
**抽象的产品接口ILight**

```java
public interface ILight
{
    void TurnOn();
    void TurnOff();
}
```

**具体的产品类：BulbLight**

```java
public class BulbLight implements ILight
{
    public void TurnOn()
    {
        Console.WriteLine("BulbLight turns on.");
    }
    public void TurnOff()
    {
        Console.WriteLine("BulbLight turns off.");
    }
}
```

**具体的产品类：TubeLight**

```java
public class TubeLight implements ILight
{
    public void TurnOn()
    {
        Console.WriteLine("TubeLight turns on.");
    }

    public void TurnOff()
    {
        Console.WriteLine("TubeLight turns off.");
    }

}
```

**抽象的工厂类**

```java
public interface ICreator
{
    ILight CreateLight();
}
```

**具体的工厂类:BulbCreator**

```java
public class BulbCreator implements ICreator
{
    public ILight CreateLight()
    {
        return new BulbLight();
    }

}
```

**具体的工厂类:TubeCreator**

```java
public class TubeCreator implements ICreator
{
    public ILight CreateLight()
    {
        return new TubeLight();
    }
}
```

**客户端调用**

```java
static void Main(string[] args)
{
    //先给我来个灯泡
    ICreator creator = new BulbCreator();
    ILight light = creator.CreateLight();
    light.TurnOn();
    light.TurnOff();

    //再来个灯管看看
    creator = new TubeCreator();
    light = creator.CreateLight();
    light.TurnOn();
    light.TurnOff();

}
```

通过一个引用变量ICreator来创建产品对象，创建何种产品对象由指向的具体工厂类决定。通过工厂方法模式，将具体的应用逻辑和产品的创建分离开，促进松耦合。

本例中每个具体工厂类只负责生产一种类型的产品，当然每个具体工厂类也内部可以维护少数几种产品实例对象，类似于简单工厂模式。

### 四、工厂方法模式的优缺点

##### 优点

①在工厂方法模式中，工厂方法用来创建客户所需要的产品，同时还向客户隐藏了哪种具体产品类将被实例化这一细节，用户只需要关心所需产品对应的工厂，无须关心创建细节，甚至无须知道具体产品类的类名。

②基于工厂角色和产品角色的多态性设计是工厂方法模式的关键。它能够使**工厂可以自主确定创建何种产品对象**，而如何创建这个对象的细节则完全封装在具体工厂内部。工厂方法模式之所以又被称为多态工厂模式，是因为所有的具体工厂类都具有同一抽象父类。

③使用工厂方法模式的另一个优点是**在系统中加入新产品时**，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，而**只要添加一个具体工厂和具体产品就可以了**。这样，系统的可扩展性也就变得非常好，**完全符合“开闭原则”**，这点比简单工厂模式更优秀。

##### 缺点

①在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，**系统中类的个数将成对增加，在一定程度上增加了系统的复杂度**，有更多的类需要编译和运行，会给系统带来一些额外的开销。

②由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度，且在实现时可能需要用到DOM、反射等技术，增加了系统的实现难度。

##### 适用场景

在以下情况下可以使用工厂方法模式：

①一个类不知道它所需要的对象的类：在工厂方法模式中，客户端不需要知道具体产品类的类名，只需要知道所对应的工厂即可，具体的产品对象由具体工厂类创建；客户端需要知道创建具体产品的工厂类。

②一个类通过其子类来指定创建哪个对象：在工厂方法模式中，对于抽象工厂类只需要提供一个创建产品的接口，而由其子类来确定具体要创建的对象，利用面向对象的多态性和里氏代换原则，在程序运行时，子类对象将覆盖父类对象，从而使得系统更容易扩展。

③将创建对象的任务委托给多个工厂子类中的某一个，客户端在使用时可以无须关心是哪一个工厂子类创建产品子类，需要时再动态指定，可将具体工厂类的类名存储在配置文件或数据库中。

### 五、工厂方法模式在Java中应用

JDBC中的工厂方法:

```java
Connection conn=DriverManager.getConnection("jdbc:microsoft:sqlserver://localhost:1433; DatabaseName=DB;user=sa;password=");
Statement statement=conn.createStatement();
ResultSet rs=statement.executeQuery("select * from UserInfo");
```