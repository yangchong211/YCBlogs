### 一、外观模式概述

##### 1.定义

外观模式\(Facade Pattern\)：外部与一个子系统的通信必须通过一个统一的外观对象进行，为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。外观模式又称为门面模式，它是一种对象结构型模式。

##### 2.定义阐述

医院的例子

现代的软件系统都是比较复杂的，设计师处理复杂系统的一个常见方法便是将其“分而治之”，把一个系统划分为几个较小的子系统。如果把医院作为一个子系统，按照部门职能，这个系统可以划分为挂号、门诊、划价、化验、收费、取药等。看病的病人要与这些部门打交道，就如同一个子系统的客户端与一个子系统的各个类打交道一样，不是一件容易的事情。

首先病人必须先挂号，然后门诊。如果医生要求化验，病人必须首先划价，然后缴费，才可以到化验部门做化验。化验后再回到门诊室。

![](http://upload-images.jianshu.io/upload_images/3985563-23b76827ff1c343d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

上图描述的是病人在医院里的体验，图中的方框代表医院。

解决这种不便的方法便是引进外观模式，医院可以设置一个接待员的位置，由接待员负责代为挂号、划价、缴费、取药等。这个接待员就是外观模式的体现，病人只接触接待员，由接待员与各个部门打交道。

![](http://upload-images.jianshu.io/upload_images/3985563-1d5a7ee14bb120cf.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

### 二、外观模式结构

外观模式没有一个一般化的类图描述，最好的描述方法实际上就是以一个例子说明。

![](http://upload-images.jianshu.io/upload_images/3985563-7599618df0714b21.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

由于门面模式的结构图过于抽象，因此把它稍稍具体点。假设子系统内有三个模块，分别是ModuleA、ModuleB和ModuleC，它们分别有一个示例方法，那么此时示例的整体结构图如下：

![](http://upload-images.jianshu.io/upload_images/3985563-891f17b65e68f454.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

在这个对象图中，出现了两个角色：  
●　　**外观\(Facade\)角色 ：**客户端可以调用这个角色的方法。此角色知晓相关的（一个或者多个）子系统的功能和责任。在正常情况下，本角色会将所有从客户端发来的请求委派到相应的子系统去。

●　　**子系统\(SubSystem\)角色 ：**可以同时有一个或者多个子系统。每个子系统都不是一个单独的类，而是一个类的集合（如上面的子系统就是由ModuleA、ModuleB、ModuleC三个类组合而成）。每个子系统都可以被客户端直接调用，或者被门面角色调用。子系统并不知道门面的存在，对于子系统而言，门面仅仅是另外一个客户端而已。

##### 时序图

![](http://upload-images.jianshu.io/upload_images/3985563-e8fdbd86d0684d28.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

**子系统角色中的类：**

```java
public class ModuleA {
    //示意方法
    public void testA(){
        System.out.println("调用ModuleA中的testA方法");
    }
}
```

```java
public class ModuleB {
    //示意方法
    public void testB(){
        System.out.println("调用ModuleB中的testB方法");
    }
}
```

```java
public class ModuleC {
    //示意方法
    public void testC(){
        System.out.println("调用ModuleC中的testC方法");
    }
}
```

**外观角色类:**

```java
public class Facade {
    //示意方法，满足客户端需要的功能
    public void test(){
        ModuleA a = new ModuleA();
        a.testA();
        ModuleB b = new ModuleB();
        b.testB();
        ModuleC c = new ModuleC();
        c.testC();
    }
}
```

**客户端角色类：**

```java
public class Client {

    public static void main(String[] args) {

        Facade facade = new Facade();
        facade.test();
    }

}
```

Facade类其实相当于A、B、C模块的外观界面，有了这个Facade类，那么客户端就不需要亲自调用子系统中的A、B、C模块了，也不需要知道系统内部的实现细节，甚至都不需要知道A、B、C模块的存在，客户端只需要跟Facade类交互就好了，从而更好地实现了客户端和子系统中A、B、C模块的解耦，让客户端更容易地使用系统。

### 三、外观模式的扩展

**使用外观模式还有一个附带的好处，就是能够有选择性地暴露方法**。一个模块中定义的方法可以分成两部分，一部分是给子系统外部使用的，一部分是子系统内部模块之间相互调用时使用的。有了Facade类，那么用于子系统内部模块之间相互调用的方法就不用暴露给子系统外部了。

比如，定义如下A、B、C模块。

```java
public class Module {
    /**
     * 提供给子系统外部使用的方法
     */
    public void a1(){};

    /**
     * 子系统内部模块之间相互调用时使用的方法
     */
    public void a2(){};
    public void a3(){};
}
```

```java
public class ModuleB {
    /**
     * 提供给子系统外部使用的方法
     */
    public void b1(){};

    /**
     * 子系统内部模块之间相互调用时使用的方法
     */
    public void b2(){};
    public void b3(){};
}
```

```java
public class ModuleC {
    /**
     * 提供给子系统外部使用的方法
     */
    public void c1(){};

    /**
     * 子系统内部模块之间相互调用时使用的方法
     */
    public void c2(){};
    public void c3(){};
}
```

```java
public class ModuleFacade {

    ModuleA a = new ModuleA();
    ModuleB b = new ModuleB();
    ModuleC c = new ModuleC();
    /**
     * 下面这些是A、B、C模块对子系统外部提供的方法
     */
    public void a1(){
        a.a1();
    }
    public void b1(){
        b.b1();
    }
    public void c1(){
        c.c1();
    }
}
```

这样定义一个ModuleFacade类可以有效地屏蔽内部的细节，免得客户端去调用Module类时，发现一些不需要它知道的方法。比如a2\(\)和a3\(\)方法就不需要让客户端知道，否则既暴露了内部的细节，又让客户端迷惑。

##### 一个系统可以有几个外观类

在外观模式中，通常只需要一个外观类，并且此外观类只有一个实例，换言之它是一个单例类。当然这并不意味着在整个系统里只有一个外观类，而仅仅是说对每一个子系统只有一个外观类。或者说，如果一个系统有好几个子系统的话，每一个子系统都有一个外观类，整个系统可以有数个外观类。

##### 为子系统增加新行为

初学者往往以为通过继承一个外观类便可在子系统中加入新的行为，这是错误的。外观模式的用意是为子系统提供一个集中化和简化的沟通管道，而不能向子系统加入新的行为。比如医院中的接待员并不是医护人员，接待员并不能为病人提供医疗服务。

### 四、外观模式的实例

##### 1.实例说明

某软件公司欲开发一个可应用于多个软件的文件加密模块，**该模块可以对文件中的数据进行加密并将加密之后的数据存储在一个新文件中**，具体的流程包括**三个部分，分别是读取源文件、加密、保存加密之后的文件**，其中，读取文件和保存文件使用流来实现，加密操作通过求模运算实现。这三个操作相对独立，为了实现代码的独立重用，让设计更符合单一职责原则，这三个操作的业务代码封装在三个不同的类中。

##### 2.实例类图

![](http://upload-images.jianshu.io/upload_images/3985563-1f2966b5104c668a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

EncryptFacade充当外观类，FileReader、CipherMachine和FileWriter充当子系统类。

##### 3.实例代码

**FileReader：文件读取类，充当子系统类。**

```java
class FileReader  
    {  
        public string Read(string fileNameSrc)   
        {  
       Console.Write("读取文件，获取明文：");  
            FileStream fs = null;  
            StringBuilder sb = new StringBuilder();  
       try  
            {  
                fs = new FileStream(fileNameSrc, FileMode.Open);  
                int data;  
               while((data = fs.ReadByte())!= -1)   
                {  
            sb = sb.Append((char)data);  
               }  
               fs.Close();  
               Console.WriteLine(sb.ToString());  
       }  
       catch(FileNotFoundException e)   
            {  
           Console.WriteLine("文件不存在！");  
       }  
       catch(IOException e)   
            {  
           Console.WriteLine("文件操作错误！");  
       }  
       return sb.ToString();  
        }  
    }
```

**CipherMachine：数据加密类，充当子系统类。**

```java
class CipherMachine  
    {  
       public string Encrypt(string plainText)   
       {  
       Console.Write("数据加密，将明文转换为密文：");  
       string es = "";  
            char[] chars = plainText.ToCharArray();  
       foreach(char ch in chars)   
            {  
                string c = (ch % 7).ToString();  
           es += c;  
       }  
            Console.WriteLine(es);  
       return es;  
    }  
    }
```

**FileWriter：文件保存类，充当子系统类。**

```java
class FileWriter  
    {  
        public void Write(string encryptStr,string fileNameDes)   
        {  
       Console.WriteLine("保存密文，写入文件。");  
            FileStream fs = null;  
       try  
            {  
               fs = new FileStream(fileNameDes, FileMode.Create);  
                byte[] str = Encoding.Default.GetBytes(encryptStr);  
                fs.Write(str,0,str.Length);  
                fs.Flush();  
               fs.Close();  
       }      
       catch(FileNotFoundException e)   
            {  
        Console.WriteLine("文件不存在！");  
       }  
       catch(IOException e)   
            {  
                Console.WriteLine(e.Message);  
           Console.WriteLine("文件操作错误！");  
       }          
        }  
    }
```

**EncryptFacade：加密外观类，充当外观类。**

```java
class EncryptFacade  
    {  
        //维持对其他对象的引用  
         private FileReader reader;  
        private CipherMachine cipher;  
        private FileWriter writer;  

        public EncryptFacade()  
        {  
            reader = new FileReader();  
            cipher = new CipherMachine();  
            writer = new FileWriter();  
        }  

        //调用其他对象的业务方法  
         public void FileEncrypt(string fileNameSrc, string fileNameDes)  
        {  
            string plainStr = reader.Read(fileNameSrc);  
            string encryptStr = cipher.Encrypt(plainStr);  
            writer.Write(encryptStr, fileNameDes);  
        }  
    }
```

**Program：客户端测试类**

```java
class Program  
    {  
        static void Main(string[] args)  
        {  
            EncryptFacade ef = new EncryptFacade();  
            ef.FileEncrypt("src.txt", "des.txt");  
            Console.Read();  
        }  
    }
```

##### 五、外观模式的优点

●　　松散耦合

外观模式松散了客户端与子系统的耦合关系，让子系统内部的模块能更容易扩展和维护。

●　　简单易用

外观模式让子系统更加易用，客户端不再需要了解子系统内部的实现，也不需要跟众多子系统内部的模块进行交互，只需要跟外观类交互就可以了。

●　　更好的划分访问层次

通过合理使用Facade，可以帮助我们更好地划分访问的层次。有些方法是对系统外的，有些方法是系统内部使用的。把需要暴露给外部的功能集中到外观中，这样既方便客户端使用，也很好地隐藏了内部的细节。

