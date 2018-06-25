# Java多线程

## 进程和线程
操作系统中每个程序就是一个进程，当一个程序运行时包含多个顺序执行流，每个顺序执行流就是一个线程。

`操作系统可以同时执行多个任务，这个任务就是进程；而进程可以同时执行多个任务，这个任务就是线程。`

`进程是操作系统中资源的组织单位，线程是CPU调度的基本单位。`

### 进程有三个特性：

* **独立性**：进程是系统中独立存在的实体，拥有独立的资源和地址空间。
* **动态性**：进程与程序的区别在于，程序是静态的指令集合，而进程是正在系统中活动的指令集合，具有时间的概念。
* **并发性**：多个进程可以在单个处理器上并发执行，相互没有影响。

多线程扩展了进程的概念，使得同一个进程可以同时并发处理多个任务。

* 进程间不能共享内存，但是线程之间很容易共享内存
* 系统创建进程需要重新为其分配资源，而创建线程代价很小，因此多线程处理并发效率比多进程处理效率高。

### 值的可见性

在单线程应用中如果对一个值进行写操作，接下来再进行读操作，那么所读到的值就是上次写入的值，也就说前面的操作对后面是可见的。但是在多线程当中就不一样了，如果不使用同步机制，那么就不能保证一个线程写入的值对另一个线程可见。造成这个问题原因有：

* CPU内部缓存：CPU直接操作的是缓存中的数据，并在需要的时候会把缓存数据和主存数据同步。因此在某些时刻缓存数据可能和主存数据不一致。
* CPU的执行执行排序：某些时候CPU的执行顺序是不确定的，导致部分需要后运行的线程却读到了较早的值。
* 编译器代码重排：出于性能优化的目的，编译器可能在编译的时候对生成的目标代码进行重新排列。

### 线程创建的方式有三种：

1. 继承Thread类
2. 实现Runnable接口
3. 使用Callable和Future

### Callable和Future

Callable提供了一个call()方法来执行线程，但是它比run()方法更强大，它可以有**返回值**，可以**声明抛出异常**。当执行get()方法的时候会阻塞线程直到线程执行完毕。

```
class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        Print("IN:" + Thread.currentThread().getName());
        Thread.sleep(20-000);
        return 3;
    }
}

public void test() {
    FutureTask<Integer> ft = new FutureTask(new MyCallable());
    new Thread(ft).start();
    // 注意ft.get()会阻塞线程
    Print("FT:" + ft.get() + "="  + Thread.currentThread().getName());
}
```

### 线程的生命周期

线程的生命周期有五种状态：

新建（New）- 就绪（Runnable）- 运行（Running）- 阻塞（Blocked）- 死亡（Dead）

当线程调用new的时候就进入到了新建状态；当调用start方法后进入就绪状态，JVM会为其创建方法调用栈和程序计数器，处于这个状态的的线程并没有开始运行，只是表示线程准备就绪了，至于什么时候开始运行取决于JVM的调度。

### join()

join方法会阻塞当前线程以等待其他线程完成。

```
class MyThread() extends Thread {
    public void run() {
        System.out.println("IN");
        Thread.sleep(3000);
    }
}

public void test() {
    MyThread t = new MyThread();
    t.start();
    // join会阻塞当前线程，直到MyThread执行完成。
    t.join();
}
```

### yield() 让步

yield()和sleep()作用一样，但是sleep会阻塞线程，而yield方法不会，它会让线程进入就绪状态。

* 另外sleep方法暂停线程后会给其他线程执行机会，而不理会其他线程优先级。而yield方法只会给优先级相同或优先级更高的线程执行机会。
* sleep会将线程转入阻塞状态，知道经过阻塞时间才会将线程转入就绪状态。而yield不会进入阻塞状态，会直接进入就绪状态，因此完全可能调用yield方法之后，立即获得处理器资源而被执行。
* yield无需声明抛出任何异常

### interrupt()/interrupted()/isInterrupted()

* **interrupt()**方法只会设置线程interrupt标识位为true，并不会真的停止线程。如果线程处于wait/join/sleep等阻塞方法时会捕获InterruptedException异常，并将interrupt标志位为false。
* **Thread.interrupted()**方法用于返回当前正在执行的线程interrupt标志位状态，并且清除标志位状态，即返回结果后将interrupt标志位改为false。
* **isInterrupted()**方法用于返回某线程的interrupt标志位状态。

简单的来说，用interrupt方法并不能真的打断线程的执行，仍需自行通过标志位判断是否结束线程。

### suspend()/stop()/resume()

早期Java定义了stop方法来终止一个线程，suspend来阻塞线程，resume来恢复线程，但是在JDK1.2中就废弃了，因为他们是不安全的。

举个例子，比如A给B转账500块钱，如果在A扣掉500块钱的时候线程被stop了，那么最终A账号少了500块钱，而B没有收到钱。因此stop方法不安全。

而suspend会造成死锁，比如A获得某个锁之后调用了suspend被阻塞了，而B需要获得该锁才能调用resume，就会因为一直无法获得锁导致死锁。

### 优先级

通过setPriority(int value)方法可以设定优先级，取值范围在1～10之间。系统也提供了3个静态常量：

* MAX_PRIORITY:10
* MIN_PRIORITY:1
* NORM_PRIORITY:5

`Java线程优先级需要依靠操作系统实现，但是并不是所有操作系统都支持10级的优先级，建议开发过程中只使用系统默认提供的三种优先级。`

### DaemonThread守护线程

特性：如果前台线程都死亡，守护线程会自动死亡

通过setDaemon(true)方法，可以将一个普通线程变成守护线程，这个方法需要在start()方法之前调用。

## 线程池

线程池的优点在于：

1. 降低资源消耗
2. 提高响应速度
3. 提高线程可管理性

## synchronized/volatile/ThreadLocal

同步锁仅存在多线程同时调用的情况下才会有锁，如果是单线程执行，同步锁是不起作用的。

```
// 该例子中是单线程执行，因此同步锁递归不会产生死锁
public static void main(String[] arg) {
    this.x();
}

synchronized public void x() {
    System.out.println("X");
    Thread.sleep(1000);
    x();
}
```

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

另外要注意的是synchronized锁的是堆内存中的对象，而不是栈内存中的引用，因此当引用变了对象之后，锁会失效。同样不要用String类型作为同步锁，否则会因为String池公用导致奇怪的问题。

```
public void test() {
    System.out.println("START");
    new Thread(new X()).start();
    Thread.sleep(5-000);
    // 这里如果修改了obj对象会导致锁失效，因为锁是在堆对象上的
    // obj = new Object();
    new Thread(new X()).start();
}

static class X implements Runnable {
  public void run() {
    synchronized (obj) {
      while (true) {
        Thread.sleep(1000);
        System.out.println("X:" + Thread.currentThread().getName());
      }
    }
  }
}
```

volatile的意思就是在线程准备读取该值的时候不从CPU缓存中读取，而从主存中读取；而写入的时候也直接写入主存。

与volatile作用相反的还有ThreadLocal对象，该对象会在线程内独立维护一个变量，而保证线程之间修改互不影响。

## 原子变量 Atomic*

原子变量是比synchronized效率更高的同步操作。

## wait()/notify()与notifyAll()

wait()/notify()和notifyAll()方法都只能执行在锁块内，与synchronized配合使用。

wait()方法不会持有线程，而notify()和notifyAll()方法都会继续持有线程。意思就是说肯能由于调用了notify方法，但是仍然在执行其他任务导致锁并没有释放，因此wait的线程也并不会继续执行。需要等notify的锁释放为止。当然也可以调用notify之后直接调用wait方法释放锁，但是notify线程会被阻塞，又需要wait的线程再次调用notify方法才行。另外wait方法通常和while循环配合使用以检查条件是否满足。

```
final Object obj = new Object();

new Thread(new Runnable() {
    public void run() {
        synchronized(obj) {
            // 执行完后，线程等待，并释放锁obj
            obj.wait();
            // 通知另一个线程继续执行
            // obj.notify();
        }
    }
}).start();

new Thread(new Runnable() {
    public void run() {
        synchronized(obj) {
            // 拿到obj锁
            Thread.sleep(2000);
            // 通知其他线程活动
            obj.notify();
            // 由于notify不释放锁，因此上面的线程还是不会执行
            // 除非这里调用wait释放线程
            // obj.wait();
            // 但是调用wait之后，本线程又回被暂停，需要上个线程调用notify
            while(true) {
                Thread.sleep(1000);
                println("X");
            }
            
        }
    }
}).start();
```

针对以上情况，我们需要把obj锁的范围缩小，只加在notify方法前后即可。

### notify和notifyAll区别

对于只有两个线程而言，notify和notifyAll其实是没有区别的；即使在多个线程的情况下，无论是notify还是notifyAll**都**只能唤醒**某一个线程**。唤醒哪个线程是由JVM决定的，通常是线程间相互竞争。

只是被notify唤醒的线程执行完毕之后，如果不调用notify/notifyAll方法，会导致之前其他wait的线程继续wait。

而调用notifyAll唤醒的线程执行完毕之后，系统会自动唤醒下一个wait的线程。

```
private static final Object obj = new Object();
static class R implements Runnable {
    int i;

    R(int i) {
        this.i = i;
    }

    public void run() {
        try {
            synchronized(obj) {
                System.out.println("线程->  " + i + " 等待中");
                obj.wait();
                System.out.println("线程->  " + i + " 在运行了");
                Thread.sleep(10-000);
            }
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}

public void test() {
    Thread[] rs = new Thread[10];
    for(int i = 0;i < 10;i++) {
        rs[i] = new Thread(new R(i));
    }
    for(Thread r : rs) {
        r.start();
    }
    
    Thread.sleep(3-000);

    synchronized(obj) {
        // 无论调用哪个方法，都只会释放一个线程
        // 区别在于释放的线程执行完毕之后还会不会唤醒其他线程
        obj.notify();
//      obj.notifyAll();
    }
}
```

## Lock 锁

Lock相比synchronized更加灵活。

```
Lock lock = new ReentrantLock();
lock.lock();
// 最好写在finally中，否则容易造成死锁
lock.unlock():
```

|类别|synchronized|Lock|
|---|------------|----|
|存在层次|Java关键字，在JVM层面|是一个类|
|锁的释放|1. 执行完同步代码自动释放<br>2. 线程执行发生异常|必须手动释放|
|锁状态|无法判断|可判断|
|性能|高|轻量|

### Condition
在Lock中无法使用wait/notify/notifyAll方法，系统提供了Condition类来实现相关功能，它对应有await/signal/signalAll三个方法。

```
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
```

### 公平锁/非公平锁

公平锁的意思就是线程等待的时间越长，越早获得执行的机会；而非公平锁则是忽略线程创建时间，任何线程都是在同一条件下抢占执行机会。

Lock的实现ReentrantLock默认采用的是非公平锁，可以通过传入参数实现公平锁。

```
public ReentrantLock() {
    sync = new NonFairSync();
}

public ReentrantLock(boolean fair) {
    if(fair) {
        sync = new FairSync();
    } else {
        this();
    }
}
```

## CountDownLatch 门栓锁

CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

CountDownLatch类只提供了一个构造器：

```
    public CountDownLatch(int count) {  };  //参数count为计数值
```

然后下面这3个方法是CountDownLatch类中最重要的方法：

```
    // 调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
    public void await()
    // 和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
    public boolean await(long timeout, TimeUnit unit) 
    // 将count值减1
    public void countDown() 
```

下面看一个例子大家就清楚CountDownLatch的用法了：

```
public class Test {
     public static void main(String[] args) {   
         final CountDownLatch latch = new CountDownLatch(2);
 
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
 
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                     Thread.sleep(3000);
                     System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                     latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
 
         try {
            System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
     }
}
```

执行结果：

```
线程Thread-0正在执行
线程Thread-1正在执行
等待2个子线程执行完毕...
线程Thread-0执行完毕
线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程
```

## CyclicBarrier

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：

```
public CyclicBarrier(int parties, Runnable barrierAction)
public CyclicBarrier(int parties)
```

参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

然后CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：

```
public int await()
public int await(long timeout, TimeUnit unit)
```

第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。

下面举几个例子就明白了：

假若有若干个线程都要进行写数据操作，并且只有所有线程都完成写数据操作之后，这些线程才能继续做后面的事情，此时就可以利用CyclicBarrier了：

```
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);
        for(int i=0;i<N;i++)
            new Writer(barrier).start();
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

执行结果：

```
线程Thread-0正在写入数据...
线程Thread-3正在写入数据...
线程Thread-2正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
```

从上面输出结果可以看出，每个写入线程执行完写数据操作之后，就在等待其他线程写入操作完毕。

当所有线程线程写入操作完毕之后，所有线程就继续进行后续的操作了。

如果说想在所有线程写入操作完之后，进行额外的其他操作可以为CyclicBarrier提供Runnable参数：

```
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N,new Runnable() {
            @Override
            public void run() {
                System.out.println("当前线程"+Thread.currentThread().getName());   
            }
        });
 
        for(int i=0;i<N;i++)
            new Writer(barrier).start();
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

运行结果：

```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2正在写入数据...
线程Thread-3正在写入数据...
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
当前线程Thread-3
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
```

从结果可以看出，当四个线程都到达barrier状态后，**会从四个线程中选择一个线程**去执行Runnable。

下面看一下为await指定时间的效果：

```
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);
 
        for(int i=0;i<N;i++) {
            if(i<N-1)
                new Writer(barrier).start();
            else {
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                new Writer(barrier).start();
            }
        }
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                try {
                    cyclicBarrier.await(2000, TimeUnit.MILLISECONDS);
                } catch (TimeoutException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

执行结果：

```
线程Thread-0正在写入数据...
线程Thread-2正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-3正在写入数据...
java.util.concurrent.TimeoutException
Thread-1所有线程写入完毕，继续处理其他任务...
Thread-0所有线程写入完毕，继续处理其他任务...
    at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
    at java.util.concurrent.CyclicBarrier.await(Unknown Source)
    at com.cxh.test1.Test$Writer.run(Test.java:58)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
    at java.util.concurrent.CyclicBarrier.await(Unknown Source)
    at com.cxh.test1.Test$Writer.run(Test.java:58)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
    at java.util.concurrent.CyclicBarrier.await(Unknown Source)
    at com.cxh.test1.Test$Writer.run(Test.java:58)
Thread-2所有线程写入完毕，继续处理其他任务...
java.util.concurrent.BrokenBarrierException
线程Thread-3写入数据完毕，等待其他线程写入完毕
    at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
    at java.util.concurrent.CyclicBarrier.await(Unknown Source)
    at com.cxh.test1.Test$Writer.run(Test.java:58)
Thread-3所有线程写入完毕，继续处理其他任务...
```

上面的代码在main方法的for循环中，故意让最后一个线程启动延迟，因为在前面三个线程都达到barrier之后，等待了指定的时间发现第四个线程还没有达到barrier，就抛出异常并继续执行后面的任务。

另外CyclicBarrier是可以重用的，看下面这个例子：

```
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);
 
        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }
 
        try {
            Thread.sleep(25000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        System.out.println("CyclicBarrier重用");
 
        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
 
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```

执行结果：

```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-3正在写入数据...
线程Thread-2正在写入数据...
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-0写入数据完毕，等待其他线程写入完毕
Thread-0所有线程写入完毕，继续处理其他任务...
Thread-3所有线程写入完毕，继续处理其他任务...
Thread-1所有线程写入完毕，继续处理其他任务...
Thread-2所有线程写入完毕，继续处理其他任务...
CyclicBarrier重用
线程Thread-4正在写入数据...
线程Thread-5正在写入数据...
线程Thread-6正在写入数据...
线程Thread-7正在写入数据...
线程Thread-7写入数据完毕，等待其他线程写入完毕
线程Thread-5写入数据完毕，等待其他线程写入完毕
线程Thread-6写入数据完毕，等待其他线程写入完毕
线程Thread-4写入数据完毕，等待其他线程写入完毕
Thread-4所有线程写入完毕，继续处理其他任务...
Thread-5所有线程写入完毕，继续处理其他任务...
Thread-6所有线程写入完毕，继续处理其他任务...
Thread-7所有线程写入完毕，继续处理其他任务...
```

从执行结果可以看出，在初次的4个线程越过barrier状态后，又可以用来进行新一轮的使用。而CountDownLatch无法进行重复使用。

## Semaphore信号量

Semaphore翻译成字面意思为信号量，Semaphore可以控同时访问的线程个数，通过acquire()获取一个许可，如果没有就等待，而release()释放一个许可。它一般用于控制对某组资源的访问权限。

Semaphore类位于java.util.concurrent包下，它提供了2个构造器：

```
//参数permits表示许可数目，即同时可以允许多少线程进行访问
public Semaphore(int permits) {          
    sync = new NonfairSync(permits);
}
//这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
public Semaphore(int permits, boolean fair) {    
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```

下面说一下Semaphore类中比较重要的几个方法，首先是acquire()、release()方法：

```
// 获取一个许可
public void acquire()
// 获取permits个许可
public void acquire(int permits) 
// 释放一个许可
public void release()      
// 释放permits个许可 
public void release(int permits)
```

acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

release()用来释放许可。注意，在释放许可之前，必须先获获得许可。

这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

```
// 尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire()  
// 尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit)
// 尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits)
// 尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
```

另外还可以通过availablePermits()方法得到可用的许可数目。

下面通过一个例子来看一下Semaphore的具体使用：

假若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现：

```
public class Test {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }
 
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
 
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();           
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

执行结果：

```
工人0占用一个机器在生产...
工人1占用一个机器在生产...
工人2占用一个机器在生产...
工人4占用一个机器在生产...
工人5占用一个机器在生产...
工人0释放出机器
工人2释放出机器
工人3占用一个机器在生产...
工人7占用一个机器在生产...
工人4释放出机器
工人5释放出机器
工人1释放出机器
工人6占用一个机器在生产...
工人3释放出机器
工人7释放出机器
工人6释放出机器
```

## BlockingQueue 阻塞队列

往阻塞队列中插入元素的时候，如果该队列已满，则该线程被阻塞；当取出原色的时候，队列为空，线程也会被阻塞。

* 插入元素：add()/offer()/put()，当队列已满的时候执行操作分别会抛出异常、返回false、阻塞队列。
* 取出并删除元素：remove()/poll()/take()，当队列为空的时候执行操作分别会抛出异常，返回false，阻塞队列。
* 取出不删除元素：element()/peek()，当队列为空的时候分别会抛出异常，返回false。

## 悲观锁和乐观锁

悲观锁（Pessimistic Locking）：它对外界所有的操作都持保守态度，总是假设最坏的情况，任何操作都会让数据处于锁定状态。synchronized就是典型的悲观锁。

乐观锁（Optimistic Locking）：它对外界所有操作都持乐观态度，在读操作的时候不会对对象加锁，只有在写的情况下才会查询一下修改数据期间有没有其他对象对数据进行了修改。Java原子操作就是典型的乐观锁。乐观锁是通过“CAS（Compare and Swap）对比和交换”算法实现。

用文字的方式简单的描述一下几种锁的差别：

应用场景为某银行账号下有10元钱，有A、B、C三个用户执行取钱行为。

* 不加锁：当A、B、C三个用户同时请求到10元钱，并执行取钱行为，各取10块钱，最后很大概率会导致账号余额错误。
* 悲观锁：每个用户取钱的时候必须等待上一个用户取钱结束。
* 乐观锁：还是同样取钱，但是会有一个值来表示修改次数，假设这个value默认是0，当A取完钱之后value等于1并写入；当B取完钱之后写入的时候会读这个值，如果value是0，就写入并将value改为1，但是当发现value不是0的时候就会重新读账户余额，将自己的value临时改为1，并重新计算可否取钱，并将value改为2之后再去尝试能否写入。

在大多数情况下悲观锁比乐观锁更消耗系统性能，而且等待锁的线程会被阻塞。乐观锁会出现脏读的情况，但是不会阻塞线程。

## 脏数据

脏数据也称脏读，当一个事务正在访问数据库，并做了修改，而这个数据还没有同步到数据库中。另外个事务也访问并使用了这个数据，那么这个数据就是同步前的数据，即脏数据（Dirty Data）。该数据可能回导致操作结果出现错误。

