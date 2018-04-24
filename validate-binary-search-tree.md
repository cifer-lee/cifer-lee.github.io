title: 合法二叉搜索树的校验
date: 2016-11-20 15:56:29
slug: validate-binary-search-tree
category: algorithm
tags: leetcode, algorithm

## Valid Binary Search Tree

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	int max(struct TreeNode *root)
	{
	    while (root->right) root = root->right;
	    return root->val;
	}
	
	int min(struct TreeNode *root)
	{
	    while (root->left) root = root->left;
	    return root->val;
	}
	
	bool isValidBST(struct TreeNode* root) {
	    if (!root) return true;
	    
	    if (root->left) {
	        if (isValidBST(root->left) && root->val > max(root->left)) {
	        } else {
	            return false;
	        }
	    }
	    
	    if (root->right) {
	        if (isValidBST(root->right) && root->val < min(root->right)) {
	        } else {
	            return false;
	        }
	    }
	    
	    return true;
	}

<!-- more -->

## 更优解

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
	int traverse(struct TreeNode *root, int *max_out, int *min_out)
	{
	    int     rc1, rc2;
	    int     maxl, minl, maxr, minr;
	
	    if (!root) return true;
	
	    if (root->left) {
	        rc1 = traverse(root->left, &maxl, &minl);
	    } else {
	        rc1 = 1;
	        maxl = root->val;
	        minl = root->val;
	    }
	
	    if (root->right) {
	        rc2 = traverse(root->right, &maxr, &minr);
	    } else {
	        rc2 = 1;
	        maxr = root->val;
	        minr = root->val;
	    }
	
	    if (max_out) *max_out = maxr;
	    if (min_out) *min_out = minl;
	
	    if ((rc1 == 1 || rc1 && root->val > maxl) &&
	            (rc2 == 1 || rc2 && root->val < minr)) {
	        return 2;
	    } else {
	        return 0;
	    }
	}
	
	bool isValidBST(struct TreeNode* root) {
	    return traverse(root, NULL, NULL);
	}

一开始犯了如下的错误:

	/**
	 * Definition for a binary tree node.
	 * struct TreeNode {
	 *     int val;
	 *     struct TreeNode *left;
	 *     struct TreeNode *right;
	 * };
	 */
    bool traverse(struct TreeNode *root, int *max, int *min)
    {
        if (!root) return true;
	    
	    if (root->left) {
	        if (isValidBST(root->left) && root->val > max) {
	        } else {
	            return false;
	        }
	    } else {
	        *min = root->val;
	    }
	    
	    if (root->right) {
	        if (isValidBST(root->right) && root->val < min) {
	        } else {
	            return false;
	        }
	    } else {
	        *max = root->val;
	    }
	    
	    return true;
    }
	
	bool isValidBST(struct TreeNode* root) {
        int max, min;
        return traverse(root, &max, &min);
	}

## 附


下面几个解法很不错:

利用中序遍历得到有序序列的思想, 思路清晰

https://discuss.leetcode.com/topic/4659/c-in-order-traversal-and-please-do-not-rely-on-buggy-int_max-int_min-solutions-any-more

精彩的思路, 自上而下. 与我的思路截然相反, 我的思路是自下而上的. 但显然, 我的思路逊一筹

https://discuss.leetcode.com/topic/18573/c-simple-recursive-solution
