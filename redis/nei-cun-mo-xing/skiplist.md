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
1.skiplist和各种平衡树（如AVL、红黑树等）的元素是有序排列的，而哈希表不是有序的。因此，在哈希表上只能做单个key的查找，不适宜做范围查找。所谓范围查找，指的是查找那些大小在指定的两个值之间的所有节点。
2.在做范围查找的时候，平衡树比skiplist操作要复杂。在平衡树上，我们找到指定范围的小值之后，还需要以中序遍历的顺序继续寻找其它不超过大值的节点。如果不对平衡树进行一定的改造，这里的中序遍历并不容易实现。而在skiplist上进行范围查找就非常简单，只需要在找到小值之后，对第1层链表进行若干步的遍历就可以实现。
3.平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速。
4.从内存占用上来说，skiplist比平衡树更灵活一些。一般来说，平衡树每个节点包含2个指针（分别指向左右子树），而skiplist每个节点包含的指针数目平均为1/(1-p)，具体取决于参数p的大小。如果像Redis里的实现一样，取p=1/4，那么平均每个节点包含1.33个指针，比平衡树更有优势。
5.查找单个key，skiplist和平衡树的时间复杂度都为O(log n)，大体相当；而哈希表在保持较低的哈希值冲突概率的前提下，查找时间复杂度接近O(1)，性能更高一些。所以我们平常使用的各种Map或dictionary结构，大都是基于哈希表实现的。
6.从算法实现难度上来比较，skiplist比平衡树要简单得多。

## sorted set的命令举例

sorted set是一个有序的数据集合，对于像类似排行榜这样的应用场景特别适合。
现在我们来看一个例子，用sorted set来存储代数课（algebra）的成绩表。原始数据如下：

```
    Alice 87.5
    Bob 89.0
    Charles 65.5
    David 78.0
    Emily 93.5
    Fred 87.5
```

![](/assets/redis_skiplist_example.png)
