## 谈谈final，finally，finalize的区别

1. final（修饰词）：

   如果一个类被声明为final，那么这个类无法被继承，所以一个类无法同时既是abstract的，也是final的，如果使用final修饰方法或者变量，可以保证它们不会被修改，但是需要注意的是引用类型的变量final对其引用生效，比如数组int \[\]a，对于数组a，a为对整个数组的引用，但是可以修改a\[0\]之类的值。被声明为final的方法不可以被重载。

   ```
     final Test test = new Test(3); 
     System.out.println("data = " + test.getA());
     test.setA(5);
     System.out.println("data = " + test.getA());
   ```

   结果：

   > data = 3
   >
   > data = 5

2. finally（异常处理部分）：

   在异常处理的时候，finally用于执行任何清除操作，在异常处理的时候，无论怎样，最后都会经历finally流程，所以如果在try中返回一个值，finally中返回一个值，finally中的返回值会覆盖之前的返回值。

3. finalize（方法名）：

   finalize是Object中的基本方法（toString，hashCode，wait，notify，notifyAll，getClass，equals），该方法用于在垃圾回收器将对象回收之前做必要的清理工作，这个方法是在垃圾收集器确定对象没有被引用时对这个对象调用的。

   功能：我们可以进行标记日志和复活动向，利用log跟踪jvm的内存回收，将要回收的对象赋值给一个新的对象实现对象的复活。在android中，我们通过ndk调用的native方法是不会被jvm自动进行回收的，我们需要在fialize（）方法中调用native方法free\(\)方法实现内存回收。

## String与StringBuffer，StringBuilder的区别。

String对象不可变，StringBuffer长度可变，另外StringBuilder是线程不安全的，StringBuffer是线程安全的，所以如果我们需要拼接字符，那么使用StringBuffer和StringBuilder是不错的。在单线程中StringBuilder的性能比StringBuffer性能高

## switch是否可以作用在byte上，是否可以作用在long上，是否能作用在String上？

switch支持的类型：char, byte, short, int, Character, Byte, Short, Integer, String, enum。

String类型是在jdk7后支持的

# java异常处理机制

Java对异常进行了分类，不同类型的异常分别用不同的Java类表示，所有异常的根类为java.lang.Throwable，Throwable下面又派生了两个子类：**Error和Exception**，Error表示应用程序本身无法克服和恢复的一种严重问题，程序只有死的份了，例如，说内存溢出和线程死锁等系统问题。Exception表示程序还能够克服和恢复的问题，其中又分为系统异常和普通异常，系统异常是软件本身缺陷所导致的问题，也就是软件开发人员考虑不周所导致的问题，软件使用者无法克服和恢复这种问题，但在这种问题下还可以让软件系统继续运行或者让软件死掉，例如，**数组脚本越界（ArrayIndexOutOfBoundsException），空指针异常（NullPointerException）、类转换异常（ClassCastException）；**普通异常是运行环境的变化或异常所导致的问题，是用户能够克服的问题，例如，网络断线，硬盘空间不够，发生这样的异常后，程序不应该死掉。

**异常处理方式**

**1\) 检查到异常出现，设置对象值为空字符串或一个默认值； 2\) 检测到异常出现，根本不执行某操作，直接跳转到其他处理中。 3\) 检查到异常出现，提示用户操作有错误。**

# java中有几种类型的流？

字节流和字符流，字节流继承自InputStream，OutputStream，字符流继承自InputStreamReader，OutputStreamWriter。

# 写clone方法时，通常有一行代码，是什么？

Clone有缺省行为，super.clone\(\)；负责产生正确大小的空间，并逐位复制。

# java语言如何进行异常处理，关键字throws, throw，try catch, finally分别代表了什么意义？

通过面向对象的方法进行异常处理，把各种不同的异常进行分类，并提供了良好的额接口，在java中，每一个异常都是一个对象，他是Throwable类或者其他子类的实例，当一个方法出现异常后便抛出一个异常对象，该对象中包含有异常信息，调用这个兑现过的方法可以捕获到这个异常并进行处理。

try执行一段程序，如果出现异常，系统会抛出（throws）一个异常，这时候你可以通过他的类型来捕获（catch）他，或者最后（finally）由缺省处理器来捕捉

throw用来抛出异常，throws哟on过来标明一个成员函数可能抛出的异常。finally是无论如何都会执行的。

