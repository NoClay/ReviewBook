# 为什么使用共享文件进行通信？ {#为什么使用共享文件进行通信}

共享文件是一种不错的进程间通信方式，两个进程通过读/写同一个文件来进行数据交换，由于[Android](http://lib.csdn.net/base/android)系统基于[Linux](http://lib.csdn.net/base/linux)，使得并发读、写不受限制，所以这可能出问题，但是对于一些时效性要求不严格的进程通信，使用文件也是不错的方式。

# 如何进行文件通信？ {#如何进行文件通信}

## 1. 需要交换的数据可以序列化 {#1-需要交换的数据可以序列化}

这里直接使用我们已经实现了Serializable和Parcelable接口的User作为例子

## 2. 将数据写入共享文件 {#2-将数据写入共享文件}

```
ObjectOutputStream out = null;
                try {
                    out = new ObjectOutputStream(
                            new FileOutputStream(MyConstants.CACHE_FILE_PATH));
                    out.writeObject(new User("测试玩家", "num123"));
                    Intent intent2 = new Intent(this, FileActivity.class);
                    startActivity(intent2);
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        out.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
```

## 3. 在目标进程组件中恢复 {#3-在目标进程组件中恢复}

```
new Thread(new Runnable() {
            @Override
            public void run() {
                User user = null;
                File cacheFile = new File(MyConstants.CACHE_FILE_PATH);
                if (cacheFile.exists()){
                    try {
                        ObjectInputStream in = new ObjectInputStream(new FileInputStream(cacheFile));
                        try {
                            user = (User) in.readObject();
                            Log.d(TAG, "onCreate: userName = " + user.getName());
                            Log.d(TAG, "onCreate: id = " + user.getId());
                        } catch (ClassNotFoundException e) {
                            e.printStackTrace();
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
```



