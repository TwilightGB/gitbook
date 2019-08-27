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

