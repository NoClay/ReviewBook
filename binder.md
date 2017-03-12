Binder实现了IBinder的接口，从IPC的角度来讲，Binder是一种跨进程的通信方式，从Android Framework角度来讲，他是ServiceManager连接各种Manager和相应的ManagerService的桥梁；从Android的应用层来讲，Binder是服务端和客户端之间进行通信的媒介。

**Binder内部概述**

1. DESCRIPTOR Binder的唯一标识， 一般用于当前Binder的类名表示

2. asInstance 用于将服务端的Binder对象转换为客户端所需的AIDL接口类型的对象，如果客户端和服务端位于同一对象，那么此方法返回的就是服务端对象本身，否则返回的就是熊封住nag后的Stub.proxy对象

3. asBinder 返回当前的Binder对象

4. onTransact 如果此方法返回false，那么客户端的请求会失败，因此我们可以利用这个特性做权限验证。

5. Proxy\#MethodName Proxy是一个代理类，当客户端远程调用此方法的时候，内部实现为：创建该方法所需要的输入型对象Parcel对象\_data,输出型对象\_reply和返回值对象.然后将方法参数写入\_data，接着调用transace方法来发起RPC（远程过程调用）请求，同时当前线程被挂起，然后服务端的onTransact方法会被调用，直到RPC过程返回后，当前线程继续执行，并从\_reply中取出返回结果



