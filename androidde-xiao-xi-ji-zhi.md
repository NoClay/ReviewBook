# Android的消息机制概述

Android的消息机制主要是指Handler的运行机制，Handler的运行需要底层的MessageQueue和Looper的支撑。

* MessageQue：消息队列，以队列的形式提供插入和删除的工作，内部利用单链表的数据结构来存储信息列表

* Looper：消息循环，无限循环的处理消息，没有则等待

* ThreadLocal：并不是线程，用来在不同的线程中互不干扰的存储并提供数据，通过ThreadLocal可以十分轻松的获取每个线程的Looper。

**注意：**当然线程默认是没有Looper的，我们使用的主线程即ActivityThread在创建的时候就会进行Looper的初始化，这也是为什么可以在主线程中使用Handler的原因。

# 为什么要引入Handler机制？

Android的UI控件不是线程安全的，如果在多线程中并发访问可能会导致UI控件处于不可预期的状态，而加锁的话有两个缺点：**1.加锁机制会让UI访问的逻辑百年的复杂 2. 加锁机制会降低UI访问的效率，因为锁机制会阻塞某些线程的执行。**所以最为高效简单的方法就是采用单线程模型来处理UI事件，对于开发者来说也不是很麻烦，只是需要通过Handler切换一下UI访问的执行线程即可。

这也就是为什么我们在主线程访问UI，会抛出异常：

```
void checkThread(){
  if(mThread != Thread.currentThread()){
    throw new CalledFromWrongThreadException(
    "Only the original thread that created a view hiderarchy can touch its views.");
  }
}
```

**注意：**此处异常提示为：只有在创建View的线程才可以操作View，但一般的Activity我们都是在主线程中创建View的，所以我们一般称为主线程才可以操纵View

