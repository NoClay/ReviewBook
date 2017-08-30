**广播接收器注册一共有两种形式 : 静态注册和动态注册.**

两者及其接收广播的区别:

1.动态注册广播不是常驻型广播，也就是说广播跟随activity的生命周期。注意: 在activity结束前，移除广播接收器。

静态注册是常驻型，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。

2. 当广播为有序广播时：

```
1 优先级高的先接收

2 同优先级的广播接收器，动态优先于静态

3 同优先级的同类广播接收器，静态：先扫描的优先于后扫描的，动态：先注册的优先于后注册的。
```

3. 当广播为普通广播时：

```
1 无视优先级，动态广播接收器优先于静态广播接收器

2 同优先级的同类广播接收器，静态：先扫描的优先于后扫描的，动态：先注册的优先于后注册的。
```

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

**sendBroadcast\(\)**这个方法的广播是能够发送给所有广播接收者，按照注册的先后顺序，如果你这个时候设置了广播接收者的优先级，**优先级如果恰好与注册顺序相同，则不会有任何问题，如果顺序不一样，会出leaked IntentReceiver 这样的异常**，并且在前面的广播接收者不能调用abortBroadcast\(\)方法将其终止，如果调用会出BroadcastReceiver trying to return result during a non-ordered broadcast的异常，当然，先接收到广播的receiver可以修改广播数据。

**sendOrderedBroadcast\(\)**方法顾名思义就是priority的属性能起作用，并且在队列前面的receiver可以随时终止广播的发送。还有这个api能指定final的receiver，这个receiver是最后一个接收广播时间的receiver，并且一定会接收到广播事件，是不能被前面的receiver拦截的。实际做实验的情况是这样的，假设我有3个receiver依序排列，并且sendOrderedBroadcast\(\)方法指定了一个finalReceiver，那么intent传递给这4个Receiver的顺序为Receiver1--&gt;finalReceiver--&gt;Receiver2--&gt;finalReceiver--&gt;Receiver3--&gt;finalReceiver。这个特性可以用来统计系统中能监听某种广播的Receiver的数目。

**sendStickyBroadcast\(\)**

字面意思是发送粘性的广播，使用这个api需要权限android.Manifest.permission.BROADCAST\_STICKY,粘性广播的特点是Intent会一直保留到广播事件结束，而这种广播也没有所谓的10秒限制，10秒限制是指普通的广播如果onReceive方法执行时间太长，超过10秒的时候系统会将这个广播置为可以干掉的candidate，一旦系统资源不够的时候，就会干掉这个广播而让它不执行。

下面是广播接收者的生命周期以及一些细节部分：

1.广播接收者的生命周期是非常短暂的，在接收到广播的时候创建，onReceive\(\)方法结束之后销毁

2.广播接收者中不要做一些耗时的工作，否则会弹出Application No Response错误对话框

3.最好也不要在广播接收者中创建子线程做耗时的工作，因为广播接收者被销毁后进程就成为了空进程，很容易被系统杀掉

4.耗时的较长的工作最好放在服务中完成

