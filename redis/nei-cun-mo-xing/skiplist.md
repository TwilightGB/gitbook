跳跃表(SkipList)，其实也是解决查找问题的一种数据结构，但是它既不属于平衡树结构，也不属于Hash结构，它的特点是元素是有序的。

有许多数据结构的定义其实是按照（结点+组织方式）来的，结点就是一个数据点，组织方式就是把结点组织起来形成数据结构，比如 双端链表 (ListNode+list)、字典（dictEntry+dictht+dict）等。

typedef struct zskiplistNode {     
    sds ele;                              //数据域
    double score;                         //分值 
    struct zskiplistNode *backward;       //后向指针，使得跳表第一层组织为双向链表
    struct zskiplistLevel {               //每一个结点的层级
        struct zskiplistNode *forward;    //某一层的前向结点
        unsigned int span;                //某一层距离下一个结点的跨度
    } level[];                            //level本身是一个柔性数组，最大值为32，由 ZSKIPLIST_MAXLEVEL 定义
} zskiplistNode;