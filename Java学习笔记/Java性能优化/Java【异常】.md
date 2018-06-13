# Java API调优 【异常】

## 受检异常/非受检异常

在API开发者的概念中，异常分为受检异常（Checked Exception）和非受检异常（Unchecked Exception）。

其中RuntimeException和Error属于受检异常，而其他则属于非受检异常。两者的区别在于，受检异常要在编译器编译过程中显示的声明或捕获，避免API使用者忽略出错的情况，但是这样也需要使用更多的代码避免编译错误。

虽然受检异常有其好处，但是主流意见仍是优先使用非受检异常。因为受检异常可能对API的迭代演化造成障碍，比如后续版本修改或删除受检异常，会直接导致API使用者的代码无法编译，因此对于API开发者而言需要很谨慎的使用受检异常。

但是无论是受检异常还是非受检异常都应该在API文档中描述清楚。

## 用Throwable替代Exception和Error

有时候会有同时判定Exception和Error，然后执行相同操作的情况。初学者通常会这么写:

```
try {
} catch (Exception e) {
} catch (Error e) {}
```

实际上Exception和Error都继承自Throwable。因此应该这么写：

```
try {
} catch (Throwable e) {}
```

## Java中，在try/catch没有抛出异常的情况下，是没有任何性能损耗的。

```
// 两者性能相同
try {
    while(true) {
        int i = 0;
    }
} catch (Exception e) {}

while(true) {
    try {
        int i = 0;
    } catch (Exception e) {}
}
```

## 用try-with-resources关闭资源

在以前版本中如果想要关闭数据库之类的资源，开发者必须自己写try...catch...finally。在Jdk7中，API开发者将类继承AutoCloseable或Closeable接口之后，API使用者可以用try-with-resource特性自动关闭资源。

```
Stream io = null;
try {
    // do sth...
} catch (Exception e) {
} finally {
    if(io != null) {
        io.close();
    }
}

// 使用try-with-resource，会自动调用close()方法
try (io = new CustomStream()) {
    // do sth...
} catch (Exception e) {}

// 如果不用try-with-resource模式，不会调用close()方法
io = new CustomStream();

class CustomStream extend Stream implements AutoCloseable {
    private Stream io;
    
    @Override
    void close() {
        if(io != null) {
            io.close();
        }
    }
}
```

