# JVM

## 类的加载和初始化

当系统运行某个Java程序的时候，会启动一个JVM来执行该程序，当出现以下几种情况的时候，JVM进程将被终止。

* 程序运行到最后正常结束
* 程序运行到System.exit()或Runtime.getRuntime().exit()代码处结束程序
* 程序执行过程中遇到未捕获的异常或错误
* 程序所在平台强制结束了JVM进程

### 类的加载

当程序主动使用某个类的时候，如果该类还没有加载到内存中，系统会通过**加载**、**连接**、**初始化**三个步骤来对该类进行初始化，如果没有意外，JVM会连续完成这三个步骤，所以把这三个步骤统称为类的加载或类的初始化。

类的加载由类加载器完成，类加载器通常由JVM提供，另外开发者可以通过继承ClassLoader基类来创建自己的类加载器。所以类加载器分为两种：**启动类加载器（bootstrap）**、**用户自定义类加载器（user-defined）**，通过ClassLoader.getSystemClassLoader()方法可以获得系统类加载器。

通常类加载器内容来源由以下几种：

* 从本地文件系统加载class文件
* 从Jar包加载class文件
* 通过网络加载class文件
* 把一个Java源文件动态编译，并执行加载

Java虚拟机规范允许系统预先加载某些类，而无需等到“首次使用”时再去加载。

### 类的连接

当类被加载之后，系统为之生成一个对应的Class对象，接着会进入连接阶段，连接阶段负责把类的二进制数据合并到JRE中。类的连接又可分为三个阶段：

1. 验证：验证阶段用于检验被加载的类是否有正确的内部结构，并和其他类协调一致。
2. 准备：类准备阶段则负责为类的静态成员变量分配内存，并设置默认初始值。
3. 解析：将类的二进制数据中的符号引用替换成直接引用。

### 类的初始化

在类的初始化阶段，虚拟机负责对类进行初始化，主要是对静态成员变量的初始化。有两种方式：

1. 声明静态成员变量时指定初始值；
2. 使用静态初始化块为静态成员变量指定初始化值。

```
public class Test {
    static {
        b = 6;
    }
    
    static int b = 9;
    public static void main(String[] args) {
        // 特别注意，这里会输出9
        System.out.println(Test.b);
    }
}
```

另外，当访问一个Java类或接口中的静态成员变量的时候，只有真正声明这个成员变量的类或接口才会被初始化。

```
class B {
    static int value 100;
    static {
        System.out.println("B is init");
    }
}

class A extends B {
    static {
        System.out.println("A is init");
    }
}

public class InitTest {
    public static void main(String[] args) {
        System.out.pringln(A.value);
    }
}
```

上述代码，InitTest类通过A.value引用了类B中声明的静态成员变量value，所以只有B类会被初始化，而A类不会。

JVM初始化一个类包含以下几个步骤：

1. 假如这个类没有被加载和连接，则程序先加载并连接该类。
2. 假如该类的直接父类还没有被初始化，则先初始化其直接父类。
3. 加入类中有初始化语句，则系统依次执行这些初始化语句。

### 类的初始化时机

当Java程序首次通过下面6种方式来使用某个类或接口的时候，系统就会初始化该类或接口。

1. 创建类的实例。为某个类创建实例的方式包括：new，反射，反序列化
2. 调用某个类的静态方法
3. 访问某个类或接口的静态成员变量，或为该静态成员变量赋值
4. 使用反射方式来强制创建某个类或接口对应的Class对象。例如：Class.forName("User")
5. 初始化某个类的子类。当初始化某个类的子类时，该子类的所有父类都会被初始化。
6. 直接使用java.exe命令来运行某个猪类。

除以上之外，对于一个final型的静态成员变量，如果该变量的值在编译时就可以确定下来，那么对于这个成员变量相当于“**宏变量**”，Java编译器会在编译的时候直接把这个成员变量出现的地方替换成对应的值，即便该成员变量使用了**static**，也**不会**导致该类的初始化。

另外使用ClassLoader的loadClass()方法加载某类的时候，也只是加载该类，并不会执行初始化；只有使用Class的forName()方法之后会才会导致强制初始化该类。

## 类加载器

类加载器负责将.class文件加载到内存中，并为之生成Class对象。一旦一个类被JVM载入，同一个类就不会被再次载入了。

当JVM启动时，会形成由3个类加载器组成的初始类加载器层次结构：

1. Bootstrap ClassLoader被称为引导（也称为原始或根）类加载器，它负责加载Java的核心类。根类加载器很特殊，他不是ClassLoader的子类，而是JVM自身实现的。
2. Extension ClassLoader被称为扩展类加载器，负责加载JRE扩展目录中Jar包的类。
3. System ClassLoader被称为系统（也称为应用）类加载器，它负责在JVM启动时加载Jar包和类路径。程序可以通过ClassLoader的静态方法getSystemClassLoader()来获取系统类加载器。如果没有特别指定，则用户自定义的类加载器都以类加载器作为父加载器。

### 类加载机制

JVM类加载机制主要有如下3种：

1. 全盘负责：当一个类加载器负责加载某个Class时，该Class所以来的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入。
2. 父类委托：先让父类加载器试图加载该Class，只有在父类加载器无法加载该类的时候才会尝试从自己的类路径中加载该类。
3. 缓存机制：缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区中搜寻该Class，只有当缓存区中不存在该Class对象时，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓冲区中。这就是为什么修改了Class后，必须重启JVM，程序所做的修改才会生效的原因。

类加载器加载Class大致经过以下8个步骤：

1. 检测此Class是否载入过，如果载入过直接执行第8步，否则执行第2步；
2. 如果父类加载器不存在，则执行第4步，否则执行第3步；
3. 请求父类加载器载入目标，如果载入成功执行第8步，否则执行第5步；
4. 使用根类加载器载入目标，如果载入成功执行第8步，否则执行第7步；
5. 当前类加载器尝试寻找Class文件，如果找到则执行第6步，否则执行第7步；
6. 从文件中载入Class，成功后执行第8步；
7. 抛出ClassNotFound异常
8. 返回对应的Class对象。

其中第5、6步允许重写ClassLoader的findClass方法来实现自己的载入策略，甚至重写loadClass方法实现自己的载入过程。

## 动态编译

一般情况下，开发人员都是在程序运行前就编写完成了所有的Java源代码；而对于某些应用来说，Java源代码的内容在运行时刻才能确定。这个时候就需要动态编译源代码生成Java字节码，再由JVM来加载运行。


