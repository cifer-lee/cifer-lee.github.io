title: 加法和乘法问题
slug: math-plus-multiply
date: 2017-03-11 14:56:27
tags: leetcode

## 两个字符串表示的大数相乘问题

> Tag: leetcode 43

> Given two non-negative integers num1 and num2 represented as strings, return the product of num1 and num2.
> 
> Note:
> 
> The length of both num1 and num2 is < 110.
> Both num1 and num2 contains only digits 0-9.
> Both num1 and num2 does not contain any leading zero.
> You must not use any built-in BigInteger library or convert the inputs to integer directly.

这个问题主要有这么几个点:

1. 两数相乘, 积的位数最长为两数的位数的和.

   这个很好解释, 有个办法可以粗糙的证明, 就是计算一下 9 x 9, 9 x 99, 9 x 999, 99 x 99, ... 就能看出规律了. 其它数相乘结果肯定比都是 9 相乘结果要短.

   所以为返回结果申请缓冲区时的长度就有了, 但注意在 C 语言中申请的缓冲区长度要加 1 用于存放 '\0'

2. 算法中可能涉及到 `+ '0'`, `- '0'` 这样的运算, 结果缓冲区的类型最好声明成 unsigned char, 否则碰到 9 * 9 + '0' (即 81 + 48) 这样的结果就会溢出为负数了.

<!-- more -->

```
char* multiply(char* num1, char* num2) {
    unsigned char    *ans, *m;
    int     len1, len2, len, i, j, k, t, carry;
    
    len1 = strlen(num1);
    len2 = strlen(num2);
    
    len = len1 + len2 + 1;
    ans = calloc(len, sizeof *ans);
    if (!ans) return NULL;
    m = calloc(len1, sizeof *m);
    if (!m) return NULL;
    
    memset(ans, '0', len);
    for (j = len2 - 1 ; j >= 0 ; --j) {
        memset(m, 0, len1);
        for (i = len1 - 1 ; i >= 0 ; --i) {
            if (num1[i] == '0' || num2[j] == '0') continue;
            else
                m[i] = (num1[i] - '0') * (num2[j] - '0');
        }
        
        /* accumalte and handle carry */
        carry = 0;
        for (k = len2 - j - 1, t = len1 - 1 ; t >=0 ; ++k, --t) {
            ans[k] += m[t] + carry;
            carry = (ans[k] - '0') / 10;
            ans[k] = (ans[k] - '0') % 10 + '0';
        }
        
        while (carry) {
            ans[k] += carry;
            carry = (ans[k] - '0') / 10;
            ans[k] = (ans[k] - '0') % 10 + '0';
            ++k;
        }
    }

    /* remove leading 0, if all 0, keep the last */
    for (j = len - 1 ; j > 0 ; --j) {
        if (ans[j] == '0') ans[j] = 0;
        else break;
    }
    
    /* reverse result */
    for (i = 0 ; i < j ; ++i, --j) {
        k = ans[i];
        ans[i] = ans[j];
        ans[j] = k;
    }
    
    return ans;
}
```

这个问题有个更好的思路在这里: https://discuss.leetcode.com/topic/30508/easiest-java-solution-with-graph-explanation/
