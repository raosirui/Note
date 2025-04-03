# JVM

### 什么是 JVM?

JVM 大致可以划分为三个部分：类加载器、运行时数据区和执行引擎。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503312304109.png" alt="截图来源于网络" style="zoom: 60%;" />](https://camo.githubusercontent.com/fc6107f4bb47f25688efdde784e66ba7e6f6b3a2c72324839512aabe1daf30a1/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f776861742d69732d6a766d2d32303233313033303138353734322e706e67)

① 类加载器，负责从文件系统、网络或其他来源加载 Class 文件，将 Class 文件中的二进制数据读入到内存当中。

② 运行时数据区，JVM 在执行 Java 程序时，需要在内存中分配空间来处理各种数据，这些内存区域按照 Java 虚拟机规范可以划分为方法区、堆、虚拟机栈、程序计数器和本地方法栈。

③ 执行引擎，也是 JVM 的心脏，负责执行字节码。它包括一个虚拟处理器、即时编译器 JIT 和垃圾回收器。





JVM，也就是 Java 虚拟机，它是 Java 实现跨平台的基石。

程序运行之前，需要先通过编译器将 Java 源代码文件编译成 Java 字节码文件；

**程序运行时，JVM 会对字节码文件进行逐行解释，翻译成机器码指令，并交给对应的操作系统去执行。**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503312238431.png" alt="三分恶面渣逆袭：Java语言编译运行" style="zoom: 50%;" />

这样就实现了 Java 一次编译，处处运行的特性。



#### 说说 JVM 的其他特性？

①、JVM 可以**自动管理内存**，通过垃圾回收器回收不再使用的对象并释放内存空间。

②、JVM 包含一个**即时编译器 JIT**，它可以在运行时将热点代码缓存到 codeCache 中，下次执行的时候不用再一行一行的解释，而是直接执行缓存后的机器码，执行效率会大幅提高。

[![截图来自美团技术](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503312238019.png)](https://camo.githubusercontent.com/afd75b0c08b756159d632ab2491a8b160942a8ef1e34ea506f9fe5b700d61295/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f6a766d2f6a69742d39613632666330322d316136612d343531652d626232622d3139666330383664356265302e706e67)

③、任何可以通过 Java 编译的语言，比如说 Groovy、Kotlin、Scala 等，都可以在 JVM 上运行。



#### 为什么要学习 JVM？

学习 JVM 可以帮助我们开发者更好地优化程序性能、避免内存问题。

比如说了解 JVM 的内存模型和垃圾回收机制，可以帮助我们更合理地配置内存、减少 GC 停顿。

比如说掌握 JVM 的类加载机制可以帮助我们排查类加载冲突或异常。

再比如说，JVM 还提供了很多调试和监控工具，可以帮助我们分析内存和线程的使用情况，从而解决内存溢出内存泄露等问题。







## 内存管理

### 能说一下 JVM 的内存区域吗？

按照 Java 虚拟机规范，JVM 的内存区域可以细分为`程序计数器`、`虚拟机栈`、`本地方法栈`、`堆`和`方法区`。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503312309225.png" alt="image-20250331230936084" style="zoom:50%;" />

其中`方法区`和`堆`是线程共享的，`虚拟机栈`、`本地方法栈`和`程序计数器`是线程私有的。



#### 介绍一下程序计数器？

程序计数器也被称为 PC 寄存器，是一块较小的内存空间。它可以看作是当前线程所执行的**字节码行号指示器**。

作用：

- 在加载阶段，虚拟机将字节码文件中的指令读取到内存之后，会将原文件中的偏移量转换成内存地址。每一条字节码指令都会拥有一个**内存地址**。如0x000001f248c072c0
- 程序计数器可以控制程序指令的进行，实现**分支、跳转、异常**等逻辑。
- 在多线程执行情况下，Java虚拟机需要通过程序计数器记录CPU切换前解释执行到那一句指令并继续解释运行。



#### 介绍一下 Java 虚拟机栈？

Java 虚拟机栈的**生命周期与线程相同**。

当线程执行一个方法时，会创建一个对应的[栈帧](https://javabetter.cn/jvm/stack-frame.html)，用于存储局部变量表、操作数栈、动态链接、方法出口等信息，然后栈帧会被压入虚拟机栈中。当方法执行完毕后，栈帧会从虚拟机栈中移除。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503312310209.png" alt="三分恶面渣逆袭：Java虚拟机栈" style="zoom:50%;" />](https://camo.githubusercontent.com/53ec48e38765a9125799fc9a87a0ce22595c8fdf92c510a5b625276a6094c5b3/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a766d2d342e706e67)



#### 一个什么都没有的空方法，空的参数都没有，那局部变量表里有没有变量？

```java
public class VarDemo1 {
    public void emptyMethod() {
        // 什么都没有
    }

    public static void staticEmptyMethod() {
        // 什么都没有
    }
}
```

对于**静态方法**，由于**不需要访问实例对象 this**，因此在局部变量表中不会有任何变量。

对于**非静态方法**，即使是一个完全空的方法，局部变量表中也会有一个用于**存储 this 引用的变量**。this 引用指向当前实例对象，在方法调用时被**隐式传入**。





#### 介绍一下本地方法栈？

本地方法栈与虚拟机栈相似，区别在于**虚拟机栈是为 JVM 执行 Java 编写的方法服务的**，而**本地方法栈是为 Java 调用[本地 native 方法](https://javabetter.cn/oo/native-method.html)服务**的，通常由 C/C++ 编写。

在本地方法栈中，主要存放了 native 方法的局部变量、动态链接和方法出口等信息。当一个 Java 程序调用一个 native 方法时，JVM 会切换到本地方法栈来执行这个方法。



#### 介绍一下本地方法栈的运行场景？

当 Java 应用需要与操作系统底层或硬件交互时，通常会用到本地方法栈。

比如**调用操作系统的特定功能，如内存管理、文件操作、系统时间、系统调用等**。





#### native 方法解释一下？

推荐阅读：[手把手教你用 C语言实现 Java native 本地方法](https://javabetter.cn/oo/native-method.html)

native 方法是在 Java 中通过 [native 关键字](https://javabetter.cn/basic-extra-meal/48-keywords.html)声明的，用于调用非 Java 语言，如 C/C++ 编写的代码。Java 可以通过 JNI，也就是 Java Native Interface 与**底层系统、硬件设备、或者本地库**进行交互。





#### 介绍一下 Java 堆？

堆是 JVM 中最大的一块内存区域，**被所有线程共享**，**在 JVM 启动时创建**，主要**用来存储 new 出来的对象。**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503312320176.png" alt="二哥的 Java 进阶之路：堆" style="zoom: 67%;" />

Java 中“几乎”所有的对象都会在堆中分配，堆也是[垃圾收集器](https://javabetter.cn/jvm/gc-collector.html)管理的目标区域。

从内存回收的角度来看，由于垃圾收集器大部分都是基于分代收集理论设计的，所以堆又被细分为`新生代`、`老年代`、`Eden空间`、`From Survivor空间`、`To Survivor空间`等。

[![三分恶面渣逆袭：Java 堆内存结构](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503312319243.png)](https://camo.githubusercontent.com/b89884570d24cae83eb155408a6c46276f2bb5affc51e845813cdbf8cb8e9127/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a766d2d352e706e67)

随着 **[JIT 编译器](https://javabetter.cn/jvm/jit.html)**的发展和**逃逸技术**的逐渐成熟，“所有的对象都会分配到堆上”就不再那么绝对了。

从 JDK 7 开始，JVM 默认开启了**逃逸分析，意味着如果某些方法中的对象引用没有被返回或者没有在方法体外使用，也就是未逃逸出去，那么对象可以直接在栈上分配内存。**



#### 堆和栈的区别是什么？

堆属于**线程共享**的内存区域，几乎所有 new 出来的对象都会堆上分配，生命周期不由单个方法调用所决定，**可以在方法调用结束后继续存在，直到不再被任何变量引用，最后被垃圾收集器回收。**

栈属于**线程私有**的内存区域，主要存储**局部变量、方法参数、对象引用**等，通常**随着方法调用的结束而自动释放**，不需要垃圾收集器处理。



#### 介绍一下方法区？

**方法区并不真实存在**，属于 Java 虚拟机规范中的一个逻辑概念，用于**存储已被 JVM 加载的类信息、常量、静态变量、即时编译器编译后的代码缓存等**。

在 HotSpot 虚拟机中，方法区的实现称为**永久代** PermGen，但在 Java 8 及之后的版本中，已经被**元空间** Metaspace 所替代。



#### 变量存在堆栈的什么位置？

对于**局部变量**，它**存储在当前方法栈帧中的局部变量表中**。当方法执行完毕，栈帧被回收，局部变量也会被释放。

```
public void method() {
    int localVar = 100;  // 局部变量，存储在栈帧中的局部变量表里
}
```

对于**静态变量**来说，它存储在 Java 虚拟机规范中的**方法区中**，在 Java 7 中是永久代，在 Java8 及以后 是元空间。

```
public class StaticVarDemo {
    public static int staticVar = 100;  // 静态变量，存储在方法区中
}
```







### 说一下 JDK 1.6、1.7、1.8 内存区域的变化？

JDK 1.6 使用**永久代**来实现方法区：

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504010949466.png" alt="三分恶面渣逆袭：JDK 1.6内存区域" style="zoom:50%;" />](https://camo.githubusercontent.com/28d9107d566f27b84fc699802e8e5ab0f5ecd68aecf5604934bc76ff8bc146fb/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a766d2d362e706e67)

JDK 1.7 时仍然是**永久代**，但发生了一些细微变化，比如**将字符串常量池、静态变量存放到了堆上**。

[
<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504010953889.png" alt="三分恶面渣逆袭：JDK 1.7内存区域" style="zoom:50%;" />](https://camo.githubusercontent.com/7a5356ed6c1e2a783ef6772d6ae25adb309270058c68176b1843108deec3fcdc/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a766d2d372e706e67)

在 JDK 1.8 时，直接在内存中划出了一块区域，叫**元空间**，来取代之前放在 JVM 内存中的永久代，并将**运行时常量池、类常量池都移动到了元空间。**

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504010949888.png" alt="三分恶面渣逆袭：JDK 1.8内存区域" style="zoom:50%;" />](https://camo.githubusercontent.com/17d3744a3793af4cca94a3984f58d4697caf42b6231c63d3a0595457c39ac249/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a766d2d382e706e67)





### 为什么使用元空间替代永久代？

客观上，**永久代**会导致 Java 应用程序更容易出现**内存溢出**的问题，因为它要**受到 JVM 内存大小的限制**。

HotSpot 虚拟机的永久代大小可以通过 `-XX：MaxPermSize` 参数来设置，32 位机器默认的大小为 64M，64 位的机器则为 85M。

而 J9 和 JRockit 虚拟机就不存在这种限制，只要没有触碰到进程可用的内存上限，例如 32 位系统中的 4GB 限制，就不会出问题。

主观上，当 Oracle 收购 BEA 获得了 JRockit 的所有权后，就准备把 JRockit 中的优秀功能移植到 HotSpot 中。

如 Java Mission Control 管理工具。

但因为两个虚拟机对方法区实现有差异，导致这项工作遇到了很多阻力。

考虑到 HotSpot 虚拟机未来的发展，JDK 6 的时候，开发团队就打算放弃永久代了。

JDK 7 的时候，前进了一小步，把原本放在永久代的字符串常量池、静态变量等移动到了堆中。

JDK 8 就终于完成了这项移出工作，这样的好处就是，元空间的大小不再受到 JVM 内存的限制，而是可以像 J9 和 JRockit 那样，**只要系统内存足够，就可以一直用。**





### 对象创建的过程了解吗？

当我们使用 new 关键字创建一个对象时，**JVM 首先会检查 new 指令的参数是否能在常量池中定位到类的符号引用，然后检查这个符号引用代表的类是否已被加载、解析和初始化。如果没有，就先执行类加载。**

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011025845.png" alt="二哥的 Java 进阶之路：对象的创建过程" style="zoom:60%;" />](https://camo.githubusercontent.com/68c582b755ac111499d26dbe41a47593adddefb123840c9ae85f159e5fafa080/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a766d2d32303234303430343039313434352e706e67)

如果已经加载，JVM 会为对象分配内存完成**初始化**，比如数值类型的成员变量初始值是 0，布尔类型是 false，对象类型是 null。

接下来会设置**对象头**，里面包含了对象**是哪个类的实例、对象的哈希码、对象的 GC 分代年龄等**信息。

最后，JVM 会执行构造方法 `<init>` 完成赋值操作，将成员变量赋值为预期的值，比如 `int age = 18`，这样一个对象就创建完成了。



### 对象的销毁过程了解吗？

当对象不再被任何引用指向时，就会变成垃圾。垃圾收集器会通过**可达性分析算法**判断对象是否存活，如果对象不可达，就会被回收。

垃圾收集器通过**标记清除**、**标记复制**、**标记整理**等算法来回收内存，将对象占用的内存空间释放出来。

可以通过 `java -XX:+PrintCommandLineFlags -version` 和 `java -XX:+PrintGCDetails -version` 命令查看 JVM 的 GC 收集器。

[![二哥的 Java 进阶之路：JVM 使用的垃圾收集器](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011035035.png)](https://camo.githubusercontent.com/b95f4987b70bfee4076acce6157b6ae3a48254a2b9cc2370c11bc9724e2ad3b2/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a766d2d32303235303131303131313631382e706e67)

可以看到，我本机安装的 **JDK 8 默认使用的是 `Parallel Scavenge + Parallel Old`。**

不同参数代表对应的垃圾收集器表单：

![image-20250401103642463](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011036543.png)





### 堆内存是如何分配的？

在堆中为对象分配内存时，主要使用两种策略：**指针碰撞**和**空闲列表**。

[![三分恶面渣逆袭：指针碰撞和空闲列表](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011058636.png)](https://camo.githubusercontent.com/93193ebb90a294aa7d434d5793134e23e66096b24a652bf780eaba39ad2de653/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a766d2d31302e706e67)

**指针碰撞**适用于**管理简单、碎片化较少的**内存区域，如**年轻代**；

**空闲列表**适用于**内存碎片化较严重或对象大小差异较大**的场景如**老年代**。



#### 什么是指针碰撞？

假设堆内存是一个连续的空间，分为**两个部分，一部分是已经被使用的内存，另一部分是未被使用的内存**。

在分配内存时，Java 虚拟机会维护一个指针，指向下一个可用的内存地址，每次分配内存时，**只需要将指针向后移动一段距离，如果没有发生碰撞，就将这段内存分配给对象实例。**



#### 什么是空闲列表？

JVM **维护一个列表**，记录堆中**所有未占用的内存块**，每个内存块都记录有大小和地址信息。

当有新的对象请求内存时，JVM 会**遍历空闲列表**，寻找足够大的空间来存放新对象。

分配后，如果选中的内存块未被完全利用，剩余的部分会作为一个新的内存块加入到空闲列表中。







### new 对象时，堆会发生抢占吗？

会。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011115091.png" alt="Baeldung：堆抢占" style="zoom: 67%;" />](https://camo.githubusercontent.com/374e0ed491dbbe22be4365b859089b44d37661b35898809f4c22ceabafcd99d8/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a766d2d32303235303131313130343633382e706e67)

new 对象时，指针会向右移动一个对象大小的距离，假如一个线程 A 正在给字符串对象 s 分配内存，另外一个线程 B 同时为 ArrayList 对象 l 分配内存，两个线程就发生了抢占。



#### JVM 怎么解决堆内存分配的竞争问题？

为了解决堆内存分配的竞争问题，JVM 为每个线程保留了一小块内存空间，被称为 TLAB，也就是**线程本地分配缓冲区**，用于存放该线程分配的对象。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011116767.png" alt="Baeldung：TLAB" style="zoom: 67%;" />](https://camo.githubusercontent.com/be196c3fc02632efdde4d96584cac3a5c766b203d6b12982f82f7706b243515f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a766d2d32303235303131313130353131392e706e67)

**当线程需要分配对象时，直接从 TLAB 中分配**。只有**当 TLAB 用尽或对象太大需要直接在堆中分配时，才会使用全局分配指针。**



 

### 能说一下对象的内存布局吗？

对象的内存布局是由 Java 虚拟机规范定义的，但具体的实现细节各有不同，如 HotSpot 和 OpenJ9 就不一样。

就拿我们常用的 HotSpot 来说吧。

对象在内存中包括三部分：**对象头**、**实例数据**和**对齐填充**。

[![三分恶面渣逆袭：对象的存储布局](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011138915.png)](https://camo.githubusercontent.com/1ddde9b325d07b17cdd858f293101db3dc9a898300d53708f4fd7dbd0f785285/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a766d2d31322e706e67)



#### 说说对象头的作用？

对象头是对象存储在内存中的元信息，包含了Mark Word、类型指针等信息。

**Mark Word** 存储了对象的运行时状态信息，包括**锁、哈希值、GC 标记**等。在 64 位操作系统下占 8 个字节，32 位操作系统下占 4 个字节。

**类型指针**指向对象所属类的元数据，也就是 **Class 对象**，用来**支持多态、方法调用等**功能。

除此之外，如果**对象是数组类型**，还会有一个**额外的数组长度**字段。占 4 个字节。





#### 类型指针会被压缩吗？

类型指针可能会被压缩，以节省内存空间。比如说在开启压缩指针的情况下占 4 个字节，否则占 8 个字节。在 JDK 8 中，**压缩指针默认是开启的。**

可以通过 `java -XX:+PrintFlagsFinal -version | grep UseCompressedOops` 命令来查看 JVM 是否开启了压缩指针。

如果压缩指针开启，输出结果中的 bool UseCompressedOops 值为 true。



#### 实例数据了解吗？

实例数据是对象实际的字段值，也就是成员变量的值，按照字段在类中声明的顺序存储。

```java
class ObjectDemo {
    int age;
    String name;
}
```

JVM 会对这些数据进行对齐/重排，以提高内存访问速度。



#### 对齐填充了解吗？

由于 JVM 的内存模型**要求对象的起始地址是 8 字节对齐**（64 位 JVM 中），因此对象的总大小必须是 8 字节的倍数。

如果对**象头和实例数据的总长度不是 8 的倍数，JVM 会通过填充额外的字节来对齐**。

比如说，如果对象头 + 实例数据 = 14 字节，则需要填充 2 个字节，使总长度变为 16 字节。



#### 为什么非要进行 8 字节对齐呢？

因为 **CPU 进行内存访问时，一次寻址的指针大小是 8 字节，正好是 L1 缓存行的大小。**如果不进行内存对齐，则**可能出现跨缓存行访问，**导致**额外的缓存行加载，CPU 的访问效率就会降低**。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011146639.png" alt="rickiyang：缓存行污染" style="zoom: 33%;" />](https://camo.githubusercontent.com/bfff5ba2380c059ba87b8b65addfc11bcbe4d72fa9a934edde5f26d7ab70871f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a766d2d32303234303332303232323035382e706e67)

比如说上图中 obj1 占 6 个字节，由于没有对齐，导致这一行缓存中多了 2 个字节 obj2 的数据，当 CPU 访问 obj2 的时候，就会导致缓存行刷新。

也就说，**8 字节对齐，是为了效率的提高，以空间换时间的一种方案**。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011147320.png" alt="rickiyang：000 结尾" style="zoom:50%;" />](https://camo.githubusercontent.com/8767b08d7641f1c563b0f2564bde525b10bdf8770c1988bd408a91f407be11e7/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a766d2d32303234303332303232323633312e706e67)



#### new Object() 对象的内存大小是多少？

推荐阅读：[高端面试必备：一个 Java 对象占用多大内存](https://www.cnblogs.com/rickiyang/p/14206724.html)

一般来说，目前的操作系统都是 64 位的，并且 JDK 8 中的压缩指针是默认开启的，因此在 64 位的 JVM 上，`new Object()`的大小是 16 字节（12 字节的对象头 + 4 字节的对齐填充）。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202504011152856.png" alt="rickiyang：Java 对象模型" style="zoom:50%;" />](https://camo.githubusercontent.com/65a3f9c880c28bb9adcc772ae780bdf270a950835d99dbeea0797f653a6c8f37/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a766d2d32303234303332303232313333302e706e67)

对象头的大小是固定的，在 32 位 JVM 上是 8 字节，在 64 位 JVM 上是 16 字节；如果开启了压缩指针，就是 12 字节。

实例数据的大小取决于对象的成员变量和它们的类型。对于`new Object()`来说，由于默认没有成员变量，因此我们可以认为此时的实例数据大小是 0。

假如 MyObject 对象有三个成员变量，分别是 int、long 和 byte 类型，那么它们占用的内存大小分别是 4 字节、8 字节和 1 字节。

```
class MyObject {
    int a;        // 4 字节
    long b;       // 8 字节
    byte c;       // 1 字节
}
```



考虑到对齐填充，MyObject 对象的总大小为 12（对象头） + 4（a） + 8（b） + 1（c） + 7（填充） = 32 字节。





































































































































































































































































































































































































