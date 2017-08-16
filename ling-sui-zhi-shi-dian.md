# 为什么在Service中创建子线程而不是在Activity中？

这是因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作。但是Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。

# Android四大组件

1. **Activity**: 通常是一个单独的屏幕，可以显示一些控件也可以监听并处理用户的事件做出响应。

2. **Service**: 是一段长生命周期的，没有用户界面的程序，可以用来开发如监控类程序。

3. **Content Provider**: 将指定数据集提供给其他应用程序，其他应用可以通过Content Resolver类从中获取或存入数据。

4. **Broadcast Receiver**: 对外部事件进行过滤只对感兴趣的外部事件进行接收并做出响应。

5. 四大组件（除动态广播）都需要在AndroidManifest.xml中注册

---

## Android五种布局

1. **FrameLayout**\(框架布局\)

   * 这种布局默认没有对子控件的的布局位置进行控制，所有子控件都默认在视图的**左上角**，通过设置`android:layout_margin`，`android:layout_gravity`等属性去控制子控件的布局位置

   * 这种布局是五种布局中**最简单**的布局

2. **LinearLayout**\(线性布局\)

   * 这种布局是一行（或一列）仅控制一个控件的线性布局，需要格外注意`android:orientation`属性

   * 这种布局在**理论上绘制效率最高**

3. **RelativeLayout**\(相对布局\)

   * 这种布局是相对自由的布局，子控件的的布局位置是按照相对位置来计算的

   * 这种布局是**最常用、最灵活**的布局，在**项目开发中绘制效率最高**

4. **AbsoluteLayout**\(绝对布局\)

   * 这种布局可以放置多个控件，通过设置子控件的x，y属性控制控制子控件的布局位置

   * 这种布局**适应性差、不常用**，可以使用RelativeLayout替代

5. **TableLayout**\(表格布局\)

   * 这种布局是将子元素的位置分配到行 或 列中，一个TableLayout由许多的TableRow组成。

---

## Android数据存储形式

1. **SQLite**:

   * SQLite是一个轻量级的数据库，支持基本的SQL语法

   * **常用于对并发数量要求不高的本地查询**

2. **SharedPreference**:

   * 其本质是使用**xml**文件存储数据

   * **常用于存储较简单的参数设置**

3. **File**:

   * 即常说的文件存储方法，缺点是更新数据困难

   * **常用于存储大数量的数据**

4. **ContentProvider**:

   * Android系统中能实现所有应用程序共享的一种数据存储方式，无论数据来源是什么，都把数据组织成表格

   * **常用于在不同的应用程序之间共享数据**

---

### 自定义View

1. 继承View完全自定义或继承View的派生子类

2. 自定义属性的声明（res/values/attrs.xml）

   * 相关属性 : left、right、top、bottom、x、y、translationX、translationY
   * 需要自定义设置的一些东西

3. 自定义属性的获取（构造方法中）

4. 重写测量方法onMeasure

5. 重写布局方法onLayout\(ViewGroup\)

6. 重写绘制方法onDraw

7. 重写事件分发机制（解决滑动冲突）

   * 相关事件 : MotionEvent（事件序列）、TouchSlop（滑动最小距离）、VelocityTracker（速度追踪）、GestureDetector（手势检测）、Scroller（弹性滑动）
   * 外部拦截：onInterceptTouchEvent，父布局需要滑动事件的时候，则触发
   * 内部拦截：我们在父容器中不拦截所有事件，所有的事件都传递给子元素，如果子元素需则处理，否则交给父容器，需要配合requestDisallowInterceptTouchEvent方法。父布局重写onInterceptTouchEvent什么都不拦截，等待子布局挑选完

---

### 动画

1. **tween animation**（补间动画）

   * 通过指定View的初末状态和变化时间、方式，对View的内容完成一系列的图形变换来实现动画效果。有4种样式 :

     * Alpha : 渐变透明度动画效果

     * Scale : 渐变尺寸伸缩动画效果

     * Translate : 移动动画效果

     * Rotate : 旋转动画效果）

2. **frame animation**（帧动画）

   * 顺序播放事先做好的图像，与电影类似。通过AnimationDrawable控制、animation-list.xml布局

3. **property animation**（属性动画）

   * 属性动画的运行机制是通过不断地对值进行操作来实现的，而初始值和结束值之间的动画过渡就是由ValueAnimator这个类来负责计算的。

# Context的区别

Activity和Service以及Application的Context是不一样的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper

* 每一个Activity和Service以及Application的Context都是一个新的ContextImpl对象
* getApplication\(\)用来获取Application实例的，但是这个方法只有在Activity和Service中才能调用的到。那么也许在绝大多数情况下我们都是在Activity或者Service中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext\(\)方法，getApplicationContext\(\)比getApplication\(\)方法的作用域会更广一些，任何一个Context的实例，只要调用getApplicationContext\(\)方法都可以拿到我们的Application对象。
* Activity在创建的时候会new一个ContextImpl对象并在attach方法中关联它，Application和Service也差不多。ContextWrapper的方法内部都是转调ContextImpl的方法
* 创建对话框传入Application的Context是不可以的
* 尽管Application、Activity、Service都有自己的ContextImpl，并且每个ContextImpl都有自己的mResources成员，但是由于它们的mResources成员都来自于唯一的ResourcesManager实例，所以它们看似不同的mResources其实都指向的是同一块内存
* Context的数量等于Activity的个数 + Service的个数 + 1，这个1为Application

---

## Android引用类型

* **强引用**: 最普遍的一种引用方式，如String s = "abc"，变量s就是字符串“abc”的强引用，只要强引用存在，则垃圾回收器就不会回收这个对象。

* **软引用**（SoftReference）用于描述还有用但非必须的对象，如果内存足够，不回收，如果内存不足，则回收。一般用于实现内存敏感的高速缓存，软引用可以和引用队列ReferenceQueue联合使用，如果软引用的对象被垃圾回收，JVM就会把这个软引用加入到与之关联的引用队列中。

* **弱引用**（WeakReference）弱引用和软引用大致相同，弱引用与软引用的区别在于 : 只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

* **虚引用**（PhantomReference）就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。 虚引用主要用来跟踪对象被垃圾回收器回收的活动。

* **虚引用与软引用与弱引用的区别**: 虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

* **虚引用与软引用与弱引用的使用场景**: 如果想尽量避免OutOfMemory异常的发生，则使用**软引用**。如果对于应用的性能更在意，想尽快回收一些占用内存比较大的对象，则使用**弱引用**。如果对象可能会经常使用，则使用**软引用**。如果该对象不被使用的可能性更大些，则使用**弱引用**。

![](http://p1.bpimg.com/567571/fd8e4512f4678098.png)

---

## Android多线程编程

* **线程创建方法**:

  1. 继承自Thread类，重写父类run方法，使用start方法启动线程

  2. 实现Runnable接口，实现run方法

  3. 使用匿名类启动线程

* **线程区别start和run**: start方法（就绪态）和run（运行态）的区别

* **线程池概念**: Java中Executors接口表示线程池

* **线程池目的**: 为了尽可能对线程资源进行复用，而不是频繁的创建和销毁

* **线程池设计**:

  1. 线程池管理器 : 用于启动、停用、管理线程池

  2. 工作线程 : 线程池中的线程

  3. 请求接口 : 创建请求对象，以供工作线程调度任务的执行

  4. 请求队列 : 用于存放和提取请求

  5. 结果队列 : 用于存储请求执行后返回的结果

![](/thread.jpeg)

* AsycTask、HandlerThread、IntentService使用场景

  1. **IntentService是Serivce+Handler的结合产物**

     * 可以在onHandleIntent直接处理耗时操作

     * 本地Serivce和远程Serivce不能在onStart方法中执行耗时操作，只能放在子线程中进行处理，当有新的intent请求过来都会线onStartCommond将其入队列，当第一个耗时操作结束后，就会调用onHandleIntent处理下一个耗时操作，都执行完了自动执行onDestory销毁IntengService服务。

  2. **HandlerThread继承自Thread**

     * 这是一种可以使用 Handler 的 Thread，在 run 方法中通过 Looper.prepare\(\) 来创建消息队列，并通过 Looper.loop\(\) 来开启消息循环。
     * 用来托管主线程的Handler，**利用异步线程的方式来处理主线程的UI，减轻主线程的负担**，我们会经常用Handler在子线程中更新UI线程，而HandlerThread新建拥有Looper的子线程又有什么用呢？**必然是执行耗时操作。**举个例子，在轮播图中，我们每5秒需要切换一下显示的图片，如果我们将这种长时间的反复调用操作放到UI线程中，虽说可以执行，但是这样的操作多了之后，很容易会让UI线程卡顿甚至崩溃。 于是，就必须在子线程中调用这些了。 HandlerThread继承自Thread，一般适应的场景，便是集Thread和Handler之所长，适用于会长时间在后台运行，并且间隔时间内会调用的情况，比如上面所说的轮播图，又比如天气预报定时更新天气等等。

  3. **AsyncTask是thread池+Handler的结合产物**

     * 可以更方便地更新UI线程，减少程序中线程过多开销过大，操作和管理更加方便

     * execute\(\)在父线程中调用，在onPreExecute\(\)、onProgressUpdate\(Progress…\)、onPostExecute\(Result\)都在父线程中执行，doInBackground\(Params…\)在子线程中执行

  4. 总而言之，数据量多且复杂使用**Thread+Handler**，数据简单实现代码简单**AsyncTask**，**IntentService**解决了传统的Service中处理完耗时操作忘记停止并销毁Service的问题

![](http://o7gy5l0ax.bkt.clouddn.com/asynctask_2.png)

---

## Android性能优化

1. **合理管理内存**

   * 节制的使用Service

   * 当界面不可见时释放内存

   * 当内存紧张时释放内存

   * 避免在Bitmap上浪费内存

   * 是有优化过的数据集合

   * 知晓内存的开支情况

   * 谨慎使用抽象编程

   * 尽量避免使用依赖注入框架

   * 使用多个进程

2. **分析内存使用情况**

   * Android的GC操作

   * Android中内存泄漏

3. **高性能编码优化**

   * 避免创建不必要的对象

   * 静态优于抽象

   * 对常量使用static final修饰符

   * 使用增强型for循环语法

   * 多使用系统封装好的API

   * 避免在内部调用Getters/Setters方法

4. **布局优化技巧**

   * 重用布局文件

   * 仅在需要时才加载布局

# Android长连接，怎么处理心跳机制

心跳机制

1. 心跳机制是定时发送一个自定义的结构体\(心跳包\)，让对方知道自己还活着，以确保连接的有效性的机制。

   当一台[智能](http://lib.csdn.net/base/aiplanning)手机连上移动网络时，其实并没有真正连接上Internet，运营商分配给手机的IP其实是运营商的内网IP，手机终端要连接上Internet还必须通过运营商的网关进行IP地址的转换，这个网关简称为NAT\(NetWork Address Translation\)，简单来说就是手机终端连接Internet 其实就是移动内网IP，端口，外网IP之间相互映射。

   为了减少网关NAT映射表的负荷，如果一个链路有一段时间没有通信时就会删除其对应表，造成链路中断，正是这种刻意缩短空闲连接的释放超时，原本是想节省信道资源的作用，却让应用不得以远高于正常频率发送心跳来维护推送的长连接。

   另外，长连接比较耗电。

2. [Android](http://lib.csdn.net/base/android)系统的推送和[iOS](http://lib.csdn.net/base/ios)的推送有什么区别

   首先我们必须知道，所有的推送功能必须有一个`客户端和服务器的长连接`，因为推送是由服务器主动向客户端发送消息，如果客户端和服务器之间不存在一个长连接，那么服务器是无法来主动连接客户端的。因而推送功能都是基于长连接的基础上的。

   [ios](http://lib.csdn.net/base/ios)长连接是`由系统来维护的`，也就是说苹果的IOS系统在系统级别维护了一个客户端和苹果服务器的长链接，IOS上的所有应用上的推送都是先将消息推送到苹果的服务器然后将苹果服务器通过这个系统级别的长连接推送到手机终端上，这样的的几个好处为：

   * 在手机终端始终只要维护一个长连接即可，而且由于这个长连接是系统级别的不会出现被杀死而无法推送的情况。

   * 省电，不会出现每个应用都各自维护一个自己的长连接。

   * 安全，只有在苹果注册的开发者才能够进行推送，等等。

   [android](http://lib.csdn.net/base/android)的长连接是`由每个应用各自维护的`，但是google也推出了和苹果技术[架构](http://lib.csdn.net/base/architecture)相似的推送框架，C2DM，云端推送功能，但是由于google的服务器不在中国境内，所以导致这个推送无法使用，android的开发者不得不自己去维护一个长连接，于是每个应用如果都24小时在线，那么都得各自维护一个长连接，这种电量和流量的消耗是可想而知的。虽然国内也出现了各种推送平台，但是都无法达到只维护一个长连接这种消耗的级别。

3. 推送的实现方式

   * **客户端不断的查询服务器，检索新内容，也就是所谓的pull或者轮询方式　**

   * **客户端和服务器之间维持一个TCP/IP长连接，服务器向客户端push　**

   * **服务器有新内容时，发送一条类似短信的信令给客户端，客户端收到后从服务器中下载新内容，也就是SMS的推送方式　**

苹果的推送系统和google C2DM推送系统其实都是在系统级别维护一个TCP/IP长连接，都是基于第二种的方式进行推送的。第三种方式由于运营商没有免费开放这种信令导致了这种推送在成本上是无法接受的，虽然这种推送的方式非常的稳定，高效和及时。

在TCP机制里面，本身是存在有`心跳包机制`的，也就是TCP选项`SO_KEEPALIVE`，系统默认是设置的`2小时的心跳频率`。

心跳包的机制，其实就是传统的长连接。或许有的人知道消息推送的机制，消息推送也是一种长连接 ，是将数据有服务器端推送到客户端这边从而改变传统的“拉”的请求方式。

下面介绍一下[安卓](http://lib.csdn.net/base/android)服务端和客户端两个数据请求的方式

* push 这个也就是由服务器推送到客户端这边 现在有第三方技术，比如极光推送。

* pull 这种方式就是客户端向服务器发送请求数据（http请求）



