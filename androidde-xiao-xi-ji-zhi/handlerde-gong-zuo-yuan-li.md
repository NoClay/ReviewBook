# Handler的工作原理

Handler的工作主要包含消息的发送和接受过程，消息的发送可以通过post的一系列方法和send的一系列方法来实现，但是post的一系列方法最终是通过send的一些列方法来实现的。

## Handler的构造

```
    /**
     * 默认的构造器，我们平常在主线程中创建Handler的时候一般采用这个方法，因为主线程已经创    
     * 建好了对应的Looper，在子线程中使用该方法，由于默认没有Looper，则抛出异常
     */
    public Handler() {
        this(null, false);
    }

    /**
     * 利用Callback方法构造，但是同样需要Looper，这里的Callback指的是只包含handlerMessage接口方        
     * 法的Handler内的接口。同样，如果当前线程没有Looper，依然会失败。
     */
    public Handler(Callback callback) {
        this(callback, false);
    }

    /**
     * 利用参数Looper代替默认的Looper，参数不能为null
     */
    public Handler(Looper looper) {
        this(looper, null, false);
    }

    /**
     * 使用参数Looper代替默认的Looper，参数不能为null，同时提供Callback处理消息
     */
    public Handler(Looper looper, Callback callback) {
        this(looper, callback, false);
    }

    /**
     * 对当前线程使用，并设置处理程序是不是异步的，处理程序默认是同步的，如果为true，则为异步     
     * 处理
     * @hide
     */
    public Handler(boolean async) {
        this(null, async);
    }

    /**
     * 可以设置处理的Callback，和设置处理是否异步
     * @hide
     */
    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

    /**
     * 可以设置Looper，Callback，和是否异步
     * @hide
     */
    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

## Handler发送消息

```
    public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }

    public final boolean post(Runnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
    private boolean enqueueMessage(MessageQueue queue, 
                                   Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

可以看出sendMessage是直接发送Message的，而post则是将Runable作为一个Callback接口放入消息中再发送的，而两者最后都在经过了层层的调用，最后调用了enqueMessage方法，将一个消息插入到了消息队列，**并且将自己的对象给了msg.target。**

## Handler的消息处理

之前在Looper的loop方法中，会调用msg.target.dispatchMessage，实际上就是调用了handler.dispatchMessage。

```
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

如果消息的Callback不为null，则执行Message的Callback方法，如果handler的Callback处理接口不为null，则调用这个接口的handlerMessage，如果没有这个接口或者接口的handlerMessage没有处理，则调用handler对象的handlerMessage，而HandlerCallBack实际上非常的简单，就只是执行了Message.callback.run\(\)方法

# 主线程的消息循环模型

ActivityThread通过ApplicationThread和AMS进行进程间通信，AMS以进程间通信的方式完成ActivityThread的请求后会回调ApplicationThread中的Binder方法，然后Application会向ActivityThread.H发送消息，H收到消息后会将ApplicationThread中的逻辑切换到ActivityThread中执行，即切换到主线程中去执行，这个过程就是主线程的消息循环模型

# 总结： 

当我们调用`handler.sendMessage(msg)`方法发送一个`Message`时，实际上这个`Message`是发送到**与当前线程绑定**的一个`MessageQueue`中，然后**与当前线程绑定**的`Looper`将会不断的从`MessageQueue`中取出新的`Message`，调用`msg.target.dispathMessage(msg)`方法将消息分发到与`Message`绑定的`handler.handleMessage()`方法中。

一个`Thread`对应多个`Handler`一个`Thread`对应一个`Looper`和`MessageQueue`，`Handler`与`Thread`共享`Looper`和`MessageQueue`。`Message`只是消息的载体，将会被发送到**与线程绑定的唯一的**`MessageQueue`中，并且被**与线程绑定的唯一的**`Looper`分发，被与其自身绑定的`Handler`消费。

