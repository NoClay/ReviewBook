Dispatcher的作用是维护请求的状态，并维护一个线程池，用于执行请求。

### 遗留的疑问
1. 线程池的缓存队列new SynchronousQueue<Runnable>()是怎么缓存的？
2. Collections.unmodifiableList(result);的作用

```java
public final class Dispatcher {
  //最大并发请求数
  private int maxRequests = 64;
  //每个主机最大请求数
  private int maxRequestsPerHost = 5;
  //用于在空闲时执行的回调接口
  private Runnable idleCallback;

  /** Executes calls. Created lazily. */
  //执行网络请求的线程池
  private ExecutorService executorService;

  /** Ready async calls in the order they'll be run. */
  //就绪状态的异步请求队列
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  //运行中的异步请求队列
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  //运行中的同步请求队列
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
  //可以使用自定义的线程池来创建Dispatcher对象
  public Dispatcher(ExecutorService executorService) {
    this.executorService = executorService;
  }

  //默认的构造方法
  public Dispatcher() {
  }

  //初始化线程池，这是一个同步的懒加载线程池的方法
  public synchronized ExecutorService executorService() {
    //懒加载
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(
        0,//核心线程数，0表示如果执行请求的线程，使用完了并且过期了(keepAliveTime)就被回收了
       Integer.MAX_VALUE,//最大线程数
       60,//keepAliveTime
       TimeUnit.SECONDS,//keepAliveTime的单位
       new SynchronousQueue<Runnable>(),//缓存队列
       Util.threadFactory("OkHttp Dispatcher", false));//创建线程的工厂
    }
    return executorService;
  }

  /**
   * Set the maximum number of requests to execute concurrently. Above this requests queue in
   * memory, waiting for the running calls to complete.
   *
   * <p>If more than {@code maxRequests} requests are in flight when this is invoked, those requests
   * will remain in flight.
   */
  //重新设置最大请求数，设置完成后调整执行队列
  public synchronized void setMaxRequests(int maxRequests) {
    if (maxRequests < 1) {
      throw new IllegalArgumentException("max < 1: " + maxRequests);
    }
    this.maxRequests = maxRequests;
    promoteCalls();
  }

  //获得当前的最大请求数
  public synchronized int getMaxRequests() {
    return maxRequests;
  }

  /**
   * Set the maximum number of requests for each host to execute concurrently. This limits requests
   * by the URL's host name. Note that concurrent requests to a single IP address may still exceed
   * this limit: multiple hostnames may share an IP address or be routed through the same HTTP
   * proxy.
   *
   * <p>If more than {@code maxRequestsPerHost} requests are in flight when this is invoked, those
   * requests will remain in flight.
   */
  //重新设置每个主机的最大请求数，设置完成后调整执行队列
  public synchronized void setMaxRequestsPerHost(int maxRequestsPerHost) {
    if (maxRequestsPerHost < 1) {
      throw new IllegalArgumentException("max < 1: " + maxRequestsPerHost);
    }
    this.maxRequestsPerHost = maxRequestsPerHost;
    promoteCalls();
  }


  public synchronized int getMaxRequestsPerHost() {
    return maxRequestsPerHost;
  }

  /**
   * Set a callback to be invoked each time the dispatcher becomes idle (when the number of running
   * calls returns to zero).
   *
   * <p>Note: The time at which a {@linkplain Call call} is considered idle is different depending
   * on whether it was run {@linkplain Call#enqueue(Callback) asynchronously} or
   * {@linkplain Call#execute() synchronously}. Asynchronous calls become idle after the
   * {@link Callback#onResponse onResponse} or {@link Callback#onFailure onFailure} callback has
   * returned. Synchronous calls become idle once {@link Call#execute() execute()} returns. This
   * means that if you are doing synchronous calls the network layer will not truly be idle until
   * every returned {@link Response} has been closed.
   */
  public synchronized void setIdleCallback(Runnable idleCallback) {
    this.idleCallback = idleCallback;
  }

  //执行异步请求
  synchronized void enqueue(AsyncCall call) {
    //如果运行请求队列的大小小于最大请求数并且在某个主机上运行的请求数小于每个主机的最大请求数，则将该请求加入到运行队列中，立即执行，否则将其加入到准备队列中。
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
  }

  /**
   * Cancel all calls currently enqueued or executing. Includes calls executed both {@linkplain
   * Call#execute() synchronously} and {@linkplain Call#enqueue asynchronously}.
   */
  //取消所有当前正在执行的任务
  public synchronized void cancelAll() {
    for (AsyncCall call : readyAsyncCalls) {
      call.get().cancel();
    }

    for (AsyncCall call : runningAsyncCalls) {
      call.get().cancel();
    }

    for (RealCall call : runningSyncCalls) {
      call.cancel();
    }
  }

  //调整执行队列
  private void promoteCalls() {
    //如果当前运行队列的大小，大于最大请求数，不进行调整，本次执行完后，下次不允许执行队列容量溢出
    if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.
    //如果准备队列为空，也不进行调整
    if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.

    //从准备队列中依次取出请求对象Call，加入到运行队列中直到运行队列满为止
    for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
      AsyncCall call = i.next();

      if (runningCallsForHost(call) < maxRequestsPerHost) {
        i.remove();
        runningAsyncCalls.add(call);
        executorService().execute(call);
      }

      if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.
    }
  }

  /** Returns the number of running calls that share a host with {@code call}. */
  //返回与参数call运行在同一主机的网络请求的数量
  private int runningCallsForHost(AsyncCall call) {
    int result = 0;
    for (AsyncCall c : runningAsyncCalls) {
      if (c.host().equals(call.host())) result++;
    }
    return result;
  }

  /** Used by {@code Call#execute} to signal it is in-flight. */
  //进行同步请求要调用的方法，直接将请求加入到同步请求队列中
  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }

  /** Used by {@code AsyncCall#run} to signal completion. */
  //取消异步请求
  void finished(AsyncCall call) {
    finished(runningAsyncCalls, call, true);
  }

  /** Used by {@code Call#execute} to signal completion. */
  //取消同步请求
  void finished(RealCall call) {
    finished(runningSyncCalls, call, false);
  }

  private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
    int runningCallsCount;
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      //如果取消的是异步请求，则需要调整异步执行队列
      if (promoteCalls) promoteCalls();
      runningCallsCount = runningCallsCount();
      idleCallback = this.idleCallback;
    }

    //如果当前执行请求数为0，并且存在idleCallback回调时，就执行该回调
    if (runningCallsCount == 0 && idleCallback != null) {
      idleCallback.run();
    }
  }

  /** Returns a snapshot of the calls currently awaiting execution. */
  //返回异步准备队列的快照
  public synchronized List<Call> queuedCalls() {
    List<Call> result = new ArrayList<>();
    for (AsyncCall asyncCall : readyAsyncCalls) {
      result.add(asyncCall.get());
    }
    return Collections.unmodifiableList(result);
  }

  /** Returns a snapshot of the calls currently being executed. */
  //返回所有正在执行的请求的集合快照
  public synchronized List<Call> runningCalls() {
    List<Call> result = new ArrayList<>();
    result.addAll(runningSyncCalls);
    for (AsyncCall asyncCall : runningAsyncCalls) {
      result.add(asyncCall.get());
    }
    return Collections.unmodifiableList(result);
  }

  //准备队列请求数
  public synchronized int queuedCallsCount() {
    return readyAsyncCalls.size();
  }
  //当前正在运行的请求数
  public synchronized int runningCallsCount() {
    return runningAsyncCalls.size() + runningSyncCalls.size();
  }
}

```
