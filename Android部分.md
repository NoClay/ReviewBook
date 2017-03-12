# Android部分

## 1.如何解决WebView内存泄漏

**原因：**WebView加载了大量的图片导致内存占用，再退出该界面后，即使在包含该WebView的Activity的destory方法中，对webView进行内存回收没有效果。

**解决：**

1. 不再xml文件中使用WebView，而是在代码中动态添加，比如在xml中使用一个LinearLayout作为WebView的父布局，在需要使用的时候动态添加。缺点：如果在WebView构造的时候传入ApplicationContext，当WebView想要弹出一个dialog，都会导致应用崩溃，如果传入ActivityContext的话，会导致对内存的引用无法回收。

   ```java
   protected void onDestroy() {
         super.onDestroy();
         mWebView.removeAllViews();
         mWebView.destroy()
   }
   ```

2. 使用单独的进程，将需要WebView的Activity单独一个进程，分配单独的虚拟机，利用IPC机制进行通信，同时与上一种结合起来，在准备销毁的时候，使用System.exit\(0\);强制退出虚拟机，即杀死当前新进程，即可实现。

   如果出现了闪屏可以利用给Activity设置Theme解决：

   ```
       <style name="ActivityTheme" parent="AppTheme">
           <item name="android:windowBackground">@color/white</item>
         	<!--设置窗口透明-->
           <item name="android:windowIsTranslucent">true</item>
       </style>
   ```

## 2. IPC机制

**原因：**Android为每一个应用分配了一个独立的虚拟机，也就是说为每一个进程都分配了一个独立的虚拟机，这样导致不同的虚拟机在内存分配上存在不同的地址空间。这就是问题1，2的原因，问题3是因为SharedPrefences不支持两个进程同时去执行写操作，否则会导致一定的机率的数据丢失，这是因为Shared{refences的底层是通过读、写xml文件实现的，并发写很显然是会出问题的。问题4是因为当一个新的组件跑在一个新的进程的过程中的时候，系统会在创建新的进程的时候分配独立的虚拟机，所以这个过程其实就是启动一个应用的过程，所以会重复的创建新的Application。总结来讲：不同的进程会拥有不同的虚拟机，application, 以及内存空间。

**方式：**

1. ​

   | 名称 | 优点 | 缺点 | 场景 |
   | :--- | :--- | :--- | :--- |
   | Bundle | 简单易用 | 只能传输Bundle支持的数据类型 | Activity，Service，Receiver间的进程间通信 |
   | 文件共享 | 简单易用 | 不适合高并发场景，并且无法做到进程间的即时通信 | 无并发访问情形，交换简单的数据是实时不高的场景 |
   | AIDL（继承自接口名.Stub实现接口内方法） | 功能强大，支持一对多并发通信，支持实时通信 | 使用稍复杂，需要处理好线程同步 | 一对多通信具有RPC需求 |
   | Messenger（轻量级IPC，需要一个Handler） | 功能一般，支持一对多串行通信，支持实时通信 | 不能很好的处理高并发情形，不支持RPC，数据通过Message进行传输，因此zhinengchuanshuBundle支持的数据类型 | 低并发的一对多即时通信，无RPC需求，或者无需要返回结果的RPC需求 |
   | ContentProvider（底层实现可以利用文件，SQLite，内存对象等） | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过Call方法拓展其他操作 | 可以理解为受约束的AIDL，主要提供数据源的增删改查的操作 | 一对多的进程间的数据共享 |
   | Socket | 功能强大，可以通过网络传输字节流，支持一对多并发实时通信 | 实现细节稍微繁琐，不支持直接的RPC | 网络数据交换 |
   | BinderPool（利用BinderPool来管理Binder） | 避免了多次创建Service | 使用稍微复杂，需要额外实现BinderPool的AIDL接口 |  |

**目的：**

1. 数据传输：一个进程需要将它的数据发送给另一个进程，发送的数据量在一个字节到几兆字节之间。

2. 共享数据：多个进程想要操作共享数据，一个进程对共享数据的修改，别的进程应该立刻看到。

3. 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要通知父进程）。

4. 资源共享：多个进程之间共享同样的资源。为了作到这一点，需要内核提供锁和同步机制。

5. 进程控制：有些进程希望完全控制另一个进程的执行（如Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

## 3. Binder

Binder实现了IBinder的接口，从IPC的角度来讲，Binder是一种跨进程的通信方式，从Android Framework角度来讲，他是ServiceManager连接各种Manager和相应的ManagerService的桥梁；从Android的应用层来讲，Binder是服务端和客户端之间进行通信的媒介。

**Binder内部概述**

1. DESCRIPTOR Binder的唯一标识， 一般用于当前Binder的类名表示

2. asInstance 用于将服务端的Binder对象转换为客户端所需的AIDL接口类型的对象，如果客户端和服务端位于同一对象，那么此方法返回的就是服务端对象本身，否则返回的就是熊封住nag后的Stub.proxy对象

3. asBinder 返回当前的Binder对象

4. onTransact 如果此方法返回false，那么客户端的请求会失败，因此我们可以利用这个特性做权限验证。

5. Proxy\#MethodName Proxy是一个代理类，当客户端远程调用此方法的时候，内部实现为：创建该方法所需要的输入型对象Parcel对象\_data,输出型对象\_reply和返回值对象.然后将方法参数写入\_data，接着调用transace方法来发起RPC（远程过程调用）请求，同时当前线程被挂起，然后服务端的onTransact方法会被调用，直到RPC过程返回后，当前线程继续执行，并从\_reply中取出返回结果

## 4. 怎么实现序列化？

1. 实现Serializable接口：

   只需要一个long serialVersionUID，即可实现序列化，在序列化对象的时候可以使用ObjectOutputStream来序列化，ObjectInputStream实现反序列化

2. 实现Parcelable接口：

   1. 序列化由writeToParcel\(Parcel dest, int flags\)来实现

   2. 反序列化由 Creator&lt;T extends Class&gt;CREATOR = new Creator&lt;User extends Class&gt;\(\)来完成

   3. 内容描述由describeContents\(\)来实现，几乎所有的情况下这个方法都应该返回0，仅当当前对象中存在文件描述符的时候，返回1

3. 对比：

   两个接口都可以实现序列化，但是相比来说Serializable是[Java](http://lib.csdn.net/base/javase)里边的实现序列化的接口，序列化和反序列化的过程中都需要大量的IO操作，而Parcelable是Android的序列化方式，因此更适用Android平台，他的缺点就是使用麻烦。我们首选的序列化方式应该是Parcelable，但是当遇到将对象序列化到存储设备中或者将对象序列化后通过网络传输建议是使用Serializable接口。

## 5. ListView的优化方式？

### 1. 优化方式

1. 重用convertView：防止重复创建ItemView

2. 使用ViewHolder：findViewById太耗时，通过convertView.setTag操作，可以直接取出而不用重复findViewById

3. 使用static class ViewHolder：如果声明成员类不要求访问外围实例，就要始终把static修饰符放在它的声明中，使它成为静态成员类，而不是非静态成员类。因为非静态成员类的实例会包含一个额外的指向外围对象的引用，保存这份引用要消耗时间和空间，并且导致外围类实例符合垃圾回收时仍然被保留。如果没有外围实例的情况下，也需要分配实例，就不能使用非静态成员类，因为非静态成员类的实例必须要有一个外围实例。

4. 在列表里面有图片的情况下，监听滑动不加载图片

5. 多个不同的布局，可以创建不同的ViewHolder和convertView进行重用

### 2.RecyclerView和ListView的对比？

1. RecyclerView支持线性布局，网格布局，瀑布流（StaggeredGridLayoutManager）

2. 支持横向滑动和纵向滑动

3. 支持局部刷新

4. RecyclerView需要自定义OnClickListener之类

5. [需要自己添加HeaderView和FooterView：](http://blog.csdn.net/lmj623565791/article/details/51854533)











