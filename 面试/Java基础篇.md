# JAVA基础篇



## JAVA概述

### JVM、JDK 和 JRE 有什么区别？

**JVM**：Java Virtual Machine，Java 虚拟机，Java 程序运行在 Java 虚拟机上。针对不同系统的实现（Windows，Linux，macOS）不同的 JVM，因此 Java 语言可以实现跨平台。

**JRE**：Java 运⾏时环境。它是运行已编译 Java 程序所需的所有内容的集合，包括 Java 虚拟机（JVM），Java 类库，Java 命令和其他的⼀些基础构件。但是，它不能⽤于创建新程序。

**JDK**: Java Development Kit，它是功能齐全的 Java SDK。它拥有 JRE 所拥有的⼀切，还有编译器（javac）和⼯具（如 javadoc 和 jdb）。它能够创建和编译程序。

简单来说，**JDK 包含 JRE，JRE 包含 JVM。**

![图片](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152143923.webp)



### 说说什么是跨平台性？原理是什么

所谓跨平台性，是指 Java 语言编写的程序，**一次编译后**，可以在多个系统平台上运行。

实现原理：Java 程序是通过 Java 虚拟机在系统平台上运行的，只要该系统可以安装相应的 Java 虚拟机，该系统就可以运行 java 程序。



### 什么是字节码？采用字节码的好处是什么?

所谓的**字节码，就是 Java 程序经过编译之类产生的.class 文件**，**字节码能够被虚拟机识别，从而实现 Java 程序的跨平台性。**

**Java** 程序从源代码到运行主要有三步：

- **编译**：将我们的代码（.java）编译成虚拟机可以识别理解的字节码(.class)
- **解释**：虚拟机执行 Java 字节码，将字节码翻译成机器能识别的机器码
- **执行**：对应的机器执行二进制机器码

[![Java程序执行过程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152144744.png)](https://camo.githubusercontent.com/010daf89f0dc0f832d5dc930156f9f39d1e9a1f2f48b43c51488848c6c7d3bdf/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a61766173652d342e706e67)

只需要把 Java 程序编译成 Java 虚拟机能识别的 Java 字节码，不同的平台安装对应的 Java 虚拟机，这样就可以可以实现 Java 语言的平台无关性。



### 为什么说 Java 语言“编译与解释并存”？

高级编程语言按照程序的执行方式分为**编译型**和**解释型**两种。

简单来说，编译型语言是指编译器针对特定的操作系统将源代码**一次性**翻译成可被该平台执行的机器码；解释型语言是指解释器对源程序**逐行**解释成特定平台的机器码并立即执行。

Java 程序要经过**先编译，后解释**两个步骤

由 Java 编写的程序需要先经过编译步骤，生成字节码（`\*.class` 文件），这种字节码必须再经过 JVM，解释成操作系统能识别的机器码，在由操作系统执行。因此，我们可以认为 Java 语言**编译**与**解释**并存。

![image-20241215214954570](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152149638.png)



## 基础语法

### Java 有哪些数据类型？

![image-20241215215056905](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152150947.png)

**Java 八大基本数据类型**的默认值和占用大小：

| 数据类型 | 默认值   | 大小            |
| -------- | -------- | --------------- |
| boolean  | false    | 1 字节或 4 字节 |
| char     | '\u0000' | 2 字节          |
| byte     | 0        | 1 字节          |
| short    | 0        | 2 字节          |
| int      | 0        | 4 字节          |
| long     | 0L       | 8 字节          |
| float    | 0.0f     | 4 字节          |
| double   | 0.0      | 8 字节          |



### 包装类型的作用，什么是自动拆箱/装箱？

1. **对象化基本数据类型**
   包装类型允许将基本数据类型当作对象使用。例如，在集合框架（如 `ArrayList`）中，只能存储对象，而不能直接存储基本数据类型：

   ```java
   ArrayList<Integer> list = new ArrayList<>();
   list.add(10);  // 自动装箱
   ```

2. **提供实用方法**
   包装类型类中提供了一些实用方法，如：

   - 将字符串解析为基本数据类型：

     ```java
     int number = Integer.parseInt("123");
     ```

   - 获取常用的常量值：

     ```java
     int max = Integer.MAX_VALUE;
     ```

3. **自动装箱与拆箱（Autoboxing & Unboxing）**
   Java 5 引入了自动装箱（将基本数据类型转换为包装类型）和拆箱（将包装类型转换为基本数据类型），简化了开发：

   ```java
   Integer num = 100;  // 自动装箱
   int value = num;    // 自动拆箱
   ```

[![三分恶面渣逆袭:装箱和拆箱](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152155051.png)](https://camo.githubusercontent.com/a7426f507b191eee5e171cd54082f56d72d4d540b4f9b8074da517a842cd83eb/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a61766173652d382e706e67)

:exclamation:**拆箱空指针异常（NullPointerException）**
自动拆箱时，若包装类型为 `null`，会抛出 `NullPointerException`：

```java
java复制代码Integer num = null;
int value = num;  // 抛出 NullPointerException
```



### &和&&有什么区别？

&是**逻辑与**运算，&&是**短路与**运算

如果&&左边的表达式的值是 false，右边的表达式会被直接短路掉，不会进行运算。

逻辑或运算符（|）和短路或运算符（||）的差别也是如此。



### switch 是否能作用在 byte/long/String 上？

Java5 以前 switch(expr)中，expr 只能是 byte、short、char、int。

从 Java 5 开始，Java 中引入了枚举类型， expr 也可以是 enum 类型。

从 Java 7 开始，expr 还可以是字符串(String)，但是长整型(long)在目前所有的版本中都是不可以的。



### float 是怎么表示小数的？

float类型的小数在计算机中是通过 **IEEE 754** 标准的单精度浮点数格式来表示的。

![image-20241215220615833](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152206870.png)

- S：符号位，0 代表正数，1 代表负数；
- M：尾数部分，用于表示数值的精度；比如说 1.25∗22；1.25 就是尾数；
- R：基数，十进制中的基数是 10，二进制中的基数是 2；
- E：指数部分，例如 10−1 中的 -1 就是指数。



### 讲一下数据准确性高是怎么保证的？

在金融计算中，保证数据准确性有两种方案，一种使用 `BigDecimal`，一种将浮点数转换为整数 int 进行计算。

在处理小额支付或计算时，通过转换为较小的货币单位（如分），这样不仅提高了运算速度，还保证了计算的准确性。





## 面向对象

### 为什么Java里面要多组合少继承？

继承容易导致类之间的强耦合

**组合模式**更加灵活，可以将形状和颜色分开，松耦合。

```java
// 圆形
class Circle extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a circle");
    }
}

// 带红色的圆形
class RedCircle extends Circle {
    @Override
    public void draw() {
        System.out.println("Drawing a red circle");
    }
}
```

```java
// 形状接口
interface Shape {
    void draw();
}

// 颜色接口
interface Color {
    void applyColor();
}
```

形状干形状的事情。



### 多态解决了什么问题？实现原理是什么？

多态指同**一个接口或方法**在不同的类中有不同的实现，比如说动态绑定，父类引用指向子类对象，方法的具体调用会延迟到运行时决定。

多态通过**动态绑定**实现，Java 使用虚方法表存储方法指针，方法调用时根据对象实际类型从虚方法表查找具体实现。

![截图来自博客园的小牛呼噜噜：虚拟方法表](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152217190.png)



### 请说说多态、重载和重写

**重载：**如果一个类有多个**名字相同但参数个数不同**的方法，我们通常称这些方法为方法重载（overload）

**重写：**如果子类具有和父类一样的方法（**参数相同、返回类型相同、方法名相同，但方法体可能不同**），我们称之为方法重写（override）。

![image-20241215222618361](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152226422.png)



### 什么是里氏代换原则？

里氏代换原则也被称为李氏替换原则（Liskov Substitution Principle, LSP），其规定任何父类可以出现的地方，子类也一定可以出现。

意味着子类在扩展父类时，不应改变父类原有的行为。

**SOLID 设计原则**除了LSP还包括：

①、**单一职责原则**（Single Responsibility Principle, SRP），指一个类应该只有一个引起它变化的原因，即一个类只负责一项职责。这样做的目的是使类更加清晰，更容易理解和维护。

②、**开闭原则**（Open-Closed Principle, OCP），指软件实体应该对扩展开放，对修改关闭。这意味着一个类应该通过扩展来实现新的功能，而不是通过修改已有的代码来实现。

③、**接口隔离原则**（Interface Segregation Principle, ISP），指客户端不应该依赖它不需要的接口。这意味着设计接口时应该尽量精简，不应该设计臃肿庞大的接口。

④、**依赖倒置原则**（Dependency Inversion Principle, DIP），指高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。这意味着设计时应该尽量依赖接口或抽象类，而不是实现类。



### 访问修饰符 public、private、protected、以及不写（默认）时的区别？

**private和protected不能修饰类（外部类）**

![image-20241215223213316](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152232369.png)



### 抽象类和接口有什么区别？

**一个类只能继承一个抽象类；但一个类可以实现多个接口**。所以我们在新建线程类的时候一般推荐使用实现 Runnable 接口的方式，这样线程类还可以继承其他类，而不单单是 Thread 类。



抽象类更多地是用来为多个相关的类提供一个共同的**基础框架**，包括状态的初始化，而接口则是定义一套**行为标准**，让不同的类可以实现同一接口，提供行为的多样化实现。



**抽象类可以定义构造方法吗**？

可以，抽象类可以有构造方法。

**接口可以定义构造方法吗**？

不能，接口主要用于定义一组方法规范，没有具体的实现细节。

**接口可以多继承吗？**

接口可以多继承，一个接口可以继承多个接口，使用逗号分隔。



**继承和抽象的区别？**

继承是一种允许子类继承父类属性和方法的机制

抽象是一种隐藏复杂性和只显示必要部分的技术

**抽象类和普通类的区别？**

抽象类使用 abstract 关键字定义，不能被实例化，只能作为其他类的父类。普通类没有 abstract 关键字，可以直接实例化。

抽象类可以包含抽象方法和非抽象方法。**抽象方法没有方法体，必须由子类实现**。普通类智能包含非抽象方法。



### static 关键字了解吗？

| 修饰对象 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| 变量     | 静态变量，类级别变量，所有实例共享同一份数据。               |
| 方法     | 静态方法，类级别方法，与实例无关。                           |
| 代码块   | 在类加载时初始化一些数据，只执行一次。                       |
| 内部类   | 与外部类绑定但独立于外部类实例。                             |
| 导入     | 可以直接访问静态成员，无需通过类名引用，简化代码书写，但会降低代码可读性。 |

#### 静态变量和实例变量的区别？

**静态变量:** 是被 static 修饰符修饰的变量，也称为类变量，它属于类，不属于类的任何一个对象，一个类不管创建多少个对象，静态变量在内存中有且仅有一个副本。

**实例变量:** 必须依存于某一实例，需要先创建对象然后通过对象才能访问到它。静态变量可以实现让多个对象共享内存。

**静态方法和实例方法类似**



### final 关键字有什么作用？

①、当 final 修饰一个**类**时，表明这个类**不能被继承**。比如，String 类、Integer 类和其他包装类都是用 final 修饰的。

②、当 final 修饰一个**方法**时，表明这个方法**不能被重写**（Override）。也就是说，如果一个类继承了某个类，并且想要改变父类中被 final 修饰的方法的行为，是不被允许的。

③、当 final 修饰一个**变量**时，表明这个变量的值一旦被初始化就**不能被修改**。

如果是基本数据类型的变量，其数值一旦在初始化之后就不能更改；如果是引用类型的变量，在对其初始化之后就不能再让其指向另一个对象。

![image-20241215224512604](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152245663.png)



### final、finally、finalize 的区别？

①、**final** 是一个修饰符，可以修饰类、方法和变量。当 final 修饰一个类时，表明这个类不能被继承；当 final 修饰一个方法时，表明这个方法不能被重写；当 final 修饰一个变量时，表明这个变量是个常量，一旦赋值后，就不能再被修改了。

②、**finally** 是 Java 中异常处理的一部分，用来创建 try 块后面的 finally 块。无论 try 块中的代码是否抛出异常，finally 块中的代码总是会被执行。通常，finally 块被用来释放资源，如关闭文件、数据库连接等。

③、**finalize** 是[Object 类](https://javabetter.cn/oo/object-class.html#_05、关于-object-类)的一个方法，用于在垃圾回收器将对象从内存中清除出去之前做一些必要的清理工作。

这个方法在垃圾回收器准备释放对象占用的内存之前被自动调用。我们不能显式地调用 finalize 方法，因为它总是由垃圾回收器在适当的时间自动调用。



### :star:==和 equals 的区别？

①、**==**：用于**比较两个对象的引用**，即它们是否指向同一个对象实例。

如果两个变量引用同一个对象实例，`==` 返回 `true`，否则返回 `false`。

对于基本数据类型（如 `int`, `double`, `char` 等），`==` 比较的是值是否相等。

②、**equals() 方法**：用于比较**两个对象的内容**是否相等。默认情况下，`equals()` 方法的行为与 `==` 相同，即比较对象引用。

```java
String a = new String("沉默王二");
String b = new String("沉默王二");

// 使用 == 比较
System.out.println(a == b); // 输出 false，因为 a 和 b 引用不同的对象

// 使用 equals() 比较
System.out.println(a.equals(b)); // 输出 true，因为 a 和 b 的内容相同
```



### 为什么重写 equals 时必须重写 hashCode ⽅法？

因为基于哈希的集合类（如 HashMap）需要基于这一点来正确存储和查找对象

**HashMap 的存储逻辑**

1. **插入元素 (`put` 方法)**
   - `HashMap` 首先调用对象的 `hashCode` 方法，计算对象的哈希值，确定其存储的桶（bucket）。
   - 如果桶中已有元素，则使用 `equals` 方法比较新插入的键和已有键是否相等，以决定是否覆盖或插入新键值对。
2. **查找元素 (`get` 方法)**
   - 同样，`HashMap` 首先通过 `hashCode` 方法定位到桶，然后再使用 `equals` 方法比较。

如果重写了 `equals()`方法而没有重写 `hashCode()`方法，那么被认为相等的对象可能会有不同的哈希码，从而导致无法在 HashMap 中正确处理这些对象。

**如果两个对象通过 `equals` 方法比较相等，它们的 `hashCode` 值也必须相等。**

Java 集合框架（如 `HashSet` 和 `HashMap`）依赖 `equals` 和 `hashCode` 方法来判断对象是否相等。
如果未重写 `equals`，默认的引用比较可能导致集合中的重复元素未被正确检测。

**示例：**

```java
User u1 = new User(1, "Alice");
User u2 = new User(1, "Alice");

Set<User> userSet = new HashSet<>();
userSet.add(u1);
userSet.add(u2);

System.out.println(userSet.size()); // 如果未重写equals，则输出2；重写后输出1
```



- **为什么两个对象有相同的 hashcode 值，它们也不⼀定相等？**

为了解决哈希冲突的问题，哈希表在处理键时，不仅会比较键对象的哈希码，还会使用 equals 方法来检查键对象是否真正相等。如果两个对象的哈希码相同，但通过 equals 方法比较结果为 false，那么这两个对象就不被视为相等。



### Java 是值传递，还是引用传递？

Java 是值传递，不是引用传递。

![image-20241215230121880](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152301948.png)



### 说说深拷贝和浅拷贝的区别?

- **浅拷贝**会创建一个新对象，但这个新对象的属性（字段）和原对象的属性完全相同。如果属性是基本数据类型，拷贝的是基本数据类型的值；如果属性是引用类型，拷贝的是引用地址，因此**新旧对象共享同一个引用对象**。

浅拷贝的实现方式为：实现 Cloneable 接口并重写 `clone()` 方法。

- **深拷贝**也会创建一个新对象，但会递归地复制所有的引用对象，确保新对象和原对象完全独立。新对象与原对象的任何更改都不会相互影响。

深拷贝的实现方式有：手动复制所有的引用对象，或者使用序列化与反序列化。



### Java 创建对象有哪几种方式？

![image-20241215230427934](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152304013.png)



### new 子类的时候，子类和父类静态代码块，构造方法的执行顺序

1. 首先执行父类的静态代码块（仅在类第一次加载时执行）。
2. 接着执行子类的静态代码块（仅在类第一次加载时执行）。
3. 再执行父类的构造方法。
4. 最后执行子类的构造方法。





## String

### String 是 Java 基本数据类型吗？可以被继承吗？

不是，**`String` 是一个类，属于引用数据类型**。Java 的基本数据类型包括八种：四种整型（`byte`、`short`、`int`、`long`）、两种浮点型（`float`、`double`）、一种字符型（`char`）和一种布尔型（`boolean`）。

不行。**String 类使用 final 修饰**，是所谓的不可变类，**无法被继承**。



#### **String 有哪些常用方法？**

我自己常用的有：

1. `length()` - 返回字符串的长度。
2. `charAt(int index)` - 返回指定位置的字符。
3. `substring(int beginIndex, int endIndex)` - 返回字符串的一个子串，从 `beginIndex` 到 `endIndex-1`。
4. `contains(CharSequence s)` - 检查字符串是否包含指定的字符序列。
5. `equals(Object anotherObject)` - 比较两个字符串的内容是否相等。
6. `indexOf(int ch)` 和 `indexOf(String str)` - 返回指定字符或字符串首次出现的位置。
7. `replace(char oldChar, char newChar)` 和 `replace(CharSequence target, CharSequence replacement)` - 替换字符串中的字符或字符序列。
8. `trim()` - 去除字符串两端的空白字符。
9. `split(String regex)` - 根据给定正则表达式的匹配拆分此字符串。



### String 和 StringBuilder、StringBuffer 的区别？

`String`、`StringBuilder`和`StringBuffer`在 Java 中都是用于处理字符串的

- **String** 是不可变的，平常开发用得最多，

  每次对`String`对象进行修改操作（如拼接、替换等）实际上都会**生成一个新的**`String`对象，而不是修改原有对象。

  适用于字符串内容不会改变的场景，比如说作为 HashMap 的 key。

  

- 当遇到大量字符串连接时，就用 **StringBuilder**，它不会生成很多新的对象

  操作都是直接在**原有字符串对象的底层数组上进行**的，而不是生成新的 String 对象。

  用于单线程环境下需要频繁修改字符串内容的场景，比如在循环中拼接或修改字符串，是 String 的完美替代品。

  在进行频繁的字符串修改操作时，`StringBuilder`能提供**更好的性能**。 Java 中的字符串连`+`操作其实就是通过`StringBuilder`实现的。

  

- **StringBuffer** 和 StringBuilder 类似，但每个方法上都加了 synchronized 关键字，所以是**线程安全**的。

​	现在已经不怎么用了，因为一般不会在多线程场景下去频繁的修改字符串内容。



### String str1 = new String("abc") 和 String str2 = "abc" 的区别

直接使用双引号为字符串变量赋值时，Java 首先会检查字符串常量池中是否已经存在相同内容的字符串。如果存在，Java 就会让新的变量引用池中的那个字符串；如果不存在，它会创建一个新的字符串，放入池中，并让变量引用它。



使用 `new String("abc")` 的方式创建字符串时，实际分为两步：

- 第一步，先检查字符串字面量 "abc" 是否在字符串常量池中，如果没有则创建一个；如果已经存在，则引用它。
- 第二步，在堆中再创建一个新的字符串对象，并将其初始化为字符串常量池中 "abc" 的一个副本。

[![三分恶面渣逆袭：堆与常量池中的String](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412152313114.png)](https://camo.githubusercontent.com/e41927d152743e6a9fb1fa924f7731470b53c4d9312248c08ad12e0895561fcd/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a61766173652d31372e706e67)

**String 变量直接赋值和构造方法赋值==比较相等吗？**

```java
String s1 = "沉默王二";
String s2 = "沉默王二";
String s3 = new String("沉默王二");

System.out.println(s1 == s2); // 输出 true，因为 s1 和 s2 引用的是字符串常量池中同一个对象。
System.out.println(s1 == s3); // 输出 false，因为 s3 是通过 new 关键字显式创建的，指向堆上不同的对象。
```



### String 是不可变类吗？字符串拼接是如何实现的？

1. 不可变性使得 String 对象在使用中更加**安全**。因为字符串经常用作参数传递给其他 Java 方法，例如网络连接、打开文件等。

   如果 String 是可变的，这些方法调用的参数值就可能在不知不觉中被改变，从而导致网络连接被篡改、文件被莫名其妙地修改等问题。

   

2. 不可变的对象因为状态不会改变，所以更容易进行**缓存和重用**。字符串常量池的出现正是基于这个原因。

   当代码中出现相同的字符串字面量时，JVM 会确保所有的引用都指向常量池中的同一个对象，从而节约内存。

   

3. 因为 String 的内容不会改变，所以它的**哈希值也就固定不变**。这使得 String 对象**特别适合作为 HashMap 或 HashSet 等集合的键**，因为计算哈希值只需要进行一次，提高了哈希表操作的效率。



#### **字符串拼接是如何实现的**

因为 String 是不可变的，因此通过“**+**”操作符进行的字符串拼接，会生成新的字符串对象。

Java 8 时，JDK 对“+”号的字符串拼接进行了优化，Java 会在编译期**基于 StringBuilder** 的 append 方法进行拼接。

通过加号拼接字符串时会创建多个 String 对象是不准确的。因为加号拼接在编译期还会创建一个 StringBuilder 对象，最终调用 `toString()` 方法的时候再返回一个新的 String 对象。



除了使用 `+` 号来拼接字符串，还有 `StringBuilder.append()`、`String.join()` 等方式拼接字符串。



#### **如何保证 String 不可变？**

第一，String 类内部使用一个**私有的字符数组**来存储字符串数据。这个字符数组在创建字符串时被初始化，之后不允许被改变。

```java
private final char value[];
```



第二，String 类**没有提供任何可以修改其内容的公共方法**，像 concat 这些看似修改字符串的操作，实际上都是返回一个新创建的字符串对象，而原始字符串对象保持不变。



第三，String **类本身被声明为 final**，这意味着它不能被继承。这防止了子类可能通过添加修改方法来改变字符串内容的可能性。

```java
public final class String
```



### intern 方法有什么作用？

- 如果当前字符串内容存在于字符串常量池（即 equals()方法为 true，也就是内容一样），直接返回字符串常量池中的字符串
- 否则，将此 String 对象添加到池中，并返回 String 对象的引用



## Integer

### Integer a= 127，Integer b = 127；Integer c= 128，Integer d = 128；相等吗?  new Integer(10) == new Integer(10) 相等吗

a 和 b 相等，c 和 d 不相等。

```java
Integer a = 127;
Integer b = 127;
```

`a`和`b`是相等的。这是因为 Java 在自动装箱过程中，会使用`Integer.valueOf()`方法来创建`Integer`对象。

`Integer.valueOf()`方法会针**对数值在-128 到 127 之间的`Integer`对象使用缓存**。因此，`a`和`b`实际上引用了常量池中相同的`Integer`对象。

```java
Integer c = 128;
Integer d = 128;
```

`c`和`d`不相等。这是因为 **128 超出了`Integer`缓存的范围**(-128 到 127)。

因此，自动装箱过程会为`c`和`d`创建两个不同的`Integer`对象，它们有不同的引用地址。



#### **什么是 Integer 缓存？**

根据实践发现，大部分的数据操作都集中在值比较小的范围，因此 Integer 搞了个缓存池，默认范围是 -128 到 127。

当我们使用自动装箱来创建这个范围内的 Integer 对象时，Java 会直接从缓存中返回一个已存在的对象，而不是每次都创建一个新的对象。这意味着，对于这个值范围内的所有 Integer 对象，它们实际上是**引用相同的对象实例**。

Integer 缓存的主要目的是优化性能和内存使用。

可以在运行的时候添加 `-Djava.lang.Integer.IntegerCache.high=1000` 来调整缓存池的最大值。



#### new Integer(10) == new Integer(10) 相等吗

在 Java 中，使用`new Integer(10) == new Integer(10)`进行比较时，结果是 false。

这是因为 **new 关键字会在堆（Heap）上为每个 Integer 对象分配新的内存空间，所以这里创建了两个不同的 Integer 对象，它们有不同的内存地址。**

当使用==运算符比较这两个对象时，实际上比较的是它们的内存地址，而不是它们的值，因此即使两个对象代表相同的数值（10），结果也是 false。



### String 怎么转成 Integer 的？原理？

String 转成 Integer，主要有两个方法：

- Integer.parseInt(String s)
- Integer.valueOf(String s)

不管哪一种，最终还是会调用 Integer 类内中的**parseInt(String s, int radix)**方法。

核心代码：

```java

public static int parseInt(String s, int radix)
                throws NumberFormatException
    {

        int result = 0;
        //是否是负数
        boolean negative = false;
        //char字符数组下标和长度
        int i = 0, len = s.length();
        ……
        int digit;
        //判断字符长度是否大于0，否则抛出异常
        if (len > 0) {
            ……
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                //返回指定基数中字符表示的数值。（此处是十进制数值）
                digit = Character.digit(s.charAt(i++),radix);
                //进制位乘以数值
                result *= radix;
                result -= digit;
            }
        }
        //根据上面得到的是否负数，返回相应的值
        return negative ? result : -result;
    }
```



去掉枝枝蔓蔓（当然这些枝枝蔓蔓可以去看看，源码 cover 了很多情况），其实剩下的就是一个简单的字符串遍历计算，不过计算方式有点反常规，是用负的值累减。

[![image-20241216001725537](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161015440.png)](https://camo.githubusercontent.com/fb5e1ad603f4e205c69868592ddca4323364fdb46e0eadc3ad018aee08f1ac24/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a61766173652d32302e706e67)



## Object

### Object 类的常见方法？

![image-20241216001827109](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161015441.png)

#### 对象比较：

①、public native int **hashCode()** ：[native 方法](https://javabetter.cn/oo/native-method.html)，用于返回对象的哈希码。

```java
public native int hashCode();
```

按照约定，相等的对象必须具有相等的哈希码。如果重写了 equals 方法，就应该重写 hashCode 方法。可以使用 Objects**.hash()** 方法来生成哈希码。

```java
public int hashCode() {
    return Objects.hash(name, age);
}
```



②、public boolean **equals**(Object obj)：用于比较 2 个对象的内存地址是否相等。

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

如果比较的是两个对象的值是否相等，就要重写该方法，比如 [String 类](https://javabetter.cn/string/string-source.html)、Integer 类等都重写了该方法。

```java
public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj instanceof Person1) {
            Person1 p = (Person1) obj;
            return this.name.equals(p.getName()) && this.age == p.getAge();
        }
        return false;
    }
```



#### 对象拷贝：

protected native Object **clone()** throws CloneNotSupportedException：

naitive 方法，返回此对象的一个副本。默认实现只做**浅拷贝**，且类**必须实现 Cloneable 接口。**



#### 对象转字符串：

public String **toString()**：返回对象的字符串表示。默认实现返回类名@哈希码的十六进制表示，但通常会被重写以返回更有意义的信息。

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

IntelliJ IDEA，直接右键选择 Generate，然后选择 toString 方法，就会自动生成一个 toString 方法。

也可以交给 [Lombok](https://javabetter.cn/springboot/lombok.html)，使用 @Data 注解，它会自动生成 toString 方法。



#### 多线程调度：

每个对象都可以调用 Object 的 **wait/notify 方法**来实现等待/通知机制。

①、`public final void wait() throws InterruptedException`：调用该方法会导致当前线程等待，直到另一个线程调用此对象的`notify()`方法或`notifyAll()`方法。

②、`public final native void notify()`：唤醒在此对象监视器上等待的单个线程。如果有多个线程等待，选择一个线程被唤醒。

③、`public final native void notifyAll()`：唤醒在此对象监视器上等待的所有线程。

④、`public final native void wait(long timeout) throws InterruptedException`：等待 timeout 毫秒，如果在 timeout 毫秒内没有被唤醒，会自动唤醒。

⑥、`public final void wait(long timeout, int nanos) throws InterruptedException`：更加精确了，等待 timeout 毫秒和 nanos 纳秒，如果在 timeout 毫秒和 nanos 纳秒内没有被唤醒，会自动唤醒。



#### 反射：

public final native Class<?> **getClass()**：用于获取对象的类信息，如类名。



#### 垃圾回收：

protected void **finalize()** throws Throwable 

垃圾回收器决定回收对象占用的内存时调用此方法。用于清理资源，但 Java 不推荐使用，因为它不可预测且容易导致问题，Java 9 开始已被弃用。





## 异常处理

### Java 中异常处理体系?

通常通过 try-catch-finally 语句和 throw 关键字来实现。

[![三分恶面渣逆袭：Java异常体系](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161015398.png)](https://camo.githubusercontent.com/6fc25c2fd330781b439046278f7c0d3b8ed06bc8d373a452fdd50a955b334f8c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a61766173652d32322e706e67)

Error 类代表那些严重的错误，这类错误通常是程序无法处理的。比如，OutOfMemoryError 表示内存不足，StackOverflowError 表示栈溢出。这些错误**通常与 JVM 的运行状态有关**，一旦发生，应用程序通常无法恢复。



Exception 类代表程序可以处理的异常。它分为两大类：编译时异常（Checked Exception）和运行时异常（Runtime Exception）。

①、编译时异常（Checked Exception）：这类异常在编译时**必须被显式处理**（捕获或声明抛出）。

②、运行时异常（Runtime Exception）：这类异常在运行时抛出，它们都是 RuntimeException 的子类。对于运行时异常，Java 编译器不要求必须处理它们（即不需要捕获也不需要声明抛出) 。通常是由程序逻辑错误导致的



### 异常的处理方式？

![三分恶面渣逆袭：异常处理](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161020860.png)

①、遇到异常时可以不处理，直接通过throw 和 throws 抛出异常，交给上层调用者处理。

throws 关键字用于声明可能会抛出的异常，而 throw 关键字用于抛出异常。

```JAVA
public void test() throws Exception {
    throw new Exception("抛出异常");
}
```



②、使用 try-catch 捕获异常，处理异常。

```JAVA
try {
    //包含可能会出现异常的代码以及声明异常的方法
}catch(Exception e) {
    //捕获异常并进行处理
}finally {
    //可选，必执行的代码
}
```



#### catch和finally的异常可以同时抛出吗？

如果 catch 块抛出一个异常，而 finally 块中也抛出异常，那么**最终抛出的将是 finally 块中的异常。catch 块中的异常会被丢弃，**而 finally 块中的异常会覆盖并向上传递。

虽然 catch 和 finally 中的异常不能同时抛出，但可以手动捕获 finally 块中的异常，并将 catch 块中的异常保留下来，避免被覆盖。常见的做法是使用一个变量临时存储 catch 中的异常，然后在 finally 中处理该异常：

```java
public class Example {
    public static void main(String[] args) {
        Exception catchException = null;
        try {
            throw new Exception("Exception in try");
        } catch (Exception e) {
            catchException = e;
            throw new RuntimeException("Exception in catch");
        } finally {
            try {
                throw new IllegalArgumentException("Exception in finally");
            } catch (IllegalArgumentException e) {
                if (catchException != null) {
                    System.out.println("Catch exception: " + catchException.getMessage());
                }
                System.out.println("Finally exception: " + e.getMessage());
            }
        }
    }
}
```



### :star:三道经典异常处理代码题

#### 题目 1

```JAVA
public class TryDemo {
    public static void main(String[] args) {
        System.out.println(test());
    }
    public static int test() {
        try {
            return 1;
        } catch (Exception e) {
            return 2;
        } finally {
            System.out.print("3");
        }
    }
}
```

①、`try`块中包含一条`return 1;`语句。正常情况下，如果`try`块中的代码能够顺利执行，那么方法将返回数字`1`。在这个例子中，`try`块中没有任何可能抛出异常的操作，因此它会正常执行完毕，并准备返回`1`。

②、由于`try`块中没有异常发生，所以`catch`块中的代码不会执行。

③、无论前面的代码是否发生异常，`finally`块总是会执行。在这个例子中，`finally`块包含一条`System.out.print("3");`语句，意味着在**方法结束前，会在控制台打印**出`3`。

当执行`main`方法时，控制台的输出将会是：

```
31
```

这是因为`finally`块确保了它包含的`System.out.print("3");`会执行并打印`3`，随后`test()`方法返回`try`块中的值`1`，最终结果就是`31`。



#### 题目 2

```java
public class TryDemo {
    public static void main(String[] args) {
        System.out.println(test1());
    }
    public static int test1() {
        try {
            return 2;
        } finally {
            return 3;
        }
    }
}
```

try 返回前先执行 finally，结果 finally 里不按套路出牌，直接 return 了，自然也就走不到 try 里面的 return 了。



#### 题目 3

```java
public class TryDemo {
    public static void main(String[] args) {
        System.out.println(test1());
    }
    public static int test1() {
        int i = 0;
        try {
            i = 2;
            return i;
        } finally {
            i = 3;
        }
    }
}
```

执行结果：2。

大家可能会以为结果应该是 3，因为在 return 前会执行 finally，而 i 在 finally 中被修改为 3 了，那最终返回 i 不是应该为 3 吗？

但其实，在执行 finally 之前，**JVM 会先将 i 的结果暂存起来**，然后 finally 执行完毕后，会返回之前暂存的结果，而不是返回 i，所以即使 i 已经被修改为 3，最终返回的还是之前暂存起来的结果 2。





## I/O

### Java 中 IO 流分为几种?

**按照数据流方向如何划分**？

- 输入流（Input Stream）：从源（如文件、网络等）读取数据到程序。
- 输出流（Output Stream）：将数据从程序写出到目的地（如文件、网络、控制台等）。



**按处理数据单位如何划分？**

[![二哥的 Java 进阶之路](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161029927.png)](https://camo.githubusercontent.com/c99a56be08bf5e9898ebf42e0e3c9c16527090fa70e3ac99675159ac8c711a32/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f696f2f7368616e67746f752d30312e706e67)

#### 

**按功能如何划分？**

- 节点流（Node Streams）：直接**与数据源或目的地相连**，如 FileInputStream、FileOutputStream。
- 处理流（Processing Streams）：对一个**已存在的流进行包装**，如缓冲流 BufferedInputStream、BufferedOutputStream。
- 管道流（Piped Streams）：用于**线程之间的**数据传输，如 PipedInputStream、PipedOutputStream。



#### IO 流用到了什么设计模式？

其实，Java 的 IO 流体系还用到了一个设计模式——**装饰器模式**。

![image-20241216103305538](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161033649.png)



#### Java 缓冲区溢出，如何预防

①、**合理设置缓冲区大小**：在创建缓冲区时，应根据实际需求合理设置缓冲区的大小，避免创建过大或过小的缓冲区。

②、**控制写入数据量**：在向缓冲区写入数据时，应该控制写入的数据量，确保不会超过缓冲区的容量。Java 的 **ByteBuffer 类**提供了**remaining()方法**，可以获取缓冲区中剩余的可写入数据量。





### 既然有了字节流,为什么还要有字符流?

字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还比较耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。

所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。



#### 文本存储是字节流还是字符流，视频文件呢？

无论是文本文件还是视频文件，它们在物理存储层面都是以字节流的形式存在。

文本和视频都是按照**字节**存储的，只是如果是文本文件的话，我们可以通过**字符**流的形式去读取，这样更方面的我们进行直接处理。



### :star:BIO、NIO、AIO 之间的区别？

![image-20241216104221909](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161042019.png)

**BIO**（Blocking I/O）：采用阻塞式 I/O 模型，线程在执行 I/O 操作时被阻塞，无法处理其他任务，适用于连接数较少的场景。



**NIO**（New I/O 或 Non-blocking I/O）：采用非阻塞 I/O 模型，线程在等待 I/O 时可执行其他任务，**通过 Selector 监控多个 Channel 上的事件，**适用于连接数多但连接时间短的场景。**Channel 来实现多路复用**

![image-20241216104239204](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161042313.png)



**AIO**（Asynchronous I/O）：使用异步 I/O 模型，线程**发起 I/O 请求后立即返回**，当 I/O 操作完成时通过**回调函数**通知线程，适用于连接数多且连接时间长的场景。





## 序列化

序列化（Serialization）是指将对象转换为字节流的过程，以便能够将该对象保存到文件、数据库，或者进行网络传输。

反序列化（Deserialization）就是将字节流转换回对象的过程，以便构建原始对象。

![image-20241216104327499](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161043596.png)



#### Serializable 接口有什么用？

`Serializable`接口用于标记一个类可以被序列化。

```java
public class Person implements Serializable {
    private String name;
    private int age;
    // 省略 getter 和 setter 方法
}
```

#### serialVersionUID 有什么用？

serialVersionUID 是 Java 序列化机制中用于**标识类版本的唯一标识符**。它的作用是确保在序列化和反序列化过程中，类的版本是兼容的。

```java
import java.io.Serializable;

public class MyClass implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;

    // getters and setters
}
```

serialVersionUID 被设置为 1L 是一种比较省事的做法，也可以使用 Intellij IDEA 进行自动生成。

**如果不显式声明 serialVersionUID，Java 运行时会根据类的详细信息自动生成一个 serialVersionUID。那么当类的结构发生变化时，自动生成的 serialVersionUID 就会发生变化，导致反序列化失败。**



#### Java 序列化不包含静态变量吗？

是的，序列化机制只会保存**对象的状态**，而静态变量属于**类的状态**，不属于对象的状态。



#### 如果有些变量不想序列化，怎么办？

可以使用**transient**关键字修饰不想序列化的变量。

```java
public class Person implements Serializable {
    private String name;
    private transient int age;
    // 省略 getter 和 setter 方法
}
```



#### 能解释一下序列化的过程和作用吗？

序列化过程通常涉及到以下几个步骤：

第一步，**实现 Serializable 接口。**

```java
public class Person implements Serializable {
    private String name;
    private int age;

    // 省略构造方法、getters和setters
}
```

第二步，**使用 ObjectOutputStream 来将对象写入到输出流中。**

```java
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("person.ser"));
```

第三步，调用 **ObjectOutputStream 的 writeObject 方法，将对象序列化并写入到输出流中。**

```java
Person person = new Person("沉默王二", 18);
out.writeObject(person);
```



### 说说有几种序列化方式？

![image-20241216105052884](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161050991.png)

- **Java 对象序列化** ：Java 原生序列化方法即通过 Java 原生流(InputStream 和 OutputStream 之间的转化)的方式进行转化，一般是对象输出流 `ObjectOutputStream`和对象输入流`ObjectInputStream`。
- **Json 序列化**：这个可能是我们最常用的序列化方式，Json 序列化的选择很多，一般会使用 jackson 包，通过 ObjectMapper 类来进行一些操作，比如将对象转化为 byte 数组或者将 json 串转化为对象。
- **ProtoBuff 序列化**：ProtocolBuffer 是一种轻便高效的结构化数据存储格式，ProtoBuff 序列化对象可以很大程度上将其压缩，可以大大减少数据传输大小，提高系统性能。





## 网络编程

### 了解过Socket网络套接字吗？

一个简单的 TCP 客户端：

```java
class TcpClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8080); // 连接服务器
        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

        out.println("Hello, Server!"); // 发送消息
        System.out.println("Server response: " + in.readLine()); // 接收服务器响应

        socket.close();
    }
}
```

TCP 服务端：

```java
class TcpServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(8080); // 创建服务器端Socket
        System.out.println("Server started, waiting for connection...");
        Socket socket = serverSocket.accept(); // 等待客户端连接
        System.out.println("Client connected: " + socket.getInetAddress());

        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

        String message;
        while ((message = in.readLine()) != null) {
            System.out.println("Received: " + message);
            out.println("Echo: " + message); // 回送消息
        }

        socket.close();
        serverSocket.close();
    }
}
```



#### RPC框架了解吗？

RPC是一种协议，允许程序调用位于远程服务器上的方法，就像调用本地方法一样。RPC 通常基于 Socket 通信实现。

RPC 框架支持高效的序列化（如 Protocol Buffers）和通信协议（如 HTTP/2）

常见的 RPC 框架包括：**只有openfeign不能跨语言**

1. gRPC：基于 **HTTP/2** 和 Protocol Buffers。
2. Dubbo：阿里开源的分布式 RPC 框架，适合微服务场景。
3. Spring Cloud OpenFeign：基于 REST 的轻量级 RPC 框架。
4. Thrift：Apache 的跨语言 RPC 框架，支持多语言代码生成。





## 泛型

### Java 泛型了解么？什么是类型擦除？介绍一下常用的通配符？

#### 什么是泛型？

泛型主要用于**提高代码的类型安全**，它允许在定义类、接口和方法时使用类型参数，这样可以在编译时检查类型一致性，避免不必要的类型转换和类型错误。

没有泛型的时候，像 List 这样的集合类存储的是 Object 类型，导致从集合中读取数据时，必须进行强制类型转换，否则会引发 ClassCastException。

```java
List list = new ArrayList();
list.add("hello");
String str = (String) list.get(0);  // 必须强制类型转换
```



泛型一般有三种使用方式:**泛型类**、**泛型接口**、**泛型方法**。

[![泛型类、泛型接口、泛型方法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161116638.png)](https://camo.githubusercontent.com/6badb390493e8429f28dac7fe6e622a46a27126feec18f301af8ed3b0cd48d56/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a61766173652d33322e706e67)

**1.泛型类**：

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

如何实例化泛型类：

```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```



**2.泛型接口** ：

```java
public interface Generator<T> {
    public T method();
}
```

实现泛型接口，指定类型：

```java
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```



**3.泛型方法** ：

```java
   public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```



使用：

```java
// 创建不同类型数组： Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```



#### 泛型常用的通配符有哪些？

**常用的通配符为： T，E，K，V，？**

- ？ 表示不确定的 java 类型
- T (type) 表示具体的一个 java 类型
- K V (key value) 分别代表 java 键值中的 Key Value
- E (element) 代表 Element



#### 什么是泛型擦除？

所谓的泛型擦除，官方名叫“**类型擦除**”。

Java 的泛型是伪泛型，这是因为 Java 在编译期间，所有的类型信息都会被擦掉。

也就是说，**在运行的时候是没有泛型的**。

例如这段代码，往一群猫里放条狗：

```java
LinkedList<Cat> cats = new LinkedList<Cat>();
LinkedList list = cats;  // 注意我在这里把范型去掉了，但是list和cats是同一个链表！
list.add(new Dog());  // 完全没问题！
```



因为 Java 的范型只存在于源码里，**编译的时候给你静态地检查一下范型类型是否正确，而到了运行时就不检查了。**上面这段代码在 JRE（Java**运行**环境）看来和下面这段没区别：

```java
LinkedList cats = new LinkedList();  // 注意：没有范型！
LinkedList list = cats;
list.add(new Dog());
```



#### 为什么要类型擦除呢？

主要是为了**向下兼容**，因为 JDK5 之前是没有泛型的，为了让 JVM 保持向下兼容，就出了类型擦除这个策略。





## 注解

### 说一下你对注解的理解？

**Java 注解本质上是一个标记**，有了标记之后，我们就可以在编译或者运行阶段去识别这些标记，然后搞一些事情，这就是注解的用处。

注解生命周期有三大类，分别是：

- RetentionPolicy.**SOURCE**：给编译器用的，不会写入 class 文件
- RetentionPolicy.**CLASS**：会写入 class 文件，在类加载阶段丢弃，也就是运行的时候就没这个信息了

- RetentionPolicy.**RUNTIME**：会写入 class 文件，永久保存，可以通过反射获取注解信息

比如 Spring 常见的 **Autowired** ，就是 RUNTIME 的，所以**在运行的时候可以通过反射得到注解的信息**，还能拿到标记的值 required 。





## :star:反射

### 什么是反射？应用？原理？

反射允许 Java 在运行时检查和操作类的方法和字段。通过反射，可以**动态地获取类的字段、方法、构造方法等信息，并在运行时调用方法或访问字段**。

反射功能主要通过 `java.lang.Class` 类及 `java.lang.reflect` 包中的类如 Method, Field, Constructor 等来实现。

[![三分恶面渣逆袭：Java反射相关类](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161138724.png)](https://camo.githubusercontent.com/4345a7e8d709e32da317837b54de35a3231ca5df65d702525e1c92893037ef1e/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a61766173652d33362e706e67)

**主要方法**：

- `Class.forName()`：加载类。
- `getDeclaredMethods()`：获取类的所有方法。
- `getDeclaredFields()`：获取类的所有字段。
- `invoke()`：调用方法。



#### 反射有哪些应用场景？

①、Spring 框架就大量使用了反射来**动态加载和管理 Bean**。

```java
Class<?> clazz = Class.forName("com.example.MyClass");
Object instance = clazz.newInstance();
```



②、Java 的**动态代理**（Dynamic Proxy）机制就使用了反射来创建代理类。代理类可以在运行时动态处理方法调用，这在实现 AOP 和拦截器时非常有用。

```java
InvocationHandler handler = new MyInvocationHandler();
MyInterface proxyInstance = (MyInterface) Proxy.newProxyInstance(
    MyInterface.class.getClassLoader(),
    new Class<?>[] { MyInterface.class },
    handler
);
```



③、JUnit 和 TestNG 等测试框架使用反射机制来发现和执行测试方法。反射允许框架扫描类，查找带有特定注解（如 `@Test`）的方法，并在运行时调用它们。

```java
Method testMethod = testClass.getMethod("testSomething");
testMethod.invoke(testInstance);
```



#### 反射的原理是什么？

Java 程序的执行分为编译和运行两步，**编译之后会生成字节码(.class)文件，JVM 进行类加载的时候，会加载字节码文件，将类型相关的所有信息加载进方法区，反射就是去获取这些信息，然后进行各种操作。**



#### 反射、动态代理和 AOP 的区别和使用场景

| 特性         | 反射                                                     | 动态代理                                                     | AOP（面向切面编程）                                    |
| ------------ | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| **定义**     | 运行时获取类的信息（如字段、方法）并动态操作对象。       | 在运行时动态生成代理类拦截方法调用。                         | 一种编程思想，通过切面分离横切关注点（如日志、事务）。 |
| **常见场景** | - 序列化/反序列化 - 注解处理 - 动态加载类 - 动态调用方法 | - RPC 框架（如 Dubbo 动态代理接口） - 数据库框架（如 MyBatis 动态代理接口） - 动态实现接口 | - 事务管理 - 日志记录 - 安全认证 - 性能监控            |
| **侧重点**   | 用于操作类的结构和运行时行为。                           | 用于拦截并增强方法调用，专注于代理逻辑实现。                 | 用于分离横切关注点，聚焦业务逻辑的解耦与复用。         |
| **使用方式** | 直接操作类或方法的元信息。                               | 使用 `Proxy` 或 `CGLIB` 实现代理。                           | 使用切面配置增强逻辑（如 Spring AOP）。                |





## :star:JDK1.8 新特性

![image-20241216120323339](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161203468.png)

①、Java 8 允许在接口中添加默认方法和静态方法。

```java
public interface MyInterface {
    default void myDefaultMethod() {
        System.out.println("My default method");
    }

    static void myStaticMethod() {
        System.out.println("My static method");
    }
}
```

④、Java 8 引入了一个全新的日期和时间 API，位于`java.time`包中。这个新的 API 纠正了旧版`java.util.Date`类中的许多缺陷。

⑤、引入 **Optional** 是为了减少空指针异常。

```java
Optional<String> optional = Optional.of("沉默王二");
optional.isPresent();           // true
optional.get();                 // "沉默王二"
optional.orElse("沉默王三");    // "bam"
optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "沉"
```



### Lambda 表达式了解多少？

所谓的函数式编程，就是把函数作为参数传递给方法，或者作为方法的结果返回。比如说我们可以配合 Stream 流进行数据过滤：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

其中 `n -> n % 2 == 0` 就是一个 Lambda 表达式。表示传入一个参数 n，返回 `n % 2 == 0` 的结果。



#### Java8 有哪些内置函数式接口？

JDK 1.8 API 包含了很多内置的函数式接口。其中就包括我们在老版本中经常见到的 **Comparator** 和 **Runnable**，Java 8 为他们都添加了 @FunctionalInterface 注解，以用来支持 Lambda 表达式。

除了这两个之外，还有 Callable、Predicate、Function、Supplier、Consumer 等等。



### Optional 了解吗？

`Optional`是**用于防范`NullPointerException`**。

可以将 `Optional` 看做是包装对象（可能是 `null`, 也有可能非 `null`）的容器。当我们定义了 一个方法，这个方法返回的对象可能是空，也有可能非空的时候，我们就可以考虑用 `Optional` 来包装它，这也是在 Java 8 被推荐使用的做法。

```java
Optional<String> optional = Optional.of("bam");

optional.isPresent();           // true
optional.get();                 // "bam"
optional.orElse("fallback");    // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```



### Stream 流用过吗？

`Stream` 流，简单来说，使用 `java.util.Stream` 对一个包含一个或多个元素的集合做各种操作。这些操作可能是 *中间操作* 亦或是 *终端操作*。 终端操作会返回一个结果，而中间操作会返回一个 `Stream` 流。

![Java Stream流](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412161211781.png)











