# IntentService

它是一个抽象类：'public abstract class IntentService extends Service'， 我们必须实现它的子类才能够使用IntentService，IntentService可用于执行后台耗时的任务，当任务完成后会自动停止，同时由于IntentService是一种Service，那么它的优先级就要比单纯的线程的优先级高很多，所以IntentService比较适合执行一些高优先级的后台任务，因为它的优先级高不容易被系统杀死。
## 1.OnCreate()
我们首先看源码：
```java
    @Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
```
当IntentSErvice被第一次启动的时候，它的onCreate方法会被调用，并创建了一个HandlerThread，获取HandlerThread的Looper，并用HandlerThread的Looper构造一个Handler，我们不禁猜想，IntentService会将执行的细节放到HandlerThread中执行。
## 2.onStart()
onStart方法会被onStartCommand进行调用，onStartCommand在每次启动IntentService的时候都会调用一次，IntentService在onStartCommand中处理每个后台任务的Intent。
```java
    @Override
    public int onStartCommand(@Nullable Intent intent, 
    int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? 
        START_REDELIVER_INTENT : START_NOT_STICKY;
    }
        @Override
    public void onStart(@Nullable Intent intent, 
    int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
```
可以看到这里的onStart的方法将intent和startId传递给ServiceHandler，那么它是怎么处理的呢？
```java
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
```
可以看出这里将任务的处理继续下放，交给onHandlerIntent中处理，并且采用stopSelf(int startId)来自动停止服务（这里没有使用stopSelf(),是因为当一条消息结束之后，可能还有其他的消息未处理，所以这里采用stopSelf(int startId)会判断最近启动服务的次数是否和startId相当，如果相当就立刻停止服务，不想等则不停止服务）
## 3.onHandleIntent
方法的声明：
>     protected abstract void onHandleIntent(@Nullable Intent intent);

这是一个抽象方法，我们需要在子类中添加具体的实现。

