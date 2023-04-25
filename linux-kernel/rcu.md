# 什么是 RCU
RCU（Read-Copy-Update）是一种用于并发访问共享数据结构的机制，主要用于读多写少的场景。RCU 机制的主要特点是无锁读（lock-free read）和延迟删除（deferred deletion），它不需要使用显式的互斥锁来保护共享数据结构的访问，而是通过一些巧妙的手段来实现并发访问共享数据结构的安全性和高效性。
    
在 RCU 机制中，读取共享数据结构的操作是无锁的，因此读取操作可以并发进行，不会相互干扰。写入共享数据结构的操作则使用了延迟删除的策略，即写入操作并不直接修改共享数据结构，而是将要删除的数据结构标记为“已删除”，并在之后的某个时间点（通常是在不会干扰读取操作的时候）真正删除这些数据结构。
    
RCU 机制的实现依赖于一些底层机制，比如内存屏障、原子操作等。在 Linux 内核中，RCU 机制被广泛应用于多个子系统，比如进程管理、网络协议栈等，以提高内核的并发性能。

# 锁与 RCU

看起来 RCU 和读写锁比较相似，当然 kernel 中并没有读写锁的实现，只提供了互斥锁和自旋锁，但 RCU 是种不同的并发编程技术，它们各自有自己的优点和适用场景。

## 适合锁的场景
锁适用于短时间内占用共享资源的情况。当一个线程获得锁后，其他线程需要等待该线程释放锁后才能获得锁。如果获得锁的线程占用时间很长，那么其他等待的线程会被阻塞，这样会造成严重的性能问题。因此，锁适用于短时间内占用共享资源的情况。

## 适合 RCU 的场景
而 RCU 则适用于读多写少的情况。RCU 允许多个线程并发地读取共享资源，而不需要获得锁或等待其他线程释放锁，从而提高了并发度。当需要修改共享资源时，RCU 会延迟对共享资源的释放，从而避免了读取共享资源的线程被阻塞的问题。因此，RCU 适用于读多写少的情况。

在实际的系统开发中，自旋锁和 RCU 可以根据实际的情况进行选择，以保证系统的性能和正确性。


# RCU 相关的 API

在 Linux 5.4.43 中，提供了以下常用的 RCU API：

* rcu_read_lock()：获取 RCU 读锁，读取共享数据结构时必须先获取读锁。
* rcu_read_unlock()：释放 RCU 读锁。
* synchronize_rcu()：等待当前所有使用 RCU 读取共享数据结构的读者完成后再执行后续操作。
* call_rcu()：将一个回调函数关联到 RCU 机制中，当没有任何 RCU 读者在使用共享数据结构时执行该回调函数。
* rcu_assign_pointer()：原子地将指针赋值给共享数据结构，并确保该指针在 RCU 读者完成读取之前一直存在。
* rcu_dereference()：读取共享数据结构中的指针，返回一个指针的副本，并确保该指针在 RCU 读者完成读取之前一直存在。

这些 API 可以帮助我们使用 RCU 机制来保护共享数据结构，避免出现数据竞争和死锁等问题。

# RCU 示例

## 示例1 

``` c
#include <linux/module.h>
#include <linux/rcupdate.h>

struct my_node {
    int val;
    struct rcu_head rcu;
    struct list_head list;
};

LIST_HEAD(my_list);

/* 添加一个节点到链表中 */
void add_node(int val)
{
    struct my_node *new_node = kmalloc(sizeof(*new_node), GFP_KERNEL);
    if (!new_node) {
        printk(KERN_ERR "Failed to allocate memory for new node\n");
        return;
    }

    new_node->val = val;
    INIT_LIST_HEAD(&new_node->list);

    /* 加入链表 */
    list_add(&new_node->list, &my_list);
}

/* 删除值为 val 的节点 */
void del_node(int val)
{
    struct my_node *node, *tmp;

    /* 遍历链表并删除匹配节点 */
    list_for_each_entry_safe(node, tmp, &my_list, list) {
        if (node->val == val) {
            list_del_rcu(&node->list);
            call_rcu(&node->rcu, kfree);
        }
    }
}

/* 遍历整个链表，打印节点的值 */
void traverse_list(void)
{
    struct my_node *node;

    /* 进入 RCU 读取临界区 */
    rcu_read_lock();

    /* 遍历链表并打印节点的值 */
    list_for_each_entry_rcu(node, &my_list, list) {
        printk(KERN_INFO "Node value: %d\n", node->val);
    }

    /* 离开 RCU 读取临界区 */
    rcu_read_unlock();
}
```

>示例中，我们使用 RCU 来保护链表的访问。添加节点时，我们不需要获取锁来保护共享资源。删除节点时，我们使用了 list_del_rcu 来删除节点，并使用 call_rcu 函数来安排释放内存的回调函数。在遍历链表时，我们使用了 rcu_read_lock 和 rcu_read_unlock 来进入和离开 RCU 读取临界区。



## 示例2

```c
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/rculist.h>
#include <linux/slab.h>

struct my_data {
    int id;
    char name[20];
    struct list_head list;
    struct rcu_head rcu;
};

static struct my_data *my_data_alloc(int id, const char *name)
{
    struct my_data *data = kmalloc(sizeof(*data), GFP_KERNEL);
    if (!data)
        return NULL;
    data->id = id;
    strncpy(data->name, name, sizeof(data->name) - 1);
    INIT_LIST_HEAD(&data->list);
    return data;
}

static void my_data_free_rcu(struct rcu_head *rcu)
{
    struct my_data *data = container_of(rcu, struct my_data, rcu);
    kfree(data);
}

static void my_data_free(struct my_data *data)
{
    call_rcu(&data->rcu, my_data_free_rcu);
}

static void my_data_add(struct my_data *data, struct list_head *head)
{
    list_add(&data->list, head);
}

static void my_data_del(struct my_data *data)
{
    list_del_rcu(&data->list);
    my_data_free(data);
}

static struct my_data *my_data_find(int id, struct list_head *head)
{
    struct my_data *data;
    rcu_read_lock();
    list_for_each_entry_rcu(data, head, list) {
        if (data->id == id) {
            rcu_read_unlock();
            return data;
        }
    }
    rcu_read_unlock();
    return NULL;
}

static int __init my_module_init(void)
{
    struct my_data *data1 = my_data_alloc(1, "Alice");
    struct my_data *data2 = my_data_alloc(2, "Bob");
    struct my_data *data3 = my_data_alloc(3, "Charlie");
    struct my_data *data4 = my_data_alloc(4, "David");

    INIT_LIST_HEAD(my_list);

    my_data_add(data1, &my_list);
    my_data_add(data2, &my_list);
    my_data_add(data3, &my_list);
    my_data_add(data4, &my_list);

    struct my_data *data = my_data_find(3, &my_list);
    if (data)
        printk(KERN_INFO "Found data: id=%d name=%s\n", data->id, data->name);

    return 0;
}

static void __exit my_module_exit(void)
{
    struct my_data *data, *next;

    list_for_each_entry_safe(data, next, &my_list, list) {
        my_data_del(data);
    }
}

module_init(my_module_init);
module_exit(my_module_exit);
MODULE_LICENSE("GPL");
```

>这个例子中定义了一个 my_data 结构体，包含一个 id、一个 name，还有一个 rcu_head 结构体，用来实现 RCU 机制。这个结构体同时还包含了一个 list_head，用于构成链表。 在 my_data_alloc 函数中，使用 kmalloc 分配内存，并将数据写入结构体。在 my_data_free_rcu 函数中，使用 container_of 宏将 rcu_head 转换为 my_data，然