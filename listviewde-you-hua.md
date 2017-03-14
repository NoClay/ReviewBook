### RecyclerView和ListView的对比？

1. RecyclerView支持线性布局，网格布局，瀑布流（StaggeredGridLayoutManager）

2. 支持横向滑动和纵向滑动

3. 支持局部刷新

4. RecyclerView需要自定义OnClickListener之类

5. [需要自己添加HeaderView和FooterView：](http://blog.csdn.net/lmj623565791/article/details/51854533)

## ListView和RecycleView缓存机制

* **缓存原理**:![](http://oa5504rxk.bkt.clouddn.com/week18_listview/1.png)

* **ListView两级缓存**:![](http://oa5504rxk.bkt.clouddn.com/week18_listview/2.png)

* **RecycleView四级缓存**:![](http://oa5504rxk.bkt.clouddn.com/week18_listview/3.jpg)

* **ListView缓存流程**:

![](http://oa5504rxk.bkt.clouddn.com/week18_listview/4.jpg)

* **RecycleView缓存流程**:

![](http://oa5504rxk.bkt.clouddn.com/week18_listview/5.jpg)

