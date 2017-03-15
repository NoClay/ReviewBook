# Activity中的琐碎知识点

布局中的每一个View默认实现了onSaveInstanceState\(\)方法，这样的话，这个UI的任何改变都会自动地存储和在activity重新创建的时候自动地恢复。**但是这种情况只有在你为这个UI提供了唯一的ID之后才起作用，如果没有提供ID，app将不会存储它的状态。**

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

3. 自定义属性的获取（构造方法中）

4. 重写测量方法onMeasure

5. 重写布局方法onLayout\(ViewGroup\)

6. 重写绘制方法onDraw

7. 重写事件分发机制（解决滑动冲突）

   * 相关事件 : MotionEvent（事件序列）、TouchSlop（滑动最小距离）、VelocityTracker（速度追踪）、GestureDetector（手势检测）、Scroller（弹性滑动）

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



