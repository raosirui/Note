# CodeTop:fire:

## ACM模式

### 整数输入 Scanner

![image-20250307173232177](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503071732402.png)

### 字符串输入 BufferedReader

![image-20250307172852289](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503071728424.png)





## 单例

### 1、饿汉式

在类加载时候就进行实例化

```java
// 单例模式 饿汉式
class Singleton {
    // 在类加载时候就进行实例化
    private static Singleton singleton=new Singleton();
    private Singleton(){}

    public static Singleton getInstance(){
        return singleton;
    }
}
```

### 2、懒汉式

在类加载时不进行实例化，在第一次使用时候再进行实例化

```java
// 单例模式 懒汉式
class Singleton {
    private static Singleton singleton;
    private Singleton() {}

    // 加锁保证单例只会被实例化一次
    public synchronized static Singleton getInstance() {
        // 第一次使用时候再进行实例化
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

### 3、双重检查锁

懒汉式中同步处理时synchronized范围为整个方法，但实例化仅仅发生在第一次一旦实例化后，下次判断就不可能为空了，就不会进行实例化了，但还会进行同步块。所以可以减小同步范围

```java
class Singleton {
    // 使用 volatile 确保实例变量的可见性和防止指令重排序
    private volatile static Singleton singleton;
    // 私有化构造器，防止外部直接实例化
    private Singleton() {}

    // 提供全局访问点
    public static Singleton getInstance() {
        if (singleton == null) { // 第一次检查
            synchronized (Singleton.class) {
                if (singleton == null) { // 第二次检查
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```



- **懒汉式**的缺点是每次获取实例时都会进行同步，导致在高并发场景下性能较差。而 **双重检查锁** 通过减少同步的次数（仅在第一次创建实例时才同步），显著提升了性能。
- **懒汉式**相对简单，但可能会引发线程安全问题，特别是在没有同步的情况下，多个线程可能会并发创建多个实例。 **双重检查锁** 使用了 `volatile` 关键字，保证了实例初始化的线程安全。









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

![image-20250304130629620](C:/Users/raosirui/AppData/Roaming/Typora/typora-user-images/image-20250304130629620.png)





### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water)

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![image-20250227141245577](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271412748.png)



### [334. 递增的三元子序列](https://leetcode.cn/problems/increasing-triplet-subsequence)

给你一个整数数组 `nums` ，判断这个数组中是否存在长度为 `3` 的递增子序列。

如果存在这样的三元组下标 `(i, j, k)` 且满足 `i < j < k` ，使得 `nums[i] < nums[j] < nums[k]` ，返回 `true` ；否则，返回 `false` 

![image-20250301173346857](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503011733094.png)





### [ 31. 下一个排列](https://leetcode.cn/problems/next-permutation)

整数数组的一个 **排列** 就是将其所有成员以序列或线性顺序排列。

- 例如，`arr = [1,2,3]` ，以下这些都可以视作 `arr` 的排列：`[1,2,3]`、`[1,3,2]`、`[3,1,2]`、`[2,3,1]` 。

整数数组的 **下一个排列** 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 **下一个排列** 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

- 例如，`arr = [1,2,3]` 的下一个排列是 `[1,3,2]` 。
- 类似地，`arr = [2,3,1]` 的下一个排列是 `[3,1,2]` 。
- 而 `arr = [3,2,1]` 的下一个排列是 `[1,2,3]` ，因为 `[3,2,1]` 不存在一个字典序更大的排列。

给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。

必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。

![image-20250303092335878](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503030923009.png)



### [151. 翻转字符串里的单词](https://leetcode.cn/problems/reverse-words-in-a-string)

给你一个字符串 `s` ，请你反转字符串中 **单词** 的顺序。

**单词** 是由非空格字符组成的字符串。`s` 中使用至少一个空格将字符串中的 **单词** 分隔开。

返回 **单词** 顺序颠倒且 **单词** 之间用单个空格连接的结果字符串。

**注意：**输入字符串 `s`中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

![image-20250303195628106](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503031956237.png)



### [27. 移除元素](https://leetcode.cn/problems/remove-element/)

给你一个数组 `nums` 和一个值 `val`，你需要 **[原地](https://baike.baidu.com/item/原地算法)** 移除所有数值等于 `val` 的元素。元素的顺序可能发生改变。然后返回 `nums` 中与 `val` 不同的元素的数量。

假设 `nums` 中不等于 `val` 的元素数量为 `k`，要通过此题，您需要执行以下操作：

- 更改 `nums` 数组，使 `nums` 的前 `k` 个元素包含不等于 `val` 的元素。`nums` 的其余元素和 `nums` 的大小并不重要。
- 返回 `k`。

![image-20250314155705894](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503141557048.png)





### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其总和大于等于 `target` 的长度最小的 **子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

![image-20250315094136594](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503150941739.png)





















































## 哈希

### [ 1. 两数之和](https://leetcode.cn/problems/two-sum)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

![image-20250226142413760](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261424819.png)



### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

![image-20250308103125509](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503081031627.png)



















































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





### [ 142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii)

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。

![image-20250227141947551](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271419705.png)

数学推导是x+y=k*l，其中右边是k圈的意思，其实龟兔只要保证一个快一个慢，不管几步都行。







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





### [19. 删除链表的倒数第N个节点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

![image-20250227154007626](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271540726.png)



### [82. 删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii)

给定一个已排序的链表的头 `head` ， *删除原始链表中所有重复数字的节点，只留下不同的数字* 。返回 *已排序的链表* 。

![image-20250227155612387](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271556494.png)

![image-20250227160214256](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271602442.png)





### [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list)

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。

![image-20250303172916589](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503031729797.png)





### [328. 奇偶链表](https://leetcode.cn/problems/odd-even-linked-list/)

给定单链表的头节点 `head` ，将所有索引为奇数的节点和索引为偶数的节点分别组合在一起，然后返回重新排序的列表。

**第一个**节点的索引被认为是 **奇数** ， **第二个**节点的索引为 **偶数** ，以此类推。

请注意，偶数组和奇数组内部的相对顺序应该与输入时保持一致。

你必须在 `O(1)` 的额外空间复杂度和 `O(n)` 的时间复杂度下解决这个问题。

![image-20250308094759548](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503080947687.png)



### [字节跳动. 排序奇升偶降链表](https://mp.weixin.qq.com/s/0WVa2wIAeG0nYnVndZiEXQ)

1->4->3->2->5 给定一个链表奇数部分递增，偶数部分递减，要求时间O(n)空间O(1)内将链表变成递增

![image-20250308101347073](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503081013308.png)

























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





### [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

给你一棵二叉树的根节点 `root` ，翻转这棵二叉树，并返回其根节点。

![image-20250306132211382](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061322576.png)







### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

给你一个二叉树的根节点 `root` ， 检查它是否轴对称。

![image-20250306114544299](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061145417.png)





### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

给定一个二叉树 `root` ，返回其最大深度。

二叉树的 **最大深度** 是指从根节点到最远叶子节点的最长路径上的节点数。

![image-20250306133923656](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061339783.png)

### [559. N 叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-n-ary-tree/)

给定一个 N 叉树，找到其最大深度。

最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。

N 叉树输入按层序遍历序列化表示，每组子节点由空值分隔（请参见示例）。

![image-20250306134632262](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061346376.png)



### [111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

**说明：**叶子节点是指没有子节点的节点。

![image-20250306140801415](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061408577.png)

![image-20250306141055180](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061410304.png)





































## 栈和队列

### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses)

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 每个右括号都有一个对应的相同类型的左括号。

![image-20250226155228283](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261552346.png)

### [ 232. 用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks)

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：

实现 `MyQueue` 类：

- `void push(int x)` 将元素 x 推到队列的末尾
- `int pop()` 从队列的开头移除并返回元素
- `int peek()` 返回队列开头的元素
- `boolean empty()` 如果队列为空，返回 `true` ；否则，返回 `false`

![image-20250228210803893](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502282108089.png)





### [150. 逆波兰表达式求值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)

给你一个字符串数组 `tokens` ，表示一个根据 [逆波兰表示法](https://baike.baidu.com/item/逆波兰式/128437) 表示的算术表达式。

请你计算该表达式。返回一个表示表达式值的整数。

**注意：**

- 有效的算符为 `'+'`、`'-'`、`'*'` 和 `'/'` 。
- 每个操作数（运算对象）都可以是一个整数或者另一个表达式。
- 两个整数之间的除法总是 **向零截断** 。
- 表达式中不含除零运算。
- 输入是一个根据逆波兰表示法表示的算术表达式。
- 答案及所有中间计算结果可以用 **32 位** 整数表示。

![image-20250306102118109](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061021262.png)





### [227. 基本计算器 II](https://leetcode.cn/problems/basic-calculator-ii/)

给你一个字符串表达式 `s` ，请你实现一个基本计算器来计算并返回它的值。

整数除法仅保留整数部分。

你可以假设给定的表达式总是有效的。所有中间结果将在 `[-231, 231 - 1]` 的范围内。

**注意：**不允许使用任何将字符串作为数学表达式计算的内置函数，比如 `eval()` 。

![image-20250308113250011](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503081132144.png)







### [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

![image-20250306105642369](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061056503.png)



















































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







### [347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。

![image-20250306112421609](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061124746.png)





















































## 动态规划

### DP五部曲

1. **dp数组以及下表的含义**
2. **递推公式**
3. **dp数组如何初始化**
4. **遍历顺序**
5. **打印数组调试**





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





### [152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray)

给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个 **32-位** 整数。

![image-20250301164605367](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503011646500.png)





### [ 70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs)

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

![image-20250303172408283](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503031724397.png)























## 排序

### [ 补充题4. 手撕快速排序](https://leetcode.cn/problems/sort-an-array)

给你一个整数数组 `nums`，请你将该数组升序排列。

你必须在 **不使用任何内置函数** 的情况下解决问题，时间复杂度为 `O(nlog(n))`，并且空间复杂度尽可能小。

![image-20250226103151324](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261031419.png)



### [ 148. 排序链表](https://leetcode.cn/problems/sort-list)

给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。

![image-20250303091303403](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503030913568.png)















































## 图论

### [ 200. 岛屿数量](https://leetcode.cn/problems/number-of-islands)

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

![image-20250226142710893](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261427972.png)

![image-20250226143549097](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261435282.png)











































































## 二分查找

### [704. 二分查找](https://leetcode.cn/problems/binary-search)

![image-20250314153256887](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503141532066.png)



### [162. 寻找峰值](https://leetcode.cn/problems/find-peak-element/)

峰值元素是指其值严格大于左右相邻值的元素。

给你一个整数数组 `nums`，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 **任何一个峰值** 所在位置即可。

你可以假设 `nums[-1] = nums[n] = -∞` 。

你必须实现时间复杂度为 `O(log n)` 的算法来解决此问题。

![image-20250308144434464](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503081444601.png)







### [ 33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array)

整数数组 `nums` 按升序排列，数组中的值 **互不相同** 。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。例如， `[0,1,2,4,5,6,7]` 在下标 `3` 处经旋转后可能变为 `[4,5,6,7,0,1,2]` 。

给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，如果 `nums` 中存在这个目标值 `target` ，则返回它的下标，否则返回 `-1` 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

![image-20250226144826996](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261448233.png)





### [ 4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays)

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。

算法的时间复杂度应该为 `O(log (m+n))` 。

![image-20250304085154918](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503040852059.png)

![image-20250304085718339](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503040857452.png)



































































## 回溯

### 回溯模板

![image-20250306145536323](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061455464.png)



### [46. 全排列](https://leetcode.cn/problems/permutations)

给定一个不含重复数字的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。

![image-20250226152506190](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502261525299.png)

### [47. 全排列 II](https://leetcode.cn/problems/permutations-ii/)

给定一个可包含重复数字的序列 `nums` ，***按任意顺序*** 返回所有不重复的全排列。

![image-20250306193932350](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061939506.png)









### [93. 复原IP地址](https://leetcode.cn/problems/restore-ip-addresses)

**有效 IP 地址** 正好由四个整数（每个整数位于 `0` 到 `255` 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。

- 例如：`"0.1.2.201"` 和` "192.168.1.1"` 是 **有效** IP 地址，但是 `"0.011.255.245"`、`"192.168.1.312"` 和 `"192.168@1.1"` 是 **无效** IP 地址。

给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能的**有效 IP 地址**，这些地址可以通过在 `s` 中插入 `'.'` 来形成。你 **不能** 重新排序或删除 `s` 中的任何数字。你可以按 **任何** 顺序返回答案。

![image-20250227145016289](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271450402.png)

![image-20250227150039760](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502271500861.png)



### [77. 组合](https://leetcode.cn/problems/combinations/)

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

你可以按 **任何顺序** 返回答案。

![image-20250306153644317](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503061536476.png)



### [131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)

给你一个字符串 `s`，请你将 `s` 分割成一些 子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

![image-20250306205706050](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503062057230.png)





### [79. 单词搜索](https://leetcode.cn/problems/word-search/)

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

![image-20250311151855867](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503111518092.png)





### [22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

![image-20250313114337326](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503131143529.png)













## 线程

1. 三个线程分别打印 A，B，C，要求这三个线程一起运行，打印 n 次，输出形如“ABCABCABC....”的字符串
2. 两个线程交替打印 0~100 的奇偶数
3. 通过 N 个线程顺序循环打印从 0 至 100
4. 多线程按顺序调用，A->B->C，AA 打印 5 次，BB 打印10 次，CC 打印 15 次，重复 10 次
5. 用两个线程，一个输出字母，一个输出数字，交替输出 1A2B3C4D...26Z



```java
public class ABCPrinter {
    private static final Object lock = new Object();
    private static int count = 0;

    public static void main(String[] args) {
        Thread threadA = new Thread(new Printer("A"));
        Thread threadB = new Thread(new Printer("B"));
        Thread threadC = new Thread(new Printer("C"));
        
        threadA.start();
        threadB.start();
        threadC.start();
    }

    static class Printer implements Runnable {
        private String letter;

        Printer(String letter) {
            this.letter = letter;
        }

        @Override
        public void run() {
            while (count < 100) {
                synchronized (lock) {
                    if ((count % 3) == getLetterIndex(letter)) {
                        System.out.print(letter);
                        count++;
                        lock.notifyAll(); // 唤醒其他线程
                    } else {
                        try {
                            lock.wait(); // 等待
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }

        private int getLetterIndex(String letter) {
            switch (letter) {
                case "A": return 0;
                case "B": return 1;
                case "C": return 2;
                default: return -1;
            }
        }
    }
}

```































