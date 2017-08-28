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
   带有一个`Consumer`参数的方法表示下游只关心onNext事件, 其他的事件我假装没看见
6. `subscribeOn()` 指定的是上游发送事件的线程, `observeOn()` 指定的是下游接收事件的线程.

   **多次指定上游的线程只有第一次指定的有效, 也就是说多次调用**`subscribeOn()`** 只有第一次的有效, 其余的会被忽略.**

   **多次指定下游的线程是可以的, 也就是说每调用一次**`observeOn()`** , 下游的线程就会切换一次.**

7. 在RxJava中, 已经内置了很多线程选项供我们选择, 例如有

   * Schedulers.io\(\) 代表io操作的线程, 通常用于网络,读写文件等io密集型的操作
   * Schedulers.computation\(\) 代表CPU计算密集型的操作, 例如需要大量计算的操作
   * Schedulers.newThread\(\) 代表一个常规的新线程
   * AndroidSchedulers.mainThread\(\) 代表Android的主线程

8. 那如果有**多个**`Disposable`**该怎么办呢**, RxJava中已经内置了一个容器`CompositeDisposable`, 每当我们得到一个`Disposable`时就调用`CompositeDisposable.add()`将它添加到容器中, 在退出的时候, 调用`CompositeDisposable.clear()` 即可切断所有的水管.

# 操作符

## from

接收一个集合作为输入，并且每次输出一个元素给观察者

## map

map是RxJava中最简单的一个变换操作符了, 它的作用就是对上游发送的每一个事件应用一个函数, 使得每一个事件都按照指定的函数去变化. 用事件图表示如下:

![](http://upload-images.jianshu.io/upload_images/1008453-2a068dc6b726568a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

图中map中的函数作用是将圆形事件转换为矩形事件, 从而导致下游接收到的事件就变为了矩形。

## flatMap

flatMap是一个非常强大的操作符, 先用一个比较难懂的概念说明一下:

`FlatMap`将一个发送事件的上游Observable变换为多个发送事件的Observables，然后将它们发射的事件合并后放进一个单独的Observable里.

这句话比较难以理解, 我们先通俗易懂的图片来详细的讲解一下, 首先先来看看整体的一个图片:

![](http://upload-images.jianshu.io/upload_images/1008453-659c8c548805fdcd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

flatmap.png

先看看上游, 上游发送了三个事件, 分别是1,2,3, 注意它们的颜色.

中间flatMap的作用是将圆形的事件转换为一个发送矩形事件和三角形事件的新的上游Observable.

还是不能理解? 别急, 再来看看分解动作:

![](http://upload-images.jianshu.io/upload_images/1008453-2ccce5cf25e8023a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

flatmap1.png

这样就很好理解了吧 !!!

上游每发送一个事件, flatMap都将创建一个新的水管, 然后发送转换之后的新的事件, 下游接收到的就是这些新的水管发送的数据. **这里需要注意的是, flatMap并不保证事件的顺序,** 也就是图中所看到的, 并不是事件1就在事件2的前面. 如果需要保证顺序则需要使用`concatMap`.

说了原理, 我们还是来看看实际中的代码如何写吧:

```
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(Integer integer) throws Exception {
                final List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                return Observable.fromIterable(list).delay(10,TimeUnit.MILLISECONDS);
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d(TAG, s);
            }
        });
```

## concatMap

和flatMap基本相同，不同的是严格按照上游发送的顺序来发送的。

## filter

filter\(\)输出和输入相同的元素，并且会过滤掉那些不满足检查条件的。

## take

take\(\)输出最多指定数量的结果。

## doOnNext

doOnNext\(\)允许我们在每次输出一个元素之前做一些额外的事情，比如这里的保存标题。

## zip

`Zip`通过一个函数将多个Observable发送的事件结合到一起，然后发送这些组合到一起的事件. 它按照严格的顺序应用这个函数。它只发射与发射数据项最少的那个Observable一样多的数据。

我们再用通俗易懂的图片来解释一下:

![](http://upload-images.jianshu.io/upload_images/1008453-e11e9d75b1775e4e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

zip.png

从这个图中可以看见, 这次上游和以往不同的是, 我们有两根水管了.

其中一根水管负责发送`圆形事件` , 另外一根水管负责发送`三角形事件` , 通过Zip操作符, 使得`圆形事件` 和`三角形事件` 合并为了一个`矩形事件` .



