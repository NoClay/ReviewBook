# 枚举根节点

**可达性分析对于时间的敏感性：**可达性分析的过程必须在一个确保一致性的快照中进行--这里“一致性”的意思是指在整个分析的过程中执行系统看起来像是被冻结在某个时间点，不可以出现分析过程中对象的引用关系还在不断变化的情况，这样子分析结果的准确性就无法得到保证。这点是导致GC进行时必须停顿所有执行线程（Stop the world）的一个重要原因，即使是在号称（几乎）不会发生停顿的CMS收集器中，梅菊根结点也是必须要停顿的。

在HotSpot虚拟机中，使用了一组成为OopMap的数据结构来达到这个目的的，在类加载完成的时候，HotSpo就可以将对象内什么偏移量上是什么类型的数据计算出来，在JIT编译过程中，也会在特定的位置记录下战和寄存器中哪些位置是引用。

# 安全点

HotSpot没有为每条指令都生成OopMap，因为这需要大量的内存空间，所以只是在“特定的位置”记录了这些信息，这些位置称为安全点（SafePoint），即程序执行时并非在所有地方都能停顿下来开始GC，只有在到达安全点的时候才能进行暂停。

安全点的选定基本上是以程序“是否具有让程序长时间执行的特征”为标准进行选定的--因为每条指令执行的时间都非常短暂，程序不太可能因为指令流长度太长这个原因而长时间运行，而“长时间执行”最明显的特征就是指令序列服用，例如方法调用、循环跳转、一场跳转等，所以具有这些功能的指令才会产生Safepoint。

## 如何中断

1. **抢先式中断：**不需要线程的执行代码主动的配合，在GC执行的时候，首先把所有线程全部中断，如果发现有线程中断的地方不再安全点上，就恢复线程，让它“跑”到安全点上。
2. **主动式中断：**当GC需要进行中断线程的时候，不直接对线程操作，仅仅简单的设置一个标志，所个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起，轮询标志的地方和安全点是重合的，另外再加上创建对象的时候需要分配内存的地方。

**几乎没有虚拟机采用抢先式中断！**

# 安全区域

使用安全点的方法保证了程序执行时，在不太长的时间内就会遇到可以进入GC的安全点，但是如果程序不执行的时候，比如所谓的程序不执行就是没有分配到CPU时间，即处于Sleep状态或者Blocked状态，这时候线程无法响应JVM的中断请求，“走“到安全的地方去中断挂起，JVM显然不太可能等待线程重新被分配CPU时间。

安全区域是指在一段代码片段中，引用关系不会发生改变，这个区域中的任何位置开始进行GC都是安全的，我们可以把安全区域看作是拓展了的安全点。

当线程执行到安全区域的时候，首先标识自己进入了安全区域，那样，当这段时间里JVM要发起GC的时候，就不需要管标识自己为安全区域的线程，在线程要离开安全区域时，它要检查系统是否完成了根节点枚举（或者是整个GC过程），如果要完成了，线程就继续执行，否则就必须等待直到回收过程完成并可以安全离开安全区域的信号为止。

# 垃圾收集器

![](http://img.my.csdn.net/uploads/201210/03/1349278110_8410.jpg)

上图展示了7种作用域不同分代的收集器，如果两个收集器之间存在连线，就说明他们可以搭配使用，虚拟机所处的区域，则表示它是属于新生代收集器还是老年代收集器。

## 1. Serial\(串行GC\)收集器

Serial收集器是一个新生代收集器，单线程执行，使用复制算法。它在进行垃圾收集时，必须暂停其他所有的工作线程\(用户线程\)。是Jvm client模式下默认的新生代收集器。对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。

## 2. ParNew\(并行GC\)收集器

ParNew收集器其实就是serial收集器的多线程版本，除了使用多条线程进行垃圾收集之外，其余行为与Serial收集器一样。在单CPU工作环境内绝对不会有比Serial的收集器有更好的效果，随着可以使用的CPU的数量的增加，它对于GC时系统资源的有效利用还是很有好处的，它默认开启的收集线程数与CPU的数量下同，在CPU非常多的环境下，可以使用-XX：ParallelGCThreads参数来限制垃圾收集的线程数。

## 3. Parallel Scavenge\(并行回收GC\)收集器

Parallel Scavenge收集器也是一个新生代收集器，它也是使用复制算法的收集器，又是并行多线程收集器。parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而parallel Scavenge收集器的目标则是达到一个可控制的吞吐量。吞吐量= 程序运行时间/\(程序运行时间 + 垃圾收集时间\)，虚拟机总共运行了100分钟。其中垃圾收集花掉1分钟，那吞吐量就是99%。Parallel Scavenge提供了两个参数用于精确控制吞吐量，分别是**控制最大垃圾收集停顿时间的-XX：MaxGCPauseMillis参数和直接设置吞吐量大小的-XXGCTimeRatio参数。**

这个收集器还有一个开关：**-XX：+UseAdaptiveSizePolicy**值得关注。这个开关打开后，虚拟机会根据当前系统的运行情况收集性能监控信息自动调整新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX：SurvivorRation）、晋升老年代对象大小（-XX：PretenureSizeThreshold）等细节参数。

使用自适应策略，只需要设置最大堆（-Xmx）,利用最大停顿时间或者吞吐量给虚拟机设置一个优化目标。

## 4. Serial Old\(串行GC\)收集器

Serial Old是Serial收集器的老年代版本，它同样使用一个单线程执行收集，使用“标记-整理”算法。主要使用在Client模式下的虚拟机。对于Server模式下有两个用途：**1. 在JDK1.5以及之前的版本中与Parallel Scavenge收集器搭配使用；2. 作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用。**

## 5. Parallel Old\(并行GC\)收集器

Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。

## 6. CMS\(并发GC\)收集器

CMS\(Concurrent Mark Sweep\)收集器是一种以获取最短回收停顿时间为目标的收集器。CMS收集器是基于“标记-清除”算法实现的，整个收集过程大致分为4个步骤：

①.初始标记\(CMS initial mark\)

②.并发标记\(CMS concurrenr mark\)

③.重新标记\(CMS remark\)

④.并发清除\(CMS concurrent sweep\)

其中初始标记、重新标记这两个步骤任然需要停顿其他用户线程。初始标记仅仅只是标记出GC ROOTS能直接关联到的对象，速度很快，并发标记阶段是进行GC ROOTS 根搜索算法阶段，会判定对象是否存活。而重新标记阶段则是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间会被初始标记阶段稍长，但比并发标记阶段要短。  
     由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以整体来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。  
**CMS收集器的优点：**并发收集、低停顿

**CMS收集器主要有三个显著缺点：**  
**1. CMS收集器对CPU资源非常敏感**。在并发阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致引用程序变慢，总吞吐量下降。CMS默认启动的回收线程数是：\(CPU数量+3\) / 4。虚拟机提供了一种称为“增量式并发收集器”的CMS收集器变种，可以在并发标记、清理的时候让GC线程、用户线程交替运行，尽量减少GC线程的独占资源的时间。  
**2. CMS收集器无法处理浮动垃圾**，可能出现“Concurrent Mode Failure“，失败后而导致另一次Full  GC的产生。由于CMS并发清理阶段用户线程还在运行，伴随程序的运行自热会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在本次收集中处理它们，只好留待下一次GC时将其清理掉。这一部分垃圾称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行，即需要预留足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分内存空间提供并发收集时的程序运作使用。在默认设置下，CMS收集器在老年代使用了68%的空间时就会被激活，也可以通过参数-XX:CMSInitiatingOccupancyFraction的值来提供触发百分比，以降低内存回收次数提高性能。要是CMS运行期间预留的内存无法满足程序其他线程需要，就会出现“Concurrent Mode Failure”失败，这时候虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。所以说参数-XX:CMSInitiatingOccupancyFraction设置的过高将会很容易致“Concurrent Mode Failure”失败，性能反而降低。  
**3. 碎片化，**最后一个缺点，CMS是基于“标记-清除”算法实现的收集器，使用“标记-清除”算法收集后，会产生大量碎片。空间碎片太多时，将会给对象分配带来很多麻烦，比如说大对象，内存空间找不到连续的空间来分配不得不提前触发一次Full  GC。为了解决这个问题，CMS收集器提供了一个-XX:UseCMSCompactAtFullCollection开关参数，用于在Full  GC之后增加一个碎片整理过程，还可通过-XX:CMSFullGCBeforeCompaction参数设置执行多少次不压缩的Full  GC之后，跟着来一次碎片整理过程。

## 7. G1收集器

G1\(Garbage First\)收集器是JDK1.7提供的一个新收集器，G1收集器基于“标记-整理”算法实现，也就是说不会产生内存碎片。还有一个特点之前的收集器进行收集的范围都是整个新生代或老年代，而G1将整个Java堆\(包括新生代，老年代\)。

**G1收集器的特点：**

1. **并行与并发：**G1利用多CPU、多核环境下的硬件优势，缩小stop-the-world的时间。
2. **分代收集：**G1不需要其他收集器配合就可以独立管理整个GC堆，但它能够采用不同的方式来处理。
3. **空间整合：**整体上是“标记-整理”，局部上是基于“复制”的算法来实现的
4. **可预测的停顿：**降低停顿时间，G1建立了可预测的停顿时间模型，能让使用者明确的指定在一个长度M毫秒内的时间片段，消耗在垃圾收集的时间不得超过N毫秒，这已经适实时java（RTSJ）的垃圾收集器的特征了

**G1收集的步骤：**

1. 初始标记
2. 并发标记
3. 最终标记
4. 筛选回收

# 理解GC日志

GC日志格式：

GC发生的时间 + GC的类型（GC/Full GC） + GC发生的区域（收集器决定的名称） + GC前java堆已使用容量 -&gt; GC后java堆已使用容量（java堆总容量） +　GC所占用的时间（秒）如：\[Times : user = 0.01 sys = 0.00 real = 0.02 secs\] 分别表示用户态消耗的时间，内核态消耗的时间和操作从开始到结束所经过的墙钟时间。

## 垃圾收集器参数总结

| 参数 | 描述 |
| :--- | :--- |
| -XX:+UseSerialGC | Jvm运行在Client模式下的默认值，打开此开关后，使用Serial + Serial Old的收集器组合进行内存回收 |
| -XX:+UseParNewGC | 打开此开关后，使用ParNew + Serial Old的收集器进行垃圾回收 |
| -XX:+UseConcMarkSweepGC | 使用ParNew + CMS +  Serial Old的收集器组合进行内存回收，Serial Old作为CMS出现“Concurrent Mode Failure”失败后的后备收集器使用。 |
| -XX:+UseParallelGC | Jvm运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge +  Serial Old的收集器组合进行回收 |
| -XX:+UseParallelOldGC | 使用Parallel Scavenge +  Parallel Old的收集器组合进行回收 |
| -XX:SurvivorRatio | 新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden:Subrvivor = 8:1 |
| -XX:PretenureSizeThreshold | 直接晋升到老年代对象的大小，设置这个参数后，大于这个参数的对象将直接在老年代分配 |
| -XX:MaxTenuringThreshold | 晋升到老年代的对象年龄，每次Minor GC之后，年龄就加1，当超过这个参数的值时进入老年代 |
| -XX:UseAdaptiveSizePolicy | 动态调整java堆中各个区域的大小以及进入老年代的年龄 |
| -XX:+HandlePromotionFailure | 是否允许新生代收集担保，进行一次minor gc后, 另一块Survivor空间不足时，将直接会在老年代中保留 |
| -XX:ParallelGCThreads | 设置并行GC进行内存回收的线程数 |
| -XX:GCTimeRatio | GC时间占总时间的比列，默认值为99，即允许1%的GC时间，仅在使用Parallel Scavenge 收集器时有效 |
| -XX:MaxGCPauseMillis | 设置GC的最大停顿时间，在Parallel Scavenge 收集器下有效 |
| -XX:CMSInitiatingOccupancyFraction | 设置CMS收集器在老年代空间被使用多少后出发垃圾收集，默认值为68%，仅在CMS收集器时有效，-XX:CMSInitiatingOccupancyFraction=70 |
| -XX:+UseCMSCompactAtFullCollection | 由于CMS收集器会产生碎片，此参数设置在垃圾收集器后是否需要一次内存碎片整理过程，仅在CMS收集器时有效 |
| -XX:+CMSFullGCBeforeCompaction | 设置CMS收集器在进行若干次垃圾收集后再进行一次内存碎片整理过程，通常与UseCMSCompactAtFullCollection参数一起使用 |
| -XX:+UseFastAccessorMethods | 原始类型优化 |
| -XX:+DisableExplicitGC | 是否关闭手动System.gc |
| -XX:+CMSParallelRemarkEnabled | 降低标记停顿 |
| -XX:LargePageSizeInBytes | 内存页的大小不可设置过大，会影响Perm的大小，-XX:LargePageSizeInBytes=128m |

Client、Server模式默认GC

| 新生代GC方式 | 老年代和持久 | **代** | GC方式 |
| :--- | :--- | :--- | :--- |
|  | Client | Serial 串行GC | Serial Old 串行GC |
|  | Server | Parallel Scavenge  并行回收GC | Parallel Old 并行GC |

Sun/[Oracle](http://lib.csdn.net/base/oracle)JDK GC组合方式

| 新生代GC方式 | 老年代和持久 | **代** | GC方式 |
| :--- | :--- | :--- | :--- |
|  | -XX:+UseSerialGC | Serial 串行GC | Serial Old 串行GC |
|  | -XX:+UseParallelGC | Parallel Scavenge  并行回收GC | Serial Old  并行GC |
|  | -XX:+UseConcMarkSweepGC | ParNew 并行GC | CMS 并发GC  当出现“Concurrent Mode Failure”时 采用Serial Old 串行GC |
|  | -XX:+UseParNewGC | ParNew 并行GC | Serial Old 串行GC |
|  | -XX:+UseParallelOldGC | Parallel Scavenge  并行回收GC | Parallel Old 并行GC |
|  | -XX:+UseConcMarkSweepGC -XX:+UseParNewGC | Serial 串行GC | CMS 并发GC  当出现“Concurrent Mode Failure”时 采用Serial Old 串行GC |



