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
4. unit：用于指定keepAliveTime参数的时间单位，这是一个枚举，常用的有TimeUnit.MILLISECONDS\(毫秒\)、TimeUnit.SECONDS\(秒\)、TimeUnit.MINUTES\(分钟\)等  
5. workQueue：线程池中的任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中。  
6. threadFactory：线程工厂，为线程池提供创建新线程的功能，只有一个方法newThread\(Runnable r\)  
7. RejectedExecutionHandler handler（不常用）：当线程池无法执行新任务时，这肯呢个是由于任务队列已满或者无法成功执行任务，咋饿个时候ThreadPoolExecutor会调用handler的rejectedExecution方法来通知调用者。默认情况下该方法会直接抛出一个RejectedExecutionException，ThreadPoolExecutor提供了几个可选值，CallerRunsPolicy、AbortPolicy、DiscardPolicy和DiscardOldsPolicy，其中AbortPolicy是默认值，会直接抛出RejectedExecutionException。

# Thread执行任务规则

1. 如果线程池中的数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务
2. 如果线程池中的线程数量已经达到或者超过核心线程的数量，那么任务就会被插入到任务队列中排队等待执行
3. 如果步骤2中无法将任务插入到任务队列中（任务队列已满），这个时候如果线程数量没有达到线程池规定的最大值，立刻启动一个非核心线程来执行任务。
4. 如果步骤3中线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务，ThreadPoolExecutor会调用RejectedExecutionHandler的rejectedExecution方法来通知调用者

我们看源码解析：

```java
 public void execute(Runnable command) {
     if (command == null)
         throw new NullPointerException();
     /*
      * Proceed in 3 steps:
      *
      * 1. If fewer than corePoolSize threads are running, try to
      * start a new thread with the given command as its first
      * task.  The call to addWorker atomically checks runState and
      * workerCount, and so prevents false alarms that would add
      * threads when it shouldn't, by returning false.
      *
      * 2. If a task can be successfully queued, then we still need
      * to double-check whether we should have added a thread
      * (because existing ones died since last checking) or that
      * the pool shut down since entry into this method. So we
      * recheck state and if necessary roll back the enqueuing if
      * stopped, or start a new thread if there are none.
      *
      * 3. If we cannot queue task, then we try to add a new
      * thread.  If it fails, we know we are shut down or saturated
      * and so reject the task.
      */
     int c = ctl.get();
     if (workerCountOf(c) < corePoolSize) {
         if (addWorker(command, true))
             return;
         c = ctl.get();
     }
     if (isRunning(c) && workQueue.offer(command)) {
         int recheck = ctl.get();
         if (! isRunning(recheck) && remove(command))
             reject(command);
         else if (workerCountOf(recheck) == 0)
             addWorker(null, false);
     }
     else if (!addWorker(command, false))
         reject(command);
 
```

注释：

1. 如果正在运行的核心线程数小于核心线程的数量，**请尝试使用给定命令作为其第一个任务启动一个新线程。**对addWorker的调用即进行原子地检查runState和workerCount，从而防止当不应该通过返回false来添加线程的虚假警报。

2. 如果一个任务可以成功排队，那么我们还需要仔细检查是否应该添加一个线程（如果之前的线程死亡），或者进入该方法后该线程池将关闭。 所以我们重新检查状态，如果有必要抛出异常，否则启动新的线程。

3. 如果我们无法排队任务，那么我们尝试添加一个新线程。 如果失败，我们知道我们被关闭或饱和，所以拒绝任务。

# 线程池的分类和对比

|   |
| :--- |


|  | FixedThreadPool | CachedThreadPool | ScheduledThreadPool | SingleThreadExecutor |
| :--- | :--- | :--- | :--- | :--- |
| 线程种类 | 仅核心 | 仅非核心 | 核心线程和非核心线程 | 只有一个核心线程 |
| 是否有超时 | 没有 | 60s回收 | 非核心闲置立即回收 | 无 |
| 线程最大量 | 自己设定 | Integer.MAX\_VALUE | 没有限制 | 1 |
| 线程数量 | 固定数量 | 不固定数量 | 核心固定，非核心不固定 | 1 |
| 空闲处理 | 不回收 | 超时回收 | 非核心闲置立即回收 | 无 |
| 任务队列大小 | 没有限制 | 空集合，任务会立即执行 |  | 外界任务在同一个线程执行 |
| 适用场景 | 耗时小 | 大量耗时较少任务 | 定时任务和具有固定周期的重复任务 |  |

  




