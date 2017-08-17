# ListView

直接继承自AbsListView，AbsListView继承自AdapterView，AdapterView又继承自ViewGroup。

Adpater在ListView和数据源之间起到了一个桥梁的作用

## RecycleBin机制

RecycleBin机制是ListView能够实现成百上千条数据都不会OOM最重要的一个原因。RecycleBin是AbsListView的一个内部类。

* RecycleBin当中使用mActiveViews这个数组来存储View，调用这个方法后就会根据传入的参数来将ListView中的指定元素存储到mActiveViews中。
* mActiveViews当中所存储的View，一旦被获取了之后就会从mActiveViews当中移除，下次获取同样位置的时候将会返回null，所以mActiveViews不能被重复利用。
* addScrapView\(\)用于将一个废弃的View进行缓存，该方法接收一个View参数，当有某个View确定要废弃掉的时候（比如滚动出了屏幕）就应该调用这个方法来对View进行缓存，RecycleBin当中使用mScrapViews和mCurrentScrap这两个List来存储废弃View。

* getScrapView 用于从废弃缓存中取出一个View，这些废弃缓存中的View是没有顺序可言的，因此getScrapView\(\)方法中的算法也非常简单，就是直接从mCurrentScrap当中获取尾部的一个scrap view进行返回。
* 我们都知道Adapter当中可以重写一个getViewTypeCount\(\)来表示ListView中有几种类型的数据项，而setViewTypeCount\(\)方法的作用就是为每种类型的数据项都单独启用一个RecycleBin缓存机制。

### ListView的优化方式

1. 重用convertView：防止重复创建ItemView

2. 使用ViewHolder：findViewById太耗时，通过convertView.setTag操作，可以直接取出而不用重复findViewById

3. 使用static class ViewHolder：如果声明成员类不要求访问外围实例，就要始终把static修饰符放在它的声明中，使它成为静态成员类，而不是非静态成员类。因为非静态成员类的实例会包含一个额外的指向外围对象的引用，保存这份引用要消耗时间和空间，并且导致外围类实例符合垃圾回收时仍然被保留。如果没有外围实例的情况下，也需要分配实例，就不能使用非静态成员类，因为非静态成员类的实例必须要有一个外围实例。

4. 在列表里面有图片的情况下，监听滑动不加载图片

5. 多个不同的布局，可以创建不同的ViewHolder和convertView进行重用

### RecyclerView和ListView的对比？

1. [需要自己添加HeaderView和FooterView：](http://blog.csdn.net/lmj623565791/article/details/51854533)

2. **ViewHolder是用来保存视图引用的类，无论是ListView亦或是RecyclerView**。只不过在ListView中，ViewHolder需要自己来定义，且这只是一种推荐的使用方式，不使用当然也可以，这不是必须的。只不过不使用ViewHolder的话，ListView每次getView的时候都会调用findViewById\(int\)，这将导致ListView性能展示迟缓。而在**RecyclerView中使用RecyclerView.ViewHolder则变成了必须，尽管实现起来稍显复杂，但它却解决了ListView面临的上述不使用自定义ViewHolder**时所面临的问题。

3. 我们知道**ListView只能在垂直方向上滚动，Android API没有提供ListView在水平方向上面滚动的支持**。或许有多种方式实现水平滑动，但是相信我，ListView并不是设计来做这件事情的。但是RecyclerView相较于ListView，在滚动上面的功能扩展了许多。它可以支持多种类型列表的展示要求，主要如下：

   1. LinearLayoutManager，可以支持水平和竖直方向上滚动的列表。
   2. StaggeredGridLayoutManager，可以支持交叉网格风格的列表，类似于瀑布流或者Pinterest。
   3. GridLayoutManager，支持网格展示，可以水平或者竖直滚动，如展示图片的画廊。

4. **列表动画是一个全新的、拥有无限可能的维度**。起初的Android API中，删除或添加item时，item是无法产生动画效果的。后面随着Android的进化，Google的Chat Hasse推荐使用ViewPropertyAnimator属性动画来实现上述需求。 相比较于ListView，RecyclerView.ItemAnimator则被提供用于在RecyclerView添加、删除或移动item时处理动画效果。同时，如果你比较懒，不想自定义ItemAnimator，你还可以使用DefaultItemAnimator。

5. ListView的Adapter中，**getView是最重要的方法，它将视图跟position绑定起来，是所有神奇的事情发生的地方。同时我们也能够通过registerDataObserver在Adapter中注册一个观察者**。RecyclerView也有这个特性，RecyclerView.AdapterDataObserver就是这个观察者。ListView有三个Adapter的默认实现，分别是ArrayAdapter、CursorAdapter和SimpleCursorAdapter。然而，RecyclerView的Adapter则拥有除了内置的内DB游标和ArrayList的支持之外的所有功能。RecyclerView.Adapter的实现的，我们必须采取措施将数据提供给Adapter，正如BaseAdapter对ListView所做的那样。

6. 在ListView中如果我们想要在item之间添加间隔符，我们只需要在布局文件中对ListView添加如下属性即可：

   ```
     android:divider="@android:color/transparent"
     android:dividerHeight="5dp"
   ```

7. ListView通过AdapterView.OnItemClickListener接口来探测点击事件。而RecyclerView则通过RecyclerView.OnItemTouchListener接口来探测触摸事件。它虽然增加了实现的难度，但是却给予开发人员拦截触摸事件更多的控制权限。

8. ListView可以设置选择模式，并添加MultiChoiceModeListener，如下所示：

   ```
    listView.setChoiceMode(ListView.CHOICE_MODE_MULTIPLE_MODAL);
   ```

## ListView和RecycleView缓存机制

* **缓存原理**:![](http://oa5504rxk.bkt.clouddn.com/week18_listview/1.png)

* **ListView两级缓存**:![](http://oa5504rxk.bkt.clouddn.com/week18_listview/2.png)

* **RecycleView四级缓存**:![](http://oa5504rxk.bkt.clouddn.com/week18_listview/3.jpg)

* **ListView缓存流程**:

![](http://oa5504rxk.bkt.clouddn.com/week18_listview/4.jpg)

* **RecycleView缓存流程**:

![](http://oa5504rxk.bkt.clouddn.com/week18_listview/5.jpg)







