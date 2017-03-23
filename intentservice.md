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
onStart方法会被onStartCommand进行调用

