final（修饰词）：

1. 如果一个类被声明为final，那么这个类无法被继承，所以一个类无法同时既是abstract的，也是final的，如果使用final修饰方法或者变量，可以保证它们不会被修改，但是需要注意的是引用类型的变量final对其引用生效，比如数组int \[\]a，对于数组a，a为对整个数组的引用，但是可以修改a\[0\]之类的值。被声明为final的方法不可以被重载。

   ```java
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



