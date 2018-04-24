title: AVL Tree 的实现
slug: avl-tree
tags: balanced binary tree, avl tree, algorithm
date: 2016-07-11 22:20:19

平衡二叉树, 首先需要对 "平衡" 作出一个定义, AVL, Red-black 对于 "平衡" 的定义各不相同. 本文讨论 AVL Tree, AVL Tree 对于平衡的定义是任何一个节点的左右子树高度差不能大于 1.

要把 AVL Tree 在一篇文章里说清楚是挺麻烦的, 所以这里上两个版本的实现, 以后慢慢完善此文. 一般有两种实现, 一种是节点存储高度信息, 一种是节点存储平衡因子.

下面的分别列出高度版和平衡因子版的实现, 高度版算上注释不到 200 行代码, 当然, 粗枝大叶都没包含, 只有最核心的代码.

## 高度版实现

### AVL Tree (过渡版)

	struct TreeNode {
	    int             val;
	    int             height;
	    struct TreeNode *left;
	    struct TreeNode *right;
	};
	
	static inline int _h(struct TreeNode *node)
	{
	    return node ? node->height : -1;
	}
	
	int height(struct TreeNode *root)
	{
	    int h1, h2;
	
	    h1 = height(root->left);
	    h2 = height(root->right);
	#define MAX(h1, h2) ((h1) > (h2) ? (h1) : (h2))
	    return MAX(h1, h2) + 1;
	}
	
	/*
	 * \return new root
	 */
	struct TreeNode *rotate(struct TreeNode *root, int dir)
	{
	    struct TreeNode *tmp;
	
	    /* dir != 0, clockwise */
	    if (dir) {
	        tmp = root->left;
	        root->left = tmp->right;
	        tmp->right = root;
	    } else {
	        tmp = root->right;
	        root->right = tmp->left;
	        tmp->left = root;
	    }
	
	    /* must first re-caculate root's height */
	    root->height = height(root);
	    tmp->height = height(tmp);
	
	    return tmp;
	}
	
	struct TreeNode *insert(struct TreeNode *root, int val)
	{
	    int hleft, hright;
	    struct TreeNode *tmp;
	
	    if (!root) {
	        root = malloc(sizeof *root);
	        root->val = val;
	        root->left = root->right = NULL;
	    } else if (root->val > val) {
	        root->left = insert(root->left, val);
	    } else if (root->val < val) {
	        root->right = insert(root->right, val);
	    }
	
	    /* Node's height is the larger height of left subtree and right subtree
	     * plus 1 */
	    hleft = height(root->left);
	    hright = height(root->right);
	    root->height = MAX(hleft, hright) + 1;
	
	    /*
	     * if left subtree's height is 1 more greater than right subtree, and
	     * the root's val of left subtree greater than inserted val, then unbalanced
	     * condition caused by new node inserted into left subtree's left child
	     *
	     * Other cases can be deduced in the same way
	     */
	    if (hleft - hright > 1) {
	        if (root->left->val > val) {
	            root = rotate(root, 1);
	        } else {
	            /* first rotation */
	            root->left = rotate(root->left, 0);
	            /* second rotation */
	            root = rotate(root, 1);
	        }
	    } else if (hleft - hright < -1) {
	        if (root->right->val < val) {
	            root = rotate(root, 0);
	        } else {
	            /* first rotation */
	            root->right = rotate(root->right, 1);
	            /* second rotation */
	            root = rotate(root, 0);
	        }
	    }
	
	    return root;
	}
	
	struct TreeNode *find(struct TreeNode *root, int val)
	{
	    if (!root)
	        return NULL;
	    else if (root->val > val)
	        return find(root->left, val);
	    else if (root->val < val)
	        return find(root->right, val);
	    else
	        return root;
	}
	
	struct TreeNode *delete(struct TreeNode *root, int val)
	{
	    struct TreeNode *tmp;
	
	    if (!root) ;
	    else if (root->val > val) {
	        root->left = delete(root->left, val);
	        if (_h(root->left) - _h(root->right) < -1) {
	            if (_h(root->right) - _h(root->right->right) == 1) {
	                root = rotate(root, 0);
	            } else {
	                root->right = rotate(root->right, 1);
	                root = rotate(root, 0);
	            }
	        }
	    } else if (root->val < val) {
	        root->right = delete(root->right, val);
	        if (_h(root->left) - _h(root->right) > 1) {
	            if (_h(root->left) - _h(root->left->left) == 1) {
	                root = rotate(root, 1);
	            } else {
	                root->left = rotate(root->left, 0);
	                root = rotate(root, 1);
	            }
	        }
        } else if (root->left && root->right) {
	        tmp = root->right;
	        while (tmp->left) tmp = tmp->left;
	        root->val = tmp->val;
	        root->right = delete(root->right, root->val);

	        if (_h(root->left) - _h(root->right) > 1) {
	            if (_h(root->left) - _h(root->left->left) == 1) {
	                root = rotate(root, 1);
	            } else {
	                root->left = rotate(root->left, 0);
	                root = rotate(root, 1);
	            }
	        }
	    } else if (root->left || root->right) {
	        tmp = root->left ? root->left : root->right;
	        free(root);
	        root = tmp;
	    } else {
	        free(root);
	        root = NULL;
	    }
	
	    /* If the deleted is an orphan node */
	    if (root)
	        root->height = height(root);
	
	    return root;
	}

<!-- more -->

关于二叉树节点删除有两种思路, 一个是用右子树最左节点代替被删节点, 然后删除右子树最左节点; 另一个是用右子树根节点代替被删节点, 然后被删节点的左子树成为原右子树最左节点左子树. 第一种方法好一些, 因为涉及更少的指针变动, 另外就是第二种方法如果在 AVL tree 中也使用的话会很麻烦, 因为涉及到旋转.

### AVL Tree (最终版)

	struct TreeNode {
	    int             val;
	    int             height;
	    struct TreeNode *left;
	    struct TreeNode *right;
	};

	/*
	 * Get height of node
	 */
	static inline int _h(struct TreeNode *node)
	{
	    return node ? node->height : -1;
	}
	
	/*
	 * Caculate height of node, this function will update node's height
	 * internally, however, this doesn't prevent you updating it using the
	 * return value
	 *
	 * \return new height
	 */
	int height(struct TreeNode *root)
	{
	    int h1, h2;
	
	    h1 = _h(root->left);
	    h2 = _h(root->right);
	
	#define MAX(h1, h2) ((h1) > (h2) ? (h1) : (h2))
	    root->height = MAX(h1, h2) + 1;
	    return root->height;
	}
	
	/*
	 * \return new root
	 */
	struct TreeNode *rotate(struct TreeNode *root, int dir)
	{
	    struct TreeNode *tmp;
	
	    /* dir != 0, clockwise */
	    if (dir) {
	        tmp = root->left;
	        root->left = tmp->right;
	        tmp->right = root;
	    } else {
	        tmp = root->right;
	        root->right = tmp->left;
	        tmp->left = root;
	    }
	
	    /* must first re-caculate root's height */
	    root->height = height(root);
	    tmp->height = height(tmp);
	
	    return tmp;
	}
	
	/*
	 * if left subtree's height is 1 more greater than right subtree, and
	 * the root's val of left subtree greater than inserted val, then unbalanced
	 * condition caused by new node inserted into left subtree's left child
	 *
	 * Other cases can be deduced in the same way
	 */
	struct TreeNode *insert(struct TreeNode *root, int val)
	{
	    if (!root) {
	        root = malloc(sizeof *root);
	        root->val = val;
	        root->left = root->right = NULL;
	    } else if (root->val > val) {
	        root->left = insert(root->left, val);
	
	        if (_h(root->left) - _h(root->right) > 1) {
	            if (root->left->val > val) {
	                root = rotate(root, 1);
	            } else {
	                root->left = rotate(root->left, 0);
	                root = rotate(root, 1);
	            }
	        }
	    } else if (root->val < val) {
	        root->right = insert(root->right, val);
	
	        if (_h(root->left) - _h(root->right) < -1) {
	            if (root->right->val < val) {
	                root = rotate(root, 0);
	            } else {
	                root->right = rotate(root->right, 1);
	                root = rotate(root, 0);
	            }
	        }
	    } /* val is already in the tree, do nothing */
	
	    root->height = height(root);
	
	    return root;
	}
	
	struct TreeNode *find(struct TreeNode *root, int val)
	{
	    if (!root)
	        return NULL;
	    else if (root->val > val)
	        return find(root->left, val);
	    else if (root->val < val)
	        return find(root->right, val);
	    else
	        return root;
	}
	
	struct TreeNode *delete(struct TreeNode *root, int val)
	{
	    struct TreeNode *tmp;
	
	    if (!root) ;
	    else if (root->val > val) {
	        root->left = delete(root->left, val);
	        if (_h(root->left) - _h(root->right) < -1) {
	            if (_h(root->right) - _h(root->right->right) == 1) {
	                root = rotate(root, 0);
	            } else {
	                root->right = rotate(root->right, 1);
	                root = rotate(root, 0);
	            }
	        }
	    } else if (root->val < val || root->left && root->right) {
	        if (root->val < val) {
	            root->right = delete(root->right, val);
	        } else {
	            tmp = root->right;
	            while (tmp->left) tmp = tmp->left;
	            root->val = tmp->val;
	            root->right = delete(root->right, root->val);
	        }
	
	        if (_h(root->left) - _h(root->right) > 1) {
	            if (_h(root->left) - _h(root->left->left) == 1) {
	                root = rotate(root, 1);
	            } else {
	                root->left = rotate(root->left, 0);
	                root = rotate(root, 1);
	            }
	        }
	    } else if (root->left || root->right) {
	        tmp = root->left ? root->left : root->right;
	        free(root);
	        root = tmp;
	    } else {
	        free(root);
	        root = NULL;
	    }
	
	    /* If the deleted is an orphan node */
	    if (root)
	        root->height = height(root);
	
	    return root;
	}

其中注意, 上述代码中, 对于被删节点是单臂节点的情况, 我们放在了 `else if (root->left || root->right)` 这个分支中去处理, 实际上因为我们对于删除双臂节点采取的是 "取右子树上的最左节点覆盖被删节点" 的策略, 这个策略是不在乎左子树是否存在的, 只要右子树存在, 我们就可以取其最左节点的值覆盖当前节点, 然后删除右子树上的最左节点. 所以实际上我们可以把上述对删除单臂和双臂节点的策略调整为如下:

    ...
    } else if (root->val < val || root->right) {
	    if (root->val < val) {
	        root->right = delete(root->right, val);
	    } else {
	        tmp = root->right;
	        while (tmp->left) tmp = tmp->left;
	        root->val = tmp->val;
	        root->right = delete(root->right, root->val);
        }

	    if (_h(root->left) - _h(root->right) > 1) {
	        if (_h(root->left) - _h(root->left->left) == 1) {
	            root = rotate(root, 1);
	        } else {
	            root->left = rotate(root->left, 0);
	            root = rotate(root, 1);
	        }
	    }
    } else if (root->left) {
        tmp = root->left;
        free(root);
        root = tmp;
    } else {
        free(root);
        root = NULL;
    }
    ...

而这样一来, 后两个分支又可以合并为:

    ...
    } else if (root->val < val || root->right) {
	    if (root->val < val) {
	        root->right = delete(root->right, val);
	    } else {
	        tmp = root->right;
	        while (tmp->left) tmp = tmp->left;
	        root->val = tmp->val;
	        root->right = delete(root->right, root->val);
        }

	    if (_h(root->left) - _h(root->right) > 1) {
	        if (_h(root->left) - _h(root->left->left) == 1) {
	            root = rotate(root, 1);
	        } else {
	            root->left = rotate(root->left, 0);
	            root = rotate(root, 1);
	        }
	    }
    } else {
        tmp = root->left;
        free(root);
        root = tmp;
    }
    ...


## 平衡因子实现

个人觉得平衡因子的实现相对于高度版的实现更麻烦一些, 不如高度版的那么直观.

### 插入过程中的关键点有如下那么几个:

#### 插入新节点后 (先假设不涉及旋转), 如何正确的判断子树的深度是否发生了变化 (如何判断节点平衡因子是否需要更新)

可以肯定的是, 节点插入后, 其直接父节点的平衡因子一定需要调整, 更高层的父节点平衡因子的情况要根据回溯路径上其它父节点的情况而定, 以下图为例

                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             /
            H

假设 H 是新插入的节点, 此时 D 的平衡因子一定会变 (从 0 -> 1), B 也会跟着从 0 -> 1, A 也会跟着从 0 -> 1

                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             /     \
            H       I

此时跟着再插入一个 I, E 一定会变, 从 0 -> -1, B 也会跟着从 1 -> 0, 由于 B 的高度没有变化, 所以 A 的平衡因子不变

                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             / \   \
            H   K   I


此时跟着再插入一个 K, D 一定会变, 从 1 -> 0, 由于 D 高度没有变化, 所以 B 的平衡因子不会变, 由于 B 的平衡因子没变, 所以 A 的平衡因子不变

由此可见, 回溯路径上, 下级节点平衡因子不变, 或者从 1 (或 -1) 变为 0 时, 上级节点的平衡因子也不变; 而如果下级平衡因子从 0 变为 1 (或 -1), 则上级节点平衡因子就需要变化

#### 何时进行旋转, 平衡因子达到多少时应该进行旋转 (关键是正确的判断子树的深度有没有变化)

#### 节点插入后, 如何判断子树是否发生了旋转

这个问题很好解决, 如果子树发生了旋转, 那么子树的根肯定要变的, 所以只要在进入子树时保存一下子树的根, 回溯回来时和返回的子树的新根比较一下, 相同就说明子树没转, 否则就说明子树旋转了.

这里有一个情况需要注意, 就是插入节点回溯第一步, 这时候父级保存的值 (NULL) 和返回的值 (非 NULL) 肯定也是不一样的, 但这并不代表旋转了.

#### 节点插入后 (涉及旋转), 如何正确的判断子树的深度是否发生了变化 (如何判断节点平衡因子是否需要更新)

以如下的二叉树为例:

                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             /
            H

1. 单旋情况

   B 平衡因子为 1, 其左子树 D 平衡因子为 1, D 高度为 1. 现在插入新元素 K:

                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             /
            H
           /
          K
                     |
                     |
                     V
 
                     A
                 /       \
                B         C
               / \       / \
              H   E     F   G
             / \
            K   D

H, K, D 平衡因子都会变为 0, B 的左子树 (现在是 H) 高度为 1, 和之前相比高度没变, 所以 B 平衡因子不变.

    
2. 双旋情况

   双旋有两种情况, 原树如下:

                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /\      \
            /m \      C 
            ----   /     \
                  /\n    /\k
                  --     --

    1) 假设新节点插在了 n 子树, 

                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /\      \
            /m \      C 
            ----   /     \
                  /\     /\k
                 / n\    --
                 ----
                        |
                        |  (旋转)
                        V

                        C
                   /         \
                  /           \
                 B             A
              /     \        /   \
             /\     /\      /\k  /\
            /m \   / n\     --  / h\
            ----   ----         ----

    这种情况下整颗树的高度没变,  A 的平衡因子由 +1 变为 -1, B 由 -1 变为 0, C 保持 0 没变

    2) 假设新节点插在了 k 子树, 

                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /\      \
            /m \      C 
            ----   /     \
                  /\n    /\
                  --    / k\
                        ----

                        |
                        |  (旋转)
                        V

                        C
                   /         \
                  /           \
                 B             A
              /     \        /   \
             /\     /\n     /\   /\
            /m \    --     / k\ / h\
            ----           ---- ----

    这种情况下整颗树的高度依然没变,  A 的平衡因子由 +1 变为 0, B 由 -1 变为 +1, C 保持 0 没变

综上可知, 对插入操作, 不管是单旋还是双旋, 都不会改变子树高度, 因此父节点也就不需要更新平衡因子.

#### 旋转操作对平衡因子的影响

1. 单旋情况, 旋转轴心节点的平衡因子的变化, 与其子节点的平衡因子有关
2. 双旋情况, 旋转轴心节点的平衡因子的变化, 与其子节点的左/右子节点的平衡因子有关

相比高度版, 最大的不同就是, 高度版, 旋转操作后的新高度可以省事的用 `height()` 方法来重新计算; 但是平衡因子版, 旋转操作后的新平衡因子需要视其子节点的平衡因子分情况处理, 这是比高度版麻烦的地方.

### 删除过程中的关键点有如下那么几个:

#### 和高度版一样, 对所有节点的删除最终都可以转化为对叶子节点或者对单臂节点的删除

#### 节点删除后 (先假设不涉及旋转), 如何正确的判断子树的深度是否发生了变化 (如何判断节点平衡因子是否需要更新)

1. 被删节点是叶子节点, 则其直接父节点的平衡因子一定需要调整, 更高层的父节点平衡因子的情况要根据被删节点回溯路径上其它父节点的情况而定, 以下图为例

     ```
                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             / \   \   /   / \
            H  I   J  K   L   M
                         /     \
                        N       O
     ```

     a. 当节点 H 被删时, D 的平衡因子一定会变, 但是 B 不变, 因为 D 从 0 -> -1, D 的高度没有变化, B 不变的话, A 自然也不会变
     b. 而如果是 J 被删除, E 一定会变, B 也会跟着变, 因为 E 从 -1 -> 0, E 的高度发生了变化, B 从 0 -> 1, 而 A 由于 B 的高度没变, 所以 A 的平衡因子不变
     c. 由此可见, 回溯路径上, 下级节点平衡因子不变, 或者从 0 变为 1 (或 -1), 时, 上级节点的平衡因子也不变; 而如果下级平衡因子从 1 (或 -1) 变为 0, 则上级节点平衡因子就需要变化

2. 被删节点是单臂节点 (注意着隐含了着这个单臂节点只有一个子树并且子树高度必定为 1), 对于被删节点的直接父节点, 其平衡因子一定发生变化 (是否是直接父节点)

     a. 如果 E 被删除, 则实际上是 J 被删除, 这就回到了删除叶子节点的情况. 所以平衡因子的变化规律和叶子节点被删除时是一样的
     b. 如果是 L 被删除, 则 G 的平衡因子一定会变, 从 0 -> -1, 由于 G 的高度没变, C 的平衡因子也不会变, C 的平衡因子不变, 则 A 的平衡因子也不会变
     c. 如果 F 被删除, 则 C 的平衡因子一定会变, 从 -1 -> 0, 由于 C 的高度变了, A 的平衡因子也会跟着变
     d. 由此可见, 单臂节点被删除, 回溯路径上的节点平衡因子的变化规律和叶子节点被删除是一样的, 即:

     > 回溯路径上, 下级节点平衡因子不变, 或者从 0 变为 1 (或 -1), 时, 上级节点的平衡因子也不变; 而如果下级平衡因子从 1 (或 -1) 变为 0, 则上级节点平衡因子就需要变化

     所以结论就是, 所有的删除实际上都是删除叶子或者删除单臂, 所以所有的删除对平衡因子的处理方式都是相同的.

#### 子树深度发生变化后 (需要更新节点平衡因子), 如何判断其是否需要旋转, 单旋还是双旋

  1. 当节点平衡因子需要减 1 (暗示回溯自左子树), 但当前已经是 -1 了 (暗示右子树一定不为空), 则需要旋转. 当右孩子平衡因子为 0 或 -1 时, 逆时针单旋. 当右孩子为 1 时, 双旋
  2. 当节点平衡因子需要加 1 (暗示回溯自右子树), 但当前已经是 1 了 (暗示左子树一定不为空), 则需要旋转. 当左孩子平衡因子为 0 或 1 时, 顺时针单旋. 当左孩子为 -1 时, 双旋

  注意上述处理单旋的时候, 把左 (右) 孩子平衡因子为 0 的情况算进了单旋, 这和高度版本中 "当发现需要旋转时, 回溯自右子树 (左子树), 优先判断当前节点高度和右孩子 (左孩子) 高度是否正好相差 1, 来决定是单旋还是双旋" 的策略是一样的.

                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             / \   \       / \
            H  I   J      L   M

  以上图为例, 当删除 F 时, C 需要减 1, 且显然此时 C 已经是 -1 了, 需要旋转, 单旋还是双旋呢? 看一下 G, G 是 0. 假设我们采取双旋的话, 变化过程将是如下这样的:

                     A
                 /       \
                B         C
               / \         \
              D   E         G
             / \   \       / \
            H  I   J      L   M

                     |
                     |
                     V

                     A
                 /       \
                B         C
               / \         \
              D   E         L
             / \   \         \
            H  I   J          G
                               \
                                M

                     |
                     |
                     V

                     A
                 /       \
                B         L
               / \       / \
              D   E     C   G
             / \   \         \
            H  I   J          M


  而假设我们采取单旋的话, 变化过程将直接如下:
           
                     A
                 /       \
                B         C
               / \         \
              D   E         G
             / \   \       / \
            H  I   J      L   M

                     |
                     |
                     V

                     A
                 /       \
                B         G
               / \       / \
              D   E     C   M
             / \   \     \    
            H  I   J      L    

  可以看到, 最终都能得到正确的结果, 但是显然单旋是更高效的选择.

#### 节点删除后 (涉及旋转), 如何正确的判断子树的深度是否发生了变化 (如何判断节点平衡因子是否需要更新)

  当不平衡的节点发生旋转后, 回溯中其直接父节点会发现孩子节点的指针发生了变化. (回溯过程中, 会发现孩子节点指针变化的情况有两种, 一种就是旋转发生时, 另一种就是被删节点是左单臂节点时 (当将要被删的节点是右单臂节点时, 删除动作会被转移到此节点的最左叶子节点上)).

  接上面的例子, 我们看看由于旋转而导致的孩子节点变化. 当 F 被删除时, 过程如下:

                     A
                 /       \
                B         C
               / \       / \
              D   E     F   G
             / \   \       / \
            H  I   J      L   M

                     |
                     |
                     V

                     A
                 /       \
                B         G
               / \       / \
              D   E     C   M
             / \   \     \    
            H  I   J      L    


  此时, 对于 A 来说, 其右孩子是变了的, 然而右子树的高度并没有变化, 旋转前是 2, 旋转后依然是 2. 然后如果这时我们再将 M 删除, 则这颗 avl 树就会再次经历两次旋转, 变成如下这样:

                     A
                 /       \
                B         L
               / \       / \
              D   E     C   G
             / \   \         
            H  I   J      

  现在情况就不一样了, A 的右孩子变了, 并且右子树的高度也由 2 变成了 1. 如此一来, A 的平衡因子也就需要更新了.

  那么旋转之后, 右子树的高度何时会变何时不变呢? 有什么规律呢? 这个问题需要解决了下面的问题才能解答.

#### 旋转后, 如何正确的更新平衡因子

  细列的话, 旋转有四种: 顺单, 逆单, 顺逆双, 逆顺双. 由高度版本的经验我们直到实际上有两种情况是重复的, 所以我们只看顺单和逆顺双这两种情况. 实际上删除节点导致的不平衡的分析几乎是可以完全模拟为插入节点导致的不平衡分析. 这里为了方便表述, 这里祭出两张发源于 *DSAA* 的神级图示, 首先是顺单的情况.

  1)

                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /      /\
            /\     / k\
           /  \    ----
          / m  \ 
         /      \
         --------


  2)
                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /       \
            /\       /\
           /  \     /  \
          / m  \   / k  \
         /      \ /      \
         -------- --------


  这图在 *DSAA* 里是解释 avl tree 插入新元素时节点的高度变化的, 放在这里来解释删除元素时的平衡因子变化也是可以的, 这图不难看懂, 实在不懂可以先看看 *DSAA* 的解释, *DSSA* 中的这个图示相对清晰一些, 但是这里得益于我逆天的符号画图技能, 其实还原的也是相当不错的.

  稍微解释一下, 在上图中, A 是不平衡节点, 导致 A 不平衡的原因是节点的删除发生在了 h 子树, h 子树的高度减 1, 而在节点被删前 A 的平衡因子已经是 1 了, 所以导致了 A 的不平衡, 需要进行顺单旋转操作.

  这里有一些一定成立的约束, 就是 h 一定比 B 小 2, 且因为是逆单, h 一定比 m 小 1, h 和 k 可能相等, 也可能比 k 小 1. 这些个关系应该很好理解, 不多做解释了.

  这个图的抽象的很好, 不能看出, 在判断出需要以 A 为轴心旋转, 到旋转完成后, m, k, h 三颗子树上的节点的平衡因子是不会受影响而变化的, m 和 k 是显然的, h 稍微一想也能知道, h 上的节点被删除后, 回溯过程中回溯路径上的节点的平衡因子也已经按照上面的规则被更新了, 等回溯到 A 节点发现需要旋转时, h 子树上的节点的平衡因子已经调整为正确的值了. 所以在实际的旋转过程中需要调整平衡因子的节点只有 A, B 两个, 这和高度版本中需要更新高度的节点是一致的.


  要知道 A 和 B 的平衡因子该如何变化, 就要知道旋转过后的样子是什么, 所以我们来看一下旋转过后变成什么样子:

  1)
                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /      /\
            /\     / k\
           /  \    ----
          / m  \ 
         /      \
         --------

                        |
                        |
                        V

                        B
                   /         \
                  /           \
                 /             A
                /\           /   \ 
               /  \         /     \
              / m  \       /\     /\
             /      \     / k\   / h\
             --------     ----   ----

  完美! 旋转过后, A 的左右子树高度一样, B 的左右子树高度也一样, 所以 A 和 B 的平衡因子都是 0. (*注意, 整颗树的高度是降低了的, 从 3 降到了 2, 记住这一点, 用来回答前一个问题*)

  2)
                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /       \
            /\       /\
           /  \     /  \
          / m  \   / k  \
         /      \ /      \
         -------- --------

                        |
                        |
                        V

                        B
                   /         \
                  /           \
                 /             A
                /\           /   \ 
               /  \         /     \
              / m  \       /\     /\
             /      \     /  \   / h\
             --------    /  k \  ----
                        /      \
                        --------

  这个就不是太完美了, 旋转过后, A 的左子树比右子树高一级, 导致 B 的右子树比左子树高一级, 所以 A 的平衡因子是 1, B 的平衡因子都是 -1. (*注意, 这个情况下整颗树的高度未变, 都是 3, 记住这一点, 用来回答前一个问题*)


  如果能够一直这么完美下去就好了, 可惜不是, 但麻烦的事情也不会多了, 我们虽然目前只解决了顺单, 但实际上逆单也解决了, 因为它们是对称的嘛! 接下来我们只要解决逆顺双, 则顺逆双也就同理可得了.

  逆顺双比逆单麻烦一些, 细加思考会发现它需要继续细分为如下三种情况:

   1)
                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /\      \
            /m \      C 
            ----   /     \
                  /\     /\
                 / n\   / k\
                 ----   ----

                        |
                        |
                        V

                        C
                   /         \
                  /           \
                 B             A
              /     \        /   \
             /\     /\      /\   /\
            /m \   / n\    / k\ / h\
            ----   ----    ---- ----

    这种情况下整颗树的高度由 3 降到了 2, A 由 +1 变为 0, B 由 -1 变为 0, C 保持为 0. 


     2)                 A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /\      \
            /m \      C 
            ----   /     \
                  /\     /\k
                 / n\    --
                 ----
                        |
                        |
                        V

                        C
                   /         \
                  /           \
                 B             A
              /     \        /   \
             /\     /\      /\k  /\
            /m \   / n\     --  / h\
            ----   ----         ----

    这种情况下整颗树的高度由 3 降到了 2, A 的平衡因子由 +1 变为 -1, B 由 -1 变为 0, C 由 +1 变为 0

     3)

                        A
                   /         \
                  /          /\
                 B          / h\
              /     \       ---- 
             /\      \
            /m \      C 
            ----   /     \
                  /\n    /\
                  --    / k\
                        ----

                        |
                        |
                        V

                        C
                   /         \
                  /           \
                 B             A
              /     \        /   \
             /\     /\n     /\   /\
            /m \    --     / k\ / h\
            ----           ---- ----

    这种情况下整颗树的高度由 3 降到了 2, B 的平衡因子由 -1 变为 +1, A 由 +1 变为 0, C 由 -1 变为 0

    这个情况稍复杂, 解释一下, 因为现在是 C 这颗子树高了导致了 A 不平衡, 所以 h 必须比 B 的高度小 2, 也所以 h 必须比 C 的高度小 1, 再所以 h 和 m 高度必须相等, 因为 h 高度比 m 大的话 A 就不会不平衡, h 高度比 m 小的话那就是 B 直接造成了 A 的不平衡, 那就是一个逆单旋转搞定的事情, 就轮不到 C 转了. 另外 h 比 C 高度小 1, 所以 h 和 n (或者 k) 的高度也相同, 所以高度上 h == m == n (或 k). 而 n 和 k 的高度不一定相同, 他俩只要有一个和 h 相同就够了.

    那么这就引发了三种情况,
    
    因为旋转后 A 的左子树将由 C 的右子树 k 来充当, 如果 k 和 n 高度相同 (因此和 h, m 也相同), 则旋转之后 A, B 的平衡因子都会是 0; 如果 k 高度比 n, h, m 它们小 1, 则旋转之后 A 平衡因子是 -1, B, C 的平衡因子都会是 0; 如果 n 高度比 k, h, m 它们小 1, 则旋转之后 B 平衡因子是 +1, A, C 的平衡因子都会是 0; 

    到此为止, 我们就能够回答 *节点删除后 (涉及旋转), 如何正确的判断子树的深度是否发生了变化 (如何判断节点平衡因子是否需要更新)* 这个问题了, 通过上面总共 5 中旋转情况, 我们发现, 凡是高度发生变化的, 新根的平衡因子都是 0 (1, 3, 4, 5 四种情况), 反之, 新根的平衡因子不是 0 (第 2 种情况)

#### 节点删除后, 如何判断子树是否发生了旋转

这个问题很好解决, 如果子树发生了旋转, 那么子树的根肯定要变的, 所以只要在进入子树时保存一下子树的根, 回溯回来时和返回的子树的新根比较一下, 相同就说明子树没转, 否则就说明子树旋转了.

这里有一个情况需要注意, 就是被删节点回溯这一步, 这时候父级保存的值和返回的值 (NULL) (由于我们采取的删除策略, 保证了删除总是发生在叶子节点, 因此回溯返回值一定是 NULL) 肯定也是不一样的, 但这并不代表旋转了.

## 参考

1. Ben Pfaff 的 libavl 库, 是平衡因子版的实现, 链接: http://adtinfo.org/libavl.html/. 我写完上面对于平衡因子版的分析之后, 想找一找有没有和我思路一样的实现, 然后就在找到了 Ben Pfaff 的 libavl, 我的库的实现思路和 Ben Pfaff 的思路完全一致, 尤其是对插入删除结点时旋转的平衡因子调整, 分为几种情况, 每种情况如何调整都完全一致!
