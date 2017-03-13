# HashMap的继承关系

> HashMap&lt;K, V&gt; extends AbstractMap&lt;K, V&gt; implements Map&lt;K, V&gt;, Cloneable, serializable
>
> AbstractMap&lt;K, V&gt; implements Map&lt;K, V&gt;

HashMap是基于哈希表的Map接口的一种实现，相对应的还有TreeMap,LinkedHashMap等实现方式，此实现提供了所有可选的映射操作，并且允许Key，和value为null值，此类不能保证映射的顺序，即某一键值对序列在顺如不同的情况下添加到Map集合中，可能出现不同的存储顺序，同时HashMap不会保证该顺序恒久不变。

# 简介

基于哈希表的 Map 接口的实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键。（除了非同步和允许使用 null 之外，HashMap 类与 Hashtable 大致相同。）此类不保证映射的顺序，特别是它不保证该顺序恒久不变。

此实现假定哈希函数将元素适当地分布在各桶之间，可为基本操作（get 和 put）提供稳定的性能。迭代 collection 视图所需的时间与 HashMap 实例的“容量”（桶的数量）及其大小（键-值映射关系数）成比例。所以，如果迭代性能很重要，则不要将初始容量设置得太高（或将加载因子设置得太低）。

HashMap 的实例有两个参数影响其性能：**初始容量 和加载因子**。容量 是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。

通常，**默认加载因子 \(.75\) 在时间和空间成本上寻求一种折衷**。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。

如果很多映射关系要存储在 HashMap 实例中，则相对于按需执行自动的 rehash 操作以增大表的容量来说，使用足够大的初始容量创建它将使得映射关系能更有效地存储。

**注意，此实现不是同步的。**如果多个线程同时访问一个哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须 保持外部同步。（结构上的修改是指添加或删除一个或多个映射关系的任何操作；仅改变与实例已经包含的键关联的值不是结构上的修改。）这一般通过对自然封装该映射的对象进行同步操作来完成。**如果不存在这样的对象，则应该使用 **[`Collections.synchronizedMap`](../../java/util/Collections.html#synchronizedMap%28java.util.Map%29)** 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的非同步访问，如下所示：   Map m = Collections.synchronizedMap\(new HashMap\(...\)\);**

**由所有此类的“collection 视图方法”所返回的迭代器都是快速失败 的**：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出 [`ConcurrentModificationException`](../../java/util/ConcurrentModificationException.html)。因此，面对并发的修改，迭代器很快就会完全失败，而不冒在将来不确定的时间发生任意不确定行为的风险。

注意，迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。

# 构造方法

| 构造方法 | 方法作用 |
| :--- | :--- |
| HashMap\(\) | 构造具有默认容量（16）和默认装载因子0.75的空HashMap |
| HashMap\(int initialCapacity\) | 构造具有指定容量和默认装载因子 |
| HashMap\(int initialCapacity, float loadFactor\) | 构造具有指定容量和指定装载因子 |
| HashMap\(Map&lt;? extends K, ? extends V&gt; m\) | 构造一个映射关系和指定Map相同的 |

# 其他方法

| 返回值 | 方法名 | 作用 |
| :--- | :--- | :--- |
| clear\(\) | clear\(\) | 移除所有的映射关系 |
| Object | clone\(\) | 返回实例的浅表本身，并不复制键和值本身 |
| boolean | containsKey\(Object Key\) | 如果包含对于指定key的映射，返回true |
| boolean | containsValue\(Object value\) | 如果包含关于value的映射，返回true |
| Set&lt;K, V&gt; | entrySet\(\) | 返回键值对Set |
| Set&lt;K&gt; | keySet\(\) | 返回key的Set集合 |
| V | get\(Object key\) | 返回指定键映射的值，无则null |
| boolean | isEmpty\(\) | 实例中是否有键值对映射 |
| V | put\(K key, V value\) | 指定键值对映射 |
| void | putAll\(Map&lt;? extends K, ? extends V&gt; m\) | 复制指定键值对映射到实例 |
| V | remove\(Object key\) | 删除指定键的映射关系 |
| int | size\(\) | 返回映射关系数 |
| Collection&lt;V&gt; | values\(\) | 返回映射的值的Collection视图 |
| V | putIfAbsent\(K key, V value\) | 如果存在指定映射，不修改 |
| V | compute\(K key, BiFunction&lt;? super K, ? super V, ? extends V&gt; remppingFunction\) | 尝试计算指定键及其当前映射值的映射（如果没有当前映射则为null） |
| V | computeIfAbsent\(K key, BiFunction&lt;? super K, ? super V, ? extends V&gt; remppingFunction\) | 如果指定键尚未建立映射（或映射到null），则尝试使用给定的映射函数计算其值，然后输入到此映射中，除非为null |
| V | computeIfPresent\(K key, BiFunction&lt;? super K, ? super V, ? extends V&gt; remppingFunction\) | 如果指定键的值存在且非空，则尝试计算给定键和当前映射值的新映射 |
| void | forEach\(BiConsumer&lt;? super K, ? super V&gt; action） | 对此映射中的每个条目执行给定的操作，直到所有条目都已处理或操作抛出异常 |
| V | merge\(K key, V value, BiFunction&lt;? super V, ? super V, ? super V&gt; remppingFunction\) | 如果指定的键尚未与值关联或与空值相关联，则将其与给定的非空值相关联 |
| V | getOrDefault\(K key, V defaultValue\) | 返回指定键映射到的值，如果此映射不包含键的映射，则返回defaultValue |



