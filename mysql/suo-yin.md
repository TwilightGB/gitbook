# 索引定义

索引（Index）是帮助MySQL高效获取数据的数据结构。索引的本质：索引是数据结构。

优点：可以快速检索，减少I/O次数，加快检索速度；根据索引分组和排序，可以加快分组和排序；

缺点：索引本身也是表，因此会占用存储空间，一般来说，索引表占用的空间是数据表的1.5倍；索引表的维护和创建需要时间成本，这个成本随着数据量增大而增大；构建索引会降低数据表的修改操作（删除，添加，修改）的效率，因为在修改数据表的同时还需要修改索引表；

# 索引的分类

### **哈希索引：**

​		哈希索引用索引列的值计算该值的hashCode，然后在hashCode相应的位置存执该值所在行数据的物理位置，因为使用散列算法，因此访问速度非常快，但是一个值只能对应一个hashCode，而且是散列的分布方式，因此哈希索引不支持范围查找和排序的功能。

​		innoDB支持的hash索引是自适应的，innoDB会根据表的使用情况自动的生成hash索引，不能人为干预在表中生成哈希索引。

### **全文索引**:

​		FULLTEXT（全文）索引，仅可用于MyISAM和InnoDB，针对较大的数据，生成全文索引非常的消耗时间和空间。对于文本的大对象，或者较大的CHAR类型的数据，如果使用普通索引，那么匹配文本前几个字符还是可行的，但是想要匹配文本中间的几个单词，那么就要使用LIKE %word%来匹配，这样需要很长的时间来处理，响应时间会大大增加，这种情况，就可使用时FULLTEXT索引了，在生成FULLTEXT索引时，会为文本生成一份单词的清单，在索引时及根据这个单词的清单来索引。FULLTEXT可以在创建表的时候创建，也可以在需要的时候用ALTER或者CREATE INDEX来添加：

```mysql
//创建表的时候添加FULLTEXT索引
CTREATE TABLE my_table(
    id INT(10) PRIMARY KEY,
    name VARCHAR(10) NOT NULL,
    my_text TEXT,
    FULLTEXT(my_text)
)ENGINE=MyISAM DEFAULT CHARSET=utf8;

//创建表以后，在需要的时候添加FULLTEXT索引
ALTER TABLE my_table ADD FULLTEXT INDEX ft_index(column_name);
```

```mysql
SELECT * FROM table_name MATCH(ft_index) AGAINST('查询字符串');
```

### **B+Tree索引**

#### **BTree索引**

BTree是平衡搜索多叉树，设树的度为2d（d>1），高度为h，那么BTree要满足以一下条件：

    每个叶子结点的高度一样，等于h；
    每个非叶子结点由n-1个key和n个指针point组成，其中d<=n<=2d,key和point相互间隔，结点两端一定是key；
    叶子结点指针都为null；
    非叶子结点的key都是[key,data]二元组，其中key表示作为索引的键，data为键值所在行的数据；
![img](https://img-blog.csdn.net/20180411150103704)

在BTree的结构下，就可以使用二分查找的查找方式，查找复杂度为h*log(n)，一般来说树的高度是很小的，一般为3左右，因此BTree是一个非常高效的查找结构。

#### B+Tree索引

B+Tree是BTree的一个变种，设d为树的度数，h为树的高度，B+Tree和BTree的不同主要在于：

    B+Tree中的非叶子结点不存储数据，只存储键值；
    B+Tree的叶子结点没有指针，所有键值都会出现在叶子结点上，且key存储的键值对应data数据的物理地址；
    B+Tree的每个非叶子节点由n个键值key和n个指针point组成；


![img](https://img-blog.csdn.net/20180411151308606)