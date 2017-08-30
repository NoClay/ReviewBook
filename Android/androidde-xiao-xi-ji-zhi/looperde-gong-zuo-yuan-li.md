# Looper的工作原理

Looper在消息机制中扮演着消息循环的角色，具体来说就是它会不断的从MessageQueue中查看是否有新消息，如果有新消息就会立刻处理，否则就会一直阻塞。

## Looper的构造方法

```
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

可以看出Looper的构造方法被私有了，构造方法中创建了一个消息队列，同时拥有对Thread的引用。这样子很显然是为了保证一个线程至多可以拥有一个looper，即对于某个线程来讲的单例模式

## Looper的prepare\(\)

```
public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

    /**
     * Initialize the current thread as a looper, marking it as an
     * application's main looper. The main looper for your application
     * is created by the Android environment, so you should never need
     * to call this function yourself.  See also: {@link #prepare()}
     */
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
```

* prepare\(\)：用于准备一个当前线程的Looper，默认允许退出

* prepare\(boolean quitAllowed\)：用于准备一个当前线程的Looper，允许自己设置是否可以退出

* prepareMainLooper\(\)：对主线程的Looper的准备，通常我们不需要自己来调用这个方法

## Looper.loop

```
public static void loop() {
        final Looper me = myLooper();  //获取当前线程的looper
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;//获取与looper绑定的MessageQueue

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();
        //不断的获取对象，分发对象到Handler中消费
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```

Loop方法的工作过程就是一个死循环，唯一跳出的方法是MessageQueue.next方法返回了null，但是当消息队列MessageQueue中没有消息的时候，MessageQueue.next方法会阻塞，而Looper.loop方法也会随之阻塞，如果意外的返回了null，会跳出Looper.loop的执行；当获取到一个Message的时候，Looper会调用`msg.target.dispatchMessage(msg)`，而msg.target就是一个目标Handler，所以从此处将消息传递给Handler处理，但这里的Handler的dispatchMessage方法是在创建Handler时使用的Looper执行的，所以就将代码逻辑切换到了指定的线程中执行了。

## Looper的退出

```
    public void quit() {
        mQueue.quit(false);
    }

    public void quitSafely() {
        mQueue.quit(true);
    }
```

* quit：直接退出Looper

* quitSafely：设定一个退出标记，当消息队列中的已有信息处理完毕后才会安全的退出

* 在子线程中，如果手动为其创建了Looper，那么在所有的事情处理完成以后应该调用quit方法来终止消息循环，否则这个子线程会一直处于等待的状态，如果退出Looper后，这个线程就会立刻终止，因此建议不需要的时候终止Looper



