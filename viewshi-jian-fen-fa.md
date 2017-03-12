# 3.1 View基础知识 {#31-view基础知识}

## 3.1.1 什么是View {#311-什么是view}

我们在[安卓](http://lib.csdn.net/base/android)中使用的各种View，如TextView和Button等等，其实都可以分为两类，一类是View，一类是ViewGroup，而ViewGroup内部可以嵌套子View，而子View可以是一个View，也可以是一个ViewGroup，同时需要注意的是ViewGroup继承自View。

## 3.1.2 View的位置参数 {#312-view的位置参数}

View具有四个位置属性：  
1. left 左上角横坐标  
2. top 左上角纵坐标  
3. right 右下角横坐标  
4. bottom 右下角纵坐标  
注意：  
5. 这些都是相对于View的父容器来说的，这是一种相对坐标  
6. 从Android3.0开始，View增加了参数x, y, translationX 和translationY， 其中x, y是View左上角的坐标，而另外两个则是View左上角相对于父容器的偏移量。

## 3.1.3 MotionEvent和TouchSlop {#313-motionevent和touchslop}

### 1 MotionEvent {#1-motionevent}

MotionEvent是手指接触屏幕发生的一系列事件：  
1. ACTION\_DOWN 手指刚接触屏幕  
2. ACTION\_MOVE 手指在屏幕上移动  
3. ACTION\_UP 手机从屏幕上松开的一瞬间  
一般的话，当触摸屏幕的时候（含点击）， Down事件一定发生，UP事件也一定会发生，但是Move事件可能发生。  
注意：通过MotionEvent对象，我们可以获得点击事件相对于当前View的x、y坐标`getX/getY`或者得到相对于手机屏幕左上角的x、 y坐标`getRawX/getRawY`

### 2 TouchSlop {#2-touchslop}

一个滑动常量，如果滑动距离小于这个常量，那么系统将无法识别出滑动，这个值由设备决定。我们可以通过`ViewConfiguration.get(getContext()).getScaledTouchSlop()`方法获取。我们可以在处理滑动的时候，利用这个值作为滑动的过滤条件

## 3.1.4 VelocityTracker 、GestureDetector 和Scoller {#314-velocitytracker-gesturedetector-和scoller}

### 1 VelocityTracker {#1-velocitytracker}

如何利用速度追踪器，如果我们想要对当前点击事件进行追踪，那么我们可以在View的onTouchEvent方法中添加追踪如下:

```
VelocityTracker velocity = VelocityTracker.obtain();
velocity.addMovement(event);
```

当我们想要知道速度的时候，可以这样获取

```
         velocityTracker.computeCurrentVelocity(1000);
        int xVelocity = (int) velocityTracker.getXVelocity();
        int yVelocity = (int) velocityTracker.getYVelocity();
```

如果最后我们不需要它的时候，我们可以将其重置并回收内存

```
        velocityTracker.clear();
        velocityTracker.recycle();
```

注意：这里的速度计算公式为：

> 速度 = （终点位置 - 起点位置） / 时间段  
> 所以，当速度为正的时候，代表手指顺着坐标系的正方向滑动

### 2 GestureDetector {#2-gesturedetector}

用于实现手势检测，我们需要以当前上下文创建GestureDetector对象并实现对应接口。  
我们可以使用下面代码，接管一个View的onTouchEvent事件，即在待监听View的onTouchEvent方法中添加如下实现。

```
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
![](http://img.blog.csdn.net/20170304165957579?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjcwMzUxMjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

![](http://img.blog.csdn.net/20170304170111215?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjcwMzUxMjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

**注意：**如果只是监听滑动相关，建议自己在View 的`onTouchEvent`中实现，如果需要监听双击这种行为，可以采用GestureDetector实现

### 3 Scroller {#3-scroller}

弹性滑动对象，用于实现View的弹性滑动，当使用View的scrollTo和scrollBy方法进行滑动的时候，过程是瞬间完成的，这给人一种很不好的用户体验，所以我们可以使用Scroller实现弹性滑动。那么弹性滑动如何利用Scroller实现呢？

实现代码：

```
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



