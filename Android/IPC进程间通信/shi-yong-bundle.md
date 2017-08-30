# 为什么使用Bundle？ {#为什么使用bundle}

四大组件中的三大组件（Activity， Service， Receiver）都支持在Intent传递Bundle数据，由于Bundle实现了Parcelable接口，所以可以十分方便的在进程间传输，当然我们传输的数据必须能够被序列化，比如基本类型、实现了Parcelable接口的对象、实现了Serializable接口的对象以及一些[Android](http://lib.csdn.net/base/android)所支持的特殊对象。

# 如何利用Bundle进行通信？ {#如何利用bundle进行通信}

## 1. 传输Bundle支持的数据类型 {#1-传输bundle支持的数据类型}

## 2. 将数据放置到Bundle中 {#2-将数据放置到bundle中}

```
User user = new User("测试玩家", "num123");
                Bundle bundle = new Bundle();
                bundle.putParcelable("user", user);
                Intent intent = new Intent(this, BundleActivity.class);
                intent.putExtra("user", bundle);
                startActivity(intent);
```

## 3. 在目标进程组件中将Bundle中的数据还原 {#3-在目标进程组件中将bundle中的数据还原}

```
        Bundle data = getIntent().getBundleExtra("user");
        User user = data.getParcelable("user");
        Log.d(TAG, "onCreate: userName = " + user.getName());
        Log.d(TAG, "onCreate: id = " + user.getId());

```

## 测试结果 {#测试结果}

两个处在不同的进程:  
![](http://img.blog.csdn.net/20170226212213270?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjcwMzUxMjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

进程间通信：

> 02-26 21:07:44.426 17850-17850/? D/BundleActivity: onCreate: userName =[测试](http://lib.csdn.net/base/softwaretest)玩家  
> 02-26 21:07:44.426 17850-17850/? D/BundleActivity: onCreate: id = num123



