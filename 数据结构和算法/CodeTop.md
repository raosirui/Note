# :fire:CodeTop

## 双指针滑动窗口

### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。

![image-20250225202447947](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252024012.png)



### [ 15. 三数之和](https://leetcode.cn/problems/3sum)

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

![image-20250226100640312](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261006400.png)



### [ 21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists)

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

![image-20250226104205982](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261042039.png)



### [ 5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring)

给你一个字符串 `s`，找到 `s` 中最长的 回文 子串。

![image-20250226105849220](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261058286.png)







## 设计类

### [146. LRU缓存机制](https://leetcode.cn/problems/lru-cache)

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

![image-20250225204247323](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252042374.png)

![image-20250225203853952](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252038035.png)







## 链表

### [ 206. 反转链表](https://leetcode.cn/problems/reverse-linked-list)

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

![image-20250225204803264](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252048305.png)



![image-20250225205848772](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252058825.png)

![image-20250225210602438](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252106487.png)

![image-20250225211112361](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252111412.png)





### [ 25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group)

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

![image-20250226095407342](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502260954438.png)







## 树

### 二叉树的数据结构定义

![image-20250226110353415](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261103462.png)

### [ 102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal)

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。









## 堆

### [ 215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array)

给定整数数组 nums 和整数 k，请返回数组中第 k 个最大的元素。

请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

你必须设计并实现时间复杂度为 O(n) 的算法解决此问题。

![image-20250225211641294](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502252116355.png)







## 动态规划

### [ 53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组**是数组中的一个连续部分。

![image-20250226101735516](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261017568.png)





## 排序

### [ 补充题4. 手撕快速排序](https://leetcode.cn/problems/sort-an-array)

给你一个整数数组 `nums`，请你将该数组升序排列。

你必须在 **不使用任何内置函数** 的情况下解决问题，时间复杂度为 `O(nlog(n))`，并且空间复杂度尽可能小。

![image-20250226103151324](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261031419.png)



















































































