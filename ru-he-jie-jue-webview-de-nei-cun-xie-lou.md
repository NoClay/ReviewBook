**原因：**WebView加载了大量的图片导致内存占用，再退出该界面后，即使在包含该WebView的Activity的destory方法中，对webView进行内存回收没有效果。

**解决：**

1. 不再xml文件中使用WebView，而是在代码中动态添加，比如在xml中使用一个LinearLayout作为WebView的父布局，在需要使用的时候动态添加。缺点：如果在WebView构造的时候传入ApplicationContext，当WebView想要弹出一个dialog，都会导致应用崩溃，如果传入ActivityContext的话，会导致对内存的引用无法回收。

   ```java
   protected void onDestroy() {
         super.onDestroy();
         mWebView.removeAllViews();
         mWebView.destroy()
   }
   ```

2. 使用单独的进程，将需要WebView的Activity单独一个进程，分配单独的虚拟机，利用IPC机制进行通信，同时与上一种结合起来，在准备销毁的时候，使用System.exit\(0\);强制退出虚拟机，即杀死当前新进程，即可实现。

   如果出现了闪屏可以利用给Activity设置Theme解决：

   ```
       <style name="ActivityTheme" parent="AppTheme">
           <item name="android:windowBackground">@color/white</item>
             <!--设置窗口透明-->
           <item name="android:windowIsTranslucent">true</item>
       </style>
   ```



