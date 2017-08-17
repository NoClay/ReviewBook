# ListView

直接继承自AbsListView，AbsListView继承自AdapterView，AdapterView又继承自ViewGroup。

Adpater在ListView和数据源之间起到了一个桥梁的作用

## RecycleBin机制

RecycleBin机制是ListView能够实现成百上千条数据都不会OOM最重要的一个原因。RecycleBin是AbsListView的一个内部类。

* RecycleBin当中使用mActiveViews这个数组来存储View，调用这个方法后就会根据传入的参数来将ListView中的指定元素存储到mActiveViews中。
* mActiveViews当中所存储的View，一旦被获取了之后就会从mActiveViews当中移除，下次获取同样位置的时候将会返回null，所以mActiveViews不能被重复利用。
* addScrapView\(\)用于将一个废弃的View进行缓存，该方法接收一个View参数，当有某个View确定要废弃掉的时候（比如滚动出了屏幕）就应该调用这个方法来对View进行缓存，RecycleBin当中使用mScrapV
* iews和mCurrentScrap这两个List来存储废弃View。
* getScrapView 用于从废弃缓存中取出一个View，这些废弃缓存中的View是没有顺序可言的，因此getScrapView\(\)方法中的算法也非常简单，就是直接从mCurrentScrap当中获取尾部的一个scrap view进行返回。
* 我们都知道Adapter当中可以重写一个getViewTypeCount\(\)来表示ListView中有几种类型的数据项，而setViewTypeCount\(\)方法的作用就是为每种类型的数据项都单独启用一个RecycleBin缓存机制。

View的流程分三步，onMeasure\(\)用于测量View的大小，onLayout\(\)用于确定View的布局，onDraw\(\)用于将View绘制到界面上。

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

### ListView的优化方式

1. 重用convertView：防止重复创建ItemView

2. 使用ViewHolder：findViewById太耗时，通过convertView.setTag操作，可以直接取出而不用重复findViewById

3. 使用static class ViewHolder：如果声明成员类不要求访问外围实例，就要始终把static修饰符放在它的声明中，使它成为静态成员类，而不是非静态成员类。因为非静态成员类的实例会包含一个额外的指向外围对象的引用，保存这份引用要消耗时间和空间，并且导致外围类实例符合垃圾回收时仍然被保留。如果没有外围实例的情况下，也需要分配实例，就不能使用非静态成员类，因为非静态成员类的实例必须要有一个外围实例。

4. 在列表里面有图片的情况下，监听滑动不加载图片

5. 多个不同的布局，可以创建不同的ViewHolder和convertView进行重用



