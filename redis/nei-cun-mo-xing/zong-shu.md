关于Redis数据存储的细节，涉及到内存分配器（如jemalloc）、简单动态字符串（SDS）、5种对象类型及内部编码、RedisObject。
下图是执行set hello world时，所涉及到的数据模型。
![](/assets/redis.png)
（1）dictEntry：Redis是Key-Value数据库，因此对每个键值对都会有一个dictEntry，里面存储了指向Key和Value的指针；next指向下一个dictEntry，与本Key-Value无关。

（2）Key：图中右上角可见，Key（“hello”）并不是直接以字符串存储，而是存储在SDS结构中。

（3）redisObject：Value(“world”)既不是直接以字符串存储，也不是像Key一样直接存储在SDS中，而是存储在redisObject中。实际上，不论Value是5种类型的哪一种，都是通过RedisObject来存储的；而RedisObject中的type字段指明了Value对象的类型，ptr字段则指向对象所在的地址。不过可以看出，字符串对象虽然经过了RedisObject的包装，但仍然需要通过SDS存储。实际上，RedisObject除了type和ptr字段以外，还有其它字段图中没有给出，如用于指定对象内部编码的字段。后面会详细介绍。

（4）jemalloc：无论是DictEntry对象，还是RedisObject、SDS对象，都需要内存分配器（如jemalloc）分配内存进行存储。以DictEntry对象为例，有3个指针组成，在64位机器下占24个字节，jemalloc会为它分配32字节大小的内存单元。

## jemalloc
Redis在编译时便会指定内存分配器；内存分配器可以是 libc 、jemalloc或者tcmalloc，默认是jemalloc。
jemalloc作为Redis的默认内存分配器，在减小内存碎片方面做的相对比较好。jemalloc在64位系统中，将内存空间划分为小、大、巨大三个范围；每个范围内又划分了许多小的内存块单位；当Redis存储数据时，会选择大小最合适的内存块进行存储。
## RedisObject
Redis对象有5种类型；无论是哪种类型，Redis都不会直接存储，而是通过RedisObject对象进行存储。RedisObject对象非常重要，Redis对象的类型、内部编码、内存回收、共享对象等功能，都需要RedisObject支持。
RedisObject的定义如下：


```
typedef struct redisObject {

　　unsigned type:4;

　　unsigned encoding:4;

　　unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */

　　int refcount;

　　void *ptr;

} robj;
```


### 1.type
type字段表示对象的类型，占4个比特；目前包括REDIS_STRING(字符串)、REDIS_LIST (列表)、REDIS_HASH(哈希)、REDIS_SET(集合)、REDIS_ZSET(有序集合)。
当我们执行type命令时，便是通过读取RedisObject的type字段获得对象的类型。
### 2.encoding
encoding表示对象的内部编码，占4个比特。
对于Redis支持的每种类型，都有至少两种内部编码，例如对于字符串，有int、embstr、raw三种编码。通过encoding属性，Redis可以根据不同的使用场景来为对象设置不同的编码，大大提高了Redis的灵活性和效率。

以列表对象为例，有压缩列表和双端链表两种编码方式；如果列表中的元素较少，Redis倾向于使用压缩列表进行存储，因为压缩列表占用内存更少，而且比双端链表可以更快载入；当列表对象元素较多时，压缩列表就会转化为更适合存储大量元素的双端链表。
### 3.lru
lru记录的是对象最后一次被命令程序访问的时间
通过对比lru时间与当前时间，可以计算某个对象的空转时间；object idletime命令可以显示该空转时间（单位是秒）。
lru还与Redis的内存回收有关系：如果Redis打开了maxmemory选项，且内存回收算法选择的是volatile-lru或allkeys—lru，那么当Redis内存占用超过maxmemory指定的值时，Redis会优先选择空转时间最长的对象进行释放。
### 4.refcount
refcount记录的是该对象被引用的次数，类型为整型。refcount的作用，主要在于对象的引用计数和内存回收：


    当创建新对象时，refcount初始化为1；

    当有新程序使用该对象时，refcount加1；

    当对象不再被一个新程序使用时，refcount减1；

    当refcount变为0时，对象占用的内存会被释放。

Redis中被多次使用的对象(refcount>1)称为共享对象。Redis为了节省内存，当有一些对象重复出现时，新的程序不会创建新的对象，而是仍然使用原来的对象。这个被重复使用的对象，就是共享对象。目前共享对象仅支持整数值的字符串对象。
### 5.ptr
ptr指针指向具体的数据，如前面的例子中，set hello world，ptr指向包含字符串world的SDS。

综上所述，redisObject的结构与对象类型、编码、内存回收、共享对象都有关系；一个redisObject对象的大小为16字节：

4bit+4bit+24bit+4Byte+8Byte=16Byte。
## SDS
Redis没有直接使用C字符串(即以空字符‘\0’结尾的字符数组)作为默认的字符串表示，而是使用了SDS。SDS是简单动态字符串(Simple Dynamic String)的缩写。
sds的结构如下：


```
struct sdshdr {

    int len;

    int free;

    char buf[];

};
```
buf表示字节数组，用来存储字符串；
len表示buf已使用的长度
free表示buf未使用的长度。
![](/assets/c.png)
