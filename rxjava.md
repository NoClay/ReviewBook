# RxJava

为什么要使用RxJava，因为RxJava能够简化逻辑，虽然代码量可能变多，但带来的是更好的逻辑体现。

# RxJava的异步实现

它的实现方式是通过一种扩展的观察者模式来实现的。

![](http://upload-images.jianshu.io/upload_images/1008453-7133ff9a13dd9a59.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

上面一根水管为事件产生的水管，叫它`上游`吧，下面一根水管为事件接收的水管叫它`下游`吧。

两根水管通过一定的方式连接起来，使得上游每产生一个事件，下游就能收到该事件。注意这里和官网的事件图是反过来的, 这里的事件发送的顺序是先1,后2,后3这样的顺序, 事件接收的顺序也是先1,后2,后3的顺序, 我觉得这样更符合我们普通人的思维, 简单明了.

这里的`上游`和`下游`就分别对应着RxJava中的`Observable`和`Observer`，它们之间的连接就对应着`subscribe()`

**注意: 只有当上游和下游建立连接之后, 上游才会开始发送事件. 也就是调用了**`subscribe()`**方法之后才开始发送事件.**

举个例子：

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        }).subscribeOn(Schedulers.newThread())                                              
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "subscribe");
            }

            @Override
            public void onNext(Integer value) {
                Log.d(TAG, "" + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "error");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "complete");
            }
        });
```

1. Observable 被观察者
2. Observer 观察者
3. ObservableEmitter 被观察者发射器，可以用来发出事件的，它可以发出三种类型的事件，通过调用emitter的`onNext(T value)`、`onComplete()`和`onError(Throwable error)`就可以分别发出next事件、complete事件和error事件。但是，请注意，并不意味着你可以随意乱七八糟发射事件，需要满足一定的规则：
   * 上游可以发送无限个onNext, 下游也可以接收无限个onNext.
   * 当上游发送了一个onComplete后, 上游onComplete之后的事件将会`继续`
     发送, 而下游收到onComplete事件之后将`不再继续`接收事件.
   * 当上游发送了一个onError后, 上游onError之后的事件将`继续`发送, 而下游收到onError事件之后将`不再继续`接收事件.
   * 上游可以不发送onComplete或onError.
   * 最为关键的是onComplete和onError必须唯一并且互斥, 即不能发多个onComplete, 也不能发多个onError, 也不能先发一个onComplete, 然后再发一个onError, 反之亦然
4. Disposeable 可丢弃的，作为下游控制的一个开关
5. 不带任何参数的`subscribe()`表示下游不关心任何事件,你上游尽管发你的数据去吧, 老子可不管你发什么
   带有一个`Consumer`参数的方法表示下游只关心onNext事件, 其他的事件我假装没看见,



