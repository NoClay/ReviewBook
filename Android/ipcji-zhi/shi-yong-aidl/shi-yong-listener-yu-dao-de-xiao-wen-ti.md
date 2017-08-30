上述中，我们采用CopyOnWriteArrayList来存储了OnNewBookArrivedListener的时候，我们将一个对象从一个进程传递到另一个进程，虽然注册和注销的时候使用的是同一个客户端对象，但是在两个进程中是两个不同的全新的对象，所以我们在上述代码实例注销的时候会出现：

> 02-27 19:57:16.270 1035-1035/com.example.no\_clay.messagertest D/BookManagerActivity: onServiceConnected: list type = java.util.ArrayList  
> 02-27 19:57:16.270 1035-1035/com.example.no\_clay.messagertest D/BookManagerActivity: onServiceConnected: list = \[com.example.no\_clay.messagertest.Data.Book@42caa858, com.example.no\_clay.messagertest.Data.Book@42caa8f0\]  
> 02-27 19:57:16.270 1035-1035/com.example.no\_clay.messagertest D/BookManagerActivity: onNewBookArrivedListener: one book new = 呵呵呵呵呵num1232  
> 02-27 19:57:16.271 1035-1035/com.example.no\_clay.messagertest D/BookManagerActivity: onServiceConnected: list type = java.util.ArrayList  
> 02-27 19:57:16.271 1035-1035/com.example.no\_clay.messagertest D/BookManagerActivity: onServiceConnected: list = \[com.example.no\_clay.messagertest.Data.Book@42cab858, com.example.no\_clay.messagertest.Data.Book@42cab8f0, com.example.no\_clay.messagertest.Data.Book@42cab990\]  
> 02-27 19:57:20.925 1035-1035/com.example.no\_clay.messagertest D/BookManagerActivity: onDestroy: unregister listener = com.example.no\_clay.messagertest.Item244.BookManagerActivity$2@42c925f8  
> 02-27 19:57:20.926 2386-2408/com.example.test D/BookManagerActivity: unregisterListener: not found the listener

# 怎么解决这个问题？ {#怎么解决这个问题}

我们可以使用RemoteCallbackList，这个是系统专门提供的用于删除跨进程Listener的接口，它是一个泛型，可以管理任何的AIDL接口，其实它的内部实现基于Map接口，key为Binder类型，value为Callback类型，而Callback中封装了真正的远程listener。

```
    IBinder key = listener.asBinder();
    Callback value = new Callback(listener, cookie)
```

虽说我们跨进程传输了Listener，但其核心Binder还是不变的，我们可以利用这个类将listener保存起来，同时RemoteCallbackList内部自动实现了线程同步的功能， 所以我们使用它来进行注册和注销的时候，不需要做额外的线程同步。  
注意：遍历RemoteCallbackList时beginBroadcast获取内部size，和finishBroadcast需要成对出现。  
修改后的Service如下：

```java
public class BookManagerService extends Service {
    private static final String TAG = "BookManagerActivity";
    private CopyOnWriteArrayList<Book> mBooks = new CopyOnWriteArrayList<>();
    private RemoteCallbackList<IOnNewBookArrivedListener> mListeners
            = new RemoteCallbackList<>();
    Binder mBinder = new IBookManager.Stub() {
        @Override
        public List<Book> getBookList() throws RemoteException {
            return mBooks;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            onNewBookArrived(book);
        }

        @Override
        public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException {
            mListeners.register(listener);
        }

        @Override
        public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException {
            mListeners.unregister(listener);
            final int size = mListeners.beginBroadcast();
            Log.d(TAG, "unregisterListener: size = " + size);
            mListeners.finishBroadcast();
        }
    };

    private void onNewBookArrived(Book book) {
        mBooks.add(book);
        final int size = mListeners.beginBroadcast();
        for (int i = 0; i < size; i++) {
            IOnNewBookArrivedListener listener = mListeners.getBroadcastItem(i);
            try {
                listener.onNewBookArrivedListener(book);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        mListeners.finishBroadcast();
    }

    @Override
    public void onCreate() {
        super.onCreate();
        mBooks.add(new Book("唯物主义", "num123"));
        mBooks.add(new Book("我也不知道是什么", "num143"));
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
}
```

# 注意 {#注意}

1. 客户端的onServiceConnected和onServiceDisConnected方法都运行在UI线程中，所以不可以在它们礼拜内调用服务器的耗时方法
2. 服务器端的方法本身运行在服务器的Binder线程池中，所以服务端方法本身可以执行大量耗时操作，这个时候切记不要在服务端开线程进行异步操作，除非你明确知道自己在干什么
3. 服务端调用客户端中的listener的方法时，被调用的方法运行在客户端的Binder线程池中，所以同样不可以在服务端中调用客户端的耗时方法
4. 客户端的listener方法运行在Binder线程池中，所以我们不能在里边进行UI操作，我们应该使用Handler切换到UI线程。



