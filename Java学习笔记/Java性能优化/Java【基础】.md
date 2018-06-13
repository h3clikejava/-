# Java基础

Java是高级语言，它的执行过程分为先编译成*.class文件，然后使用JVM解释成特定平台的机器码。
不同平台上的JVM是不同的，但是他们向上提供了相同的接口，JVM是Java跨平台的基础。

## Java基本类型和引用类型
| 其中整数类型 | 字符型 | 浮点型 | 布尔型 | 引用类型 |
|---|---|---|---|---|
|byte (1个字节)<br>short(2个字节)<br>int&nbsp;&nbsp;&nbsp;&nbsp;(4个字节)<br>long&nbsp;(8个字节)|char(2个字节)|float&nbsp;&nbsp;&nbsp;&nbsp;(4个字节)<br>double(8个字节)| boolean(1个字节)|类<br>接口<br>数组<br>Null|

### Char
Java使用2字节（16位）Unicode字符集作为编码格式，一共可以表示65535个字符，这其中包括了**世界上所有常见字符**。

超过这个范围的字符**无法**用Char存储，可以用String存储。String底层由char[]实现，数组最大长度是int类型，因此String最大长度是2\^32次方即2G，因为一个char占2个字节，因此内存为**4G**，考虑到其他因素，实际上无法存储这么多。

另外在做高可靠性加密的时候不要用String存储密码，因为String类型会在内存中生成常量，可以用内存快照截获。应该用char[]存储，因为其内存空间可被复用，另外可以加盐加密。

### 浮点型
Java中用二进制数据科学计算法表示浮点数，因此当小数点后数字很多的时候容易丢失精度。除了float和double，可以考虑使用BigDecimal类。

另外浮点型数据还有三个特殊的值：
* POSITIVE_INFINITY 正无穷大
* NEGATIVE_INFINITY 负无穷大
* NaN 非数（表示出错）

用一个正数除以0得到正无穷大，用一个负数除以0得到负无穷大，0.0除以0或对负数开方会得到非数。其中Double.POSITIVE_INFINITY等于Float.POSITIVE_INFINITY；NEGATIVE_INFINITY同理；但是NaN**不**与任何对象相等，即便另一个对象也是NaN。

这里特别要注意的是**浮点数可以除以0**，程序不会报任何错误，包括运行时异常；**整形不可以除0**，程序会报运行时异常。

```
// Java7开始支持用下划线分割长数字，用来方便查看
int a = 2014_05_31; // 20140531
long b = 3_000_000_000L; // 3000000000
double pi = 3.14_15_92_6; // 3.1415926
```

### 布尔型

Java中boolean类型的值只能是true或false，其他数据类型无法转换成布尔型。

#### 避免对Boolean判断

简单的来说就是直接用boolean变量判断，而不要用等于判定。如：

```
boolean flag = true;
if(flag == false) {} // 不要这样使用
if(!flag) {} // 推荐这样使用
```

这样做的好处一个是能让代码运行更快（生成字节码少5个字节），二个是让代码看起来简洁。

### 自动装箱

基本数据类型都有对应的引用类型。不过这里有个Trick的点：

```
Integer a = 8;
Integer b = 8;
Integer c = new Integer(8);
Integer d = 200;
Integer e = 200;

// a == b & a != c & d != e
System.out.print((a == b) + "=" + (c == d));
```

究其原因是在Byte/Short/Integer/Long中，系统初始化的时候就生成了-128～127的缓存。取值在该范围的对象会直接返回缓存数据，因此内存地址相同。Char对象初始化的是0～127的缓存，其余数据类型没有缓存。但是new出来的对象单独存在于堆内存中，无法共用缓存数据。

## 访问限制符 AccessSpecifier

| 限制符 | 同类 | 同包 | 子类 | 不同包 |
|-------|-----|-----|------|-------|
|public |Y    |Y    |Y     |Y      |
|protected|Y  |Y    |Y     |N      |
|friendly|Y   |Y    |N     |N      |
|private|Y    |N    |N     |N      |

## RandomAccessFile

支持文件的随机访问读写，它把文件看作一个大型**byte数组**，通过seek或skipByte方法移动到任意位置读写数据。它提供了按字节读取文件或写文件的能力，通常用作大文件读写，或用作临时快速读取文件某部分内容。

seek()绝对位置移动/skipByte()相对位置移动。

---
## 其他

### 用clone方法重复创建对象比用new更快

但是要注意系统默认实现的clone()只是浅拷贝，创建了一个地址引用而已。如果要实现深拷贝，需要自己复写clone方法，并且实现对象new创建。

### 用三目运算符替代if...else...

在一些简单的情况下，用三目运算符替代if...else...，这样会让代码看起来更简洁。这里需要注意的是，千万不要为优化而优化，JVM默认会对if...else...进行类似于三目运算的优化，因此该优化只是为了代码看起来更简洁。不要将复杂的if...else...方法体改为三目运算符，那样不仅起不到优化作用，对代码阅读和维护都会造成障碍。

同样用if...else...代替简单的switch，性能也可以提高一半。以下这个示例可能看不懂，但是这不重要，它是一种极端情况。它的意义是告诉你在写switch...case...语句的时候该思考一下能不能用if...else...实现，另外这里不要带跑偏了，是if...else...不是if...else if...

```
int i = value % 10 + 1;
switch(i) {
    case 1: return 10;
    case 2: return 11;
    case 3: return 12;
    case 4: return 13;
    case 5: return 14;
    case 6: return 15;
    case 7: return 16;
    case 8: return 17;
    case 9: return 18;
    default: return -1;
}

if(i > 9 || i< 1) return -1 else return value[i]
```

### 静态方法代替实例方法

这个情况其实需要具体分析。
**静态方法**：

* 速度很快，并且不需要创建对象实例；
* 静态方法**不能被继承**，因此无法实现多态；
* 静态方法会在类加载初期就被记录在方法区中；
* 只能访问静态变量和静态方法；

**实例方法**：

* 需要消耗更多的系统资源，需要维护一张虚拟函数导向表；
* 动态绑定，当其被调用的时候加载进方法区；

资料显示过多的，甚至全是静态方法对程序不会造成性能影响。理论上不依赖内部成员变量的所有方法都应该声明为static的。

### 建议使用局部变量

方法调用时，其传递的参数和局部变量都保存在栈中，因此访问速度很快。而静态变量，成员变量都保存在堆中，相对而言读取速度较慢。

### 布尔运算代替位运算
用 `&&`、`||`替代`&`、`|`。布尔运算在判定条件已经成立的时候不会继续执行剩下的判断，但是位运算不会，会导致计算。

### 显示的对不需要用到的对象置空
在内存章节我们就提到过，在函数体内对于不用的对象直接置Null可以节省内存。在一些极端情况下，显示置Null还能提高代码性能，因此对于不需要用的的对象，及时置Null是有必要的。

```
// 在单位时间内，将a置空比不置空可以创建更多的对象实例
private static Object a;
public static void test() {
    int n = 0;
    while(true) {
        // a = null;
        a = new SizeableObject();
        n++;
    }
}
```

### Java随机数

可以直接用Math.random()方法生成一个0～1之间的随机数。

### 多变量赋值

```
int a = 7, b = 7 , c = 7;

int d, e, f;
d = e = f =8;
```

### 初始化块

初始化块分为静态初始化块和普通初始化块。static初始化块会在构造函数之前执行，且只会执行一次。普通初始化块也会在构造函数之前执行，但是每次调用构造函数之前都会执行。特别注意，非静态代码块是紧跟着构造函数执行的，中间不会插其他代码。

```
public class A {
    static {
        print("父类静态代码块");
    }
    
    public A() {
        print("父类构造函数");
    }
    
    {
        print("父类非静态代码块");
    }
}

public class B extends A {
    public static void main(String[] args) {
        print("main方法");
        B b = new B();
    }

    static {
        print("子类静态代码块");
    }
    
    public B() {
        print("子类构造函数");
    }
    
    {
        print("子类非静态代码块");
    }
}

// 输出顺序
// 父类静态代码块
// 子类静态代码块
// main方法
// 父类非静态代码块
// 父类构造函数
// 子类非静态代码块
// 子类构造函数
```

这个例子要特别仔细，首先由于Main方法写在B类中，因此程序执行的时候一定会**先调用静态代码块**，而不是main函数；其次非静态代码块的执行是**贴着构造函数**的，因此父类构造函数执行完毕了之后才会执行子类的非静态代码块。

`如果是直接调用的父类静态成员变量，那么子类是不会被初始化的，即便存在静态代码块`

```
class A {
    static int a = 100;
    static {
        print("static A");
    }
}

class B extends A {
    static {
        print("static B");
    }
}

class C {
    public static void main(String[] args) {
        print("main函数");
        print(B.a);
    }
}

// 输出顺序
// main函数
// Static A
// 100
```

`类作为数组时候，也不会触发初始化；访问常量也不会触发初始化。`

```
public class A {
    public static final String color = "red";//新增一段常量，编译阶段户直接放入常量池
    static {
        print("A");
    }
}

public class B {
    public static void main(String[] args) {
        A[] a = new A[5];
        print(A.color);
    }
}

// 不会触发A类初始化
```

### label 语句

label提供了控制多层循环的方法，可与break或continue配合使用。一般是在某个循环体前加上标号，在break或continue后使用该标号，从而控制循环。例如下面这个例子可以用break直接跳出两层循环。

```
other: for(int i = 0; i < 10; i++) {
    for(int j = 0; j < 10; j++) {
        if(i > j) break other;
    }
}
```

### 关键字strictfp/transient

**strictfp**表示精确浮点，使类采用更精确的浮点计算方法。
**transient**表示变量不会被持久化。

### System/Runtime类

System类表示当前Java程序运行的平台。Runtime类表示当前Java程序运行时环境。

跟System.gc()一样，也存在Runtime.getRuntime().gc()方法，实际上System.gc()就是调用的Runtime.gc()方法。

通过System.getevn()方法可以获取当前系统的平台环境变量；通过System.getProperties()方法可以获取当前系统属性。例如：

```
// 通过该方法可以获得浏览器UA
// Dalvik/2.1.0 (Linux; U; Android 7.0; SM-G9350 Build/NRD90M)
System.getProperty("http.agent")
```

通过Runtime可以获得JVM相关信息，如CPU数量，内存信息。

```
// 获得CPU个数
Runtime.getRuntime().availableProcessors();
// 获得已使用内存大小
Runtime.getRuntime().totalMemory();
// 获得剩余内存大小
Runtime.getRuntime().freeMemory();
// 获得总内存大小
Runtime.getRuntime().maxMemory();
```

通过以上这个例子，我们发现freeM总是很小，原因是Java程序只有当内存不够用的时候才会向内存申请内存，totalM是已经占用的内存，freeM是申请到还没用到的内存，因此占用内存大小是totalM+freeM<maxM，当超过maxM的时候就会报OOM。在Android上一个App最大可用内存大小是**512M**。

Runtime还可以单独启动一个进程来执行控制台命令：

```
Runtime.getRuntime().exec("notepad.exe");
```

### 局部内部类

常见的内部类有内部类、静态内部类、匿名内部类。其实还存在一种更为隐蔽的局部内部类：

```
void myMethod() {
    class LocalInnerClass {
        ...
    }
}
```

以上示例中LocalInnerClass类仅供myMehtod方法访问。

### 接口的继承

接口可以继承其他接口的常量和方法，而且支持多继承。

```
interface A {
    int YES = 1;
    int NO = 2;
    a();
}

interface B {
    int SUMMER = 3;
    int WINTER = 4;
    b();
}

interface c extends A, B {
    c();
}
```

### 接口的默认方法

实际开发中会遇到一种情况，如果想要在旧的接口文件中加一个方法，那么所有该接口的实现类都需要修改。在**JAVA8**中，允许在接口中定义一个默认方法，让实现类可以不用实现该方法，类似于抽象类中的方法。

```
interface MyInterface {
    void a();
    default String getMyName() {
        return "H3c";
    }
}
```

当一个类需要实现MyInterface接口时，必须实现a()方法，但是他可以选择是否重写getMyName()方法，即使不重写也可以正常调用getMyName()方法。




