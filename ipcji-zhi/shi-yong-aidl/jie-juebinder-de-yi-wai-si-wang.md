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

