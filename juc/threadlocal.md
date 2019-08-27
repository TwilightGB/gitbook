## ThreadLocal定义
ThreadLocal是保存的线程的本地变量，访问get/set方法都是对线程独立的。ThreadLocal是和线程相关的，在一个线程没有结束之前，在任意方法中get/set在ThreadLocal中设置的值都是只和当前线程有关。

将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不同的变量值完成操作的场景。

可以用来在一个线程中传递参数，或者某些情况（比如session，数据库操作句柄）只跟线程相关的时候来使用。

![](/assets/threadlocal/threadlocal.jpg)

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
总结一下就是当我们使用ThreadLocal的set方法时，ThreadLocal为key，保存的泛型对象为value，存到了ThreadLocal的内部类ThreadLocalMap中，然后ThreaLocalMap的键值对实际上是放在静态类Entry里面。这里稍微提一句Entry是继承了WeakReference弱引用（弱引用，生命周期只能存活到下次GC前），key是被弱引用的构造函数给创建的，value是强引用。

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

**hash冲突怎么解决**

和HashMap的最大的不同在于，ThreadLocalMap结构非常简单，没有next引用，也就是说ThreadLocalMap中解决Hash冲突的方式并非链表的方式，而是采用线性探测的方式，所谓线性探测，就是根据初始key的hashcode值确定元素在table数组中的位置，如果发现这个位置上已经有其他key值的元素被占用，则利用固定的算法寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。

ThreadLocalMap解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。

显然ThreadLocalMap采用线性探测的方式解决Hash冲突的效率很低，如果有大量不同的ThreadLocal对象放入map中时发送冲突，或者发生二次冲突，则效率很低。


1.ThreadLocal在同一个线程传值，或者只跟线程相关的场景使用
2.初始化ThreadLocal的值可以用ThreadLocal.withInitial，或重写initialValue()
3.父子线程共享相同的值使用InheritableThreadLocal
4.不管是正常使用还是线程池使用ThreadLocal都一定要使用完就remove，否则会内存泄漏或者数据出错


