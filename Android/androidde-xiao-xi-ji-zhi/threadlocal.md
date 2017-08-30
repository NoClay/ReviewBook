# ThreadLocal的工作原理

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储后，只有在指定线程中才可以获取到对应的存储的数据。一般来讲，当某些数据是以线程为作用域并且不同的线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal，比如Looper、ActivityThread、AMS。Threadlocal还可以用于复杂逻辑下的对象传递，比如监听器的传递，有些时候一个线程的任务过于复杂，这可能表现为函数调用栈比较深以及代码入口的多样性。

## ThreadLocal.set\(\)

我们看源码：

```
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

我们可以看到这里的存储使用了ThreadLocalMap这个类，这个类实际上就是一个哈希映射表，但是并没有实现map接口。下面我们看官方对这个类的解释：

> ThreadLocalMap is a customized hash map suitable only for maintaining thread local values. No operations are exported outside of the ThreadLocal class. The class is package private to allow declaration of fields in class Thread. To help deal with very large and long-lived usages, the hash table entries use WeakReferences for keys. However, since reference queues are not used, stale entries are guaranteed to be removed only when the table starts running out of space.
>
> ThreadLocalMap是一个自定义的哈希映射，仅适用于维护线程局部值。 没有定义在ThreadLocal类外部导出任何操作。 类是包私有的，允许在类Thread中声明字段。 为了帮助处理非常大和长期使用，哈希表条目使用WeakReferences（弱引用）的密钥。 但是，由于不使用引用队列，因此只有当表开始耗尽空间时，才保证删除过时的条目。

## ThreadLocal.get\(\)

我们看源码：

```
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }
    protected T initialValue() {
        return null;
    }
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

可以看出，当我们从ThreadLocal中获取数值的时候，如果并没有设置，会获取一个null值

