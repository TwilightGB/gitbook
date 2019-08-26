intset是Redis内存数据结构之一，是一个由整数组成的有序集合，从而便于在上面进行二分查找，用于快速地判断一个元素是否属于这个集合。它在内存分配上与ziplist有些类似，是连续的一整块内存空间，而且对于大整数和小整数（按绝对值）采取了不同的编码，尽量对内存的使用进行了优化。它的特点有：


元素类型只能为数字。
元素有三种类型：int16_t、int32_t、int64_t。
元素有序，不可重复。
intset和sds一样，内存连续，就像数组一样。

**数据结构定义：**


```
typedef struct intset {
    uint32_t encoding;  // 编码类型 int16_t、int32_t、int64_t
    uint32_t length;    // 长度 最大长度:2^32
    int8_t contents[];  // 柔性数组
} intset;
```
其中需要注意的是，intset可能会随着数据的添加而改变它的数据编码：

最开始，新创建的intset使用占内存最小的INTSET_ENC_INT16（值为2）作为数据编码。
每添加一个新元素，则根据元素大小决定是否对数据编码进行升级。

![](/assets/redis_intset_add_example.png)

1.新创建的intset只有一个header，总共8个字节。其中encoding = 2, length = 0。
2.添加13, 5两个元素之后，因为它们是比较小的整数，都能使用2个字节表示，所以encoding不变，值还是2。
3.当添加32768的时候，它不再能用2个字节来表示了（2个字节能表达的数据范围是215~215-1，而32768等于215，超出范围了），因此encoding必须升级到INTSET_ENC_INT32（值为4），即用4个字节表示一个元素。
4.在添加每个元素的过程中，intset始终保持从小到大有序。
5.与ziplist类似，intset也是按小端（little endian）模式存储的（参见维基百科词条Endianness）。比如，在上图中intset添加完所有数据之后，表示encoding字段的4个字节应该解释成0x00000004，而第5个数据应该解释成0x000186A0 = 100000。

intset与ziplist相比：

ziplist可以存储任意二进制串，而intset只能存储整数。

ziplist是无序的，而intset是从小到大有序的。因此，在ziplist上查找只能遍历，而在intset上可以进行二分查找，性能更高

ziplist可以对每个数据项进行不同的变长编码（每个数据项前面都有数据长度字段len），而intset只能整体使用一个统一的编码（encoding）。
## redis的set

set命令：

sadd用于分别向集合s1和s2中添加元素。添加的元素既有数字，也有非数字（”a”和”b”）。

sismember用于判断指定的元素是否在集合内存在。

sinter, sunion和sdiff分别用于计算集合的交集、并集和差集。

