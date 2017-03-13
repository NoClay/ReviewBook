# HashMap中的常量

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

# HashMap中的变量

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

---

**注意：这里的HashMapEntry是Android加的**

```java
    /**
     * An empty table instance to share when the table is not inflated.
     */
    static final HashMapEntry<?,?>[] EMPTY_TABLE = {};

    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     */
    transient HashMapEntry<K,V>[] table = (HashMapEntry<K,V>[]) EMPTY_TABLE;
```

### 1、HashMapEntry {#1hashmapentry}

看HashMapEntry的构造函数。

```
        /**
         * Creates new entry.
         */
        HashMapEntry(int h, K k, V v, HashMapEntry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
```

从中可以看出，这是一个单链表的[数据结构](http://lib.csdn.net/base/datastructure)，存有key、value、hash值以及下一个节点。

### 2、HashMap的的初始化 {#2hashmap的的初始化}

* HashMap\(\)
* HashMap\(int capacity\)
* HashMap\(int capacity,float loadFactor\)

前两个方法都是调用了第三个构造方法，我们直接看源码：

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

可以看出：

1. 和Java中的HashMap构造方法基本相同

2. 这里先将阀值的值设为初始化的容量，之后在inflateTable（）方法中，将会将阀值看作容量，而阀值也会被赋予真正的阀值。

# 那么我们是么时候会真正的需要一个哈希表呢？

当我们get的时候，如果没有哈希表可以返回空，当我们put的时候则必须有一个哈希表，才能put，所以这里一定会inflateTable。我们直接看源码：

```java
public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = sun.misc.Hashing.singleWordWangJenkinsHash(key);
        int i = indexFor(hash, table.length);
        for (HashMapEntry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
    //为了null的key专门的方法
        private V putForNullKey(V value) {
        for (HashMapEntry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
```

**可以看出：**

1. 如果表为空，则调用inflateTable，加载表。

2. 如果key为null，则直接调用putForNullKey\(value\)，即直接在table\[0\]底下的链表遍历

3. **这里的hash采用一个**[**Wang/Jenkins Hash**](https://en.wikipedia.org/wiki/Jenkins_hash_function)**的算法生成其key的哈希值（具体的算法不甚了解）**

4. 通过indexFor方法获得对应的位置，与Java中的相同（static int indexFor\(int h, int length\) { return h & \(length-1\);}）

5. 遍历对应位置的结点的单链表，如果有这个结点，则更新value，否则插入

6. recordAccess方法是个空实现（在LinkedHashMap中用于实现LRU）

我们看 添加结点的代码：

```java
    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```

**可以看出：**

1. 这里如果结点数到达阀值的时候，会进行扩容，并且对原来的哈希表进行重映射。

2. createEntry方法应该是真正的添加结点的方法

   我们看其源码，很显然，是头插法，利用头插法可以将插入效率提升，不需要判定原来是否有链表。

```java
    void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMapEntry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new HashMapEntry<>(hash, key, value, e);
        size++;
    }
```

# 很显然，Android中的HashMap是基于数组加单链表的结构的，所以这里的get方法我们不进行解析，那么为什么Android中的HashMap不采用更加良好的数组加单链表/红黑树的结构呢？

我想应该是Android中采用的Hash算法更好，Wang/Jenkins Hash算法特性就是：

1.雪崩性（更改输入参数的任何一位，就将引起输出有一半以上的位发生变化）

2.可逆性

所以Hash即使是一种信息摘要算法，它还叫做哈希，或者散列。我们平时使用的MD5,SHA1都属于Hash算法，通过输入key进行Hash计算，就可以获取key的HashCode\(\)，比如我们通过校验MD5来验证文件的完整性。对于HashCode，它是一个本地方法，实质就是地址取样运算。

**碰撞**：好的Hash算法可以出计算几乎出独一无二的HashCode，如果出现了重复的hashCode，就称作碰撞;

就算是MD5这样优秀的算法也会发生碰撞，即两个不同的key也有可能生成相同的MD5。这里的算法就应该是非常优秀的算法，以至于很少产生碰撞。

  


