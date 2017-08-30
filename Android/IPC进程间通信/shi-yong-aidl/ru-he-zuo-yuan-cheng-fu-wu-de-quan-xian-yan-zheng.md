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



