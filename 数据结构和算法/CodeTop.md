# CodeTop:fire:

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

### [ 54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix)

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

![image-20250227105920542](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271059614.png)

### [ 415. 字符串相加](https://leetcode.cn/problems/add-strings)

给定两个字符串形式的非负整数 `num1` 和`num2` ，计算它们的和并同样以字符串形式返回。

你不能使用任何內建的用于处理大整数的库（比如 `BigInteger`）， 也不能直接将输入的字符串转换为整数形式。

![image-20250227113332531](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271133610.png)



### [ 56. 合并区间](https://leetcode.cn/problems/merge-intervals)

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

![image-20250227140510597](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271405179.png)

### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water)

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![image-20250227141245577](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271412748.png)





















































































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

### [92. 反转链表 II](https://leetcode.cn/problems/reverse-linked-list-ii)

给你单链表的头指针 `head` 和两个整数 `left` 和 `right` ，其中 `left <= right` 。请你反转从位置 `left` 到位置 `right` 的链表节点，返回 **反转后的链表** 。

![image-20250227105248387](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271052461.png)





### [ 25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group)

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

![image-20250226095407342](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502260954438.png)



### [ 141. 环形链表](https://leetcode.cn/problems/linked-list-cycle)

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

*如果链表中存在环* ，则返回 `true` 。 否则，返回 `false` 。

![image-20250227103949920](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271227143.png)



### [ 160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists)

![image-20250227122709146](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271228474.png)





### [ 143. 重排链表](https://leetcode.cn/problems/reorder-list)

给定一个单链表 `L` 的头节点 `head` ，单链表 `L` 表示为：

```
L0 → L1 → … → Ln - 1 → Ln
```

请将其重新排列后变为：

```
L0 → Ln → L1 → Ln - 1 → L2 → Ln - 2 → …
```

不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

![image-20250227134300378](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271343455.png)





### [ 142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii)

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。

![image-20250227141947551](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271419705.png)



### [19. 删除链表的倒数第N个节点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

![image-20250227154007626](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271540726.png)



### [82. 删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii)

给定一个已排序的链表的头 `head` ， *删除原始链表中所有重复数字的节点，只留下不同的数字* 。返回 *已排序的链表* 。

![image-20250227155612387](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271556494.png)

![image-20250227160214256](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271602442.png)







































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



### [124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum)

二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。

![image-20250227143533347](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271435536.png)



### [ 94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal)

给定一个二叉树的根节点 `root` ，返回 *它的 **中序** 遍历* 。

![image-20250227160530994](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271605152.png)

















































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

### [ 23. 合并K个排序链表](https://leetcode.cn/problems/merge-k-sorted-lists)

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

![image-20250227111748114](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271117209.png)



























































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



### [ 300. 最长上升子序列](https://leetcode.cn/problems/longest-increasing-subsequence)

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

![image-20250227112233147](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271122213.png)



### [72. 编辑距离](https://leetcode.cn/problems/edit-distance)

给你两个单词 `word1` 和 `word2`， *请返回将 `word1` 转换成 `word2` 所使用的最少操作数* 。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

![image-20250227142944718](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271429834.png)





### [ 1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence)

给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。

一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

- 例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。

两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

![image-20250227153824147](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271538257.png)

































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

### [704. 二分查找](https://leetcode.cn/problems/binary-search)

![image-20250227160720118](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271607239.png)





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



### [93. 复原IP地址](https://leetcode.cn/problems/restore-ip-addresses)

**有效 IP 地址** 正好由四个整数（每个整数位于 `0` 到 `255` 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。

- 例如：`"0.1.2.201"` 和` "192.168.1.1"` 是 **有效** IP 地址，但是 `"0.011.255.245"`、`"192.168.1.312"` 和 `"192.168@1.1"` 是 **无效** IP 地址。

给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能的**有效 IP 地址**，这些地址可以通过在 `s` 中插入 `'.'` 来形成。你 **不能** 重新排序或删除 `s` 中的任何数字。你可以按 **任何** 顺序返回答案。

![image-20250227145016289](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271450402.png)

![image-20250227150039760](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271500861.png)





















































