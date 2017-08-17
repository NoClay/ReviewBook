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

### 页面加载速度优化

影响页面加载速度的因素有非常多，我们在对 WebView 加载一个网页的过程进行调试发现，每次加载的过程中都会有较多的网络请求，除了 web 页面自身的 URL 请求，还会有 web 页面外部引用的JS、CSS、字体、图片等等都是个独立的 http 请求。这些请求都是串行的，这些请求加上浏览器的解析、渲染时间就会导致 WebView 整体加载时间变长，消耗的流量也对应的真多。接下来我们就来说说几种优化方案来是怎么解决这个问题的。

#### 选择合适的 WebView 缓存

WebView 缓存看似就是开启几个开关的问题，但是要弄懂这几种缓存机制还是很有深度。下图是腾讯某工程师总结六种 H5 常用的缓存机制的优势及适用场景。

![](https://user-gold-cdn.xitu.io/2016/11/29/77ba511beb2f988e7504a118a89c05e3.png?imageView2/0/w/1280/h/960 "图片来自Bugly")

##### 浏览器缓存机制：

主要前端负责，Android 端不需要进行特别的配置。

##### Dom Storage（Web Storage）存储机制：

配合前端使用，使用时需要打开 DomStorage 开关。

```
WebView myWebView = (WebView) findViewById(R.id.webview);
WebSettings webSettings = myWebView.getSettings();
webSettings.setDomStorageEnabled(
true
);
```

##### Web SQL Database 存储机制：

虽然已经不推荐使用了，但是为了兼容性，还是提供下 Android 端使用的方法

```
WebView myWebView = (WebView) findViewById(R.id.webview);
WebSettings webSettings = myWebView.getSettings();
webSettings.setDatabaseEnabled(
true
);

final
String
 dbPath = getApplicationContext().getDir(
"db"
,Context.MODE_PRIVATE).getPath();
webSettings.setDatabasePath(dbPath)
```

##### Application Cache 存储机制

Application Cache（简称 AppCache\)似乎是为支持 Web App 离线使用而开发的缓存机制。它的缓存机制类似于浏览器的缓存（Cache-Control 和 Last-Modified）机制，都是以文件为单位进行缓存，且文件有一定更新机制。但 AppCache 是对浏览器缓存机制的补充，不是替代。

不过根据官方文档，AppCache 已经不推荐使用了，标准也不会再支持。现在主流的浏览器都是还支持 AppCache的，以后就不太确定了。同样给出 Android 端启用 AppCache 的代码。

```
WebView myWebView = (WebView) findViewById(R.id.webview);
WebSettings webSettings = myWebView.getSettings();
webSettings.setAppCacheEnabled(
true
);

final
String
 cachePath = getApplicationContext().getDir(
"cache"
,Context.MODE_PRIVATE).getPath();
webSettings.setAppCachePath(cachePath);
webSettings.setAppCacheMaxSize(
5
*
1024
*
1024
);
```

##### Indexed Database 存储机制

IndexedDB 也是一种数据库的存储机制，但不同于已经不再支持的 Web SQL Database。IndexedDB 不是传统的关系数据库，可归为 NoSQL 数据库。IndexedDB 又类似于 Dom Storage 的 key-value 的存储方式，但功能更强大，且存储空间更大。

Android 在4.4开始加入对 IndexedDB 的支持，只需打开允许 JS 执行的开关就好了。

```
WebView myWebView = (WebView) findViewById(R.id.webview);
WebSettings webSettings = myWebView.getSettings();
webSettings.setJavaScriptEnabled(
true
);
```

##### File System API

File System API 是 H5 新加入的存储机制。它为 Web App 提供了一个虚拟的文件系统，就像 Native App 访问本地文件系统一样。由于安全性的考虑，这个虚拟文件系统有一定的限制。Web App 在虚拟的文件系统中，可以进行文件（夹）的创建、读、写、删除、遍历等操作。很可惜到目前，Android 系统的 WebView 还不支持 File System API。

> 简单的介绍完了上面六种 H5 常用的缓存模式，想必大家能对 Android WebView 所支持的缓存模式有个粗略的了解。如果想和前端更好的配合使用 Android WebView 所支持的缓存，建议看下这篇文章[《H5 缓存机制浅析 移动端 Web 加载性能优化》](https://link.juejin.im/?target=http%3A%2F%2Fbugly.qq.com%2Fbbs%2Fforum.php%3Fmod%3Dviewthread%26tid%3D267)

#### 常用资源预加载

上面介绍的缓存技术，能优化二次启动 WebView 的加载速度，那首次加载 H5 页面的速度该怎么优化呢？上面分析了一次加载过程会有许多外部依赖的 JS、CSS、图片等资源需要下载，那我们能不能提前将这些资源下载好，等H5 加载时直接替换呢？

好在从 API 11（Android 3.0）开始，WebView 引入了 shouldInterceptRequest 函数，这个函数有两种重载。

* **public WebResourceResponse shouldInterceptRequest\(WebView webView, String url\)**
  从 API 11 引入，API 21 废弃
* **public WebResourceResponse shouldInterceptRequest \(WebView view, WebResourceRequest request\)**
  从 API 21 开始引入

考虑到目前大多数 App 还要支持 API 14，所以还是使用 shouldInterceptRequest \(WebView view, String url\) 为例。

```
WebView mWebView = (WebView) findViewById(R.id.webview);
mWebView.setWebViewClient(
new
 WebViewClient() {
            @Override

public
 WebResourceResponse shouldInterceptRequest(WebView webView, final 
String
 url) {
                WebResourceResponse response = 
null
;

boolean
 resDown = JSHelper.isURLDownValid(url);

if
 (resDown) {
                    jsStr = JsjjJSHelper.getResInputStream(url);

if
 (url.endsWith(
".png"
)) {
                        response = getWebResourceResponse(url, 
"image/png"
, 
".png"
);
                    } 
else
if
 (url.endsWith(
".gif"
)) {
                        response = getWebResourceResponse(url, 
"image/gif"
, 
".gif"
);
                    } 
else
if
 (url.endsWith(
".jpg"
)) {
                        response = getWebResourceResponse(url, 
"image/jepg"
, 
".jpg"
);
                    } 
else
if
 (url.endsWith(
".jepg"
)) {
                        response = getWebResourceResponse(url, 
"image/jepg"
, 
".jepg"
);
                    } 
else
if
 (url.endsWith(
".js"
) 
&
&
 jsStr != 
null
) {
                        response = getWebResourceResponse(
"text/javascript"
, 
"UTF-8"
, 
".js"
);
                    } 
else
if
 (url.endsWith(
".css"
) 
&
&
 jsStr != 
null
) {
                        response = getWebResourceResponse(
"text/css"
, 
"UTF-8"
, 
".css"
);
                    } 
else
if
 (url.endsWith(
".html"
) 
&
&
 jsStr != 
null
) {
                        response = getWebResourceResponse(
"text/html"
, 
"UTF-8"
, 
".html"
);
                    }
                }

return
 response;
            }
        });

private
 WebResourceResponse getWebResourceResponse(
String
 url, 
String
 mime, 
String
 style) {
        WebResourceResponse response = 
null
;

try
 {
            response = 
new
 WebResourceResponse(mime, 
"UTF-8"
, 
new
 FileInputStream(
new
 File(getJSPath() + TPMD5.md5String(url) + style)));
        } 
catch
 (FileNotFoundException e) {
            e.printStackTrace();
        }

return
 response;
    }

public
String
 getJsjjJSPath() {

String
 splashTargetPath = JarEnv.sApplicationContext.getFilesDir().getPath() + 
"/JS"
;

if
 (!TPFileSysUtil.isDirFileExist(splashTargetPath)) {
            TPFileSysUtil.createDir(splashTargetPath);
        }

return
 splashTargetPath + 
"/"
;
    }
```

#### 常用 JS 本地化及延迟加载

比预加载更粗暴的优化方法是直接将常用的 JS 脚本本地化，直接打包放入 apk 中。比如 H5 页面获取用户信息，设置标题等通用方法，就可以直接写入一个 JS 文件，放入 asserts 文件夹，在 WebView 调用了onPageFinished\(\) 方法后进行加载。需要注意的是，在该 JS 文件中需要写入一个 JS 文件载入完毕的事件，这样前端才能接受都爱 JS 文件已经种植完毕，可以调用 JS 中的方法了。 附上一段本地化的 JS 代码。

```
javascript: ;
(function() {
  try{
    window.JSBridge = {
      'invoke': function(name) {
        var args = [].slice.call(arguments, 1),
          callback = args.pop(),
          params, obj = this[name];
        if (typeof callback !== 'function') {
          params = callback;
          callback = function() {}
        } else {
          params = args[0]
        } if (typeof obj !== 'object' || typeof obj.func !== 'function') {
          callback({
            'err_msg': 'system:function_not_exist'
          });
          return
        }
        obj.callback = callback;
        obj.params = params;
        obj.func(params)
      },
      'on': function(event, callback) {
        var obj = this['on' + event];
        if (typeof obj !== 'object') {
          callback({
            'err_msg': 'system:function_not_exist'
          });
          retrun
        }
        if (typeof callback !== 'undefined') obj.callback = callback
      },
      'login': {
        'func': function(params) {
          prompt("login", JSON.stringify(params))
        },
        'params': {},
        'callback': function(res) {}
      },
      'settitle': {
        'func': function(params) {
          prompt("settitle",JSON.stringify(params))
        },
        'params': {},
        'callback': function(res) {}
      },
    }catch(e){
    alert('demo.js error:'+e);
  }
  var readyEvent = document.createEvent('Events');
  readyEvent.initEvent('JSBridgeReady', true, true);
  document.dispatchEvent(readyEvent)
  })();javascript: ;
(function() {
  try{
    window.JSBridge = {
      'invoke': function(name) {
        var args = [].slice.call(arguments, 1),
          callback = args.pop(),
          params, obj = this[name];
        if (typeof callback !== 'function') {
          params = callback;
          callback = function() {}
        } else {
          params = args[0]
        } if (typeof obj !== 'object' || typeof obj.func !== 'function') {
          callback({
            'err_msg': 'system:function_not_exist'
          });
          return
        }
        obj.callback = callback;
        obj.params = params;
        obj.func(params)
      },
      'on': function(event, callback) {
        var obj = this['on' + event];
        if (typeof obj !== 'object') {
          callback({
            'err_msg': 'system:function_not_exist'
          });
          retrun
        }
        if (typeof callback !== 'undefined') obj.callback = callback
      },
      'login': {
        'func': function(params) {
          prompt("login", JSON.stringify(params))
        },
        'params': {},
        'callback': function(res) {}
      },
      'settitle': {
        'func': function(params) {
          prompt("settitle",JSON.stringify(params))
        },
        'params': {},
        'callback': function(res) {}
      },
    }catch(e){
    alert('demo.js error:'+e);
  }
  var readyEvent = document.createEvent('Events');
  readyEvent.initEvent('JSBridgeReady', true, true);
  document.dispatchEvent(readyEvent)
  })();
```

**关于 JS 延迟加载**

> Android 的 OnPageFinished 事件会在 Javascript 脚本执行完成之后才会触发。如果在页面中使 用JQuery，会在处理完 DOM 对象，执行完**$\(document\).ready\(function\(\) {}\);**事件自会后才会渲染并显示页面。而同样的页面在 iPhone 上却是载入相当的快，因为 iPhone 是显示完页面才会触发脚本的执行。所以我们这边的解决方案延迟 JS 脚本的载入，这个方面的问题是需要Web前端工程师帮忙优化的。

### 使用第三方 WebView 内核

WebView 的兼容性一直也是困扰我们 Android 开发者的一个大问题，不说 Android 4.4 版本 Google 使用了Chromium 替代 Webkit 作为 WebView 内核，就看看国内众多的第三方 ROM 都有可能会对原生的 WebView 做出修改，这时候如果出现兼容问题，是非常难定位到问题和解决的。

在一次使用微信浏览订阅公众号文章的过程中，发现微信的 H5 页面有一行 『QQ 浏览器 X5 内核提供技术支持』。顺着这个线索我就找到了[腾讯浏览服务](https://link.juejin.im/?target=http%3A%2F%2Fx5.tencent.com%2Findex)。发现腾讯已经把这个功能开放了，而且集成的 SDK 很小只有212 KB。这是很惊人的，通过介绍才发现这个 SDK 是可以共享微信和手机 QQ 的 X5 内核。这就很方便了，作为国内市场最不可或缺的两个 App，我们能只需要集成一个很小的 SDK 就可以共享使用 X5 内核了，不得不说腾讯还是很有想法的。

简单摘录些功能亮点，想必能让大家高潮一番。详细内容大家可以直接到[腾讯浏览服务](https://link.juejin.im/?target=http%3A%2F%2Fx5.tencent.com%2Findex)看看，我相信不会让你们失望的。

> **网页浏览能力**
>
> Web页面crash率降低75%
>
> 页面打开速度提升35%
>
> 流量节省60%
>
> **阅读模式**
>
> 去除网页中广告等杂质
>
> 优化文章的阅读体验
>
> **文件打开能力**
>
> 包括会话页的互传文件及邮件中附件
>
> 支持doc、ppt、xls、pdf等办公格式
>
> 支持jpg、gif、png、bmp等图片格式
>
> 支持zip、rar等压缩文件
>
> 支持mp3、mp4、RMVB等音视频格式
>
> **视频菜单能力**
>
> 支持屏幕调节等常规视频菜单功能
>
> 灵活切换全屏&小窗功能

### WebView 导致的内存泄露

> Android 中的 WebView 存在很大的兼容性问题，不仅仅是 Android 系统版本的不同对 WebView 产生很大的差异，另外不同的厂商出货的 ROM 里面 WebView 也存在着很大的差异。更严重的是标准的 WebView 存在内存泄露的问题，看这里[WebView causes memory leak - leaks the parent Activity](https://link.juejin.im/?target=https%3A%2F%2Fcode.google.com%2Fp%2Fandroid%2Fissues%2Fdetail%3Fid%3D5067)。所以通常根治这个问题的办法是为 WebView 开启另外一个进程，通过 AIDL 与主进程进行通信，WebView 所在的进程可以根据业务的需要选择合适的时机进行销毁，从而达到内存的完整释放。

这段话来自[胡凯](https://link.juejin.im/?target=http%3A%2F%2Fhukai.me%2Fandroid-performance-oom%2F)翻译的 Google Android 内存优化之 OOM 。这里提到的让 WebView 独立运行在一个进程里，用完 WebView 后直接销毁这个进程，即使内存泄露了，也不会影响到主进程。微信，手 Q 等 App 也采用了这个方案。但是这就涉及到了跨进程通讯，处理起来就比较麻烦。

另外个解决方案，就是使用自己封装的 WebView，比如上面提到的 X5 内核，且使用 WebView 的时候，不在 XML 里面声明，而是在代码中直接 new 出来，传入 application context 来防止 activity 引用被滥用。

```
WebView webView =  
new
 WebView(getContext().getApplicationContext());
webFrameLayout.addView(webView, 
0
);
```



