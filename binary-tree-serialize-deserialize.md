title: 二叉树的序列化与反序列化
slug: nary-tree-serialize-deserialize
date: 2017-01-21 18:35:41
tags: algorithm

<!-- more -->

```
/*
 * A very crude queue implementation
 */
typedef
struct queue {
    struct TreeNode *val;
    struct queue    *next;
} Queue;

void queueCreate(Queue **queue)
{
    *queue = NULL;
}

void enqueue(Queue **queue, struct TreeNode *c)
{
    Queue       *tmp, *p;
    
    tmp = malloc(sizeof *tmp);
    tmp->val = c;
    tmp->next= NULL;
    
    if (! *queue) {
        *queue = tmp;
        return;
    }
    
    for (p = *queue ; p->next ; p = p->next) ;
    
    p->next = tmp;
}

void dequeue(Queue **queue, struct TreeNode **c)
{
    Queue       *tmp;
    
    if (! *queue) return;
    
    tmp = *queue;
    *c = tmp->val;
    *queue = tmp->next;
    free(tmp);
}

int queueSize(Queue **queue)
{
    Queue       *p;
    int         size;
    
    for (size = 0, p = *queue ; p ; p = p->next) ++size;
    
    return size;
}

int queueEmpty(Queue **queue)
{
    return *queue ? 0 : 1;
}

void queueDestroy(Queue **queue)
{
    /* TODO release all elements' space */
    *queue = NULL;
} 

/** Encodes a tree to a single string. */
char* serialize(struct TreeNode* root) {
    char            *buf;
    int             count;
    Queue           *queue;
    struct TreeNode *node;
    
    count = 0;
    buf = calloc(100000, sizeof *buf);

    queueCreate(&queue);
    enqueue(&queue, root);

    while (!queueEmpty(&queue)) {
        dequeue(&queue, &node);
        if (!node)
            count += snprintf(buf + count, 100000 - count, "n");
        else {
            count += snprintf(buf + count, 100000 - count, "%d", node->val);

            enqueue(&queue, node->left);
            enqueue(&queue, node->right);
        }

        if (!queueEmpty(&queue))
            count += snprintf(buf + count, 100000 - count, ",");
    }
    return buf;
}

/** Decodes your encoded data to tree. */
struct TreeNode* deserialize(char* data) {
    struct TreeNode     *root, *node, *tmp;
    char                *p, *q;
    Queue               *queue;
    
    queueCreate(&queue);
    
    p = strtok(data, ",");
    if (!p || !*p || *p == 'n') return NULL;
    
    root = malloc(sizeof *root);
    sscanf(p, "%d", &root->val);
    enqueue(&queue, root);
    
    while ((p = strtok(NULL, ",")) && (q = strtok(NULL, ","))) {
        dequeue(&queue, &tmp);
        
        node = NULL;
        if (*p != 'n') {
            node = malloc(sizeof *node);
            sscanf(p, "%d", &node->val);
            enqueue(&queue, node);
        }
        tmp->left = node;
        
        node = NULL;
        if (*q != 'n') {
            node = malloc(sizeof *node);
            sscanf(q, "%d", &node->val);
            enqueue(&queue, node);
        }
        tmp->right = node;
    }
    
    return root;
}

// Your functions will be called as such:
// char* data = serialize(root);
// deserialize(data);
```

节省了最末层的 null:

```
/*
 * A very crude queue implementation
 */
typedef
struct queue {
    struct TreeNode *val;
    struct queue    *next;
} Queue;

void queueCreate(Queue **queue)
{
    *queue = NULL;
}

void enqueue(Queue **queue, struct TreeNode *c)
{
    Queue       *tmp, *p;
    
    tmp = malloc(sizeof *tmp);
    tmp->val = c;
    tmp->next= NULL;
    
    if (! *queue) {
        *queue = tmp;
        return;
    }
    
    for (p = *queue ; p->next ; p = p->next) ;
    
    p->next = tmp;
}

void dequeue(Queue **queue, struct TreeNode **c)
{
    Queue       *tmp;
    
    if (! *queue) return;
    
    tmp = *queue;
    *c = tmp->val;
    *queue = tmp->next;
    free(tmp);
}

int queueSize(Queue **queue)
{
    Queue       *p;
    int         size;
    
    for (size = 0, p = *queue ; p ; p = p->next) ++size;
    
    return size;
}

int queueEmpty(Queue **queue)
{
    return *queue ? 0 : 1;
}

void queueDestroy(Queue **queue)
{
    /* TODO release all elements' space */
    *queue = NULL;
}

int maxdepth(struct TreeNode *root)
{
    int     d1, d2;

    if (!root) return -1;
    else {
        d1 = maxdepth(root->left) + 1;
        d2 = maxdepth(root->right) + 1;
        return d1 > d2 ? d1 : d2;
    }
}

/** Encodes a tree to a single string. */
char* serialize(struct TreeNode* root) {
    char            *buf;
    int             count;
    Queue           *queue, *queue2;
    long            depth, mdepth;
    struct TreeNode *node;
    
    count = 0;
    buf = calloc(100000, sizeof *buf);

    mdepth = maxdepth(root);

    queueCreate(&queue);
    queueCreate(&queue2);
    
    enqueue(&queue, root);
    enqueue(&queue2, 0);

    while (!queueEmpty(&queue)) {
        dequeue(&queue, &node);
        dequeue(&queue2, &depth);
        if (!node) {
            if (depth <= mdepth)
                count += snprintf(buf + count, 100000 - count, "n");
        } else {
            count += snprintf(buf + count, 100000 - count, "%d", node->val);

            enqueue(&queue, node->left);
            enqueue(&queue2, depth + 1);
            
            enqueue(&queue, node->right);
            enqueue(&queue2, depth + 1);
        }

        if (!queueEmpty(&queue) && depth <= mdepth)
            count += snprintf(buf + count, 100000 - count, ",");
    }

    return buf;
}

/** Decodes your encoded data to tree. */
struct TreeNode* deserialize(char* data) {
    struct TreeNode     *root, *node, *tmp;
    char                *p, *q;
    Queue               *queue;
    
    queueCreate(&queue);
    
    p = strtok(data, ",");
    if (!p || !*p || *p == 'n') return NULL;
    
    root = malloc(sizeof *root);
    sscanf(p, "%d", &root->val);
    root->left = root->right = NULL;
    enqueue(&queue, root);
    
    while ((p = strtok(NULL, ","))) {
        dequeue(&queue, &tmp);
        
        node = NULL;
        if (*p != 'n') {
            node = malloc(sizeof *node);
            sscanf(p, "%d", &node->val);
            node->left = node->right = NULL;
            enqueue(&queue, node);
        }
        tmp->left = node;
        
        if (!(q = strtok(NULL, ","))) break;
        
        node = NULL;
        if (*q != 'n') {
            node = malloc(sizeof *node);
            sscanf(q, "%d", &node->val);
            node->left = node->right = NULL;
            enqueue(&queue, node);
        }
        tmp->right = node;
    }
    
    return root;
}

// Your functions will be called as such:
// char* data = serialize(root);
// deserialize(data);
```
