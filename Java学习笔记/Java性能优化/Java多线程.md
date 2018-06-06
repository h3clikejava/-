# Java多线程

# synchronized与volatile

synchronized保证可见性和原子性，而volatile只保证可见性不保证同步，且不会阻塞，所以volatile不是线程安全的。volatile轻量级，只能修饰变量。synchronized重量级，还可修饰方法。

```
int n = 0;
volatile m = 0;

// 创建10个线程执行n++一万次
// 输出n的值很小，因为线程间不同步

// 同样创建以上代码，执行m++一万次
// 同样输出m值不到十万，虽然比n大。
// 因为volatile只保证读的时候是内存中最新的值，但是写的时候并不会同步

// 只有对n++方法执行同步锁之后，最终结果才会是十万
// 但是同步锁会阻塞

```

