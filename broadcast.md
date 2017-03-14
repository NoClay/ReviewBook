**广播接收器注册一共有两种形式 : 静态注册和动态注册.**



两者及其接收广播的区别:

1.动态注册广播不是常驻型广播，也就是说广播跟随activity的生命周期。注意: 在activity结束前，移除广播接收器。

 静态注册是常驻型，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。

2.当广播为有序广播时：

    1 优先级高的先接收

    2 同优先级的广播接收器，动态优先于静态

    3 同优先级的同类广播接收器，静态：先扫描的优先于后扫描的，动态：先注册的优先于后注册的。

3当广播为普通广播时：

         1 无视优先级，动态广播接收器优先于静态广播接收器

         2 同优先级的同类广播接收器，静态：先扫描的优先于后扫描的，动态：先注册的优先于后注册的。

动态注册代码:

```java
UpdateBroadcast  broadcast= new UpdateBroadcast();  
IntentFilter filter = new IntentFilter("com.unit.UPDATE");  
registerReceiver(broadcast, filter);  
```

静态注册代码：

```java
<receiver android:name="net.youmi.android.AdReceiver" >  
            <intent-filter>  
                <action android:name="android.intent.action.PACKAGE_ADDED" />  
                <data android:scheme="package" />  
            </intent-filter>  
 </receiver> 
```



