## 3.2.1 使用View的ScrollTo/ScrollBy {#321-使用view的scrolltoscrollby}

View提供了两个方法来实现滑动功能，分别是scrollTo和scrollBy，其中需要注意的是：

1. ScrollTo是一种绝对滑动
2. ScrollBy是一种相对滑动
3. View的内部属性mScrollX和mScrollY一直等于View边缘和View内容在两个方向上的距离
4. 使用这个方法只可以将View的内容滑动，View本身并不会有任何移动

## 3.2.2 使用动画 {#322-使用动画}

平移是一种滑动，我们可以使用动画移动，这里主要操作的是View的translationX和translationY属性，既可以采用传统的View动画，也可以采用属性动画，如果采用属性动画而且需要兼容到3.0以下版本的话，可以采用开源动画库nineoldandroids。

```
ObjectAnimator.ofFloat(targetView, "translationX", 0, 100)
    .setDuration(100).start();
```

我们首先看上边的源码：

```
public static ObjectAnimator ofFloat(Object target, String propertyName, float... values) {
        ObjectAnimator anim = new ObjectAnimator(target, propertyName);
        anim.setFloatValues(values);
        return anim;
    }
```

很显然使用属性动画是对View的移动，所以如果我们不需要兼容到[Android](http://lib.csdn.net/base/android)3.0以下的话，属性动画是一个不错的选择

## 3.2.3 改变布局参数 {#323-改变布局参数}

很显然在android中如果想要使用一个View，那么这个View必须有其各种各样的布局参数。我们可以通过改变LayoutParams的方法进行滑动。

```
ViewGroup.MarginLayoutParams params = targetView.getLayoutParams();
        params.leftMargin += 100;
        params.width += 100;
        targetView.requestLayout();
        //或者targetView.setLayoutParams(params);
```

## 3.2.4 滑动方式总结 {#324-滑动方式总结}

针对上边的方式做个小总结：

* scrollTo/scrollBy： 操作简单，适合对于View内容的滑动
* 动画：操作简单，主要适用于没有交互的View和实现复杂的动画效果
* 改变布局参数：操作稍微复杂，适用于有交互的View

下面我们使用onTouchEvent实现一个跟手滑动的效果，当然父布局不能够对触摸事件进行拦截，否则无效（我们之后会提到事件拦截）

```
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



