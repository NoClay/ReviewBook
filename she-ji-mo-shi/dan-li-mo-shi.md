### 懒汉模式

```
public class Singleton {
    private static Singleton instance = null;
    private static Object lockObject = new Object();
    //私有构造方法
    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null){
            synchronized (lockObject){
                if (instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**注意：**

1. 第一个判空是为了只有在创建实例的时候进行加同步锁。

2. 第二个判空是为了防止多个线程竞争创建实例，重复创建实例。

### 饿汉模式

```
public class Singleton{
  private Singleton(){
    
  }
  private static final Singleton instance = new Singleton();
  public static Singleton getInstance(){
    return instance;
  }
}
```

**饿汉式和懒汉式区别**

这两种乍看上去非常相似，其实是有区别的，主要两点

1、线程安全：

饿汉式是线程安全的，可以直接用于多线程而不会出现问题，懒汉式就不行，它是线程不安全的，如果用于多线程可能会被实例化多次，失去单例的作用。

如果要把懒汉式用于多线程，有两种方式保证安全性，一种是在getInstance方法上加同步，另一种是在使用该单例方法前后加双锁。

2、资源加载：

饿汉式在类创建的同时就实例化一个静态对象出来，不管之后会不会使用这个单例，会占据一定的内存，相应的在调用时速度也会更快，

而懒汉式顾名思义，会延迟加载，在第一次使用该单例的时候才会实例化对象出来，第一次掉用时要初始化，如果要做的工作比较多，性能上会有些延迟，之后就和饿汉式一样了。

