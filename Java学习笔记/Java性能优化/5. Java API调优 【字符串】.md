# Java API调优 【字符串】

## String.intern()

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

## String.trim()

String.trim()方法可以去掉收尾的空格。但是准确的说，这个方法会去掉首位所有小于等于32的char。之所以大家通常说trim方法去掉空格，是因为在char中，第32位char是空格，而之前的32个char都无法正常显示。

## String的subString()内存溢出【仅限于JDK6】

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

## char速度比String快

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

## StringBuffer/StringBuilder
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

