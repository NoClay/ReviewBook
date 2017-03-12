## 线程有几种实现方法，都是什么？同步有几种实现方法，都是什么？

继承自Thread或者实现Runnable接口，实现Callable接口，重写call函数。

Callable是类似于Runnable的接口，实现Callable接口的类和实现Runnable的类都是可被其它线程执行的任务。 Callable和Runnable有几点不同:

1. Callable规定的方法是call\(\)，而Runnable规定的方法是run\(\).

2. Callable的任务执行后可返回值，而Runnable的任务是不能返回值的

3. call\(\)方法可抛出异常，而run\(\)方法是不能抛出异常的。

4. 运行Callable任务可拿到一个Future对象，Future表示异步计算的结果。它提供了检查计算是否完成的方法,以等待计算的完成,并检索计算的结果.通过Future对象可了解任务执行情况,可取消任务的执行,还可获取任务执行的结果

```
class TaskWithResult implements Callable<T> {  
    @Override  
    public T call() throws Exception {  
        return new T();  
    }  
  	public static void main(String[] args) throws 		 InterruptedException,ExecutionException {  
        ExecutorService exec = Executors.newCachedThreadPool();  
        Future<T> result = exec.submit(new TaskWithResult());   
      	//Future 相当于是用来存放Executor执行的结果的一种容器,可以利用result.isDone()判断任务完成否
        TaskWithResult data = result.get();
        exec.shutdown();  
    }  
} 
```

**同步方法：**

1. wait\(\)：使一个线程处于等待状态，并释放所持有的对象的lock

2. sleep\(\)：使一个正在运行的线程处于睡眠状态，是一个静态方法，调用此方法要捕获InterruptedException异常

3. notify\(\)：唤醒一个处于等待状态的线程，这里的唤醒由jvm决定调度，且与优先级无关。

4. Allnotify\(\)：唤醒所有处于等待状态的线程。

5. synchronized：利用对象的监视器属性加同步锁。



