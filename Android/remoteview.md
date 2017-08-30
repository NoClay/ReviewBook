# 版权说明

文章原载于：[天意博文](http://www.haotianyi.win/2017/04/07/view/RemoteViews详细解释/)

本文在此基础上进行了部分修改。

# AppWidget

想要完全的理解RetmoteViews必须要说明一下Android Widet。

Android widget 也称为桌面插件，其是android系统应用开发层面的一部分。Android中的AppWidget与google widget和中移动的widget并不是一个概念，这里的AppWidget只是把一个进程的控件嵌入到别外一个进程的窗口里的一种方法。

## AppWidgetFramework

Android系统增加了AppWidget 框架，用以支持widget类型应用的开发。AppWidget 框架主要由两个部件来组成：

（1）AppWidgetService是框架的的核心类，是系统 service之一，它负责widgets的管理工作。加载，删除，定时事件等都需要AppWidgetService的处理。开机自启动的。

​       AppWidgetService存在的目的主要是解开AppWidgetProvider和AppWidgetHost之间的耦合。如果 AppWidgetProvider和AppWidgetHost的关系固定死了，AppWidget就无法在任意进程里显示了。而有了 AppWidgetService，AppWidgetProvider根本不需要知道自己的AppWidget在哪里显示了。

（2）AppWidgetManager 负责widget视图的实际更新以及相关管理。

## 工作流程

![绘制流程201704071445](http://oaxelf1sk.bkt.clouddn.com/绘制流程201704071445.png)

1. **编写一个widget（先不考虑后台服务以及用户管理界面等）**

   实际是写一个事件监听类即一个BroadcastReceiver子类，当然框架已经提供了一个辅助类AppWidgetProvider，实现的类只要实现其方法即可，其中必须实现的方法是onUpdate ，其实就是一个定时事件，widget监听此事件 另外就是规划好视图（layout），将此widget打包安装。

2. **当android系统启动时，AppWidgetService 就将负责检查所有的安装包**

   将检查AndroidManifest.xml（不要告诉我不知道，如果不知道可要看看基本开发知识了）文件中有`<metadata android:name="android.appwidget.provider" android:resource="@xml/appwidget_info" />` 信息的程序包记录下来

3. **从用户菜单将已经安装的widget添加到桌面 也就是将widget在桌面上显示出来**

   这个是由AppWidgetService和AppWidgetManager完成的，其中AppWidgetManager 将负责将视图发送到桌面显示出来，并将此widget记录到系统文件中

4. **AppWidgetService将根据widget配置中的updatePeriodMillis属性来定时发送ACTION\_APPWIDGET\_UPDATE事件**

   此事件将激活widget的事件监听方法onUpdate，此方法将通过AppWidgetManager完成widget内容的更新和其他操作。

## AppWidgetHost

AppWidgetHost 是实际控制widget的地方，大家注意，widget不是一个单独的用户界面程序，他必须寄生在某个程序（activity）中，这样如果程序要支持widget寄生就要实现AppWidgetHost，桌面程序（Launcher）就实现了这个接口。

AppWidgetHost和AppWidgetHostView是在框架中定义的两个基类。

AppWidgetHostView是真正的View，但它只是一个容器，用来容纳实际的AppWidget的View。这个AppWidget的View是根据RemoteViews的描述来创建。

## AppWidgetProvider

AppWidgetProvider是AppWidget提供者需要实现的接口，它实际上是一个BroadcastReceiver。只不过子类要实现的不再是onReceive。作为AppWidgetProvider的实现者，一定要实现**onUpdate**函数，因为这个函数决定widget的显示方式，如果没有这个函数widget根本没办法出现

# RemoteViews介绍

RemoteViews表示的是一个view结构，它可以在**其他进程中显示**。由于它在其他进程中显示，为了能够更新它的界面，RemoteViews提供了一组基础的操作用于跨进程更新它的界面。

RemoteViews主要用于**通知栏通知和桌面小部件**的开发，通知栏通知是通过`NotificationManager`的`notify`方法来实现的；桌面小部件是通过`AppWidgetProvider`来实现的，它本质上是一个广播\(BroadcastReceiver\)。这两者的界面都是运行在`SystemServer`进程中（跨进程）

**RemoteViews并不是一个真正的View，它没有实现View的接口，而只是一个用于描述View的实体。**比如：创建View需要的资源ID和各个控件的事件响应方法。RemoteViews会通过进程间通信机制传递给AppWidgetHost。

现在我们可以看出，Android中的AppWidget与google widget和中移动的widget并不是一个概念，这里的AppWidget只是把一个进程的控件嵌入到别外一个进程的窗口里的一种方法。View在另 外一个进程里显示，但事件的处理方法还是在原来的进程里。

![snipaste\_20170407\_114428](http://www.jcodecraeer.com/uploads/userup/12809/1F41GG453-1c2-1.png)

# RemoteViews应用

## 在通知栏的应用

创建一个通知：

```java
    public void showNotification(View view) {
        Notification notification = new Notification();
        notification.icon = R.mipmap.ic_launcher;
        notification.tickerText = "天意博文";
        notification.flags = Notification.FLAG_AUTO_CANCEL;
        notification.when = System.currentTimeMillis();
        Intent intent = new Intent(this, MainActivity.class);
        intent.putExtra("ceshi",0);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);

        RemoteViews remoteViews = new RemoteViews(getPackageName(), R.layout.notification);
        remoteViews.setTextViewText(R.id.tv,"天意博文textview");
        remoteViews.setImageViewResource(R.id.iv,R.mipmap.ic_launcher);
        remoteViews.setTextColor(R.id.tv,getResources().getColor(R.color.colorPrimaryDark));
        PendingIntent pendingIntent1 = PendingIntent.getActivity(this, 0, new Intent(this, Main2Activity.class), PendingIntent.FLAG_UPDATE_CURRENT);
        remoteViews.setOnClickPendingIntent(R.id.iv,pendingIntent1);
        notification.contentView = remoteViews;
        notification.contentIntent = pendingIntent;
        NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        notificationManager.notify(0,notification);
    }
```

对用的布局notification.xml：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="horizontal">

    <TextView
        android:id="@+id/tv"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"/>

    <ImageView
        android:id="@+id/iv"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"/>
</LinearLayout>
```

显示如下图所示的通知栏：

![snipaste\_20170407\_100103](http://www.jcodecraeer.com/uploads/userup/12809/1F41GG453-5109-2.png)

并且点击图片的时候会跳转到Main2Activity：

![snipaste\_20170407\_100141](http://www.jcodecraeer.com/uploads/userup/12809/1F41GG453-IA-3.png)

给对应的布局View设置点击事件：

> remoteViews.setOnClickPendingIntent\(R.id.iv,pendingIntent1\)

单击通知时的响应事件：

> notification.contentIntent = pendingIntent//对应的是第一个pendingIntent

## RemoteViews在桌面小部件的应用

新建桌面小部件，在as中创建十分简单，在布局中新建widget，下一步即可：

![snipaste\_20170407\_101710](http://www.jcodecraeer.com/uploads/userup/12809/1F41GG453-4551-4.png)

创建完成之后会创建如下几个文件：![snipaste\_20170407\_101919](http://www.jcodecraeer.com/uploads/userup/12809/1F41GG453-6239-5.png)

home\_widget.xml是小部件的布局文件，home\_widget\_info.xml是小部件的配置文件，Home\_Widget.java是小部件的逻辑控制文件

![snipaste\_20170407\_102124](http://www.jcodecraeer.com/uploads/userup/12809/1F41GG453-2U3-6.png)

小部件的本质是一个BroadcastReceiver，所以还要在mainifest.xml中注册

home\_widget.xml具体实现，都是自动生成，和普通的布局没有区别

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#09C"
                android:padding="@dimen/widget_margin">

    <TextView
        android:id="@+id/appwidget_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:layout_margin="8dp"
        android:background="#09C"
        android:contentDescription="@string/appwidget_text"
        android:text="@string/appwidget_text"
        android:textColor="#ffffff"
        android:textSize="24sp"
        android:textStyle="bold|italic"/>

</RelativeLayout>
```

home\_widget\_info.xml具体实现，是小部件的配置文件，指定了布局，大小更新时间等

```
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:initialKeyguardLayout="@layout/home__widget"
    android:initialLayout="@layout/home__widget"
    android:minHeight="40dp"
    android:minWidth="40dp"
    android:previewImage="@drawable/example_appwidget_preview"
    android:resizeMode="horizontal|vertical"
    android:updatePeriodMillis="86400000"
    android:widgetCategory="home_screen">
</appwidget-provider>
```

Androidmainfest.xml更新标签，注意在intent-filter中一定要含有`<action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>`，这是系统的规范

```
        <receiver android:name="layout.Home_Widget">
            <intent-filter>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>
                <!--能够响应自定义的action-->
                <action android:name="haotianyi.win"/>
            </intent-filter>

            <meta-data
                android:name="android.appwidget.provider"
                android:resource="@xml/home__widget_info"/>
        </receiver>
```

### 具体实现逻辑

当每一次点击小部件的时候，显示的textview都会显示当前的时间

![201704071058](http://www.jcodecraeer.com/uploads/userup/12809/1F41GG453-2113-7.gif)

```
public class Home_Widget extends AppWidgetProvider {
    public static final String ACTION_CLICK = "haotianyi.win";
    public static final String TAG = "haotianyi.win";
    public static int click_count = 0;

    public void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                                int appWidgetId) {

        String widgetText = context.getString(R.string.appwidget_text);
        // Construct the RemoteViews object
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.home__widget);
        views.setTextViewText(R.id.appwidget_text, widgetText);

        Intent intentClick = new Intent();
        intentClick.setAction(ACTION_CLICK);

        PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, intentClick, 0);
        views.setOnClickPendingIntent(R.id.appwidget_text, pendingIntent);

        // Instruct the widget manager to update the widget
        appWidgetManager.updateAppWidget(appWidgetId, views);
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
        Log.e(TAG, "onReceive: onReceive");
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.home__widget);
        views.setTextViewText(R.id.appwidget_text, "天意博文" + System.currentTimeMillis());

        Intent intentClick = new Intent();
        intentClick.setAction(ACTION_CLICK);

        PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, intentClick, 0);
        views.setOnClickPendingIntent(R.id.appwidget_text, pendingIntent);
        AppWidgetManager manager = AppWidgetManager.getInstance(context);

        // Instruct the widget manager to update the widget
        manager.updateAppWidget(new ComponentName(context, Home_Widget.class), views);
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        // There may be multiple widgets active, so update all of them
        Log.e(TAG, "onReceive: onUpdate");
        for (int appWidgetId : appWidgetIds) {
            updateAppWidget(context, appWidgetManager, appWidgetId);
        }
    }

    @Override
    public void onEnabled(Context context) {
        // Enter relevant functionality for when the first widget is created
        Log.e(TAG, "onReceive: onEnabled");
    }

    @Override
    public void onDisabled(Context context) {
        // Enter relevant functionality for when the last widget is disabled
    }
}
```

实现逻辑，首先第一次添加的时候会执行onReceive方法，在方法中设置了点击监听，当发生点击事件的时候，由于自定义了action，所以含有特定action的broadcastReceiver会启动，在当前案例中也就是Home\_Widget在一次启动，同时又执行了onReceive，更新视图，同时设置事件监听。

### 小部件的生命周期

`onEnable`：当小部件**第一次添加到桌面时调用**，小部件可以添加多个但是只在第一次添加的时候调用；

`onUpdate`：小部件**被添加时或者每次小部件更新时**都会调用一次该方法，每个周期小部件都会自动更新一次，不是点击的时候更新，而是到指定配置文件时间的时候才更新

`onDeleted`：每删除一次小部件就调用一次；

`onDisabled`：当最后一个该类型的小部件被删除时调用该方法；

`onReceive`：这是广播内置的方法，用于分发具体的事件给其他方法，所以该方法一般要调用`super.onReceive(context, intent);` 如果自定义了其他action的广播，就可以在调用了父类方法之后进行判断，如上面代码所示。

# PendingIntent

`PendingIntent`表示一种处于Pending状态的Intent，pending表示的是即将发生的意思，它是在将来的某个不确定的时刻放生，而Intent是立刻发生。

PendingIntent支持三种待定意图：启动Activity\(getActivity\)，启动Service\(getService\)，发送广播\(getBroadcast\)。

## 匹配规则

如果两个Intent的ComponentName和intent-filter都相同，那么这两个Intent就是相同的，Extras不参与Intent的匹配过程。

参数flags常见的类型有：`FLAG_ONE_SHOT`、`FLAG_NO_CREATE`、`FLAG_CANCEL_CURRENT`、`FLAG_UPDATE_CURRENT`。

`FLAG_ONE_SHOT`：当前描述的PendingIntent只能被调用一次，然后它就会被自动cancel。如果后续还有相同的PendingIntent，那么它们的send方法就会调用失败。对于通知栏消息来说，如果采用这个flag，那么同类的通知只能使用一次，后续的通知单击后将无法打开。

`FLAG_NO_CREATE`：当前描述的PendingIntent不会主动创建，如果当前PendingIntent之前不存在，那么getActivity、getService和getBroadcast方法会直接返回null，即获取PendingIntent失败。这个标志位使用很少。

`FLAG_CANCEL_CURRENT`：当前描述的PendingIntent如果已经存在，那么它们都会被cancel，然后系统会创建一个新的PendingIntent。

对于通知栏消息来说，那些被cancel的通知单击后将无法打开。

`FLAG_UPDATE_CURRENT`：当前描述的PendingIntent如果已经存在，那么它们都会被更新，即它们的Intent中的Extras会被替换成最新的。

# RemoteViews机制

RemoteView没有findViewById方法，因此无法访问里面的View元素，而必须通过RemoteViews所提供的一系列set方法来完成，这是通过反射调用的

通知栏和小组件分别由NotificationManager\(NM\)和AppWidgetManager\(AWM\)管理，而NM和AWM通过Binder分别和SystemService进程中的NotificationManagerService以及AppWidgetService中加载的，而它们运行在系统的SystemService中，这就和我们进程构成了跨进程通讯。

## 构造方法

`public RemoteViews(String packageName, int layoutId)`，第一个参数是当前应用的包名，第二个参数是待加载的布局文件。

## 支持组件

布局：`FrameLayout、LinearLayout、RelativeLayout、GridLayout`

组件：`Button、ImageButton、ImageView、ProgressBar、TextView、ListView、GridView、ViewStub`等（例如EditText是不允许在RemoveViews中使用的，使用会抛异常）。

## 原理

系统将view操作封装成`Action`对象，Action同样实现了Parcelable接口，通过Binder传递到SystemServer进程。远程进程通过RemoteViews的`apply`方法来进行view的更新操作，RemoteViews的apply方法内部则会去遍历所有的action对象并调用它们的apply方法来进行view的更新操作。

这样做的好处是不需要定义大量的Binder接口，其次批量执行RemoteViews中的更新操作提高了程序性能。

## 工作流程

首先RemoteViews会通过Binder传递到SystemService进程，因为RemoteViews实现了Parcelable接口，因此它可以跨进程传输，系统会根据RemoteViews的包名等信息拿到该应用的资源；然后通过LayoutInflater去加载RemoteViews中的布局文件。接着系统会对View进行一系列界面更新任务，这些任务就是之前我们通过set来提交的。set方法对View的更新并不会立即执行，会记录下来，等到RemoteViews被加载以后才会执行。

## apply和reApply的区别

apply会加载布局并更新界面，而reApply则只会更新界面。通知栏和桌面小部件在界面的初始化中会调用apply方法，而在后面的更新界面中会调用reapply方法

## 源码分析

首先从setTextViewText方法切入：

```
    public void setTextViewText(int viewId, CharSequence text) {
        setCharSequence(viewId, "setText", text);
    }
```

继续跟进：

```
    public void setCharSequence(int viewId, String methodName, CharSequence value) {
        addAction(new ReflectionAction(viewId, methodName, ReflectionAction.CHAR_SEQUENCE, value));
    }
```

没有对view直接操作，但是添加了一个ReflectionAction，继续跟进：

```
    private void addAction(Action a) {
        if (hasLandscapeAndPortraitLayouts()) {
            throw new RuntimeException("RemoteViews specifying separate landscape and portrait" +
                    " layouts cannot be modified. Instead, fully configure the landscape and" +
                    " portrait layouts individually before constructing the combined layout.");
        }
        if (mActions == null) {
            mActions = new ArrayList<Action>();
        }
        mActions.add(a);

        // update the memory usage stats
        a.updateMemoryUsageEstimate(mMemoryUsageCounter);
    }
```

这里仅仅把每一个action存进list，似乎线索断掉了，这时候换一个切入点查看`updateAppWidget`方法，或者是`notificationManager.notify`因为跟新视图都要调用者两个方法

```
    public void updateAppWidget(int appWidgetId, RemoteViews views) {
        if (mService == null) {
            return;
        }
        updateAppWidget(new int[] { appWidgetId }, views);
    }
```

继续跟进：

```
    public void updateAppWidget(int[] appWidgetIds, RemoteViews views) {
        if (mService == null) {
            return;
        }
        try {
            mService.updateAppWidgetIds(mPackageName, appWidgetIds, views);
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
```

出现方法无法访问，这时我们思考，RemoteViews不是真正的view啊，所以是否可以去`AppWidgetHostView`看看，调转到updateAppWidget方法：

```
    public void updateAppWidget(RemoteViews remoteViews) {
        applyRemoteViews(remoteViews);
    }
```

继续跟进，好多代码：

```
   protected void applyRemoteViews(RemoteViews remoteViews) {
        if (LOGD) Log.d(TAG, "updateAppWidget called mOld=" + mOld);

        boolean recycled = false;
        View content = null;
        Exception exception = null;

//各种判断，省略
        if (remoteViews == null) {
            if (mViewMode == VIEW_MODE_DEFAULT) {
                // We've already done this -- nothing to do.
                return;
            }
            content = getDefaultView();
            mLayoutId = -1;
            mViewMode = VIEW_MODE_DEFAULT;
        } else {
            if (mAsyncExecutor != null) {
                inflateAsync(remoteViews);
                return;
            }
            // Prepare a local reference to the remote Context so we're ready to
            // inflate any requested LayoutParams.
            mRemoteContext = getRemoteContext();
            int layoutId = remoteViews.getLayoutId();

            // If our stale view has been prepared to match active, and the new
            // layout matches, try recycling it
            if (content == null && layoutId == mLayoutId) {
                try {
                    remoteViews.reapply(mContext, mView, mOnClickHandler);
                    content = mView;
                    recycled = true;
                    if (LOGD) Log.d(TAG, "was able to recycle existing layout");
                } catch (RuntimeException e) {
                    exception = e;
                }
            }

            // Try normal RemoteView inflation
            if (content == null) {
                try {
                    content = remoteViews.apply(mContext, this, mOnClickHandler);
                    if (LOGD) Log.d(TAG, "had to inflate new layout");
                } catch (RuntimeException e) {
                    exception = e;
                }
            }

            mLayoutId = layoutId;
            mViewMode = VIEW_MODE_CONTENT;
        }

        applyContent(content, recycled, exception);
        updateContentDescription(mInfo);
    }
```

这么多行代码，好像只有`remoteViews.reapply(mContext, mView, mOnClickHandler);`有点意思，和RemoteViews相关联了，那么跳转到RemoteViews的reapply方法：

```
    public void reapply(Context context, View v, OnClickHandler handler) {
        RemoteViews rvToApply = getRemoteViewsToApply(context);

        // In the case that a view has this RemoteViews applied in one orientation, is persisted
        // across orientation change, and has the RemoteViews re-applied in the new orientation,
        // we throw an exception, since the layouts may be completely unrelated.
        if (hasLandscapeAndPortraitLayouts()) {
            if ((Integer) v.getTag(R.id.widget_frame) != rvToApply.getLayoutId()) {
                throw new RuntimeException("Attempting to re-apply RemoteViews to a view that" +
                        " that does not share the same root layout id.");
            }
        }

        rvToApply.performApply(v, (ViewGroup) v.getParent(), handler);
    }
```

继续跟进：

```
    private void performApply(View v, ViewGroup parent, OnClickHandler handler) {
        if (mActions != null) {
            handler = handler == null ? DEFAULT_ON_CLICK_HANDLER : handler;
            final int count = mActions.size();
            for (int i = 0; i < count; i++) {
                Action a = mActions.get(i);
                a.apply(v, parent, handler);
            }
        }
    }
```

有点意思了，刚才我们说吧视图转换成action，现在终于看到了，由于action是抽象类，我们可以看看它子类的实现：

```
        @Override
        public void apply(View root, ViewGroup rootParent, OnClickHandler handler) {
            final View view = root.findViewById(viewId);
            if (view == null) return;

            Class<?> param = getParameterType();
            if (param == null) {
                throw new ActionException("bad type: " + this.type);
            }

            try {
                getMethod(view, this.methodName, param).invoke(view, wrapArg(this.value));
            } catch (ActionException e) {
                throw e;
            } catch (Exception ex) {
                throw new ActionException(ex);
            }
        }
```

终于看到反射调用改变内容的方法了

# RemoteViews遇上ListView

如果想要使用在桌面小部件中使用ListView，我们需要多实现一个服务继承自RemoteViewsService，这里我们举一个例子

## 布局文件medicinelist.xml

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:tools="http://schemas.android.com/tools"
    android:background="@drawable/widget_background"
    android:padding="@dimen/widget_margin">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_alignParentTop="true"
        android:background="@drawable/widget_background_half_top"
        android:orientation="horizontal"
        android:weightSum="10">

        <TextView
            android:id="@+id/appwidget_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            android:layout_gravity="center_vertical"
            android:layout_margin="8dp"
            android:layout_weight="8"
            android:paddingLeft="10dp"
            android:textColor="@color/titleTextColor"
            android:textSize="@dimen/text_title_largest" />

        <ImageView
            android:layout_marginRight="25dp"
            android:scaleType="fitCenter"
            android:id="@+id/editBtn"
            android:layout_width="0dp"
            android:layout_height="35dp"
            android:layout_gravity="center_vertical"
            android:layout_weight="2"
            android:src="@drawable/icon_edit" />
    </LinearLayout>

    <ImageView
        android:layout_marginTop="50dp"
        android:src="@color/divider"
        android:layout_width="match_parent"
        android:layout_height="1.5dp" />

    <ListView
        android:id="@+id/medicineList"
        android:divider="@color/divider"
        android:dividerHeight="1.5dp"
        android:layout_marginTop="51.5dp"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </ListView>

</RelativeLayout>
```

## 布局文件item.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginBottom="5dp"
    android:orientation="vertical"
    android:layout_marginLeft="@dimen/cardMarginLeft"
    android:layout_marginRight="@dimen/cardMarginRight"
    android:layout_marginTop="5dp"
    android:elevation="1px">

    <LinearLayout
        android:paddingLeft="@dimen/padding_left"
        android:paddingRight="@dimen/padding_right"
        android:paddingTop="@dimen/padding_top"
        android:paddingBottom="@dimen/padding_bottom"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:orientation="horizontal"
        android:weightSum="10">

        <TextView
            android:id="@+id/nextTime"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_gravity="center_vertical"
            android:layout_weight="4"
            android:gravity="center"
            android:text="12:23"
            android:textColor="@color/secondary_text"
            android:textSize="@dimen/text_title_largest" />

        <ImageView
            android:layout_width="@dimen/direction_size"
            android:layout_height="match_parent"
            android:layout_marginLeft="5dp"
            android:layout_marginRight="5dp"
            android:background="@color/divider" />

        <TextView
            android:id="@+id/showTag"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_gravity="center_vertical"
            android:layout_weight="7"
            android:gravity="center_vertical|left"
            android:lines="1"
            android:paddingLeft="5dp"
            android:text="治疗感冒"
            android:textColor="@color/secondary_text"
            android:textSize="@dimen/text_title_middle" />

    </LinearLayout>

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="@dimen/direction_size"
        android:background="@color/divider" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="5dp"
        android:orientation="horizontal">

        <ImageView
            android:id="@+id/showMedicineIcon"
            android:layout_width="150dp"
            android:layout_height="match_parent"
            android:layout_gravity="center_vertical"
            android:src="@drawable/medicine" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:paddingLeft="@dimen/padding_left"
            android:paddingTop="@dimen/padding_top">

            <TextView
                android:id="@+id/showMedicineName"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:gravity="center_horizontal"
                android:text="维他命"
                android:textColor="@color/black"
                android:textSize="@dimen/text_title_largest"
                android:textStyle="bold" />

            <TextView

                android:id="@+id/showMedicineUseType"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="5dp"
                android:gravity="center_horizontal"
                android:text="口服3.5粒"
                android:textColor="@color/black"
                android:textSize="@dimen/text_title_middle" />

            <TextView
                android:id="@+id/showMedicineLength"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="5dp"
                android:gravity="center_horizontal"
                android:text="还需服用3天"
                android:textColor="@color/black"
                android:textSize="@dimen/text_title_middle" />
        </LinearLayout>
    </LinearLayout>
</LinearLayout>
```

## 配置文件info.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:initialKeyguardLayout="@layout/app_widget_medicine_list"
    android:initialLayout="@layout/app_widget_medicine_list"
    android:minHeight="250dp"
    android:minWidth="250dp"
    android:previewImage="@drawable/example_appwidget_preview"
    android:resizeMode="horizontal|vertical"
    android:updatePeriodMillis="86400000"
    android:widgetCategory="home_screen|keyguard"></appwidget-provider>
```

## AndroidMainFest.xml

```xml
        <!-- 声明桌面小部件 -->
        <service
            android:name=".Service.UpdateService"
            android:exported="false"
            android:permission="android.permission.BIND_REMOTEVIEWS" />

        <receiver android:name=".MyViewPackage.AppWidgetMedicineList">
            <intent-filter>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
            </intent-filter>

            <meta-data
                android:name="android.appwidget.provider"
                android:resource="@xml/app_widget_medicine_list_info" />
        </receiver>
```

## AppWidgetList

```java
public class AppWidgetMedicineList extends AppWidgetProvider {

    static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                                int appWidgetId) {

        CharSequence widgetText = context.getString(R.string.appwidget_text);
        // Construct the RemoteViews object
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.app_widget_medicine_list);
        views.setTextViewText(R.id.appwidget_text, widgetText);
        Intent intent = new Intent(context, UpdateService.class);
        intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
        intent.setData(Uri.parse(intent.toUri(Intent.URI_INTENT_SCHEME)));
        views.setRemoteAdapter(R.id.medicineList, intent);
        // Instruct the widget manager to update the widget
        views.setOnClickPendingIntent(R.id.editBtn, PendingIntent.getActivity(context, 1,
                new Intent(context, MedicineActivity.class), 0));
        appWidgetManager.updateAppWidget(appWidgetId, views);
        appWidgetManager.notifyAppWidgetViewDataChanged(appWidgetId, R.id.medicineList);
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        // There may be multiple widgets active, so update all of them
        for (int appWidgetId : appWidgetIds) {
            updateAppWidget(context, appWidgetManager, appWidgetId);
        }
    }

    @Override
    public void onEnabled(Context context) {
        // Enter relevant functionality for when the first widget is created
    }

    @Override
    public void onDisabled(Context context) {
        // Enter relevant functionality for when the last widget is disabled
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
    }
}
```

## UpdateServices

```java
public class UpdateService extends RemoteViewsService {
    private static final String TAG = "UpdateService";

    public List<MedicineDetail> getMedicines(Context mContext) {
        MyDataBase dbHelper = new MyDataBase(mContext,
                "LocalStore.db", null, MyConstants.DATABASE_VERSION);
        List<MedicineDetail> temp = dbHelper.getMedicineDetail(
                SharedPreferenceHelper.getLoginUser().getObjectId());
        if (temp == null) {
            return new ArrayList<>();
        }
        return temp;
    }




    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        Log.d(TAG, "onGetViewFactory: Here is stable!");
        return new ListRemoteViewsFactory(this.getApplicationContext(), intent);
    }

    class ListRemoteViewsFactory implements RemoteViewsService.RemoteViewsFactory {

        private Context mContext;
        private List<MedicineDetail> medicineDetails;

        public ListRemoteViewsFactory(Context mContext, Intent intent) {
            this.mContext = mContext;
            if (Looper.myLooper() == null) {
                Looper.prepare();
            }
            if ((medicineDetails = getMedicines(mContext)) != null) {
                Log.d(TAG, "ListRemoteViewsFactory: 加载数据");
            }
        }

        @Override
        public void onCreate() {
            Log.d(TAG, "onCreate: 服务创建");
        }

        @Override
        public void onDataSetChanged() {
            Log.d(TAG, "onDataSetChanged: 设置数据");
        }

        @Override
        public void onDestroy() {
            medicineDetails.clear();
        }

        @Override
        public int getCount() {
            return medicineDetails.size();
        }

        @Override
        public RemoteViews getViewAt(int position) {
            if (position < 0 || position >= medicineDetails.size()) {
                return null;
            }
            Log.d(TAG, "getViewAt: position = " + position);
            MedicineDetail medicineDetail = medicineDetails.get(position);
            Log.d(TAG, "getViewAt: size = " + medicineDetails.size());
            RemoteViews remoteViews = new RemoteViews(mContext.getPackageName(), R.layout.item_medicine_widget);
            int nextTimePos = medicineDetail.getNextTime();
            remoteViews.setTextViewText(R.id.nextTime, medicineDetail.getTimes().get(nextTimePos));
            remoteViews.setTextViewText(R.id.showTag, medicineDetail.getTag());
            remoteViews.setTextViewText(R.id.showMedicineName, medicineDetail.getMedicineName());
            remoteViews.setTextViewText(R.id.showMedicineLength, "还需要服用\n" + medicineDetail.getDayLength() + "天");
            remoteViews.setTextViewText(R.id.showMedicineUseType, medicineDetail.getUseType()
                    + "\n"
                    + medicineDetail.getDoses().get(nextTimePos)
                    + medicineDetail.getUnit());
            Bitmap image = UtilClass.getBitmapFromGlide(mContext, medicineDetail.getMedicinePicture());
            if (image != null){
                remoteViews.setImageViewBitmap(R.id.showMedicineIcon, image);
                Log.d(TAG, "getViewAt: 这个操作耗时吗？");
            }
            return remoteViews;
        }

        @Override
        public RemoteViews getLoadingView() {
            return null;
        }

        @Override
        public int getViewTypeCount() {
            return 1;
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public boolean hasStableIds() {
            return true;
        }
    }
}
```

## 最后实现的结果

![最后结果](http://www.picb.cc/image/lFqS0)

# PS：需要注意的事情

## 1. 不要在xml文件中写入不支持的View，注意View本身是不被支持的，所以想要实现分割线，不如尝试用ImageView

## 2. 不要使用诸如`android:layout_height="?attr/actionBarSize"`这样的属性值，不然你的RemoteView会莫名其妙显示不出来，可能是因为？对于RemoteView是一个耗时操作吧



