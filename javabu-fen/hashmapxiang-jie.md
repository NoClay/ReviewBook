# HashMap的继承关系

> HashMap&lt;K, V&gt; extends AbstractMap&lt;K, V&gt; implements Map&lt;K, V&gt;, Cloneable, serializable
>
>  AbstractMap&lt;K, V&gt; implements Map&lt;K, V&gt;

HashMap是基于哈希表的Map接口的一种实现，相对应的还有TreeMap,LinkedHashMap等实现方式，此实现提供了所有可选的映射操作，并且允许Key，和value为null值，此类不能保证映射的顺序，即某一键值对序列在顺如不同的情况下添加到Map集合中，可能出现不同的存储顺序，同时HashMap不会保证该顺序恒久不变。

# HashMap内部的常量

```java
 /**
     * The default initial capacity - MUST be a power of two.
     * 默认的初始容量为4，必须是2的次方
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
     * An empty table instance to share when the table is not inflated.
     * 当table没有加载完毕的时候用来share的一个空的table
     */
    static final HashMapEntry<?,?>[] EMPTY_TABLE = {};

    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     * 用来存储的table，当需要重新调整大小的时候自动调整，长度必须是2的次方
     */
    transient HashMapEntry<K,V>[] table = (HashMapEntry<K,V>[]) EMPTY_TABLE;

    /**
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    /**
     * The next size value at which to resize (capacity * load factor).
     * @serial
     */
    // If table == EMPTY_TABLE then this is the initial capacity at which the
    // table will be created when inflated.
    int threshold;

    /**
     * The load factor for the hash table.
     *
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
     */
    transient int modCount;
```



