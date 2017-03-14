## 三、初始化

> Initialization of a class consists of executing its static initializers and the initializers for`static`fields \(class variables\) declared in the class. Initialization of an interface consists of executing the initializers for fields \(constants\) declared there.

类的初始化包括：执行静态区块和静态方法的初始化。比如下面这两种代码都会被执行，包括new B\(\)。

> static{ ...}
>
> static B b=new B\(\);

接口中不允许有static initializer（也就是static{...}），所以对于接口，只会执行静态字段的初始化。

初始化前，装载，链接一定已经执行过！

类初始化前，它的直接父类一定要先初始化（递归），但它实现的接口不需要先被初始化。类似的，接口在初始化前，父接口不需要先初始化。

什么情况下，类的初始化会被触发？

> A class or interface type_T_will be initialized immediately before the first occurrence of any one of the following:_T_is a class and an instance of_T_is created._T_is a class and a static method declared by_T_is invoked.A static field declared by_T_is assigned.A static field declared by_T_is used and the field is not a constant variable[\(§4.12.4\)](http://blog.sina.com.cn/s/typesValues.html#10931)._T_is a top-level class, and an`assert`statement[\(§14.10\)](http://blog.sina.com.cn/s/statements.html#35518)lexically nested within_T_is executed.
>
> 当使用类的字段时，即便可以通过子类或子接口访问该字段，但只有真正定义该字段的类会被触发初始化。如下例。

```java
class Super { static int taxi = 1729; }
class Sub extends Super { static { System.out.print("Sub "); } }
class Test {
    public static void main(String[] args) {
               System.out.println(Sub.taxi);
    }}
```

只会输出“1729”，不会输出"Sub"，也就是说，Sub其实没有被初始化。

