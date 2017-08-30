AIDL文件中，可以接受的数据类型为：  
1. 基本数据类型（int/ long/ char/ boolean等）  
2. String和CharSequence  
3. List：只支持ArrayList，里面每个元素都必须能够被AIDL支持，例如CopyOnWriteArrayList（支持并发读、写），在Binder中会按照List的规范访问数据，并形成一个ArrayList  
4. Map：只支持HashMap，里面的每个元素都必须被AIDL所支持，包括key和value，例如ConcurrentHashMap  
5. Parcelable：支持所艘实现了Parcelable接口的对象  
6. AIDL：所有的AIDL接口本身也可以在AIDL文件中使用

AIDL中的参数，除了基本类型，其它类型的参数都需要标上方向，如in， out或者inout。AIDL接口支持方法，但不支持静态变量  
Book.aidl

```
// IBookManager.aidl
package
 com.example.no_clay.messagertest.Data;
// Declare any non-default types here with import statements
parcelable Book;
```

BookManager.aidl

```
// IBookManager.aidl
package com.example.no_clay.messagertest.Data;
import com.example.no_clay.messagertest.Data.Book;
import com.example.no_clay.messagertest.Data.IOnNewBookArrivedListener;
// Declare any non-default types here with import statements

interface IBookManager {
     List<Book> getBookList();
     void addBook(in Book book);
     void registerListener(IOnNewBookArrivedListener listener);
     void unregisterListener(IOnNewBookArrivedListener listener);
}
```

Listener.aidl

```
// IOnNewBookArrivedListener.aidl
package com.example.no_clay.messagertest.Data;
import com.example.no_clay.messagertest.Data.Book;
// Declare any non-default types here with import statements

interface IOnNewBookArrivedListener {
    void onNewBookArrivedListener(in Book newBook);
}
```



