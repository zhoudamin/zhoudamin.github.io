---
title: 'Pow(x, n)'
categories:
  - 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2020-04-25 11:26:31
tags: leetcode
---
<p>实现&nbsp;<a href="https://www.cplusplus.com/reference/valarray/pow/" target="_blank">pow(<em>x</em>, <em>n</em>)</a>&nbsp;，即计算 x 的 n 次幂函数。</p>

<p><strong>示例 1:</strong></p>

<strong>输入:</strong> 2.00000, 10
<strong>输出:</strong> 1024.00000

<p><strong>示例&nbsp;2:</strong></p>

<strong>输入:</strong> 2.10000, 3
<strong>输出:</strong> 9.26100


<p><strong>示例&nbsp;3:</strong></p>

<strong>输入:</strong> 2.00000, -2
<strong>输出:</strong> 0.25000
<strong>解释:</strong> 2<sup>-2</sup> = 1/2<sup>2</sup> = 1/4 = 0.25


<p><strong>说明:</strong></p>

<ul>
	<li>-100.0 &lt;&nbsp;<em>x</em>&nbsp;&lt; 100.0</li>
	<li><em>n</em>&nbsp;是 32 位有符号整数，其数值范围是&nbsp;[&minus;2<sup>31</sup>,&nbsp;2<sup>31&nbsp;</sup>&minus; 1] 。</li>
</ul>
<div><div>Related Topics</div><div><li>数学</li><li>二分查找</li></div></div>

<!--more-->
思路： 

* 暴力会超时
* 不long 一个N 无法解决越边的问题

```java
class Solution {
    private double fastPow(double x, long n) {
        if (n == 0) {
            return 1.0;
        }
        double half = myPow(x, (int) (n / 2));
        if (n % 2 == 0) {
            return half * half;
        } else {
            return half * half * x;
        }
    }
    
    public double myPow(double x, int n) {
        long N = n;
        if (N < 0) {
            x = 1 / x;
            N = -N;
        }
        return fastPow(x, N);
    }
}
```