# hash table 内核中的哈西表结构

## 常用 API

>在 Linux 内核中，Hash Table 和 RCU 是经常一起使用的。Hash Table 主要用于快速查找元素，而 RCU 用于读操作的并发保护，以便多个线程能够同时访问 Hash Table。以下是 Hash Table 与 RCU 相关的一些 API：

* hlist_add_head_rcu() 和 hlist_add_tail_rcu()：在 Hash Table 中添加新节点，使用 RCU 确保并发读访问的正确性。

* hlist_del_rcu()：从 Hash Table 中删除一个节点，使用 RCU 确保并发读访问的正确性。

* hlist_for_each_entry_rcu()：使用 RCU 遍历 Hash Table，读取节点数据时使用 rcu_dereference() 等 RCU API 以确保正确性。

* rcu_assign_pointer()：用于使用 RCU 保护指针赋值操作。

* synchronize_rcu()：等待所有已经开始的 RCU 读操作完成，以便对 Hash Table 进行修改。

>使用 Hash Table 时，需要特别注意并发读写的正确性，以避免数据不一致等问题。使用 RCU 可以有效地提高 Hash Table 的并发读取性能，并减少锁竞争。


## demo

首先，定义一个哈希表结构体 my_file_ht，包含了一个哈希数组和一个自旋锁用于保护哈希表的并发操作。

```c
#define HT_SIZE 1024

struct my_file_ht {
    struct hlist_head head[HT_SIZE];
    spinlock_t lock;
};
```

接着，定义一个缓存文件状态的结构体 my_file_ar，包含了文件描述符的 inode 号和状态掩码。

```c
struct my_file_ar {
    __le32 ino;
    int mask;
    struct hlist_node node;
};
```

接下来，提供哈希表的初始化函数，初始化哈希表的每个哈希槽都为空链表，并初始化哈希表的自旋锁。

```c
void my_file_ht_init(struct my_file_ht *ht)
{
    int i;
    for (i = 0; i < HT_SIZE; i++)
        INIT_HLIST_HEAD(&ht->head[i]);
    spin_lock_init(&ht->lock);
}
```

然后，提供一个添加节点的函数，使用哈希表的自旋锁来保护节点的添加操作。

```c
void my_file_ht_add(struct my_file_ht *ht, struct my_file_ar *ar)
{
    unsigned int hash = my_file_ht_hashfn(ar->ino);
    spin_lock_bh(&ht->lock);
    hlist_add_head_rcu(&ar->node, &ht->head[hash]);
    spin_unlock_bh(&ht->lock);
}
```


接下来，提供一个查找节点的函数，使用 RCU 机制来保护节点的读操作。需要注意的是，这里使用了 hlist_for_each_entry_rcu 函数来遍历链表。

```c
struct my_file_ar *my_file_ht_lookup(struct my_file_ht *ht, __le32 ino)
{
    unsigned int hash = my_file_ht_hashfn(ino);
    struct my_file_ar *ar = NULL;
    struct hlist_node *n;
    rcu_read_lock();
    hlist_for_each_entry_rcu(ar, n, &ht->head[hash], node) {
        if (ar->ino == ino) {
            rcu_read_unlock();
            return ar;
        }
    }
    rcu_read_unlock();
    return NULL;
}
```

最后，提供一个删除节点的函数，使用哈希表的自旋锁来保护节点的删除操作。

```c
void my_file_ht_del(struct my_file_ht *ht, struct my_file_ar *ar)
{
    unsigned int hash = my_file_ht_hashfn(ar->ino);
    spin_lock_bh(&ht->lock);
    hlist_del_rcu(&ar->node);
    spin_unlock_bh(&ht->lock);
    call_rcu(&ar->rcu, my_file_ht_rcu_free);
}
```

同时，我们还需要提供一个回调函数 my_file_ht_rcu_free，用于释放节点内存空间：

```c
static void my_file_ht_rcu_free(struct rcu_head *rcu)
{
    struct my_file_ar *ar = container_of(rcu, struct my_file_ar, rcu);
    kfree(ar);
}
```
在这个回调函数中，我们使用 container_of 宏将 RCU 头转换为节点指针，然后调用 kfree 函数释放节点内存空间。

这样，就完成了一个基于哈希表和 RCU 的缓存文件描述符及其状态的 demo。在多线程环境下，读操作不需要加锁，写操作只需要用自旋锁来保护并发修改，从而提高了读写操作的效率。


```c
static inline unsigned int my_file_ht_hashfn(__le32 ino)
{
    return le32_to_cpu(ino) % HT_SIZE;
}
```
这个哈希函数使用了 inode 的值对哈希表大小进行求模，从而获得在哈希表中的索引值。其中，le32_to_cpu 用于将 inode 的字节序从小端转换为主机字节序，以便更好地进行哈希计算。

请注意，这个哈希函数只是一个简单的示例，实际应用中可能需要根据具体的场景来编写更加复杂的哈希函数。
