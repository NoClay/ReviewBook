# JVM加载类的原理机制？

Java中，在调用类的静态成员，或新建该类的对象等之前，类一定要先装入Java虚拟机中，这是勿庸置疑的。但虚拟机怎样把类装载进来的呢？要经过三步：装载（Load），链接（Link），初始化（Initializ）。其中链接又可分为校验（Verify），准备（Prepare），解析（Resolve）三步。

![](http://www.ibm.com/developerworks/cn/java/j-dclp1/clphases.gif)

## Class.forName\(\)与ClassLoader.loadClass\(\)

这两方法都可以通过一个给定的类名去定位和加载这个类名对应的 java.long.Class 类对象，区别如下：

1. 初始化Class.forName\(\)会对类初始化，而loadClass\(\)只会装载或链接。可见的效果就是类中静态初始化段及字节码中对所有静态成员的初始工作的执行\(这个过程在类的所有父类中递归地调用\). 这点就与ClassLoader.loadClass\(\)不同. ClassLoader.loadClass\(\)加载的类对象是在第一次被调用时才进行初始化的。你可以利用上述的差异. 比如,要加载一个静态初始化开销很大的类, 你就可以选择提前加载该类\(以确保它在classpath下\), 但不进行初始化, 直到第一次使用该类的域或方法时才进行初始化

2. 类加载器可能不同Class.forName\(String\) 方法\(只有一个参数\), 使用调用者的类加载器来加载, 也就是用加载了调用forName方法的代码的那个类加载器。当然，它也有个重载的方法，可以指定加载器。 相应的, ClassLoader.loadClass\(\)方法是一个实例方法\(非静态方法\), 调用时需要自己指定类加载器, 那么这个类加载器就可能是也可能不是加载调用代码的类加载器（调用代用代码类加载器通getClassLoader0\(\)获得）

参考资料：

1. [http://java.sun.com/docs/books/jls/third\_edition/html/execution.html\#44487](http://java.sun.com/docs/books/jls/third_edition/html/execution.html#44487)这章是“Execution”，很详细讲了类装载过程，很权威！

2. [http://www.ibm.com/developerworks/cn/java/j-dclp1/](http://www.ibm.com/developerworks/cn/java/j-dclp1/)类装入问题解秘，不错。

3. [http://baike.baidu.com/view/160708.htm](http://baike.baidu.com/view/160708.htm)百度百科



