# 1 Activity的生命周期解析

## 1.1 典型情况下的生命周期

在正常的情况下，Activity可能经历的生命周期：

* onCreate：创建Activity，做初始化工作，包括setContentView加载界面资源
* onRestart：在Activity从不可见到可见的时候调用
* onStart：Activity正在启动，这个时候Activity已经可见了，但是没有在前台显示出来，无法和用户交互
* onResume：表示Activity已经可见了，而且出现在前台并开始活动
* onPause：Activity正在停止，可以做一些存储数据，停止动画的工作，但是不可以太耗时，否则会影响新的Activity的显示。onPause必须执行完毕，之后新的Activity的onResume才可以执行。
* onStop：表示Activity即将停止，可以做一些稍微重量级的回收工作，但是还不能太耗时
* onDestory：表示Activity即将销毁，这是生命周期中最后一个回调，可以在这里做一些回收工作和最后的资源释放。

正常情况下Activity生命周期图如下：  
![](https://timgsa.baidu.com/timg?image&quality=80&size=b10000_10000&sec=1488616947&di=3d5f24c1feba35962ab60fe29aec5129&src=http://www.oschina.net/uploads/space/2011/1117/210131_1I26_195301.png)

需要注意的是：  
1. 针对一个特定的Activity，第一次启动，回调：onCreate-&gt;onStart-&gt;onResume  
2. 当用户打开新的Activity或者切换到桌面的时候，回调：onPause-&gt;onStop，但是如果新的Activity采用了透明主题，则当前Activity不会回调onStop  
3. onStart和onStop是从Activity是否可见这个角度来回调的，onResume和onPause是从Activity是否位于前台这个角度来回调的  
4. 在新的Activity启动之前，位于栈顶的Activity必须要先onPause后，新的Activity才能启动  
5. 不能在onPause中做重量级的操作，因为必须onPause后才会显示新的Activity，所以我们应该尽量在onStop中做操作，使得新的Activity可以更快的切换到前台。

## 1.2 异常情况下的生命周期

### 1. 情况1：资源相关的系统配置发生改百年导致Activity被杀死并重新创建

```
当系统的配置（横屏或者竖屏切换等）发生了改变的时候，Activity会销毁并重新创建，生命周期如下图所示：
```

![这里写图片描述](http://img.blog.csdn.net/20170220152414073?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjcwMzUxMjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注意：  
1. onSaveInstancestate方法调用在onStop之前，但是与onPause没有既定的时序关系

1. 在activity异常发生的时候，Activity会调用onSaveInstanceState来保存数据，然后activity会委托Window去保存数据，然后Window会委托DecorView去保存数据，然后顶层容器一一通知它的子元素进行保存数据。

2. 布局中的每一个View默认实现了onSaveInstanceState\(\)方法，这样的话，这个UI的任何改变都会自动地存储和在activity重新创建的时候自动地恢复。**但是这种情况只有在你为这个UI提供了唯一的ID之后才起作用，如果没有提供ID，app将不会存储它的状态。**

### 2. 情况2：内存不足导致低优先级的Activity被杀死

Activity按照优先级从高到低，可以分为以下三种：  
（1） 前台Activity--正在和用户进行交互的Activity，优先级最高  
（2） 可见但非前台Activity--比如Activity弹出了一个对话框，导致Activity可见但是位于后台无法交互  
（3） 后台Activity--已经被暂停的Activity，比如执行了onStop，优先级最低

### 1.3 常见生命周期的变化



**启动Activity:**

> onCreate\(\)—&gt;onStart\(\)—&gt;onResume\(\)\`，Activity进入运行状态

**Activity退居后台**:

> 原Activity 转到 新Activity 或 按Home键回到主屏 :`onPause()—>onStop()`，Activity进入停滞状态。

**Activity返回前台**:

> `onRestart()—>onStart()—>onResume()`，Activity再次回到运行状态。

**Activity退居后台，且系统内存不足，系统会杀死这个后台状态的Activity，若再次回到这个Activity**:

> `onCreate()–>onStart()—>onResume()`\(将重新走一次Activity的初始化生命周期\)

**锁屏**:

> `onPause()->onStop()`

**解锁**:

> `onStart()->onResume()`

**切屏**:

> 不设置Activity的`android:configChanges`时，切屏会重新走一次初始化生命周期，切横屏时会执行一次，切竖屏时会执行两次，设置Activity的`android:configChanges="orientation"`时，切屏还是会重新走一次初始化生命周期，切横、竖屏时只会执行一次，设置Activity的`android:configChanges="orientation|keyboardHidden"`时，切屏不会重新走一次初始化生命周期，只会执行onConfigurationChanged方法

**横屏：**

> onSaveInstanceState\(\)-&gt;onPause\(\)-&gt;onStop\(\)-&gt;onDestroy\(\)-&gt;onCreate\(\)-&gt;onStart\(\)-&gt;onRestoreInstanceState\(\)-&gt;onResume\(\)

**竖屏:**

> `onSaveInstanceState()->onPause()->onStop-()->onDestroy()->onCreate()->onStart()->onRestoreInstanceState()->onResume()->onSaveInstanceState()->onPause()->onStop()->onDestroy()->onCreate()->onStart()->onRestoreInstanceState()->onResume()`

**Activity失去焦点**:

> onPause\(\)\`

**Activity不可见**:

> `onStop()`

### 如何让Activity在系统配置改变的时候不销毁重建？

答：给Activity的configChanges的属性加上值即可，通常我们常用的值有locale（本地位置发生了改变，一般为切换了系统语言），orientation（屏幕方向发生了改变，比如旋转了屏幕），keyboardHidden（键盘的可访问性发生了改变，比如调出了键盘），值得注意的是screenSize和smallestScreenSize，它们的行为和编译选项有关，但是和运行环境无关。  
具体的使用方法如下：

```xml
<activity
        android:name = 
        android:configChanges= "orientation|screenSize"
        android:label="">
</activity>
```

当我们配置了这个以后，这个activity就不会重新创建，取而代之的是调用Activity.onConfigurationChanged方法

### 1.5 缓存方法

1. **onSaveInstanceState**\(Bundle outState\)

   * onSaveInstanceState方法被调用的情况有 :

     * 当用户按下HOME键时。

     * 长按HOME键，选择运行其他的程序时。

     * 按下电源按键（关闭屏幕显示）时。

     * 从Activity A中启动一个新的Activity时。

     * 屏幕方向切换时，例如从竖屏切换到横屏时。（如果不指定configchange属性）

2. **onRestoreInstanceState**\(Bundle outState\)

   * onRestoreInstanceState方法需要注意的是 :

     * onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用的。

     * onRestoreInstanceState的bundle参数也会传递到onCreate方法中，你也可以选择在onCreate方法中做数据还原。

     * onRestoreInstanceState在onstart之后执行。

   * onRestoreInstanceState方法被调用的前提是 :

     * Activity A“确实”被系统销毁了，例如，当正在显示Activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到Activity A，这种情况下Activity A一般不会因为内存的原因被系统销毁，故Activity A的onRestoreInstanceState方法不会被执行。

## 2 Activity的启动模式

### 2.1 Activity的LaunchMode

#### （1） standard：标准模式

在这种模式下，谁启动了这个Activity，那么这个Activity就会运行在启动它的那个Activity所在的栈中，而且每次启动都会创建一个新的实例，这就是当我们使用ApplicationContext启动Activity的时候会报错，但是我们可以为待启动的Activity指定FLAG\_ACTIVITY\_NEW\_TASK，这时候就可以用singleTask模式启动Activity

#### （2） singleTop： 栈顶复用模式

在这种模式下，如果新的Activity已经位于任务栈的栈顶，那么此Activity就不会重建，但是会调用它的onNewIntent方法，但是如果不是位于栈顶，那么已然会重新创建实例

#### （3）singleTask：栈内复用模式

在这种模式下，系统首先会查找是否存在A想要的任务栈，如果不存在，就重新创建一个任务栈，然后创建实例放入，如果存在任务栈，如果内部有该活动实例，则将其调到栈顶，并调用onNewIntent方法，如果不存在实例，则创建实例。  
注意：singleTask有clearTop的效果，如果任务栈内有活动ABCD，这时候请求启动B，则会清除栈顶，导致最后任务栈为AB。

#### （4）singleInstance：单例模式

在这种模式下，这是一种加强的singleTask模式，它具有singleTask模式的所有特性，同时具有这种模式的Activity只能单独地位于一个任务栈中。

### 如何设置任务栈？

答：默认情况下，所有活动默认的任务栈是包名，但我们可以利用Activity的TaskAffinity的可以设置任务栈。

### 如何设置启动模式？

1. 利用AndroidMenifest设置

   ```xml
   <activity
        android:name = 
        android:configChanges= "orientation|screenSize"
        android:launchMode="singleInstance"
        android:label="">
   </activity>
   ```

2. 在intent中设置标志位

   ```java
   Intent intent = new Intent(context, Activity.class);
   intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
   ```

   注意：第一种方式无法直接为Activity设置FLAG\_ACTIVITY\_CLEAR\_TOP,第二种方式无法为Activity指定singleInstance。

### 2.2 Activity的Flags

常用的Flags：  
1. FLAG\_ACTIVITY\_NEW\_TASK：指定singleTask启动模式  
2. FLAG\_ACTIVITY\_SINGLE\_TOP：指定singleTop启动模式  
3. FLAG\_ACTIVITY\_CLEAR\_TOP：当这个活动启动时，所有位于之上的活动都会出栈，通常会和第一个flag配合使用  
4. FLAG\_ACTIVITY\_EXCLUDE\_FROM\_RECENTS：当我们不希望用户通过历史列表返回我们的活动时候进行标记，相当于在xml中加入`android:excludeFromRecents="true"`

## 3 IntentFilter的匹配规则

IntentFilter的过滤信息有action、category、data。下面介绍匹配规则：

### 3.1 action的匹配规则

一个过滤规则可以有多个action，但是只要Intent中的任何一个和过滤规则中的任何一个相同即可匹配成功，也就是说action匹配规则要求action存在而且必须和过滤规则中的其中一个action相同

### 3.2 category的匹配规则

一个Intent中可以添加多个category，但只能添加一个action，所以category的匹配规则是如果不存在也可匹配，但如果存在多个category，任何一个都要和过滤规则中的category中的其中一个相同

### 3.3 data的匹配规则

data由两部分组成，mimeType和URI，mimeType指的是媒体类型，而URI的结构为：  
`<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]`

* scheme：URI的模式，比如http，file，content，URI必须指定scheme。
* host：主机名，URI必须指定主机名
* port：端口号，仅当scheme和host指定后有意义
* path、pathPrefix、pathPattern：路径信息，path为完整的路径，pathPattern为完整的路径信息，但是可以包含通配符“\*”，pathPrefix表示路径的前缀信息。  
  如下两种特殊的写法，作用是一样的：

  ```xml
    <intent-filter ....>
       <data android:scheme="file" android:host="www.baidu.com"/>
    </intent-filter>
    <intent-filter ....>
       <data android:scheme="file" />
       <data android:host="www.baidu.com"/>
    </intent-filter>
  ```

  注意：

* 如果在过滤规则中没有设定URI，默认值为content和file

* 使用Intent设置数据的时候推荐使用setDataAndType，否则的话setData和setType会互相抹除对方设置

### 注意

`<category android:name="android.intent.catefory.DEFAULT"/>`的意义是在我们使用`queryActivity, resolveActivity`  的时候返回结果不为空，那么一定可以启动成功。

