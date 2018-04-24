title: leetcode Tag Tree (Easy)
slug: leetcode-tree-easy
date: 2015-09-16 21:25:48
tags: leetcode, algorithm, tree

## Binary Tree Level Order Traversal

下面是 DFS 法, 这题回头再用 BFS 做一遍

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	/**
	 * Return an array of arrays of size *returnSize.
	 * The sizes of the arrays are returned as *columnSizes array.
	 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
	 */
	
	void traverse(struct TreeNode *root, int depth, int ***arr, int **columnSizes, int *returnSize)
	{
	    if (!root) return;
	
	    if (*returnSize < depth + 1) {
	        *returnSize = depth + 1;
	        
	        /*
	         * Should initialise the one more allocated space to NULL (or 0)
	         */
	        *arr = realloc(*arr, (depth + 1) * sizeof(int *));
	        (*arr)[depth] = NULL;
	    
	        *columnSizes = realloc(*columnSizes, (depth + 1) * sizeof(int));
	        (*columnSizes)[depth] = 0;
	    }
	    
	    (*arr)[depth] = realloc((*arr)[depth], ((*columnSizes)[depth] + 1) * sizeof(int));
	    (*arr)[depth][(*columnSizes)[depth]] = root->val;
	    ++(*columnSizes)[depth];
	    
	    traverse(root->left, depth + 1, arr, columnSizes, returnSize);
	    traverse(root->right, depth + 1, arr, columnSizes, returnSize);
	}
	
	int** levelOrder(struct TreeNode* root, int** columnSizes, int* returnSize) {
	    int **arr;
	    
	    arr = NULL;
	    *returnSize = 0;
	    traverse(root, 0, &arr, columnSizes, returnSize);
	    
	    return arr;
	}

<!-- more -->

## Binary Tree Level Order Traversal II

Compare to `Binary Tree Level Order Traversal` which was resovled using prefix traversal, this just the postfix case.

(My solution of `Binary Tree Level Order Traversal`: https://discuss.leetcode.com/topic/59106/c-solution-using-dfs)

But, if want to using postfix traversal, we need to know the max depth in advance, so it seems that I have to do an extra traversal to get the max depth

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	/**
	 * Return an array of arrays of size *returnSize.
	 * The sizes of the arrays are returned as *columnSizes array.
	 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
	 */
	
	void traverse(struct TreeNode *root, int depth, int ***arr, int **columnSizes, int *returnSize)
	{
	    int     idx;
	    
	    if (!root) return;
	
	    traverse(root->left, depth + 1, arr, columnSizes, returnSize);
	    traverse(root->right, depth + 1, arr, columnSizes, returnSize);
	    
	    idx = *returnSize - depth - 1;
	    (*arr)[idx] = realloc((*arr)[idx], ((*columnSizes)[idx] + 1) * sizeof(int));
	    (*arr)[idx][(*columnSizes)[idx]] = root->val;
	    ++(*columnSizes)[idx];
	}
	
	void maxdepth(struct TreeNode *root, int depth, int ***arr, int **columnSizes, int *returnSize)
	{
	    if (!root) return;
	
	    if (*returnSize < depth + 1) {
	        *returnSize = depth + 1;
	        
	        /*
	         * Should initialise the one more allocated space to NULL (or 0)
	         */
	        *arr = realloc(*arr, (depth + 1) * sizeof(int *));
	        (*arr)[depth] = NULL;
	    
	        *columnSizes = realloc(*columnSizes, (depth + 1) * sizeof(int));
	        (*columnSizes)[depth] = 0;
	    }
	    
	    maxdepth(root->left, depth + 1, arr, columnSizes, returnSize);
	    maxdepth(root->right, depth + 1, arr, columnSizes, returnSize);
	}
	
	
	int** levelOrderBottom(struct TreeNode* root, int** columnSizes, int* returnSize) {
	    int **arr;
	    
	    arr = NULL;
	    *returnSize = 0;
	    maxdepth(root, 0, &arr, columnSizes, returnSize);
	    traverse(root, 0, &arr, columnSizes, returnSize);
	    
	    return arr;
	}

## isSameTree

我一开始写了一段比较撮的实现:

    void traverse(struct TreeNode *p, struct TreeNode *q, int *same)
    {
        if (!p && !q || !*same);
        else if (!q || !p) *same = 0;
        else if (p->val != q->val) *same = 0;
        else {
            traverse(p->left, q->left, same);
            traverse(p->right, q->right, same);
        }
    }

    bool isSameTree(struct TreeNode* p, struct TreeNode* q)
    {
        int same = 1;
        traverse(p, q, &same);
        return same;
    }

后来在 Disccus 中看到别人更精简的实现, 我更加意识到自己还是没有理解递归的精髓...

递归的优点就是, 在程序的运行过程中自动为我们开辟了额外的空间来保存我们的参数, 以及返回值, 所以我上面的解法中的 `same` 参数是一点意义也没有. 我总是不善于利用递归的返回值, 以后一定要把递归的返回值多加利用起来.

有效利用了返回值之后, 更 cleaner 一些的代码就是这样了:
	
	bool isSameTree(struct TreeNode* p, struct TreeNode* q)
	{
	    if (!p && !q) return true;
	    else if (!q || !p) return false;
	    else if (p->val != q->val) return false;
	    else
	        return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
	}

## Invert Binary Tree

做这道题的时候我突然意识到, 这题就是 Homebrew 的作者 Max Howell 面试 Google 时没有能在白板上徒手写出来的题目啊... 反转二叉树, 我现在毫无外界压力的情况下大概花了 3 分钟才写出来. 如果事先没做过这题, 而且有幸能进入和 Max Howell 同样的面试阶段, 也让我在白板上写, 我基本也跪了.

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	struct TreeNode* invertTree(struct TreeNode* root) {
	    struct TreeNode     *tmp;
	    if (root) {
	        tmp = invertTree(root->left);
	        root->left = invertTree(root->right);
	        root->right = tmp;
	    }
	    return root;
	}

## Symmetric Tree

一种方法是结合前面的 invertTree 与 sameTree, 先反转右子树, 然后判断左子树和反转后的右子树是否 same. 然后再次反转右子树恢复原状. 我最先想到的是这个方法, 但这个方法需要三次遍历, 按照经验来讲这肯定不是出题者想要的方法, 但还是贴出来吧.

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	
	struct TreeNode* invertTree(struct TreeNode* root) {
	    struct TreeNode     *tmp;
	    if (root) {
	        tmp = invertTree(root->left);
	        root->left = invertTree(root->right);
	        root->right = tmp;
	    }
	    return root;
	}
	
	bool isSameTree(struct TreeNode* p, struct TreeNode* q)
	{
	    if (!p && !q) return true;
	    else if (!q || !p) return false;
	    else if (p->val != q->val) return false;
	    else
	        return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
	}
	
	bool isSymmetric(struct TreeNode* root) {
	    bool sym;
	    
	    if (!root) return true;
	    
	    /* invert right subtree */
	    invertTree(root->right);
	    sym = isSameTree(root->left, root->right);
	    /* re-invert right subtree */
	    invertTree(root->right);
	    
	    return sym;
	}

另一种思路, 通过观察可以发现, 要判断这颗树是不是对称的, 实际上就是判断其左右子树是不是镜像的; 而左右子树若是镜像的, 那就需要:

1. 左右子树的根相等
2. 左子树的根的左子树和右子树的根的右子树成镜像; 左子树的根的右子树和右子树的根的左子树成镜像

上面的这两个条件就是我们代码的递归逻辑:

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	bool isMirror(struct TreeNode *p, struct TreeNode *q)
	{
	    if (!p && !q) return true;
	    else if (!p || !q) return false;
	    else if (p->val != q->val) return false;
	    else
	        return isMirror(p->left, q->right) && isMirror(p->right, q->left);
	}
	
	bool isSymmetric(struct TreeNode* root) {
	    if (!root) return true;
	    return isMirror(root->left, root->right);
	}

## Balanced Binary Tree

我一开始又陷入了 "不能好好利用返回值" 的弱点, 写出了以下代码. 这个代码不但没有好好利用返回值, 还非常的不协调. 因为当第一次发现有两子树高度差大于 1, 且 `*balanced` 被赋值 0 之后, 递归开始退出, 然而退出的过程中所有的上层递归都要再执行一遍下面注释符之内的内容, 而这个时候的 h1, h2 的值就是混乱的了, 因为受到 `! *balanced` 为 `true` 的影响, 原本高度不是 -1 的子树, 现在也会返回高度 -1 了.

	int traverse(struct TreeNode *r, int *balanced)
	{
	    int     h1, h2;
	    
	    if (!r || !*balanced) return -1;
	    
	    h1 = traverse(r->left);
	    h2 = traverse(r->right);
	    
        /***/
	    if (h1 - h2 > 1 || h1 - h2 < -1)
            *balanced = 0;
	    
	    if (h1 > h2)
	        return h1 + 1;
	    else
	        return h2 + 1;
        /***/
	}

	bool isBalanced(struct TreeNode* root) {
        int balanced = 1;
	    traverse(root, &balanced);
        return balanced;
	}

受 Discuss 区的启发, 我又写出了如下代码, 这回利用好返回值了. 而且这段代码还纠正了上面那段的另一个问题, 就是 `!r` 的时候返回 -1 还是 0, 其实返回 0 是更合理的, 因为 `h1 = traverse(r->left)` 这样的写法表明了 `traverse(r->left)` 的返回值是要作为 `r` 节点的高度的, 如果 `r->left` 为 NULL 的话也就说明 `r` 有可能是叶子节点, 叶子节点的高度自然是 0, 当然 `r` 是不是叶子节点还要看 `r->right`, `r` 的最终高度也是要取 `r->left` 和 `r->right` 高度的较大者.

也就是说我们实际上返回 `当前节点高度 + 1` 是比较合理的, 只有当出现不平衡的情况时我们直接返回 -1. 所以下面代码里的 `return h1 + 1`, `return h2 + 1` 是比较好理解的, `return -1 + 1` 就纯粹为了书写一致性了.

	int traverse(struct TreeNode *r)
	{
	    int     h1, h2;
	    
	    if (!r) return -1 + 1;
	    
	    h1 = traverse(r->left);
	    h2 = traverse(r->right);
	    
	    if ((h1 == -1 || h2 == -1)
	        || (h1 - h2 > 1 || h1 - h2 < -1))
	        return -1;
	    
	    if (h1 > h2)
	        return h1 + 1;
	    else
	        return h2 + 1;
	}
	bool isBalanced(struct TreeNode* root) {
	    return traverse(root) != -1;
	}

反思自己不能好好利用递归返回值的原因, 大概是在于我平时写函数时特别喜欢使用传出参数, 返回值只用来判断函数执行结果是成功还是失败, 以后一定一定得注意在递归里不能犯这种极不优雅的错误了.
