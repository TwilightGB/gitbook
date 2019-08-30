## ThreadLocal定义

ThreadLocal是保存的线程的本地变量，访问get/set方法都是对线程独立的。ThreadLocal是和线程相关的，在一个线程没有结束之前，在任意方法中get/set在ThreadLocal中设置的值都是只和当前线程有关。

将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不同的变量值完成操作的场景。

可以用来在一个线程中传递参数，或者某些情况（比如session，数据库操作句柄）只跟线程相关的时候来使用。

![](D:\books\Import\java_base\assets\threadlocal\threadlocal.jpg)

### TheradLocal结构图

![](D:\books\Import\java_base\assets\threadlocal\threadlocal uml.jpg)

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

###### **Get方法**

get流程如下：

1. 获取当前线程的threadLocals（map结构），从threadLocals中获取当前ThreadLocal变量对应的ThreadLocalMap.Entry(pair类型，包含了当前ThreadLocal变量及其对应的value)，非空直接返回对应的value
2. 为空时使用默认值（默认为null）构造ThreadLocalMap.Entry，放到当前线程的threadLocals中，下次再get时直接返回ThreadLocalMap.Entry对应的value即可

```java
/**
 * 当前线程的threadLocalMap中获取当前ThreadLocal对应的value
 */
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
    // 设置null值，下次直接返回null了
    return setInitialValue();
}

/**
 * 如果一次找到了entry，直接返回；否则就是set时hash冲突了
 * 遍历后续的slot，进行查找
 * 这里其实JDK可以做个优化，在set之后，将slot位置记录在Threadlocal变量中，下次直接到对应slot位置get即可
 */
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

```

###### **Set方法**

- set操作就是将ThreadLocal变量的值put到当前线程的threadLocals中，ThreadLocal变量及其对应的值会构造成一个ThreadLocalMap.Entry放到threadLocals中。
- 因为线程的threadLocals是一个基于开放定址法实现的map结构，所以在出现hash冲突后会继续寻找下一个空位进行set操作。
- 因为是基于开放定址法，如果map中元素过多，会影响get和put性能，所以需要扩容，map的数组结构默认大小为`INITIAL_CAPACITY = 16`，默认扩容阈值为`threshold = INITIAL_CAPACITY * 2 / 3`，扩容时按照成倍扩容。

```java
/**
 * 获取当前线程的threadLocalMap，非空直接set value;
 * 否则新建一个包含value的threadLocalMap。
 * threadLocalMap的key对应程序中定义的ThreadLocal变量，value对应要set的值
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t); // Thread.threadLocals
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

// Entry，里面保存在ThreadLocal变量，也就是key，是弱引用
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}

/**
 * hash码的生成，这里所有的ThreadLocal对象hash生成都是基于static变量nextHashCode来做的
 * 创建ThreadLocal对象时threadLocalHashCode已初始化完成
 */
private final int threadLocalHashCode = nextHashCode();
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
private static AtomicInteger nextHashCode =
        new AtomicInteger();

/**
 * 当前线程的threadLocalMap非空直接set value
 */
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    // 如果当前table[i] hash冲突，那么就以i为起点，遍历后续table[i]，
    // 这其实就是hash冲突中的开放定址法，另外一种是分离链接法
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        // key已存在，更新vlaue即可
        if (k == key) {
            e.value = value;
            return;
        }
        // key为null，复制value即可
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    // 新建Entry，清理一部分Entry.key为null，value不为null的数据，避免内存泄露
    // 超过了threshold时rehash操作
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}


```

###### remove

```java
/**
 * 从ThreadLocalMap删除对应key
 */
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            // 清除Entry.key弱引用，设置为null
            e.clear();
            // 清除Entry.value引用，可能还涉及部分key为null的Entry数据清理
            expungeStaleEntry(i);
            return;
        }
    }
}
```

###### hash冲突怎么解决**

和HashMap的最大的不同在于，ThreadLocalMap结构非常简单，没有next引用，也就是说ThreadLocalMap中解决Hash冲突的方式并非链表的方式，而是采用**线性探测**的方式，所谓线性探测，就是根据初始key的hashcode值确定元素在table数组中的位置，如果发现这个位置上已经有其他key值的元素被占用，则利用固定的算法寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。

ThreadLocalMap解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。

显然ThreadLocalMap采用线性探测的方式解决Hash冲突的效率很低，如果有大量不同的ThreadLocal对象放入map中时发送冲突，或者发生二次冲突，则效率很低。

所以这里引出的良好建议是：每个线程只存一个变量，这样的话所有的线程存放到map中的Key都是相同的ThreadLocal，如果一个线程要保存多个变量，就需要创建多个ThreadLocal，多个ThreadLocal放入Map中时会极大的增加Hash冲突的可能。

###### **关于内存泄露**

每个Thread内部都维护一个ThreadLocalMap字典数据结构，字典的Key值是ThreadLocal，那么当某个ThreadLocal对象不再使用（没有其它地方再引用）时，每个已经关联了此ThreadLocal的线程怎么在其内部的ThreadLocalMap里做清除此资源呢？

JDK中的ThreadLocalMap没有继承java.util.Map类，而是自己实现了一套专门用来定时清理无效资源的字典结构。其内部存储实体结构Entry<ThreadLocal, T>继承自java.lan.ref.WeakReference，这样当ThreadLocal不再被引用时，因为弱引用机制原因，当发生gc时，会自动回收弱引用指向的实例内存，即其线程内部ThreadLocalMap会释放其对ThreadLocal的引用从而让jvm回收ThreadLocal对象。

这里是重点**强调**下，是回收对ThreadLocal对象，而非整个Entry，所以线程变量中的值T对象还是在内存中存在的，所以内存泄漏的问题还没有完全解决。接着分析JDK的实现，会发现在调用ThreadLocal.get()或者ThreadLocal.set(T)时都会定期执行回收无效的Entry操作。

```java
private static final ThreadLocal<UserInfo> userInfoLocal = new ThreadLocal<UserInfo>(); 
```

Entry中的key是弱引用，key 弱指向ThreadLocal<UserInfo> 对象，并且Key只是userInfoLocal强引用的副本（结合第一个问题），value是userInfo对象。

当我显示的把userInfoLocal = null 时就只剩下了key这一个弱引用，GC时也就会回ThreadLocal<UserInfo> 对象。

但是我们最好避免threadLocal=null的操作，尽量用threadLocal.remove()来清除。因为前者中的userInfo对象还是存在强引用在当前线程中，只有当前thread结束以后, current thread就不会存在栈中,强引用断开, 会被GC回收。但是如果用的是线程池，那么的话线程就不会结束，只会放在线程池中等待下一个任务，但是这个线程的 map 还是没有被回收，它里面存在value的强引用，所以会导致内存溢出。
![](D:\books\Import\java_base\assets\threadlocal\ref.jpg)

ThreadLocal的实现是这样的：每个Thread 维护一个 `ThreadLocalMap` 映射表，这个映射表的 key 是 `ThreadLocal`实例本身，value 是真正需要存储的 Object。

###### 关于弱引用

```
如果一个对象只具有弱引用，那么垃圾回收器在扫描到该对象时，无论内存充足与否，都会回收该对象的内存。
```

当使用cache的时候, 由于cache的对象正是程序运行需要的, 那么只要程序正在运行, cache中的引用就不会被GC给(或者说, cache中的reference拥有了和主程序一样的life cycle). 那么随着cache中的reference越来越多, GC无法回收的object也越来越多, 无法被自动回收。(可以用软引用)

当一个对象仅仅被weak reference指向, 而没有任何其他strong reference指向的时候, 如果GC运行, 那么这个对象就会被回收. weak reference的语法是:

```java
WeakReference<Car> weakCar = new WeakReference(Car)(car);
```

 当要获得weak reference引用的object时, 首先需要判断它是否已经被回收:

```java
weakCar.get();
```

如果Entry.referent弱类型指向的对象回收了（没调用ThreadLocal.remove操作），Entry.value对象还在，并且Entry.value可是强引用的，此时就发生了内存泄露。这也就是ThreadLocal使用不当（没调用ThreadLocal.remove）时产生的内存泄漏问题。

### **总结：**

1.ThreadLocal在同一个线程传值，或者只跟线程相关的场景使用
2.初始化ThreadLocal的值可以用ThreadLocal.withInitial，或重写initialValue()
3.父子线程共享相同的值使用InheritableThreadLocal
4.不管是正常使用还是线程池使用ThreadLocal都一定要使用完就remove，否则会内存泄漏或者数据出错

