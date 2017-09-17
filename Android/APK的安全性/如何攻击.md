# APK的安全性

可以通过修改客户端文件篡改客户端行为。攻击者可以让客户端显示自己制作的钓鱼网站，偷取用户信息

以下攻击部分摘自知乎：

https://www.zhihu.com/question/41368839/answer/112182560

防御部分摘自如下博文：

http://blog.csdn.net/jiangwei0910410003/article/details/48415225

http://www.cnblogs.com/ijiami/archive/2013/08/27/3284209.html

# 如何攻击

##工具

工欲善其事，必先利其器。首先准备好工具：

**反编译工具**

1，  **apktool**反编译利器

2，  **dex2jar**将dex文件反编译成jar文件（java代码）工具，用于解读代码

3，  **gui **打开jar文件工具

**签名工具**

1， **apksign**给java程序签名的工具

2，  **testkey.pk8 teskkey.x509.pem**用于签名的文件

首先下载好apk

![](https://pic2.zhimg.com/ca7b6d5101848e064d50cc63426a0269_b.png)

##一，用ApkTool反编译android程序

用apktool反编译，命令如下：

```java
apktool.bat d test.apk
...
  
```

成功后会在同级目录生成一个test文件夹

![https://pic4.zhimg.com/7d2a8a7eaf2df76816507bc53749721b_b.png](https://pic4.zhimg.com/7d2a8a7eaf2df76816507bc53749721b_b.png)

这就是反编译之后的android程序了，可以看出，这个目录结构跟我们编写android代码时的目录结构非常相似，除了java代码是以smali的格式呈现之外，其他都基本是原来的代码。其实有很多人抄界面，到这一步就可以抄出完整的界面了。如manifest文件，里面的Activity定义都可以看的很清楚了。然后layout文件，各种res都可以看见了。

![](https://pic4.zhimg.com/6844ea56cfcd5d2e9f5b2074ccae7d87_b.png)

其实写到这，我就有个问题了，这一步怎么防？我不知道，愿请教一二。
如果我们要参考（chao）一个程序的界面，到这一步已经够了，以为所有的res和layout文件已经能看到了。
改代码重新编译也是要在这个文件夹中改smali文件的，所以smali的语法还是要熟悉一点。但是看代码逻辑我们不用去看晦涩难懂的smali语言，这就是下一步要做的工作。反编译出java代码。

##二、用dex2jar反编译出java源代码

第一步做的工作先放在这，我们需要重新操作apk文件，其实apk文件就是一种压缩包，所以我们把后缀名改成rar，用解压缩工具打开。

![](https://pic2.zhimg.com/0cc84c7c48c658d26b2962291a2174ed_b.png)

看到这里，有人会问，为什么不直接解压缩，跟我们刚才用apktool反编译出来的不一样吗，你可以试一下。

这里其他文件在apktool那一步已经反编译出来了，我们需要的仅仅是class文件，这是java代码编译后生成的文件，用dex2jar这个工具就可以反编译出原代码（java格式）了。把这个class文件解压出来，放在dex2jar的同级目录下。

```
d2j-dex2jar.bat test.dex
```

命令如上，成功之后就会在同级目录下生成jar文件了。

##三、用gui查看代码

还记得一开始我们说过的工具gui，通过gui打开jar文件，就能看到java代码了

![](https://pic3.zhimg.com/28abd6abc1e39c2bc99071498a831bc6_b.png)

这里所有的引入的包代码都会有，那么怎么寻找我们要的主程序代码呢，这就要依赖在第一步我们反编译出的manifest文件，熟悉android的朋友知道，在manifest文件中有两个信息比较重要。
一是包名，也就是主程序的路径，在manifest的最开始一行。

![](https://pic2.zhimg.com/a48eade665b3c3f60b87ca64a35295f5_b.png)

第二个信息是入口activity，这个很简单，只要找到有launcher标识的activity就是入口activity。

![](https://pic1.zhimg.com/2c4114ec7bfbb950ffe6b021526ed494_b.png)

现在你就可以去gui里面找到这个入口类了

![](https://pic3.zhimg.com/0139fc2bd7060553aa2637353df5eeca_b.png)

代码有混淆，但是混淆只是替换了一些变量名或者类名而已，增加了代码阅读的困难性，并不会修改程序逻辑本身，所以只要静下心来慢慢看，还是看到懂得。

至此，反编译的过程就结束了，你想看到一个程序的逻辑或者一个程序的界面逻辑都可以看的到的。

##四、重新打包，签名，运行

下面，开始进行最重要的工作，修改代码，二次打包。其实这里你可以什么代码先都别改，只重新打包一次，看看程序是否能够正常运行，如果不能，看看程序是哪一步阻止了运行，这也方便你后期定位签名验证的位置。目前我见过的签名验证有以下几种：

- 直接抛出异常，禁止运行
- 弹出提示框提示用户，提示框消失后，退出程序
- 跟服务器交互传递签名信息，如果不正确则服务器不返回数据

重新打包是这样的，还要用到apktool，记得在第一步反编译出的那个文件夹吗，就是用这些文件再重新打包。打包命令如下：

```java
apktool b test -o test1.apk
```

成功后，在同级目录下会看到test1.apk文件，这里只是打包成功了，程序还没有签名，没有签名的程序是无法安装到手机上的。签名用的的是apksign这个工具，这是java提供给开发者用于程序签名的工具，android的各类IDE也是用这个工具在签名。使用方法如下，将signapk.jar，testkey.pk8，testkey.x509.pem放在一个目录下，写一个signapk.bat文件，如下

```
java -jar signapk.jar testkey.x509.pem testkey.pk8 %1 %2

```

然后运行命令

![](https://pic1.zhimg.com/571290f4f0b2246af5551a13ed6c6d38_b.png)

成功后会在同级目录下生成一个签过名的apk文件，这个文件我们需要的最终文件，只要你改过代码并且签完名后这个apk可以正常安装运行，那么本次的任务就算完成了。现在安装一下，看看会发生什么。

程序启动，然后弹出提示框

![](https://pic3.zhimg.com/464cc8da74a8c9f6dc66fbc14edeaa82_b.png)

程序弹出提示，点击确认后退出程序，看来这个app的签名验证是用了我说的上面第二种方法，下面来进行一些尝试来绕过这个签名验证。

##**五、绕过程序防二次打包机制**

首先，我**建议大家先全局搜一下signatures这个字符串，因为程序要获取app的签名就要通过packageInfo.signatures这种方式，如果在这里我们不让程序获取到真正的签名，而是直接返回给它那个“正确”的签名，岂不是瞒天过海，一步搞定。当然了，你必须要有原来那个程序的“正确”签名**，不过这个简单，android系统并不阻止你去获取其他程序的签名，所以我们可以写个小的test程序，然后安装原来的apk，去获取一次正确的签名，记录下来。

获取其他程序签名代码如下

```
private static String getSignture(Application paramApplication) {
    try {
        String packageName = "packageName";
        List<PackageInfo> packages = paramApplication.getPackageManager().getInstalledPackages(PackageManager.GET_SIGNATURES);
        for (PackageInfo packageInfo : packages) {
            Signature[] signs = packageInfo.signatures;
            Signature sign = signs[0];
            String signString = sign.toCharsString();
            System.out.println(signString);
            return signString;
        }
    } catch (Exception e) {
        return "";
    }
    return "";
}

```

先装上原来从正常渠道下载的程序，然后改一下包名，运行这个程序，就能得到正确程序的正确签名了，记录一下签名，然后去我们反编译的代码里面找signatures相关的代码，看在哪里获取了签名并验证。

![](https://pic1.zhimg.com/54c0feec5cb2402ae06010630cf0a0a4_b.png)



是不是跟我写的那个方法完全一样，这个方法其实是获取程序的本来的签名的，这就好说了，我们直接返回刚才记录的“正确”签名就可以瞒过程序了。![img](https://pic4.zhimg.com/e8360db4cbebf20ae728b98cc8ba9e83_b.png)

好，第一次尝试，去apktool反编译出的文件中的smali文件夹下找到这个类MainActivity，如下

![](https://pic2.zhimg.com/aa1d1b77c476b5152cca20775ecffa01_b.png)

这是smali的语法，挺复杂的，感兴趣的朋友可以自己再翻阅一下资料。这里我们把这个方法全部注掉，直接返回“正确”的签名。如下

![](https://pic3.zhimg.com/669685cfc92b8321babe719e778f1c2a_b.png)

按照前面说的签名的方法，重新打包，签名，安装。

我们会发现，程序第一次进入是不行的，还是会提示，签名验证失败，第二次之后就可以正常进入了，这不是我们要的完美效果，思考一下，为什么会有这个情况，我想到以下几种原因：

- 第一次的时候signinfo还没有获取，为空，所以认为是非法的
- 除了这里，程序在另外的地方做了二次验证，而且这个二次验证并不一定每次都能执行成功，这个很像是一个网络请求方法，跟服务器做验证，所以根据网络情况，并不一定每次都成功。

如果是第一种情况，为什么正常的程序没有问题，我们就只是让返回值变了一下，其他并没有改变逻辑。我推测是时间差，因为原来的方法执行获取签名需要较长的时间，而直接返回正确签名很快，难道是这个时间差的影响？我决定把原来那个方法改回来，只修改返回值。如下：

![](https://pic1.zhimg.com/0e52a0ccad38bfcae4d6b849ae14f5bc_b.png)

只修改返回值，原来的逻辑不变，时间差应该也排除了，重新打包签名运行。好吧，很明显不是，而且情况更严重了，前面这些只是我的经验之谈，你在完全不了解逻辑的情况下，可以这样先试一下，我想能绕过30%的app吧。如果是上面说的第二种情况，我们还是来看一下代码逻辑吧。

全局搜一下应用签名验证失败这句话，看看什么情况下会触发。

![](https://pic2.zhimg.com/64d20284a59fdd051065d0c95fd5e8d5_b.png)

一共有两处，我们先看第一处

![](https://pic3.zhimg.com/f17bcd65b7d7b7833d0f4b1027ed8be6_b.png)

其实混淆后的代码挺恶心的，你看这个逻辑好像是如果LoginActivity的c方法为null就执行，但是你去看c方法就会发现根本就没有返回值，稳稳的null。这里代码其实是这样看的，要跳出前面那个while，所以我们去loginActivity找what值是19的情况。

![](https://pic3.zhimg.com/c6bc57cbf39a5318ac9314416beba9b2_b.png)

往前看，可以发现他调用了一个方法

![](https://pic1.zhimg.com/1be57e9d3b3d6c16fa77d7921470fc14_b.png)

看来验证应该是在这里了，而且是一个网络请求验证，所以这个app的防二次打包的机制已经做的比较好的。研究下这个方法，混淆代码不是很容易看，我先用抓包工具抓了一下包。

发现程序在启动的时候发了两个用来验证的请求，第一个请求没有参数，服务器返回如下字段

![](https://pic2.zhimg.com/cc2f258e334c83bd8c2c840a79c63709_b.png)
第二次请求带有如下参数

![](https://pic1.zhimg.com/6ded05461a4f28b77a1a02eb841babb4_b.png)

正常的包服务器返回的是status=1，而我重新打包后服务器返回的是status=0

这是一种典型的challenge-response的方法，服务器发来challenge，然后程序用自身特性的一个字符串加密后再返回response，如果正确，则通过验证，反之则阻止运行。

这里我想的是，我找的加密challenge的那一段算法，看他是用什么方式加密的，用的是程序的哪一段特征值，然后像前面改签名一样，用“正确”的特征值替换下。

但是，恕我愚钝，看不懂代码，这里我贴一下逻辑，有大神对混淆比较了解的可以跟我交流下。

![](https://pic3.zhimg.com/d02dedfe6b366e88c53bfb66dbf23b46_b.png)

首先loginActivity调了这个Post请求，第一次调用参数为空，服务器会返回challenge 四个字符串

![](https://pic1.zhimg.com/75f50a3c03b445e68c2709d59fbe1e10_b.png)

程序会把这四个字符串交给一个handler处理

![](https://pic2.zhimg.com/8bbebe111963ddbe00baa380d97c0afd_b.png)

抱歉我追到这就追不下去了，因为中间这几个不管a还是b都因为混淆无法直接找到，我也没想出什么能间接找到的方法。

是不是到这就束手无策了呢，其实也不是，前面的分析是希望在最上游解决问题，如果我们能在最上游把问题解决了，下面不管什么逻辑都不用担心了，但是现在最上游无解了，那么我们就往下找一找，前面说过， 签名验证失败弹框是在服务器返回后根据服务器返回信息来判断的，那么我们可以把判断的逻辑改掉。

![](https://pic3.zhimg.com/931afe0774022b399fbb27dfe6300112_b.png)

将这个代码改成永true

我们去smali找到LoginActivity里的f类，smali编译时会把所有的内部类编成一个单独的文件，所有我们去找LoginActivity$f这个文件

![](https://pic1.zhimg.com/5d9eb6949c195f673d0c7ac783fcb4a4_b.png)

这段代码是比较status和1，如果为0则跳到cond_2，cond_2就是会给message19的那部分代码，这里我们不让他跳转，所以删掉这一句即可。另外MainActivity里也有一个同样的校验，一起改掉就行了。

现在打包，签名，运行

程序正常启动，没有弹出任何异常提醒，试试其他功能，都正常。既然签名验证我们搞定，现在往里面加一句弹toast的代码，轻而易举，我准备加在MainActivity的onCreate的时候，找到这部分代码。

![](https://pic2.zhimg.com/3f4d31d8de5455b35b5b21dd971f30f1_b.png)

注意要加在super.onCreate之后。弹框代码如下

![](https://pic4.zhimg.com/0a76ca1cf0f00aa77e194b9a833be33b_b.png)

加完代码之后如下

![](https://pic3.zhimg.com/bdf88bba17abf5941140c5ef8546343a_b.png)

打包，签名，运行

![](https://pic1.zhimg.com/cd20fa0db8cc1d8afbcc315f67837c38_b.png)

效果如上，至此，这篇文章就结束了，我们绕过了这个app的防二次打包机制，并成功的修改了代码。

##总结一下

​      1，  混淆确实是有用处的，虽然混淆后的逻辑仍然可以看懂，但是如果你想去追踪一些细节逻辑，很难，当然，我混淆代码研究的太少，经验太少也是一个方面。

​      2，  App层面上的签名验证基本是无效的的，比如一开始我们说的getSignature这里。

​      3，  采用challenge-response的方式跟服务器验证，如果使用不恰当，基本也是完全无效的，比如该应用，成功与否只判断服务器返回的一个字符串，而且判断语句是在本地，这个完全是可以绕过的。

至于更好的方法，我查资料的时候，网上看到这样一个方法，同样是跟服务器验证，但是服务器不是返回一个字段，而是返回一段核心代码，然后程序动态执行这段核心代码。我觉得采用这种方法，难度会上升一个层级。但还是无法有效避免二次打包。

[【原创】Android程序的签名保护及绕过方法**](https://link.zhihu.com/?target=http%3A//bbs.pediy.com/showthread.php%3Ft%3D180655)

几个问题：

​      1，  跟服务器验证的时候，验证的是什么东西，前面讲了因为那段代码没跟出来，所以不知道实现逻辑。以我的经验，二次打包唯一变动的应该就是签名了，但是签名我们已经绕过去了，不知道还有什么可以拿来验证的东西。

​      2，  Android资源层面的东西有没有防反编译的方法，我是说res，layout这些。

ok，洋洋洒洒的终于写完了，我是觉得自己写得已经很详细了，已经到了读者完全可以复制过程的程度。但难免有一些地方我觉得可以省略，但是读者不懂，可以在评论区提问，我会回答的。

另外，再次强调一下，绕过程序的防二次打包机制毕竟不是一件好事，搞不好做这个程序的程序员要背锅，所以文章中代码都是以图片形式给出，关键识别位置都打了马赛克，但是我想一些有心人还是可以看出这是什么程序，你看出来就看出来吧，就不要说出来了，好吗。

