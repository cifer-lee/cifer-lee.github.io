---
title: 对快速排序的理解与实现
date: 2016-07-24
slug: quick-sort
category: 
  - Algorithm
tags:
  - algorithm
  - quick-sort
---

## 快速排序的思想

选出一个 pivot, 比 pivot 小的全都换到左边, 比 pivot 大的全都换到右边, 最后安置好 pivot. 然后以此 pivot 位置为分界线, 对两个区间递归执行快速排序, 当最终每个区间大小为 1 的时候, 排序也就完成了.

要实践这个思想, 需要解决几个问题:

1. pivot 的选取策略
2. 交换策略, 也就是分区策略
3. pivot 的最终位置确定

关于 pivot 的选取, 原则是尽量是整个数组中所有的元素中位数. 这是个矛盾的问题, 我们找的 pivot 越接近中位数, 快排效率自然越高, 但是我们花费在找 pivot 上的时间也越长. 快排算法的核心还是要放在分治本身, 不必对 pivot 的要求过于苛刻.

目前公认比较不错的 pivot 选取策略叫做 "Median-of-Three", 顾名思义, 它的意思就是在整个数组中选取三个元素, 然后再选出这三个元素的中位数作为 pivot. 那么选择哪个元素呢? 就是第一, 最后, 和中间.

第二个问题, 交换. 交换这个过程在快排里有个专门的术语叫做 "Partition", 也就是分区. 因为快排的核心其实就是在不断的分区, 将 `< pivot` 的分一区, `> pivot` 的分另一区, 当分区结束后, 所有 `< pivot` 的都在 `pivot` 左边, 反之都在 `pivot` 右边. 所以实际上分区操作就是快排算法主要在干的事, 也是最核心的部分. 这个过程一定要快, 整个快排算法才会快.

pivot 最终位置的确定与分区策略是紧密相关的, 一个合格的分区策略, 在分区结束的时候应该能够指示出 pivot 的最终位置.

分区策略是重点, 我们着重讨论一下.

## 分区策略

目前被纳入实践的快排分区策略不止一种, 简单的分区策略还是不难设计的, 我刚看快排时, 也装模作样的自己设计了一个分区策略, 然后看了书里的策略后果断放弃自己的设计. 后来才知道我想的那个分区策略早被人想过了, 还有个名字叫做 "Lomuto Partition", 书里看到的策略叫做 "Hoare Partition", 这是一个非常简单但是非常高效的策略.

Hoare 策略是用两个游标 i, j, 从数组的两端同时向内搜索. 左边的游标 i 一直递增直到发现比 pivot 大的元素, 右边的游标 j 一直递减直到发现比 pivot 小的元素, 然后这两个游标所指的两个元素交换. 然后继续这一过程直到两个游标相遇为止. 这时候可以确定的是, i 左边的元素全都小于 pivot, 而 j 右边的元素全都大于 pivot.

这个过程中有一个至关重要的决策就是当游标搜索过程中发现元素和 pivot 相等时该怎么办, 我们的决策是相等时也进行交换, 原因我们后面会详细讨论.

经过上述可以看出, pivot 的最终位置也有线索了, 就是 i 或者 j 所指的位置, 但还有一个问题是, 把 pivot 放到了 i 或 j 所指的位置, 那么 i 或 j 所指的原先的元素放哪儿呢? 这时我们不禁会想到, 在上面的交换过程中, pivot 放在哪里了呢? 既然 pivot 要放在 i 或 j 的位置, 那 i 或 j 位置处的元素放在 pivot 的位置不就行了吗?

但问题是, 经过上述不可预测的交换过程后, 我们不知道 pivot 被换到哪里去了, 比如下面这个数组 `a[8]`:

      3   1   8   2   9   4   6   7
    pivot

使用 Median3 我们能够选出 3 为 pivot. 然后 i, j 分别从首位开始搜索, 过程如下: 

    1)  3   1   8   2   9   4   6   7         (pivot == 3)
        i           j

                    |
                    |
                    v

    2)  2   1   8   3   9   4   6   7
                i,j


第一次交换发生在 `a[0]` (pivot 自身) 与 `a[4]` 之间. 然后 i, j 相遇, 过程结束. 此时 i, j 的位置就是 pivot 的位置, 而 pivot 跑到了 `a[3]` 的位置, 我们随便改变一下数组结构, 如下:

      2   8   9   3   1   6   4   7
                pivot

显然 pivot 仍然是 3, 交换过程如下:

    1)  2   8   9   3   1   6   4   7         (pivot == 3)
            i           j

                    |
                    |
                    v

    2)  2   1   9   3   8   6   4   7
                i   j

                    |
                    |
                    v

    3)  2   1   3   9   8   6   4   7
                j   i

可以看到, 同样的数组元素, 同样的 pivot, 这次 pivot 却被换到了 `a[1]` 的位置, 这就尴尬了. 不止是 pivot 的位置不确定, 综合这两个例子来看, 除了 pivot 的位置不确定之外, i 和 j 该用谁来作为 pivot 的最终位置也是不确定的. 第一个例子中该用 i, 而第二个例子中该用 j.

我们必须解决这两个问题, 才能写出我们的算法.

第一个问题其实不难解决, 既然 pivot 最终可能会不知被换到什么地方去, 那么我们就能想到是否可以不让它参与交换过程呢? i, j 在搜索的时候搜到了 pivot 所在的位置时就跳过去, 这样 pivot 的位置就始终和刚被选出来时一样了. 不难想象, 这对最终的交换结果的正确性是没有影响的, 因为实际上不管 pivot 参不参与交换, 当交换结束也就是 i, j 相遇或交错 (不难理解, 交换规则保证了交错距离不可能大于 1) 时, pivot 可能的位置有:

1. i 右边
2. i 重叠
3. j 重叠
4. j 左边

不管 pivot 位于这 4 个位置的哪个位置, i 右边的元素肯定都不比 pivot 小, j 右边的元素也肯定都不比 pivot 大. pivot 的最终位置仍然是从 i, j 二者之中选出. 由此可知 pivot 是否参与整个交换过程甚至参与局部交换过程对结果的正确性都是没有影响的.

第二个问题也不难解决, 上面说了, 当交换结束时, pivot 的位置有 4 中情况, 显然, 对于 1, 2 两种情况, i 就是 pivot 的最终位置; 3, 4 两种情况则 j 是 pivot 的最终位置. 说白了, pivot 离谁近谁就是最终位置. 既然如此我们干脆就在交换开始前把 pivot 换到最后, 然后交换开始时直接跳过最后的元素, 因为通过第一个问题的讨论我们知道 pivot 的位置对交换结果的正确性是没有影响的, 只要交换结束后能让我们找着就行了. 这样一来好处是交换结束后 pivot 的最终位置无须判断距离, 一定就是 i 所指的位置.


根据上述的原理, 我们可以写出如下的代码:

	void swap(int *a, int *b)
	{
	    int tmp;
	    tmp = *a; *a = *b; *b = tmp;
	}
	
	int median3(int *arr, int left, int right)
    {
        int mid, pi;

        mid = (left + right) / 2;

        if (arr[left] < arr[right]) {
            if (arr[left] > arr[mid]) pi = left;
            else pi = mid;
        } else {
            if (arr[right] > arr[mid]) pi = right;
            else pi = mid;
        }

        swap(&arr[right], &arr[pi]);
        return arr[right];
    }

    void qsort(int *arr, int left, int right)
    {
        int i, j, pivot;

        if (left >= right) return;

        pivot = median3(arr, left, right);

        i = left - 1;
        j = right;
        do {
            while (i < right && arr[++i] < pivot);  /* 1 */
            while (j > left && arr[--j] > pivot);   /* 2 */

            if (i < j)
                swap(&arr[i], &arr[j]);
            else
                break;
        } while (1);

        /* i points to the position pivot should on */
        swap(&arr[i], &arr[right]);

        qsort(arr, left, i - 1);
        qsort(arr, i + 1, right);
    }

这段代码看起来没什么出奇, 但实际逻辑是很严格的, 尤其是分区逻辑, 一不小心改动一点, 就会破坏整个算法. 我们来稍微解释下, 可以看到在 `median3()` 方法中, 我们选出了左, 中, 右三个元素的中位数作为 `pivot`, 将其与最右的元素交换, 然后返回这个 `pivot`.

在接下来的分区过程中, `i`, `j` 两个游标分别从 `left`, `right - 1` 处开始搜索. 注意这里我们采用了前缀自增运算, 所以 `i`, `j` 分别初始化为 `left - 1` 和 `right`, 之所以用前缀自增而不用后缀是因为后面的 `if (i < j)` 判断需要用到 `i`, `j` 不满足 `pivot` 的大小关系时的值.

另外可以看到分区过程中右对边界检测的逻辑, 也就是上述代码的 `/* 1 */` 和 `/* 2 */` 这两行. 实际上稍加分析就会发现, 在上述代码实现的分区过程中, 要发生溢出其实是非常非常难的, 但是难不代表没有, 我们先来解释下什么时候 j 会出现越界的情况. 假设 `/* 1 */`, `/* 2 */` 处没有越界判断, 我们有一个只有两个元素的数组 `a[2]:

        5       6

显然经过 `median3()` 操作之后, 选出的 pivot 是 5, `a[2]` 就会变成这样:

        6       5
              pivot

然后开始分区过程, left 为 0, right 为 1. i 为 -1, 然后从 -1 自增到 0, a[0] 为 6, 不小于 5, 停下; j 为 1, 自减到 0, a[0] 为 6, 大于 5, 继续自减到 -1, a[-1] 溢出.

这是 j 的情况, 溢出的原因就是出现了 pivot 左边的元素全都大于 pivot 的尴尬场景. 其实仔细想想这种情况只可能会发生在数组元素只有两个的情况下, 因为但凡大于两个元素, 就一定存在比 pivot 小的元素, 这个元素要么本就在 j 左边, 要么一定会在 j 自减到 left 之前被换到左边.

然后再来看一下 i 什么时候会溢出, 你会发现 i 根本没机会溢出, 因为 pivot 就在 a[right] 上, 所以 i 最终一定会在 right 处停下, 然后到达 `if (i < j)` 的判断时, i 绝对不会小于 j, 因为 j 是从 right 开始递减的. 这个逻辑同时也确保了即时 i 增到了 right, 也就是指向了 pivot, pivot 也不会参与交换过程, 这是很重要的.

由此可见, 针对 j 的边界检测只对两个元素的数组有意义. 针对 i 的边界检测其实完全没有存在的必要, 我之所以代码里那么写, 完全是为了对称.

进一步, 由此我们能构想到, 既然这两句边界检测逻辑这么鸡肋, 那我们是不是可以对算法稍作改善, 彻底去掉这两句边界检测逻辑呢? 想必对于追求代码精炼的人来说, 多省掉几个字符还是很有成就感的. 这点到后面快排算法优化的时候我们再一并讨论.

接下来我们先来讨论一下前面提到的一个关键的问题, 那就是

## 与 pivot 相等的元素要不要交换?

先说答案, 要.

虽然这么做回增加交换次数, 但是不这么做的话则可能会破坏二分的平衡. 想象一个最极端的情况, 比如说一个数组的元素全都一样:

    6 6 6 6 6 6 6 6 6 6 6 6 6 6 6
    i                         j pivot

如果相等的情况不做交换, 那么结果就是 i 一直增加, 导致了 i, j 相遇的地点偏右. 显然, 如果 i, j 相遇的地点偏向一极而不是在中间的话, 那么后续的递归层级也就会相应的加深.

另外, 如果采用这种策略的话, 那么上述代码逻辑上还有个不好的地方就是 i, j 会一直增加或减小直至数组溢出. 这增加了边界溢出的情况, 也给我们后续要将这两个边界检测逻辑优化掉的宏图大志带来的麻烦.

而如果相等的情况也交换的话, 可以看到, i 和 j 会同步的向内移动, 每移动一步就会发生一次交换, 虽然这带来了很多不必要的交换, 但是 i 和 j 最终相遇的地点是在中间, 这能够产生一个更浅的递归层级以及更小的时间复杂度 `O(NlogN)`.

不要小看了这两者的差距, 假如我们有一个包含 1024 个相同元素的数组, 采用相等不交换策略的话递归层级将是 1024, 而采用相等交换策略的话递归层级只有 10 层. 你可能会说这例子太极端了, 是的, 排序一个元素全相同的数组确实很少见. 但是像 "一个有 10000 个元素的数组, 其中有 5000 个元素相同" 类似的例子就不那么罕见了.

递归层级最大限度的降低, 带来的空间消耗也是低的令人惊喜, 就算我们有一个 2^32 大小的数组, 要排序它也只需要 32 层递归, 按照我们上面算法也只需要消耗 6 * 4 * 32 == 768 字节的栈空间, 连 1K 都不到.

## 轻微改动导致的错误

前面我们说了我们的实现代码逻辑是比较严格的, 一点看似等价的改动实际都可能会导致算法错误.

其中一种情况是将分区逻辑改成如下:

        i = left;                                    /* modified */
        j = right - 1;                               /* modified */
        do {
            while (i < right && arr[i] < pivot) ++i; /* modified */
            while (j > left && arr[j] > pivot) --j;  /* modified */

            if (i < j)
                swap(&arr[i], &arr[j]);
            else
                break;
        } while (1);

对比原代码, i 初值改成 left, j 初值改成了 right - 1, 然后后面的搜索过程中对 i, j 的自增自减放到了 while 循环的 body 里. 看起来好象是等价的改动吧? 实际不是的, 原代码中的前缀自增/自减隐藏了一个逻辑是: 当 `arr[++i] < pivot` 不成立时, i 仍然会 +1, j 也是同理; 而这里的改动导致了当 `arr[i] < pivot` 不成立时, while 循环退出, i 没有捞得着 +1. 当 i, j 同时分别指向了与 pivot 相同的元素, 并且 i, j 还没相遇时, 程序就陷入了无限循环中, 大家可以自行分析一下.

要解决这个无线循环的问题, 就得在交换操作之后再加上这么一句: `++i, --j;`, 这样这段逻辑才真的和原代码等价

## 优化之一, 去掉边界检测

前面说了我们要去掉分区逻辑中那两句丑陋的边界判断, 这能够节省不少的计算资源.

那该怎么去掉边界检测呢? 据我所知, 其实所有去掉边界检测的方法本质就是在边界上安置哨兵. 所以我最初想到的方法是设立哨兵, 遗憾的是这个方法并不能解决两个元素的数组的溢出问题, 但我觉得还是值得写一下的.

前面说了对 i 的边界检测实际是毫无意义的, 原因就在于 pivot 已经处于了右边界, 使得 `while (arr[++i] < pivot);` 一定能够最终退出, 这里处于右边界的 pivot 就是起到了哨兵的作用. 那么对于 j 来说应该也可以应用同样的思想, 那就是在分区开始时将一个肯定不大于 pivot 的元素换到到最左边去, 这个工作什么时候做呢, 显然, 在 "Median-of-Three" 过程中做是最合适不过的了.

为此我们的 `median3` 方法可以改成如下:

    int median3(int *arr, int left, int right)
    {
        int mid;

        mid = (left + right) / 2;

        if (arr[left] > arr[mid])
            swap(&arr[left], &arr[mid]);
        if (arr[left] > arr[right])
            swap(&arr[left], &arr[right]);
        if (arr[mid] < arr[right])
            swap(&arr[mid], &arr[right]);

        /* Now arr[left] is the smallest, the arr[right] is the medium */
        return arr[right];
    }

这个应该不用多解释, 经过那三个 `if` 之后, 三个元素中最小的就到了最左边, 而 pivot 则仍然是在最右边. 这个方法确实对于大于等于三个元素的数组是能够正确的设立哨兵, 但是对于两个元素的数组依然是无效的, 仍以 `[6,5]` 为例, 可以看到经过新的 `median3` 之后, 数组仍会保持 `[6,5]`, 因为在只有两个元素的情况下, mid 和 left 是同一个值, 并没有哨兵被设立, j 还是会溢出...

虽然这个尝试没有成功, 但是如此一来, 分区逻辑那里 i 的初始值就可以是 left, 而不用是 left - 1 了, 这样后面的 `while()` 循环里就会从 left + 1 开始和 pivot 比, 因为我们知道 left 的位置的元素肯定比 pivot 小, 那也就不用多此一举再比一次了. 这使得代码稍微更美观了一些, 这也是为什么我觉得这一段还是值得改动的原因.

既然哨兵这个办法不行, 那就上另一个办法, 那就是对两个元素以及小于两个元素的数组区分对待, 其实我们的代码里 `qsort` 顶上的这一句:

    if (left >= right) return;

已经显示了我们对于小于等于 1 个元素的数组的区分对待了, 一来一个元素以下的数组还排什么序, 二来递归嘛, 得有终止条件. 现在这里我们要把两元素数组的情况也加进来了, 综合起来, 最终的代码如下:

	void swap(int *a, int *b)
	{
	    int tmp;
	    tmp = *a; *a = *b; *b = tmp;
	}
	
    int median3(int *arr, int left, int right)
    {
        int mid;

        mid = (left + right) / 2;

        if (arr[left] > arr[mid])
            swap(&arr[left], &arr[mid]);
        if (arr[left] > arr[right])
            swap(&arr[left], &arr[right]);
        if (arr[mid] < arr[right])
            swap(&arr[mid], &arr[right]);

        /* Now arr[left] is the smallest, and arr[right] is the medium */
        return arr[right];
    }

    void qsort(int *arr, int left, int right)
    {
        int i, j, pivot;

        if (left + 1 >= right) {
            if (arr[left] > arr[right])
                swap(&arr[left], &arr[right]);

            return;
        }

        pivot = median3(arr, left, right);

        i = left;
        j = right;
        do {
            while (arr[++i] < pivot);   /* 1 */
            while (arr[--j] > pivot);   /* 2 */

            if (i < j)
                swap(&arr[i], &arr[j]);
            else
                break;
        } while (1);

        /* i points to the position pivot should on */
        swap(&arr[i], &arr[right]);

        qsort(arr, left, i - 1);
        qsort(arr, i + 1, right);
    }

这里我们把两元素数组区分对待, 看似曲线救国, 代码有点丑陋, 但实际上这是我们第二阶段优化的引子, 在下一次的优化中我们将会看到, 不单单是两元素, 将元素数量低于 10, 甚至 20 的数组从快排的核心逻辑中剔出来是要提高快排效率的必经之路.

## 优化之二, 绕过小分区

在上面的过程中我们被两元素的数组搞的不轻, 同时敏锐的我们可能也发现了, 对于两元素的数组我们直接比较交换比走快排逻辑省事多了, 快排还要多此一举的去选什么无意义的 pivot, 然后进行分区过程, 这对两元素的数组都是没有意义的. 实际上, 当数组的元素比较少时, 快排算法的优势确实成了劣势, 复杂的 pivot 选择与分区逻辑反而令其表现可能还不如插入排序. 然而, 由于快排是递归分区的, 数组越大, 最终就会出现越多的小分区, 这些小分区简直是快排的噩梦, 大量的 pivot 选择与分区操作被多余的附加到这些小分区上, 严重影响了快排算法的效率.

所以, 我们这一阶段的优化就是, 当数组的元素个数比较少的时候, 停用快速排序, 改用插入排序, 因为我们知道, 插入排序非常适合用来排序那些已经基本有序的数组, 而快排得到的小分区, 也正是基本有序的小数组, 这两者完美的组合是巧合还是必然呢? 算法的魅力就在于此吧.

好了, 我们最终的代码就是这样了:

    #define UPBOUND     9

	void swap(int *a, int *b)
	{
	    int tmp;
	    tmp = *a; *a = *b; *b = tmp;
	}

    void isort(int *arr, int left, int right)
    {
        int i, j, tmp;

        for (i = left + 1 ; i <= right ; ++i) {
            tmp = arr[i];
            for (j = i ; j > left && arr[j - 1] > tmp ; --j)
                arr[j] = arr[j - 1];
            arr[j] = tmp;
        }
    }
	
    int median3(int *arr, int left, int right)
    {
        int mid;

        mid = (left + right) / 2;

        if (arr[left] > arr[mid])
            swap(&arr[left], &arr[mid]);
        if (arr[left] > arr[right])
            swap(&arr[left], &arr[right]);
        if (arr[mid] < arr[right])
            swap(&arr[mid], &arr[right]);

        /* Now arr[left] is the smallest, the arr[right] is the medium */
        return arr[right];
    }

    void qsort(int *arr, int left, int right)
    {
        int i, j, pivot;

        if (left + UPBOUND >= right) {
            isort(arr, left, right);
            return;
        }

        pivot = median3(arr, left, right);

        i = left;
        j = right;
        do {
            while (arr[++i] < pivot);  /* 1 */
            while (arr[--j] > pivot);   /* 2 */

            if (i < j)
                swap(&arr[i], &arr[j]);
            else
                break;
        } while (1);

        /* i points to the position pivot should on */
        swap(&arr[i], &arr[right]);

        qsort(arr, left, i - 1);
        qsort(arr, i + 1, right);
    }

我们定义了一个宏 `UPBOUND`, 并且将其赋值为 9, 这样一来, 小于等于 10 的分区都会走插入排序流程了. 关于 `UPBOUND` 的取值, 有研究发现在 5 ~ 20 之内取值结果都差不多, 想比之前, 都能够带来 15% 左右的效率提升.
