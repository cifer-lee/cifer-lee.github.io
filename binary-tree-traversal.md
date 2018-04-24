title: 二叉树递归遍历的本质以及用迭代遍历精确模拟递归遍历
slug: binary-tree-traversal
date: 2016-10-22 17:24:36
catagories: algorithm
tags: algorithm, binary tree, leetcode, DFS

递归遍历是实现深度优先遍历的既直观又简单的方式, 二叉树递归遍历的本质其实就是在不断的压栈与出栈, 明白了这一道理之后就很容易借助栈结构来将递归遍历转换成迭代遍历.

    void traverse(struct TreeNode *root)
    {
        if (root) {
            /* 1 */
            traverse(root->left);
            /* 2 */
            traverse(root->right);
            /* 3 */
        }
    }

上面的 1, 2, 3 三个位置是节点访问可能发生的位置, 分别为先序, 中序, 后序遍历. 下面我们不考虑对节点数据的访问, 单独分析一下递归遍历过程的细节.

<!-- more -->

## 函数调用与递归的本质

要分析递归遍历的细节, 我们就要知道递归的过程中发生了什么, 而递归实际上也只是函数调用的一种, 所以我们需要先来看一下函数调用的过程中发生了什么.

当函数调用发生时, 传给该函数的参数, 以及被调函数内部的局部变量都会被压入栈中 (现在的处理器和编译器基本都支持通过寄存器传参, 这里我们没必要考虑这些个情况), 而在被调函数返回时, 这些信息又都会从栈中弹出. 递归只是一种自己调用自己的函数调用, 也是符合这个规则的. 

我们以上面的 `traverse(struct TreeNode *root)` 方法和下面的一颗二叉树为例, 观察一下递归遍历过程中的栈变化.

                            A
                         /     \
                        B       C
                       / \     / \
                      D   E   F   G

其中 A 是根节点, 遍历过程由 `traverse(root)` 被调用开始, 那么按照上述的规则, 会发生如下几步, 为方便起见, 栈的增长为从左往右:

1. traverse(A) 被调用, A 节点入栈

       ------------
       | A |   |
       ------------

2. 在 A 层 traverse 中, 判断 A 不是 NULL, 于是调用 `traverse(root->left)`, 也就是 `traverse(B)`, 于是 B 入栈

       ------------
       | A | B |
       ------------

3. 判断 B 不是 NULL, 继续调用 `traverse(root->left)`, 即 `traverse(D)`, 于是 D 入栈

       ----------------
       | A | B | D |
       ----------------

4. 判断 D 不是 NULL, 继续调用 `traverse(root->left)`, 即 `traverse(NULL)`, 于是 NULL 入栈

       ------------------
       | A | B | D | NULL
       ------------------

5. 这层 `traverse` 的 `if` 判断为 NULL, 函数直接返回, 返回时将弹出栈顶的 NULL

       ------------------
       | A | B | D |
       ------------------

6. 到达 D 层的 `traverse`, 继续调用 `traverse(root->right)`, 即 `traverse(NULL)`, 于是 NULL 入栈

       ------------------
       | A | B | D | NULL
       ------------------

7. 和 `5.` 的情况一样, 这层函数将直接返回, 并弹出栈顶的 NULL

       ------------------
       | A | B | D |
       ------------------

8. 此时 D 层 `traverse` 也结束了, 函数返回, 并弹出栈顶的 D

       ------------
       | A | B |
       ------------

9. 回到 B 层 `traverse`, 调用 `traverse(root->right)`, 即 `traverse(E)`, 于是 E 入栈

.....


下面的代码能够完美的模拟递归遍历过程


	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	
	struct TreeNode {
	    int val;
	    struct TreeNode *left;
	    struct TreeNode *right;
	};
	
	typedef struct stack {
	    void           *c;
	    struct stack   *next;
	} Stack;
	
	void create(Stack **ptop)
	{
	    *ptop = NULL;
	}
	
	void push(Stack **ptop, void *c)
	{
	    Stack   *tmp;
	
	    tmp = malloc(sizeof *tmp);
	    tmp->c = c;
	    tmp->next = *ptop;
	
	    *ptop = tmp;
	}
	
	void pop(Stack **ptop, void **c)
	{
	    Stack   *tmp;
	
	    if (! *ptop) return;
	
	    tmp = *ptop;
	    if (c) *c = tmp->c;
	    *ptop = (*ptop)->next;
	    free(tmp);
	}
	
	void top(Stack **ptop, void **c)
	{
	    *c = (*ptop)->c;
	}
	
	int empty(Stack **ptop)
	{
	   return *ptop == NULL; 
	}
	
	/**
	 * Return an array of size *returnSize.
	 * Note: The returned array must be malloced, assume caller calls free().
	 */
	int* postorderTraversal(struct TreeNode* root, int* returnSize)
	{
	    int             *arr;
	    long            mark;
	    Stack           *s;
	    Stack           *pc;
	    
	    create(&s);
	    create(&pc);
	    arr = malloc(10000 * sizeof *arr);
	    *returnSize = 0;
	    
	    push(&s, root);
	    push(&pc, 0);
	    while (!empty(&s)) {
	        top(&s, &root);
	        
	        if (!root) {
	            pop(&s, &root);
	            pop(&pc, &mark);
	        } else {
	            pop(&pc, &mark);
	            
	            if (mark == 0) {
	                push(&s, root->left);
	                push(&pc, 1);
	                push(&pc, 0);
	                arr[(*returnSize)++] = root->val;
	            } else if (mark == 1) {
	                push(&s, root->right);
	                push(&pc, 2);
	                push(&pc, 0);
	            } else {
	                pop(&s, &root);
	            }
	        }
	
	    }
	    
	    return arr;
	}

## 不精确模拟

下面第二种方式是不精确的实现:

	void push_to_leftmost(Stack **s, struct TreeNode *root)
	{
	    while (root) {
	        push(s, root);
	        root = root->left;
	    }
	}
	
	/**
	 * Return an array of size *returnSize.
	 * Note: The returned array must be malloced, assume caller calls free().
	 */
	int* postorderTraversal(struct TreeNode* root, int* returnSize)
	{
	    int             *arr;
	    long            mark;
	    Stack           *s;
	    struct TreeNode *last;
	    
	    create(&s);
	    arr = malloc(10000 * sizeof *arr);
	    *returnSize = 0;
	    last = NULL;
	    
	    push_to_leftmost(&s, root);
	    while (!empty(&s)) {
	        top(&s, &root);
	
	        if (!root->right || last == root->right) {
	            pop(&s, NULL);
	            arr[(*returnSize)++] = root->val;
	            last = root;
	        } else {
	            /* arr[(*returnSize)++] = root->val; */
	            push_to_leftmost(&s, root->right);
	        }
	    }
	    
	    return arr;
	}


如果撇开对数据的访问单看遍历过程, 会发现每遍历到一个节点时, 这个节点都会被入两次栈, 出两次栈, 为什么呢? 因为每一次函数调用, 传参和被调函数内的局部变量都会被压栈, 而上面的 `traverse` 里递归调用了自己两次, 所以就有了如下第三种方法:

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	typedef struct stack {
	    void           *c;
	    struct stack   *next;
	} Stack;
	
	void create(Stack **ptop)
	{
	    *ptop = NULL;
	}
	
	void push(Stack **ptop, void *c)
	{
	    Stack   *tmp;
	
	    tmp = malloc(sizeof *tmp);
	    tmp->c = c;
	    tmp->next = *ptop;
	
	    *ptop = tmp;
	}
	
	void pop(Stack **ptop, void **c)
	{
	    Stack   *tmp;
	
	    if (! *ptop) return;
	
	    tmp = *ptop;
	    *c = tmp->c;
	    *ptop = (*ptop)->next;
	    //free(tmp);
	}
	
	int empty(Stack **ptop)
	{
	   return *ptop == NULL; 
	}
	
	#define PUSH_LEFT(s, root)      \
	    do {                        \
	        while (root) {          \
	            push(&s, 0);        \
	            push(&s, root);     \
	            root = root->left;  \
	        }                       \
	    } while (0)
	
	/**
	 * Return an array of size *returnSize.
	 * Note: The returned array must be malloced, assume caller calls free().
	 */
	int* postorderTraversal(struct TreeNode* root, int* returnSize)
	{
	    int             *arr;
	    long            mark;
	    Stack           *s;
	    
	    create(&s);
	    arr = malloc(10000 * sizeof *arr);
	    *returnSize = 0;
	    
	    PUSH_LEFT(s, root);
	    while (!empty(&s)) {
	        pop(&s, &root);
	        pop(&s, &mark);
	        
	        if (!mark) {
	            push(&s, 1);
	            push(&s, root);
	            root = root->right;
	            
	            PUSH_LEFT(s, root); 
	        } else {
	            arr[(*returnSize)++] = root->val;
	        }
	    }
	    
	    return arr;
	}

## 参考

这位作者的解法, 展示了另一种入栈出栈方式

https://discuss.leetcode.com/topic/2917/accepted-code-explaination-with-algo
