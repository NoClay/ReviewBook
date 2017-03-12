# 什么是IPC？ {#什么是ipc}

IPC是Inter-Process Communication的缩写，含义是进程间通信，指的是不同进程之间进行数据交换的过程。一个进程可以包含许多的线程，但是可以只有一个主线程（在[Android](http://lib.csdn.net/base/android)里边叫作UI线程）。

# 如何开启多进程模式？ {#如何开启多进程模式}

名义上Android的多进程模式的开启只需要一行代码，但随之产生的许多问题却需要我们来进行考虑，比如进程间的数据的共享，内存共享等，如下为开启多进程模式代码：

```
<activity 
    android:name="!!!"
    android:label="@string/app_name"
    android:process=":remote">
</activity>
```

注意：precess中的：的含义是指在当前的进程名前边加上当前的包名，这是一种简写的方法，如果我们指定了进程名如“com.example.testipc.remote”,则当前进程为全局进程，但是以“:”开头的进程名为私有进程。

# 多进程会出现什么问题？ {#多进程会出现什么问题}

1. 静态成员和单例模式完全失效
2. 线程同步机制完全失效
3. SharedPrefences的可靠性下降
4. Application会多次创建

# 为什么会产生进程通信问题？ {#为什么会产生进程通信问题}

Android为每一个应用分配了一个独立的虚拟机，也就是说为每一个进程都分配了一个独立的虚拟机，这样导致不同的虚拟机在内存分配上存在不同的地址空间。这就是问题1，2的原因，问题3是因为SharedPrefences不支持两个进程同时去执行写操作，否则会导致一定的机率的数据丢失，这是因为Shared{refences的底层是通过读、写xml文件实现的，并发写很显然是会出问题的。问题4是因为当一个新的组件跑在一个新的进程的过程中的时候，系统会在创建新的进程的时候分配独立的虚拟机，所以这个过程其实就是启动一个应用的过程，所以会重复的创建新的Application。总结来讲：不同的进程会拥有不同的虚拟机，application, 以及内存空间。

# IPC基本概念 {#ipc基本概念}

IPC主要包括三部分内容：Serializable接口，Parcelable接口和Binder，其中个Serializable接口和Parcelable接口都是用来进行序列化和反序列化过程的接口。

##  {#serializable接口}

## Binder {#binder}

Binder实现了IBinder的接口，从IPC的角度来讲，Binder是一种跨进程的通信方式，从Android Framework角度来讲，他是ServiceManager连接各种Manager和相应的ManagerService的桥梁；从Android的应用层来讲，Binder是服务端和客户端之间进行通信的媒介。

### Binder内部概述 {#binder内部概述}

1. DESCRIPTOR Binder的唯一标识， 一般用于当前Binder的类名表示
2. asInstance 用于将服务端的Binder对象转换为客户端所需的AIDL接口类型的对象，如果客户端和服务端位于同一对象，那么此方法返回的就是服务端对象本身，否则返回的就是熊封住nag后的Stub.proxy对象
3. asBinder 返回当前的Binder对象
4. onTransact 如果此方法返回false，那么客户端的请求会失败，因此我们可以利用这个特性做权限验证。
5. Proxy\#MethodName Proxy是一个代理类，当客户端远程调用此方法的时候，内部实现为：创建该方法所需要的输入型对象Parcel对象\_data,输出型对象\_reply和返回值对象.然后将方法参数写入\_data，接着调用transace方法来发起RPC（远程过程调用）请求，同时当前线程被挂起，然后服务端的onTransact方法会被调用，直到RPC过程返回后，当前线程继续执行，并从\_reply中取出返回结果



