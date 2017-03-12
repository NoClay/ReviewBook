### 1. 优化方式

1. 重用convertView：防止重复创建ItemView

2. 使用ViewHolder：findViewById太耗时，通过convertView.setTag操作，可以直接取出而不用重复findViewById

3. 使用static class ViewHolder：如果声明成员类不要求访问外围实例，就要始终把static修饰符放在它的声明中，使它成为静态成员类，而不是非静态成员类。因为非静态成员类的实例会包含一个额外的指向外围对象的引用，保存这份引用要消耗时间和空间，并且导致外围类实例符合垃圾回收时仍然被保留。如果没有外围实例的情况下，也需要分配实例，就不能使用非静态成员类，因为非静态成员类的实例必须要有一个外围实例。

4. 在列表里面有图片的情况下，监听滑动不加载图片

5. 多个不同的布局，可以创建不同的ViewHolder和convertView进行重用

### 2.RecyclerView和ListView的对比？

1. RecyclerView支持线性布局，网格布局，瀑布流（StaggeredGridLayoutManager）

2. 支持横向滑动和纵向滑动

3. 支持局部刷新

4. RecyclerView需要自定义OnClickListener之类

5. [需要自己添加HeaderView和FooterView：](http://blog.csdn.net/lmj623565791/article/details/51854533)



