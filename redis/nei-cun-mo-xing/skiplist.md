跳跃表(SkipList)，其实也是解决查找问题的一种数据结构，但是它既不属于平衡树结构，也不属于Hash结构，它的特点是元素是有序的。

有许多数据结构的定义其实是按照（结点+组织方式）来的，结点就是一个数据点，组织方式就是把结点组织起来形成数据结构，比如 双端链表 (ListNode+list)、字典（dictEntry+dictht+dict）等。

```
typedef struct zskiplistNode {     
    sds ele;                              //数据域
    double score;                         //分值 
    struct zskiplistNode *backward;       //后向指针，使得跳表第一层组织为双向链表
    struct zskiplistLevel {               //每一个结点的层级
        struct zskiplistNode *forward;    //某一层的前向结点
        unsigned int span;                //某一层距离下一个结点的跨度
    } level[];                            //level本身是一个柔性数组，最大值为32，由 ZSKIPLIST_MAXLEVEL 定义
} zskiplistNode;
    
typedef struct zskiplist {
    struct zskiplistNode *header;     //头部
    struct zskiplistNode *tail;       //尾部
    unsigned long length;             //长度，即一共有多少个元素
    int level;                        //最大层级，即跳表目前的最大层级
} zskiplist;
```

skiplist构建过程：
![](/assets/skiplist_insertions.png)
从上面skiplist的创建和插入过程可以看出，每一个节点的层数（level）是随机出来的，而且新插入一个节点不会影响其它节点的层数。因此，插入操作只需要修改插入节点前后的指针，而不需要对很多节点都进行调整。这就降低了插入操作的复杂度。实际上，这是skiplist的一个很重要的特性，这让它在插入性能上明显优于平衡树的方案。

skiplist查找过程：
![](/assets/search_path_on_skiplist.png)

**skiplist与平衡树、哈希表的比较**
