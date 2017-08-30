# 热修复技术

APP提早发出去的包，如果出现客户端的问题，实在是干着急，覆水难收。因此线上修复方案迫在眉睫。

# 概述

基于Xposed中的思想，通过修改c层的Method实例描述，来实现更改与之对应的java方法的行为，从而达到修复的目的。阿里的基于C/C++层操控method指针的Dexposed,AndFix,以及QQ空间的基于dex分包的HotFix，后者和前者的热修复方案在原理上截然不同，可以说各有千秋。而我在查阅资料的时候，发现很多Blog都不够严谨，往往标题声称热修复技术但是只解释QQ空间的解决方案，可以说这种做法是容易误导人的，虽然不能算错误，但是不太严谨。

# 1.Xposed

诞生于XDA论坛，类似一个应用平台，不同的是其提供诸多系统级的应用。可实现许多神奇的功能。Xposed需要以越狱为前提，像是iOS中的cydia。

Xposed可以修改任何程序的任何java方法（需root），github上提供了XposedInstaller，是一个android app。提供很多framework层，应用层级的程序。开发者可以为其开发一些系统或应用方面的插件，自定义android系统，它甚至可以做动态权限管理（XposedMods）。

## Android系统启动与应用启动

Zygote进程是Android手机系统启动后，常驻的一个名为‘受精卵’的进程。

* zygote的启动实现脚本在/init.rc文件中
* 启动过程中执行的二进制文件在/system/bin/app\_process

任何应用程序启动时，会从zygote进程fork出一个新的进程。并装载一些必要的class，invoke一些初始化方法。这其中包括像：

* ActivityThread
* ServiceThread
* ApplicationPackageManager

等应用启动中必要的类，触发必要的方法，比如：handleBindApplication，将此进程与对应的应用绑定的初始化方法；同时，会将zygote进程中的dalvik虚拟机实例复制一份，因此每个应用程序进程都有自己的dalvik虚拟机实例；会将已有Java运行时加载到进程中；会注册一些android核心类的jni方法到虚拟机中，支撑从c到java的启动过程。

## Xposed做了手脚

Xposed在这个过程改写了app\_process\(源码在Xposed : a modified app\_process binary\)，替换/system/bin/app\_process这个二进制文件。然后做了两个事：

1. 通过Xposed的hook技术，在上述过程中，对上面提到的那些加载的类的方法hook。
2. 加载XposedBridge.jar

这时hook必要的方法是为了方便开发者为它开发插件，加载XposedBridge.jar是为动态hook提供了基础。在这个时候加载它意味着，所有的程序在启动时，都可以加载这个jar（因为上面提到的fork过程）。结合hook技术，从而达到了控制所有程序的所有方法。

为获得/system/bin/目录的读写权限，因而需要以root为前提。

## Xposed的hook思想

那么Xposed是怎么hook java方法的呢？要从XposedBridge看起，重点在 XposedBridge.hookmethod\(原方法的Member对象，含有新方法的XC\_MethodHook对象\)；，这里会调到

```
private native synchronized static void hookMethodNative(Member method, Class<?> declaringClass, int slot, Object additionalInfo);
```

这个native的方法，通过这个方法，可以让所hook的方法，转向native层的一个c方法。如何做到？

```
When a transmit from java to native occurs, dvm sets up a native stack.
In dvmCallJNIMethod(), dvmPlatformInvoke is used to call the native method(signature in Method.insns).
```

在jni这个中间世界里，类型数据由jni表来沟通java和c的世界；方法由c++指针结合DVM\*系\(如dvmSlotToMethod,dvmDecodeIndirectRef等方法\)的api方法，操作虚拟机，从而实现java方法与c方法的世界。

那么hook的过程是这样：首先通过dexclassload来load所要hook的方法，分析类后，进c层，见代码XposedBridge\_hookMethodNative方法，拿到要hook的Method类，然后通过dvmslotTomethod方法获取Method\*指针，

```
Method* method = dvmSlotToMethod(declaredClass, slot);
```

declaredClass就是所hook方法所在的类，对应的jobject。slot是Method类中，描述此java对象在vm中的索引；那么通过这个方法，我们就获取了c层的Method指针,通过

```
SET_METHOD_FLAG(method, ACC_NATIVE);
```

将该方法标记为一个native方法，然后通过

```
method->nativeFunc = &hookedMethodCallback;
```

定向c层方法到hookedMethodCallback，这样当被hook的java方法执行时，就会调到c层的hookedMethodCallback方法。

通过meth-&gt;nativeFunc重定向MethodCallBridge到hookedMethodCallback这个方法上，控制这个c++指针是无视java的private的。

另外，在method结构体中有

```
method->insns = (const u2*) hookInfo;
```

用insns指向替换成为的方法，以便hookedMethodCallback可以获取真正期望执行的java方法。

现在所有被hook的方法，都指向了hookedMethodCallbackc方法中，然后在此方法中实现调用替换成为的java方法。

## 从Xposed提炼精髓

回顾Xposed，以root为必要条件，在app\_process加载XposedBidge.jar，从而实现有hook所有应用的所有方法的能力；而后续动态hook应用内的方法，其实只是load了从zypote进程复制出来的运行时的这个XposedBidge.jar，然后hook而已。因此，若在一个应用范围内的hook，root不是必须的，只是单纯的加载hook的实现方法，即可修改本应用的方法。

业界内也不乏通过「修改BaseDexClassLoader中的pathList，来动态加载dex」方式实现热修复。后者纯java实现，但需要hack类的优化流程，将打CLASS\_ISPREVERIFIED标签的类，去除此标签，以解决类与类引用不在一个dex中的异常问题。这会放弃dex optimize对启动运行速度的优化。原则上，这对于方法数没有大到需要multidex的应用，损失更明显。而前者不触犯原有的优化流程，只点杀需要hook的方法，更为纯粹、有效。

# 2.基于C层指针替换的Dexposed和AndFix

这两个热修复的框架，在底层原理上是基本一致的，所以我想把他们放在一起探讨，

他们都做了大致三件事：  
1，在C/C++层将Java层中出问题的方法修改为native方法  
2，获取问题方法call到C层的指针  
3，通过获取的指针做相应的操作：调用Java层的回调方法继续处理（DexPosed）或者直接通过反射调用Java层的补丁方法\(AndFix\)。

以Dexposed为例：![](http://upload-images.jianshu.io/upload_images/689802-e28a1a6ae68627f1.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

至于具体的代码解释，请直接看[Android中免Root实现Hook的Dexposed框架实现原理解析以及如何实现应用的热修复](http://gold.xitu.io/entry/5730a04f79df540060cea838)

这两种热修复框架的区别在于：

* Dexposed暂时不支持ART模式，AndFix支持
* AndFix方案更加成熟，更加自动化（毕竟是支付宝出的）

# 3.基于Dex分包的HotFix

这个解决方案很巧妙，基于Google推出的的Multidex方案，以ClassLoader的方式完成问题类的替换。所以这个问题一定会先谈Android的分包方案：为了解决Android4.x系统中65536的方法数限制，Android推出Multidex方案，将一个完整的APK中的Dex拆分成好几个dex，通过PathClassLoader 这个加载器来加载。

当点开程序的时候，PathClassLoader 会把分包的多个dex添加到父类中的一个DexPathList 中

DexPathList 详情如下：

```
public class BaseDexClassLoader extends ClassLoader {

    private final DexPathList pathList;
｝
```

```
/*package*/ final class DexPathList {
    private static final String DEX_SUFFIX = ".dex";
    private static final String JAR_SUFFIX = ".jar";
    private static final String ZIP_SUFFIX = ".zip";
    private static final String APK_SUFFIX = ".apk";

    /** class definition context */
    private final ClassLoader definingContext;

    /** list of dex/resource (class path) elements  也就是dex列表咯*/
    private final Element[] dexElements;

    /** list of native library directory elements */
    private final File[] nativeLibraryDirectories;
```

那么当需要加载某个类的时候，是怎么加载的呢？

```
//BaseDexClassLoader:  
    @Override  
    protected Class< ?> findClass(String name) throws ClassNotFoundException {  
        List<Throwable> suppressedExceptions = new ArrayList<Throwable>(); 
        Class c = pathList.findClass(name, suppressedExceptions);  
        if (c == null) {  
            ClassNotFoundException cnfe = new ClassNotFoundException("Didn't find class \"" + name + "\" on path: " + pathList);  
            for (Throwable t : suppressedExceptions) {  
                cnfe.addSuppressed(t);  
           }  
        throw cnfe;  
        }  
        return c;  
    }
```

findClass\(\)方法如下：

```
 public Class findClass(String name, List<Throwable>suppressed) {      
         for (Element element : dexElements) {  
           DexFile dex = element.dexFile;  
            if (dex != null) {  
                Class clazz = dex.loadClassBinaryName(name, definingContext, suppressed);  
                if (clazz != null) {  
                    return clazz;  
                }  
            }  
       }  
        if (dexElementsSuppressedExceptions != null) {  
            suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));  
        }  
        return null;  
    }
```

如你所见，**当需要加载一个类的时候，会在pathList中去寻找，并且是通过顺序遍历各个dex包的方式，一旦找到目标类，则停止遍历**。

![](http://upload-images.jianshu.io/upload_images/689802-1692057ea5279167.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

这就给了我们一个想法，有没有可能把打了补丁的dex插入到pathList中，当需要加载有问题的类的时候，根据遍历，首先查到已经修复的类，遍历结束，也就完成了修复。（当然了，这个想法是腾讯空间Android工程师想到的）

有了想法，也得有合适的加载器啊。结果你猜怎么着？Android还真提供了这样的机会。

在Android中也有三个类加载器，分别是UrlClassLoader,PathClassLoader,DexClassLoader.

* UrlClassLoader 从Url列表中加载相关的jar文件，但是dalvik无法直接识别jar，so.....

* PathClassLoader 它只会去读取 /data/dalvik-cache 目录下的 dex 文件，就是已安装的apk,

* DexClassLoader 可以用来从.jar和.apk类型的文件内部加载classes、dex文件。而且，它和PathClassLoader继承自共同的父类。显然，这是最合适的加载器。  
  ![](http://upload-images.jianshu.io/upload_images/689802-a15c1c2fccc5d87c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

* 如何防止自己的类被打上 CLASS\_ISPREVERIFIED标志  
  这个标志是虚拟机的一种优化手段，打上这个标志之后，就不会引用其他dex中的类，如果引用了，则报错。解决方案也很简单，就是在类中引用其他dex包的引用，具体方法请直接Google。



