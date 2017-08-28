# 背压问题

所谓的`Backpressure`其实就是为了控制流量, 水缸存储的能力毕竟有限, 因此我们还得从`源头`去解决问题, 既然你发那么快, 数据量那么大, 那我就想办法不让你发那么快呗.

# 根源

产生背压问题的根源就是上游发送速度与下游的处理速度不均导致的，所以如果想要解决这个问题就需要通过匹配两个速率达到解决这个背压根源的措施。

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



