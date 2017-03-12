

#3.1 View基础知识
##3.1.1 什么是View
我们在安卓中使用的各种View，如TextView和Button等等，其实都可以分为两类，一类是View，一类是ViewGroup，而ViewGroup内部可以嵌套子View，而子View可以是一个View，也可以是一个ViewGroup，同时需要注意的是ViewGroup继承自View。
## 3.1.2 View的位置参数

View具有四个位置属性：
1. left 左上角横坐标
2. top 左上角纵坐标
3. right 右下角横坐标
4. bottom 右下角纵坐标
   注意：
5. 这些都是相对于View的父容器来说的，这是一种相对坐标
6. 从Android3.0开始，View增加了参数x, y, translationX 和translationY， 其中x, y是View左上角的坐标，而另外两个则是View左上角相对于父容器的偏移量。

##3.1.3 MotionEvent和TouchSlop
###1 MotionEvent
MotionEvent是手指接触屏幕发生的一系列事件：
1. ACTION_DOWN 手指刚接触屏幕
2. ACTION_MOVE 手指在屏幕上移动
3. ACTION_UP  手机从屏幕上松开的一瞬间
   一般的话，当触摸屏幕的时候（含点击）， Down事件一定发生，UP事件也一定会发生，但是Move事件可能发生。
   注意：通过MotionEvent对象，我们可以获得点击事件相对于当前View的x、y坐标`getX/getY` 或者得到相对于手机屏幕左上角的x、 y坐标` getRawX/getRawY`
###2 TouchSlop
一个滑动常量，如果滑动距离小于这个常量，那么系统将无法识别出滑动，这个值由设备决定。我们可以通过`ViewConfiguration.get(getContext()).getScaledTouchSlop()` 方法获取。我们可以在处理滑动的时候，利用这个值作为滑动的过滤条件
##3.1.4 VelocityTracker 、GestureDetector 和Scoller
###1 VelocityTracker
如何利用速度追踪器，如果我们想要对当前点击事件进行追踪，那么我们可以在View的onTouchEvent方法中添加追踪如下:
```java
VelocityTracker velocity = VelocityTracker.obtain();
velocity.addMovement(event);
```
当我们想要知道速度的时候，可以这样获取
```java
		velocityTracker.computeCurrentVelocity(1000);
        int xVelocity = (int) velocityTracker.getXVelocity();
        int yVelocity = (int) velocityTracker.getYVelocity();
```
如果最后我们不需要它的时候，我们可以将其重置并回收内存
```java
		velocityTracker.clear();
        velocityTracker.recycle();
```
注意：这里的速度计算公式为：
> 速度 = （终点位置 - 起点位置） / 时间段
> 所以，当速度为正的时候，代表手指顺着坐标系的正方向滑动

###2 GestureDetector
用于实现手势检测，我们需要以当前上下文创建GestureDetector对象并实现对应接口。
我们可以使用下面代码，接管一个View的onTouchEvent事件，即在待监听View的onTouchEvent方法中添加如下实现。
```java
        final GestureDetector gestureDetector = new GestureDetector(
                new GestureDetector.OnGestureListener() {
            @Override
            public boolean onDown(MotionEvent e) {
                return false;
            }

            @Override
            public void onShowPress(MotionEvent e) {

            }

            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                return false;
            }

            @Override
            public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
                return false;
            }

            @Override
            public void onLongPress(MotionEvent e) {

            }

            @Override
            public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
                return false;
            }
        });
        //解决长按屏幕无法拖动的现象
        gestureDetector.setIsLongpressEnabled(false);
        //将其放入对应View的onTouchEvent
        boolean consume = gestureDetector.onTouchEvent(event);
        return consume;
```

对于手势监听，不同的接口具有不同的方法如下：
![这里写图片描述](http://img.blog.csdn.net/20170304165957579?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjcwMzUxMjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170304170111215?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjcwMzUxMjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**注意：**如果只是监听滑动相关，建议自己在View 的 `onTouchEvent`中实现，如果需要监听双击这种行为，可以采用GestureDetector实现

### 3 Scroller

弹性滑动对象，用于实现View的弹性滑动，当使用View的scrollTo和scrollBy方法进行滑动的时候，过程是瞬间完成的，这给人一种很不好的用户体验，所以我们可以使用Scroller实现弹性滑动。那么弹性滑动如何利用Scroller实现呢？

实现代码：

```java
	Scroller scroller = new Scroller(mContext);
    //缓慢滚动到指定位置
    private void smoothScrollTo(int destX, int destY){
        int scrollX = getScrollX();
        int delta = destX - scrollX;
        //1000ms内滑向destX，效果就是慢慢滑动
		mScroller.startScroll(scrollX, 0, delta, 0, 1000);
		invalidate();
    }
	
	@Override
	public void computeScroll(){
		if(mScroller.computeScrollOffset()){
			scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
			postInvalidate();
		}
	}
```

# 3.2 View的滑动

## 3.2.1 使用View的ScrollTo/ScrollBy

View提供了两个方法来实现滑动功能，分别是scrollTo和scrollBy，其中需要注意的是：

1. ScrollTo是一种绝对滑动
2. ScrollBy是一种相对滑动
3. View的内部属性mScrollX和mScrollY一直等于View边缘和View内容在两个方向上的距离
4. 使用这个方法只可以将View的内容滑动，View本身并不会有任何移动

## 3.2.2  使用动画

平移是一种滑动，我们可以使用动画移动，这里主要操作的是View的translationX和translationY属性，既可以采用传统的View动画，也可以采用属性动画，如果采用属性动画而且需要兼容到3.0以下版本的话，可以采用开源动画库nineoldandroids。

```java
	ObjectAnimator.ofFloat(targetView, "translationX", 0, 100)
	.setDuration(100).start();
```

我们首先看上边的源码：

```java
public static ObjectAnimator ofFloat(Object target, String propertyName, float... values) {
        ObjectAnimator anim = new ObjectAnimator(target, propertyName);
        anim.setFloatValues(values);
        return anim;
    }
```

很显然使用属性动画是对View的移动，所以如果我们不需要兼容到android 3.0以下的话，属性动画是一个不错的选择

## 3.2.3 改变布局参数

很显然在android中如果想要使用一个View，那么这个View必须有其各种各样的布局参数。我们可以通过改变LayoutParams的方法进行滑动。

```java
ViewGroup.MarginLayoutParams params = targetView.getLayoutParams();
        params.leftMargin += 100;
        params.width += 100;
        targetView.requestLayout();
        //或者targetView.setLayoutParams(params);
```

另外LayoutParams可以改变View的很多布局参数：

```java
        /**
         * The left margin in pixels of the child. Margin values should be positive.
         * Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value
         * to this field.
         */
        @ViewDebug.ExportedProperty(category = "layout")
        public int leftMargin;

        /**
         * The top margin in pixels of the child. Margin values should be positive.
         * Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value
         * to this field.
         */
        @ViewDebug.ExportedProperty(category = "layout")
        public int topMargin;

        /**
         * The right margin in pixels of the child. Margin values should be positive.
         * Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value
         * to this field.
         */
        @ViewDebug.ExportedProperty(category = "layout")
        public int rightMargin;

        /**
         * The bottom margin in pixels of the child. Margin values should be positive.
         * Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value
         * to this field.
         */
        @ViewDebug.ExportedProperty(category = "layout")
        public int bottomMargin;

        /**
         * The start margin in pixels of the child. Margin values should be positive.
         * Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value
         * to this field.
         */
        @ViewDebug.ExportedProperty(category = "layout")
        private int startMargin = DEFAULT_MARGIN_RELATIVE;

        /**
         * The end margin in pixels of the child. Margin values should be positive.
         * Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value
         * to this field.
         */
        @ViewDebug.ExportedProperty(category = "layout")
        private int endMargin = DEFAULT_MARGIN_RELATIVE;

        /**
         * The default start and end margin.
         * @hide
         */
        public static final int DEFAULT_MARGIN_RELATIVE = Integer.MIN_VALUE;

```

## 3.2.4 滑动方式总结

针对上边的方式做个小总结：

- scrollTo/scrollBy： 操作简单，适合对于View内容的滑动
- 动画：操作简单，主要适用于没有交互的View和实现复杂的动画效果
- 改变布局参数：操作稍微复杂，适用于有交互的View

下面我们使用onTouchEvent实现一个跟手滑动的效果，当然父布局不能够对触摸事件进行拦截，否则无效（我们之后会提到事件拦截）

```java
package com.example.no_clay.helloworld.Test3;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;

/**
 * Created by no_clay on 2017/3/5.
 */

public class FollowYourHandView extends View {

    Paint mPaint;
    int mlastX;
    int mlastY;

    public FollowYourHandView(Context context) {
        super(context);
        initPaint();
    }

    public FollowYourHandView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initPaint();
    }

    public FollowYourHandView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initPaint();
    }

    void initPaint(){
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.RED);
        mPaint.setStrokeWidth(3f);
    }

    /**
     * 此处实现跟手滑动
     * @param event 事件
     * @return 返回是否处理
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //获取绝对坐标
        int x = (int) event.getRawX();
        int y = (int) event.getRawY();
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:{
                break;
            }
            case MotionEvent.ACTION_MOVE:{
                int deltaX = x - mlastX;
                int deltaY = y - mlastY;
                int translationX = (int) (getTranslationX() + deltaX);
                int translationY = (int) (getTranslationY() + deltaY);
                setTranslationX(translationX);
                setTranslationY(translationY);
                break;
            }
            case MotionEvent.ACTION_UP:{
                break;
            }
        }
        mlastY = y;
        mlastX = x;
        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
      //画一个圆形
        int width = getWidth();
        int height = getHeight();
        canvas.drawCircle(width / 2, height / 2, width / 2, mPaint);
        super.onDraw(canvas);
    }
}

```

# 3.3 弹性滑动

弹性滑动的一般实现思想是：将一次大的滑动分成若干次滑动实现，在一个时间段完成即可。

## 3.3.1 使用Scroller

我们之前介绍过如何使用Scroller进行弹性滑动，这里只是分析Scroller的滑动原理。

```flow
st=>start: invalidate
op=>operation: View重绘
op1=>operation: draw方法
op2=>operation: 调用computeScroll方法
op3=>operation: 调用postInvalidate二次重绘
cond=>condition: 滑动结束?
e=>end
st->op->op1->op2->op3->cond
cond(yes)->e
cond(no)->op
```

**注意：**

1. Scroller的computeScrollOffset的方法用来计算当前的偏移量，它利用当前已经流逝的事件与时间段的百分比来计算当前应该的偏移量
2. 上述方法返回值为boolean，如果返回true表示滑动还没有结束，false则表示滑动已经结束。
3. Scroller的设计，并没有对View产生任何的引用，只是利用了view的重绘机制，相当值得称赞

## 3.3.2 通过动画

这里我们只探讨属性动画，利用属性动画实现弹性滑动相当的简单，但是我们也可以使用属性动画内部原理，来模仿一下Scroller的弹性滑动。代码如下：

```java
public void valueAnimation(final View targetView, final int startX, final int deltaX, long duration){
        final ValueAnimator animator = ValueAnimator.ofInt(0, 1).setDuration(duration);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float fraction = animator.getAnimatedFraction();
                targetView.scrollTo((int) (startX + (deltaX * fraction)), 0);
            }
        });
        animator.start();
    }
```

## 3.3.3 使用延时策略

延时策略：通过发送一系列延时消息从而达到一种渐进式的效果，可以使用Handler或者View的postDelayed方法，也可以是哟on个线程的sleep方法。

下边是使用Handler的实例：

```java
private static final int MESSAGE_SCROLL_TO = 1;
    private static final int FRAME_COUNT = 30;
    private static final int DELAYED_TIME = 33;

    private Button mButton1;
    private View mButton2;

    private int mCount = 0;

    @SuppressLint("HandlerLeak")
    private Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
            case MESSAGE_SCROLL_TO: {
                mCount++;
                if (mCount <= FRAME_COUNT) {
                    float fraction = mCount / (float) FRAME_COUNT;
                    int scrollX = (int) (fraction * 100);
                    mButton1.scrollTo(scrollX, 0);
                    mHandler.sendEmptyMessageDelayed(MESSAGE_SCROLL_TO, DELAYED_TIME);
                }
                break;
            }

            default:
                break;
            }
        };
    };
```

# 3.4 View的事件分发机制

## 3.4.1 点击事件的传递规则

首先介绍三个方法：

1. public boolean dispatchTouchEvent(MotionEvent ev)

   用来进行事件的分发，如果事件能够传递给当前View，则此方法一定会被调用，返回结果受当前View的onTouchevent方法的影响，表示是否消耗当前事件。

2. public boolean onInterceptTouchEvent(MotionEvent event)

   用来判断是否拦截某个事件，需要注意的是，如果当前View拦截了某个事件，那么同一个事件序列，此方法不会被再次调用，返回结果表示是否拦截当前事件

3. public boolean onTouchEvent(MotionEvent event)

   在dispatchTouchEvent中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次收到事件。

其实它们的关系可以用如下伪代码表示：

```java
public boolean dispatchTouchEvent(MotionEvent ev){
	if(onIterceptTouchevent(ev)){
      	boolean consume = false;
		if(onTouchListener != null){
     		consume = onTouchListener.onTouch(event); 
		}
  		if(!consume){
      		return onTouchEvent(event);
  		}
      	return consume;
	}else{
      	return child.dispatchTouchEvent(ev);
	}
  	return false;
}
public boolean onTouchEvent(MotionEvent event){
  	//onTouchEvent处理
  	if(onClickListener != null){
        return onClickListener.onClick(View v);
  	}
}
```

**注意：**

1. onTouchListener比onTouchEvent优先级高
2. onClickListner的优先级最低，处于事件传递的尾端

关于事件传递的机制有以下结论：

1. 同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列<u>以down事件开始，中间含有数量不定的move事件，最终以up事件结束。</u>
2. 正常情况下，<u>一个事件序列只能被一个View拦截且消耗。因为一旦一个元素拦截了某次事件，那么同一个事件序列内的所有事件都会直接交给它处理，</u>因此用一个事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过onTouchEvent强行传递给其他View。
3. 某个View一旦决定拦截，那么这一个事件序列都只能由它来处理，并且它的onInterceptTouchEvent不会再被调用。
4. 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回了false），那么同一事件序列的其他事件都不会再交给它来处理，并且事件将重新交由它的父元素去处理，即父元素的onTouchEvent会被调用。
5. 如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终这些消失的点击事件会传递给Activity处理。
6. <u>ViewGroup默认不拦截任何事件。</u>
7. <u>View没有onInterceptTouchEvent方法</u>，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。
8. <u>View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的。View的longClickable属性都默认为false。</u>
9. <u>View的enable属性不影响onTouchEvent的默认返回值。</u>哪怕一个View是disable状态的，只有它的clickable或者longClickable有一个为true，那么它的onTouchEvent就返回true。
10. <u>onClick会发生的前提是当前View是可点击的，并且它收到了down和up的事件。</u>
11. 事件传递过程是由外向内的，即事件总是先传递给父元素，然后再由父元素分发给子View，通过<u>requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。</u>

## 3.4.2 事件分发的源码解析

### 1  Activity对点击事件的分发过程

我们先看源码：

```java
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
          //在Activity中是空实现，内部什么都没有
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
    public boolean onTouchEvent(MotionEvent event) {
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }

        return false;
    }
```

可以看出：事件首先交给Activity所附属的Window进行分发，如果返回true， 整个事件循环就结束了，如果返回false，意味着没有内部View消耗事件，则交给Activity的onTouchEvent处理。只处理一种情况：当mWindow.shouldCloseOnTouch(this, event)返回true时调用finish()方法结束Activity。其他情况一律不处理。那么mWindow.shouldCloseOnTouch(this, event)在哪种情况下返回true就显得非常重要。我们来看下Window类中shouldCloseOnTouch()方法的的源码：

```
      public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
        if (mCloseOnTouchOutside && event.getAction() == MotionEvent.ACTION_DOWN
                && isOutOfBounds(context, event) && peekDecorView() != null) {
            return true;
        }
        return false;
    }
```

1. mCloseOnTouchOutside是一个boolean变量，它是由Window的android:windowCloseOnTouchOutside属性值决定。
2. isOutOfBounds(context, event)是判断该event的坐标是否在context(对于本文来说就是当前的Activity)之外。是的话，返回true；否则，返回false。
3. peekDecorView()则是返回PhoneWindow的mDecor。

也就是说，如果设置了android:windowCloseOnTouchOutside属性为true，并且当前事件是ACTION_DOWN，而且点击发生在Activity之外，同时Activity还包含视图的话，则返回true；表示该点击事件会导致Activity的结束。比较典型的情况就是dialog形的Activity。

### 2 那么Window如何将事件传递给ViewGroup呢?

我们看上边源码，可以看到，事件被传递给了和Activity对应的Window对象。通过查看Window源码我们知道，Window是一个虚拟类。Window的累注释中明确说明目前只有一个实现类叫做PhoneWindow。

我们来看一下PhoneWindow是如何传递触摸事件的：

```java
@Override public boolean superDispatchTouchEvent(MotionEvent event) {     
  return mDecor.superDispatchTouchEvent(event); 
}
```

可以看到事件被传递给了Activity的DecorView。我们知道，DecorView是Activity的顶级视图。他是PhoneWindow的一个内部类。
DecorView对事件的传递：

```java
private final class DecorView extends FrameLayout implements RootViewSurfaceTaker { 
  public boolean superDispatchTouchEvent(MotionEvent event) {         
    return super.dispatchTouchEvent(event);     
  }
}`
```

我们看到，DecorView的superDispatchTouchEvent方法调用了父类的dispatchTouchEvent(event)方法。而DecorView是继承自FrameLayout的。FrameLayout继承自ViewGroup并且没有重写dispatchTouchEvent(event)方法。

至此，触摸事件已经从Activity传递到了和该Activity对应的ViewTree的顶级ViewGroup中。

### 3 顶层View对点击事件的分发过程

顶级View一般都是一个ViewGroup。会调用ViewGroup里边的dispatchTouchEvent方法，之后的逻辑是：如果拦截事件返回true，则事件由ViewGroup处理，如果这时候ViewGroup中的mOnTouchListener不为空，则onTouch被调用，否则调用onTouchEvent， 在onTouchEvent中，如果设置了mOnClickListener，则onClick会被调用，如果顶级ViewGroup不拦截事件，则事件会传递给它所在点击事件链上的子View，这个时候子View的dispatchTouchEvent会被调用。基本关系如下：

```java
public boolean dispatchTouchEvent(MotionEvent ev){
	if(onIterceptTouchevent(ev)){
      	boolean consume = false;
		if(onTouchListener != null){
     		consume = onTouchListener.onTouch(event); 
		}
  		if(!consume){
      		return onTouchEvent(event);
  		}
      	return consume;
	}else{
      	return child.dispatchTouchEvent(ev);
	}
  	return false;
}
public boolean onTouchEvent(MotionEvent event){
  	//onTouchEvent处理
  	if(onClickListener != null){
        return onClickListener.onClick(View v);
  	}
}
```

查看ViewGroup中拦截的源码：

```java
 // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
```

**注意：**

1. mFirstTouchTarget 即当事件由ViewGroup的子元素处理成功的时候，mFirstTouchTarget会被赋值并指向子元素，这也是为什么，一旦不拦截事件交给子元素后，再也不会处理其它事件
2. 如果mFitstTarget == null && actionMasked  != MotionEvent.ACTION_DOWN，则这个ViewGroup的onInterceptTouchEvent不会被调用。
3.  FLAG_DISALLOW_INTERCEPT是一个标志位，当设置之后，ViewGroup将无法拦截除了ACTION_DOWN的其他事件， 这是因为ACTION_DOWN会重置该标志位，这个标志位由requestDisallowInterceptTouchEvent这个方法设置。
4. onInterceptTouchevent并不是每次斗会调用
5. 我们如果想处理所有事件，可以设置在dispatchTouchEvent，只有它才保证了每次都会调用

我们看ViewGroup向下分发子View的源码：

```java
final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }
```

基本流程图如下：

```flow
st=>start: 获取子元素列表
op1=>operation: 获取当前元素
op2=>operation: 交给当前子元素处理
cond1=>condition: 是否在播动画 && 点击事件坐标是否落在子元素区域内
e=>end: 遍历结束
st->op1->cond1
cond1(yes)->op2->e
cond1(no)->op1
```

**注意：**

1. dispatchTransformedTouchEvent实际上就是调用子元素的dispatchTouchEvent，从而完成了一轮事件分发
2. 如果子元素的dispatchTouchEvent返回true，那么mFirstTouchTarget会被赋值，同时跳出循环
3. 如果遍历所有的子元素都没有被处理，呢么可能是没有子元素，或者子元素处理了点击事件，但是子元素在onTouchEvent中返回了false，从而导致dispatchTouchEvent中返回false，这种情况ViewGroup会自己处理点击事件

### 4 View对点击事件的处理

View对点击事件的处理十分简单，由于View没有子元素，所以只需要自己处理，我们看源码：

```java
   public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }

```

View对点击事件的处理可以用下列伪代码实现：

```java
public boolean dispatchTouchEvent(MotionEvent ev){
	if(li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)){
      //首先判断OnTouchListener，如果处理事件返回true
      	result = true;
	}
  	if (!result && onTouchEvent(event)) {
        result = true;
    }
  	return result;
}
public boolean onTouchEvent(MotionEvent event){
  	//onTouchEvent处理
        if ((viewFlags & ENABLED_MASK) == DISABLED) {
          //即使View不可用依然会消耗点击事件
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return (((viewFlags & CLICKABLE) == CLICKABLE
                    || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                    || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
        }
        if (mTouchDelegate != null) {
          //如果设置了代理，会执行代理中的onTouchEvent
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }

        if (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
                (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
          //只要view的CLICKABLE 和 LONG_CLICKABLE 有一个为true，就会消耗事件
         	switch(event.getAction()){
              case ACTION_UP:{
                performClick();
                break;
              }
         	}
        }
}
    public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
          //如果有listener， 则onClick
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        return result;
    }
```

# 3.5 View的滑动冲突

## 3.5.1 常见的滑动冲突场景

### 1 外部滑动方向和内部滑动的方向不一致

![这里写图片描述](http://img.blog.csdn.net/20160724160722340)

这种情况我们经常遇见，比如使用viewpaper+listview时，在这种效果中，可以通过左右滑动切换页面，而每一个页面往往又是一个listview，本来在这种情况下是有冲突的，但是Viewpaper内部处理了这个滑动冲突，因此采用viewpaper我们无需关注这个问题，如果我们采用的不是Viewpaper而是ScrollView等，那么必须手动处理滑动冲突，否则内外两层只能有一层滑动，那就是滑动冲突。另外内部左右滑动，外部上下滑动也同样属于该类。

### 2 外部滑动方向和内部滑动方向一致

![这里写图片描述](http://img.blog.csdn.net/20160724160754294)

这种情况就比较复杂，当内外两层都在同一个方向可以滑动的时候，显然存在逻辑问题，因为当手指开始滑动的时候，系统无法知道用户到底是想让那一层动，所以当手指滑动的时候就会出现问题，要么只能一层动，要么内外两成动的都很卡顿。

### 3 上述两种场景的嵌套

不做过多描述

## 3.5.2 滑动冲突的处理规则

一般来讲，处理规则可以分为滑动的方向，距离差，滑动的角度，速度差，以及业务需求来判断。

## 3.5.3 滑动冲突的解决办法

### 1 外部拦截法

我们可以在父容器进行拦截处理，如果父容器需要当前点击事件，则拦截，否则不拦截

伪代码：

```java
@Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        boolean intercepted = false;
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN: {
            intercepted = false;
            break;
        }
        case MotionEvent.ACTION_MOVE: {
            if (父容器需要点击事件) {
                intercepted = true;
            } else {
                intercepted = false;
            }
            break;
        }
        case MotionEvent.ACTION_UP: {
            intercepted = false;
            break;
        }
        default:
            break;
        }

        mLastXIntercept = x;
        mLastYIntercept = y;
        return intercepted;
    }
```

### 2 内部拦截法

我们在父容器中不拦截所有事件，所有的事件都传递给子元素，如果子元素需则处理，否则交给父容器，需要配合requestDisallowInterceptTouchEvent方法。

伪代码如下：

```java
@Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN: {
            parent.requestDisallowInterceptTouchEvent(true);
            break;
        }
        case MotionEvent.ACTION_MOVE: {
            int deltaX = x - mLastX;
            int deltaY = y - mLastY;
            Log.d(TAG, "dx:" + deltaX + " dy:" + deltaY);
            if (父容器需要此类点击事件) {
                mHorizontalScrollViewEx2.requestDisallowInterceptTouchEvent(false);
            }
            break;
        }
        case MotionEvent.ACTION_UP: {
            break;
        }
        default:
            break;
        }

        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(event);
    }
```

当然父容器也需要对应的实现，不拦截任何事件：

```java
   @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        int action = event.getAction();
        if (action == MotionEvent.ACTION_DOWN) {
            mLastX = x;
            mLastY = y;
            if (!mScroller.isFinished()) {
                mScroller.abortAnimation();
                return true;
            }
            return false;
        } else {
            return true;
        }
    }
```



