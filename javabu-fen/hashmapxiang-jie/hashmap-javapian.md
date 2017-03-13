# HashMap内部的常量（Java篇）

与Android比较共同拥有的：

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

Android中HashMap中没有的：

```java
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



