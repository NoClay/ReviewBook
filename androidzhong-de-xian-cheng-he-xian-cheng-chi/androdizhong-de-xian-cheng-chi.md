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
1. corePoolSize：线程池的核心线程数，默认情况下，核心线程会在线程池中一直存活，即使它们处于闲置状态，如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么闲置的核心线程在等待新任务的到来的时候会有超时策略，这个时间间隔由keepAliveTime所制定，超出后，核心线程会被终止
2. maximumPoolSize：线程池所能容纳的最大线程数，当活动线程数达到这个数值后，后续的新任务会被阻塞
3. keepAliveTime：非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。
4. unit：用于指定keepAliveTime参数的时间单位，这是一个枚举，常用的有TimeUnit.MILLISECONDS(毫秒)、TimeUnit.SECONDS(秒)、TimeUnit.MINUTES(分钟)等
5. workQueue：线程池中的任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中。
6. threadFactory：线程工厂，为线程池提供创建新线程的功能，只有一个方法newThread(Runnable r)
7. RejectedExecutionHandler handler（不常用）：当线程池无法执行新任务时，这肯呢个是由于任务队列已满或者无法成功执行任务，咋饿个时候ThreadPoolExecutor会调用handler的rejectedExecution方法来通知调用者。默认情况下该方法会直接抛出一个RejectedExecutionException，ThreadPoolExecutor提供了几个可选值，CallerRunsPolicy、AbortPolicy、DiscardPolicy和DiscardOldsPolicy，其中AbortPolicy是默认值，会直接抛出RejectedExecutionException。
