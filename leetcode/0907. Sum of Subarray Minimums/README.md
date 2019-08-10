# [907. Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/)

## 题目

Given an array of integers A, find the sum of min(B), where B ranges over every (contiguous) subarray of A.

Since the answer may be large, return the answer modulo 10^9 + 7.

 

Example 1:

```c
Input: [3,1,2,4]
Output: 17
Explanation: Subarrays are [3], [1], [2], [4], [3,1], [1,2], [2,4], [3,1,2], [1,2,4], [3,1,2,4]. 
Minimums are 3, 1, 2, 4, 1, 1, 2, 1, 1, 1.  Sum is 17.
```

Note:

1. 1 <= A.length <= 30000
2. 1 <= A[i] <= 30000


## 题目大意

给定一个整数数组 A，找到 min(B) 的总和，其中 B 的范围为 A 的每个（连续）子数组。

由于答案可能很大，因此返回答案模 10^9 + 7。


## 解题思路

- 首先想到的是暴力解法，用两层循环，分别枚举每个连续的子区间，区间内用一个元素记录区间内最小值。每当区间起点发生变化的时候，最终结果都加上上次遍历区间找出的最小值。当整个数组都扫完一遍以后，最终结果模上 10^9+7。
- 上面暴力解法时间复杂度特别大，因为某个区间的最小值可能是很多区间的最小值，但是我们暴力枚举所有区间，导致要遍历的区间特别多。优化点就在如何减少遍历的区间。第二种思路是用 2 个单调栈。想得到思路是 `res = sum(A[i] * f(i))`，其中 f(i) 是子区间的数，A[i] 是这个子区间内的最小值。为了得到 f(i) 我们需要找到 left[i] 和 right[i]，left[i] 是 A[i] 左边严格大于 A[i](>关系)的区间长度。right[i] 是 A[i] 右边非严格大于(>=关系)的区间长度。left[i] + 1 等于以 A[i] 结尾的子数组数目，A[i] 是唯一的最小值；right[i] + 1 等于以 A[i] 开始的子数组数目，A[i] 是第一个最小值。于是有 `f(i) = (left[i] + 1) * (right[i] + 1)`。例如对于 [3,1,4,2,5,3,3,1] 中的“2”，我们找到的串就为[4,2,5,3,3]，2 左边有 1 个数比 2 大且相邻，2 右边有 3 个数比 2 大且相邻，所以 2 作为最小值的串有 2 * 4 = 8 种。用排列组合的思维也能分析出来，2 的左边可以拿 0，1，…… m 个，总共 (m + 1) 种，同理右边可以拿 0，1，…… n 个，总共 (n + 1) 种，所以总共 (m + 1)(n + 1)种。只要计算出了 f(i)，这个题目就好办了。以 [3,1,2,4] 为例，left[i] + 1 = [1,2,1,1]，right[i] + 1 = [1,3,2,1]，对应 i 位的乘积是 f[i] = [1 * 1，2 * 3，1 * 2，1 * 1] = [1，6，2，1]，最终要求的最小值的总和 res = 3 * 1 + 1 * 6 + 2 * 2 + 4 * 1 = 17。
- **看到这种 mod1e9+7 的题目，首先要想到的就是dp**。最终的优化解即是利用 DP + 单调栈。单调栈维护数组中的值逐渐递增的对应下标序列。定义 `dp[i + 1]` 代表以 A[i] 结尾的子区间内最小值的总和。状态转移方程是 `dp[i + 1] = dp[prev + 1] + (i - prev) * A[i]`，其中 prev 是比 A[i] 小的前一个数，由于我们维护了一个单调栈，所以 prev 就是栈顶元素。(i - prev) * A[i] 代表在还没有出现 prev 之前，这些区间内都是 A[i] 最小，那么这些区间有 i - prev 个，所以最小值总和应该是 (i - prev) * A[i]。再加上 dp[prev + 1] 就是 dp[i + 1] 的最小值总和了。以 [3, 1, 2, 4, 3] 为例，当 i = 4, 所有以 A[4] 为结尾的子区间有:  
	
		[3]  
		[4, 3]  
		[2, 4, 3]  
		[1, 2, 4, 3]  
		[3, 1, 2, 4, 3] 
	在这种情况下, stack.peek() = 2, A[2] = 2。前两个子区间 [3] and [4, 3], 最小值的总和 = (i - stack.peek()) * A[i] = 6。后 3 个子区间是 [2, 4, 3], [1, 2, 4, 3] 和 [3, 1, 2, 4, 3], 它们都包含 2，2 是比 3 小的前一个数，所以 dp[i + 1] = dp[stack.peek() + 1] = dp[2 + 1] = dp[3] = dp[2 + 1]。即需要求 i = 2 的时候 dp[i + 1] 的值。继续递推，比 2 小的前一个值是 1，A[1] = 1。dp[3] = dp[1 + 1] + (2 - 1) * A[2]= dp[2] + 2。dp[2] = dp[1 + 1]，当 i = 1 的时候，prev = -1，即没有人比 A[1] 更小了，所以 dp[2] = dp[1 + 1] = dp[-1 + 1] + (1 - (-1)) * A[1] = 0 + 2 * 1 = 2。迭代回去，dp[3] = dp[2] + 2 = 2 + 2 = 4。dp[stack.peek() + 1] = dp[2 + 1] = dp[3] = 4。所以 dp[i + 1] = 4 + 6 = 10。
- 与这一题相似的解题思路的题目有第 828 题，第 891 题。