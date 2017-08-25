## 观察者模式

首先在Android中，我们往ListView添加数据后，都会调用Adapter的notifyDataChanged\(\)方法，其中使用了观察者模式。

当ListView的数据发生变化时，调用Adapter的notifyDataSetChanged函数，这个函数又会调用DataSetObservable的notifyChanged函数，这个函数会调用所有观察者\(AdapterDataSetObserver\)的onChanged方法，在onChanged函数中又会调用ListView重新布局的函数使得ListView刷新界面。

Android中应用程序发送广播的过程：

* 通过sendBroadcast把一个广播通过Binder发送给ActivityManagerService，ActivityManagerService根据这个广播的Action类型找到相应的广播接收器，然后把这个广播放进自己的消息队列中，就完成第一阶段对这个广播的异步分发。
* ActivityManagerService在消息循环中处理这个广播，并通过Binder机制把这个广播分发给注册的ReceiverDispatcher，ReceiverDispatcher把这个广播放进MainActivity所在线程的消息队列中，就完成第二阶段对这个广播的异步分发：
* ReceiverDispatcher的内部类Args在MainActivity所在的线程消息循环中处理这个广播，最终是将这个广播分发给所注册的BroadcastReceiver实例的onReceive函数进行处理：

观察者模式面向的需求是：A 对象（观察者）对 B 对象（被观察者）的某种变化高度敏感，需要在 B 变化的一瞬间做出反应。举个例子，新闻里喜闻乐见的警察抓小偷，警察需要在小偷伸手作案的时候实施抓捕。在这个例子里，警察是观察者，小偷是被观察者，警察需要时刻盯着小偷的一举一动，才能保证不会漏过任何瞬间。程序的观察者模式和这种真正的『观察』略有不同，观察者不需要时刻盯着被观察者（例如 A 不需要每过 2ms 就检查一次 B 的状态），而是采用**注册**\(Register\)**或者称为**订阅**\(Subscribe\)**的方式，告诉被观察者：我需要你的某某状态，你要在它变化的时候通知我。 Android 开发中一个比较典型的例子是点击监听器`OnClickListener`。对设置`OnClickListener`来说，`View`是被观察者，`OnClickListener`是观察者，二者通过`setOnClickListener()`

方法达成订阅关系。订阅之后用户点击按钮的瞬间，Android Framework 就会将点击事件发送给已经注册的`OnClickListener`。采取这样被动的观察方式，既省去了反复检索状态的资源消耗，也能够得到最高的反馈速度。当然，这也得益于我们可以随意定制自己程序中的观察者和被观察者，而警察叔叔明显无法要求小偷『你在作案的时候务必通知我』。



