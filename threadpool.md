# 线程池

## 基本概念

线程：进程中负责程序执行的执行单元。一个进程中至少有一个线程。

多线程：解决多任务同时执行的需求，合理的使用CPU资源。

线程池：基本思想还是一种对象池的思想，开辟一块内存空间，里面存放了众多的（未死亡）的线程，池中线程执行调度由池管理器来处理。当有任务时，从池中取出一个，执行完后线程对象归池，这样可以避免反复创建线程对象带来的性能开销，节省了系统的资源。

## Runnable和Callable的不同点

Runnable从JDK1.0引入，Callable从JDK1.5引入，它们的主要区别是：Callable可以有返回值，可以抛出异常，Callable可以返回装载有计算结果的Future对象，通过Future对象可以了解任务的执行情况，可以取消任务的执行，也可以获取任务的执行结果，Runnable可以和Thread搭配使用，而Callable可以和ExecutorService搭配使用

## Future

```java
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

Future 定义了5个方法：

1）boolean cancel(boolean mayInterruptIfRunning)：试图取消对此任务的执行。如果任务已完成、或已取消，或者由于某些其他原因而无法取消，则此尝试将失败。当调用 cancel() 时，如果调用成功，而此任务尚未启动，则此任务将永不运行。如果任务已经启动，则 mayInterruptIfRunning 参数确定是否应该以试图停止任务的方式来中断执行此任务的线程。此方法返回后，对 isDone() 的后续调用将始终返回 true。如果此方法返回 true，则对 isCancelled() 的后续调用将始终返回 true。
2）boolean isCancelled()：如果在任务正常完成前将其取消，则返回 true。
3）boolean isDone()：如果任务已完成，则返回 true。 可能由于正常终止、异常或取消而完成，在所有这些情况中，此方法都将返回 true。
4）V get()throws InterruptedException,ExecutionException：如有必要，等待计算完成，然后获取其结果。
5）V get(long timeout,TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException： 如有必要，最多等待为使计算完成所给定的时间之后，获取其结果（如果结果可用）。

FutureTask 是一种可以取消的异步的计算任务，它的计算是通过 Callable 实现的，它等价于可以携带结果的 Runnable，并且有三个状态：等待、运行和完成。完成包括所有计算以任意的方式结束，包括正常结束、取消和异常。

## 使用多线程的优缺点

优点：
1）适当的提高程序的执行效率（多个线程同时执行）。
2）适当的提高了资源利用率（CPU、内存等）。
缺点：
1）占用一定的内存空间。
2）线程越多CPU的调度开销越大。
3）程序的复杂度会上升。

## wait和sleep的区别

sleep()：在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），该线程不丢失任何监视器的所属权，sleep() 是 Thread 类专属的静态方法，针对一个特定的线程。
wait() 方法使实体所处线程暂停执行，从而使对象进入等待状态，直到被 notify() 方法通知或者 wait() 的等待的时间到。sleep() 方法使持有的线程暂停运行，从而使线程进入休眠状态，直到用 interrupt 方法来打断他的休眠或者 sleep 的休眠的时间到。
**wait() 方法进入等待状态时会释放同步锁，而 sleep() 方法不会释放同步锁。**

## volatile 关键字

volatile 是一个特殊的修饰符，只有成员变量才能使用它。在Java并发程序缺少同步类的情况下，多线程对成员变量的操作对其它线程是透明的。volatile 变量可以保证下一个读取操作会在前一个写操作之后发生。**线程都会直接从内存中读取该变量并且不缓存它。这就确保了线程读取到的变量是同内存中是一致的。**

## 线程创建规则

ThreadPoolExecutor对象初始化时，不创建任何执行线程，当有新任务进来时，才会创建执行线程。构造ThreadPoolExecutor对象时，需要配置该对象的核心线程池大小和最大线程池大小

　　1. 当目前执行线程的总数小于核心线程大小时，所有新加入的任务，都在新线程中处理。
　　2. 当目前执行线程的总数大于或等于核心线程时，所有新加入的任务，都放入任务缓存队列中。
　　3. 当目前执行线程的总数大于或等于核心线程，并且缓存队列已满，同时此时线程总数小于线程池的最大大小，那么创建新线程，加入线程池中，协助处理新的任务。

　　4. 当所有线程都在执行，线程池大小已经达到上限，并且缓存队列已满时，就rejectHandler拒绝新的任务。

## 默认的RejectExecutionHandler拒绝执行策略

1. AbortPolicy 直接丢弃新任务，并抛出RejectedExecutionException通知调用者，任务被丢弃
2. CallerRunsPolicy 用调用者的线程，执行新的任务，如果任务执行是有严格次序的，请不要使用此policy
3. DiscardPolicy 静默丢弃任务，不通知调用者，在处理网络报文时，可以使用此任务，静默丢弃没有几乎处理的报文
4. DiscardOldestPolicy 丢弃最旧的任务，处理网络报文时，可以使用此任务，因为报文处理是有时效的，超过时效的，都必须丢弃

## 任务队列BlockingQueue

阻塞队列（BlockingQueue）是java.util.concurrent下的主要用来控制线程同步的工具。如果BlockQueue是空的,从BlockingQueue取东西的操作将会被阻断进入等待状态,直到BlockingQueue进了东西才会被唤醒。同样,如果BlockingQueue是满的,任何试图往里存东西的操作也会被阻断进入等待状态,直到BlockingQueue里有空间才会被唤醒继续操作。
阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。具体的实现类有LinkedBlockingQueue,ArrayBlockingQueued等。一般其内部的都是通过Lock和Condition([显示锁（Lock）及Condition的学习与使用](http://www.silencedut.com/2016/06/12/%E6%98%BE%E7%A4%BA%E9%94%81%EF%BC%88Lock%EF%BC%89%E5%8F%8ACondition%E7%9A%84%E5%AD%A6%E4%B9%A0%E4%B8%8E%E4%BD%BF%E7%94%A8/))来实现阻塞和唤醒。

排队原则

　　1. 如果运行的线程少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。

　　2. 如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。

　　3. 如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 maximumPoolSize，在这种情况下，任务将被拒绝。

   常见几种BlockingQueue实现

​     1. ArrayBlockingQueue :  有界的数组队列
　 2. LinkedBlockingQueue : 可支持有界/无界的队列，使用链表实现
　 3. PriorityBlockingQueue : 优先队列，可以针对任务排序
　 4. SynchronousQueue : 队列长度为1的队列，和Array有点区别就是：client thread提交到block queue会是一个阻塞过程，直到有一个worker thread连接上来poll task。

## 线程调度策略

1.抢占式调度策略

Java运行时系统的线程调度算法是抢占式的。Java运行时系统支持一种简单的固定优先级的调度算法。如果一个优先级比其他任何处于可运行状态的线程都高的线程进入就绪状态，那么运行时系统就会选择该线程运行。新的优先级较高的线程抢占了其他线程。但是Java运行时系统并不抢占同优先级的线程。换句话说，Java运行时系统不是分时的。当系统中的处于就绪状态的线程都具有相同优先级时，线程调度程序采用一种简单的、非抢占式的轮转的调度顺序。

2.时间片轮转调度策略

有些系统的线程调度采用时间片轮转调度策略。这种调度策略是从所有处于就绪状态的线程中选择优先级最高的线程分配一定的CPU时间运行。该时间过后再选择其他线程运行。只有当线程运行结束、放弃(yield)CPU或由于某种原因进入阻塞状态，低优先级的线程才有机会执行。如果有两个优先级相同的线程都在等待CPU，则调度程序以轮转的方式选择运行的线程。

## 线程池的工作过程

1. 线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。
2. 当调用 execute() 方法添加一个任务时，线程池会做如下判断：
   - 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务；
   - 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列；
   - 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；
   - 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会抛出异常RejectExecutionException。
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
4. 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

## 线程池的优点

1）避免线程的创建和销毁带来的性能开销。
2）避免大量的线程间因互相抢占系统资源导致的阻塞现象。
3｝能够对线程进行简单的管理并提供定时执行、间隔执行等功能。

## 线程池的分类

- 1）newCachedThreadPool 是一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。对于执行很多短期异步任务的程序而言，这些线程池通常可提高程序性能。调用 execute() 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。因此，长时间保持空闲的线程池不会使用任何资源。注意，可以使用 ThreadPoolExecutor 构造方法创建具有类似属性但细节不同（例如超时参数）的线程池。
- 2）newSingleThreadExecutor 创建是一个单线程池，也就是该线程池只有一个线程在工作，所有的任务是串行执行的，如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它，此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
- 3）newFixedThreadPool 创建固定大小的线程池，每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小，线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
- 4）newScheduledThreadPool 创建一个大小无限的线程池，此线程池支持定时以及周期性执行任务的需求。

## 线程池的AtomicInteger

在ThreadPoolExecutor有个ctl的AtomicInteger变量。通过这一个变量保存了两个内容：

- 所有线程的数量
- 每个线程所处的状态

其中低29位存线程数，高3位存runState，通过位运算来得到不同的值。

## 线程池相关参数介绍

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, 
        threadFactory, defaultHandler);
```

- 1）corePoolSize：线程池的核心线程数，一般情况下不管有没有任务都会一直在线程池中一直存活，只有在 ThreadPoolExecutor 中的方法 allowCoreThreadTimeOut(boolean value) 设置为 true 时，闲置的核心线程会存在超时机制，如果在指定时间没有新任务来时，核心线程也会被终止，而这个时间间隔由第3个属性 keepAliveTime 指定。
- 2）maximumPoolSize：线程池所能容纳的最大线程数，当活动的线程数达到这个值后，后续的新任务将会被阻塞。
- 3）keepAliveTime：控制线程闲置时的超时时长，超过则终止该线程。一般情况下用于非核心线程，只有在 ThreadPoolExecutor 中的方法 allowCoreThreadTimeOut(boolean value) 设置为 true时，也作用于核心线程。
- 4）unit：用于指定 keepAliveTime 参数的时间单位，TimeUnit 是个 enum 枚举类型，常用的有：TimeUnit.HOURS(小时)、TimeUnit.MINUTES(分钟)、TimeUnit.SECONDS(秒) 和 TimeUnit.MILLISECONDS(毫秒)等。
- 5）workQueue：线程池的任务队列，通过线程池的 execute(Runnable command) 方法会将任务 Runnable 存储在队列中。
- 6）threadFactory：线程工厂，它是一个接口，用来为线程池创建新线程的。

## 线程池的关闭

ThreadPoolExecutor 提供了两个方法，用于线程池的关闭，分别是 shutdown() 和 shutdownNow()。

shutdown()：不会立即的终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

## 面试题

1）什么是 Executor 框架？

Executor框架在Java 5中被引入，Executor 框架是一个根据一组执行策略调用、调度、执行和控制的异步任务的框架。

无限制的创建线程会引起应用程序内存溢出，所以创建一个线程池是个更好的的解决方案，因为可以限制线程的数量并且可以回收再利用这些线程。利用 Executor 框架可以非常方便的创建一个线程池。

2）Executors 类是什么？

Executors为Executor、ExecutorService、ScheduledExecutorService、ThreadFactory 和 Callable 类提供了一些工具方法。Executors 可以用于方便的创建线程池。