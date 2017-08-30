### Service的生命周期

![](https://timgsa.baidu.com/timg?image&quality=80&size=b10000_10000&sec=1488617077&di=72cd6b25e1308e12e6640c9d4099d276&src=http://images.cnitblog.com/blog/670764/201409/122149233873958.png)

### 启动方法

1. 通过**Context.bindService\(\)**方法来进行Service与Context的关联并启动，并且Service的生命周期依附于Context，通过**Context.unbindService\(\)**方法结束。

2. 通过**Context.startService\(\)**方法去启动一个Service，此时Service的生命周期与启动它的Context无关，通过**Context.stopService\(\)**方法结束。

3. 要注意的是，**使用Service必须在AndroidManifest.xml里注册**，就像这样 :

```
<service
android:name=".packname.servicename"
android:enabled="true" /      >
```

### 如何保证Service不被杀死

* 1.Service设置成START\_STICKY

  kill后会被重启（等待5秒左右），重传Intent，保持与重启前一样

* 2.onDestroy方法里重启Service

  Service+broadcast的方式，就是当Service走onDestory\(\)的时候，发送一个自定义的广播，当收到广播的时候，重新启动Service。也可以直接在onDestroy\(\)里startService。

* 3.提升进程优先级。（有6个优先级）

* 4.监听系统广播判断Service状态

  通过系统的一些广播，比如 : 手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的Service是否还存活，别忘记加权限。



