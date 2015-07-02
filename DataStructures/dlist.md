Linux 内核中的数据结构(Data Structures in the Linux Kernel)
================================================================================

双向链表(Doubly linked list)
--------------------------------------------------------------------------------

Linux kernel provides its own doubly linked list implementation which you can find in the [include/linux/list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h). We will start `Data Structures in the Linux kernel` from the doubly linked list data structure. Why? Because it is very popular in the kernel, just try to [search](http://lxr.free-electrons.com/ident?i=list_head)

Linux 内核提供一套双向链表的实现，你可以在 [include/linux/list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h) 中找到。我们以双向链表着手开始介绍 `Linux 内核中的数据结构` ，原因在于这个是在 Linux 内核中使用最为广泛的数据结构，具体你可以 [查看](http://lxr.free-electrons.com/ident?i=list_head) 这里。

First of all let's look on the main structure:

首先让我们看一下主要的结构体：

```C
struct list_head {
	struct list_head *next, *prev;
};
```

You can note that it is different from many lists implementations which you could see. For example this doubly linked list structure from the [glib](http://www.gnu.org/software/libc/):

你可以看到其与常见的结构体实现有显著不同，比如 [glib](http://www.gnu.org/software/libc/) 中所使用到的双向链表实现。

```C
struct GList {
  gpointer data;
  GList *next;
  GList *prev;
};
```

Usually a linked list structure contains a pointer to the item. Linux kernel implementation of the list does not. So the main question is - `where does the list store the data?`. The actual implementation of lists in the kernel is - `Intrusive list`. An intrusive linked list does not contain data in its nodes - A node just contains pointers to the next and previous node and list nodes part of the data that are added to the list. This makes the data structure generic, so it does not care about entry data type anymore.

通常来说链表结构体要包括一个指向数据的指针，不过 Linux 内核的链表却不包含此实现。那么首要的疑问 `链表是用什么方式存储数据的? `。Linux 内核所实现的是一种被称为侵入式的链表（Intrusive list），这种链表并不在链表结构中包含数据，而仅提供用于维护前向与后向访问结构的指针。这种实现方式使得链表数据结构非常通用，因为它并不需要关注链表所维护的具体数据类型。

For example:

```C
struct nmi_desc {
    spinlock_t lock;
    struct list_head head;
};
```

Let's look at some examples, how `list_head` is used in the kernel. As I already wrote about, there are many, really many different places where lists are used in the kernel. Let's look for example in miscellaneous character drivers. Misc character drivers API from the [drivers/char/misc.c](https://github.com/torvalds/linux/blob/master/drivers/char/misc.c) for writing small drivers for handling simple hardware or virtual devices. This drivers share major number:

接下来让我们看一些内核使用 `list_head` 的具体例子。正如在前文所述的，Linux 内核中诸多模块都使用了 `list_head`. 这里我们以内核杂项字符设备驱动（miscellaneous character drivers）部分实现为例。驱动的 API 在 [drivers/char/misc.c](https://github.com/torvalds/linux/blob/master/drivers/char/misc.c) 中，其实现了简单硬件外设以及虚拟设备的驱动，这个驱动共享主设备号（Major number）：

```C
#define MISC_MAJOR              10
```

but has own minor number. For example you can see it with:

每个设备有自己的次设备号，具体可以看这个列子：

```
ls -l /dev |  grep 10
crw-------   1 root root     10, 235 Mar 21 12:01 autofs
drwxr-xr-x  10 root root         200 Mar 21 12:01 cpu
crw-------   1 root root     10,  62 Mar 21 12:01 cpu_dma_latency
crw-------   1 root root     10, 203 Mar 21 12:01 cuse
drwxr-xr-x   2 root root         100 Mar 21 12:01 dri
crw-rw-rw-   1 root root     10, 229 Mar 21 12:01 fuse
crw-------   1 root root     10, 228 Mar 21 12:01 hpet
crw-------   1 root root     10, 183 Mar 21 12:01 hwrng
crw-rw----+  1 root kvm      10, 232 Mar 21 12:01 kvm
crw-rw----   1 root disk     10, 237 Mar 21 12:01 loop-control
crw-------   1 root root     10, 227 Mar 21 12:01 mcelog
crw-------   1 root root     10,  59 Mar 21 12:01 memory_bandwidth
crw-------   1 root root     10,  61 Mar 21 12:01 network_latency
crw-------   1 root root     10,  60 Mar 21 12:01 network_throughput
crw-r-----   1 root kmem     10, 144 Mar 21 12:01 nvram
brw-rw----   1 root disk      1,  10 Mar 21 12:01 ram10
crw--w----   1 root tty       4,  10 Mar 21 12:01 tty10
crw-rw----   1 root dialout   4,  74 Mar 21 12:01 ttyS10
crw-------   1 root root     10,  63 Mar 21 12:01 vga_arbiter
crw-------   1 root root     10, 137 Mar 21 12:01 vhci
```

Now let's look how lists are used in the misc device drivers. First of all let's look on `miscdevice` structure:

现在我们看看驱动是如何使用链表维护设备列表的，首先，我们看一下 `miscdevice` 的 struct 定义：

```C
struct miscdevice
{
      int minor;
      const char *name;
      const struct file_operations *fops;
      struct list_head list;
      struct device *parent;
      struct device *this_device;
      const char *nodename;
      mode_t mode;
};
```

We can see the fourth field in the `miscdevice` structure - `list` which is list of registered devices. In the beginning of the source code file we can see definition of the:

可以看到 `miscdevice` 的第四个成员 `list` ，这个就是用于维护已注册设备链表的结构。在源代码文的首部我们可以看到以下定义：

```C
static LIST_HEAD(misc_list);
```

which expands to definition of the variables with `list_head` type:

这个定义宏展开，可以看到是用于定义 `list_head` 类型变量：

```C
#define LIST_HEAD(name) \
	struct list_head name = LIST_HEAD_INIT(name)
```

and initializes it with the `LIST_HEAD_INIT` macro which set previous and next entries:

`LIST_HEAD_INIT` 这个宏用于对定义的变量进行双向指针的初始化：

```C
#define LIST_HEAD_INIT(name) { &(name), &(name) }
```

Now let's look on the `misc_register` function which registers a miscellaneous device. At the start it initializes `miscdevice->list` with the `INIT_LIST_HEAD` function:

现在我看可以看一下函数 `misc_register` 是如何进行设备注册的。首先是用 `INIT_LIST_HEAD` 对 `miscdevice->list` 成员变量进行初始化：

```C
INIT_LIST_HEAD(&misc->list);
```

which does the same that `LIST_HEAD_INIT` macro:

这个操作与 `LIST_HEAD_INIT` 宏一致：

```C
static inline void INIT_LIST_HEAD(struct list_head *list)
{
	list->next = list;
	list->prev = list;
}
```

In the next step after device created with the `device_create` function we add it to the miscellaneous devices list with:

接下来，在通过函数 `device_create` 进行设备创建，同时将设备添加到 Misc 设备列表中：

```
list_add(&misc->list, &misc_list);
```

Kernel `list.h` provides this API for the addition of new entry to the list. Let's look on it's implementation:

内核的 `list.h` 提供向链表添加节点的 API，这里是添加操作的实现：

```C
static inline void list_add(struct list_head *new, struct list_head *head)
{
	__list_add(new, head, head->next);
}
```

It just calls internal function `__list_add` with the 3 given parameters:

函数实现很简单，就是入参转换为三个参数后调用内部 `__list_add` ：

* new  - new entry;
* head - list head after which will be inserted new item;
* head->next - next item after list head.
* new - 新节点
* head - 新节点插入的双向链表头
* head->next - 链表头的下一个节点

Implementation of the `__list_add` is pretty simple:

`_list_add` 函数的实现更加简单：

```C
static inline void __list_add(struct list_head *new,
			      struct list_head *prev,
			      struct list_head *next)
{
	next->prev = new;
	new->next = next;
	new->prev = prev;
	prev->next = new;
}
```

Here we set new item between `prev` and `next`. So `misc` list which we defined at the start with the `LIST_HEAD_INIT` macro will contain previous and next pointers to the `miscdevice->list`.

这里设置了新添加结点的 `prev` 与 `next` 指针，通过这些操作，就将先前使用 `LIST_HEAD_INIT` 所定义的 `misc` 链表的双向指针与 `miscdevice->list`  结构关联起来。

There is still only one question how to get list's entry. There is special special macro for this point:

这里还有一个问题就是如何获取链表中的数据，`list_head` 提供了一个特殊的宏用于获取数据指针。

```C
#define list_entry(ptr, type, member) \
	container_of(ptr, type, member)
```

which gets three parameters:

这里有三个参数

* ptr - the structure list_head pointer;
* type - structure type;
* member - the name of the list_head within the struct; 

* ptr - list_head 结构指针
* type - 数据对应的 struct 类型
* member - 数据中 list_head 成员对应的成员变量名

For example:

举例如下：

```C
const struct miscdevice *p = list_entry(v, struct miscdevice, list)
```

After this we can access to the any `miscdevice` field with `p->minor` or `p->name` and etc... Let's look on the `list_entry` implementation:

接下来我们就够访问 `miscdevice` 的各个成员，如 `p->minor`、`p->name` 等等，我们看一下 `list_entry` 的实现：

```C
#define list_entry(ptr, type, member) \
	container_of(ptr, type, member)
```

As we can see it just calls `container_of` macro with the same arguments. For the first look `container_of` looks strange:

其实现非常简单，就是使用入参调用 `container_of` 宏，宏的实现如下：

```C
#define container_of(ptr, type, member) ({                      \
    const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```

First of all you can note that it consists from two expressions in curly brackets. Compiler will evaluate the whole block in the curly braces and use the value of the last expression.

注意，宏使用了大括号表达式，对于大括号表达式，编译器会展开所有表达式，同时使用最后一个表达式的结果进行返回。

For example:

举个例子：

```
#include <stdio.h>

int main() {
	int i = 0;
	printf("i = %d\n", ({++i; ++i;}));
	return 0;
}
```

will print `2`.

输出结果为 `2` 。

The next point is `typeof`, it's simple. As you can understand from its name, it just returns the type of the given variable. When I first saw the implementation of the `container_of` macro, the strangest thing for me was the zero in the `((type *)0)` expression. Actually this pointer magic calculates the offset of the given field from the address of the structure, but as we have `0` here, it will be just a zero offset alongwith the field width. Let's look at a simple example:

另一个关键是 `typeof` 关键字，这个非常简单，这个正如它的名字一样，这个关键字返回的结果是变量的类型。当我第一次看到这个宏时，最让我觉得奇怪的是表达式 `((type*)0)`  中的 `0` 值，实际上，使用 `0` 值作为地址这个是成员变量取得 struct 内相对偏移地址的巧妙实现，我们再来看个例子：

```C
#include <stdio.h>

struct s {
        int field1;
        char field2;
		char field3;
};

int main() {
	printf("%p\n", &((struct s*)0)->field3);
	return 0;
}
```

will print `0x5`.

输出结果为 `0x5` 。

The next offsetof macro calculates offset from the beginning of the structure to the given structure's field. Its implementation is very similar to the previous code:

还有一个专门用于获取结构体中某个成员变量偏移的宏，其实现与前面提到的宏非常类似：

```C
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```

Let's summarize all about `container_of` macro. `container_of` macro returns address of the structure by the given address of the structure's field with `list_head` type, the name of the structure field with `list_head` type and type of the container structure. At the first line this macro declares the `__mptr` pointer which points to the field of the structure that `ptr` points to and assigns it to the `ptr`. Now `ptr` and `__mptr` point to the same address. Technically we don't need this line but its useful for type checking. First line ensures that that given structure (`type` parameter) has a member called `member`. In the second line it calculates offset of the field from the structure with the `offsetof` macro and subtracts it from the structure address. That's all.

这里对 `container_of` 宏做个综述，`container_of` 宏通过 struct 中的 `list_head` 成员返回 struct 对应数据的内存地址。在宏的第一行定义了指向 `list_head` 成员的指针 `__mptr` ，并将 `ptr` 地址赋给 `__mptr` 。从技术实现的角度来看，实际并不需要这一行定义，但这个对于类型检查而言非常有意义。这一行代码确保结构体（ `type` ）中存在 `member` 对应的成员。第二行使用 `offsetoff ` 宏计算出包含 `member` 的结构体所对应的内存地址，就是这么简单。

Of course `list_add` and `list_entry` is not only functions which provides `<linux/list.h>`. Implementation of the doubly linked list provides the following API:

当然 `list_add` 与 `list_entry` 并非是 `<linux/list.h>` 中的全部函数，对于双向链表 `list_head` ，内核还提供了以下的接口：

* list_add
* list_add_tail
* list_del
* list_replace
* list_move
* list_is_last
* list_empty
* list_cut_position
* list_splice

and many more.

当然内核代码中不仅仅只有上述这些接口。
