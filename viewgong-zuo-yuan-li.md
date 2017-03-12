# 引言： {#引言}

我们之前在View的事件体系提到了Window和DecorView，这两者是两个不同的概念，所以我们需要一个ViewRoot（对应于ViewRootImpl）作为纽带，实现View的三大流程，当Activity对象被创建完毕后，会将DecorView加载到Window中，同时创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView产生关联。

```
root = 
new
 ViewRootImpl(View.getContext(), display);
root.setView(view, wparams, panelParentView);


```

# 初识ViewRoot {#初识viewroot}

View的绘制流程是从ViewRoot的performTraversals方法开始的，图示如下：

![](http://www.2cto.com/uploadfile/Collfiles/20160906/20160906092852134.png "performTraversals的工作流程")

首先我们需要理解的是，ViewRoot的遍历过程，这里用measure测量过程举例，测量过程中，ViewRoot调用performMeasure方法会从ViewTree的上方到下方测量完成，当正在测量的View是ViewGroup的时候，首先调用measure方法测量自身，其次调用onMeasure方法测量子类，如果正在测量的View是一个View元素的话，那么会调用measure测量自身。这样子一个View树就被测量完成，layout流程和draw流程也和measure类似，唯一不同的是performDraw的传递过程中是在draw方法中通过dispatchDraw来实现的。

# View中的静态内部类MeasureSpec {#view中的静态内部类measurespec}

MeasureSpec代表的是一个32位int值，高2位代表的是SpecMode测量模式，低30位代表的是SpecSize某种测量模式下的规格，这个内部类中的测量模式如下：

1. UNSPECIFIED

   > Measure specification mode: The parent has not imposed any constraint on the child. It can be whatever size it wants.
   >
   > 测量规范模式:父控件对子控件没有施加任何限制。它可以是任何大小。

   这种情况一般用于系统内部，表示一种测量的状态。

2. EXACTLY

   > Measure specification mode: The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.
   >
   > 测量规范模式:父控件决定子控件的确切大小。子控件将会被设置这个边界不管它想要多大

   它对应LayoutParams中的match\_parent和具体的数值这两种模式\_

3. AT\_MOST

   > Measure specification mode: The child can be as large as it wants up\* to the specified size.
   >
   > 测量规范模式:子控件在父控件的给定大小内取值

   它对应LayoutParams中的wrap\_content

\*\*注意：\*\*MeasureSpec不是唯一由LayoutParams决定的，LayoutParams需要和父容器一起才能决定View的MeasureSpec，从而进一步的决定View的高、宽。

# 子控件的MeasureSpec {#子控件的measurespec}

对于一个子控件，他的MeasureSpec必然由父容器的MeasureSpec和自身的LayoutParams共同决定。下面是一个基本的对照表（横行为父容器的模式，竖列为子控件的LayoutParams）：

| EXACTLY | AT\_MOST | UNSPECIFIED |
| :--- | :--- | :--- |
| dp/px | EXACTLY\|childSize | EXACTLY\|childSize |
| match\_parent | EXACTLY\|parentSize | AT\_MOST\|parentSize |
| wrap\_content | AT\_MOST\|parentSize | AT\_MOST\|parentSize |

# View的工作流程 {#view的工作流程}

* measure操作：
 
  * measure操作主要是用于计算视图的大小，也就是视图的宽度和长度。在view中定义为final类型，要求子类不能修改，measure\(\)函数中又会调用onMeasure\(\)，视图的最终在onMeasure方法中确定。也就是说，measure只是对onMeasure的一个包装，子类可以覆写onMeasure\(\)实现自己计算的视图大小的方式，并通过setMeasureDimension\(width,hight\)保存计算结果。这里有一个奇葩的逻辑，measure过程中比如TextView，如果设置了背景（图片之类），则minWidth将在android:minWindth和背景的最小宽度中取一个最大值，如果没有设置背景，则为android:minWindth。
* layout操作：
 
  * layout操作是设置视图在屏幕中显示的位置，在view中定义为final类型，要求子类不能修改，layout\(\)函数中有两个基本操作：
  * setFrame\(l,t,r,b\)；ltrb是自视图在父view中的具体的位置。
  * onLayout\(\)在view中这个函数什么都不做，提供该函数主要是为了viweGroup类型布局自视图用的。
* draw操作：
 
  * draw操作利用前两部都得到的参数，将视图显示在屏幕上，然后也就基本上完成了整个视图的绘制工作。子类也不应该修改该方法，因为其内部定义了绘图的基本操作。
  * 绘制背景
  * 如果要绘制视图显示渐变框，这里会做一些准备
  * 绘制视图本身，即调用onDraw\(\)函数。在view中onDraw\(\)是个空函数，也就是说具体的视图都要覆写该函数来实现自己的显示（比如TextView在这里实现了绘制文字的过程）。而对于ViewGroup则不需要实现该函数，因为作为容器是“没有内容“的，其包含了多个子view，而子View已经实现了自己的绘制方法，因此只需要告诉子view绘制自己就可以了，也就是下面的dispatchDraw\(\)方法;
  * 绘制子视图，即dispatchDraw\(\)函数。在view中这是个空函数，具体的视图不需要实现该方法，它是专门为容器类准备的，也就是容器类必须实现该方法；
  * 绘制滚动条；

**注意：**

1. onMeasure方法由measure方法唤起，每个View都有measure和onMeasure方法。

2. measure方法更像是对自身控件视图大小应该的尺寸的获取，View中的注释是这样子的

   > This is called to find out how big a view should be. The parent supplies constraint information in the width and height parameters.
   >
   > 这就是所谓的找出一个视图应该多大。获取父控件约束在宽度和高度参数的信息。

3. onMeasure方法更像是对自身子View（如果自身是ViewGroup）或者内容的尺寸的计算，通过这种计算对自身尺寸提供一个更加精确，有效的计算或测量。

   > Measure the view and its content to determine the measured width and the measured height. This method is invoked by {@link \#measure\(int, int\)} and should be overridden by subclasses to provide accurate and efficient measurement of their contents.
   >
   > 测量视图和它的内容来确定测量的宽度和高度测量。该方法调用自{ @link measure\(int,int\)},应该被子类覆盖提供准确、有效的测量的内容。
   >
   > When overriding this method, you_must_call {@link \#setMeasuredDimension\(int, int\)} to store the measured width and height of this view. Failure to do so will trigger an`IllegalStateException`, thrown by {@link \#measure\(int, int\)}. Calling the superclass’ {@link \#onMeasure\(int, int\)} is a valid use.
   >
   > 重写该方法时,你必须调用{ @link \# setMeasuredDimension\(int,int\)}来存储这个视图的宽度和高度测量。不这么做将触发一个IllegalStateException,抛出的{ @link \#measure\(int,int\)}。调用超类的{ @link \# onMeasure\(int,int\)}是一个有效的使用。
   >
   > The base class implementation of measure defaults to the background size, unless a larger size is allowed by the MeasureSpec. Subclasses should override {@link \#onMeasure\(int, int\)} to provide better measurements of their content.
   >
   > 基类实现测量大小默认为背景,除非MeasureSpec允许一个更大的规模。子类应该覆盖{ @link \# onMeasure\(int,int\)}提供更好的测量的内容。
   >
   > If this method is overridden, it is the subclass’s responsibility to make sure the measured height and width are at least the view’s minimum height and width \({@link \#getSuggestedMinimumHeight\(\)} and{@link \#getSuggestedMinimumWidth\(\)}\).
   >
   > 如果该方法被覆盖,子类必须确保测量的高度和宽度至少视图的最小高度和宽度\({ @link \# getSuggestedMinimumHeight\(\)}和{ @link \# getSuggestedMinimumWidth\(\)}\)。

4. 直接继承View的自定义控件需要重写onMeasure方法并设置wrap\_content时候的自身大小，否则在副局中使用wrap\_content就相当与match\_parent。

# 如何获取一个View的尺寸？ {#如何获取一个view的尺寸}

1. 通过getMeasuredWidth/Height可以获取测量后的View的尺寸，但是建议在onLayout中获取宽和高，否则可能为0.

2. 因为Activity的生命周期和View的生命周期不同步，所以我们如果需要在Activity中获取View的尺寸，可以在onWindowFocusChanged方法中获取，当Activity窗口得到、失去焦点的时候都会调用此方法。

3. 通过’targetView.post\(runnable\)’：通过post方法将一个runnable投递到消息队列的尾部，然后等待Looper调用此Runnable的时候，View也已经初始化好了。

4. 使用ViewTreeObserver：使用此类的众多回调可以实现这个功能，比如使用OnGlobalLayoutListener，当View树的状态发生改变或者View树内部的View的可见性发生改变的时候，会执行此回调。

   ```
          ViewTreeObserver observer = targetView.getViewTreeObserver();
          observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
              @Override
              public void onGlobalLayout() {
                  targetView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
                  int width = targetView.getMeasuredWidth();
                  int height = targetView.getMeasuredHeight();
              }
          });
   ```

# View的测量尺寸和最终尺寸有什么区别？ {#view的测量尺寸和最终尺寸有什么区别}

getMeasuredWidth/Height是测量高，在measure过程生成，getWidth/Height是最终高，在layout过程中生成，除此之外，基本一致，除非重写了View的layout方法。

# 自定义View {#自定义view}

自定义View的分类：

1. 继承自View重写onDraw方法：静态显示或者动态显示一些不规则图形
2. 继承自ViewGroup派生特殊的Layout
3. 继承特定的View
4. 继承特定的ViewGroup

自定义View须知：

1. 支持wrap\_content：在onMeasure中处理wrap\_content
2. 支持padding：直接继承自View的控件需要在draw方法中处理padding，直接继承自ViewGroup的控件，需要在onMeasure和onLayout中考虑padding和子元素的margin对其造成的影响。
3. 尽量不要在View中使用Handler，没必要：View内部提供了post系列方法
4. View中如果有线程或者动画，需要及时停止。可以在onDetachedFromWindow方法中停止线程或者动画，在onAttachedToWindow方法中开始动画
5. 处理好滑动冲突：外部拦截或者内部拦截

# 自定义View的一些重要方法 {#自定义view的一些重要方法}

## 1.invalidate\(\)方法 ： {#1invalidate方法}

说明：请求重绘View树，即draw\(\)过程，假如视图发生大小没有变化就不会调用layout\(\)过程，并且只绘制那些“需要重绘的”视图，即谁\(View的话，只绘制该View ；ViewGroup，则绘制整个ViewGroup\)请求invalidate\(\)方法，就绘制该视图。

一般引起invalidate\(\)操作的函数如下：

1. 直接调用invalidate\(\)方法，请求重新draw\(\)，但只会绘制调用者本身。
2. setSelection\(\)方法：请求重新draw\(\)，但只会绘制调用者本身。
3. setVisibility\(\)方法 ：当View可视状态在INVISIBLE转换VISIBLE时，会间接调用invalidate\(\)方法，继而绘制该View。
4. setEnabled\(\)方法 ：请求重新draw\(\)，但不会重新绘制任何视图包括该调用者本身。

## 2.requestLayout\(\)方法 ：会导致调用measure\(\)过程 和 layout\(\)过程 。 {#2requestlayout方法-会导致调用measure过程-和-layout过程}

说明：只是对View树重新布局layout过程包括measure\(\)和layout\(\)过程，不会调用draw\(\)过程，但不会重新绘制任何视图包括该调用者本身。

一般引起操作的方法如下：

setVisibility\(\)方法：

当View的可视状态在INVISIBLE/VISIBLE 转换为GONE状态时，会间接调用requestLayout\(\) 和invalidate方法。 同时，由于整个个View树大小发生了变化，会请求measure\(\)过程以及draw\(\)过程，同样地，只绘制需要“重新绘制”的视图。

## 3.requestFocus\(\)方法： {#3requestfocus方法}

说明：请求View树的draw\(\)过程，但只绘制“需要重绘”的视图。

自定义View的实例：[https://github.com/NoClay/TestView.git](https://github.com/NoClay/TestView.git)

