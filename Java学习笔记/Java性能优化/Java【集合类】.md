# Java集合类

Java集合大致可分为Set、List、Map、Queue四种体系。而集合类主要又由Collection和Map两个接口派生。

![集合体系继承树](https://github.com/h3clikejava/H3cStudyDiary/blob/master/Java%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Photos/Collection%E9%9B%86%E5%90%88%E4%BD%93%E7%B3%BB%E7%BB%A7%E6%89%BF%E6%A0%91.png?raw=true)

Map体系继承树

![Map体系继承树](https://github.com/h3clikejava/H3cStudyDiary/blob/master/Java%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Photos/Map%E4%BD%93%E7%B3%BB%E7%BB%A7%E6%89%BF%E6%A0%91.png?raw=true)

## List

List用来存储有序的Collection。

ArrayList、Vector、LinkedList均继承自AbstractList，AbstractList继承自AbstractCollection，并实现List接口。

ArrayList超过数组预定义最大限度时，需要对数组进行扩容，扩容过程会执行大量复制操作。而LinkedList使用链表结构，无需扩容。

另外List还有个专用目的的实现叫CopyOnWriteArrayList，由写时复制新数组实现，适用于遍历较多，变化不多的情况。

List/Collections有几个有趣的方法：

* indexOf(obj)：找到列表中第一次出现的位置，没有找到返回-1。
* lastIndexOf(Obj)：找到列表中最后一次出现的位置，没有找到返回-1。
* shuffle()：随机打乱顺序。
* reverse()：顺序反向。
* rotate()：循环，用作一次移动多个数据的位置。
* swap()：元素交换。
* replaceAll()：全部替换。
* fill()：填充。
* copy()：复制。
* binarySearch()：二分查找。

### Vector

vector时Java早期版本的类，具有同步性质，是线程安全的，它会对所有操作进行加锁，所以一般情况下不要使用。

尽管如此，平时在开发过程中也尽量不要使用Vector和HashTable，用Collections.synchronizedList(list)。

### ArrayList

优势就是**读取**和**搜索**速度很快，删除数据开销很大，数组需要重排。

### LinkedList

优势就是**插入**数据很快，这里要注意的是LinkedList用的是链表结构，因此每次获取数据都要从第一个元素开始遍历，因此数据越多性能越慢。但是即便是这样其**删除**数据也比ArrayListy要快。

```
LinkedList list;
for(int n = 0; n < list.size; n++) {
    // 不要这样使用LinkedList
    // 否则每次get都是从第一个元素开始遍历
    list.get(n);
}

// 推荐这样遍历
Iterator iterator = list.iterator();
while(iterator.hasNext())
```

因此当数据增删不是很频繁的情况下，尽量优先使用ArrayList；反之当数据需要频繁增删，查询较少，很少用到get(index)的时候，应该使用LinkedList。这里需要结合自身业务需求，而不是盲目使用ArrayList。

ArrayList和Vector底层用数组实现，ArrayList是线程不安全的，LinkedList是采用的双向链表实现。

### 范围视图操作

用List的subList()方法返回的是源数据的直接引用，只能用作暂时的对象，否则会报错。

对原数据任意位置进行增删操作之后，再次调用已经创建的subList引用会报ConcurrentModificationException。

而对subList创建的引用进行增删操作后，会对原数据对应位置进行增删操作。

### RandomAccess

RandomAccess是一个接口，它本身并没有提供任何方法，只是用来标识哪些类可以支持快速随机访问。

任何基于数组的List都实现了RandomAccess接口，而基于链表结构的都没有。因为只有数组才能快速随机访问，而链表只能通过遍历的方式访问。因此可以根据是否支持RandomAccess来优化程序性能。

```
if(list instanceof RandomAccess) {
    for(int n = 0; n < list.size; n++);
} else {
    Iterator iterator = list.iterator();
    while(iterator.hasNext());
}
```

### foreach
foreach是一种高效的迭代器，其与iterator的区别就是foreach不支持迭代过程中remove对象。因此无论对象是否支持RandomAccess，用foreach迭代效率是最高的。

## MAP
java中常见的Map有以下四种：

### HashMap
速度快，没有顺序，可以存储Null值，非同步

### TreeMap
默认按照Key升序，不允许Null值，非同步

### HashTable
线程安全的HashMap，不允许Null值，尽量使用Collections.synchronizedMap(HashMap)实现同步。

### LinkedHashMap
按照插入顺序排序，允许Null值，非同步。LinkedHashMap继承自HashMap，它在其基础上又在内部增加了一个链表，用以存放元素的插入顺序。


## Set

Set是用来存放不重复元素的Collection，Java平台有三种通用的Set实现：HashSet、TreeSet和LinkedHashSet。

1. HashSet用哈希表存放元素，性能最好，但是不能保证顺序。
2. TreeSet用红黑树存放元素，按照元素值排序，比HashSet慢。
3. LinkedHashSet用链表实现哈希表，按元素插入集合的顺序排序，效率介于HashSet和TreeSet之间。

HashMap实际上是链表数组。HashSet是由HashMap实现的，即使用Key存储，而Value存储了一个PRESENT，是一个静态的Object对象。

TreeSet底层用红黑树结构来存储集合元素。其插入的对象必须实现Comparable接口。

HashSet速度比TreeSet快很多。如果不是为了排序，建议一般不要使用TreeSet。

LinkedHashSet插入、删除速度比HashSet要慢一点，因为它需要维护链表结构。但是其遍历速度会更快。

除此之外，Set有两个专用目的的实现：EnumSet和CopyOnWriteArraySet，前者是枚举类型的高性能Set实现；后者是由写时复制新数组支持的Set实现的，所有变动性的操作都是由新复制数组来完成，这种实现仅适用于经常列举而很少修改的集合。

###既然HashSet是由HashMap实现的，HashMap又支持存储Null对象，为什么HashSet底层的Value是PRESENT对象而不是空呢？

![HashSet Value](https://pic2.zhimg.com/80/v2-dfb154a2ff37b9f790cca23b8910cfec_hd.jpg)

这是一个好问题，查看HashSet源码可知，HashSet.add()方法会返回一个boolean来表示该Key之前是否有插入过。而HashMap.add()方法会返回之前该Key插入的Value，如果该Key没有插入过即返回Null。如果HashSet的Value默认值是Null，那就不知道该Key之前是否插入过了。

## Queue

Queue是保持元素重于处理元素的Collection，通常是按照先进先出的方式安排元素，在队尾插入，队头删除；但优先队列是按值来安排的，不一定是在队尾插入。

Queue接口有个子接口叫Deque，支持在队列两端都可以插入或删除元素，也称双端队列。

LinkedList就实现了Queue接口。除此之外还有基于优先堆数据结构的优先队列（自动排序队列）PriorityQueue、阻塞队列BlockingQueue和同步队列SynchronousQueue。

## 数组

### 使用arrayCopy()复制数组

System.arrayCopy()是Native方法，速度很快。

```
// 耗时0
System.arrayCopy(array, 0, arrayDest, 0, size);
// 相比耗时严重
for (int n = 0; n < array.length; n++) {
    arrayDest[n] = array[n];
}
```

### 系统API

使用Arrays.fill(type[] a, type val);方法可以快速填充数组。

## 其他

### 集合内部避免返回Null

当一个函数需要返回数组或集合的时候，要避免直接返回Null。这里并不是说不能返回Null，而是返回Null之后需要判空，否则会抛出空指针异常。这样做只是为了让代码更美观简洁。
有的人会认为返回一个空数组或空集合需要花费时间和空间，其实可以使用公用对象来减少性能影响。
自己写的接口尽量遵循这个规范，并在接口说明中标明。

```
public String[] getBooks() {
    if(book.size() == 0)
        // 不建议用这样的返回
        return null;
    ...
}

private static final String[] NULL = new String[0];
public String[] getBooks2() {
    if(book.size() == 0) 
        return NULL;
    ...
}
public List<String> getBooks3() {
    if(book.size() == 0) 
        return Collections.emptyList();
        // Collections.emptyMap()
        // Collections.emptySet();
    ...
}
```

**特别注意：**

* Collections.emptyX()方法返回的对象都是**immutable**的，即不可更改，如果对该集合进行增删操作会导致UnsupportedOperationException异常。
* List转换为数组，直接使用toArray()方法即可，后面参数可以不写。写参数时，如果数组长度比集合长，会将多余的数组元素赋空；如果数组长度比集合短，会自动创建集合长度的数组。


