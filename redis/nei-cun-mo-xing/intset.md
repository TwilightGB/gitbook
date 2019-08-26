intset是Redis内存数据结构之一，用来实现Redis的Set结构（当元素较小且为数字类型时），它的特点有：
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

