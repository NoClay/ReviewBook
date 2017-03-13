# HashMap的继承关系（ps：java的jdk中的HashMap和Android中的略有不同，此处以Android中的为例）

> HashMap&lt;K, V&gt; extends AbstractMap&lt;K, V&gt; implements Map&lt;K, V&gt;, Cloneable, serializable
>
> AbstractMap&lt;K, V&gt; implements Map&lt;K, V&gt;

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
    /** 
 * 当一个桶中存储的节点数    增加到 
 * 大于8个，则将链表转换为红黑树结构存储。 
 */  
static final int TREEIFY_THRESHOLD = 8;  
  
/** 
 * 当一个桶中的存储的节点数   减少到 
 * 小于6个，则将红黑树转换为链表结构存储 
 */  
static final int UNTREEIFY_THRESHOLD = 6;  
  
/** 
 * 最小的树化容量，即当HashMap的容量至少为64的时候 
 * 才能进行链表转化为红黑树的操作，这样做是为了解决 
 * 树化阀值和容量扩容之间的冲突，即当HashMap第一次扩容 
 * 即扩展到64，，，16----->64->128->256，如此 
 */  
static final int MIN_TREEIFY_CAPACITY = 64;  
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
        if (initialCapacity > MAXIMUM_CAPACITY) {
            initialCapacity = MAXIMUM_CAPACITY;
        } else if (initialCapacity < DEFAULT_INITIAL_CAPACITY) {
            initialCapacity = DEFAULT_INITIAL_CAPACITY;
        }

        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        // Android-Note: We always use the default load factor of 0.75f.

        // This might appear wrong but it's just awkward design. We always call
        // inflateTable() when table == EMPTY_TABLE. That method will take "threshold"
        // to mean "capacity" and then replace it with the real threshold (i.e, multiplied with
        // the load factor).
        threshold = initialCapacity;
        init();
    }
```

**注意：**

1. 这里验证了我们之前的想法，当我们制定的容量大于最大容量或者小于最小值，会等于默认值
2. 我们初始化哈希表的时候，会将阈值设置为初始容量，在创建哈希表的时候会将阈值设定为真正的值（容量 \* 装载因子）
3. 这里的table并没有初始化，也就是没有申请空间，只是确定了参数而已。

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
    }
```

putVal参数列表依次为：

1. key经过处理的hash值，具体处理过程不做简述
2. key的对象
3. value的对象
4. 如果为true, 不改变已经存在的值
5. 如果是false，表处于创建模式



