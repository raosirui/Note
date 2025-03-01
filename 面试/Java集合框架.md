# Java集合框架



### 说说有哪些常见的集合框架？

Java 集合框架可以分为两条大的支线：

①、Collection，主要由 List、Set、Queue 组成：

- List 代表有序、可重复的集合，典型代表就是封装了动态数组的 [ArrayList](https://javabetter.cn/collection/arraylist.html) 和封装了链表的 [LinkedList](https://javabetter.cn/collection/linkedlist.html)；
- Set 代表无序、不可重复的集合，典型代表就是 HashSet 和 TreeSet；
- Queue 代表队列，典型代表就是双端队列 [ArrayDeque](https://javabetter.cn/collection/arraydeque.html)，以及优先级队列 [PriorityQueue](https://javabetter.cn/collection/PriorityQueue.html)。

②、Map，代表键值对的集合，典型代表就是 [HashMap](https://javabetter.cn/collection/hashmap.html)。![二哥的 Java 进阶之路：Java集合主要关系](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161218256.png)

#### 简单介绍一下队列 Queue

Java 中的队列主要通过 java.util.Queue 接口和 java.util.concurrent.BlockingQueue 两个接口来实现。

PriorityQueue 是一个基于优先级堆的无界队列，它的元素按照自然顺序排序或者 Comparator 进行排序。

ArrayDeque 是一个基于数组的双端队列，可以在两端插入和删除元素。

LinkedList，它既可以当作 List 使用，也可以当作 Queue 使用。



#### 用过哪些集合类，它们的优劣？

在 Java 中，常用的集合类有 ArrayList、LinkedList、HashMap、LinkedHashMap 等。

1. ArrayList：ArrayList 可以看作是一个动态数组，它可以在运行时动态扩容。优点是访问速度快，可以通过索引直接查到元素。缺点是**插入和删除元素可能需要移动元素**，效率就会降低。
2. LinkedList：LinkedList 是一个双向链表，它适合频繁的插入和删除操作。优点是插入和删除元素的时候只需要改变节点的前后指针，缺点是**访问元素时需要遍历链表**。
3. HashMap：HashMap 是一个基于哈希表的键值对集合。优点是插入、删除和查找元素的速度都很快。缺点是它**不保留键值对的插入顺序**。
4. LinkedHashMap：LinkedHashMap 在 HashMap 的基础上增加了一个双向链表来保持键值对的插入顺序。



| 特性            | HashMap                                  | TreeMap                 |
| --------------- | ---------------------------------------- | ----------------------- |
| **底层结构**    | 哈希表                                   | 红黑树                  |
| **顺序**        | **无序**                                 | **按键排序**            |
| **时间复杂度**  | O(1)O(1)O(1)（理想情况下）               | O(log⁡n)O(\log n)O(logn) |
| **Null 键支持** | 允许一个 Null 键                         | **不允许 Null 键**      |
| **适用场景**    | 频繁的插入、删除、查找操作，不关心顺序。 | 有序存储，需按键排序    |



#### 哪些是线程安全的？

像 Vector、Hashtable、ConcurrentHashMap、CopyOnWriteArrayList、ConcurrentLinkedQueue、ArrayBlockingQueue、LinkedBlockingQueue 这些都是线程安全的。



#### Collection 继承了哪些接口？

**Collection 继承了 Iterable 接口**，这意味着所有实现 Collection 接口的类都必须实现 `iterator()` 方法，之后就可以使用增强型 for 循环遍历集合中的元素了。





## List

### ArrayList 和 LinkedList 有什么区别？

ArrayList 是基于数组实现的，LinkedList 是基于链表实现的。

#### 用途有什么不同？

多数情况下，ArrayList 更利于查找，LinkedList 更利于增删

②、ArrayList 如果增删的是数组的尾部，直接插入或者删除就可以了，时间复杂度是 O(1)；如果 add 的时候涉及到扩容，时间复杂度会提升到 O(n)。

但如果插入的是中间的位置，就需要把插入位置后的元素向前或者向后移动，甚至还有可能触发扩容，效率就会低很多，O(n)。

 二者增删的时间复杂度都是 O(n)，都需要遍历列表,但LinkedList 的增删只需要改变引用，而 ArrayList 的增删可能需要移动元素。



#### 是否支持随机访问？

①、ArrayList 是基于数组的，也实现了 RandomAccess 接口，所以它支持随机访问，可以通过下标直接获取元素。

②、LinkedList 是基于链表的，所以它没法根据下标直接获取元素，不支持随机访问，所以它也没有实现 RandomAccess 接口。



#### 内存占用有何不同？

ArrayList 是基于数组的，是一块连续的内存空间，所以它的内存占用是比较紧凑的；但如果涉及到扩容，就会重新分配内存，空间是原来的 1.5 倍，存在一定的空间浪费。

LinkedList 是基于链表的，每个节点都有一个指向下一个节点和上一个节点的引用，于是每个节点占用的内存空间稍微大一点。



### ArrayList 的扩容机制了解吗？

ArrayList 确切地说，应该叫做动态数组，因为它的底层是通过数组来实现的，当往 ArrayList 中添加元素时，会**先检查是否需要扩容，如果当前容量+1 超过数组长度，就会进行扩容。**

[![三分恶面渣逆袭：ArrayList扩容](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161458492.png)](https://camo.githubusercontent.com/037f698433269593ffe49ec363a0b0a9f64acd9ef493e47f7ef6be37ff61fb3a/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d352e706e67)

扩容后的新数组长度是原来的 1.5 倍，然后再把原数组的值拷贝到新数组中。

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```



### ArrayList 怎么序列化的知道吗？ 为什么用 transient 修饰数组？

ArrayList 的序列化不太一样，它使用`transient`修饰存储元素的`elementData`的数组，`transient`关键字的作用是让被修饰的成员属性不被序列化。

**为什么最 ArrayList 不直接序列化元素数组呢？**

出于效率的考虑，数组可能长度 100，但实际只用了 50，剩下的 50 不用其实不用序列化，这样可以提高序列化和反序列化的效率，还可以节省内存空间。

**那 ArrayList 怎么序列化呢？**

ArrayList 通过两个方法**readObject、writeObject**自定义序列化和反序列化策略，实际直接使用两个流`ObjectOutputStream`和`ObjectInputStream`来进行序列化和反序列化。



### 快速失败(fail-fast)和安全失败(fail-safe)了解吗？

**快速失败（fail—fast）**：快速失败是 Java 集合的一种错误检测机制

- 在用迭代器遍历一个集合对象时，**如果线程 A 遍历过程中，线程 B 对集合对象的内容进行了修改（增加、删除、修改）**，则会抛出 Concurrent Modification Exception。
- 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 ` modCount` 变量。集合在被遍历期间如果内容发生变化，就会改变`modCount`的值。每当迭代器使用 hashNext()/next()遍历下一个元素之前，都会检测 modCount 变量是否为 expectedmodCount 值，是的话就返回遍历；否则抛出异常，终止遍历。
- 注意：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改 modCount 值刚好又设置为了 expectedmodCount 值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的 bug。
- 场景：**java.util 包下的集合类都是快速失败的，不能在多线程下发生并发修改**（迭代过程中被修改），比如 ArrayList 类。

**安全失败（fail—safe）**

- 采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是**先复制原有集合内容，在拷贝的集合上进行遍历。**
- 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发 Concurrent Modification Exception。
- 缺点：基于拷贝内容的优点是避免了 Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。
- 场景：**java.util.concurrent 包下的容器都是安全失败，可以在多线程下并发使用**，并发修改，比如 CopyOnWriteArrayList 类。



### 有哪几种实现 ArrayList 线程安全的方法？

可以使用 **Collections.synchronizedList()方法**，它将返回一个线程安全的 List。

```java
SynchronizedList list = Collections.synchronizedList(new ArrayList());
```

内部是通过 [synchronized 关键字](https://javabetter.cn/thread/synchronized-1.html)加锁来实现的。



也可以直接使用 **CopyOnWriteArrayList**，它是线程安全的，遵循写时复制的原则，每当对列表进行修改（例如添加、删除或更改元素）时，都会创建列表的一个新副本，这个新副本会替换旧的列表，而对旧列表的所有读取操作仍然可以继续。

```java
CopyOnWriteArrayList list = new CopyOnWriteArrayList();
```

通俗的讲，CopyOnWrite 就是当我们往一个容器添加元素的时候，不直接往容器中添加，而是先复制出一个新的容器，然后在新的容器里添加元素，添加完之后，再将原容器的引用指向新的容器。多个线程在读的时候，不需要加锁，因为当前容器不会添加任何元素。这样就实现了线程安全。



#### ArrayList 和 Vector 的区别？

**Vector 属于 JDK 1.0 时期的遗留类**，已不推荐使用，仍然保留着是因为 Java 希望向后兼容。

ArrayList 是在 JDK 1.2 时引入的，用于替代 Vector 作为主要的非同步动态数组实现。因为 **Vector 所有的方法都使用 synchronized 关键字进行了同步**，单线程环境下效率较低。



### CopyOnWriteArrayList 了解多少？

CopyOnWriteArrayList 就是线程安全版本的 ArrayList。

CopyOnWriteArrayList 采用了一种**读写分离的并发策略**。CopyOnWriteArrayList 容器允许并发读，**读操作是无锁的**，性能较高。至于**写操作**，比如向容器中添加一个元素，则首先**将当前容器复制一份**，然后在新副本上执行写操作，结束之后再将原容器的引用指向新容器。

[![CopyOnWriteArrayList原理](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161505936.png)](https://camo.githubusercontent.com/24774944935a78100281c1d6b4376b69f5b35c6d8e6e330138cc36bb81a1e4e6/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d372e706e67)



## Map

### 能说一下 HashMap 的底层数据结构吗？

JDK 8 中 HashMap 的数据结构是`数组`+`链表`+`红黑树`。

[![三分恶面渣逆袭：JDK 8 HashMap 数据结构示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161508352.png)](https://camo.githubusercontent.com/99394a28167231b7d2331b8536d7e9e40e28977128c674f04dc78f6ca349aefa/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d382e706e67)



数组（`Node[] table`）用来存储键值对。数组中的每个元素被称为“桶”（Bucket），每个桶的索引通过对键的哈希值进行 `hash()` 处理得到。

当多个键经过哈希处理后得到相同的索引时，需要通过链表来解决哈希冲突——即将具有相同索引的键值对通过链表连接起来。

不过，链表过长时，查询效率会比较低，于是**当链表的长度超过 8 时（且数组的长度大于 64），链表就会转换为红黑树。**红黑树的查询效率是 O(logn)，比链表的 O(n) 要快。

`hash()` 的目标是尽量减少哈希冲突，保证元素能够均匀地分布在数组的每个位置上。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

如果键已经存在，其对应的值将被新值覆盖。

当从 HashMap 中获取元素时，也会使用 `hash()` 计算键的位置，然后根据位置在数组查找元素。



HashMap 的初始容量是 16，随着元素的不断添加，HashMap 的容量可能不足，于是就需要进行扩容，阈值是`capacity * loadFactor`，capacity 为容量，loadFactor 为负载因子，默认为 0.75。

扩容后的数组大小是原来的 2 倍，然后把原来的元素重新计算哈希值，放到新的数组中，这一步也是 HashMap 最耗时的操作。



### 你对红黑树了解多少？为什么不用二叉树/平衡树呢？

1. 每个节点要么是红色，要么是黑色；
2. 根节点永远是黑色；
3. 所有的叶子节点都是是黑色的（下图中的 NULL 节点）；
4. 红色节点的子节点一定是黑色的；
5. 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。

![image-20241216151324734](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161513795.png)

#### 为什么不用二叉树？

二叉树是最基本的树结构，每个节点最多有两个子节点，但是二叉树容易出现极端情况，比如插入的数据是有序的，那么二叉树就会退化成链表，查询效率就会变成 O(n)。

#### 为什么不用平衡二叉树？

平衡二叉树比红黑树的要求更高，每个节点的左右子树的高度最多相差 1，这种高度的平衡保证了极佳的查找效率，但在进行插入和删除操作时，可能需要频繁地进行旋转来维持树的平衡，维护成本更高。

#### 为什么用红黑树？

链表的查找时间复杂度是 `O(n)`，当链表长度较长时，查找性能会下降。红黑树是一种折中的方案，查找、插入、删除的时间复杂度都是 `O(log n)`。



### 红黑树怎么保持平衡的？

红黑树有两种方式保持平衡：`旋转`和`染色`。

①、旋转：旋转分为两种，左旋和右旋

![image-20241216151551524](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161515594.png)

②、染⾊：

[![三分恶面渣逆袭：染色](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161515944.png)](https://camo.githubusercontent.com/ee56def1d4d35f8fd3cd894af9aed73e419d2093e6d8c38d1f8ea9d08f048158/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d31322e706e67)



### :star:HashMap 的 put 流程知道吗？



[![三分恶面渣逆袭：HashMap插入数据流程图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161516764.jpeg)](https://camo.githubusercontent.com/c647c70999df881ddd4dec92ae9dd5ee8176b99b581be60ebf8ae3614e841b29/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d31332e6a7067)

第一步，通过 hash 方法计算 key 的哈希值。

第二步，数组进行第一次扩容。

第三步，根据哈希值计算 key 在数组中的下标，如果对应下标没有数组，直接插入。

否则，判断是否为相同的 key，是则覆盖 value。不是的话需要判断是否为树节点，是则向树中插入节点，否则向链表中插入数据。

注意，在链表中插入节点的时候，如果链表长度大于等于 8，则需要把链表转换为红黑树。

所有元素处理完后，还需要判断是否超过阈值`threshold`，超过则扩容。



#### 只重写 equals 没重写 hashcode，map put 的时候会发生什么?

如果只重写 equals 方法，没有重写 hashcode 方法，那么**会导致 equals 相等的两个对象，hashcode 不相等**，这样的话，这两个对象会被放到不同的桶中，这样就会导致 get 的时候，找不到对应的值。



### HashMap 怎么查找元素的呢？

先看流程图：

[![HashMap查找流程图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161521860.png)](https://camo.githubusercontent.com/9fa7c591a5d3e19620afcdcf9dc43648e3e960b640ebebce839c84d50a36b6e1/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d31342e706e67)

HashMap 的查找就简单很多：

1. 使用扰动函数，获取新的哈希值
2. 计算数组下标，获取节点
3. 当前节点和 key 匹配，直接返回
4. 否则，当前节点是否为树节点，查找红黑树
5. 否则，遍历链表查找



### HashMap 的 hash 函数是怎么设计的?

HashMap 的哈希函数是先拿到 key 的 hashcode，是一个 32 位的 int 类型的数值，然后让 hashcode 的高 16 位和低 16 位进行异或操作。

**key的hashCode和key的hashCode右移16位做异或运算**

```java
static final int hash(Object key) {
    int h;
    // key的hashCode和key的hashCode右移16位做异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这么设计是为了降低哈希碰撞的概率。



### :star:为什么要用高低做异或运算？为什么非得高低 16 位异或？

> [!IMPORTANT]
>
> **概括：首先 HashMap的hash函数是要进行一个取模操作，而取模效率太低就用hash & (n-1)来代替，所以n必须是为2的幂次方才能实现这样取模，但是一般n-1是个比较小的数，而hashcode是个32位的int类型，当hashcode&（n-1）时，往往只会取出后面几位的数字，更容易造成hash冲突，那么就想出来一个办法，将hashcode自身与自身右移16位来与一下，就能利用起来前面的高位，为什么是16位呢因为刚好是32的一半，既考虑了利用起来高位的信息，也尽最大可能保留低位原本信息的重要性，达到一个平衡状态**
>
> **并且通过高位低位与+n必须为2的幂次方两个条件，能够保证扩容后，元素的新位置要么是原位置，要么是原位置加上旧容量大小。**



那当数组长度比较小的时候，我们就需要设计一种比较巧妙的 hash 算法，来避免发生哈希冲突，尽可能地让元素均匀地分布在数组当中。

要达到这个目的，HashMap 在两方面下足了功夫，第一个就是数组的长度必须是 2 的整数次幂，这样可以保证 `hash & (n-1)` 的结果能均匀地分布在数组中。

其作用就相当于 hash % n，n 为数组的长度，比如说数组长度是 16，hash 值为 20，那么 20 % 16 = 4，也就是说 20 这个元素应该放在数组的第 4 个位置；hash 值为 23，那么 23 % 16 = 7，也就是说 23 这个元素应该放在数组的第 7 个位置。

`&` 操作的结果就是哈希值的高位全部归零，只保留 n 个低位，用来做数组下标访问。

比如说 hash & (24−1) 的结果实际上是取 hash 的低 4 位，这四位能表示的取值范围刚好是 0000 到 1111，也就是 0 到 15，正好是数组长度为 16 的下标范围。

以初始长度 16 为例，16-1=15。2 进制表示是`0000 0000 0000 0000 0000 0000 0000 1111`。和某个哈希值做 `&` 运算，结果就是截取了最低的四位。

[![三分恶面渣逆袭：哈希&运算](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161524378.png)](https://camo.githubusercontent.com/8b7face134724af84bae37a9324629646b57aefd67e1a5dee41a8884fece83e0/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d31352e706e67)

那问题又来了，那么大一个哈希值，也只取最后 4 位，不就等于哈希值的高位都丢弃了吗？

比如说 1111 1111 1111 1111 1111 1111 1111 1111，取最后 4 位，也就是 1111。

比如说 1110 1111 1111 1111 1111 1111 1111 1111，取最后 4 位，也是 1111。

不就发生哈希冲突了吗？

这时候 hash 函数 `(h = key.hashCode()) ^ (h >>> 16)` 就派上用场了呀。

![image-20241216152837612](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161528673.png)

将哈希值无符号右移 16 位，意味着原哈希值的高 16 位被移到了低 16 位的位置。这样，原始哈希值的高 16 位和低 16 位就可以参与到最终用于索引计算的低位中。

选择 16 位是因为它是 32 位整数的一半，这样处理既考虑了高位的信息，又没有完全忽视低位原本的信息，尝试达到一个平衡状态。

举个例子（数组长度为 16）。

- 第一个数：h1 = 0001 0010 0011 0100 0101 0110 0111 1000
- 第二个数：h2 = 0001 0010 0011 0101 0101 0110 0111 1000

如果没有 hash 函数，直接取低 4 位，那么 h1 和 h2 的低 4 位都是 1000，也就是两个数都会放在数组的第 8 个位置。

来看一下 hash 函数的处理过程。

①、对于第一个数`h1`的计算：

```cmd
原始: 0001 0010 0011 0100 0101 0110 0111 1000
右移: 0000 0000 0000 0000 0001 0010 0011 0100
异或: ---------------------------------------
结果: 0001 0010 0011 0100 0100 0100 0100 1100
```

②、对于第二个数`h2`的计算：

```cmd
原始: 0001 0010 0011 0101 0101 0110 0111 1000
右移: 0000 0000 0000 0000 0001 0010 0011 0101
异或: ---------------------------------------
结果: 0001 0010 0011 0101 0100 0100 0100 1101
```

通过上述计算，我们可以看到`h1`和`h2`经过`h ^ (h >>> 16)`操作后得到了不同的结果。

现在，考虑数组长度为 16 时（需要最低 4 位来确定索引）：

- 对于`h1`的最低 4 位是`1100`（十进制中为 12）
- 对于`h2`的最低 4 位是`1101`（十进制中为 13）

这样，`h1`和`h2`就会被分别放在数组的第 12 个位置和第 13 个位置上，避免了哈希冲突。



### 为什么 HashMap 的容量是 2 的整数次幂呢？为什么不取模而是按位与？

因为（数组长度-1）正好相当于一个“低位掩码”——这个掩码的低位最好全是 1，这样 & 操作才有意义，否则结果就肯定是 0。

2 的整次幂（或者叫 2 的整数倍）刚好是偶数，偶数-1 是奇数，奇数的二进制最后一位是 1，保证了 `hash &(length-1)` 的最后一位可能为 0，也可能为 1（取决于 hash 的值），即 & 运算后的结果可能为偶数，也可能为奇数，这样便可以保证哈希值的均匀分布。

```cmd
	 10100101 11000100 00100101
&	 00000000 00000000 00001111
----------------------------------
	 00000000 00000000 00000101
```

#### hashCode 对数组长度取模定位数组下标，这块有没有优化策略？

取模运算（Modulo Operation）和取余运算（Remainder Operation）从严格意义上来讲，是两种不同的运算方式，它们在计算机中的实现也不同。

在 Java 中，通常使用 % 运算符来表示取余，用 `Math.floorMod()` 来表示取模。

- 当操作数都是正数的话，取模运算和取余运算的结果是一样的。
- 只有当操作数出现负数的情况，结果才会有所不同。
- **取模运算的商向负无穷靠近；取余运算的商向 0 靠近**。这是导致它们两个在处理有负数情况下，结果不同的根本原因。
- 当数组的长度是 2 的 n 次方，或者 n 次幂，或者 n 的整数倍时，取模运算/取余运算可以用位运算来代替，效率更高，毕竟计算机本身只认二进制嘛。

那为什么 HashMap 在计算下标的时候，并没有直接使用取余运算（或者取模运算），而是直接使用位与运算 & 呢？

这其实就是 HashMap 的一个优化策略。

> [!IMPORTANT]
>
> **hash % length 等价于 hash & (length - 1)**

在计算机中，位运算的速度要远高于取余运算，因为计算机本质上就是二进制嘛。



### :star:如果初始化 HashMap，传一个 17 容量，它会怎么处理？

> [!IMPORTANT]
>
> **HashMap的初始化会向将传入的参数上取2的幂次方作为容量，算法是通过不断右移（`>>>`）并与自身进行或运算（`|=`），将 n 的二进制表示中的所有低位设置为 1。**

果你传入 17 作为初始容量，HashMap 实际上会被初始化为大小为 32 的哈希表。

在 HashMap 的初始化构造方法中，有这样⼀段代码：

```java
public HashMap(int initialCapacity, float loadFactor) {
 ...
 this.loadFactor = loadFactor;
 this.threshold = tableSizeFor(initialCapacity);
}
```

阀值 threshold 会通过⽅法` tableSizeFor()` 进⾏计算。

**接下来通过不断右移（`>>>`）并与自身进行或运算（`|=`），将 n 的二进制表示中的所有低位设置为 1。**

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

- `n |= n >>> 1;` 把 n 的二进制表示中最高位的 1 之后的一个 0 变成 1。
- `n |= n >>> 2;` 接着把后两位中的 0 都变成 1。
- 依此类推，直到 `n |= n >>> 16;`，此时 n 的二进制表示中，从最高位的 1 开始到最低位，都变成了 1。

③、如果 n 小于 0，说明 cap 是负数，直接返回 1（理论上哈希表的大小不应该是负数或 0）。

如果 n 大于或等于 MAXIMUM_CAPACITY（通常是$2^{30}$），则返回 MAXIMUM_CAPACITY。

否则，返回 n + 1，这是因为 n 的所有低位都是 1，所以 n + 1 就是大于 cap 的最小的 2 的幂次方。



#### 初始化 HashMap 的时候需要传入容量值吗？

在创建 HashMap 时可以指定初始容量值。这个容量是指 Map 内部用于存储数据的数组大小。

因为每次扩容时，HashMap 需要新分配一个更大的数组并重新将现有的元素插入到这个新数组中，这个过程相对耗时，尤其是当 Map 中已有大量数据时。

**HashMap 将使用默认的初始容量 16。**



### 你还知道哪些哈希函数的构造方法呢？

HashMap 里哈希构造函数的方法叫：

- **除留取余法**：`H(key)=key%p(p<=N)`，关键字除以一个不大于哈希表长度的正整数 p，所得余数为地址，当然 HashMap 里进行了优化改造，效率更高，散列也更均衡。

除此之外，还有这几种常见的哈希函数构造方法：

- **直接定址法**

  直接根据`key`来映射到对应的数组位置，例如 1232 放到下标 1232 的位置。

- **数字分析法**

  取`key`的某些数字（例如十位和百位）作为映射的位置

- **平方取中法**

  取`key`平方的中间几位作为映射的位置

- **折叠法**

  将`key`分割成位数相同的几段，然后把它们的叠加和作为映射的位置

[![散列函数构造](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161544336.png)](https://camo.githubusercontent.com/adc1236a01c7999c1d8d5a1a75582aaa26ec12467c5a2ee769aa34e12ce7a69c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d31392e706e67)





### 解决哈希冲突有哪些方法呢？

解决哈希冲突的方法我知道的有 3 种：

①、再哈希法

准备两套哈希算法，当发生哈希冲突的时候，使用另外一种哈希算法，直到找到空槽为止。对哈希算法的设计要求比较高。

②、开放地址法

遇到哈希冲突的时候，就去寻找下一个空的槽。有 3 种方法：

- 线性探测：从冲突的位置开始，依次往后找，直到找到空槽。
- 二次探测：从冲突的位置 x 开始，第一次增加 12 个位置，第二次增加 22，直到找到空槽。
- 双重哈希：和再哈希法类似，准备多个哈希函数，发生冲突的时候，使用另外一个哈希函数。

[![三分恶面渣逆袭：拉链法 VS 开放地址法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161545674.png)](https://camo.githubusercontent.com/dc205523861aa2e790d65b5acdfddb07fcfed9a7f2ec31a67691f8d4d8dc15fc/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d32302e706e67)

③、拉链法

也就是所谓的链地址法，当发生哈希冲突的时候，使用链表将冲突的元素串起来。**HashMap 采用的正是拉链法。**



#### 怎么判断 key 相等呢？

`HashMap`判断两个`key`是否相等，依赖于`key`的`equals()`方法和`hashCode()`方法，以及 `==` 运算符。

```java
if (e.hash == hash &&
((k = e.key) == key || (key != null && key.equals(k))))
```

①、**hashCode()** ：首先，使用`key`的`hashCode()`方法计算`key`的哈希码。由于不同的`key`可能有相同的哈希码，`hashCode()`只是第一步筛选。

②、**equals()** ：当两个`key`的哈希码相同时，`HashMap`还会调用`key`的`equals()`方法进行精确比较。只有当`equals()`方法返回`true`时，两个`key`才被认为是完全相同的。

③、==：当然了，如果两个`key`的引用指向同一个对象，那么它们的`hashCode()`和`equals()`方法都会返回`true`，所以在 equals 判断之前会优先使用`==`运算符判断一次。



### 为什么 HashMap 链表转红黑树的阈值为 8 ,红黑树转回链表的阈值是 6?

树化发生在 table 数组的长度大于 64，且链表的长度大于 8 的时候。

红黑树节点的大小大概是普通节点大小的两倍，所以**转红黑树，牺牲了空间换时间，更多的是一种兜底的策略，保证极端情况下的查找效率。**

阈值为什么要选 8 呢？和统计学有关。理想情况下，使用随机哈希码，链表里的节点符合**泊松分布**，出现节点个数的概率是递减的，节点个数为 8 的情况，发生概率仅为`0.00000006`。

至于**红黑树转回链表的阈值为什么是 6，而不是 8**？是因为如果这个阈值也设置成 8，**假如发生碰撞，节点增减刚好在 8 附近，会发生链表和红黑树的不断转换，导致资源浪费。**

```java
In usages with well-distributed user hashCodes, tree bins 
are rarely used.  Ideally, under random hashCodes, the 
frequency of nodes in bins follows a Poisson distribution 
(http://en.wikipedia.org/wiki/Poisson_distribution) with a 
parameter of about 0.5 on average for the default resizing 
threshold of 0.75, although with a large variance because 
of resizing granularity. Ignoring variance, the expected 
occurrences of list size k are (exp(-0.5) * pow(0.5, k) / 
factorial(k)). The first values are:
 
 0:    0.60653066
 1:    0.30326533
 2:    0.07581633
 3:    0.01263606
 4:    0.00157952
 5:    0.00015795
 6:    0.00001316
 7:    0.00000094
 8:    0.00000006
 more: less than 1 in ten million
```





### 扩容在什么时候呢？为什么扩容因子是 0.75？

HashMap 会在存储的键值对数量超过阈值（即容量 * 加载因子）时进行扩容。

默认的加载因子是 0.75，这意味着当 HashMap 填满了大约 75%的容量时，就会进行扩容。

默认的初始容量是 16，那就是大于`16x0.75=12`时，就会触发第一次扩容操作。



#### 那么为什么选择了 0.75 作为 HashMap 的默认加载因子呢？

简单来说，这是对`空间`成本和`时间`成本平衡的考虑。

- 假如我们设的比较大，元素比较多，空位比较少的时候才扩容，那么发生哈希冲突的概率就增加了，查找的时间成本就增加了。
- 我们设的比较小的话，元素比较少，空位比较多的时候就扩容了，发生哈希碰撞的概率就降低了，查找时间成本降低，但是就需要更多的空间去存储元素，空间成本就增加了





### 那扩容机制了解吗？

扩容时，HashMap 会创建一个新的数组，其容量是原数组容量的两倍。然后将键值对放到新计算出的索引位置上。一部分索引不变，另一部分索引为“**原索引+旧容量**”。



我们用 JDK 8 的哈希算法来计算一下哈希值，就会发现别有洞天。

假设扩容前的数组长度为 16（n-1 也就是二进制的 0000 1111，1X${2^0}$+1X${2^1}$+1X${2^2}$+1X${2^3}$=1+2+4+8=15），key1 为 5（二进制为 0000 0101），key2 为 21（二进制为 0001 0101）。

- key1 和 n-1 做 & 运算后为 0000 0101，也就是 5；
- key2 和 n-1 做 & 运算后为 0000 0101，也就是 5。
- 此时哈希冲突了，用拉链法来解决哈希冲突。

现在，HashMap 进行了扩容，容量为原来的 2 倍，也就是 32（n-1 也就是二进制的 0001 1111，1X${2^0}$+1X${2^1}$+1X${2^2}$+1X${2^3}$+1X${2^4}$=1+2+4+8+16=31）。

- key1 和 n-1 做 & 运算后为 0000 0101，也就是 5；
- key2 和 n-1 做 & 运算后为 0001 0101，也就是 21=5+16，也就是数组扩容前的位置+原数组的长度。

神奇吧？

[![三分恶面渣逆袭：扩容位置变化](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161557101.png)](https://camo.githubusercontent.com/67bb90b032e1806b97f8f5231bfdc94af25efcfcb8a5dff111e447c54182ffd4/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d32362e706e67)

换句话说，在 JDK 8 的新 hash 算法下，数组扩容后的索引位置，要么就是原来的索引位置，要么就是“原索引+原来的容量”，遵循一定的规律。

![image-20241216155934868](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161559954.png)

当然了，这个功劳既属于新的哈希算法，也离不开 n 为 2 的整数次幂这个前提，这是它俩通力合作后的结果 `hash & (newCapacity - 1)`。



#### 那你说说扩容的时候每个节点都要进行位运算吗，如果我这个 HashMap 里面有几十万条数据，都要进行位运算吗？

在 JDK 8 的新 hash 算法下，数组扩容后的索引位置，要么就是原来的索引位置，要么就是“原索引+原来的容量”，遵循一定的规律。

具体来说，就是判断原哈希值的高位中新增的那一位是否为 1，如果是，该元素会被移动到原位置加上旧容量的位置；如果不是，则保持在原位置。

所以，尽管有几十万条数据，每个数据项的位置决定仅需要一次简单的位运算。位运算的计算速度非常快，因此，尽管扩容操作涉及遍历整个哈希表并对每个节点进行操作，但这部分操作的计算成本是相对较低的。



### JDK 8 对 HashMap 主要做了哪些优化呢？为什么？

相比较 JDK 7，JDK 8 的 HashMap 主要做了四点优化：

①、底层数据结构**由数组 + 链表改成了数组 + 链表或红黑树**的结构。

原因：如果多个键映射到了同一个哈希值，链表会变得很长，在最坏的情况下，当所有的键都映射到同一个桶中时，性能会退化到 O(n)，而红黑树的时间复杂度是 O(logn)。

②、链表的插入方式由**头插法改为了尾插法**。

原因：头插法虽然简单快捷，但扩容后容易改变原来链表的顺序。

③、**扩容的时机由插入时判断改为插入后判断**。

原因：可以避免在每次插入时都进行不必要的扩容检查，因为有可能插入后仍然不需要扩容。

④、**优化了哈希算法**。

JDK 7 进行了多次移位和异或操作来计算元素的哈希值。

JDK 8 优化了这个算法，只进行了一次异或操作，但仍然能有效地减少冲突。并且能够保证扩容后，元素的新位置要么是原位置，要么是原位置加上旧容量大小。

![image-20241216160604510](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161606584.png)



### 你能自己设计实现一个 HashMap 吗？

整体的设计：

- 散列函数：hashCode()+除留余数法
- 冲突解决：链地址法
- 扩容：节点重新 hash 获取位置

[![自定义HashMap整体结构](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161607923.png)](https://camo.githubusercontent.com/3f3f63c677406228526e796e066e3eb7818f539120a7ec59dd1b6e835d07c23d/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d32392e706e67)

手撕代码：

```java
import java.util.Objects;

public class ThirdHashMap<K, V> {
    /**
     * 节点类
     */
    class Node<K, V> {
        // 键值对
        private K key;
        private V value;

        // 链表，后继
        private Node<K, V> next;

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
        }

        public Node(K key, V value, Node<K, V> next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    // 默认容量
    private final int DEFAULT_CAPACITY = 16;
    // 负载因子
    private final float LOAD_FACTOR = 0.75f;
    // HashMap的大小
    private int size;
    // 桶数组
    private Node<K, V>[] buckets;

    /**
     * 无参构造器，设置桶数组默认容量
     */
    @SuppressWarnings("unchecked") // 避免泛型数组创建警告
    public ThirdHashMap() {
        buckets = new Node[DEFAULT_CAPACITY];
        size = 0;
    }

    /**
     * 有参构造器，指定桶数组容量
     */
    @SuppressWarnings("unchecked")
    public ThirdHashMap(int capacity) {
        buckets = new Node[capacity];
        size = 0;
    }

    /**
     * 哈希函数，获取地址
     */
    private int getIndex(K key, int length) {
        // 确保 key 不为空
        int hashCode = Objects.hashCode(key);
        // 和桶数组长度取余
        int index = hashCode % length;
        return Math.abs(index);
    }

    /**
     * put方法
     */
    public void put(K key, V value) {
        if (size >= buckets.length * LOAD_FACTOR) resize();
        putVal(key, value, buckets);
    }

    /**
     * 将元素存入指定的 node 数组
     */
    private void putVal(K key, V value, Node<K, V>[] table) {
        // 获取位置
        int index = getIndex(key, table.length);
        Node<K, V> node = table[index];
        // 插入的位置为空
        if (node == null) {
            table[index] = new Node<>(key, value);
            size++;
            return;
        }
        // 插入位置不为空，说明发生冲突，使用链地址法，遍历链表
        Node<K, V> current = node;
        while (current != null) {
            // 如果 key 相同，就覆盖掉
            if ((current.key.hashCode() == key.hashCode()) &&
                    (current.key == key || current.key.equals(key))) {
                current.value = value;
                return;
            }
            // 遍历到链表尾部
            if (current.next == null) break;
            current = current.next;
        }
        // 当前 key 不在链表中，插入链表尾部
        current.next = new Node<>(key, value);
        size++;
    }

    /**
     * 扩容
     */
    @SuppressWarnings("unchecked")
    private void resize() {
        // 创建一个两倍容量的桶数组
        Node<K, V>[] newBuckets = new Node[buckets.length * 2];
        // 将当前元素重新散列到新的桶数组
        rehash(newBuckets);
        buckets = newBuckets;
    }

    /**
     * 重新散列当前元素
     */
    private void rehash(Node<K, V>[] newBuckets) {
        // map 大小重新计算
        size = 0;
        // 将旧的桶数组的元素全部刷到新的桶数组里
        for (Node<K, V> bucket : buckets) {
            if (bucket == null) {
                continue;
            }
            Node<K, V> node = bucket;
            while (node != null) {
                // 将元素放入新数组
                putVal(node.key, node.value, newBuckets);
                node = node.next;
            }
        }
    }

    /**
     * 获取元素
     */
    public V get(K key) {
        // 获取 key 对应的地址
        int index = getIndex(key, buckets.length);
        if (buckets[index] == null) return null;
        Node<K, V> node = buckets[index];
        while (node != null) {
            if ((node.key.hashCode() == key.hashCode()) &&
                    (node.key == key || node.key.equals(key))) {
                return node.value;
            }
            node = node.next;
        }
        return null;
    }

    /**
     * 返回 HashMap 大小
     */
    public int size() {
        return size;
    }
}

```



### HashMap 是线程安全的吗？多线程下会有什么问题？

> [!IMPORTANT]
>
> **头插法和尾插法分别有什么问题**
>
> **1. 头插法的问题**
>
> 在JDK 1.7及之前的版本中，`HashMap`的链表插入采用的是头插法。头插法的主要问题如下：
>
> - **破坏插入顺序**：头插法会将新插入的元素放在链表头部，导致链表的**顺序与插入顺序相反**。
> - **并发扩容问题**：在多线程并发扩容时，**头插法容易导致链表成环，从而引发死循环**。这是因为多个线程可能同时对链表进行操作，导致链表的头节点被反复修改，最终可能丢失尾部的空指针。
> - **CPU占用过高**：一旦发生链表成环，程序可能会陷入无限循环，导致CPU占用率飙升至100%，严重影响程序性能。
>
> **2. 尾插法的问题**
>
> 从JDK 1.8开始，`HashMap`的链表插入改为尾插法。尾插法虽然解决了头插法的大部分问题，但也存在一些不足：
>
> - **扩容效率较低**：理论上，尾插法在扩容时的时间复杂度为O(N)，而头插法为O(1)。这是因为**尾插法需要遍历链表找到尾部节点**。
> - **不满足时间局部性原理**：尾插法将新插入的元素放在链表尾部，而**最近插入的元素通常是最有可能被访问的**。这种插入方式可能导致访问效率降低。
> - **并发问题依然存在**：虽然尾插法避免了链表成环的问题，但在并发环境下，`HashMap`仍然可能面临数据覆盖等问题。因此，在多线程场景下，仍然建议使用`ConcurrentHashMap`。



HashMap 不是线程安全的，主要有以下几个问题：

①、多线程下扩容会死循环。JDK1.7 中的 HashMap 使用的是头插法插入元素，在多线程的环境下，扩容的时候就有可能导致出现环形链表，造成死循环。

[![二哥的 Java 进阶之路](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161655213.png)](https://camo.githubusercontent.com/6c91c49f407cf038991f21f4329056dc38ce69d920179d4d89ec952ca0cabbb3/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f636f6c6c656374696f6e2f686173686d61702d7468726561642d6e6f736166652d30372e706e67)

不过，JDK 8 时已经修复了这个问题，扩容时会保持链表原来的顺序。

②、多线程的 put 可能会导致元素的丢失。因为计算出来的位置可能会被其他线程的 put 覆盖。本来哈希冲突是应该用链表的，但多线程时由于没有加锁，相同位置的元素可能就被干掉了。

[![二哥的 Java 进阶之路](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161655427.png)](https://camo.githubusercontent.com/f3bf0857004eb18d1220520b58db5095f4fb69b64c49e02a71732c29f495dcbb/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f636f6c6c656374696f6e2f686173686d61702d7468726561642d6e6f736166652d31302e706e67)

③、put 和 get 并发时，可能导致 get 为 null。线程 1 执行 put 时，因为元素个数超出阈值而导致出现扩容，线程 2 此时执行 get，就有可能出现这个问题。

因为线程 1 执行完 table = newTab 之后，线程 2 中的 table 此时也发生了变化，此时去 get 的时候当然会 get 到 null 了，因为元素还没有转移。



#### map的同步和非同步?

**Hashtable** 是 Map 接口的一个早期的同步实现，它的所有方法都是同步的，即**每个方法都用 synchronized 关键字修饰**，以确保线程安全。

随着 JDK 版本的升级，Java 提供了更好的线程安全 Map 实现，如 **ConcurrentHashMap。**

如果是在单线程环境下，可以使用 HashMap。



### 有什么办法能解决 HashMap 线程不安全的问题呢？

HashMap 不是线程安全的，因此在早期的 JDK 版本中，是用 Hashtable 来保证线程安全的。

①、Hashtable 是直接在方法上加 [synchronized 关键字](https://javabetter.cn/thread/synchronized-1.html)，比较粗暴。

因此在 Java 的后期版本中，更推荐使用 [ConcurrentHashMap](https://javabetter.cn/thread/ConcurrentHashMap.html)和`Collections.synchronizedMap(Map)`包装器。

②、`Collections.synchronizedMap` 返回的是 [Collections](https://javabetter.cn/common-tool/collections.html) 工具类的内部类。

内部是通过 synchronized 对象锁来保证线程安全的。

③、[ConcurrentHashMap](https://javabetter.cn/thread/ConcurrentHashMap.html) 在 JDK 7 中使用了**分段锁**来保证线程安全，在 JDK 8 中使用了**CAS（Compare-And-Swap）+ synchronized 关键字**





### HashMap 内部节点是有序的吗？

HashMap 是无序的，根据 hash 值随机插入。如果想使用有序的 Map，可以使用 LinkedHashMap 或者 TreeMap。



### 讲讲 LinkedHashMap 怎么实现有序的？

LinkedHashMap 维护了一个双向链表，有头尾节点，同时 LinkedHashMap 节点 Entry 内部除了继承 HashMap 的 Node 属性，还有 before 和 after 用于标识前置节点和后置节点。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161715840.png" alt="image-20241216171512753" style="zoom:50%;" />

可以实现按插入的顺序或访问顺序排序。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161714728.png" alt="LinkedHashMap实现原理" style="zoom: 80%;" />](https://camo.githubusercontent.com/42753b0e7c0618b2cfd2bf1b2210b1d03417a81e369826754c3b5ae61a9c4777/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d33342e706e67)



### 讲讲 TreeMap 怎么实现有序的？

TreeMap 通过 key 的比较器来决定元素的顺序，如果没有指定比较器，那么 **key 必须实现 Comparable 接口**

TreeMap 的底层是红黑树，红黑树是一种自平衡的二叉查找树，**每个节点都大于其左子树中的任何节点，小于其右子节点树种的任何节点。**

![image-20241216172216762](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161722852.png)



### TreeMap 和 HashMap 的区别

①、HashMap 是基于数组+链表+红黑树实现的，put 元素的时候会先计算 key 的哈希值，然后通过哈希值计算出元素在数组中的存放下标，然后将元素插入到指定的位置，如果发生哈希冲突，会使用链表来解决，如果链表长度大于 8，会转换为红黑树。

②、**TreeMap 是基于红黑树实现的，put 元素的时候会先判断根节点是否为空，如果为空，直接插入到根节点，如果不为空，会通过 key 的比较器来判断元素应该插入到左子树还是右子树。**

在没有发生哈希冲突的情况下，HashMap 的查找效率是 `O(1)`。**适用于查找操作比较频繁**的场景。

TreeMap 的查找效率是 `O(logn)`。并且保证了元素的顺序，因此**适用于需要大量范围查找或者有序遍历**的场景。





## Set

### 讲讲 HashSet 的底层实现？

HashSet 其实是由 HashMap 实现的，只不过**值由一个固定的 Object 对象填充**，而键用于操作。

HashSet 主要用于去重

HashSet 会自动去重，因为它是用 HashMap 实现的，HashMap 的键是唯一的（哈希值），**相同键的值会覆盖掉原来的值**，于是第二次 set.add("沉默") 的时候就覆盖了第一次的 set.add("沉默")。

[![三分恶面渣逆袭：HashSet套娃](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161728653.png)](https://camo.githubusercontent.com/d25d5bdb3a6f8394354dd40a7a5045d068df7d08b958911a298006c7c4ecf297/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d33362e706e67)



#### HashSet 和 ArrayList 的区别

- ArrayList 是基于动态数组实现的，HashSet 是基于 HashMap 实现的。
- ArrayList 允许重复元素和 null 值，可以有多个相同的元素；HashSet 保证每个元素唯一，不允许重复元素，基于元素的 hashCode 和 equals 方法来确定元素的唯一性。
- ArrayList 保持元素的插入顺序，可以通过索引访问元素；**HashSet 不保证元素的顺序，元素的存储顺序依赖于哈希算法**，并且可能随着元素的添加或删除而改变。



#### HashSet 怎么判断元素重复，重复了是否 put

**HashSet 的 add 方法是通过调用 HashMap 的 put 方法实现的**：

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

所以 HashSet 判断元素重复的逻辑底层依然是 HashMap 的底层逻辑

HashSet 通过元素的哈希值来判断元素是否重复，如果重复了，会覆盖原来的值。

```java
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```
