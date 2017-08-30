# View的滑动冲突 {#35-view的滑动冲突}

## 3.5.1 常见的滑动冲突场景 {#351-常见的滑动冲突场景}

### 1 外部滑动方向和内部滑动的方向不一致 {#1-外部滑动方向和内部滑动的方向不一致}

![](http://img.blog.csdn.net/20160724160722340 "这里写图片描述")

这种情况我们经常遇见，比如使用viewpaper+listview时，在这种效果中，可以通过左右滑动切换页面，而每一个页面往往又是一个listview，本来在这种情况下是有冲突的，但是Viewpaper内部处理了这个滑动冲突，因此采用viewpaper我们无需关注这个问题，如果我们采用的不是Viewpaper而是ScrollView等，那么必须手动处理滑动冲突，否则内外两层只能有一层滑动，那就是滑动冲突。另外内部左右滑动，外部上下滑动也同样属于该类。

### 2 外部滑动方向和内部滑动方向一致 {#2-外部滑动方向和内部滑动方向一致}

![](http://img.blog.csdn.net/20160724160754294 "这里写图片描述")

这种情况就比较复杂，当内外两层都在同一个方向可以滑动的时候，显然存在逻辑问题，因为当手指开始滑动的时候，系统无法知道用户到底是想让那一层动，所以当手指滑动的时候就会出现问题，要么只能一层动，要么内外两成动的都很卡顿。

### 3 上述两种场景的嵌套 {#3-上述两种场景的嵌套}

不做过多描述

## 3.5.2 滑动冲突的处理规则 {#352-滑动冲突的处理规则}

一般来讲，处理规则可以分为滑动的方向，距离差，滑动的角度，速度差，以及业务需求来判断。

## 3.5.3 滑动冲突的解决办法 {#353-滑动冲突的解决办法}

### 1 外部拦截法 {#1-外部拦截法}

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

### 2 内部拦截法 {#2-内部拦截法}

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



