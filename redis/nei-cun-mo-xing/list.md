链表定义：

```
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;
    
typedef struct list { 

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void(*free)(void *ptr);

    // 节点值对比函数
    int(*match)(void *ptr, void *key);

    // 链表所包含的节点数量
    unsigned long len;

} list;
```
![](/assets/list.png)

**Redis链表的特性：**

1.双端：获取某个结点的前驱和后继结点都是O(1)

2.无环：表头的prev指针和表尾的next指针都指向NULL，对链表的访问都是以NULL为终点

3.带表头指针和表尾指针：获取表头和表尾的复杂度都是O(1)

4.带链表长度计数器：len属性记录，获取链表长度O(1)

5.多态：链表结点使用void*指针来保存结点的值，并且可以通过链表结构的三个函数为结点值设置类型特定函数，所以链表可以保存各种不同类型的值