# MySQL高级

[TOC]

## 1、存储引擎

### 1.1、MySQL体系结构

**连接层**
最上层是一些客户端和链接服务，主要完成一些类似于连接处理、授权认证、及相关的安全方案。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

**服务层**
第二层架构主要完成大多数的核心服务，如 SQL 接口，并完成缓存的查询，SQL 的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。

**引擎层**
存储引擎真正的负责了 MySQL 中数据的存储和提取，服务器通过 API 和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选去合适的存储引擎。

**存储层**
主要是将数据存储在文件系统之上，并完成与存储引擎的交互。

![image-20241111224747054](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411112247159.png)

### 1.2、存储引擎简介

在创建表时，指定存储引擎：

```mysql
CREATE TABLE 表名(
	字段1 字段1类型 [COMMENT 字段1注释]
    ...
    字段n 字段n类型 [COMMENT 字段n注释]
)ENGINE = INNODB [COMMENT 表注释];
```

查看当前数据库支持的存储引擎：

```mysql
SHOW ENGINES;
```

![image-20241111224936200](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411112249232.png)

案例：

```mysql
-- 1.查询建表语句  -- 默认存储引擎:InnoDB
show create table account;

-- 2.创建表 my_myisam ，并指定 MyISAM 存储引擎
create table my_myisam(
	id int,
    name varchar(10)
)engine = MyISAM;
```

### 1.3、存储引擎特点

#### 1.3.1、InnoDB

- **介绍**
  - InnoDB 是一种兼顾高可靠性和高性能的通用存储引擎，在 MySQL 5.5 之后，InnoDB 是默认的 MySQL 存储引擎。

- **特点**

  - DML 操作遵循 ACID 模型，支持事务。


  - 行级锁，提高并发访问性能。


  - 支持外键 FOREIGN KEY 约束，保证数据的完整性和正确性。


- - **文件**

  - xxx.ibd ：xxx 代表的是表名，InnoDB 引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm 、sdi）、数据和索引。


  - 参数：innodb_file_per_table


![image-20241111225207065](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411112252120.png)

#### 1.3.2、MyISAM

- **介绍**
  - MyISAM 是 MySQL 早期的默认存储引擎。

- **特点**

  - 不支持事务，不支持外键。

  - 支持表锁，不支持行锁。

  - 访问速度快。

- **文件**

  - .sdi 存储表结构信息

  - .MYD 存储数据

  - .MYI 存储索引

#### 1.3.3、Memory

- **介绍**

  - Memory 引擎的表数据是存储在内存中的，由于收到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用。

- **特点**

  - 内存存放。
  - hash 索引（默认）

- **文件**

  - xxx.sdi ：存储表结构信息

  

  :star:**区别（面试题）**

  | 特点     | InnoDB | MyISAM |
  | -------- | ------ | ------ |
  | 事务安全 | 支持   | \-     |
  | 锁机制   | 行锁   | 表锁   |
  | 支持外键 | 支持   | \-     |

  

### 1.4、存储引擎选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

- InnoDB ：是 MySQL 的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新。删除操作，那么 InnoDB 存储引擎是比较合适的选择。
- MyISAM ：如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事物的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的。（MongoDB）
- MEMORY ：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。MEMORY 的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。（Redis）



## 2、索引

### 2.1、索引概述

**介绍**

索引是帮助 `MySQL` **高效获取数据**的**数据结构（有序）**。

**演示**

**注意：** 下图中的二叉树索引结构只是一个示意图，并不是真实的索引结构。

![image-20241112141917564](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121420277.png)

**优缺点**

|                             优势                             |                             劣势                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|            提高数据检索的效率，降低数据的 IO 成本            |                    索引列也是要占用空间的                    |
| 通过索引列对数据进行排序，降低数据排序的成本，降低 CPU 的消耗 | 索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行 INSERT 、UPADTE 、DELETE 时，效率降低 |



#### 2.1.1、 索引结构

##### 2.1.1.1、分类

`MySQL` 的索引是在存储引擎层实现的，不同的存储引擎有不同的结构，主要包含以下几种：

|       索引结构        |                             描述                             |
| :-------------------: | :----------------------------------------------------------: |
|    **B+Tree 索引**    |       **最常见的索引类型，大部分引擎都支持 B+ 树索引**       |
|       Hash 索引       | 底层数据结构是用哈希表实现的，只有精确匹配索引列的查询才有效，不支持范围查询 |
|  R-tree（空间索引）   | 空间索引是 MyISAM 引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 |
| Full-text（全文索引） | 是一种通过建立倒排索引，快速匹配文档的方式。类似于 Lucene,Solr,ES 。 |



| 索引        | InnoDB          | MyISAM | Memory |
| ----------- | --------------- | ------ | ------ |
| B+Tree 索引 | 支持            | 支持   | 支持   |
| Hash 索引   | 不支持          | 不支持 | 支持   |
| R-tree      | 不支持          | 支持   | 不支持 |
| Full-text   | 5.6版本之后支持 | 支持   | 不支持 |

> **注意：** 我们平常所说的索引，如果没有特别指明，都是指 B+ 树结构组织的索引。



##### 2.1.1.2、B-Tree（多录平衡查找树）

以一颗最大度数（`max-degree`）为 `5`（`5` 阶）的 `b-tree` 为例（每个结点最多存储 `4` 个 `key` ，`5` 个指针）：

![image-20241112141649921](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121416021.png)

> **提示：** 树的度数指的是一个结点的子节点个数。



##### 2.1.1.3、B+Tree

以一颗最大度数（`max-degree`）为 `4`（`4` 阶）的 `b+tree` 为例

![](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121420278.png)

> 相对于 B-Tree 区别：
>
> ① 所有的数据都会出现在叶子结点。
>
> ② 叶子结点形成一个单向链表。

`MySQL` 索引数据结构对经典的 `B+Tree` 进行了优化。在原 `B+Tree` 的基础上，增加了一个指向相邻叶子结点的链表指针，就形成了带有顺序指针的 `B+Tree` ，提高区间访问的性能。

![image-20241112141632322](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121420279.png)

##### 2.1.1.4、Hash

哈希索引就是采用一定的 `hash` 算法，将键值换算成新的 `hash` 值，映射到对应的槽位上，然后存储在 `hash` 表中。

如果两个（或多个）键值，映射到一个相同的槽位上，他们就产生了 `hash` 冲突（也称为 `hash` 碰撞），可以通过链表来解决。

![image-20241112141602732](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121416851.png)

**Hash 索引特点**

- Hash 索引只能用于对等比较（= ，in），不支持范围查询（between ，> ，< ，…）。
- 无法利用索引完成排序操作。
- 查询效率高，通常只需要一次索引就可以了，效率通常要高于 B+tree 索引。

**存储引擎支持**

在 `MySQL` 中，支持 `hash` 索引的是 `Memory` 引擎，而 `InnoDB` 中具有自适应 `hash` 功能，`hash` 索引是存储引擎根据 `B+Tree` 索引在指定条件下自动构建的。

**思考（面试题）**

为什么 `InnoDB` 存储引擎选择使用 `B+tree` 索引结构？

- 相对于二叉树，层级更少，搜索效率高。
- 相对 `B-tree` ，无论是叶子结点还是非叶子结点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量的数据，只能增加树的高度，导致性能降低。
- 相对 `Hash` 索引，`B+tree` 支持范围匹配及排序操作。



#### 2.1.2、索引分类

##### 2.1.2.1、分类

|   分类   |                         含义                         |           特点           |  关键字  |
| :------: | :--------------------------------------------------: | :----------------------: | :------: |
| 主键索引 |               针对于表中主键创建的索引               | 默认自动创建，只能有一个 | PRIMARY  |
| 唯一索引 |           避免同一个表中某数据列中的值重复           |        可以有多个        |  UNIQUE  |
| 常规索引 |                   快速定位特定数据                   |        可以有多个        |          |
| 全文索引 | 全文索引查找的是文本中的关键词，而不是比较索引中的值 |        可以有多个        | FULLTEXT |

在 `InnoDB` 存储引擎中，根据引擎的存储形式，又可以分为以下两种：

| 分类                      | 含义                                                       | 特点                 |
| ------------------------- | ---------------------------------------------------------- | -------------------- |
| 聚集索引(Clustered Index) | 将数据存储与索引放到了一块，索引结点的叶子结点保存了行数据 | 必须有，而且只有一个 |
| 二级索引(Secondary Index) | 将数据与索引分开存储，索引结构的叶子结点关联的是对应的主键 | 可以存在多个         |



##### 2.1.2.2、聚集索引选取规则

- 如果存在主键，主键索引就是聚集索引。
- 如果不存在主键，将使用第一个唯一索引作为聚集索引。
- 如果表没有主键，或没有合适的唯一索引，则 `InnoDB` 会自动生成一个 `rowid` 作为隐藏的聚集索引。

![image-20241112141331314](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121413453.png)

##### 2.1.2.3、案例

    select * from user where name = 'Arm';

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121413812.png" alt="image-20241112141351681" style="zoom: 50%;" />

> 通过二级索引找到对应的值，然后再到聚集索引，这样的操作被称为**回表查询**。

##### 2.1.2.4、思考

（1）以下 `SQL` 语句，哪个执行效率高？为什么？

```mysql
-- id为主键，name字段创建的索引
select * from user where id = 10;
select * from user where name = 'Arm';
```

**答案：** 第一行 `id` 查询的执行效率更高，因为它只用通过一次查询即可找到对应的值，而 `name` 查询需要通过回表查询操作，效率是不及 `id` 查询的。

（2）`InnoDB` 主键索引的 `B+tree` 高度为多高呢？

**假设：** 一行数据大小为 `1k` ，一页中可以存储 `16` 行这样的数据。`InnoDB` 的指针占用 `6` 个字节的空间，主键假设为 `bigint` ，占用字节数为 `8` 。

**高度为 2 ：** n ∗ 8 + ( n + 1 ) ∗ 6 = 16 ∗ 1024 n\*8+(n+1)\*6=16*1024 n∗8+(n+1)∗6=16∗1024 ，其中 `n` 指代当前结点存储 `key` 的数量且 `n+1` 指代指针的数量，算出 `n` 约为 `1170` 即有 `1170` 个 `key` 和 `1171` 个指针，故大概能存储的数据量为 1171 ∗ 16 = 18736 1171*16=18736 1171∗16=18736 。

**高度为 3 ：** 大概能存储的数据量为 1171 ∗ 1171 ∗ 16 = 21939856 1171\*1171\*16=21939856 1171∗1171∗16=21939856 ，因为 `1171` 个指针指向的每个结点又有 `1171` 个指针指向下面的数据。



#### 2.1.3、索引语法

**创建索引**

```mysql
CREATE [UNIQUE|FULLTEXT] INDEX index_name ON table_name (index_col_name,...);
```

**查看索引**

```mysql
SHOW INDEX FROM table_name;
```

**删除索引**

```mysql
DROP INDEX index_name ON table_name;
```



**案例**

按照下列需求，完成索引的创建：

1.  `name` 字段为姓名字段，该字段的值可能会重复，为该字段创建索引。
2.  `phone` 手机号字段的值，是非空，且唯一的，为该字段创建唯一索引。
3.  为 `profession` 、`age` 、`status` 创建联合索引。
4.  为 `email` 建立合适的索引来提升查询效率。

![image-20241112141459081](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121414271.png)

**需求一**

`name` 字段为姓名字段，该字段的值可能会重复，为该字段创建索引。

修改前：

![image-20241112141532034](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121415259.png)

进行操作：

```mysql
create index idx_user_name on tb_user(name);
```

操作后：

![image-20241112141516244](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121415355.png)

**需求二**

`phone` 手机号字段的值，是非空，且唯一的，为该字段创建唯一索引。

进行操作：

```mysql
create unique index idx_user_phone on tb_user(phone);
```

操作后：

![image-20241112142146880](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121421079.png)

**需求三**

为 `profession` 、`age` 、`status` 创建联合索引。

进行操作：

```mysql
create index idx_user_pro_sta on tb_user(profession,age,status);
```

操作后：

![image-20241112142156489](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121421625.png)

**需求四**

为 `email` 建立合适的索引来提升查询效率。

进行操作：

```mysql
create index idx_user_email on tb_user(email);
```

操作后：

![image-20241112142204162](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121422282.png)

再删除它：

```mysql
drop index idx_user_email on tb_user;
```



#### 2.1.4、SQL性能分析

##### 2.1.4.1、SQL执行频率

`MySQL` 客户端连接成功后，通过 `show [session|global] status` 命令可以提供服务器状态信息。通过如下指令，可以查看当前数据库的 `INSERT` 、`DELETE` 、`SELECT` 的访问频次：

```mysql
SHOW GLOBAL STATUS LIKE 'Com_______';
```

**案例**

每一个下划线代表一个字符，下面是 `7` 个下划线。我们可以通过查询 `SQL` 的执行频次来判断该数据库是以什么操作为主，从而可以针对数据库进行性能的优化。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121427304.png" alt="image-20241112142738132" style="zoom:50%;" />

##### 2.1.4.2、慢查询日志

慢查询日志记录了所有执行时间超过指定参数（`long_query_time` ，单位：秒，默认 `10` 秒）的所有 `SQL` 语句的日志。

`MySQL` 的慢查询日志默认没有开启，需要在 `MySQL` 的配置文件（`/etc/my.cnf`）中配置如下信息：

```mysql
# 开启MySQL慢日志查询开关
slow_query_log = 1
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time = 2
```

##### 2.1.4.3、profile详情

`show profiles` 能够在左 `SQL` 优化时帮助我们了解时间都耗费到哪里去了。通过 `have_profiling` 参数，能够看到当前 `MySQL` 是否支持 `profile` 操作：

```mysql
SELECT @@have_profiling;
```

默认 `profiling` 是关闭的，可以用过 `set` 语句在 `session/global` 级别开启 `profiling` ：

```mysql
SET profiling = 1;
```

执行一系列的业务 `SQL` 的操作，然后通过如下指令查看指令的执行耗时：

```mysql
# 查看每一条SQL的耗时基本情况
show profiles;

# 查看指定query_id的SQL语句各个阶段的耗时情况
show profile for query_id;

# 查看指定query_id的SQL语句CPU的使用情况
show profile cpu for query_id;
```

##### 2.1.4.4、explain执行计划

`EXPLAIN` 或者 `DESC` 命令获取 `MySQL` 如何执行 `SELECT` 语句的信息，包括在 `SELECT` 语句执行过程中表如何连接和连接的顺序。

语法：

```mysql
# 直接在select语句之前加上关键字explain/desc
EXPALIN SELECT 字段列表 FROM 表名 WHERE 条件;
```

`EXPALIN` 执行计划各字段含义：

- `id`

  `select` 查询的序列号，表示查询中执行 `select` 字句或者是操作表的顺序（若 `id` 相同，执行顺序从上到下；若 `id` 不同，值越大，越先执行）。

- `select_type`

  表示 `SELECT` 的类型，常见的取值有 `SIMPLE`（简单表，即不使用表连接或者子查询）、`PRIMARY`（主查询，即外层的查询）、`UNION`（`UNION` 中的第二个或者后面的查询语句）、`SUBQUERY`（`SELECT/WHERE` 之后包含了子查询）等。

- `type`

  表示连接类型，性能由好到差的连续类型为 `NULL` 、`system` 、`const` 、`eq_ref` 、`ref` 、`range` 、`index` 、`all` 。

- `possible_key`

  显示可能应用在这张表上的索引，一个或多个。

- `key`

  实际使用的索引，如果为 `NULL` ，则没有使用索引。

- `Key_len`

  表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好。

- `rows`

  `MySQL` 认为必须要执行查询的行数，在 `innodb` 引擎的表中，是一个估计值，可能并不是准确的。

- `filtered`

  表示返回结果的行数站需读取行数的百分比，`filtered` 的值越大越好。



#### 2.1.5、索引使用

##### 2.1.5.1、验证索引效率

在未建立索引之前，执行如下 `SQL` 语句，查看 `SQL` 的耗时。

```mysql
SELECT * FROM tb_sku WHERE sn = '100000003145001';
```

![image-20241112142830311](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121428403.png)

针对字段创建索引：

```mysql
create index idx_sku_sn on tb_sku(sn);
```

然后再执行相同的 `SQL` 语句，再次查看 `SQL` 的耗时。

```mysql
SELECT * FROM tb_sku WHERE sn = '100000003145001';
```

![image-20241112142849544](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121428626.png)



##### 2.1.5.2、最左前缀法则

如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，**索引将部分失效（后面的字段索引失效）**。

```mysql
-- 全都生效
explain select * from tb_user where profession='软件工程' and age=31 and status='0';
-- 全部生效
explain select * from tb_user where profession='软件工程' and age=31;
-- 全部生效
explain select * from tb_user where profession='软件工程';
-- 全部失效，因为少了最左列profession
explain select * from tb_user where age=31 and status='0';
-- 全部失效，因为少了最左边的profession和age
explain select * from tb_user where status='0';
```



##### 2.1.5.3、范围查询

联合索引中，出现范围查询（`>` ，`<`），**范围查询右侧的列索引失效**。如果想避免失效，尽量使用 `>=` 和 `<=` 。

```mysql
-- status失效
explain select * from tb_user where profession='软件工程' and age>30 and status='0';
-- 全部生效
explain select * from tb_user where profession='软件工程' and age>=30 and status='0';
```



##### 2.1.5.4、索引失效情况

**索引列运算**

不要在索引列上进行运算操作，**索引将失效**。

```mysql
-- 索引失效
explain select * from tb_user where substring(phone,10,2)='15';
```

**字符串不加引号**

字符串类型字段使用时，不加引号，**索引失效**。

```mysql
-- 索引失效
explain select * from tb_user where phone=17799990015;
```

**模糊查询**

如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

```mysql
-- 索引生效
explain select * from tb_user where profession like '软件%';
-- 索引失效
explain select * from tb_user where profession like '%工程';
```

**or 连接的条件**

用 `or` 分割开的条件，如果 `or` 前的条件中的列有索引，而后面的列没有索引，那么涉及的索引都不会被用到。

```mysql
-- 由于age没有索引，所以即使id、phone有索引，索引也会失效
explain select * from tb_user where id=10 or age=23;
explain select * from tb_user phone='17799990017' or age=23;
```

**数据分布影响**

如果 `MySQL` 评估使用索引比全表更慢，则不使用索引。

```mysql
-- 全表扫描
select * from tb_user where phone >= '17799990005';
-- 使用索引
select * from tb_user where phone >= '17799990015';
```



##### 2.1.5.4、SQL提示

`SQL` 提示是优化数据库的一个重要手段，简单来说，就是在 `SQL` 语句中加入一些人为的提示来达到优化操作的目的。

**use index：** 建议用该索引

```mysql
explain select * from tb_user use index(idx_user_pro) where profession='软件工程'；
```

**ignore index：** 忽略该索引

```mysql
explain select * from tb_user ignore index(idx_user_pro) where profession='软件工程';
```

**force index：** 强制用该索引

```mysql
explain select * from tb_user force index(idx_user_pro) where profession='软件工程';
```



##### 2.1.5.6、覆盖索引

尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到），减少 `select *` 。

```mysql
-- 没有回表
explain select id,profession from tb_user where profession='软件工程' and age=31 and status='0';
explain select id,profession,age,status from tb_user where profession='软件工程' and age=31 and status='0';

-- 回表查询，name字段需要回表
explain select id,profession,age,status,name from tb_user where profession='软件工程' and age=31 and status='0';
explain select * from tb_user where profession='软件工程' and age=31 and status='0';
```

> 提示：
>
> `using index condition` ：查找使用了索引，但是需要回表查询数据。
>
> `using where; using index` ：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据。

**案例**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121439396.png" alt="image-20241112143923292" style="zoom:67%;" />

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121439389.png" alt="image-20241112143935208" style="zoom:50%;" />

```mysql
-- 只用到了聚集索引
select * from tb_user where id=2;
-- 只用到了辅助索引
select id,name from tb_user where name='Arm';
-- gender需要进行回表查询
select id,name,gender from tb_user where name='Arm';
```

**思考**

一张表，有四个字段（`id,username,password,status`），由于数据量大，需要对以下 `SQL` 语句进行优化，该如何进行才是最优方案：

```mysql
select id,username,password from tb_user where username='itcast';
```

**答案：** 应该对 `username` 和 `password` 建立联合索引，这样就不用进行回表查询了。

> **更正：** 这里之前写的是对 `id` 和 `password` 简历联合索引，这是错的，应该是对 `username` 和 `password` 简历联合索引，感谢 @TheShy_liang 大佬纠正！



##### 2.1.5.6、前缀索引

当字段类型为字符串（`varchar` ，`text` 等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘 `IO` ，影响查询效率。此时可以只讲字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

**语法**

```mysql
create index idx_xxx on table_name(column(n));
```

**前缀长度**

可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则效率越高，唯一索引的选择性为 `1` ，这是最好的索引选择性，性能也是最好的。

```mysql
select count(distinct email)/count(*) from tb_user;
select count(distinct substring(email,1,5))/count(*) from tb_user;
create index idx_email_5 on tb_user(email(5));
```

**案例**

![image-20241112144643909](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121446086.png)



##### 2.1.5.6、单列索引与联合索引

**单列索引：** 即一个索引只包含单个列。

**联合索引：** 即一个索引包含了多个列。

在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。

**单列索引情况：**

```mysql
explain select id,phone,name from tb_user where phone='17799990010' and name='韩信';
```

![image-20241112144657753](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121446861.png)

> **注意：** 多条件联合查询时，`MySQL` 优化器会评估哪个字段的索引效率更高，会选择该索引完成本次查询。

**联合索引情况：**

```mysql
create unique index idx_phone_name on tb_user(phone,name);
```

![image-20241112144705287](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121447390.png)



#### 2.1.6、 索引设计原则

1.  针对于数据量较大，且查询比较频繁的表建立索引。
2.  针对于常作为查询条件（`where`）、排序（`order by`）、分组（`group by`）操作的字段建立索引。
3.  尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4.  如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5.  尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
6.  要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7.  如果索引列不能存储 `NULL` 值，请在创建表时使用 `NOT NULL` 约束它。当优化器知道每列是否包含 `NULL` 值时，它可以更好的确定哪个索引最有效地用于查询。





## 3、SQL优化

### 3.1、插入数据

#### 3.1.1、insert优化

**批量插入**

```mysql
insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
```

**手动提交事务**

```mysql
start transaction;
insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
insert into tb_test values(4,'Tom'),(5,'Cat'),(6,'Jerry');
insert into tb_test values(7,'Tom'),(8,'Cat'),(9,'Jerry');
commit;
```


**主键顺序插入**

主键顺序插入性能高于乱序插入。

```mysql
主键乱序插入：8  1  9  21  88  2  4  15  89  5  7  3
主键顺序插入：1  2  3  4  5  7  8  9  15  21  88  89
```



#### 3.1.2、大批量插入数据

如果一次性需要插入大批量数据，使用 `insert` 语句插入性能较低，此时可以使用 `MySQL` 数据库提供的 `load` 指令进行插入。操作如下：

```mysql
# 客户端连接服务端时，加上参数 --local-infile
mysql --local-infile -u root -p;
# 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile = 1;
# 执行load指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table 'tb_user' fields terminated by ',' lines terminated by '\n';
```


![image-20241112150259839](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121503081.png)



### 3.2、主键优化

##### 3.2.1、数据组织方式

在 `InnoDB`存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为**索引组织表**。

![image-20241112150307356](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121503587.png)

##### 3.2.2、页分裂

页可以为空，也可以填充一半，也可以填充 `100％` 。每个页包含了 `2-N` 行数据（如果一行数据过大，会行溢出），根据主键排列。

**主键顺序插入**

每次页满了就会去开辟新的页，以此类推。

![image-20241112150346622](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121503840.png)

**主键乱序插入**

看下面这个例子，如果插入 `50` 就发现不能直接开辟新的页插入到后面，因为其索引值在第一页和第二页之间。

![image-20241112150405169](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121504278.png)

这就需要将要插入地方的那个页进行分裂，将一部分放到新开辟的那个页当中。

![image-20241112150416038](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121504150.png)

这样就有地方可以插入 `50` 了，即直接插入到第三页的后面。

![image-20241112150425782](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121504882.png)

最后还需要调整一下顺序，现在第三页需要接到第一页的后面去。

![image-20241112150441724](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121504836.png)

##### 3.2.3、页合并

当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（`flaged`）为删除并且它的空间变得允许被其他记录声明使用。

![image-20241112150453434](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121504538.png)

当页中删除的记录达到 `MERGE_THRESHOLD`（默认为页的 `50％`），`InnoDB` 会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。

![image-20241112150513589](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121505715.png)

> **注意：** `MERGE_THRESHOLD` ：合并页的阈值，可以自己设置，在创建表或者创建索引时指定。

##### 3.2.4、主键的设计原则

1.  满足业务需求的情况下，尽量降低主键的长度。
2.  插入数据时，尽量选择顺序插入，选择使用 `AUTO_INCREMENT` 自增主键。
3.  尽量不要使用使用 `UUID` 做主键或者是其它自然主键，如身份证号。
4.  业务操作，避免对主键的修改。



### 3.3、order by优化

#### 3.3.1、情况

（1）`Using filesort` ：通过表的索引或全表扫描，读取满足条件的数据行，然后再排序缓冲区 `sort buffer` 中完成排序操作，所有不是通过索引直接返回排序结果的都叫 `FileSort` 排序。

（2）`Using index` ：通过有序索引顺序扫描直接返回有序数据，这种情况即为 `using index` ，不需要额外排序，操作效率高。

```mysql
# 没有创建索引时，根据age,phone进行排序会显示Using filesort
explain select id,age,phone from tb_user order by age,phone;
# 创建索引
create index idx_user_age_phone_aa on tb_user(age,phone);
# 创建索引后，根据age,phone进行升序排序会显示Using index
explain select id,age,phone from tb_user order by age,phone;
# 创建索引后，根据age,phone进行降序排序会显示Using index
explain select id,age,phone from tb_user order by age desc,phone desc;

# 根据age,phone进行排序一个升序，一个降序会显示Using filesort
explain select id,age,phone from tb_user order by age asc,phone desc;
# 创建索引
create index idx_user_age_phone_ad on tb_user(age,asc,phone desc);
# 根据age,phone进行排序一个升序，一个降序会显示Using index
explain select id,age,phone from tb_user order by age asc,phone desc;
```


![image-20241112150526667](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411121505782.png)

#### 3.3.2、总结

1.  根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。
2.  尽量使用覆盖索引。
3.  多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则（`ASC/DESC`）
4.  如果不可避免的出现 `filesort` ，大数据量排序时，可以适当增大排序缓冲区大小 `sort_buffer_size`（默认 `256k`）。



### 3.4、group by优化

在分组操作时，可以通过索引来提高效率。

分组操作时，索引的使用也是满足最左前缀法则的。

```mysql
# 删除掉目前的联合索引idx_user_pro_age_sta
drop index idx_user_pro_age_sta on tb_user;
# 执行分组操作，根据profession字段分组
explain select profession,count(*) from tb_user group by profession;
# 创建索引
create index idx_user_pro_age_sta on tb_user(profession,age,status);
# 执行分组操作，根据profession字段分组
explain select profession,count(*) from tb_user group by profession;
# 执行分组操作，根据profession字段分组
explain select profession,count(*) from tb_user group by profession,age;
```



### 3.5、 limit优化

一个常见又非常头疼的问题是 `limit 2000000,10` ，此时需要 `MySQL` 排序前 `2000010` 的记录，仅仅返回 `2000000 - 2000010` 的记录，其他记录丢弃，查询排序的代价非常大。

**优化思路：** 一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。

```mysql
explain select * from tb_sku t,(select id from tb_sku order by id limit 2000000,10) a where t.id=a.id;
```



### 3.6、 count优化

#### 3.6.1、操作

```mysql
explain select count(*) from tb_user;
```


`MuISAM` 引擎把一个表的总行数存放在了磁盘上，因此执行 `count(*)` 的时候会直接返回这个数，效率很高。

`InnoDB` 引擎就麻烦了，它执行 `count(*)` 的时候，需要把数据一行一行地从引擎里面读出来，然后累计计数。

**优化思路：** 自己计数。

#### 3.6.2、count的几种用法

`count()` 是一个聚合函数，对于返回的结果集，一行行地判断，如果 `count` 函数的参数不是 `NULL` ，累计值就加 `1` ，否则不加，最后返回累计值。

**用法：** `count(*)、count(主键)、count(字段)、count(1)`

> 按照效率排序的话，`count(字段) < count(主键id) < count(1) ≈ count(*)` ，所以尽量使用 `count(*)`
>
> 



### 3.7、update优化

尽量根据主键/索引字段进行数据更新。

```mysql
update student set no='2000100100' where id=1;
update student set no='2000100105' where name='韦一笑';
```

`InnoDB` 的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，否则会从行锁升级为表锁。





## 4、视图/存储过程/触发器

### 4.1、 视图

#### 4.1.1、介绍

视图（`View`）是一种虚拟存在的表。视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

通俗的讲，视图只保存了查询的 `SQL` 逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在创建这条 `SQL` 查询语句上。

#### 4.1.2、创建

```mysql
CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 [WITH[CASCADED|LOCAL]CHECK OPTION];
```

**案例**

```mysql
create or replace view stu_v_1 as select id,name from student where id<=10;
```


#### 4.1.3、查询

```mysql
查看创建视图语句：SHOW CREATE VIEW 视图名称;
查看视图数据：SELECT * FROM 视图名称 ......;
```

**案例**

```mysql
show create view stu_v_1;
```


![image-20241112210612006](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122106243.png)

```mysql
select * from stu_v_1;
```


![image-20241112210624946](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122106144.png)

#### 4.1.4、修改

```mysql
方式一：CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 [WITH[CASCADED|LOCAL]CHECK OPTION];
方式二：ALTER VIEW 视图名称[(列名列表)] AS SELECT语句 [WITH[CASCADED|LOCAL]CHECK OPTION];
```

**案例**

```mysql
create or replace view stu_v_1 as select id,name,no from student where id<=10;
alter view stu_v_1 as select id,name from student where id<=10;
```


#### 4.1.5、删除

```mysql
DROP VIEW [IF EXISTS] 视图名称 [,视图名称] ...;
```


**案例**

```mysql
drop view if exists stu_v_1;
```



#### 4.1.6、检查选项

在创建视图的那个案例中，我们执行了下面这条语句：

```mysql
create or replace view stu_v_1 as select id,name from student where id<=10;
```


但是上面这个案例会出现一个问题，它并不会帮忙检查插入的选项是否有问题，比如：

```mysql
insert into stu_v_1 values(6,'Tom');
```


上面我往视图中插入一个 `(6,'Tom')` 的数据，在基表和视图中就会增加这一条数据，但是如果我执行下面这条语句：

```mysql
insert into stu_v_1 values(30,'Jack');
```


同样会插入成功，但是 `(30,'Jack')` 这条数据只出现在了基表当中，视图中并没有出现，这是因为我们创建视图时要求的条件为 `id<=10` ，这就导致视图中的数据和基表中的不一致。

为了改变这种情况，我们可以在尾部加上检查选项的语法：

```mysql
# 这个场景下，cascaded和local都可以
create or replace view stu_v_1 as select id,name from student where id<=10 with cascaded check option;
create or replace view stu_v_1 as select id,name from student where id<=10 with local check option;
```


这样，当我们再次插入 `(30,'Jack')` 这条数据时就会报错，系统帮我们检查出了这个问题。

所以，当使用 `WITH CHECK OPTION` 字句创建视图时，`MySQL` 会通过视图检查正在更改的每个行，例如插入、更新和删除，以使其符合视图的定义。`MySQL` 允许基于另一个视图创建视图，它还会检查依赖视图中的规则以保持一致性。为了确定检查的范围，`MySQL` 提供了两个选项：

`CASCADED` 和 `LOCAL` ，默认值为 `CASCADED` 。

##### 4.1.6.1、CASCADED

不加检查选项：

执行下面两个插入语句并不会报错。

```mysql
create or replace view stu_v_1 as select id,name from student where id<=20;

insert into stu_v_1 values(5,'Tom');

insert into stu_v_1 values(25,'Tom');
```


新创建一个基于 `stu_v_1` 的视图 `stu_v_2` ，使它们关联起来，并且创建 `stu_v_2` 时加上检查选项的语句：

```mysql
create or replace view stu_v_1 as select id,name from stu_v_1 where id>=10 with cascaded check option;
```


这时候再执行插入语句：

```mysql
# 执行失败，因为视图2要求id>=10
insert into stu_v_2 values(7,'Tom');
# 执行失败，因为视图1要求id<=20
insert into stu_v_2 values(26,'Tom');
# 执行成功，满足10<=id<=20
insert into stu_v_2 values(15,'Tom');
```


可以发现，如果 `stu_v_2` 加上检查选项的语句，那么与它关联的 `stu_v_1` 也相当于加上了检查选项的语句。

我们再创建一个基于 `stu_v_2` 的视图，但是不加检查选项的语句：

```mysql
create or replace view stu_v_3 as select id,name from stu_v_2 where id<=15;
```


再去执行插入操作：

```mysql
# 执行成功，满足id<=15 && id>=10 && id<=20
insert into stu_v_2 values(11,'Tom');
# 执行成功，因为视图3没有加检查选择选项，所以不会被id<=15所限制
insert into stu_v_2 values(17,'Tom');
# 执行失败，虽然不会被视图3所限制，但是它依赖的视图2依赖视图1，满足视图2但不满足视图1的id<=20
insert into stu_v_2 values(28,'Tom');
```

##### 4.1.6.2、LOCAL

不加检查选项：

我们同样像上面一样创建一个视图先，下面两个插入语句同样可以执行成功。

```mysql
create or replace view stu_v_4 as select id,name from student where id<=15;

insert into stu_v_4 values(5,'Tom');

insert into stu_v_4 values(16,'Tom');
```


我们再基于 `stu_v_4` 创建一个视图 `sty_v_5` ：

```mysql
create or replace view stu_v_5 as select id,name from stu_v_4 where id>=10 with local check option;
```


再执行插入操作：

```mysql
# 执行成功
insert into stu_v_5 values(13,'Tom');
# 执行成功
insert into stu_v_5 values(17,'Tom');
```


可以发现，如果 `stu_v_5` 它依赖的视图 `stu_v_4` 没有加检查选项的语句，是不用去满足 `stu_v_4` 的条件的。

我们再创建一个基于 `stu_v_5` 的视图：

```mysql
create or replace view stu_v_6 as select id,name from stu_v_5 where id<20;
```


继续执行插入操作：

```mysql
# 执行成功
insert into stu_v_5 values(14,'Tom');
```



#### 4.1.7、总结

下面归纳的是创建一个关联视图即视图 `2` 依赖于视图 `1` 的情况：

|   选项   |                             功能                             |
| :------: | :----------------------------------------------------------: |
| CASCADED | 如果视图 `2` 加了检查选项，那么不管视图 `1` 有没有加检查选项，都要对视图 `1` 的条件进行检查；如果视图 `2` 没有加检查选项，那么只有当视图 `1` 加了检查选项才对视图 `1` 的条件进行检查。 |
|  LOCAL   | 不管视图 `2` 有没有加检查选项，视图 `1` 只有加了检查选项才对视图 `1` 的条件进行检查。 |

##### 4.1.7.1、视图的更新

要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含以下任何一项，则视图不可更新：

1.  聚合函数或窗口函数（`SUM()` 、`MIN()` 、`MAX()` 、`COUNT()` 等）
2.  `DISTINCT`
3.  `GROUP BY`
4.  `HAVING`
5.  `UNION` 或者 `UNION ALL`

##### 4.1.7.2、作用

* **简单**

  视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次都指定全部的条件。

* **安全**

  数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据。

* **数据独立**

  视图可以帮助用户屏蔽真实表结构变化带来的影响。

##### 4.1.7.3、案例

根据如果需求，定义视图：

1. 为了保证数据库的安全性，开发人员在操作 `tb_user` 表时，只能看到用户的基本字段，屏蔽手机号和邮箱两个字段。

2. 查询每个学生所选修的课程（三张表联查），这个功能在很多的业务中都有使用到，为了简化操作，定义一个视图。

   ```mysql
   -- 需求1
   create view tb_user_view as select id,name,profession,age,gender,status,createtime from tb_user;
   
   select * from tb_user_view;
   
   -- 需求2
   # 三表联查
   # select s.name,s.no,c.name from student s,student_course sc,course c where s.id=sc.student_id and sc.courseid=c.id;
   
   # 将上面的语句接到视图创建的语句之后即可
   # 下面给s.name和s.no和c.name取了别名，不取就会报错，因为在s表和c表中都有name这个标题，不取别名会出现重复标题
   create view tb_stu_course_view as select s.name student_name,s.no student_no,c.name course_name from student s,student_course sc,course c where s.id=sc.student_id and sc.courseid=c.id;
   ```

   

### 4.2、 存储过程

#### 4.2.1、介绍

存储过程是事先经过编译并存储在数据库中的一段 `SQL` 语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和服务器之间的传输，对于提高数据处理的效率是有好处的。

存储过程思想很想简单，就是数据库 `SQL` 语言层面的代码封装与重用。

![image-20241112211116169](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122111461.png)

**特点：**

*   封装，复用。
*   可以接收参数，也可以返回数据。
*   减少网络交互，效率提升。



#### 4.2.2、创建

```mysql
CREATE PROCEDURE 存储过程名称([参数列表])
BEGIN
	-- SQL语句
END;
```


> **注意：** 在命令行中，执行创建存储过程的 `SQL` 时，需要通过关键字 `delimiter` 指定 `SQL` 语句的结束符。



#### 4.2.3、调用

```mysql
CALL 名称([参数]);
```


**案例**

没改变结束符：

```mysql
create procedure p1()
begin
	select count(*) from student;
end;

call p1();
```


改变结束符：

```mysql
delimiter $$	-- 修改结束符为$$

create procedure p1()
begin
	select count(*) from student;
end$$

call p1();
$$

delimiter ;		-- 将结束符改回;
```



#### 4.2.4、查看

```mysql
-- 查询指定数据库的存储过程及状态信息
SELECT * FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'xxx';
-- 查询某个存储过程的定义
SHOW CREATE PROCEDURE 存储过程名称;
```


#### 4.2.5、删除

```mysql
DROP PROCEDURE [IF EXISTS] 存储过程名称;
```



#### 4.2.6、变量

**系统变量**是 `MySQL` 服务器提供，不是用户定义的，属于服务器层面。分为全局变量（`GLOBAL`）、会话变量（`SESSION`）。

**查看系统变量**

```mysql
SHOW [SESSION|GLOBAL] VARIABLES;	-- 查看所有系统变量
SHOW [SESSION|GLOBAL] VARIABLES LIKE '...';	-- 可以通过LIKE模糊匹配方式查找变量
SELECT @@[SESSION|GLOBAL] 系统变量名;	-- 查看指定变量的值
```


**设置系统变量**

```mysql
SET [SESSION|GLOBAL] 查看变量名=值;
SET @@[SESSION|GLOBAL] 系统变量名=值;
```


> **注意：**
>
> 如果没有指定 `SESSION/GLOBAL` ，默认是 `SESSION` ，会话变量。
>
> `MySQL` 服务重新启动后，所设置的全局变量参数会失效，要想不失效，可以在 `/etc/my.cnf/ 中配置。

**用户定义变量**是用户根据需要自己定义的变量，用户变量不同提前声明，在用的时候直接用"@变量名"使用就可以。其作用域为当前连接。

**赋值**

```mysql
SET @var_name = expr [,@var_name = expr]...;
SET @var_name := expr [,@var_name := expr]...;
```


```mysql
SELECT @var_name := expr [,@var_name := expr]...;
SELECT 字段名 INTO @var_name FROM 表名;
```

**使用**

```mysql
SELECT @var_name;
```


**案例**

```mysql
-- 赋值
set @myname = 'itcast';
set @mysqg := 10;
set @mygender := '男',@myhobby := 'java';

select @mycolor := 'red';
select count(*) into @mycount from tb_user;

-- 使用
select @myname,@myage,@mygender,@myhobby;

select @mycolor,@mycount;
```


> **注意：** 用户定义的变量无需对其进行声明或初始化，只不过获取到的值为 `NULL` 。

**局部变量**是根据需要定义的在局部生效的变量，访问之前，需要 `DECLARE` 声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的 `BEGIN ... END` 块。

**声明**

```mysql
DECLARE 变量名 变量类型 [DEFAULT ...];
```


变量类型就是数据库字段类型：`INT` 、`BIGINT` 、`CHAR` 、`VARCHAR` 、`DATE` 、`TIME` 等

**赋值**

```mysql
SET 变量名 = 值;
SET 变量名 := 值;
SELECT 字段名 INTO 变量名 FROM 表名 ...;
```


**案例**

```mysql
create procedure p()
begin
	declare stu_count int default 0;
	select count(*) into stu_count from student;
	select stu_count;
end;

call p();
```



#### 4.2.7、if

**语法：**

```mysql
IF 条件1 THEN
...
ELSEIF 条件2 THEN 	-- 可选
...
ELSE	 -- 可选
...
END IF;
```

**案例**

根据定义的分数 `score` 变量，判定当前分数对应的分数等级。

1. score >= 85 分，等级为优秀

2. score >= 60 分且 score < 85 分，等级为及格。

3. score < 60 分，等级为不及格。

   ```mysql
   create procedure p()
   begin
   	declare score int default 58;
   	declare result varchar(10);
   	
   	if score >= 85 then
   		set result := '优秀';
   	elseif score >= 60 then
   		set result := '及格';
   	else
   		set result := '不及格';
   	end if;
   	select result;
   end;
   
   call p();
   ```

   

#### 4.2.8、参数

| 类型  |                     含义                     | 备注 |
| :---: | :------------------------------------------: | :--: |
|  IN   |   该类参数作为输入，也就是需要调用时传入值   | 默认 |
|  OUT  | 该类参数作为输出，也就是该参数可以作为返回值 |      |
| INOUT |    既可以作为输入参数，也可以作为输出参数    |      |

**用法：**

```mysql
CREATE PROCEDURE 存储过程名称 ([IN/OUT/INOUT 参数名 参数类型])
BEGIN
	-- SQL语句
END;
```


**案例**

（1）根据传入参数 `score` ，判定当前分数对应的分数等级，并返回。

1. score >= 85 分，等级为优秀

2. score >= 60 分且 score < 85 分，等级为及格。

3. score < 60 分，等级为不及格。

   ```mysql
   create procedure p(in score int, out result varchar(10))
   begin
   	if score >= 85 then
   		set result := '优秀';
   	elseif score >= 60 then
   		set result := '及格';
   	else
   		set result := '不及格';
   	end if;
   	select result;
   end;
   
   call p(68, @result);
   select @result;
   ```

（2）将传入的 200 分制的分数，进行换算，换算成百分制，然后返回。

```mysql
create procedure p(inout score double)
begin
	set score := score * 0.5;
end;

set @score = 78;
call p(@score);
select @score;
```



#### 4.2.9、case

**语法一**

```mysql
CASE case_value
	WHEN when_value1 THEN statement_list1
	[WHEN when_value2 THEN statement_list2]...
	[ELSE statement_list]
END CASE;
```


**语法二**

```mysql
CASE
	WHEN search_condition1 THEN statement_list1
	[WHEN search_condition2 THEN statement_list2]...
	[ELSE statement_list]
END CASE;
```


**案例**

根据传入的月份，判定月份所属的季节（需求采用 `case` 结构）。

1. 1-3 月份，为第一季度

2. 4-6 月份，为第二季度

3. 7-9 月份，为第三季度

4. 10-12 月份，为第四季度

   ```mysql
   create procedure p(in month int)
   begin
   	declare result varchar(10);
   	
   	case
   		when month >= 1 and month <= 3 then
   			set result := '第一季度';
   		when month >= 4 and month <= 6 then
   			set result := '第二季度';
   		when month >= 7 and month <= 9 then
   			set result := '第三季度';
   		when month >= 10 and month <= 12 then
   			set result := '第四季度';
   		else
   			set result := '非法参数';
   	end case;
   	
   	select concat('您输入的月份为：',month,'，所属的季度为：',result);
   end;
   
   call p(4);
   ```



#### 4.2.10、while

`while` 循环是有条件的循环控制语句。满足条件后，再执行循环体中的 `SQL` 语句。具体语法为：

```mysql
# 先判定条件，如果条件为true，则执行逻辑；否则，不执行逻辑
WHILE 条件 DO
	SQL逻辑...
END WHILE;
```


**案例**

计算从 `1` 累加到 `n` 的值，`n` 为传入的参数值。

```mysql
create procedure p(in n int)
begin
	declare total int default 0;
	
	while n>0 do
		set total := total + n;
		set n := n - 1;
	end while;
	
	select total;
end;

call p(10);
```



#### 4.2.11、repeat

`repeat` 是有条件的循环控制语句，当满足条件的时候退出循环。具体语法为：

```mysql
#限制性一次逻辑，然后判定逻辑是否满足，如果满足，则推出。如果不满足，则继续下一次循环
REPEAT
	SQL逻辑...
	UNTIL 条件
END REPEAT;
```


**案例**

计算从 `1` 累加到 `n` 的值，`n` 为传入的参数值。

```mysql
create procedure p(in n int)
begin
	declare total int default 0;
	
	repeat
		set total := total + n;
		set n := n-1;
	until n<= 0;
	end repeat;
	
	select total;
end;

call p(10);
```



#### 4.2.12、loop

`LOOP` 实现简单的循环，如果不在 `SQL` 逻辑中增加退出循环的条件，可以用来其来实现简单的死循环。`LOOP` 可以配合一下两个语句使用：

* `LEAVE` ：配合循环使用，退出循环。

* `ITERATE` ：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。

  [begin_label:] LOOP
  	SQL逻辑...
  END LOOP [end_label];


  LEAVE label;	-- 退出指定标记的循环体
  ITERATE label;	-- 直接进入下一次循环

**案例**

1. 计算从 `1` 累加到 `n` 的值，`n` 为传入的参数值。

   ```mysql
   create procedure p(in n int)
   begin
   	declare total int default 0;
   	
   	sum:loop
   		if n <= 0 then
   			leave sum;
   		end if;
   		
   		set total := total + n;
   		set n := n - 1;
   	end loop sum;
   	
   	select total;
   end;
   
   call p(10);
   ```

1. 计算从 `1` 到 `n` 之间的偶数累加的值，`n` 为传入的参数值。

   ```mysql
   create procedure p(in n int)
   begin
   	declare total int default 0;
   	
   	sum:loop
   		if n <= 0 then
   			leave sum;
   		end if;
   		
   		if n % 2 = 1 then
   			set n := n - 1;
   			iterate sum;
   		end if;
   		
   		set total := total + n;
   		set n := n - 1;
   	end loop sum;
   	
   	select total;
   end;
   
   call p(10);
   ```

   

#### 4.2.13、游标

**游标（`CURSOR`）** 是用来存储查询结果集的数据类型，在存储过程和函数中可以使用游标对结果集进行循环的处理。游标的使用包括游标的声明、`OPEN` 、`FETCH` 和 `CLOSE` ，其语法分别如下。

**声明游标**

```mysql
DECLARE 游标名称 CURSOR FOR 查询语句;
```


**打开游标**

```mysql
OPEN 游标名称;
```


**获取游标记录**

```mysql
FETCH 游标名称 INTO 变量[,变量];
```


**关闭游标**

```mysql
CLOSE 游标名称;
```


**案例**

使用游标创建一个存储过程，统计 `STUDENT` 表年龄大于 `19` 的记录的数量，如下为 `STUDENT` 表的创建代码：

```mysql
DELIMITER $
CREATE PROCEDURE PROC_STAT()
BEGIN
    # 创建 用于接收游标值的变量
    DECLARE ID,AGE,TOTAL INT;
    DECLARE NAME,SEX CHAR(10);
    # 游标结束的标志
    DECLARE DONE INT DEFAULT 0;
    # 声明游标
    DECLARE CUR CURSOR FOR SELECT STUID,STUNAME,STUSEX,STUAGE FROM STUDENT WHERE STUAGE > 19;
    # 指定游标循环结束时的返回值 
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET DONE = 1;
    # 打开游标
    OPEN CUR;
    # 初始化变量
    SET TOTAL = 0;
    # WHILE 循环
    WHILE DONE != 1 DO
        FETCH CUR INTO ID,NAME,SEX,AGE;
          IF DONE != 1 THEN
             SET TOTAL = TOTAL + 1;
          END IF;    
    END WHILE;
    # 关闭游标
    CLOSE CUR;
    # 输出累计的结果
    SELECT TOTAL;
END$
DELIMITER ;
# 调用存储过程
CALL PROC_STAT();
```



#### 4.2.14、条件处理程序

**条件处理程序（Handler）** 可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤。具体语法为：

```mysql
DECLARE handler_action HANDLER FOR condition_value [,condition_value]... statement;

handler_action
	CONTINUE:继续执行当前程序
	EXIT:中止执行当前程序
condition_value
	SQLSTATE sqlstate_value:状态码，如02000，表示抓取的时候没数据
	SQLWARNING:所有以01开头的SQLSTATE代码的简写
	NOT FOUND:所有以02开头的SQLSTATE代码的简写
	SQLEXCEPTION:所有没有被SQLWARNNING或NOT FOUND捕获的SQLSTATE代码的简写
```


**案例**

根据传入的参数 `uage` ，来查询用户表 `tb_user` 中，所有的用户年龄小于等于 `uage` 的用户姓名（`name`）和专业（`profession`），并将用户的姓名和专业插入到所创建的一张新表（`id,name,profession`） 中。

逻辑：

1. 声明游标，存储查询结果集

2. 准备：创建表结构

3. 开启游标

4. 获取游标中的记录

5. 插入数据到新表中

6. 关闭游标

   ```mysql
   create procedure p(in uage int)
   begin
   	-- 需要先声明普通变量，再声明游标，不然会报错
   	declare uname varchar(100);
   	declare upro varchar(100);
   	declare u_cursor cursor for select name,profession from tb_user where age < = uage;
   	declare exit handler for SQLSTATE '02000' close u_cursor;
   	
   	drop table if exists tb_user_pro;
   	create table if not exists tb_user_pro(
       	id int primary key auto_increment,
           name varchar(100),
           profession varchar(100)
       );
       
       open u_cursor;
       while true do
       	fetch u_cursor into uname,upro;
       	insert into tb_user_pro values (null, uname, upro);
       end while;
       close u_cursor;
       
   end;
   
   call p(40);
   ```





### 4.3、存储函数

存储函数是有返回值的存储过程，存储函数的参数只能是 `IN` 类型的。具体语法如下：

```mysql
CREATE FUNCTION 存储函数名称([参数列表])
RETURNS type [characteristic...]
BEGIN
	-- SQL语句
	RETURN ...;
END;
```


`characteristic` 说明：

*   `DETERMINISTIC` ：相同的输入参数总是产生相同的结果。
*   `NO SQL` ：不包含 `SQL` 语句。
*   `READS SQL DATA` ：包含读取数据的语句，但不包含写入数据的语句。

**案例**

计算从 `1` 累加到 `n` 。

```mysql
create function fun1(n int)
return int deterministic
begin
	declare total int default 0;
	
	while n>0 do
		set total := total + n;
		set n := n - 1;
	end while;
	
	return total;
end;

select fun1(50);
```



### 4.4、触发器

触发器是与表有关的数据库对象，指在 `insert/update/delete` 之前或之后，触发并执行触发器中定义的 `SQL` 语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。

使用别名 `OLD` 和 `NEW` 来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。

|   触发器类型    |                       NEW 和 OLD                       |
| :-------------: | :----------------------------------------------------: |
| INSERT 型触发器 |             NEW 表示将要或者已经新增的数据             |
| UPDATE 型触发器 | OLD 表示修改之前的数据，NEW 表示将要或已经修改后的数据 |
| DELETE 型触发器 |             OLD 表示将要或者已经删除的数据             |



#### 4.4.1、创建

```mysql
CREATE TRIGGER trigger_name
BEFORE/AFTER INSERT/UPDATE/DELETE
ON tbl_name FOR EACH ROW  -- 行级触发器
BEGIN
	trigger_stmt;
END;
```


**查看**

```mysql
SHOW TRIGGERS;
```



#### 4.4.2、删除

```mysql
DROP TRIGGER [schema_name.]trigger_name;	--如果没有指定schema_name，默认为当前数据库
```


**案例**

通过触发器记录 `tu_user` 表的数据变更日志，将变更日志插入到日志表 `user_logs` 中，包含增加、修改和删除。

```mysql
-- 创建表结构
create table user_logs(
  id int(11) not null auto_increment,
  operation varchar(20) not null comment '操作类型, insert/update/delete',
  operate_time datetime not null comment '操作时间',
  operate_id int(11) not null comment '操作的ID',
  operate_params varchar(500) comment '操作参数',
  primary key(`id`)
)engine=innodb default charset=utf8;
```



#### 4.4.3、INSERT类型触发器：

**案例1**

编写一个名为 `INSERT_S` 的触发器，在 `S` 表执行 `INSERT` 语句后被激发，此触发器将新供应商的 `SNO`、`SNAME`、`STATUS`、`CITY` 及执行此操作的用户（`USER`）插入 `N_S` 表，`N_S` 表比 `S` 表增添操作用户一列。

```mysql
delimiter $
create trigger INSERT_S after insert on S for each row
begin
    insert into N_S(SNO,SNAME,STATUS,CITY,USER) VALUES(new.SNO,new.SNAME,new.STATUS,new.CITY,USER());
end$
delimiter ;
#将记录插入S表
INSERT INTO S VALUES ('S6', '深技大', '20', '深圳');
#查看N_S表
SELECT * FROM N_S;
```


**案例2**

编写一个名为 `INSERT_SPJ` 的触发器，当 `SPJ` 表中有插入某条记录时，自动更新表 `SPJ_SUMQTY` 表。

```mysql
delimiter $
create trigger INSERT_SPJ after insert on SPJ for each row
begin
    if exists(select * from SPJ_SUMQTY as s where s.PNO = new.PNO and s.JNO = new.JNO) then
        update SPJ_SUMQTY as s1 set s1.SUMQTY = s1.SUMQTY + new.QTY where s1.JNO = new.JNO and s1.PNO = new.PNO;
    else
        insert into SPJ_SUMQTY VALUES(new.JNO, new.PNO, new.QTY);
    end if;
end$
delimiter ;
#将记录插入SPJ表
INSERT INTO SPJ VALUES ('S6', 'P1', 'J6', 200);
INSERT INTO SPJ VALUES ('S6', 'P3', 'J5', 300);
#查看SPJ_SUMQTY表
SELECT * FROM SPJ_SUMQTY;
```


**案例3**

```mysql
-- 插入数据触发器
create trigger tb_user_insert_trigger
	after insert on tb_user for each row
begin
	insert into user_logs(id, operation, operate_time, operate_id, operate_params) VALUES
	(null, 'insert', now(), new.id, concat('插入的数据内容为：id=',new.id,',name=',new.name,',phone=',new.phone,',email=',new.email,',profession=',new.profession));
end;

-- 查看
show triggers;

-- 插入数据
insert into tb_user(id, name, phone, email, profession, age, gender, status, createtime)
VALUES (25,'二皇子','18809091212','erhuangzi@163.com','软件工程',23,'1','1',now());

-- 删除
drop trigger tb_user_insert_trigger;
```



#### 4.4.4、UPDATE 类型触发器：

**案例1**

编写一个名为 `UPDATE_S` 的触发器，检查 `S` 表的 `STATUS` ，只允许 `0-100` 之间，超过 `100` 后，改为 `100` 。

```mysql
delimiter $
create trigger UPDATE_S before update on S for each row
begin
    if new.STATUS > 100 then
    set new.STATUS = 100;
    end if;
end$
delimiter ;
#更新S表
UPDATE S SET S.STATUS = 300 WHERE S.CITY = '天津';
#查看S表
SELECT * FROM S;
```


**案例2**

编写一个名为 `UPDATE_SPJ` 的触发器，当 `SPJ` 表中有更新某条记录时，自动更新表 `SPJ_SUMQTY` 表。

```mysql
delimiter $
create trigger UPDATE_SPJ before update on SPJ for each row
begin
    update SPJ_SUMQTY as s1 set s1.SUMQTY = s1.SUMQTY + new.QTY - old.QTY where s1.JNO = new.JNO and s1.PNO = new.PNO;
end$
delimiter ;
#更新SPJ表
UPDATE SPJ SET SPJ.QTY = SPJ.QTY + 200 WHERE SPJ.JNO = 'J5';
#查看SPJ_SUMQTY表
SELECT * FROM SPJ_SUMQTY;
```


**案例3**

```mysql
-- 修改数据触发器
create trigger tb_user_update_trigger
	after update on tb_user for each row
begin
	insert into user_logs(id, operation, operate_time, operate_id, operate_params) VALUES
	(null, 'update', now(), new.id, concat('更新之前的数据：id=',new.id,',name=',old.name,',phone=',old.phone,',email=',old.email,',profession=',old.profession,
' | 更新之后的数据：id=',new.id,',name=',new.name,',phone=',new.phone,',email=',new.email,',profession=',new.profession));
end;

-- 查看
show triggers;

-- 更新数据
update tb_user set age = 32 where id = 23;
```



#### 4.4.5、DELETE类型触发器：

**案例1**

编写一个名为 `DELETE_SPJ` 的触发器，当 `SPJ` 表中有删除某条记录时，自动更新表 `SPJ_SUMQTY` 表。

```mysql
delimiter $
create trigger DELETE_SPJ after delete on SPJ for each row
begin
    update SPJ_SUMQTY as s1 set s1.SUMQTY = s1.SUMQTY - old.QTY where s1.JNO = old.JNO and s1.PNO = old.PNO;
    delete from SPJ_SUMQTY as s1 where s1.SUMQTY = 0;
end$
delimiter ;
#删除SPJ表的某条记录
DELETE FROM SPJ WHERE SPJ.SNO = 'S2' AND SPJ.PNO = 'P3' AND SPJ.JNO = 'J5';
DELETE FROM SPJ WHERE SPJ.SNO = 'S2' AND SPJ.PNO = 'P3' AND SPJ.JNO = 'J1';
#查看SPJ_SUMQTY表
SELECT * FROM SPJ_SUMQTY;
```


**案例2**

```mysql
-- 修改数据触发器
create trigger tb_user_delete_trigger
	after delete on tb_user for each row
begin
	insert into user_logs(id, operation, operate_time, operate_id, operate_params) VALUES
	(null, 'delete', now(), old.id, concat('删除之前的数据：id=',new.id,',name=',old.name,',phone=',old.phone,',email=',old.email,',profession=',old.profession));
end;

-- 查看
show triggers;

-- 更新数据
delete from tb_user where id = 25;
```





## 5、锁

### 5.1、概述

#### 5.1.1、介绍

锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（`CPU` 、`RAM` 、`I/O`）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

#### 5.1.2、分类

`MySQL` 中的锁，按照锁的粒度分，分为以下三类：

1.  全局锁：锁定数据库中的所有表。
2.  表级锁：每次操作锁住整张表。
3.  行级锁：每次操作锁住对应的行数据。



### 5.2、全局锁

#### 5.2.1、介绍

全局锁就是对整个数据库实例加锁，加锁后整个整个实例就处于只读状态，后续的 `DML` 的写语句，`DDL` 语句，已经更新操作的事务提交语句都将被阻塞。

将典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性。

**没加锁之前（数据不一致）：**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122154130.png" alt="image-20241112215400738" style="zoom: 67%;" />

**加锁过程：**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122154884.png" alt="image-20241112215408340" style="zoom:67%;" />

**加锁**

```mysql
flush tables with read lock;
```


**备份数据**

注意下面这个命令需要在 `windows` 下的命令行执行：

```shell
mysqldump -uroot -p1234 itcast > itcast.sql
```


**解锁**

```mysql
unlock tables;
```


![image-20241112215424208](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122154542.png)



#### 5.2.2、特点

数据库中加全局锁，是一个比较重的操作，存在以下问题：

1.  如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆。
2.  如果在从库上更新，那么在备份期间不能执行主库同步过来的二进制日志（binlog），会导致主从延迟。

在 `InnoDB` 引擎中，我们可以在备份时加上参数 `--single-transaction` 参数来完成**不加锁的一致性数据备份**。

```shell
mysqldump --single-transaction -uroot -p123456 itcast > itcast.sql
```


![image-20241112220038559](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122200988.png)



### 5.3、表级锁

#### 5.3.1、介绍

表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在 `MyISAM` 、`InnoDB` 、`BDB` 等存储引擎中。

对于表级锁，主要分为以下三类：

1.  表锁
2.  元数据锁（`meta data lock` ，`MDL`）
3.  意向锁



#### 5.3.2、表锁

对于表锁，分为两类：

读锁不会阻塞其他客户端的读，但会阻塞写。写锁既会阻塞其他客户端的读，又会阻塞其他客户端的写。

1.  **表共享读锁（`read lock`）**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122154555.png" alt="image-20241112215445289" style="zoom: 50%;" />

2.  **表独占写锁（`write lock`）**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122154652.png" style="zoom:50%;" />

> 注意：上面两图左边代表本客户端的操作，右边代表其他客户端的操作，并且绿色代表课操作，红色代表不可操作。

语法：

1. 加锁：

   ```mysql
   lock tables 表名... read/write
   ```

2. 释放锁

   ```mysql
   unlock tables / 客户端断开连接
   ```

   

#### 5.3.3、元数据锁

`MDL` 加锁过程是系统自动控制，无需显示使用，在访问一张表的时候会自动加上。`MDL` 锁主要作用是维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作。为了避免 `DML` 与 `DDL` 冲突，保证读写的正确性。

在 `MYSQL5.5` 中引入 `MDL` ，当对一张表进行增删改查的时候，加 `MDL` 读锁（共享）；当对表结构进行变更操作的时候，加 `MDL` 写锁（排他）。

|                    对应SQL                     |                  锁类型                   |                          说明                           |
| :--------------------------------------------: | :---------------------------------------: | :-----------------------------------------------------: |
|           lock tables xxx read/write           | SHARED\_READ\_ONLY/SHARED\_NO\_READ_WRITE |                                                         |
|      select 、select … lock in share mode      |                SHARED_READ                | 与 SHARED\_READ 、SHARED\_WRITE 兼容，与 EXCLUSIVE 互斥 |
| insert 、update 、delete 、select … for update |               SHARED_WRITE                | 与 SHARED\_READ 、SHARED\_WRITE 兼容，与 EXCLUSIVE 互斥 |
|                 alter table …                  |                 EXCLUSIVE                 |                   与其它的 MDL 都互斥                   |

查看元数据锁：

```mysql
select object_type,object_schema,object_name,lock_type,lock_duration from performance_schema.metadata_locks;
```



#### 5.3.4、意向锁

为了避免 `DML` 在执行中，加的行锁与表锁的冲突，在 `InnoDB` 中引入了意向锁，使得表锁不用检查没行数据是否加锁，使用意向锁来减少表锁的检查。

1.  意向共享锁（`IS`）：由语句 `select ... lock in share mode` 添加。与表锁共享锁（`read`）兼容，与表锁排他锁（`write`）互斥。
2.  意向排他锁（`IX`）：由 `insert 、update 、delete 、select ... for update` 添加。与表锁共享锁（`read`）及与排他锁（`write`）都互斥。意向锁之间不会互斥。

可以通过一下 `SQL` ，查看意向锁及行锁的加锁情况：

```mysql
select object_schema,object_name,index_name,lock_type,lock_mod,lock_data from performance_schema.data locks;
```



### 5.4、行级锁

#### 5.4.1、介绍

行级锁，每次操作锁住对应的行数据。行定粒度最小，发生锁冲突的概率最低，并发度最高。应用在 `InnoDB` 存储引擎中。

`InnoDB` 的数据是基于索引组织，行锁是通过对索引上的索引项加锁来实现的，并不是对记录加的锁。对于行级锁，主要分为以下三类：

1.  行锁（`Record Lock`）：锁定单个行记录的锁，防止其他事物对此进行 `update` 和 `delete` 。在 `RC` 、`RR` 隔离级别下都支持。
2.  间隙锁（`Gap Lock`）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，防止其他事物在这个间隙进行 `insert` ，产生幻读。在 `RR` 隔离级别下都支持。
3.  临键锁（`Next-Key Lock`）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙 `Gap` 。在 `RR` 隔离级别下支持。

#### 5.4.2、行锁

`InnoDB` 实现了以下两种类型的行锁：

1.  共享锁（`S`）：允许一个事物去读一行，阻止其他事物获得相同数据集的排他锁。
2.  排他锁（`X`）：允许获取排他锁的事物更新数据，阻止其他事物获得相同数据集的共享锁和排他锁。

| 当前锁类型\\请求所类型 | S（共享锁） | X（排他锁） |
| :--------------------: | :---------: | :---------: |
|      S（共享锁）       |    兼容     |    冲突     |
|      X（排他锁）       |    冲突     |    冲突     |

| SQL                         | 行锁类型   | 说明                                        |
| --------------------------- | ---------- | ------------------------------------------- |
| INSERT …                    | 排他锁     | 自动加锁                                    |
| UPDATE …                    | 排他锁     | 自动加锁                                    |
| DELETE …                    | 排他锁     | 自动加锁                                    |
| SELECT（正常）              | 不加任何锁 |                                             |
| SELECT … LOCK IN SHARE MODE | 共享锁     | 需要手动在 SELECT 之后加 LOCK IN SHARE MODE |
| SELECT … FOR UPDATE         | 排他锁     | 需要手动在 SELECT 之后 FOR UPDATE           |

默认情况下，`InnoDB` 在 `REPEATEABLE READ` 事物隔离级别运行，`InnoDB` 使用 `next_key` 锁进行搜索和索引扫描，以防止幻读。

1.  针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。
2.  `InnoDB` 的行锁是针对于索引加的锁，不通过索引条件检索数据，那么 `InnoDB` 将对表中的所有记录加锁，此时升级为表锁。

可以通过以下 `SQL` ，查看意向锁及行锁的加锁情况：

```mysql
select object_schema,object_name,index_name,lock_type,lock_mod,lock_data from performance_schema.data_locks;
```



#### 5.4.3、间隙锁/临键锁

默认情况下，`InnoDB` 在 `REPEATABLE READ` 事务隔离级别运行，`InnoDB` 使用 `next_key` 锁进行搜索和索引扫描，以防止幻读。

1.  索引上的等值查询（唯一索引），给不存在的记录加锁时，优化为间隙锁。
2.  索引上的等值查询（普通索引），向右遍历时最后一个值不满足查询需求时，`next-key lock` 退化为间隙锁。
3.  索引上的范围查询（唯一索引）–会访问到不满足条件的第一个值为止。

> **注意：** 间隙锁唯一目的是防止其他事物插入间隙。间隙锁可以共存，一个事物采用的间隙锁不会阻止另一个事务在同一间隙锁上采用间隙锁。





## 6、InnoDB引擎

### 6.1、逻辑存储结构

*   表空间（`ibd` 文件）：一个 `MySQL` 实例可以对应多个表空间，用于存储记录、索引等数据。
*   段：分为数据段（`Leaf node segment`）、索引段（`Non-leaf node segment`）、回滚段（`Rollback segment`），`InnoDB` 是索引组织表，数据段就是 `B+` 树的叶子结点，索引段就是 `B+` 树的非叶子结点。段用来当管理多个 `Extent`（区） 。
*   区：表空间的单元结构，每个区的大小为 `1M` 。默认情况下，`InnoDB` 存储引擎页大小为 `16K` ，即一个区中一共有 `64` 个连续的页。
*   页：是 `InnoDB` 存储引擎磁盘管理的最小单元，每个页的大小默认为 `16KB` 。为了保证页的连续性，`InnoDB` 存储引擎每次从磁盘申请 `4-5` 个区。
*   行：`InnoDB` 存储引擎数据就是按行进行存放的。

![image-20241112215519764](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122155060.png)



### 6.2、架构

`MySQL5.5` 版本开始，默认使用 `InnoDB` 存储引擎，它擅长事务处理，具有崩溃恢复特性，在日常开发中使用非常广泛。下面是 `InnoDB` 架构图，左侧为内存结构，右侧为磁盘结构。

![image-20241112215541336](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122155609.png)

#### 6.2.1、内存结构

![image-20241112215550856](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122155112.png)

![image-20241112215605734](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122156028.png)

![image-20241112215623460](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122156712.png)

![image-20241112215633314](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122156586.png)

#### 6.2.2、磁盘结构

![image-20241112215642569](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122156855.png)

![image-20241112215654182](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122156457.png)

![image-20241112215702489](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122157750.png)

#### 6.2.3、后台线程

![image-20241112215711216](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122157474.png)



### 6.3、事务原理

#### 6.3.1、事务

事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。



#### 6.3.2、ACID特性

*   原子性：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
*   一致性：事务完成时，必须使所有的数据都保持一致状态。
*   隔离性：数据库提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下与运行。
*   持久性：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

![image-20241112215719469](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122157738.png)



#### 6.3.3、redo log（持久性）

重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的永久性。

该日志文件由两部分组成：重做日志缓冲（`redo log buffer`）以及重做日志文件（`redo log file`），前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到该日志文件中，用于在刷新脏页到磁盘，发生错误时，进行数据恢复使用。

![image-20241112215727490](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122157752.png)

#### 6.3.4、undo log（原子性）

回滚日志，用于记录数据被修改前的信息，作用包含两个：提供回滚和 `MVCC`（多版本并发控制）。

`undo log` 和 `redo log` 记录物理日志不一样，它是逻辑日志。可以认为当 `delete` 一条记录时，`undo log` 中会记录一条对应的 `insert` 记录，反之亦然，当 `update` 一条记录时，它记录一条对应相反的 `update` 记录。当执行 `rollback` 时，就可以从 `undo log` 中的逻辑记录读取到相应的内容并进行回滚。

`Undo log` 销毁：`undo log` 在事务执行时产生，事务提交时，并不会立即删除 `undo log` ，因为这些日志可能还用与 `MVCC` 。

`Undo log` 存储：`undo log` 采用段的方式进行管理和记录，存放在前面介绍的 `rollback segment` 回滚段中，内部包含 `1024` 个 `undo log segment` 。



### 6.4、MVCC

#### 6.4.1、基本概念

* 当前读

  读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如：`select ... lock in share mode`（共享锁），`select ... for update、update、insert、delete`（排他锁）都是一种当前读。

* 快照读

  简单的 `select`（不加锁）就是快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。

  *   `Read Committed` ：每次 `select` ，都生成一个快照读。
  *   `Repeatable Read` ：开启事务后第一个 `select` 语句才是快照读的地方。
  *   `Serializable` ：快照读会退化成当前读。

* `MVCC`

  全称 `Multi-Version Concurrency Control` ，多版本并发控制。指维护一个数据的多版本，使得读写操作没有冲突，快照读为 `MySQL` 实现 `MVCC` 提供了一个非阻塞读功能。`MVCC` 的具体实现，还需要依赖于数据库记录中的三个隐式字段、`undo log` 日志和 `readView` 。

#### 6.4.2、实现原理

##### 6.4.2.1、记录中的隐藏字段

|   隐藏字段    |                             含义                             |
| :-----------: | :----------------------------------------------------------: |
|  DB\_TRX\_ID  | 最近修改事务 ID ，记录插入这条记录或最后一次修改该记录的事务 ID |
| DB\_ROLL\_PTR | 回滚指针，指向这条记录的上一个版本，用于配合 undo log ，指向上一个版本 |
|  DB\_ROW\_ID  |     隐藏主键，如果表结构没有指定主键，将会生成该隐藏字段     |

![image-20241112215738295](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122157551.png)

##### 6.4.2.2、undo log

回滚日志，在 `insert、update、delete` 的时候产生的便于数据回滚的日志。

当 `insert` 的时候，产生的 `undo log` 日志只在回滚时需要，在事务提交后，可被立即删除。

而 `update、delete` 的时候，产生的 `undo log` 日志不仅在回滚时需要，在快照读时也需要，不会立即被删除。

##### 6.4.2.3、undo log 版本链

不同事物或相同事物对同一条记录进行修改，会导致该记录的 `undolog` 生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122157472.png" alt="image-20241112215745212" style="zoom: 67%;" />
<img src="C:/Users/raosirui/AppData/Roaming/Typora/typora-user-images/image-20241112215756369.png" alt="image-20241112215756369" style="zoom:50%;" />

##### 6.4.2.4、readview

`ReadView`（读视图）是快照读 `SQL` 执行时 `MVCC` 提取数据的依据，记录并维护系统当前活跃的事务（未提交的）`id` 。

`ReadView` 中包含了四个核心字段：

| 字段             | 含义                                                      |
| ---------------- | --------------------------------------------------------- |
| m_ids            | 当前活跃的事务 ID 集合                                    |
| min\_trx\_id     | 最小活跃事务 ID                                           |
| max\_trx\_id     | 预分配事务 ID ，当前最大事务 ID+1（因为事务 ID 是自增的） |
| creator\_trx\_id | ReadView 创建者的事务 ID                                  |

`trx_id` ：代表是当前事务 `ID` 。

![image-20241112215818499](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122158752.png)

不同隔离级别，生成 `ReadView` 的时机不同：

*   `READ COMMITTED` ：在事务中每一次执行快照读时生成的 `ReadView` 。
*   `REPEATABLE READ` ：仅在事务中第一次执行快照读时生成 `ReadView` ，后续复用该 `ReadView` 。

`RC` 隔离级别下，在事务中每一次执行快照读生成 `ReadView` 。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122159766.png" alt="image-20241112215904500" style="zoom:80%;" />

`RR` 隔离级别下，仅在事务中第一次执行快照读生成 `ReadView` ，后续复用该 `ReadView` 。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122158297.png" alt="image-20241112215839034" style="zoom:80%;" />

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411122159303.png" alt="image-20241112215914035" style="zoom: 67%;" />





## 7、MySQL管理

### 7.1、系统数据库

`MySQL` 数据库安装完成后，自带了以下四个数据库，具体作用如下：

| 数据库             | 含义                                                         |
| ------------------ | ------------------------------------------------------------ |
| mysql              | 存储 MySQL 服务器正常运行所需要的各种信息（时区、主从、用户、权限等） |
| information_schema | 提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类型及访问权限等 |
| performance_schema | 为 MySQL 服务器运行时状态提供了一个底层监控功能，主要用于收集数据库服务器性能和参数 |
| sys                | 包含了一系列方便 DBA 和开发人员利用 performance_schma 性能数据库进行性能调优和诊断的视图 |



### 7.2、常用工具

#### 7.2.1、MySQL

该 `MySQL` 不是指 `MySQL` 服务，而是指 `MySQL` 的客户端工具。

```shell
语法：
    mysql [options][database]
选项：
    -u, --user=name			#指定用户名
    -p, --password[=name]	#指定密码
    -h, --host=name			#指定服务器IP或域名
    -P, --port=port			#指定连接端口
    -e, --execute=name		#执行SQL语句并退出
```


`-e` 选项可以在 `MySQL` 客户端执行 `SQL` 语句，而不用连接到 `MySQL` 数据库再执行，对于一些批处理脚本，这种方式尤其方便。

```shell
示例：
	mysql -h192.168.200.202 -P3306 -uroot -p1234 itcast -e "select * from stu"
```

#### 7.2.2、mysqladmin

`mysqladmin` 是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。

```shell
通过帮助文档查看选项：
	mysqladmin --help
示例：
	mysqladmin -uroot -p123456 drop 'test01';
	musqladmin -uroot -p123456 version;
```

#### 7.2.3、mysqlbinlog

由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到 `mysqlbinlog` 日志管理工具。

```shell
语法：
	mysqlbinlog [options] log-files1 log-files2 ...
选项：
	-d, --database=name		#指定数据库名称，只列出指定的数据库相关操作
	-o, --offset=#			#忽略掉日志中的前n行命令
	-r, --result-file=name	#将输出的文本格式日志输出到指定文件
	-s, --short-form		#显示简单个格式，省略掉一些信息
	--start-datatime=date1 --stop-datetime=date2	#指定日期间隔内的所有日志
	--start-postion=pos1 --stop-postion=pos2		#指定位置间隔内的所有日志
```

#### 7.2.4、mysqlshow

`mysqlshow` 客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或索引。

```shell
语法：
	mysqlshow [options][db_name[table_name[col_name]]]
选项：
	--count		#显示数据库及表的统计信息（数据库，表均可以不指定）
	-i			#显示指定数据库或者指定表的状态信息
示例：
	#查询每个数据库的表的数量及表中记录的数量
	mysqlshow -uroot -p2143 --count
	
	#查询test库中每个表中的字段数，及行数
	mysqlshow -uroot -p2143 --count
	
	#查询test库中book表的详细情况
	mysqlshow -uroot -p2143 test book --count
```

#### 7.2.5、mysqldump

`mysqldump` 客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的 `SQL` 语句。

```shell
语法：
	mysqldump [options] db_name [tables]
	mysqldump [options] --database/-B db1 [db2 db3 ...]
	mysqldump [options] -all-databases/-A
连接选项：
	-u, --user=name			#指定用户名
	-p, -password[=name]	#指定密码
	-h, --host=name			#指定服务器ip或域名
	-P, --port=#			#指定连接端口
输出选项：
	--add-drop-database		#在每个数据库创建语句前加上drop database语句
	--add-drop-table		#在每个表创建语句前加上rop table语句，默认开启；不开启(--skip-add-drop-table)
	-n, --no-create-db		#不包含数据库的创建语句
	-t, --no-create-info	#不包含数据库的创建语句
	-d, --no-dat			#不包含数据
	-T, --tab=name			#自动生成两个文件：一个.sql文件，创建表结构的语句；一个.txt文件，数据文件
```

#### 7.2.6、mysqlimport/souce

`mysqlimport` 是客户端数据导入工具，用来导入 `mysqldump` 加 `-T` 参数后导出的文本文件。

```shell
语法：
	mysqlimport [options] db_name testfile1 [textfile2...]
示例：
	mysqlimport -uroot -p2143 test tmp/city.txt
```


如果需要导入 `sql` 文件，可以使用 `mysql` 中的 `source` 指令：

```shell
语法：
	source /root/xxxxx.sql
```
