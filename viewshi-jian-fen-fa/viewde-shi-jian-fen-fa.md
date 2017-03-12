# View的事件分发机制 {#34-view的事件分发机制}

## 3.4.1 点击事件的传递规则 {#341-点击事件的传递规则}

首先介绍三个方法：

1. public boolean dispatchTouchEvent\(MotionEvent ev\)

   用来进行事件的分发，如果事件能够传递给当前View，则此方法一定会被调用，返回结果受当前View的onTouchevent方法的影响，表示是否消耗当前事件。

2. public boolean onInterceptTouchEvent\(MotionEvent event\)

   用来判断是否拦截某个事件，需要注意的是，如果当前View拦截了某个事件，那么同一个事件序列，此方法不会被再次调用，返回结果表示是否拦截当前事件

3. public boolean onTouchEvent\(MotionEvent event\)

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

1. 同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列
   以down事件开始，中间含有数量不定的move事件，最终以up事件结束。
2. 正常情况下，
   一个事件序列只能被一个View拦截且消耗。因为一旦一个元素拦截了某次事件，那么同一个事件序列内的所有事件都会直接交给它处理，
   因此用一个事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过onTouchEvent强行传递给其他View。
3. 某个View一旦决定拦截，那么这一个事件序列都只能由它来处理，并且它的onInterceptTouchEvent不会再被调用。
4. 某个View一旦开始处理事件，如果它不消耗ACTION\_DOWN事件（onTouchEvent返回了false），那么同一事件序列的其他事件都不会再交给它来处理，并且事件将重新交由它的父元素去处理，即父元素的onTouchEvent会被调用。
5. 如果View不消耗除ACTION\_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终这些消失的点击事件会传递给Activity处理。
6. ViewGroup默认不拦截任何事件。
7. View没有onInterceptTouchEvent方法
   ，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。
8. View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的。View的longClickable属性都默认为false。
9. View的enable属性不影响onTouchEvent的默认返回值。
   哪怕一个View是disable状态的，只有它的clickable或者longClickable有一个为true，那么它的onTouchEvent就返回true。
10. onClick会发生的前提是当前View是可点击的，并且它收到了down和up的事件。
11. 事件传递过程是由外向内的，即事件总是先传递给父元素，然后再由父元素分发给子View，通过
    requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION\_DOWN事件除外。

## 3.4.2 事件分发的源码解析 {#342-事件分发的源码解析}

### 1 Activity对点击事件的分发过程 {#1-activity对点击事件的分发过程}

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

可以看出：事件首先交给Activity所附属的Window进行分发，如果返回true， 整个事件循环就结束了，如果返回false，意味着没有内部View消耗事件，则交给Activity的onTouchEvent处理。只处理一种情况：当mWindow.shouldCloseOnTouch\(this, event\)返回true时调用finish\(\)方法结束Activity。其他情况一律不处理。那么mWindow.shouldCloseOnTouch\(this, event\)在哪种情况下返回true就显得非常重要。我们来看下Window类中shouldCloseOnTouch\(\)方法的的源码：

```java
      public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
        if (mCloseOnTouchOutside && event.getAction() == MotionEvent.ACTION_DOWN
                && isOutOfBounds(context, event) && peekDecorView() != null) {
            return true;
        }
        return false;
    }
```

也就是说，如果设置了android:windowCloseOnTouchOutside属性为true，并且当前事件是ACTION\_DOWN，而且点击发生在Activity之外，同时Activity还包含视图的话，则返回true；表示该点击事件会导致Activity的结束。比较典型的情况就是dialog形的Activity。

### 2 那么Window如何将事件传递给ViewGroup呢? {#2-那么window如何将事件传递给viewgroup呢}

我们看上边源码，可以看到，事件被传递给了和Activity对应的Window对象。通过查看Window源码我们知道，Window是一个虚拟类。Window的累注释中明确说明目前只有一个实现类叫做PhoneWindow。

我们来看一下PhoneWindow是如何传递触摸事件的：

```
@Override public boolean superDispatchTouchEvent(MotionEvent event) {     
  return mDecor.superDispatchTouchEvent(event); 
}
```

可以看到事件被传递给了Activity的DecorView。我们知道，DecorView是Activity的顶级视图。他是PhoneWindow的一个内部类。  
DecorView对事件的传递：

    private final class DecorView extends FrameLayout implements RootViewSurfaceTaker { 
      public boolean superDispatchTouchEvent(MotionEvent event) {         
        return super.dispatchTouchEvent(event);     
      }
    }`

我们看到，DecorView的superDispatchTouchEvent方法调用了父类的dispatchTouchEvent\(event\)方法。而DecorView是继承自FrameLayout的。FrameLayout继承自ViewGroup并且没有重写dispatchTouchEvent\(event\)方法。

至此，触摸事件已经从Activity传递到了和该Activity对应的ViewTree的顶级ViewGroup中。

### 3 顶层View对点击事件的分发过程 {#3-顶层view对点击事件的分发过程}

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
2. 如果mFitstTarget == null 
   &
   &
    actionMasked != MotionEvent.ACTION\_DOWN，则这个ViewGroup的onInterceptTouchEvent不会被调用。
3. FLAG\_DISALLOW\_INTERCEPT是一个标志位，当设置之后，ViewGroup将无法拦截除了ACTION\_DOWN的其他事件， 这是因为ACTION\_DOWN会重置该标志位，这个标志位由requestDisallowInterceptTouchEvent这个方法设置。
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

**注意：**

1. dispatchTransformedTouchEvent实际上就是调用子元素的dispatchTouchEvent，从而完成了一轮事件分发
2. 如果子元素的dispatchTouchEvent返回true，那么mFirstTouchTarget会被赋值，同时跳出循环
3. 如果遍历所有的子元素都没有被处理，呢么可能是没有子元素，或者子元素处理了点击事件，但是子元素在onTouchEvent中返回了false，从而导致dispatchTouchEvent中返回false，这种情况ViewGroup会自己处理点击事件

### 4 View对点击事件的处理 {#4-view对点击事件的处理}

View对点击事件的处理十分简单，由于View没有子元素，所以只需要自己处理，我们看伪代码：

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



