## 线程有几种实现方法，都是什么？同步有几种实现方法，都是什么？

继承自Thread或者实现Runnable接口，实现Callable接口，重写call函数。

Callable是类似于Runnable的接口，实现Callable接口的类和实现Runnable的类都是可被其它线程执行的任务。 Callable和Runnable有几点不同:

1. Callable规定的方法是call\(\)，而Runnable规定的方法是run\(\).

2. Callable的任务执行后可返回值，而Runnable的任务是不能返回值的

3. call\(\)方法可抛出异常，而run\(\)方法是不能抛出异常的。

4. 运行Callable任务可拿到一个Future对象，Future表示异步计算的结果。它提供了检查计算是否完成的方法,以等待计算的完成,并检索计算的结果.通过Future对象可了解任务执行情况,可取消任务的执行,还可获取任务执行的结果

```
class TaskWithResult implements Callable<T> {  
    @Override  
    public T call() throws Exception {  
        return new T();  
    }  
      public static void main(String[] args) throws          InterruptedException,ExecutionException {  
        ExecutorService exec = Executors.newCachedThreadPool();  
        Future<T> result = exec.submit(new TaskWithResult());   
          //Future 相当于是用来存放Executor执行的结果的一种容器,可以利用result.isDone()判断任务完成否
        TaskWithResult data = result.get();
        exec.shutdown();  
    }  
}
```

# 多线程**同步方法：**

1. wait\(\)：使一个线程处于等待状态，并释放所持有的对象的lock

2. sleep\(\)：使一个正在运行的线程处于睡眠状态，是一个静态方法，调用此方法要捕获InterruptedException异常

3. notify\(\)：唤醒一个处于等待状态的线程，这里的唤醒由jvm决定调度，且与优先级无关。

4. Allnotify\(\)：唤醒所有处于等待状态的线程。

5. synchronized：利用对象的监视器属性加同步锁。

# **Synchronized**

Synchronized是Java中解决并发问题的一种最常用的方法，也是最简单的一种方法。Synchronized的作用主要有三个：（1）确保线程互斥的访问同步代码（2）保证共享变量的修改能够及时可见（3）有效解决重排序问题。从语法上讲，Synchronized总共有三种用法：

（1）修饰普通方法

（2）修饰静态方法

（3）修饰代码块

原理：执行monitorexit的线程必须是objectref所对应的monitor的所有者。

指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个monitor的所有权。

通过这两段描述，我们应该能很清楚的看出Synchronized的实现原理，Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

# Synchronized和Lock的区别？

同步的实现当然是采用锁了，java中使用锁的两个基本工具是 synchronized 和 Lock。

synchronized 用在方法和代码块上有什么区别呢？

synchronized 用在方法签名上（以test为例），当某个线程调用此方法时，会获取该实例的对象锁，方法未结束之前，其他线程只能去等待。当这个方法执行完时，才会释放对象锁。其他线程才有机会去抢占这把锁，去执行方法test,但是发生这一切的基础应当是所有线程使用的同一个对象实例，才能实现互斥的现象。否则synchronized关键字将失去意义。

（**但是如果该方法为类方法，即其修饰符为static，那么synchronized 意味着某个调用此方法的线程当前会拥有该类的锁，只要该线程持续在当前方法内运行，其他线程依然无法获得方法的使用权！**）

synchronized 用在代码块的使用方式：synchronized\(obj\){//todo code here}

当线程运行到该代码块内，就会拥有obj对象的对象锁，如果多个线程共享同一个Object对象，那么此时就会形成互斥！特别的，当obj == this时，表示当前调用该方法的实例对象。即使用synchronized代码块，可以只对需要同步的代码进行同步，这样可以大大的提高效率。

小结：

使用synchronized 代码块相比方法有两点优势：

1、可以只对需要同步的使用

2、与wait\(\)/notify\(\)/nitifyAll\(\)一起使用时，比较方便

wait\(\) 与notify\(\)/notifyAll\(\)

**这三个方法都是Object的方法，并不是线程的方法！**

wait\(\):释放占有的对象锁，线程进入等待池，释放cpu,而其他正在等待的线程即可抢占此锁，获得锁的线程即可运行程序。而sleep\(\)不同的是，线程调用此方法后，会休眠一段时间，休眠期间，会暂时释放cpu，但并不释放对象锁。也就是说，在休眠期间，其他线程依然无法进入此代码内部。休眠结束，线程重新获得cpu,执行代码。**wait\(\)和sleep\(\)最大的不同在于wait\(\)会释放对象锁，而sleep\(\)不会！**

notify\(\): 该方法会唤醒因为调用对象的wait\(\)而等待的线程，其实就是**对对象锁的唤醒，从而使得wait\(\)的线程可以有机会获取对象锁**。调用notify\(\)后，并不会立即释放锁，而是继续执行当前代码，直到synchronized中的代码全部执行完毕，才会释放对象锁。JVM则会在等待的线程中调度一个线程去获得对象锁，执行代码。需要注意的是，**wait\(\)和notify\(\)必须在synchronized代码块中调用**。

notifyAll\(\)则是唤醒所有等待的线程。

为了说明这一点，举例如下：

```java
public  class Consumer implements Runnable {
    private Integer count;

    public Consumer(Integer count) {
        this.count = count;
    }

    @Override
    public void run() {
        while (count > 0) {
            synchronized (ThreadDemo.obj) {
                if (count > 0) {
                    System.out.println("当前剩余：" + count + "，消费了一个");
                    count --;
                }
                try {
                    Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                ThreadDemo.obj.notify();
                try {
                    ThreadDemo.obj.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
 public class Producer implements Runnable {
    private Integer count;

    public Producer(Integer count) {
        this.count = count;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (ThreadDemo.obj) {
                System.out.println("当前剩余：" + count + "，生产了一个");
                count ++;
                ThreadDemo.obj.notify();
                try {
                    Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                try {
                    ThreadDemo.obj.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
public class ThreadDemo {
    public static final Object obj = new Object();

    public static void main(String[] args) {
        ThreadDemo data = new ThreadDemo();
        Integer count = 15;
        new Thread(new Consumer(count)).start();
        new Thread(new Producer(count)).start();
    }
}
```

这里使用static obj作为锁的对象，当线程Produce启动时（假如Produce首先获得锁，则Consumer会等待），打印“A”后，会先主动释放锁，然后阻塞自己。Consumer获得对象锁，打印“B”，然后释放锁，阻塞自己，那么Produce又会获得锁，然后...一直循环下去，直到count = 0.这样，使用Synchronized和wait\(\)以及notify\(\)就可以达到线程同步的目的。

**除了wait\(\)和notify\(\)协作完成线程同步之外，使用Lock也可以完成同样的目的。**

ReentrantLock 与synchronized有相同的并发性和内存语义，还包含了中断锁等候和定时锁等候，意味着线程A如果先获得了对象obj的锁，那么线程B可以在等待指定时间内依然无法获取锁，那么就会自动放弃该锁。

但是由于synchronized是在JVM层面实现的，因此系统可以监控锁的释放与否，而ReentrantLock使用代码实现的，系统无法自动释放锁，需要在代码中finally子句中显式释放锁lock.unlock\(\);

使用建议：

在并发量比较小的情况下，使用synchronized是个不错的选择，但是在并发量比较高的情况下，其性能下降很严重，此时ReentrantLock是个不错的方案。

