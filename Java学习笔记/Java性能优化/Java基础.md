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

---
## Java随机数
可以直接用Math.random()方法生成一个0～1之间的随机数。

## 多变量赋值

```
int a = 7, b = 7 , c = 7;

int d, e, f;
d = e = f =8;
```


