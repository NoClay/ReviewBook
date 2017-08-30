# 为什么使用Messenger？ {#为什么使用messenger}

Messenger是一种轻量级的IPC方案，它的底层实现是AIDL，同时由于一次只是处理一个请求，那么我们就不需要考虑线程同步的问题。

# 怎样实现一个Messenger？ {#怎样实现一个messenger}

## 1. 服务端进程 {#1-服务端进程}

我们需要在服务端简历一个Service来处理客户端的连接请求，同时创建一个Handler，并通过它来创建一个Messenger对象，然后在Service的onBind中返回这个Messenger对象底层的Binder。

## 2. 客户端进程 {#2-客户端进程}

首先绑定服务端的Service，绑定成功后利用服务端返回的IBinder对象创建一个Messenger，通过这个Messenger就可以向客户端发送消息，消息类型为Message对象，如果需要服务端可以回应客户端，我们需要创建一个Handler并创建一个新的Messenger，并把这个Messenger对象通过Message的replyTo参数传递给服务端，服务端通过这个参数回应客户端。

例子如下：  
MessengerService

```
public class MessengerService extends Service {
    private static final String TAG = "MessengerService";
    private final Messenger mMessenger = new Messenger(new MessengerHandler());

    private static class MessengerHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MyConstants.MSG_FROM_CLIENT: {
                    Log.d(TAG, "handleMessage: receive msg from client: = "
                            + msg.getData().getString("msg"));
                    Messenger client = msg.replyTo;
                    Message replyMessage = Message.obtain(null, MyConstants.MSG_FROM_SERVICE);
                    Bundle bundle = new Bundle();
                    bundle.putString("reply", "I receive a message!");
                    replyMessage.setData(bundle);
                    try {
                        client.send(replyMessage);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                    break;
                }
                default:
                    super.handleMessage(msg);
            }
        }
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind: bindService");
        return mMessenger.getBinder();
    }

    @Override
    public boolean onUnbind(Intent intent) {
        Log.d(TAG, "onUnbind: unbindService");
        return super.onUnbind(intent);
    }
}
```

MessengerActivity:

```
public class MessengerActivity extends AppCompatActivity {
    private static final String TAG = "MessengerService";
    private Messenger mService;
    Context context = this;
    private Messenger mMessenger = new Messenger(new Handler(){
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what){
                case MyConstants.MSG_FROM_SERVICE:{
                    Log.d(TAG, "handleMessage: get = " + msg.getData().getString("reply"));
                    break;
                }
                default:super.handleMessage(msg);
            }
        }
    });
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mService = new Messenger(service);
            Message message = Message.obtain(null, MyConstants.MSG_FROM_CLIENT);
            Bundle bundle = new Bundle();
            bundle.putString("msg", "hello， this is the client!");
            message.replyTo = mMessenger;
            message.setData(bundle);
            try {
                mService.send(message);
            } catch (RemoteException e) {
                Log.e(TAG, "onServiceConnected: ", e);
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_messenger);
        Intent intent = new Intent(context, MessengerService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        unbindService(mConnection);
        super.onDestroy();
    }
}
```

XML文件声明：

```
     <activity android:name=".Item243.MessengerActivity"/>

        <service
            android:name=".Item243.MessengerService"
            android:process=":remote">
        </service>
```



