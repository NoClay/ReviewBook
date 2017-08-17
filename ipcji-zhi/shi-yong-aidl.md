# 为什么使用AIDL？ {#为什么使用aidl}

我们之前使用Messenger实现了IPC，但是Messenger是一个接着一个处理的，对于大量的并发请求，那么用messenger就不太合适了，同时Messenger主要是为了传递消息，很多时候我们需要跨进程调用服务器的方法。

# 怎么使用AIDL进程进程间通信？ {#怎么使用aidl进程进程间通信}

## 1. 服务端 {#1-服务端}

服务端需要简历一个Service来监听客户端的连接请求，然后创建一个AIDL文件，将暴露在客户端的接口在这个AIDL文件中声明，最后在Service中实现这个AIDL接口即可。  
BookManagerService.[Java](http://lib.csdn.net/base/javase)

```
public class BookManagerService extends Service {

    private CopyOnWriteArrayList<Book> mBooks = new CopyOnWriteArrayList<>();
    private CopyOnWriteArrayList<IOnNewBookArrivedListener> mListeners
            = new CopyOnWriteArrayList<>();
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

            if (!mListeners.contains(listener)){
                mListeners.add(listener);
            }
        }

        @Override
        public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException {
            if (mListeners.contains(listener)){
                mListeners.remove(listener);
            }
        }
    };

    private void onNewBookArrived(Book book) {
        mBooks.add(book);
        for (IOnNewBookArrivedListener listener :
                mListeners) {
            try {
                listener.onNewBookArrivedListener(book);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
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

## 2. 客户端 {#2-客户端}

客户端需要绑定服务端的Service，绑定成功后，将服务端反悔的Binder对象转狂欢为AIDL接口所属的类型，接着就可以调用AIDL中的方法。  
BookManagerActivity.java

```
public class BookManagerActivity extends AppCompatActivity {
    private static final String TAG = "BookManagerActivity";
    IBookManager mBookManager = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_book_manager);
        Intent intent = new Intent(this, BookManagerService.class);
        bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
    }

    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            try {
                mBookManager = bookManager;
                bookManager.registerListener(listener);
                List<Book> list = bookManager.getBookList();
                Log.d(TAG, "onServiceConnected: list type = " + list.getClass().getCanonicalName());
                Log.d(TAG, "onServiceConnected: list = " + list.toString());
                bookManager.addBook(new Book("呵呵呵呵呵", "num1232"));
                list = bookManager.getBookList();
                Log.d(TAG, "onServiceConnected: list type = " + list.getClass().getCanonicalName());
                Log.d(TAG, "onServiceConnected: list = " + list.toString());
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mBookManager = null;
        }
    };

    private IOnNewBookArrivedListener listener = new IOnNewBookArrivedListener.Stub(){

        @Override
        public void onNewBookArrivedListener(Book newBook) throws RemoteException {
            Log.d(TAG, "onNewBookArrivedListener: one book new = " + newBook.getName() + newBook.getId());
        }
    };

    @Override
    protected void onDestroy() {
        if (mBookManager != null && mBookManager.asBinder().isBinderAlive()){
            try {
                mBookManager.unregisterListener(listener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        unbindService(mServiceConnection);
        super.onDestroy();
    }


}
```

# 创建AIDL接口

AIDL文件中，可以接受的数据类型为：  
1. 基本数据类型（int/ long/ char/ boolean等）  
2. String和CharSequence  
3. List：只支持ArrayList，里面每个元素都必须能够被AIDL支持，例如CopyOnWriteArrayList（支持并发读、写），在Binder中会按照List的规范访问数据，并形成一个ArrayList  
4. Map：只支持HashMap，里面的每个元素都必须被AIDL所支持，包括key和value，例如ConcurrentHashMap  
5. Parcelable：支持所艘实现了Parcelable接口的对象  
6. AIDL：所有的AIDL接口本身也可以在AIDL文件中使用

AIDL中的参数，除了基本类型，其它类型的参数都需要标上方向，如in， out或者inout。AIDL接口支持方法，但不支持静态变量  
Book.aidl

```
// IBookManager.aidl
package
 com.example.no_clay.messagertest.Data;
// Declare any non-default types here with import statements
parcelable Book;
```

BookManager.aidl

```
// IBookManager.aidl
package com.example.no_clay.messagertest.Data;
import com.example.no_clay.messagertest.Data.Book;
import com.example.no_clay.messagertest.Data.IOnNewBookArrivedListener;
// Declare any non-default types here with import statements

interface IBookManager {
     List<Book> getBookList();
     void addBook(in Book book);
     void registerListener(IOnNewBookArrivedListener listener);
     void unregisterListener(IOnNewBookArrivedListener listener);
}
```

Listener.aidl

```
// IOnNewBookArrivedListener.aidl
package com.example.no_clay.messagertest.Data;
import com.example.no_clay.messagertest.Data.Book;
// Declare any non-default types here with import statements

interface IOnNewBookArrivedListener {
    void onNewBookArrivedListener(in Book newBook);
}
```

# 使用Listener遇到的问题

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

# 如何解决Binder的意外死亡 {#如何解决binder的意外死亡}

Binder通常会因为服务端进程意外停止，那么如何解决这种意外情况的出现呢？

## 1. 给Binder设置DeathRecipient死亡监听 {#1-给binder设置deathrecipient死亡监听}

当Binder死亡的时候，我们会收到binderDied方法的回调，在这个方法中我们可以重新连接远程服务， 具体方法如下：

```java
public class BookManagerActivity extends AppCompatActivity {
    private static final String TAG = "BookManagerActivity";
    IBookManager mBookManager = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_book_manager);
        Intent intent = new Intent(this, BookManagerService.class);
        bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
    }

    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            try {
                mBookManager.asBinder().linkToDeath(mRecipient, 0);
                mBookManager = bookManager;
                bookManager.registerListener(listener);
                List<Book> list = bookManager.getBookList();
                Log.d(TAG, "onServiceConnected: list type = " + list.getClass().getCanonicalName());
                Log.d(TAG, "onServiceConnected: list = " + list.toString());
                bookManager.addBook(new Book("呵呵呵呵呵", "num1232"));
                list = bookManager.getBookList();
                Log.d(TAG, "onServiceConnected: list type = " + list.getClass().getCanonicalName());
                Log.d(TAG, "onServiceConnected: list = " + list.toString());
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mBookManager = null;
        }
    };

    private IOnNewBookArrivedListener listener = new IOnNewBookArrivedListener.Stub(){

        @Override
        public void onNewBookArrivedListener(Book newBook) throws RemoteException {
            Log.d(TAG, "onNewBookArrivedListener: one book new = " + newBook.getName() + newBook.getId());
        }
    };

    @Override
    protected void onDestroy() {
        if (mBookManager != null && mBookManager.asBinder().isBinderAlive()){
            try {
                Log.d(TAG, "onDestroy: unregister listener = " + listener);
                mBookManager.unregisterListener(listener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        unbindService(mServiceConnection);
        super.onDestroy();
    }

    private IBinder.DeathRecipient mRecipient = new IBinder.DeathRecipient() {
        @Override
        public void binderDied() {
            if (mBookManager != null){
                mBookManager.asBinder().unlinkToDeath(mRecipient, 0);
                mBookManager = null;
                Intent intent = new Intent(BookManagerActivity.this
                        , BookManagerService.class);
                bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
            }
        }
    };

}
```

## 2. 在onServiceDisconnected中重新连接远程服务 {#2-在onservicedisconnected中重新连接远程服务}

这种方法极为简洁，故此不做说明。

## 对比： {#对比}

·两种方法所艘在的进程不同，onServiceDisconnected在客户端的UI线程中被回调，binderDied在客户端的Binder线程池中被回调，也就是说在binderDied中我们不可以访问UI。

# 如何进行权限验证

## 1.在onBind中验证： {#1在onbind中验证}

首先我们需要在xml文件中声明所需要的权限：

```xml
 <permission
        android:name="com.example.no_clay.messagertest.permission
    .ACCESS_BOOK_MANAGER_SERVICE"
        android:protectionLevel="normal">
```

接着在onBind中进行权限验证，如果没有权限，则返回空

```java
   @Override
    public IBinder onBind(Intent intent) {
        int check = checkCallingOrSelfPermission("com.example.no_clay.messagertest" +
                ".permission.ACCESS_BOOK_MANAGER_SERVICE");
        if (check == PackageManager.PERMISSION_DENIED){
            return null;
        }
        return mBinder;
    }
```

## 2. 在服务端的跨进程接口中的onTransact进行权限验证： {#2-在服务端的跨进程接口中的ontransact进行权限验证}

```java
 Binder mBinder = new IBookManager.Stub() {

        @Override
        public boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
            //利用Permission权限验证
            int check = checkCallingOrSelfPermission("com.example.no_clay.messagertest" +
                    ".permission.ACCESS_BOOK_MANAGER_SERVICE");
            if (check == PackageManager.PERMISSION_DENIED){
                return false;
            }
            //利用包名验证
            String packageName = null;
            String[] packages = getPackageManager()
                    .getPackagesForUid(getCallingUid());
            if (packages != null && packages.length > 0){
                packageName = packages[0];
            }
            if (!packageName.startsWith("com.example.no_clay")){
                return false;
            }
            return super.onTransact(code, data, reply, flags);
        }

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
```



