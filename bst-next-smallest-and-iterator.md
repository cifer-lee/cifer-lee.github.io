title: BST 获取指定元素的下一元素与 BST 迭代器
slug: bst-next-smallest-and-iterator
date: 2016-10-15 22:20:19
updated: 2016-10-22 17:36:20
category: algorithm
tags: binary search tree, algorithm

## 在 BST 中获取指定元素的下一元素

给定一个元素, 从 BST 中获取刚好比它大的下一个元素, 是我最近工作中碰到的一个需求.

这里有一个问题需要先确认一下, 就是给定的元素可能原本就是从 BST 查询出来的, 也可能是任意给定的, 未必存在于 BST 中, 那么当给定的元素不在 BST 中时, 我们还要不要返回刚好比这个元素大的元素呢? 我认为是要的, 这显然是更加符合大多数业务逻辑的, 因为显然上层应用调用这个接口, 目的不是判断给定的元素是否在树中存在, 而是不管这个元素在树中存不存在, 都给我返回刚好比它大的元素.

我们的 BST 实现中带有父节点的指针, 所以实现起来比较容易, 当时时间仓促我借着父节点指针的便利写出了如下的实现:

	struct TreeNode *next(struct TreeNode *root, int val)
	{
	    struct TreeNode *t;
	
	    if (!root) {
	    } else if (root->val > val) {
	        root = next(root->left, val);
	    } else if (root->val < val) {
	        root = next(root->right, val);
	    } else if (root->right) {
	        t = root->right;
	        while (t->left) t = t->left;
	        root = t;
	    } else {
            do {
                t = root->parent;
                if (!t || t->val > val) break;
            } while (1);
	        root = t;
	    }
	
	    return root;
	}

<!-- more -->

后来我在领悟了递归的真谛之后重新又想起了这个问题, 想到实际上这个问题是不需要借助父节点指针的, 而且上面这个实现有个致命的缺陷和一个十分不优雅的设计. 缺陷是, 当给定的值不在 BST 中时, 即时 BST 中有刚好比给定值大的节点, 该实现也会返回 NULL; 不优雅之处是, 在最后一个条件分支处引用了节点父指针, 这使得程序的数据流倒置, 因为此时程序控制流仍处于深层次的递归中没有返回, 数据流却向上回溯引用了父级节点, 现在看来这真是极度不优雅的行为.

算法问题, 精益求精, 所以我一定要重新写一个精炼一些的实现, 如下:

	struct TreeNode *next(struct TreeNode *root, int val)
	{
	    struct TreeNode *t;
	
	    if (!root) {
	    } else if (root->val > val) {
	        t = next(root->left, val);
	        if (t) root = t;
	    } else if (root->val < val) {
	        root = next(root->right, val);
	    } else if (root->right) {
	        t = root->right;
	        while (t->left) t = t->left;
	        root = t;
	    } else {
	        root = NULL;
	    }
	
	    return root;
	}

注意 `root->val > val` 这个分支的处理, `if (t) root = t` 这句的处理是很重要的, 它代表这如果深层递归返回的是 NULL, 那么我就返回当前节点作为给定元素的下一个元素.

上述逻辑中可能会有两处地方返回 NULL, 分别是:

1. 当给定元素在 BST 中不存在时, 查找过程最终就会落入第一个条件分支, 此时返回 NULL. 这种情况包含了整棵树是棵空树的情况
2. 当给定元素在 BST 中搜索到了, 但是搜到的节点不存在右子树

这两种情况的共性就是, 它们都代表说 "好了, 再递归下去你也不可能找到下一个元素的, 下一个元素在上面不在下面", 于是这两种情况我们就直接返回 NULL, 这样递归上层看到返回值为 NULL, 就知道下层递归一无所获了. 那么既然下一个元素在上层, 怎么把它揪出来呢? 我们知道递归是一个压栈的过程, 在递归往下搜索的过程中, 这条搜索路径已经存在于栈中了, 而回溯就是出栈的过程, 所以我们只需要在回溯过程中判断出栈值和给定值的大小, 发现的第一个比给定值大的就是我们要找的下一个元素.

在把上面那些想法转化成代码之前, 我们还需要思考一下, 找到下一个元素的时候, 这个回溯有可能是沿着右分支回溯上去的吗? 显然是不可能的, 我们要找的下一个元素是比给定元素要大的元素, 而毫无疑问回溯过程中沿着右分支的节点一定都比给定元素小, 否则之前递归时就不会进入其右分支.

所以在代码上我们只需要在第二个条件分支里判断一下返回值, 如果是 NULL, 那说明我自己就是刚好比给定元素大的下一个元素了.

## BST 的迭代器

由上面的问题引申出来的问题就是 BST 迭代器, 这也是 leetcode 上的 173 号题目, 借助上述的 `next` 的方法, 是可以实现 BST 迭代器的, 实现如下, 但是显然, 这个实现是低效的, 丑陋的, 毫无优雅可言的.

	/**
	 * Definition for binary tree
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	struct BSTIterator {
	    struct TreeNode *root;
	    struct TreeNode *curr;
	};
	
	struct BSTIterator *bstIteratorCreate(struct TreeNode *root) {
	    struct BSTIterator  *i;
	    
	    i = malloc(sizeof *i);
	    i->root = root;
	    i->curr = NULL;
	    return i;
	}
	
	struct TreeNode *next(struct TreeNode *root, int val)
	{
	    struct TreeNode *t;
	    
	    if (!root) {
	        return NULL;
	    } else if (root->val > val) {
	        t = next(root->left, val);
	        if (t) return t;
	        else return root;
	    } else if (root->val < val) {
	        return next(root->right, val);
	    } else if (root->right) {
	        t = root->right;
	        while (t->left) t = t->left;
	        return t;
	    } else {
	        return NULL;
	    }
	}
	
	/** @return whether we have a next smallest number */
	bool bstIteratorHasNext(struct BSTIterator *iter) {
	    struct TreeNode *d;
	    
	    if (!iter->curr) {
	        d = iter->root;
	        while (d && d->left) d = d->left;
	        if (!d) {
	            return false;
	        } else {
	            return true;
	        }
	    } else {
	        if (next(iter->root, iter->curr->val)) return true;
	        else return false;
	    }
	}
	
	/** @return the next smallest number */
	int bstIteratorNext(struct BSTIterator *iter) {
	    struct TreeNode *t;
	    
	    if (!iter->curr) {
	        t = iter->root;
	        while (t && t->left) t = t->left;
	        if (!t) {
	            return 0;
	        } else {
	            iter->curr = t;
	            return t->val;
	        }
	    } else {
	        iter->curr = next(iter->root, iter->curr->val);
	        
	        if (iter->curr) return iter->curr->val;
	        else return 0;
	    }
	}
	
	/** Deallocates memory previously allocated for the iterator */
	void bstIteratorFree(struct BSTIterator *iter) {
	    if (iter) free(iter);
	}
	
	/**
	 * Your BSTIterator will be called like this:
	 * struct BSTIterator *i = bstIteratorCreate(root);
	 * while (bstIteratorHasNext(i)) printf("%d\n", bstIteratorNext(i));
	 * bstIteratorFree(i);
	 */


迭代器和上面说的找出比给定元素刚好大的下一元素虽有相似之处, 但其是是不同的问题, 找下一元素每一次操作都是相互独立的, 这次找 9 的下一元素, 下次可能找 2 的下一元素, 两次之间没有关联, 所以每次都需要从头开始重新遍历, 所以每一次查找都是一个 O(n) 过程. 而迭代操作是有上下文语境的, 在一次迭代 session 中, 可以有一个游标指示当前迭代的位置, 并且这个位置肯定是只会向前不会后退, 所以每一次迭代理论上都是 O(1) 过程, 所以这个过程肯定也不是用递归实现了.

仔细想想其实 BST 的迭代器, 每一步迭代也就是中序遍历的每一步嘛, 所以实际上 BST 迭代器的实现就是非递归中序遍历过程的拆分, 我在 [树的深度优先遍历](/tree-dfs/) 这篇文章里有写过中序遍历树的非递归写法, 拆开来用在这里就可以了

> Before I come up with this solution, I really draw a lot binary trees and try inorder traversal on them. We all know that, once you get to a TreeNode, in order to get the smallest, you need to go all the way down its left branch. So our first step is to point to pointer to the left most TreeNode. The problem is how to do back trace. Since the TreeNode doesn't have father pointer, we cannot get a TreeNode's father node in O(1) without store it beforehand. Back to the first step, when we are traversal to the left most TreeNode, we store each TreeNode we met ( They are all father nodes for back trace).

	typedef struct stack {
	    void         *c;
	    struct stack *next;
	} Stack;
	
	void stackCreate(Stack **ptop)
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
	
	int stackEmpty(Stack **ptop)
	{
	   return *ptop == NULL; 
	}
	
	/**
	 * Definition for binary tree
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	struct BSTIterator {
	    struct TreeNode *root;
	    struct TreeNode *curr;
	    Stack           *stack;
	};
	
	struct BSTIterator *bstIteratorCreate(struct TreeNode *root) {
	    struct BSTIterator  *iter;
	    
	    iter = malloc(sizeof *iter);
	    iter->curr = iter->root = root;
	    stackCreate(&iter->stack);
	    
	    while (iter->curr) {
	        push(&iter->stack, iter->curr);
	        iter->curr = iter->curr->left;
	    }
	    
	    return iter;
	}
	
	/** @return whether we have a next smallest number */
	bool bstIteratorHasNext(struct BSTIterator *iter) {
	    return !stackEmpty(&iter->stack);
	}
	
	/** @return the next smallest number */
	int bstIteratorNext(struct BSTIterator *iter) {
	    int     val;
	    
	    pop(&iter->stack, &iter->curr);
	    val = iter->curr->val;
	    iter->curr = iter->curr->right;
	    while (iter->curr) {
	        push(&iter->stack, iter->curr);
	        iter->curr = iter->curr->left;
	    }
	    
	    return val;
	}
	
	/** Deallocates memory previously allocated for the iterator */
	void bstIteratorFree(struct BSTIterator *iter) {
	    if (iter) free(iter);
	}
	
	/**
	 * Your BSTIterator will be called like this:
	 * struct BSTIterator *i = bstIteratorCreate(root);
	 * while (bstIteratorHasNext(i)) printf("%d\n", bstIteratorNext(i));
	 * bstIteratorFree(i);
	 */
