# 背压问题

**背压是指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略**

简而言之，**背压是流速控制的一种策略**。

需要强调两点：

* 背压策略的一个前提是**异步环境**，也就是说，被观察者和观察者处在不同的线程环境中。
* 背压（Backpressure）并不是一个像flatMap一样可以在程序中直接使用的操作符，他只是一种控制事件流速的策略。

## 响应式拉取（reactive pull）

首先我们回忆之前那篇《关于Rxjava最友好的文章》，里面其实提到，在RxJava的观察者模型中，**被观察者是主动的推送数据给观察者，观察者是被动接收的**。而响应式拉取则反过来，**观察者主动从被观察者那里去拉取数据，而被观察者变成被动的等待通知再发送数据**。

结构示意图如下：

![](https://dn-mhke0kuv.qbox.me/45aaa155f3b4ed976a65.png)

  


观察者可以根据自身实际情况按需拉取数据，而不是被动接收（也就相当于告诉上游观察者把速度慢下来），最终实现了上游被观察者发送事件的速度的控制，实现了背压的策略。

# 根源

产生背压问题的根源就是上游发送速度与下游的处理速度不均导致的，所以如果想要解决这个问题就需要通过匹配两个速率达到解决这个背压根源的措施。

通常有两个策略可供使用：

1. 从数量上解决，对数据进行采样
2. 从速度上解决，降低发送事件的速率
3. 利用flowable和subscriber

# 使用Flowable

```java
Flowable<Integer> upstream = Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "emit 1");
                emitter.onNext(1);
                Log.d(TAG, "emit 2");
                emitter.onNext(2);
                Log.d(TAG, "emit 3");
                emitter.onNext(3);
                Log.d(TAG, "emit complete");
                emitter.onComplete();
            }
        }, BackpressureStrategy.ERROR); //增加了一个参数

        Subscriber<Integer> downstream = new Subscriber<Integer>() {

            @Override
            public void onSubscribe(Subscription s) {
                Log.d(TAG, "onSubscribe");
                s.request(Long.MAX_VALUE);  //注意这句代码
            }

            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "onNext: " + integer);

            }

            @Override
            public void onError(Throwable t) {
                 Log.w(TAG, "onError: ", t);
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "onComplete");
            }
        };

        upstream.subscribe(downstream);
```

我们注意到这次和`Observable`有些不同. 首先是创建`Flowable`的时候增加了一个参数, 这个参数是用来选择背压,也就是出现上下游流速不均衡的时候应该怎么处理的办法, 这里我们直接用`BackpressureStrategy.ERROR`这种方式, 这种方式会在出现上下游流速不均衡的时候直接抛出一个异常,这个异常就是著名的`MissingBackpressureException`. 其余的策略后面再来讲解.

另外的一个区别是在下游的`onSubscribe`方法中传给我们的不再是`Disposable`了, 而是`Subscription`, 它俩有什么区别呢, 首先它们都是上下游中间的一个开关, 之前我们说调用`Disposable.dispose()`方法可以切断水管, 同样的调用`Subscription.cancel()`也可以切断水管, 不同的地方在于`Subscription`增加了一个`void request(long n)`方法, 这个方法有什么用呢, 在上面的代码中也有这么一句代码:

```
 s.request(Long.MAX_VALUE);
```

这是因为`Flowable`在设计的时候采用了一种新的思路也就是`响应式拉取`的方式来更好的解决上下游流速不均衡的问题, 与我们之前所讲的`控制数量`和`控制速度`不太一样, 这种方式用通俗易懂的话来说就好比是`叶问打鬼子`, 我们把`上游`看成`小日本`, 把`下游`当作`叶问`, 当调用`Subscription.request(1)`时, `叶问`就说`我要打一个!` 然后`小日本`就拿出`一个鬼子`给叶问, 让他打, 等叶问打死这个鬼子之后, 再次调用`request(10)`, 叶问就又说`我要打十个!` 然后小日本又派出`十个鬼子`给叶问, 然后就在边上看热闹, 看叶问能不能打死十个鬼子, 等叶问打死十个鬼子后再继续要鬼子接着打...

所以我们把request当做是一种能力, 当成`下游处理事件`的能力, 下游能处理几个就告诉上游我要几个, 这样只要上游根据下游的处理能力来决定发送多少事件, 就不会造成一窝蜂的发出一堆事件来, 从而导致OOM. 这也就完美的解决之前我们所学到的两种方式的缺陷, 过滤事件会导致事件丢失, 减速又可能导致性能损失. 而这种方式既解决了事件丢失的问题, 又解决了速度的问题, 完美 !

# 同步情况

```java
Observable.create(new ObservableOnSubscribe<Integer>() {                         
    @Override                                                                    
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception { 
        for (int i = 0; ; i++) {   //无限循环发事件                                              
            emitter.onNext(i);                                                   
        }                                                                        
    }                                                                            
}).subscribe(new Consumer<Integer>() {                                           
    @Override                                                                    
    public void accept(Integer integer) throws Exception {                       
        Thread.sleep(2000);                                                      
        Log.d(TAG, "" + integer);                                                
    }                                                                            
});
```

当上下游工作在`同一个线程`中时, 这时候是一个`同步`的订阅关系, 也就是说`上游`每发送一个事件`必须`等到`下游`接收处理完了以后才能接着发送下一个事件.

同步与异步的区别就在于有没有缓存发送事件的缓冲区。

# 异步情况

通过subscribeOn和observeOn来确定对应的线程，达到异步的效果，异步时会有一个对应的缓存区来换从从上游发送的事件。

```java
public enum BackpressureStrategy {
    /**
     * OnNext events are written without any buffering or dropping.
     * Downstream has to deal with any overflow.
     * <p>Useful when one applies one of the custom-parameter onBackpressureXXX operators.
     */
    MISSING,
    /**
     * Signals a MissingBackpressureException in case the downstream can't keep up.
     */
    ERROR,
    /**
     * Buffers <em>all</em> onNext values until the downstream consumes it.
     */
    BUFFER,
    /**
     * Drops the most recent onNext value if the downstream can't keep up.
     */
    DROP,
    /**
     * Keeps only the latest onNext value, overwriting any previous value if the
     * downstream can't keep up.
     */
    LATEST
}
```

背压策略：

1. error， 缓冲区大概在128
2. buffer， 缓冲区在1000左右
3. drop， 把存不下的事件丢弃
4. latest， 只保留最新的
5. missing, 缺省设置，不做任何操作

上游从哪里得知下游的处理能力呢？我们来看看上游最重要的部分，肯定就是`FlowableEmitter`了啊，我们就是通过它来发送事件的啊，来看看它的源码吧\(别紧张，它的代码灰常简单\)：

```
public interface FlowableEmitter<T> extends Emitter<T> {
    void setDisposable(Disposable s);
    void setCancellable(Cancellable c);

    /**
     * The current outstanding request amount.
     * <p>This method is thread-safe.
     * @return the current outstanding request amount
     */
    long requested();

    boolean isCancelled();
    FlowableEmitter<T> serialize();
}
```

FlowableEmitter是个接口，继承Emitter，Emitter里面就是我们的onNext\(\),onComplete\(\)和onError\(\)三个方法。我们看到FlowableEmitter中有这么一个方法：

```
long requested();
```

![](http://upload-images.jianshu.io/upload_images/1008453-0b408cf6d2360677.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

同步request.png

这张图的意思就是当上下游在同一个线程中的时候，在`下游`调用request\(n\)就会直接改变`上游`中的requested的值，多次调用便会叠加这个值，而上游每发送一个事件之后便会去减少这个值，当这个值减少至0的时候，继续发送事件便会抛异常了。

![](http://upload-images.jianshu.io/upload_images/1008453-6ae614d9cb3927cb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

异步request.png

可以看到，当上下游工作在不同的线程里时，每一个线程里都有一个requested，而我们调用request（1000）时，实际上改变的是下游主线程中的requested，而上游中的requested的值是由RxJava内部调用request\(n\)去设置的，这个调用会在合适的时候自动触发。

