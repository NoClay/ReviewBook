弹性滑动

弹性滑动的一般实现思想是：将一次大的滑动分成若干次滑动实现，在一个时间段完成即可。

## 3.3.1 使用Scroller {#331-使用scroller}

我们之前介绍过如何使用Scroller进行弹性滑动，这里只是分析Scroller的滑动原理。

![](http://i1.piimg.com/567571/8c9e27b165959ae5.png)

**注意：**

1. Scroller的computeScrollOffset的方法用来计算当前的偏移量，它利用当前已经流逝的事件与时间段的百分比来计算当前应该的偏移量
2. 上述方法返回值为boolean，如果返回true表示滑动还没有结束，false则表示滑动已经结束。
3. Scroller的设计，并没有对View产生任何的引用，只是利用了view的重绘机制，相当值得称赞

## 3.3.2 通过动画 {#332-通过动画}

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

## 3.3.3 使用延时策略 {#333-使用延时策略}

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



