Linux 内核中的数据结构(Data Structures in the Linux Kernel)
================================================================================

Radix 树（Radix tree）
--------------------------------------------------------------------------------

As you already know linux kernel provides many different libraries and functions which implement different data structures and algorithm. In this part we will consider one of these data structures - [Radix tree](http://en.wikipedia.org/wiki/Radix_tree). There are two files which are related to `radix tree` implementation and API in the linux kernel:

正如你所知道的 Linux 内核通过许多不同库以及函数提供各种数据结构以及算法。这个部分我们将介绍其中一个数据结构 [Radix tree](http://en.wikipedia.org/wiki/Radix_tree)。Linux 内核中有两个文件与 `radix tree` 的实现和API相关：

* [include/linux/radix-tree.h](https://github.com/torvalds/linux/blob/master/include/linux/radix-tree.h)
* [lib/radix-tree.c](https://github.com/torvalds/linux/blob/master/lib/radix-tree.c)

Lets talk about what is `radix tree`. Radix tree is a `compressed trie` where [trie](http://en.wikipedia.org/wiki/Trie) is a data structure which implements interface of an associative array and allows to store values as `key-value`. The keys are usually strings, but any other data type can be used as well. Trie is different from any `n-tree` in its nodes. Nodes of a trie do not store keys, instead, a node of a trie stores single character labels. The key which is related to a given node is derived by traversing from the root of the tree to this node. For example:

首先说明一下什么是 `radix tree` ，Radix tree 是一个 `压缩 trie`， [trie](http://en.wikipedia.org/wiki/Trie) 是一种通过保存关联数组（associative array）来提供 `关键字-值（key-value）` 存储与查找的数据结构。通常关键字是字符串，不过也可以是其他数据类型。 trie 结构的节点与 `n-tree` 不同，其节点中并不存储关键字，取而代之的是存储单个字符标签。关键字查找时，通过从树的根开始遍历关键字相关的所有字符标签节点，直至到达最终的叶子节点。下面是个例子：


```
               +-----------+
               |           |
               |    " "    |
               |           |
        +------+-----------+------+
        |                         |
        |                         |
   +----v------+            +-----v-----+
   |           |            |           |
   |    g      |            |     c     |
   |           |            |           |
   +-----------+            +-----------+
        |                         |
        |                         |
   +----v------+            +-----v-----+
   |           |            |           |
   |    o      |            |     a     |
   |           |            |           |
   +-----------+            +-----------+
                                  |
                                  |
                            +-----v-----+
                            |           |
                            |     t     |
                            |           |
                            +-----------+
```

So in this example, we can see the `trie` with keys, `go` and `cat`. The compressed trie or `radix tree` differs from `trie`, such that all intermediates nodes which have only one child are removed.

这个例子中，我们可以看到 `trie` 所存储的关键字信息 `go` 与 `cat`，压缩 trie 或 `radix tree` 与 `trie` 所不同的是，对于只有一个孩子的中间节点将被压缩。

Radix tree in linux kernel is the datastructure which maps values to the integer key. It is represented by the following structures from the file [include/linux/radix-tree.h](https://github.com/torvalds/linux/blob/master/include/linux/radix-tree.h):

Linux 内核中的 Radix 树将值映射为整型关键字，Radix 的数据结构定义在 [include/linux/radix-tree.h](https://github.com/torvalds/linux/blob/master/include/linux/radix-tree.h) 文件中 :


```C
struct radix_tree_root {
         unsigned int            height;
         gfp_t                   gfp_mask;
         struct radix_tree_node  __rcu *rnode;
};
```

This structure presents the root of a radix tree and contains three fields:

上面这个是 radix 树的 root 节点的结构体，它包括三个成员：

* `height`   - height of the tree;
* `gfp_mask` - tells how memory allocations are to be performed;
* `rnode`    - pointer to the child node.

* `height`   - 从叶节点向上计算出的树高度。
* `gfp_mask` - 内存申请的标识。
* `rnode`    - 子节点指针。

The first structure we will discuss is `gfp_mask`:

这里首先要讨论的结构体成员是 `gfp_mask` : 

Low-level kernel memory allocation functions take a set of flags as - `gfp_mask`, which describes how that allocation is to be performed. These `GFP_` flags which control the allocation process can have following values, (`GF_NOIO` flag) be sleep and wait for memory, (`__GFP_HIGHMEM` flag) is high memory can be used, (`GFP_ATOMIC` flag) is allocation process high-priority and can't sleep etc.

Linux 底层的内存申请接口需要提供一类标识（flag） - `gfp_mask` ，用于描述内存申请的行为。这个以 `GFP_` 前缀开头的内存申请控制标识主要包括，`GFP_NOIO` 禁止所有IO操作但允许睡眠等待内存，`__GFP_HIGHMEM` 允许申请内核的高端内存，`GFP_ATOMIC` 高优先级申请内存且操作不允许被睡眠。

The next structure is `rnode`:

接下来说的节结构体成员是`rnode`：

```C
struct radix_tree_node {
        unsigned int    path;
        unsigned int    count;
        union {
                struct {
                        struct radix_tree_node *parent;
                        void *private_data;
                };
                struct rcu_head rcu_head;
        };
        /* For tree user */
        struct list_head private_list;
        void __rcu      *slots[RADIX_TREE_MAP_SIZE];
        unsigned long   tags[RADIX_TREE_MAX_TAGS][RADIX_TREE_TAG_LONGS];
};
```

This structure contains information about the offset in a parent and height from the bottom, count of the child nodes and fields for accessing and freeing a node. The fields are described below:

这个结构体中包括这几个内容，节点与父节点的偏移以及到树底端的高度，子节点的个数，节点的存储数据域，具体描述如下：

* `path` - offset in parent & height from the bottom;
* `count` - count of the child nodes;
* `parent` - pointer to the parent node;
* `private_data` - used by the user of a tree;
* `rcu_head` - used for freeing a node;
* `private_list` - used by the user of a tree;
* `path` - 从叶节点
* `count` - 子节点的个数。
* `parent` - 父节点的指针。
* `private_data` - 存储数据内容缓冲区。
* `rcu_head` - 用于节点释放的RCU链表。
* `private_list` - 存储数据。

The two last fields of the `radix_tree_node` - `tags` and `slots` are important and interesting. Every node can contains a set of slots which are store pointers to the data. Empty slots in the linux kernel radix tree implementation store `NULL`. Radix tree in the linux kernel also supports tags which are associated with the `tags` fields in the `radix_tree_node` structure. Tags allow to set individual bits on records which are stored in the radix tree.

结构体 `radix_tree_node` 的最后两个成员 `tags` 与 `slots` 是非常重要且需要特别注意的。每个  Radix  树节点都可以包括一个指向存储数据指针的 slots 集合，空闲 slots 的指针指向 NULL。 Linux 内核的 Radix 树结构体中还包含用于记录节点存储状态的标签 `tags` 成员，标签通过位设置指示 Radix 树的数据存储状态。

Now we know about radix tree structure, time to look on its API.

至此，我们了解到 radix 树的结构，接下来看一下 radix 树所提供的 API。

Linux 内核 radix 树 API（Linux kernel radix tree API）
---------------------------------------------------------------------------------

We start from the datastructure intialization. There are two ways to initialize new radix tree. The first is to use `RADIX_TREE` macro:

我们从数据结构的初始化开始看，radix 树支持两种方式初始化。

第一个是使用宏 `RADIX_TREE` ：

```C
RADIX_TREE(name, gfp_mask);
````

As you can see we pass the `name` parameter, so with the `RADIX_TREE` macro we can define and initialize radix tree with the given name. Implementation of the `RADIX_TREE` is easy:

正如你看到，只需要提供 `name` 参数，就能够使用 `RADIX_TREE` 宏完成 radix 的定义以及初始化，`RADIX_TREE` 宏的实现非常简单：

```C
#define RADIX_TREE(name, mask) \
         struct radix_tree_root name = RADIX_TREE_INIT(mask)

#define RADIX_TREE_INIT(mask)   { \
        .height = 0,              \
        .gfp_mask = (mask),       \
        .rnode = NULL,            \
}
```

At the beginning of the `RADIX_TREE` macro we define instance of the `radix_tree_root` structure with the given name and call `RADIX_TREE_INIT` macro with the given mask. The `RADIX_TREE_INIT` macro just initializes `radix_tree_root` structure with the default values and the given mask.

`RADIX_TREE` 宏首先使用 `name` 定义了一个 `radix_tree_root` 实例并用 `RADIX_TREE_INIT` 宏带参数 `mask` 进行初始化。宏 `RADIX_TREE_INIT` 将 `radix_tree_root` 初始化为默认属性并将 gfp_mask 初始化为入参 `mask` 。

The second way is to define `radix_tree_root` structure by hand and pass it with mask to the `INIT_RADIX_TREE` macro:

第二种方式是手工定义 `radix_tree_root` 变量，之后再使用 `mask` 调用 `INIT_RADIX_TREE` 宏对变量进行初始化。

```C
struct radix_tree_root my_radix_tree;
INIT_RADIX_TREE(my_tree, gfp_mask_for_my_radix_tree);
```

where:

`INIT_RADIX_TREE` 宏定义：

```C
#define INIT_RADIX_TREE(root, mask)  \
do {                                 \
        (root)->height = 0;          \
        (root)->gfp_mask = (mask);   \
        (root)->rnode = NULL;        \
} while (0)
```

makes the same initialziation with default values as it does `RADIX_TREE_INIT` macro.

宏 `INIT_RADIX_TREE` 所初始化的属性与 `RADIX_TREE_INIT` 一致

The next are two functions for the inserting and deleting records to/from a radix tree:

接下来是 radix 树的节点插入以及删除，这两个函数：

* `radix_tree_insert`;
* `radix_tree_delete`.

The first `radix_tree_insert` function takes three parameters:

第一个函数 `radix_tree_insert` 需要三个入参：

* root of a radix tree;
* index key;
* data to insert;

* radix 树 root 节点结构
* 索引关键字
* 需要插入存储的数据

The `radix_tree_delete` function takes the same set of parameters as the `radix_tree_insert`, but without data.

第二个函数 `radix_tree_delete` 除了不需要存储数据参数外，其他与 `radix_tree_insert` 一致。

The search in a radix tree implemented in two ways:

radix 树的查找实现有以下几个函数：

* `radix_tree_lookup`;
* `radix_tree_gang_lookup`;
* `radix_tree_lookup_slot`;

The first `radix_tree_lookup` function takes two parameters:

第一个函数 `radix_tree_lookup` 需要两个参数：

* root of a radix tree;
* index key;

* radix  树 root 节点结构
* 索引关键字

This function tries to find the given key in the tree and returns associated record with this key. The second `radix_tree_gang_lookup` function have the following signature

这个函数通过给定的关键字查找 radix 树，并返关键字所对应的结点。

第二个函数 `radix_tree_gang_lookup` 具有以下特征：

```C
unsigned int radix_tree_gang_lookup(struct radix_tree_root *root,
                                    void **results,
                                    unsigned long first_index,
                                    unsigned int max_items);
```

and returns number of records, sorted by the keys, starting from the first index. Number of the returned records will be not greater than `max_items` value.

函数返回查找到记录的条目数，并根据关键字进行排序，返回的总结点数不超过入参 `max_items` 的大小。

And the last `radix_tree_lookup_slot` function will return the slot which will contain the data.

最后一个函数 `radix_tree_lookup_slot` 返回结点 slot 中所存储的数据。

链接（Links）
---------------------------------------------------------------------------------

* [Radix tree](http://en.wikipedia.org/wiki/Radix_tree)
* [Trie](http://en.wikipedia.org/wiki/Trie)
