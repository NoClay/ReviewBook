# 线程池
线程池的好处：
1. 重用线程池中的线程，避免因为线程的创建和销毁带来的性能开销
2. 能有效的控制线城池的最大的并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象
3. 能够对线程进行简单的管理，并提供实时执行以及指定间隔循环执行等功能。
# ThreadPoolExecutor
这是线程池的真正实现。我们先看构造方法：
```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, 
        keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```
参数解释：
1. corePoolSize:线程池