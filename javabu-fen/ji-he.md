这里**写在前面：  
请多多指正。**

* 集合类概述  
  参考文献  
  [http://blog.csdn.net/liulin\_good/article/details/6213815](http://blog.csdn.net/liulin_good/article/details/6213815)

* [Java](http://lib.csdn.net/base/javase)集合类图

![](http://pic002.cnblogs.com/images/2012/80896/2012053020260298.jpg "这里写图片描述")

上述类图中，实线边框的是实现类，比如ArrayList，LinkedList，HashMap等，折线边框的是抽象类，比如AbstractCollection，AbstractList，AbstractMap等，而点线边框的是接口，比如Collection，Iterator，List等。

　　所有的集合类，都实现了Iterator接口，这是一个用于遍历集合中元素的接口，主要包含hashNext\(\),next\(\),remove\(\)三种方法。它的一个子接口LinkedIterator在它的基础上又添加了三种方法，分别是add\(\),previous\(\),hasPrevious\(\)。也就是说如果是先Iterator接口，那么在遍历集合中元素的时候，只能往后遍历，被遍历后的元素不会在遍历到，通常无序集合实现的都是这个接口，比如HashSet，HashMap；而那些元素有序的集合，实现的一般都是LinkedIterator接口，实现这个接口的集合可以双向遍历，既可以通过next\(\)访问下一个元素，又可以通过previous\(\)访问前一个元素，比如ArrayList。  
（PS：本段属于转载）  
**以下为楼主自己总结的集合类的继承关系图（Ps：集合类下方为常用方法）**  
![](http://img.blog.csdn.net/20160422214246710 "这里写图片描述")

# Collection接口

Collection接口是层次结构中的根接口，构成Collection的单位为元素。Collection接口通常不能直接使用（可以向继承自它的集合添加元素），但它提供了添加元素、删除元素、管理数据的方法，这对于继承自它的List集合和Set集合是通用的。

# List集合

两种实现方法：

1.ArrayList  
以可变数组的方式保存所有的元素，包括null，可以根据索引位置对集合进行快速的随机访问，但是进行插入对象和删除对象的时候速度较慢。  
2. LinkedList  
采用链表的结构保存对象，随机访问的时候由于链表的结构实现导致效率较低，但是在插入对象和删除对象的时候效率很高。

# Set集合

Set接口的实现方法有两种：  
1.HashSet  
由哈希表支持，并不能保证该顺序恒久不变。此类允许使用null元素  
2.TreeSet  
不仅实现了Set接口，还实现了java.util.SortedSet接口，因此TreeSet类中在遍历集合时按照自然顺序递增排序，也可以按照指定的选择器递增排序。  
![](http://img.blog.csdn.net/20160422215331839 "这里写图片描述")

Ps：存入TreeSet集合的Set集合必须实现Commparable接口，该接口中的compareTo（Object o）方法比较此对象与指定对象的顺序，如果该对象小于、等于或者大于指定对象，则分别返回负整数、0、正整数**（可以修改返回，实现递减哦）。**

# Map集合

Map接口的两种试下方法：  
1.HashMap  
基于哈希表的实现方法，此实现提供所有的可选的映射操作，并允许使用null值和null键，但必须保证键的唯一性，HashMap通过哈希表对其内部的映射关系进行快速查找，此类不影响映射的顺序，特别是它不保证该顺序恒久不变。使用此实现进行Map集合添加和删除映射关系效率更高。  
2.TreeMap  
不仅实现了Map接口，还实现了java.util.SortedMap接口，因此，集合中的映射关系有一定的顺序，但在添加、删除和定位映射关系的时候，TreeMap类较之于HashMap类性能稍差，由于TreeMap类实现的Map集合中的映射关系是按照键对象按照一定的顺序排列的，因此不允许键对象是null

可以通过HashMap创建Map集合，需要顺序输出的时候，创建一个完成相同映射关系的TreeMap类实例。

