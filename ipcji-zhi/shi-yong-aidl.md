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



