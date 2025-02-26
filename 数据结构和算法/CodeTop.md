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

### [ 88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array)

给你两个按 **非递减顺序** 排列的整数数组 `nums1` 和 `nums2`，另有两个整数 `m` 和 `n` ，分别表示 `nums1` 和 `nums2` 中的元素数目。

请你 **合并** `nums2` 到 `nums1` 中，使合并后的数组同样按 **非递减顺序** 排列。

**注意：**最终，合并后数组不应由函数返回，而是存储在数组 `nums1` 中。为了应对这种情况，`nums1` 的初始长度为 `m + n`，其中前 `m` 个元素表示应合并的元素，后 `n` 个元素为 `0` ，应忽略。`nums2` 的长度为 `n` 。

![image-20250226154216543](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261542602.png)







## 哈希

### [ 1. 两数之和](https://leetcode.cn/problems/two-sum)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

![image-20250226142413760](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261424819.png)





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

### 链表的数据结构定义

![image-20250226112400153](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261124194.png)



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

![image-20250226112201614](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261122718.png)

### [ 103. 二叉树的锯齿形层次遍历](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal)

![image-20250226162931861](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261629970.png)



### [ 236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/最近公共祖先/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

![image-20250226170246895](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261702957.png)











## 栈和队列

### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses)

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 每个右括号都有一个对应的相同类型的左括号。

![image-20250226155228283](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261552346.png)













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

### [ 121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock)

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

![image-20250226153427672](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261534741.png)



## 排序

### [ 补充题4. 手撕快速排序](https://leetcode.cn/problems/sort-an-array)

给你一个整数数组 `nums`，请你将该数组升序排列。

你必须在 **不使用任何内置函数** 的情况下解决问题，时间复杂度为 `O(nlog(n))`，并且空间复杂度尽可能小。

![image-20250226103151324](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261031419.png)





## 图论

### [ 200. 岛屿数量](https://leetcode.cn/problems/number-of-islands)

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

![image-20250226142710893](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261427972.png)

![image-20250226143549097](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261435282.png)



## 二分查找

### [ 33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array)

整数数组 `nums` 按升序排列，数组中的值 **互不相同** 。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。例如， `[0,1,2,4,5,6,7]` 在下标 `3` 处经旋转后可能变为 `[4,5,6,7,0,1,2]` 。

给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，如果 `nums` 中存在这个目标值 `target` ，则返回它的下标，否则返回 `-1` 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

![image-20250226144826996](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261448233.png)





## 回溯

### [46. 全排列](https://leetcode.cn/problems/permutations)

给定一个不含重复数字的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。

![image-20250226152506190](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261525299.png)





























































