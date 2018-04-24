title: LeetCode Tag Linkedlist (Easy)
slug: leetcode-linkedlist-easy
date: 2015-04-05 11:35:00
tags: leetcode, algorithm, linked list

### Delete Node in a Linked List

之前从没刷过题的我, 刚看到这题时还是比较懵的, 但好在我马上清醒了过来. 删除单链表中的节点, 但是却没有给出前继节点或者表头节点, 所以这题肯定不是链表操作的常规套路.

    void deleteNode(struct ListNode *node)
    {
        struct ListNode *tmp;

        if (!node || !node->next) return;

        node->val = node->next->val;
        tmp = node->next;
        node->next = tmp->next;

        free(tmp);
    }

<!-- more -->

### Remove Duplicates from Sorted List

这题没啥可说, 顺着思路写就行了

    struct ListNode* deleteDuplicates(struct ListNode* head)
    {
        struct ListNode *p;
        struct ListNode *tmp;

        p = head;
        while (p && p->next) {
            if (p->val == tmp->val) {
                tmp = p->next;
                p->next = tmp->next;
                free(tmp); /* we can remove this line to get a better run time */
            } else
                p = p->next;
        }
        return head;
    }

### Remove Linked List Elements

这题需要注意的地方在于对头节点的处理, 如果头节点的值和给定的值相等因此需要移除头节点的话, 那么我们就不得不维护一个变量保存新的头节点, 并且新的头节点也是可能会被移除的, 所以我们就需要在遍历的过程中实时的监控要被移除的节点是不是头节点, 这样一来不仅逻辑复杂, 代码写起来更是不好看.

所以引入一个 dummy 的节点指向原链表的头节点, 这样使得原链表的头节点也能和普通节点一样被对待. 这是解决链表问题时常用的 trick.

    struct ListNode *removeElements(struct ListNode *head, int val)
    {
        struct ListNode     dh; /* dummy head */
        struct ListNode     *p; /* cursor */
        struct ListNode     *t; /* temp node */

        p = &dh;
        p->next = head;

        while (p && p->next) {
            if (p->next->val == val) {
                t = p->next;
                p->next = t->next;
                free(t);
            } else {
                p = p->next;
            }
        }

        return dh.next;
    }

### Linked List Cycle

判断链表内是不是有环, 常规方法的话我们可以再构造一个链表, 新链表节点的值是老链表节点的地址. 这样我们遍历老的链表, 每遍历一个节点就在新链表里找这个节点的地址是否已经存在, 存在的话就存在环; 否则就将这个节点地址插入新链表, 然后继续遍历老链表. 这样的话, 时间复杂度会是 O(n^2), 空间复杂度是 O(n).

而这题要求空间复杂得是 O(1), 其实我们也可以用一样的思路, 构造一个新链表, 只是不需分配新的节点, 还是复用老链表的节点就行了.

    int hasCycle(struct ListNode *head)
    {
        struct ListNode     nh;
        struct ListNode     *p;
        struct ListNode     *q;
        struct ListNode     *tmp;

        p = head;
        nh.next = NULL;
        while (p) {
            q = &nh;
            while (q->next) {
                if (q->next == p)
                    return 1;
                q = q->next;
            }

            q->next = p;

            tmp = p->next;
            p->next = NULL;
            p = tmp;
        }

        return 0;
    }

然而当我提交时, 却 TLE 了... 这看来 O(n^2) 的时间复杂度不行. 那么至少得是 O(nlogn) 才行. 由于遍历老链表时间复杂度已经是 O(n) 了, 于是我想着必须要将在新链表中查找老链表的节点这个操作优化成 O(logn) 才行. 但是我发现这是不可能的, 我们知道单链表的查找操作最快是 O(nlogn), 而且要达到 O(nlogn) 还必须要满足以下条件才可以:

1. 基于链表节点的值查找且单链表已经处于有序
2. 如果使用递归, 那么空间复杂度就成了 O(logn), 也和题设要求的 O(1) 空间复杂度不符了

然而我们此处的查找是基于节点地址查找的, 也没有什么有序不有序一说了. 总之要想确定老链表中的节点是否在新链表中, 就必须要做 n 次比较.

想到这里我不得不怀疑从一开始我的思路就错了...

然后我点了一下 `Show Tags`, 然后发现 `Two Pointers` 标签, 才恍然大悟, 于是有了以下解法

    int hasCycle(struct ListNode *head)
    {
        struct ListNode *fast;
        struct ListNode *slow;

        fast = slow = head;
        while (slow && fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;

            if (slow == fast) return 1;
        }

        return 0;
    }

### Merge Two Sorted Lists

这题没啥别的可说的, 和前面 *Remove Linked List Elements* 那题一样我们引入 dummy head 来简化代码逻辑.

    struct ListNode *mergeTwoLists(struct ListNode *l1, struct ListNode *l2)
    {
        struct ListNode dh;
        struct ListNode *p;
        struct ListNode *q;
        struct ListNode *n;

        n = &dh;
        p = l1;
        q = l2;

        while (p && q) {
            n->next = NULL;

            if (p->val < q->val) {
                n->next = p;
                p = p->next;
            } else {
                n->next = q;
                q = q->next;
            }

            n = n->next;
        }

        if (p) n->next = p;
        else n->next = q;

        return dh.next;
    }


### Reverse Linked List

反转链表, 没啥可说的

    struct ListNode* reverseList(struct ListNode* head) {
        struct ListNode     *h;
        struct ListNode     *p;
        struct Listnode     *n;
    
        h = NULL;
        p = head;
    
        while (p) {
            n = p->next;
            p->next = h;
            h = p;
            p = n;
        }

        return h;
    }

### Palindrome Linked List

判断是不是回文链表, 要求时间复杂度 O(n), 空间复杂度 O(1). 那我们必须反转一半链表了.

首先使用 two pointers 法定位中间节点; 然后反转前半部分 (或者后半部分) 链表; 然后同时遍历这两半截链表; 最后恢复被反转的那半截链表.

    bool isPalindrome(struct ListNode *head)
    {
        struct ListNode *fast;
        struct ListNode *slow;
        struct ListNode *p;
        struct ListNode *r;
        struct ListNode *n;
        bool            rc;

        slow = fast = head;

        /* find the middle node */
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }

        /* reverse the later half list */
        p = slow;
        r = NULL;
        while (p) {
            n = p->next;
            p->next = r;
            r = p;
            p = n;
        }

        /* now r points to later half's new head */
        p = head;
        n = r;
        rc = 1;
        while (p && n) {
            if (p->val != n->val) {
                rc = 0;
                break;
            }
            p = p->next;
            n = n->next;
        }

        /* 
         * reach here means palindrome
         * but we should recover the later half list
         */
        p = r;
        r = NULL;
        while (p) {
            n = p->next;
            p->next = r;
            r = p;
            p = n;
        }

        return rc;
    }

### Intersection of Two Linked List

这个问题的技巧是: 两个链表的长度加起来是一定的

    struct ListNode *
    getIntersectionNode(struct ListNode *headA, struct ListNode *headB)
    {
        struct ListNode  *p;
        struct ListNode  *q;

        if (!headA || !headB) return NULL;

        p = headA;
        q = headB;

        while (1) {
            if (!p && !q) break;
            if (!p) p = headB;
            if (!q) q = headA;

            if (p == q) return p;

            p = p->next;
            q = q->next;
        }

        return NULL;
    }

### Remove Nth Node From End of List

使用 two pointers 方法, 定位到倒数第 n 个节点的前一个节点, 然后将第 n 个节点移除之

    struct ListNode *removeNthFromEnd(struct ListNode *head, int n)
    {
        struct ListNode *t;
        struct ListNode *p;
        struct ListNode dh;

        dh.next = head;
        p = t = &dh;
        while (n--) t = t->next;

        while (t->next) {
            p = p->next;
            t = t->next;
        }

        t = p->next;
        p->next = p->next->next;
        free(t);

        return dh.next;
    }

### Swap Nodes in Pairs

这个指针的处理稍微有点乱, 可以画个图模拟一下流程, 然后写出来就行

    struct ListNode *swapPairs(struct ListNode *head)
    {
        struct ListNode dh;
        struct ListNode *r;
        struct ListNode *p;
        struct ListNode *n;

        dh.next = head;
        p = n = &dh;

        while (1) {
            r = p;
            p = p->next;
            if (!p) break;
            n = p->next;
            if (!n) break;

            r->next = n;
            p->next = n->next;
            n->next = p;
        }

        return dh.next;
    }
