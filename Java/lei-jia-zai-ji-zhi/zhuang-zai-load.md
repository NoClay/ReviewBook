## 一、装载\(Load\)

ClassLoader就是用来装载的。通过指定的className，找到二进制码，生成Class实例，放到JVM中。ClassLoader从顶向下分为 Bootstrap ClassLoader、Extension ClassLoader、System ClassLoader以及User-Defined ClassLoader（分叉，可以多个）。如下图。![](http://www.ibm.com/developerworks/java/library/j-dclp1/clhierarchy.gif)

![](http://storage1.imgchr.com/ZuyTO.png)

这是Tomcat装载器的例子：

装载过程从源码清析可见：

```
protected synchronized Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    // 先检查是否已被当前ClassLoader装载。
    Class c = findLoadedClass(name);
    if (c == null) {
        try {
        if (parent != null) {
            // 如果没被当前装载，则递归的到父中装载。
            c = parent.loadClass(name, false);
        } else {
        // 装载器树已到顶，还没找到的话就到Bootstrap装载器中找。
        // 注意：虽Bootstrap是所有加载器的根，但它是C++实现的，不可能放到子的"parent"中，
        // 因此，第二层装载器是所有的根了。
            c = findBootstrapClass0(name);
        }
        } catch (ClassNotFoundException e) {
        // 如果祖先都无法装载，则用当前的装载。子类可在findClass方法中调用defineClass，
        // 把从自定义位置获得的字节码转换成Class。
            c = findClass(name);
        }
    }
    if (resolve) { // Links the specified class.
        resolveClass(c);  // 注
    }
    return c;
}
```

注：

resolveClass\(c\)方法的注释是链接类，而不只是解析，从该即可看出。调用resolveClass时语义上是去链接，是否真的链接了我不是很清楚，但可以肯定的是没有初始化。当A类中有static B b=new B\(\)时，最晚会在初始化时去装载B。如果改成static B b=null，那么把B.class删掉后，即使A已经链接，初始化过了，但也不会报错，也就是A所引用的B类没有被加载过。解析时难道没有真的去装入它所引用的B类？还是链接时，没有执行解析的步骤？ 问题的关键就是 1.对“解析”的理解，解析时是否会去装载B类？2. JVM在链接时是否执行了解析？（毕竟有资料说，解析是可选步骤）

