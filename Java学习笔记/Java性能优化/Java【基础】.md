# Java基础

Java是高级语言，它的执行过程分为先编译成*.class文件，然后使用JVM解释成特定平台的机器码。
不同平台上的JVM是不同的，但是他们向上提供了相同的接口，JVM是Java跨平台的基础。

## Java基本类型和引用类型
| 其中整数类型 | 字符型 | 浮点型 | 布尔型 | 引用类型 |
|---|---|---|---|---|
|byte (1字节)<br>short(2字节)<br>int&nbsp;&nbsp;&nbsp;&nbsp;(4字节)<br>long&nbsp;(8字节)|char(2字节)|float&nbsp;&nbsp;&nbsp;&nbsp;(4字节)<br>double(8字节)| boolean(1字节)|类<br>接口<br>数组<br>Null|

`switch可以作用在byte/short/char/int/enum上，也可以作用在String上，但是不可以作用在long上`

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

`因此不能使用x == Double.NaN来检查是否为非数，只能用Double.isNaN(x)来判断；用Double.isInfinite可以测试是否为无穷大；用Double.isFinite检查既不是无穷也不是NaN`

这里特别要注意的是**浮点数可以除以0**，程序不会报任何错误，包括运行时异常；**整形不可以除0**，程序会报运行时异常。

```
// Java7开始支持用下划线分割长数字，用来方便查看
int a = 2014_05_31; // 20140531
long b = 3_000_000_000L; // 3000000000
double pi = 3.14_15_92_6; // 3.1415926
```
`如果long类型不够，则使用BigInteger类；如果double类型不够则使用BigDecimal。精确的浮点数计算也需要用到BigDecimal类。`

`另外注意 “I like ” + year + 1 的结果是"I like 20181"，而不是"I like 2019"。`

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

```
short s1 = 1;
s1 = s1 + 1;// 错误
s1 += 1;// 正确
```

由于1是int类型，无法将int降级为short；而s1 += 1等同于(short)(s1 + 1)因此可以正常运行。

## String

### String.intern()

intern()是一个本地方法，它返回String池中的对象，如果池中没有该对象，会把其加入到池中再返回。与equals()意义完全不同。

new String()产生的对象不会加入到String池中。

例如：

```
String str1 = "a";
String str2 = "b";
String str3 = "ab";
String str4 = "a" + "b";
String str5 = "a" + str2;
String str6 = new String("ab");

if(str3 == str6)          // false
// str6是new出来的String，因此它不会复用String池中的对象

if(str3.equale(str6))     // true
// 但是str6内容和str3一样，因此equale方法相等

if(str6.intern() == str3) // true
// str6.intern()返回的是str6在String池中相等的对象，因此就是str3。


if(str3 == str4)          // true
// JVM编译器会优化两个静态字符串拼接
if(str3 == str5)          // false
// 动态字符串拼接会在池中创建新对象
if(str3.equale(str5))     // true
if(str3.intern() == str5) // false
if(str5.intern() == str3) // true
// str5的String池内容就是str3
```

### String.trim()

String.trim()方法可以去掉收尾的空格。但是准确的说，这个方法会去掉首位所有小于等于32的char。之所以大家通常说trim方法去掉空格，是因为在char中，第32位char是空格，而之前的32个char都无法正常显示。

### String的subString()内存溢出【仅限于JDK6】

从String源代码中可以看到subString方法的实现是new了一个String对象，里面保存了原始的String的char[]和偏移量。这样做的好处在大多数情况下节省了内存空间，因为new出来的String复用了旧String。但是对于从一个巨大的String中截取少量的String来说，这种方式会导致大量的冗余。

因此当出现需要从巨大String中截取少量String的时候，建议new一个新的字符串用来保存。

```
static class HugeStr {
    private String str = new String(new char[10000000]);
    
    public String getSubString(int begin, int end) {
        return str.substring(begin, end);
        // return new String(str.substring(begin, end));
    }
}

public static void main(String[] args) {
    List<String> list = new ArrayList();
    for(int n = 0;n < 1000; n++) {
        list.add(new HugeStr().getSubString(1, 5));
    }
}
```

上面这个代码会导致OOM，但是要注意的是，这是一个极端的例子，开发过程中很少会遇到，其中一个条件缺失都不会导致OOM，主要是为了了解触发OOM的原因。

之前提到过，在JDK6中，subString存放的是索引和字符串引用，因此循环结束后new出来的HugeStr对象并不能释放，并且每个对象中都保留了原始String，必然会导致内存溢出。

但是如果采用注释掉的方式，每次创建出来的都是新字符串对象，每轮循环都会释放掉之前的大额数据，因此不会内存溢出。

另外这个问题我之前想了好久，一直觉得subString复用的内存空间，怎么可能导致内存溢出呢，后来才发现其实是被网上部分代码及文字错误诱导了。如果str是主类的成员变量，确实不会出现OOM，但是这个例子中str是子类的成员变量，因此就存在GC无法回收的情况。

**这个问题在JDK7中就已经修复了，官方用类似的方法牺牲时间来换取空间损耗。这里仍然记录下来是为了告诉大家，出现问题的时候查看源码了解实现逻辑的重要性。而不是单纯的认为系统API有问题就不处理了。**

### char速度比String快

```
if(s.startsWith("a"))
// 用charAt()速度较快
if('a' == s.charAt(0))
```

```
String a = "abc" + "d";
// 用char速度更快
String a = "abc" + 'd';
```

### StringBuffer/StringBuilder
StringBuffer是线程安全的，而StringBuilder非线程安全。在函数内部用StringBuilder效率可以提升10%～15%。

## 正则表达式

| 字符 | 说明 | 字符 | 说明 |
|:---:|-----|:----:|-----|
| $ | 匹配一行结尾 | ^ | 匹配一行开头 |
| () | 标记子表达式开始和结束的位置 | * | 前面子表达式出现0次或多次 |
| [] | 表示枚举 | + | 前面子表达式出现1次或多次 |
| {} | 用于标记前面子表达式出现的频度 | ？| 前面子表达式出现0次或1次 |
| . | 匹配除\n之外的任何单字符 | \| | 制定两项之间任意一项 |
| \d | 匹配0～9任意数字 | \D | 匹配所有非数字 |
| \s | 匹配所有空白字符（空格、制表符、回车符、换行符、换页符等）| \S | 匹配所有非空白字符 |
| \w | 匹配所有单词字符（a-z、0-9、_）| \W | 匹配所有非单词字符 |
| - | 表示范围 | ^ | 表示求否 |
| && | 与运算 | [[]] | 并运算|

d是digit，s是space， w是word的意思。

示例:

```
[abc] // a、b、c任意一个字符
[gz] // g、z任意一个字符
[a-f] // a～f之间任意一个字符
[^abc] // 非a、b、c任意一个字符
[^a~f] // 非a~f之间
[a-z&&[def]] // a-z之间且d、e、f中任意一个，即d、e、f
[a-z&&[^bc]] // a-z之间且不是b、c
[a-d[m-p]] // a～d之间并且m~p之间
(public|protected|private) // 匹配访问控制符

```

### 贪婪模式/勉强模式/占有模式

| 贪婪模式 | 勉强模式 | 占有模式 | 说明 |
|---|---|---|---|
| X? | X?? | X?+ | X表达式出现0次或1次 |
| X\* | X\*? | X\*+| X表达式出现0次或多次 |
| X+ | X+? | X++ | X表达式出现1次或多次 |
| X{n} | X{n}? | X{n}+ | X表达式出现n次 |
| X{n,} | X{n,}? | X{n,}+ | X表达式最少出现n次 |
| X{n,m} | X{n,m}? | X{n,m}+ | X表达式最少出现n次，最多出现m次 |

* 贪婪模式（Greedy）：表达式会一直匹配，直到无法匹配。
* 勉强模式（Reluctant）：只会匹配最小字符。
* 占有模式（Possessive）：只有Java支持占有模式。

```
String test = "a<tr>aava </tr>abb "

// 贪婪模式
String reg = "<.+>";
System.out.println(test.replaceAll(reg, "###"));
// 输出a###abb
// 首先<匹配的<
// .+匹配的tr>aava</tr>abb
// >发现没有可匹配的就回溯
// 因此最终匹配<tr>aava</tr>

// 勉强模式
String reg = "<.+?>";
System.out.println(test.replaceAll(reg, "###"));
// 输出a###aava ###abb
// 首先<匹配<
// .+匹配a
// >尝试匹配>失败，继续用.+匹配直到匹配上>
// 因此最终匹配<tr></tr>

// 占有模式
String test = "a<tr>aava </tr>abb ";
String reg = "<.++>";
System.out.println(test.replaceAll(reg, "###"));
// 输出a<tr>aava </tr>abb 
// 占有模式同贪婪模式，只是不做回溯
// 如果发现最后结尾与预期不同，则完全比配失败
```

## 面向对象的特性

继承/封装/多态/抽象

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

### 抽象类和接口的异同点

#### 相同：

* 不能够实例化
* 可以作为引用类型
* 继承的类或实现接口的类都必须实现所有的抽象方法

#### 不同：

##### 抽象类

* 可以有成员变量
* 可以定义构造器
* 可以有具体的方法
* 可以包含静态方法
* 一个类只能继承一个抽象类

##### 接口

* 所有的成员变量都是常量
* 不可以定义构造器
* 所有方法都是抽象方法，不可以实现方法
* 所有方法都为public的
* 不能有静态方法
* 一个类可以实现多个接口

#### 另外抽象方法不可以是static，不可以是native，不可以是synchronized

因为抽象方法需要依赖子类重写，静态方法无法被重写，native方法需要具体实现，synchronized类同。





