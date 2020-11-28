### 一、简单工厂模式简介

##### 1.定义

简单工厂模式\(Simple Factory Pattern\)：又称为静态工厂方法\(Static Factory Method\)模式，它属于类创建型模式。在简单工厂模式中，可以根据参数的不同返回不同类的实例。简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

##### 2.使用动机

考虑一个简单的软件应用场景：一个软件系统可以提供多个外观不同的按钮（如圆形按钮、矩形按钮、菱形按钮等）， 这些按钮都源自同一个基类，不过在继承基类后不同的子类修改了部分属性从而使得它们可以呈现不同的外观。

如果我们希望在使用这些按钮时，不需要知道这些具体按钮类的名字，只需要知道表示该按钮类的一个参数，并提供一个调用方便的方法，把该参数传入方法即可返回一个相应的按钮对象，此时，就可以使用简单工厂模式。

### 二、简单工厂模式结构

##### 1.模式结构

![](http://upload-images.jianshu.io/upload_images/3985563-5e3b5da5f25c51a6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

简单工厂模式包含如下角色：

●Factory：工厂角色  
工厂角色负责实现创建所有实例的内部逻辑

●Product：抽象产品角色  
抽象产品角色是所创建的所有对象的父类，负责描述所有实例所共有的公共接口

●ConcreteProduct：具体产品角色  
具体产品角色是创建目标，所有创建的对象都充当这个角色的某个具体类的实例。

###### 2.时序图

![](http://upload-images.jianshu.io/upload_images/3985563-1e2ced2ec4865bce.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

①先调用工厂类中的静态方法createProduct\(\)

②根据传入产品类型参数，获得具体的产品对象

③返回产品对象并使用

### 三、简单工厂的使用实例

以登录功能来说，假如应用系统需要支持多种登录方式如：口令认证、域认证（口令认证通常是去数据库中验证用户，而域认证则是需要到微软的域中验证用户）。那么自然的做法就是建立一个各种登录方式都适用的接口，如下图所示：

![](http://upload-images.jianshu.io/upload_images/3985563-c40f9e18f891d00a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

**抽象产品Login**

```java
public interface Login {
    //登录验证
    public boolean verify(String name , String password);
}
```

**具体产品DomainLogin**

```java
public class DomainLogin implements Login {

    @Override
    public boolean verify(String name, String password) {
        // TODO Auto-generated method stub
        /**
         * 业务逻辑
         */
        return true;
    }
}
```

**具体产品PasswordLogin**

```java
public class PasswordLogin implements Login {

    @Override
    public boolean verify(String name, String password) {
        // TODO Auto-generated method stub
        /**
         * 业务逻辑
         */
        return true;
    }
}
```

**工厂类LoginManager**  
根据调用者不同的要求，创建出不同的登录对象并返回。而如果碰到不合法的要求，会返回一个Runtime异常。

```java
public class LoginManager {
    public static Login factory(String type){
        if(type.equals("password")){

            return new PasswordLogin();

        }else if(type.equals("passcode")){

            return new DomainLogin();

        }else{
            /**
             * 这里抛出一个自定义异常会更恰当
             */
            throw new RuntimeException("没有找到登录类型");
        }
    }
}
```

**测试调用**

```java
public class Test {
    public static void main(String[] args) {
        String loginType = "password";
        String name = "name";
        String password = "password";
        Login login = LoginManager.factory(loginType);
        boolean bool = login.verify(name, password);
        if (bool) {
            /**
             * 业务逻辑
             */
        } else {
            /**
             * 业务逻辑
             */
        }
    }
}
```

假如不使用简单工厂模式则验证登录Servlet代码如下：

```java
public class Test {
    public static void main(String[] args) {
        // TODO Auto-generated method stub

        String loginType = "password";
        String name = "name";
        String password = "password";
        //处理口令认证
        if(loginType.equals("password")){
            PasswordLogin passwordLogin = new PasswordLogin();
            boolean bool = passwordLogin.verify(name, password);
            if (bool) {
                /**
                 * 业务逻辑
                 */
            } else {
                /**
                 * 业务逻辑
                 */
            }
        }
        //处理域认证
        else if(loginType.equals("passcode")){
            DomainLogin domainLogin = new DomainLogin();
            boolean bool = domainLogin.verify(name, password);
            if (bool) {
                /**
                 * 业务逻辑
                 */
            } else {
                /**
                 * 业务逻辑
                 */
            }    
        }else{
            /**
             * 业务逻辑
             */
        }
    }
}
```

可以看到非常麻烦，代码重复很多，而且不利于扩展维护。

### 四、简单工厂模式优缺点

##### 优点：

通过使用工厂类，外界不再需要关心如何创造各种具体的产品，只要提供一个产品的名称作为参数传给工厂，就可以直接得到一个想要的产品对象，并且可以按照接口规范来调用产品对象的所有功能（方法）。

构造容易，逻辑简单。

##### 缺点：

1.简单工厂模式中的if else判断非常多，完全是Hard Code，如果有一个新产品要加进来，就要同时添加一个新产品类，并且必须修改工厂类，再加入一个 else if 分支才可以， 这样就违背了 “开放-关闭原则”中的对修改关闭的准则了。当系统中的具体产品类不断增多时候，就要不断的修改工厂类，对系统的维护和扩展不利。

2.一个工厂类中集合了所有的类的实例创建逻辑，违反了高内聚的责任分配原则，将全部的创建逻辑都集中到了一个工厂类当中，所有的业务逻辑都在这个工厂类中实现。什么时候它不能工作了，整个系统都会受到影响。因此一般只在很简单的情况下应用，比如当工厂类负责创建的对象比较少时。

3.简单工厂模式由于使用了静态工厂方法，造成工厂角色无法形成基于继承的等级结构。

###### 适用环境

在以下情况下可以使用简单工厂模式：

工厂类负责创建的对象比较少：由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂。

客户端只知道传入工厂类的参数，对于如何创建对象不关心：客户端既不需要关心创建细节，甚至连类名都不需要记住，只需要知道类型所对应的参数。

### 五、简单工厂模式在Java中的应用

①JDK类库中广泛使用了简单工厂模式，如工具类java.text.DateFormat，它用于格式化一个本地日期或者时间。

```java
public final static DateFormat getDateInstance();
public final static DateFormat getDateInstance(int style);
public final static DateFormat getDateInstance(int style,Locale
locale);
```

②Java加密技术  
获取不同加密算法的密钥生成器:

```java
KeyGenerator keyGen=KeyGenerator.getInstance("DESede");
```

创建密码器:

```java
Cipher cp=Cipher.getInstance("DESede");
```



