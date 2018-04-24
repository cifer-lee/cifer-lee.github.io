title: 嵌入式哈系表的实现
slug: embedded-hashtable-implementation
date: 2017-01-05
category: algorithm
tags: algorithm

这篇文章是一个例子, 重点在于阐述嵌入式哈系表结构, 而不是针对不同键类型进行哈希的方法.

关于哈希冲突的解决, 我们使用链表存储冲突的节点, 这在一些书里被称为 "Separate Chaning" 方法.

由于在链表中插入以及删除节点需要更新节点的前继和后继的指针, 所以为了方便的从链表中插入以及删除节点, `hlist_node_t` 结构中定义两个指针, 分别指向前继和后继.

```
typedef struct hlist_node_s {
    struct hlist_node_s *prev;
    struct hlist_node_s *next;
} hlist_node_t;
```

`hlist_t` 结构中的 `heads` 是一个数组, 每个数组元素都只是一个头节点, 头节点不会嵌入到别的结构体中.

```
typedef struct xhash {
    hlist_node_t    *heads;
    int             size;
} xhash_t;

void hlist_node_init(hlist_node_t *node)
{
    node->next = node->prev = node;
}

xhash_t *xhash_create(int size)
{
    xhash_t     *hash;
    int         i;

    /* TODO Get next prime bigger than *size* */
    /* size = next_prime(size);*/

    hash = calloc(1, sizeof *hash);
    if (!hash)
        return NULL;

    hash->heads = calloc(size, sizeof *hash->heads);
    if (!hash->heads) {
        free(hash);
        return NULL;
    }

    for (i = 0 ; i < size ; ++i)
        hlist_node_init(&hash->heads[i]);

    hash->size = size;

    return hash;
}
```

为了方便链表的遍历, 我们还需要定义如下几个宏, 其中 `offsetof` 宏一般 C 标准库会为我们定义, 并且 C 库的定义会考虑到不同系统上的差异. 为了完整的阐述我们的实现, 这里我还是将它 "bare" 的定义写了出来.

```
#define offsetof(type, member)              \
    ({                                      \
        type s;                             \
        (char *)(&s.member) - (char *)(&s); \
    })

#define container_of(node, type, member)                \
    ((type *)((char *)(node) - offsetof(type, member)))

#define hlist_for_each(head, node)                      \
    for ((node) = (head)->next ;                        \
            (node) != (head) ; (node) = (node)->next)

#define hlist_for_each_safe(head, node, next)           \
    for ((node) = (head)->next, (next) = (node)->next ; \
            (node) != (head) ;                          \
            (node) = (next), (next) = (node)->next)

void hlist_add_head(hlist_node_t *head, hlist_node_t *new)
{
    head->next->prev = new;
    new->next = head->next;
    head->next = new;
    new->prev = head;
}

void hlist_add_tail(hlist_node_t *head, hlist_node_t *new)
{
    head->prev->next = new;
    new->prev = head->prev;
    head->prev = new;
    new->next = head;
}

void hlist_del(hlist_node_t *node)
{
    node->next->prev = node->prev;
    node->prev->next = node->next;
}
```

如前所说, 为方便起见我们仅实现整数类型的键, 哈系函数采用简单的取模运算. 其它类型的键的哈系后面只要进一步扩展就可以了, 这里不再多说.

```
void xhash_int_add(xhash_t *hash, int key, hlist_node_t *node)
{
    key %= hash->size;
    hlist_add_head(&hash->heads[key], node);
}

void xhash_int_del(xhash_t *hash, int key, hlist_node_t *node)
{
    key %= hash->size;
    hlist_del(&hash->heads[key], node);
}

/** return hlist head */
hlist_node_t *xhash_int_find(xhash_t *hash, int key)
{
    key %= hash->size;
    return &hash->heads[key];
}
```

在 `xhash_destroy` 方法中, 要不要帮助用户将所有嵌入的节点从链表中移除是个问题, 可能比较人性化的方法是帮助用户移除. 但假如节点嵌入的结构体是用户动态申请的, 然后用户调用 `xhash_destroy` 之前就把结构体释放了呢? 这样节点成员所处的内存就是无效的了, 我们的 `xhash_destroy` 方法就很可能会崩溃. 所以这活儿还是不干了.

```
void xhash_destroy(xhash_t *hash)
{
    /*hlist_node_t    *node, *next;
    int             i;*/

    /* TODO
     * whether we should del all node from corresponding list? what if user remove
     * the node member from embedded struct?
     */
    /*
    for (i = 0 ; i < size ; ++i) {
        hlist_for_each_safe(&hash->heads[i], node, next) {
            hlist_del(node);
            /* restore to initial state */
            hlist_node_init(node);
        }

    }
    */

    free(hash->heads);
    free(hash);
}
```

下面我们用一个例子测试一下我们的实现:

```
typedef struct xy_s {
    int             key;
    int             x;
    int             y;
    hlist_node_t    node;
} xy_t;

int xy_print(xhash_t *hash)
{
    int             idx;
    hlist_node_t    *node;
    xy_t            *xy;

    if (!hash) return -1;

    for (idx = 0 ; idx < hash->size ; ++idx) {
        printf("bucket[%d]: ", idx);
        hlist_for_each(&hash->heads[idx], node) {
            xy = container_of(node, xy_t, node);
            printf("%d, ", xy->key);
        }

        printf("\n");
    }

    return 0;
}

int main()
{
    xhash_t     *hash;
    xy_t        *xy;
    int         i;

    assert((hash = xhash_create(13)) != NULL);

    printf("\ninsert 0 to 99 nodes \n\n");

    for (i = 0 ; i < 100 ; ++i) {
        xy = calloc(1, sizeof *xy);
        xy->key = i;
        xy->x = i * 3;
        xy->y = i * 4;
        hlist_node_init(&xy->node);
        xhash_int_add(hash, xy->key, &xy->node);
    }

    xy_print(hash);

    printf("\ndelete 99\n\n");

    xhash_int_del(hash, 99, &xy->node);
    xy_print(hash);

    return 0;
}
```

编译运行以上程序, 输出的信息如下:

```
insert 0 to 99 nodes 

bucket[0]: 91, 78, 65, 52, 39, 26, 13, 0, 
bucket[1]: 92, 79, 66, 53, 40, 27, 14, 1, 
bucket[2]: 93, 80, 67, 54, 41, 28, 15, 2, 
bucket[3]: 94, 81, 68, 55, 42, 29, 16, 3, 
bucket[4]: 95, 82, 69, 56, 43, 30, 17, 4, 
bucket[5]: 96, 83, 70, 57, 44, 31, 18, 5, 
bucket[6]: 97, 84, 71, 58, 45, 32, 19, 6, 
bucket[7]: 98, 85, 72, 59, 46, 33, 20, 7, 
bucket[8]: 99, 86, 73, 60, 47, 34, 21, 8, 
bucket[9]: 87, 74, 61, 48, 35, 22, 9, 
bucket[10]: 88, 75, 62, 49, 36, 23, 10, 
bucket[11]: 89, 76, 63, 50, 37, 24, 11, 
bucket[12]: 90, 77, 64, 51, 38, 25, 12, 

delete 99

bucket[0]: 91, 78, 65, 52, 39, 26, 13, 0, 
bucket[1]: 92, 79, 66, 53, 40, 27, 14, 1, 
bucket[2]: 93, 80, 67, 54, 41, 28, 15, 2, 
bucket[3]: 94, 81, 68, 55, 42, 29, 16, 3, 
bucket[4]: 95, 82, 69, 56, 43, 30, 17, 4, 
bucket[5]: 96, 83, 70, 57, 44, 31, 18, 5, 
bucket[6]: 97, 84, 71, 58, 45, 32, 19, 6, 
bucket[7]: 85, 72, 59, 46, 33, 20, 7, 
bucket[8]: 86, 73, 60, 47, 34, 21, 8, 
bucket[9]: 87, 74, 61, 48, 35, 22, 9, 
bucket[10]: 88, 75, 62, 49, 36, 23, 10, 
bucket[11]: 89, 76, 63, 50, 37, 24, 11, 
bucket[12]: 90, 77, 64, 51, 38, 25, 12, 
```
