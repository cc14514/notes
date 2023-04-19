# linux kernel 中的链表


## 常用 API

Linux kernel 5.4.43 中常用的链表 API 为双向循环链表，定义在 include/linux/list.h 中。常用的 API 包括：

* LIST_HEAD(name)：定义一个名为 name 的头结构体，用于表示链表头。
* INIT_LIST_HEAD(ptr)：初始化 ptr 指向的链表头结构体，使之成为一个空链表。
* list_add(new, head)：将 new 结点插入到链表头 head 之后。
* list_add_tail(new, head)：将 new 结点插入到链表头 head 之前。
* list_del(entry)：将 entry 结点从链表中删除。
* list_replace(old, new)：将 old 结点替换为 new 结点。
* list_empty(head)：判断链表 head 是否为空。
* list_entry(ptr, type, member)：获取 ptr 所在的结构体 type 的地址，member 是 type 结构体中链表的成员名。
* list_for_each(pos, head)：遍历链表，pos 为当前结点，head 为链表头。
* list_for_each_entry(pos, head, member)：遍历链表，pos 为当前结构体，head 为链表头，member 是 type 结构体中链表的成员名。

以上是常用的链表 API，可以在 include/linux/list.h 中找到详细信息。

## 示例

```c
#include <linux/list.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");

struct my_struct {
    int data;
    struct list_head list;
};

struct my_struct list_head;

static int __init my_init(void)
{
    struct my_struct *node;
    struct list_head *pos, *n;

    printk(KERN_INFO "Loading module\n");

    // Initialize the list
    INIT_LIST_HEAD(&list_head.list);

    // Add some nodes
    for (int i = 0; i < 5; i++) {
        node = kmalloc(sizeof(*node), GFP_KERNEL);
        node->data = i;
        list_add(&node->list, &list_head.list);
    }

    // Traverse the list
    list_for_each(pos, &list_head.list) {
        node = list_entry(pos, struct my_struct, list);
        printk(KERN_INFO "Node data: %d\n", node->data);
    }

    // Remove the nodes
    list_for_each_safe(pos, n, &list_head.list) {
        node = list_entry(pos, struct my_struct, list);
        list_del(pos);
        kfree(node);
    }

    printk(KERN_INFO "Module loaded\n");
    return 0;
}

static void __exit my_exit(void)
{
    printk(KERN_INFO "Unloading module\n");
}

module_init(my_init);
module_exit(my_exit);
```

这个例子中定义了一个结构体 my_struct，其中包含了一个 int 类型的 data 和一个 list_head 类型的 list，用于链表操作。

在 my_init 函数中，先使用 INIT_LIST_HEAD 初始化链表头部 list_head.list，然后使用 list_add 在链表中加入五个节点，每个节点的 data 为 0 到 4。接着使用 list_for_each 遍历链表，并输出每个节点的 data。最后使用 list_for_each_safe 和 list_del 删除链表中的节点，并释放它们所占用的内存。

在 my_exit 函数中，仅仅输出一个卸载模块的信息。