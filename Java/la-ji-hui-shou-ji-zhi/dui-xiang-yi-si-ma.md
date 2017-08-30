设想一下，如果我们作为一个GC的话，我们会如何进行垃圾回收？就像如何将一头大象放入冰箱，我们的垃圾回收也可以分为三步来实现：

1. 哪些内存需要进行回收？

2. 什么时候回收？

3. 怎么进行内存回收？

# 如何判断那些内存需要回收？

## 两种熟知的方法

同样我们站在GC的角度思考，对象存在的意义就是为了被引用，那么利用计数器，没有被引用的对象是不是可以看作死亡了呢？

**引用计数法：**如果有地方引用该对象，该对象的引用计数就+1，如果引用失效的话就减一。计数器为0的对象不可以被使用。

**答案是不行的。**试想一下，如果有两个对象互相引用，比如objA.instance = objB, objB.instance = objB，这个时候两个对象都不能被访问，但是互相引用导致引用计数不为0，这不就无法判定为死亡了吗？我们如果是GC，能允许这种长生不死的存在吗？肯定不。所以引用计数法并没有被采用在目前的JVM垃圾回收器中。

**可达性分析法：**如果我们将一些GC Roots对象作为起始点，从这些节点向下搜索，搜索到的路径为引用链，如果有一些对象没有任何引用链相连，那么这个对象对于GC Roots是不可达的，即使它们之间可能相互产生关联，所以将其判定为可回收对象。如下图：

GC Roots:

* 虚拟机栈（栈帧中的本地变量表）中引用的对象

* 方法区中类静态属性引用的对象

* 方法区中常量引用的对象

* 本地方法栈中JNI（即一般说的Native方法）引用的对象

## 那么什么是引用呢？

**jdk1.2之前，定义为：**如果reference类型的数据中存储的数值代表的是另一块内存的起始地址，就成为这块内存代表着一个引用。

那么我们好像对于那种如果希望内存足够的时候保留，不够的时候回收的对象一个十分明确的解释。

**jdk1.2之后，扩展为：**

* **强引用：**只要存在，垃圾收集器就不会回收对象。

  > Object obj = new Object\(\);之类

* **软引用：**用来描述一些还有用但是不必须的对象，系统将要发生内存溢出异常之前，将会把这些对象列入回收范围之中进行第二次回收，如果还是不够那就只能抛出内存溢出的异常了。

  > SoftReference&lt;String&gt;s = new SoftReference&lt;&gt;\("我还有用但不是必须的!"\);

* **弱引用：**用来描述非必须对象，但是强度比弱引用更弱，被弱引用关联的对象只能生存到下一次垃圾收集发生之前，垃圾收集工作的时候，无论是否必要都会回收掉只被弱引用关联的对象。

  > WeakReference&lt;String&gt;s = new WeakReference&lt;String&gt;\("我只能活到下一次垃圾收集之前"\);

* **虚引用（幽灵引用或幻影引用）：**一个对象是否有虚引用，与其生命周期毫无关系，也无法通过虚引用取得一个对象实例，只被虚引用的对象，随时都会被回收掉

  > PhantomReference&lt;String&gt;ref = new PhantomReference&lt;String&gt;\("我只能接受死亡通知"\) , targetReferenceQueue&lt;String&gt;\);

## 不可达对象非死不可吗

看下面示意图：

![](http://p1.bqimg.com/567571/f1764d510804d741.png)

```
public class SaveSelf {
    public static SaveSelf instance = null;

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed!");
        instance = this;
    }
    public void isAlive(){
        System.out.println("I'm alive!");
    }

    public static void main(String[] args) {
        instance = new SaveSelf();
        instance = null;
        System.gc();
        try {
            Thread.sleep(500);//用于等待Finalize线程执行finalize方法
            if (instance != null){
                instance.isAlive();
            }else{
                System.out.println("I will dead!");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        instance = null;
        System.gc();
        try {
            Thread.sleep(500);
            if (instance != null){
                instance.isAlive();
            }else{
                System.out.println("I will dead!");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

> finalize method executed!
>
> I'm alive!
>
> I will dead!
>
> Process finished with exit code 0

任何一个兑现过的finalize方法都会只被系统自动调用一次。**当然finalize方法一般用来回收一些外部资源。**

## 回收方法区

方法区也是有垃圾收集的，那么为什么会有呢？因为如果常量池中存在一个"abc",而没有任何的String对象引用常量“abc”，那么我们需要对这个常量进行回收的。另外永久代的垃圾收集主要包括废弃常量和无用的类。

判定一个废弃常量很简单，那么如何判定一个无用的类（类对象比如Integer）呢？

* 该类的所有实例都已经被回收，也就是java堆中不存在任何该类的实例

* 加载该类的ClassLoader已经被回收

* 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过访问该类的方法。

当然这里的是可以回收，但不一定必然回收。

