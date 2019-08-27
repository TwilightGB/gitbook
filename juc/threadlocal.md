## ThreadLocal定义
ThreadLocal是保存的线程的本地变量，访问get/set方法都是对线程独立的。
大白话就是ThreadLocal是和线程相关的，在一个线程没有结束之前，在任意方法中get/set在ThreadLocal中设置的值都是只和当前线程有关。
因此呢，ThreadLocal的使用场景也可以推测出来，可以用来在一个线程中传递参数，或者某些情况（比如session，数据库操作句柄）只跟线程相关的时候来使用。



```
static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
   }
```

ThreadLocalMap，它里面有一个静态类Entry（可以类比Map中的entry，保存实际的key和value，），ThreadLocalMap其实就是保存了ThreadLocal调用set方法设置的value，key就是ThreadLocal。
总结一下就是当我们使用ThreadLocal的set方法时，ThreadLocal为key，保存的泛型对象为value，存到了ThreadLocal的内部类ThreadLocalMap中，然后ThreaLocalMap的键值对实际上是放在静态类Entry里面。这里稍微提一句Entry是继承了WeakReference弱引用，key是被弱引用的构造函数给创建的，value是强引用。

**Get方法**



```
public T get() {
       Thread t = Thread.currentThread();
       ThreadLocalMap map = getMap(t);
       if (map != null) {
           ThreadLocalMap.Entry e = map.getEntry(this);
           if (e != null) {
               @SuppressWarnings("unchecked")
               T result = (T)e.value;
               return result;
           }
       }
       return setInitialValue();
   }
```

**Set方法**


```

public void set(T value) {
      Thread t = Thread.currentThread();
      ThreadLocalMap map = getMap(t);
      if (map != null)
          map.set(this, value);
      else
          createMap(t, value);
  }

```



1.ThreadLocal在同一个线程传值，或者只跟线程相关的场景使用
2.初始化ThreadLocal的值可以用ThreadLocal.withInitial，或重写initialValue()
3.父子线程共享相同的值使用InheritableThreadLocal
4.不管是正常使用还是线程池使用ThreadLocal都一定要使用完就remove，否则会内存泄漏或者数据出错

