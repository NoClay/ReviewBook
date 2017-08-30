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

# HashMap内部的常量

```java
 /**
     * The default initial capacity - MUST be a power of two.
     * 默认的初始容量，必须是2的次方
     */
    static final int DEFAULT_INITIAL_CAPACITY = 4;

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * 最大的容量，如果指定的容量大于这个值，则将容量设置为最大容量，后边的构造方法有体现。
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     * 默认的装载因子，如果没有指定装载因子，则使用默认的装载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

# HashMap中几个重要的变量

```java
    /**
     * The number of key-value mappings contained in this map.
     * 用来记录HashMap中已经存在的key的个数
     */
    transient int size;

    /**
     * The next size value at which to resize (capacity * load factor).
     * 阀值，当size达到这个值的时候进行扩容， 阀值 = 容量 * 装载因子
     * @serial
     */
    // If table == EMPTY_TABLE then this is the initial capacity at which the
    // table will be created when inflated.
    int threshold;

    /**
     * The load factor for the hash table.
     * 哈希表的装载因子
     * @serial
     */
    // Android-Note: We always use a load factor of 0.75 and ignore any explicitly
    // selected values.
    final float loadFactor = DEFAULT_LOAD_FACTOR;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     * 此HashMap已被结构修改的次数
     */
    transient int modCount;
```

# HashMap的构造方法

```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
```

**注意：**

1. 这里验证了我们之前的想法，当我们制定的容量大于最大容量或者小于最小值，会等于默认值
2. 这里的阈值会那个过一个方法计算出应该的容量（2的次方）

# HashMap的put过程

两个put方法

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    @Override
    public V putIfAbsent(K key, V value) {
        return putVal(hash(key), key, value, true, true);
    }
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
                   //put
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

putVal参数列表依次为：

1. key经过处理的hash值，具体处理过程不做简述（null为0，否则将高16位和低16位异或）
2. key的对象
3. value的对象
4. 如果为true, 不改变已经存在的值
5. 如果是false，表处于创建模式

**我们可以看到：**

1. 如果table为空，或者长度为0则进行resize（创建或者将表的大小加倍\)
2. 如果对应的哈希表中的位置为空，则直接生成新的结点放入。\[对应的位置 = \(长度 - 1\) & hash\]
3. 如果对应结点的hash值（Node结点保存了hash, key, value, nextNode）等于要插入的键值对的hash， 而且结点的key等于要插入键值对的key，或者要插入的键值对key不为null且等于结点的key**\(这里是因为之前求key的hash。null为0，否则高16位和低16位异或，意味着有可能一个不为null的key和null拥有相同的hash\)**
4. 如果结点属于TreeNode，即该处已经是红黑树结构，则进入红黑树的putTreeVal过程（以结点hash为标准，插入对应的位置）
5. 遍历单链表，如果找到相同key的结点，则结束，否则插入新节点
6. 如果找到了结点，代表原来的hashMap中保存了相同的key的结点，则判断是否更新该数据，同时判断是否应该树化这个单链表
7. 判断是否扩容

# HashMap的get过程

```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

没什么好说的，直接看getNode

```java
   final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

我们可以看到：

1. 如果表为null，或者长度为0， 或者对应位置的结点为null，则直接放回null

2. 选取对应位置的结点\[对应的位置 = \(长度 - 1\) & hash\], 如果对应位置的结点的key和我们要获取的结点的key相同，则直接返回这个结点

3. 如果第一个结点不为空，则进入下一层判断

   1. 如果这个结点是树结点，则直接在红黑树中查找

   2. 遍历单链表查询结点

# 思考

我们回顾之前的put过程，如果有两个结点的key值相同，但是一个是在扩容前放入的，一个是在扩容后放入的，那么根据**对应位置 = （长度 - 1） & hash**的计算方法，两个结点对应位置是不是就不同了，那么是不是就可以放入两个相同的key值的键值对？

当然不可以，记得之前的规定吗，之前规定table的长度必须是2的次方，那么如果一个数和2 的次方减一相与，就像下边这样

> 12 & \(4 - 1\) = 2
>
> 12 & \(64 - 1\) = 50

那么这样是不是意味着会出现问题？不

resize中有一段源码显示，在我们进行扩容的时候，会将整个哈希表的元素重新映射到新的哈希表，所以是不会出现问题的。

源码:

```java
 if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
```



