# HandlerThread

HandlerThread就是一个加上了Handler的Thread，它的实现相当简单。首先看构造方法：

```java
    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }

    /**
     * Constructs a HandlerThread.
     * @param name
     * @param priority The priority to run the thread at. The value supplied must be from 
     * {@link android.os.Process} and not from java.lang.Thread.
     */
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }
```

可以看出，name用来给Thread一个名字，而priority则是优先级的设定。

我们看run方法的源码：

```java
    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
```

可以看出，这里的线程里边实现了Looper。

### HandlerThread的特点 {#handlerthread的特点}

* HandlerThread将loop转到子线程中处理，说白了就是将分担MainLooper的工作量，降低了主线程的压力，使主界面更流畅。

* 开启一个线程起到多个线程的作用。处理任务是串行执行，按消息发送顺序进行处理。HandlerThread本质是一个线程，在线程内部，代码是串行处理的。

* 但是由于每一个任务都将以队列的方式逐个被执行到，一旦队列中有某个任务执行时间过长，那么就会导致后续的任务都会被延迟处理。

* HandlerThread拥有自己的消息队列，它不会干扰或阻塞UI线程。

* 对于网络IO操作，HandlerThread并不适合，因为它只有一个线程，还得排队一个一个等着。



