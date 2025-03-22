# MySQL

## MySQL 基础

### 说一下数据库的三大范式？

三大范式的作用是为了减少数据冗余，提高数据完整性。

[![三分恶面渣逆袭：数据库三范式](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412192038665.jpeg)](https://camo.githubusercontent.com/2728eb8cdd78f92531014969f087d2d9efcec784b5f38892fc8e12d7c54886f5/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d31366537346136622d613432612d343634652d396231302d3032353265653765636336652e6a7067)

①、第一范式：**确保表的每一列都是不可分割的基本数据单元**，比如说用户地址，应该拆分成省、市、区、详细信息等 4 个字段。

[![Ruthless：第一范式](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412192039814.png)](https://camo.githubusercontent.com/7039e0faf9405e4abb39489b6c9a234055f3f494c9abb98e819368e06280a37a/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303431383039333233352e706e67)

②、第二范式：要求**表中的每一列都和主键直接相关，而不能只与主键的某一部分相关。**

比如在一个订单表中，可能会存在订单编号和商品编号。

[![Ruthless：不符合第二范式](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412192039654.png)](https://camo.githubusercontent.com/c87339390f44b8bb2e408552c447694063b6f9c3402eb21c0970f2ea909575ed/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303431383039333335312e706e67)

这个订单表中就存在冗余数据，比如说商品名称、单位、商品价格等，应该将其拆分为订单表、订单商品关联表、商品表。

③、第三范式：非主键列应该只依赖于主键列，不依赖于其他非主键列。

比如说在设计订单信息表的时候，可以把客户名称、所属公司、联系方式等信息拆分到客户信息表中，然后在订单信息表中用客户编号进行关联。

[![Ruthless：第三范式](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412192040824.png)](https://camo.githubusercontent.com/a019beb05b89f98e1bf92fabba24ba2063c5716029186369d4f8cead00155c72/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303431383039343333322e706e67)





### varchar 与 char 的区别？

[![varchar](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412192048416.jpeg)](https://camo.githubusercontent.com/71f5ef61bdb0f4102030375d0b09f9b69b3715b655cc05955c01eff7f49523b8/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d34306634326435392d613239352d343534332d386130332d3433393235646134643664392e6a7067)

| **特性**       | **CHAR**                                         | **VARCHAR**                                    |
| -------------- | ------------------------------------------------ | ---------------------------------------------- |
| **定义**       | 定长字符串，长度固定                             | 可变长字符串，长度可变                         |
| **存储方式**   | 长度不足时用空格填充到固定长度                   | 按实际数据长度存储，无需填充                   |
| **存取速度**   | 存取速度快（长度固定，效率高，可能快约 50%）     | 存取速度较慢（长度可变，需要额外开销）         |
| **空间利用率** | 空间利用率低，可能浪费存储空间                   | 空间利用率高，仅存储实际数据                   |
| **最大长度**   | 最多能存放 255 个字符，与编码无关                | 最多能存放 65532 个字符                        |
| **适用场景**   | 适用于存储长度固定的数据（如身份证号、邮政编码） | 适用于存储长度变化较大的数据（如姓名、地址等） |



### blob 和 text 有什么区别？

- blob 用于存储**二进制数据**，而 text 用于存储**大字符串**。
- **blob 没有字符集，text 有一个字符集**，并且根据字符集的校对规则对值进行**排序和比较**



### DATETIME 和 TIMESTAMP 的异同？

**相同点**：

1. 两个数据类型存储时间的表现格式一致。均为 `YYYY-MM-DD HH:MM:SS`
2. 两个数据类型都包含「日期」和「时间」部分。
3. 两个数据类型都可以存储微秒的小数秒（秒后 6 位小数秒）

**区别**：

| **特性**     | **DATETIME**                                                 | **TIMESTAMP**                                                |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **日期范围** | `1000-01-01 00:00:00.000000` 到 `9999-12-31 23:59:59.999999` | `1970-01-01 00:00:01.000000` UTC 到 `2038-01-09 03:14:07.999999` UTC |
| **存储空间** | 8 字节                                                       | 4 字节                                                       |
| **时区相关** | 与时区无关                                                   | 与时区有关，存储的是 UTC 时间，显示时会转换为当前时区        |
| **默认值**   | 默认值为 `NULL`                                              | 默认值为当前时间（`CURRENT_TIMESTAMP`），默认非空            |
| **使用场景** | 适合表示独立于时区的时间（如日志、计划等）                   | 适合需要与时区相关的时间（如记录事件的精确时间戳）           |



### in 和 exists 的区别？

MySQL 中的 **in 语句是把外表和内表作 hash 连接，而 exists 语句是对外表作 loop 循环，每次 loop 循环再对内表进行查询。**我们可能认为 exists 比 in 语句的效率要高，这种说法其实是不准确的，要区分情景：

1. 如果查询的两个表大小相当，那么用 in 和 exists 差别不大。
2. 如果两个表中一个较小，一个是大表，则**子查询表大的用 exists，子查询表小的用 in**。
3. not in 和 not exists：如果查询语句使用了 not in，那么内外表都进行全表扫描，没有用到索引；而 not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用 not exists 都比 not in 要快。



### 记录货币用什么字段类型比较好？

货币在数据库中 MySQL 常用 **Decimal 和 Numeric** 类型表示，这两种类型**被 MySQL 实现为同样的类型**。他们被用于保存与货币有关的数据。

例如 salary DECIMAL(9,2)，9(precision)代表将被用于存储值的总的小数位数，而 2(scale)代表将被用于存储小数点后的位数。存储在 salary 列中的值的范围是从-9999999.99 到 9999999.99。

DECIMAL 和 NUMERIC 值**作为字符串存储**，而不是作为二进制浮点数，以便保存那些值的小数精度。

**之所以不使用 float 或者 double 的原因：因为 float 和 double 是以二进制存储的，所以有一定的误差。**



### 怎么存储 emoji?

MySQL 的 utf8 字符集仅支持最多 3 个字节的 UTF-8 字符，但是 emoji 表情（😊）是 4 个字节的 UTF-8 字符，所以在 MySQL 中存储 emoji 表情时，需要使用 utf8mb4 字符集。

MySQL 8.0 已经默认支持 utf8mb4 字符集





### drop、delete 与 truncate 的区别？

三者都表示删除，但是三者有一些差别：

| 区别     | delete                                       | truncate                           | drop                                                   |
| -------- | -------------------------------------------- | ---------------------------------- | ------------------------------------------------------ |
| 类型     | 属于 DML                                     | 属于 DDL                           | 属于 DDL                                               |
| 回滚     | **可回滚**                                   | **不可回滚**                       | **不可回滚**                                           |
| 删除内容 | **表结构还在，删除表的全部或者一部分数据行** | **表结构还在，删除表中的所有数据** | **从数据库中删除表，所有数据行，索引和权限也会被删除** |
| 删除速度 | 删除速度慢，需要逐行删除                     | 删除速度快                         | 删除速度最快                                           |

因此，**在不再需要一张表的时候，用 drop；在想删除部分数据行时候，用 delete；在保留表而删除所有数据的时候用 truncate。**



### UNION 与 UNION ALL 的区别？

| **特性**     | **UNION**                                      | **UNION ALL**                      |
| ------------ | ---------------------------------------------- | ---------------------------------- |
| **重复数据** | 默认会对结果进行去重，**移除重复行**           | **不去重，保留所有重复行**         |
| **性能**     | 因为需要去重，所以性能较低                     | 因为不需要去重，所以性能更高       |
| **使用场景** | 适用于需要合并且确保结果集中无重复数据的情况   | 适用于需要合并并保留所有数据的情况 |
| **去重机制** | 使用 `DISTINCT` 对结果集去重，增加额外计算开销 | 不执行去重操作                     |



### count(1)、count(*) 与 count(列名) 的区别？

![三种计数方式](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201103678.jpeg)

| **用法**        | **描述**                                                   | **计数对象**                     | **性能**                 | **特殊情况**                     |
| --------------- | ---------------------------------------------------------- | -------------------------------- | ------------------------ | -------------------------------- |
| **COUNT(1)**    | 计算表中所有行的数量，`1` 是常量，结果与 `COUNT(*)` 一致。 | 所有行，不论列是否有 `NULL`。    | 与 `COUNT(*)` 性能相当。 | 不依赖任何具体列的值。           |
| **COUNT(\*)**   | 计算表中所有行的数量，包括 `NULL` 行。                     | 所有行，不论列是否有 `NULL`。    | 最常用且优化程度最高。   | 表示统计所有记录，不依赖特定列。 |
| **COUNT(列名)** | 计算指定列中非 `NULL` 值的数量。                           | 仅统计指定列中不为 `NULL` 的行。 | 取决于索引和优化程度。   | 结果忽略该列中 `NULL` 的行。     |

假设有如下表 `students`：

| id   | name  | age  |
| ---- | ----- | ---- |
| 1    | Alice | 20   |
| 2    | Bob   | NULL |
| 3    | NULL  | 22   |
| 4    | Carol | NULL |

- **COUNT(1)**：结果为 `4`（所有行都计入统计）。
- **COUNT(\*)**：结果为 `4`（所有行都计入统计）。
- **COUNT(name)**：结果为 `2`（忽略 `name` 列中 `NULL` 的行）。



### SQL 查询语句的执行顺序了解吗？

![image-20250301085139264](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202503010851480.png)



[![查询语句执行顺序](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201108210.jpeg)](https://camo.githubusercontent.com/0b0295f7eea8629e892cc2713d432fe9f76ef3713f4e229688d7c36273690119/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d34376464656139322d636638662d343963342d616232652d3639613832396666316265322e6a7067)

1. **FROM**：对 FROM 子句中的左表<left_table>和右表<right_table>执行笛卡儿积（Cartesianproduct），产生虚拟表 VT1
2. **ON**：对虚拟表 VT1 应用 ON 筛选，只有那些符合<join_condition>的行才被插入虚拟表 VT2 中
3. **JOIN**：如果指定了 OUTER JOIN（如 LEFT OUTER JOIN、RIGHT OUTER JOIN），那么保留表中未匹配的行作为外部行添加到虚拟表 VT2 中，产生虚拟表 VT3。如果 FROM 子句包含两个以上表，则对上一个连接生成的结果表 VT3 和下一个表重复执行步骤 1）～步骤 3），直到处理完所有的表为止
4. **WHERE**：对虚拟表 VT3 应用 WHERE 过滤条件，只有符合<where_condition>的记录才被插入虚拟表 VT4 中
5. **GROUP BY**：根据 GROUP BY 子句中的列，对 VT4 中的记录进行分组操作，产生 VT5
6. **CUBE|ROLLUP**：对表 VT5 进行 CUBE 或 ROLLUP 操作，产生表 VT6
7. **HAVING**：对虚拟表 VT6 应用 HAVING 过滤器，只有符合<having_condition>的记录才被插入虚拟表 VT7 中。
8. **SELECT**：第二次执行 SELECT 操作，选择指定的列，插入到虚拟表 VT8 中
9. **DISTINCT**：去除重复数据，产生虚拟表 VT9
10. **ORDER BY**：将虚拟表 VT9 中的记录按照<order_by_list>进行排序操作，产生虚拟表 VT10。11）
11. **LIMIT**：取出指定行的记录，产生虚拟表 VT11，并返回给查询用户



### 介绍一下 MySQL 的常用命令

#### 说说数据库操作命令？

①、**创建数据库**:

```mysql
CREATE DATABASE database_name;
```

②、**删除数据库**:

```mysql
DROP DATABASE database_name;
```

③、**选择数据库**:

```mysql
USE database_name;
```



#### 说说表操作命令？

①、**创建表**:

```mysql
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    ...
);
```

②、**删除表**:

```mysql
DROP TABLE table_name;
```

③、**显示所有表**:

```mysql
SHOW TABLES;
```

④、**查看表结构**:

```mysql
DESCRIBE table_name;
```

⑤、**修改表**（添加列）:

```mysql
ALTER TABLE table_name ADD column_name datatype;
```



#### 说说 CRUD 命令？

①、**插入数据**:

```mysql
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...);
```

②、**查询数据**:

```mysql
SELECT column_names FROM table_name WHERE condition;
```

③、**更新数据**:

```mysql
UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;
```

④、**删除数据**:

```mysql
DELETE FROM table_name WHERE condition;
```



#### 说说索引和约束的创建修改命令？

①、**创建索引**:

```mysql
CREATE INDEX index_name ON table_name (column_name);
```

②、**添加主键约束**:

```mysql
ALTER TABLE table_name ADD PRIMARY KEY (column_name);
```

③、**添加外键约束**:

```mysql
ALTER TABLE table_name ADD CONSTRAINT fk_name FOREIGN KEY (column_name) REFERENCES parent_table (parent_column_name);
```



#### 说说用户和权限管理的命令？

①、**创建用户**:

```mysql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

②、**授予权限**:

```mysql
GRANT ALL PRIVILEGES ON database_name.table_name TO 'username'@'host';
```

③、**撤销权限**:

```mysql
REVOKE ALL PRIVILEGES ON database_name.table_name FROM 'username'@'host';
```

④、**删除用户**:

```mysql
DROP USER 'username'@'host';
```



#### 说说事务控制的命令？

①、**开始事务**:

```mysql
START TRANSACTION;
```

②、**提交事务**:

```mysql
COMMIT;
```

③、**回滚事务**:

```mysql
ROLLBACK;
```





### MySQL bin 目录下的可执行文件了解吗？

- mysql：客户端程序，用于连接 MySQL 服务器
- mysqldump：一个非常实用的 MySQL 数据库**备份工具**，用于创建一个或多个 MySQL 数据库级别的 SQL 转储文件，包括数据库的表结构和数据。对数据备份、迁移或恢复非常重要。
- mysqladmin：mysql 后面加上 admin 就表明这是一个 MySQL 的**管理工具**，它可以用来执行一些管理操作，比如说创建数据库、删除数据库、查看 MySQL 服务器的状态等。
- mysqlcheck：mysqlcheck 是 MySQL 提供的一个**命令行工具**，用于检查、修复、分析和优化数据库表，对数据库的维护和性能优化非常有用。
- mysqlimport：用于**从文本文件中导入数据**到数据库表中，非常适合用于批量导入数据。
- mysqlshow：用于**显示** MySQL 数据库服务器中的数据库、表、列等信息。
- mysqlbinlog：用于查看 MySQL **二进制日志文件的内容**，可以用于恢复数据、查看数据变更等。





### MySQL 第 3-10 条记录怎么查？

在 MySQL 中，要查询第 3 到第 10 条记录，可以使用 **limit** 语句，结合**偏移量 offset 和行数 row_count** 来实现。

limit 语句用于限制查询结果的数量，偏移量表示从哪条记录开始，行数表示返回的记录数量。

```mysql
SELECT * FROM table_name LIMIT 2, 8;
```

- 2：偏移量，表示跳过前两条记录，从第三条记录开始。
- 8：行数，表示从偏移量开始，返回 8 条记录。

偏移量是从 0 开始的，即第一条记录的偏移量是 0；如果想从第 3 条记录开始，偏移量就应该是 2。





### 用过哪些 MySQL 函数？

#### 用过哪些字符串函数来处理文本？

```mysql
-- 连接字符串
SELECT CONCAT('沉默', ' ', '王二') AS concatenated_string;

-- 获取字符串长度
SELECT LENGTH('沉默 王二') AS string_length;

-- 提取子字符串
SELECT SUBSTRING('沉默 王二', 1, 5) AS substring;

-- 替换字符串内容
SELECT REPLACE('沉默 王二', '王二', 'MySQL') AS replaced_string;

-- 字符串转小写
SELECT LOWER('HELLO WORLD') AS lower_case;

-- 字符串转大写
SELECT UPPER('hello world') AS upper_case;

-- 去除字符串两侧的空格
SELECT TRIM('  沉默 王二  ') AS trimmed_string;
```

#### 用过哪些数值函数？

```mysql
-- 返回绝对值
SELECT ABS(-123) AS absolute_value;

-- 向上取整
SELECT CEILING(123.45) AS ceiling_value;

-- 向下取整
SELECT FLOOR(123.45) AS floor_value;

-- 四舍五入
SELECT ROUND(123.4567, 2) AS rounded_value;

-- 余数
SELECT MOD(10, 3) AS modulus;
```

#### 用过哪些日期和时间函数？

```mysql
-- 返回当前日期和时间
SELECT NOW() AS current_date_time;

-- 返回当前日期
SELECT CURDATE() AS current_date;

-- 返回当前时间
SELECT CURTIME() AS current_time;

-- 在日期上添加天数
SELECT DATE_ADD(CURDATE(), INTERVAL 10 DAY) AS date_in_future;

-- 计算两个日期之间的天数
SELECT DATEDIFF('2024-12-31', '2024-01-01') AS days_difference;

-- 返回日期的年份
SELECT YEAR(CURDATE()) AS current_year;
```

#### 用过哪些汇总函数？

```mysql
-- 创建一个表并插入数据进行聚合查询
CREATE TABLE sales (
    product_id INT,
    sales_amount DECIMAL(10, 2)
);

INSERT INTO sales (product_id, sales_amount) VALUES (1, 100.00);
INSERT INTO sales (product_id, sales_amount) VALUES (1, 150.00);
INSERT INTO sales (product_id, sales_amount) VALUES (2, 200.00);

-- 计算总和
SELECT SUM(sales_amount) AS total_sales FROM sales;

-- 计算平均值
SELECT AVG(sales_amount) AS average_sales FROM sales;

-- 计算总行数
SELECT COUNT(*) AS total_entries FROM sales;

-- 最大值和最小值
SELECT MAX(sales_amount) AS max_sale, MIN(sales_amount) AS min_sale FROM sales;
```

#### 用过哪些逻辑函数？

```mysql
-- IF函数
SELECT IF(1 > 0, 'True', 'False') AS simple_if;

-- CASE表达式
SELECT CASE WHEN 1 > 0 THEN 'True' ELSE 'False' END AS case_expression;

-- COALESCE函数
SELECT COALESCE(NULL, NULL, 'First Non-Null Value', 'Second Non-Null Value') AS first_non_null;
```

#### 用过哪些格式化函数？

```mysql
-- 格式化数字
SELECT FORMAT(1234567.8945, 2) AS formatted_number;
```

#### 用过哪些类型转换函数？

```mysql
-- CAST函数
SELECT CAST('2024-01-01' AS DATE) AS casted_date;

-- CONVERT函数
SELECT CONVERT('123', SIGNED INTEGER) AS converted_number;
```





### 说说 SQL 的隐式数据类型转换？

数据类型隐式转换会导致意想不到的结果，所以要尽量避免隐式转换。

可以通过显式转换来规避这种情况。

```mysql
SELECT CAST('1' AS SIGNED INTEGER) + 1; -- 结果为 2
```





### 说说 SQL 的语法树解析？

语法树（或抽象语法树，AST）是 SQL 解析过程中的中间表示，它使用树形结构表示 SQL 语句的层次和逻辑。语法树由节点（Node）组成，每个节点表示 SQL 语句中的一个语法元素。

- **根节点**：通常是 SQL 语句的主要操作，例如 SELECT、INSERT、UPDATE、DELETE 等。
- **内部节点**：表示语句中的操作符、子查询、连接操作等。例如，WHERE 子句、JOIN 操作等。
- **叶子节点**：表示具体的标识符、常量、列名、表名等。例如，users 表、id 列、常量 1 等。

以一个简单的 SQL 查询语句为例：

```mysql
SELECT name, age FROM users WHERE age > 18;
```

这个查询语句的语法树可以表示为：

```mysql
          SELECT
         /      \
     Columns     FROM
    /      \      |
  name      age  users
               |
             WHERE
               |
            age > 18
```

根节点：SELECT，表示这是一个查询操作。

子节点：Columns 和 FROM。

- Columns 子树表示查询的列，包含两个叶子节点：name 和 age。
- FROM 子树表示查询的数据源，包含一个叶子节点：users 表。

条件节点：WHERE 子树表示查询条件，包含条件表达式 `age > 18`。





## 数据库架构

### 说说 MySQL 的基础架构？

MySQL 的架构大致可以分为三层，从上到下依次是：连接层、服务层、和存储引擎层。

[![三分恶面渣逆袭：Redis 的基础架构](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201140289.jpeg)](https://camo.githubusercontent.com/12e877d5ef201ef5faa0e3a6db8ab27fb8e777c2d109f3d8e05d4ed700e83abe/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d37373632366664622d643262302d343235362d613438332d6431633630653638643865632e6a7067)

①、连接层主要负责客户端连接的管理，包括**验证用户身份、权限校验、连接管理**等。可以通过数据库连接池来提升连接的处理效率。

②、服务层是 MySQL 的核心，主要负责**查询解析、优化、执行**等操作。在这一层，SQL 语句会经过解析、优化器优化，然后转发到存储引擎执行，并返回结果。这一层包含查询解析器、优化器、执行计划生成器、缓存（如查询缓存）、日志模块等。

③、存储引擎层负责数据的**实际存储和提取**，是 MySQL 架构中与数据交互最直接的层。MySQL 支持多种存储引擎，如 InnoDB、MyISAM、Memory 等。



#### binlog写入在哪一层？

binlog 在服务层，负责记录 SQL 语句的变化。它记录了所有对数据库进行更改的操作，用于数据恢复、主从复制等。





### :star:一条查询语句如何执行？

![二哥的 Java 进阶之路：SQL 执行](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201141810.png)

第一步，客户端发送 SQL 查询语句到 MySQL 服务器。

第二步，MySQL 服务器的连接器开始处理这个请求，跟客户端建立连接、获取权限、管理连接。

~~第三步（MySQL 8.0 以后已经干掉了），连接建立后，MySQL 服务器的查询缓存组件会检查是否有缓存的查询结果。如果有，直接返回给客户端；如果没有，进入下一步~~。

第三步，解析器**对 SQL 语句进行解析**，检查语句是否符合 SQL 语法规则，确保引用的数据库、表和列都是存在的，并处理 SQL 语句中的名称解析和权限验证。

第四步，优化器负责**确定 SQL 语句的执行计划**，这包括选择使用哪些索引，以及决定表之间的连接顺序等。

第五步，执行器会**调用存储引擎的 API** 来进行数据的读写。

第六步，MySQL 的存储引擎是插件式的，不同的存储引擎在细节上面有很大不同。例如，InnoDB 是支持事务的，而 MyISAM 是不支持的。之后，会将执行结果返回给客户端

第七步，客户端接收到查询结果，完成这次查询请求。





### 一条更新语句怎么执行的？

更新语句的执行是 Server 层和引擎层配合完成，数据除了要写入表中，还要记录相应的日志。

[![update 执行](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201149095.jpeg)](https://camo.githubusercontent.com/6c0ba6ee8e5c0e510617163a7578797d2febf318740b1b0bcf3dbf7ccb36a131/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d38313266623033382d333964652d343230342d616339662d3933643862373434386131382e6a7067)





### 说说 MySQL 的数据存储形式

MySQL 是以表的形式存储数据的，而表空间的结构则由段、区、页、行组成。

[![不要迷恋发哥：段、区、页、行](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201150049.png)](https://camo.githubusercontent.com/7eff7ce542bb93b729c84cdccfc05dcb140158d71a9c9842d1508eb540e21e26/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303531353131303033342e706e67)

①、段（Segment）：表空间由多个段组成，常见的段有**数据段、索引段、回滚段**等。

创建索引时会创建两个段，数据段和索引段，数据段用来存储叶子阶段中的数据；索引段用来存储非叶子节点的数据。

回滚段包含了事务执行过程中用于数据回滚的旧数据。

②、区（Extent）：段由一个或多个区组成，区是一组连续的页，通常**包含 64 个连续的页**，也就是 1M 的数据。

使用区而非单独的页进行数据分配可以优化磁盘操作，减少磁盘寻道时间，特别是在大量数据进行读写时。

③、页（Page）：页是 InnoDB 存储数据的基本单元，**标准大小为 16 KB**，**索引树上的一个节点就是一个页**。

也就意味着数据库每次读写都是以 16 KB 为单位的，一次最少从磁盘中读取 16KB 的数据到内存，一次最少写入 16KB 的数据到磁盘。

④、行（Row）：InnoDB 采用行存储方式，意味着数据按照行进行组织和管理，行数据可能有多个格式，比如说 COMPACT、REDUNDANT、DYNAMIC 等。



#### MySQL的行溢出机制

**MySQL 8.0 默认的行格式是 DYNAMIC，由COMPACT 演变而来，意味着这些数据如果超过了页内联存储的限制，则会被存储在溢出页中。**





## 存储引擎

![存储引擎](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201203226.png)

我来做一个表格对比：

| 功能          | InnoDB | MyISAM | MEMORY |
| ------------- | ------ | ------ | ------ |
| 支持事务      | Yes    | No     | No     |
| 支持全文索引  | Yes    | Yes    | No     |
| 支持 B+树索引 | Yes    | Yes    | Yes    |
| 支持哈希索引  | Yes    | No     | Yes    |
| 支持外键      | Yes    | No     | No     |

除此之外，我还了解到：

①、MySQL 5.5 之前，默认存储引擎是 MyISAM，5.5 之后是 InnoDB。

②、InnoDB 支持的哈希索引是自适应的，不能人为干预。

③、InnoDB 从 MySQL 5.6 开始，支持全文索引。

④、InnoDB 的最小表空间略小于 10M，最大表空间取决于页面大小（page size）。

[![MySQL 官网：innodb-limits.html](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201203375.png)](https://camo.githubusercontent.com/72857852d535c934ed20ec454e6a955e67528898091c569d267681480b90746e/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303430383037343633302e706e67)

#### 如何切换 MySQL 的数据引擎？

可以通过 alter table 语句来切换 MySQL 的数据引擎。

```mysql
ALTER TABLE your_table_name ENGINE=InnoDB;
```

不过不建议，应该提前设计好到底用哪一种存储引擎。





### 那存储引擎应该怎么选择？

- 大多数情况下，使用默认的 InnoDB 就对了，InnoDB 可以提供**事务、行级锁、外键、B+ 树索引**等能力。
- **MyISAM 适合读更多的场景**。
- **MEMORY 适合临时表，数据量不大**的情况。由于数据都存放在内存，所以速度非常快。





### InnoDB 和 MyISAM 主要有什么区别？

[
![三分恶面渣逆袭：InnoDB 和 MyISAM 主要有什么区别](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201206088.jpeg)](https://camo.githubusercontent.com/94493aaf3bbf6b570e67a4cc1856504601bc1313c8534f09189c0ea2465b9136/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d62376161303430652d613361372d343133332d386334332d6261636363336338643031322e6a7067)

InnoDB 和 MyISAM 之间的区别主要表现在存储结构、事务支持、最小锁粒度、索引类型、主键必需、表的具体行数、外键支持等方面。

| **对比项**       | **MyISAM**                                                   | **InnoDB**                                     |
| ---------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| **存储结构**     | `.frm` 文件存储表定义；`.MYD` 存储数据；`.MYI` 存储索引。    | `.frm` 文件存储表定义；`.ibd` 存储数据和索引。 |
| **事务支持**     | 不支持事务。                                                 | 支持事务。                                     |
| **最小锁粒度**   | 表级锁，高并发写操作性能低。                                 | 行级锁，并发写入性能高。                       |
| **索引类型**     | 非聚簇索引，**索引和数据分开存储**，索引保存的是数据文件的指针。 | 聚簇索引，**索引和数据不分开存储**。           |
| **外键支持**     | 不支持外键。                                                 | 支持外键。                                     |
| **主键要求**     | 可以没有主键。                                               | 必须有主键。                                   |
| **表的具体行数** | 表的具体行数存储在表属性中，**查询时直接返回**。             | 表的具体行数需要**扫描整个表才能返回。**       |

MyISAM 为非聚簇索引，索引和数据分开存储，索引保存的是数据文件的指针。

[![未见初墨：MyIsam](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201208323.png)](https://camo.githubusercontent.com/3a5e67694b16b7100ed8019588f4654f4a20d462cd0041dd94bb3951ca8d7d5a/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303430333133303130342e706e67)

InnoDB 为聚簇索引，索引和数据不分开。

[![yangh124：InnoDB](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201208012.png)](https://camo.githubusercontent.com/4b87aa7f50dc1d834f26bbb931a9f35ea98f391b3ca982e246012dd85493fe6f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303430333133303530382e706e67)



### InnoDB 的 Buffer Pool了解吗？

Buffer Pool 是 InnoDB 存储引擎中的一个**内存缓冲区**，它会**将数据以页（page）的单位保存在内存中**，当查询请求需要读取数据时，优先从 Buffer Pool 获取数据，避免直接访问磁盘。

[![图片来源于网络](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201209676.png)](https://camo.githubusercontent.com/9a747096d57bfb46576207fa51ab4dadf7fa670a12d6a6a74fcd6cb63888eaec/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234313130343230303931352e706e67)

也就是说，即便我们只访问了一行数据的一个字段，InnoDB 也会将整个数据页加载到 Buffer Pool 中，以便后续的查询。

修改数据时，也会先在缓存页面中修改。当数据页被修改后，会在 Buffer Pool 中变为脏页。

脏页不会立刻写回到磁盘。

InnoDB 会**定期将这些脏页刷新到磁盘**，保证数据的一致性。通常采用**改良的 LRU 算法来管理缓存页**，也就是将最近最少使用的数据移出缓存，为新数据腾出空间。

[![极客时间：改良的 LRU 算法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201209335.png)](https://camo.githubusercontent.com/0d9c2e82c89621df201a95a8daa13d6ef3c6a3959394a185532fd504f1a11dd9/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234313130343230323735322e706e67)

Buffer Pool 能够显著**减少对磁盘的访问，从而提升数据库的读写性能**。

在调优方面，我们可以设置合理的 Buffer Pool 大小（通常为物理内存的 70%），并配置多个 Buffer Pool 实例（通过 innodb_buffer_pool_instances）来提升并发能力。此外，还可以通过调整刷新策略参数，比如 innodb_flush_log_at_trx_commit，来平衡性能和数据持久性。





## 日志

### MySQL 日志文件有哪些？分别介绍下作用？

![三分恶面渣逆袭：MySQL的主要日志](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201211452.jpeg)

MySQL 的日志文件主要包括：

①、**错误日志**（Error Log）：记录 MySQL 服务器启动、运行或停止时出现的问题。

②、**慢查询日志**（Slow Query Log）：记录执行时间超过 long_query_time 值的所有 SQL 语句。这个时间值是可配置的，默认情况下，慢查询日志功能是关闭的。可以用来识别和优化慢 SQL。

③、**一般查询日志**（General Query Log）：记录所有 MySQL 服务器的连接信息及所有的 SQL 语句，不论这些语句是否修改了数据。

④、**二进制日志**（Binary Log）：记录了所有修改数据库状态的 SQL 语句，以及每个语句的执行时间，如 INSERT、UPDATE、DELETE 等，但**不包括 SELECT 和 SHOW** 这类的操作。

⑤、**重做日志**（Redo Log）：**记录了对于 InnoDB 表的每个写操作**，不是 SQL 级别的，而是物理级别的，主要用于崩溃恢复。

⑥、**回滚日志**（Undo Log，或者叫事务日志）：记录数据被修改前的值，**用于事务的回滚**。



#### 请重点说说 binlog？

binlog 是一种物理日志，会在磁盘上记录下数据库的所有修改操作，以便进行**数据恢复和主从复制**。

- 当发生数据丢失时，binlog 可以将数据库恢复到特定的时间点。
- 主服务器（master）上的二进制日志可以被从服务器（slave）读取，从而实现数据同步。

binlog 包括两类文件：

- **二进制索引文件**（.index）
- **二进制日志文件**（.00000*）

| 文件类型  | 作用                   | 文件格式   | 关键管理命令                                    |
| --------- | ---------------------- | ---------- | ----------------------------------------------- |
| `.index`  | 记录所有 Binlog 文件名 | 文本格式   | `RESET MASTER`，`PURGE BINARY LOGS`             |
| `.00000*` | 记录数据库变更事件     | 二进制格式 | `SHOW BINARY LOGS`，`mysqlbinlog`，`FLUSH LOGS` |

binlog 默认是没有启用的。要启用它，需要在 MySQL 的配置文件（my.cnf 或 my.ini）中设置 log_bin 参数。



#### 有了binlog为什么还要undolog redolog？

- binlog 主要用于**数据恢复和主从复制**。它**记录了所有对数据库执行的修改操作**（如 INSERT、UPDATE、DELETE），以逻辑日志的形式保存。binlog 是 MySQL Server 层提供的日志，独立于存储引擎。
- redo log 主要用于数据**持久化和崩溃恢复**。redo log 是 InnoDB 存储引擎特有的日志，**用于记录数据的物理修改**，确保数据库在崩溃或异常宕机后能够恢复到一致状态。
- undo log 主要用于支持**事务回滚和多版本并发控制**（MVCC）。undo log 是 InnoDB 存储引擎提供的逻辑日志，用于记录数据的逻辑操作，如删除、更新前的**数据快照**。

当一个事务在 MySQL 中执行时，redo log、undo log 和 binlog 共同协作以确保数据的可靠性和一致性：

1. 事务启动时，undo log 开始记录修改前的数据快照，以便在发生错误或显式回滚时恢复数据。
2. 数据被修改时，InnoDB 会将修改记录到 redo log 中，同时也会生成相应的 undo log。
3. 事务提交时，InnoDB 首先将 redo log 刷入磁盘，然后再将整个事务的操作记录到 binlog 中。这一过程称为“两阶段提交”，确保 binlog 和 redo log 的一致性。
4. 如果数据库发生崩溃，InnoDB 会使用 redo log 进行恢复，确保数据不会丢失。binlog 则可以用来做主从复制或数据恢复到特定时间点。



#### 说说 redolog的工作机制？

redo log 由两部分组成：ib_logfile0 和 ib_logfile1。这两个文件的总大小是固定的，默认情况下每个文件为 48MB，总共 96MB。它们以循环的方式写入，即当写满后，从头开始覆盖旧的日志。

每次修改数据时，都会生成一个新的日志序列号（Log Sequence Number），用于标记 redo log 中的日志位置，以确保数据恢复的一致性。

当一个事务对数据进行修改时，**InnoDB 会首先将这些修改记录到 redo log 中，而不是直接写入磁盘的数据文件**。具体步骤分为三步：

1. 在缓冲池（Buffer Pool）中修改数据页。
2. 将修改操作记录到 redo log buffer 中（这是内存中的一个日志缓冲区）。
3. 当事务提交时，InnoDB 会将 redo log buffer 中的数据刷新到磁盘上的 redo log 文件中（ib_logfile0、ib_logfile1 等），保证事务的持久性。
4. 当 MySQL 发生崩溃后，InnoDB 会在重启时读取 redo log，找到最近一次的检查点（checkpoint），然后从该检查点开始，重放（replay） redo log 中的日志记录，将所有已提交事务的修改重做一遍，恢复数据库到崩溃前的一致性状态。



#### 说说 WAL？

WAL（Write-Ahead Logging，预写日志）的核心思想是**先写日志，再写数据**，即在对数据进行任何修改之前，必须先将修改的日志记录（redo log）持久化到磁盘。

通过先写日志，确保系统在发生故障时可以通过重做日志恢复数据。





### binlog 和 redo log 有什么区别？

- redo log 是固定大小的，通常配置为一组文件，使用环形方式写入，旧的日志会在空间需要时被覆盖。binlog 是追加写入的，新的事件总是被添加到当前日志文件的末尾，当文件达到一定大小后，会创建新的 binlog 文件继续记录。

| **特点**       | **Binlog**                       | **Redo Log**             |
| -------------- | -------------------------------- | ------------------------ |
| **作用**       | 数据恢复、主从复制               | 崩溃恢复、保证事务持久性 |
| **记录内容**   | **逻辑操作**（SQL 语句或行变化） | **数据页物理修改**       |
| **记录时机**   | 事务提交后 **WAL**               | 事务提交前               |
| **作用层级**   | MySQL **Server 层**              | **存储引擎层**（InnoDB） |
| **大小**       | **动态增长**，可设置过期         | **固定大小**，循环覆盖   |
| **是否可关闭** | 可选择关闭                       | 不可关闭                 |



### 为什么要两阶段提交呢？

采用“单阶段”进行提交,即要么先写入 redo log，后写入 binlog；要么先写入 binlog，后写入 redo log。这两种方式的提交都会导致原先数据库的状态和被恢复后的数据库的状态不一致。

redo log 和 binlog 都可以用于表示**事务**的提交状态，而两阶段提交就是让这两个状态保持逻辑上的一致。



### redo log 怎么刷入磁盘？

redo log 的写入不是直接落到磁盘，而是在内存中设置了一片称之为`redo log buffer`的连续内存空间，也就是`redo 日志缓冲区`。

[![redo log 缓冲](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201939796.jpeg)](https://camo.githubusercontent.com/92e96669c8c7d396c50a3f1b991e41ad410d34ef3708b229d3c9e686358300dd/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d65316635393334312d303639352d343564622d623735392d3330646237333331346533392e6a7067)

#### 什么时候会刷入磁盘？

在如下的一些情况中，log buffer 的数据会刷入磁盘：

- **log buffer 空间不足时**

log buffer 的大小是有限的，如果不停的往这个有限大小的 log buffer 里塞入日志，很快它就会被填满。如果当前写入 log buffer 的 redo 日志量已经占满了 log buffer 总容量的大约**一半**左右，就需要把这些日志刷新到磁盘上。

- **事务提交时**

在事务提交时，为了保证持久性，会把 log buffer 中的日志全部刷到磁盘。注意，这时候，除了本事务的，可能还会刷入其它事务的日志。

- **后台线程输入**

有一个后台线程，大约每秒都会刷新一次`log buffer`中的`redo log`到磁盘。

- **正常关闭服务器时**
- **触发 checkpoint 规则**

重做日志缓存、重做日志文件都是以**块（block）的方式进行保存的，称之为重做日志块（redo log block）**,块的大小是固定的 512 字节。我们的 redo log 它是固定大小的，可以看作是一个逻辑上的 **log group**，由一定数量的**log block** 组成。

[![redo log 分块和写入](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201940841.jpeg)](https://camo.githubusercontent.com/4558c33dbd273749f180c8b508f050cbd8c2c7fb08ce69b5ccb45c78692d8ea8/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d38643934346537362d383962612d346661362d393036362d3634666634663535623533322e6a7067)

它的写入方式是从头到尾开始写，写到末尾又回到开头循环写。

其中有两个标记位置：

`write pos`是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。`checkpoint`是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到磁盘。

[![write pos 和 checkpoint](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201940628.jpeg)](https://camo.githubusercontent.com/8cf402f4ed9152c1acebb3cc2ddf5948322c6e12cbdff44a6f49baa854c43150/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d33316131343134392d623236312d343564392d626433622d3661666165633136653133362e6a7067)

当`write_pos`追上`checkpoint`时，表示 redo log 日志已经写满。这时候就不能接着往里写数据了，需要执行`checkpoint`规则腾出可写空间。

所谓的**checkpoint 规则**，就是 checkpoint 触发后，将 buffer 中日志页都刷到磁盘。



## SQL 优化

> [!IMPORTANT]
>
> **一、数据库设计优化**
>
> 1. **合理的表结构设计**
>    - **范式化与反范式平衡**：范式化可以减少数据冗余，提高数据一致性。例如，通过将数据分解到多个表中，按照第一范式（1NF）确保每列都是不可再分的原子数据项，第二范式（2NF）消除部分依赖，第三范式（3NF）消除传递依赖。但过度范式化可能会导致查询时需要频繁地进行表连接操作，影响性能。反范式则可以在一定程度上减少连接操作，通过增加冗余数据来提高查询速度。例如，在电商系统中，对于频繁查询的订单和订单详情信息，可以适当将一些关键信息（如商品名称、价格）冗余到订单表中，减少连接查询。
>    - **合适的数据类型选择**：为字段选择合适的数据类型可以节省存储空间并提高查询效率。例如，对于存储整数的字段，如果数据范围较小，可以选择较小的数据类型，如 `TINYINT`（范围是 -128 到 127）而不是 `INT`。对于日期时间字段，使用 `DATE` 或 `DATETIME` 类型，而不是将日期存储为字符串类型，这样可以利用数据库对日期时间类型内置的优化和函数操作。
> 2. **索引设计**
>    - **创建有效的索引**：索引是提高查询性能的关键。对于经常作为查询条件的列，如用户表中的用户名、身份证号等字段，应该创建索引。例如，在一个员工管理系统中，如果经常根据员工编号查询员工信息，那么在员工编号列上创建索引可以加快查询速度。同时，对于复合查询条件，可以考虑创建复合索引。比如，查询订单时经常根据订单日期和订单状态进行筛选，那么可以在这两个字段上创建复合索引。
>    - **避免过度索引**：过多的索引会增加数据库的维护成本，尤其是在数据插入、更新和删除操作时，索引也需要同步更新。而且，过多的索引会占用大量的存储空间。例如，一个表有几十个字段，如果为每个字段都创建索引，会导致数据库性能下降，因为每次数据变动都需要更新多个索引。
>    - **索引的维护**：定期检查索引的状态，如索引的碎片化情况。当索引碎片化严重时，会影响查询性能。可以通过数据库提供的工具（如 SQL Server 的 `DBCC SHOWCONTIG`）来检查索引碎片，并进行重建或重新组织索引。
> 3. **分区设计**
>    - **水平分区**：将表中的数据按照某种规则划分到不同的分区中。例如，对于一个大型的日志表，可以根据时间（如按月或按年）进行分区。这样，在查询特定时间范围的日志时，数据库只需要扫描相关的分区，大大减少了数据扫描量。同时，分区也便于数据的维护，比如可以快速删除旧分区中的数据。
>    - **垂直分区**：将表中的列按照使用频率或数据类型进行划分。比如，将一个包含大量文本字段和少量关键字段的表进行垂直分区，将关键字段和文本字段分别存储在不同的表中，这样可以提高对关键字段的查询效率。
>
> 
>
> **二、SQL语句优化**
>
> 1. **优化查询语句**
>    - **避免使用 SELECT \***：尽量明确指定需要查询的列。使用 `SELECT *` 会查询表中的所有列，这可能会导致不必要的数据传输和处理开销。例如，在一个包含几十个字段的表中，如果只需要查询其中的 3 个字段，使用 `SELECT *` 会将所有字段的数据都加载到内存中，然后过滤出需要的字段，这是非常低效的。
>    - **减少嵌套查询**：嵌套查询可能会导致数据库进行多次扫描和计算。例如，可以通过连接（JOIN）操作来替代一些嵌套查询。对于复杂的嵌套查询，可以尝试将其分解为多个简单的查询，并通过临时表或视图来存储中间结果。
>    - **优化连接操作**：在使用 JOIN 时，尽量确保连接的字段上有合适的索引。例如，在连接两个表时，如果连接条件是两个表的主键或外键，并且这些字段上有索引，那么连接操作的性能会更好。同时，要注意连接的顺序，尽量先连接小表，这样可以减少数据的中间处理量。
> 2. **优化更新和删除语句**
>    - **批量操作**：对于大量数据的更新和删除操作，尽量使用批量操作而不是逐条操作。例如，在删除一批符合条件的记录时，可以使用 `DELETE FROM table_name WHERE condition`，而不是通过循环逐条删除。同时，要注意批量操作对事务日志的影响，避免事务日志过大导致性能问题。
>    - **限制更新和删除范围**：在更新和删除语句中，尽量明确指定条件，避免全表扫描。例如，在更新用户表中的用户状态时，明确指定需要更新的用户 ID 范围，而不是直接更新整个表。
>
> 
>
> **三、数据库配置和硬件资源优化**
>
> 1. **数据库参数调整**
>
>    - **内存参数**：根据服务器的物理内存和数据库的使用情况，合理配置数据库的内存参数。例如，在 SQL Server 中，可以调整 `max server memory` 和 `min server memory` 参数，为数据库分配合适的内存空间。如果内存分配过多，会影响其他应用程序的运行；如果分配过少，会导致数据库频繁进行磁盘 I/O 操作，降低性能。
>    - **并发参数**：调整数据库的并发连接数等参数，以适应应用程序的并发需求。例如，在高并发的电商系统中，需要适当增加数据库的并发连接数，同时要防止过多的并发连接导致数据库服务器过载。
>
> 2. **硬件资源优化**
>
>    - **存储优化**：使用高性能的存储设备，如固态硬盘（SSD）。SSD 的读写速度比传统的机械硬盘要快得多，可以显著提高数据库的 I/O 性能。同时，合理配置存储的 RAID 级别，如 RAID 10 可以在一定程度上提高数据的安全性和读写性能。
>    - **服务器性能提升**：增加服务器的 CPU 核心数、内存容量等硬件资源可以提高数据库的处理能力。例如，在处理复杂查询和大量数据时，更多的 CPU 核心可以加快查询的计算速度，更多的内存可以容纳更多的数据缓存，减少磁盘 I/O 操作。
>
> 3. **构建高可用集群**
>
>    





### 慢 SQL 怎么定位呢？

#### 什么是慢 SQL？

慢 SQL 也就是执行时间较长的 SQL 语句，MySQL 中 long_query_time 默认值是 10 秒，也就是执行时间超过 10 秒的 SQL 语句会被记录到慢查询日志中。

不过，生产环境中，10 秒太久了，超过 1 秒的都可以认为是慢 SQL 了。

#### SQL 的执行过程了解吗？

1. 客户端发送 SQL 语句给 MySQL 服务器。
2. 如果查询缓存打开则会优先查询缓存，缓存中有对应的结果就直接返回。不过，MySQL 8.0 已经移除了查询缓存。
3. 分析器对 SQL 语句进行语法分析，判断是否有语法错误。
4. 搞清楚 SQL 语句要干嘛后，MySQL 会通过优化器生成执行计划。
5. 执行器调用存储引擎的接口，执行 SQL 语句。

[![三个猪皮匠：SQL 执行过程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201943327.png)](https://camo.githubusercontent.com/5f9b2249f0ef01dba36203438f803f1c43ac84da8a197eed67766b8e7962bc9a/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303332373038333833382e706e67)

SQL 执行过程中，优化器通过成本计算预估出执行效率最高的方式，基本的预估维度得出影响 SQL 执行效率的因素有

**①、IO 成本**

- 数据量：数据量越大，IO 成本越高。所以要避免 `select *`；尽量分页查询。
- 数据从哪读取：尽量通过索引加快查询。

**②、CPU 成本**

- 尽量避免复杂的查询条件，如有必要，考虑对子查询结果进行过滤。
- 尽量缩减计算成本，比如说为排序字段加上索引，提高排序效率；比如说使用 union all 替代 union，减少去重处理。



#### 如何优化慢 SQL？

- 首先，找到那些比较慢的 SQL，可以通过**启用慢查询日志**，记录那些超过指定执行时间的查询。
- 也可以使用 `show processlist;` 命令查看当前正在执行的 SQL 语句，找出执行时间较长的 SQL。
- 或者在业务基建中加入对慢 SQL 的监控，常见的方案有**字节码插桩**、**连接池扩展**、**ORM 框架扩展**。

- 然后，使用 **EXPLAIN 查看查询执行计划**，判断查询是否使用了索引，是否有全表扫描等。


- 最后，根据分析结果，通过添加或优化索引、调整查询语句或者增加内存缓冲区来优化 SQL。




#### 慢sql日志怎么开启？

- 直接编辑 MySQL 的配置文件 my.cnf 或 my.ini然后重启 MySQL 服务就好了，
- 也可以通过 set global 命令动态设置。





### :star:有哪些方式优化 SQL？

[![沉默王二：SQL 优化](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412201946832.png)](https://camo.githubusercontent.com/bcf358bc2bc4c086c87050eef43846ae113741930144507a9717cb9602f4f56f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303332373130343035302e706e67)

#### 如何进行分页优化？

**①、延迟关联**

延迟关联**适用于需要从多个表中获取数据且主表行数较多的情况**。它**首先从索引表中检索出需要的行 ID，然后再根据这些 ID 去关联其他的表获取详细信息。**

```mysql
SELECT e.id, e.name, d.details
FROM employees e
JOIN department d ON e.department_id = d.id
ORDER BY e.id
LIMIT 1000, 20;
```

延迟关联后：

```mysql
SELECT e.id, e.name, d.details
FROM (
    SELECT id
    FROM employees
    ORDER BY id
    LIMIT 1000, 20
) AS sub
JOIN employees e ON sub.id = e.id
JOIN department d ON e.department_id = d.id;
```

首先对`employees`表进行分页查询，仅获取需要的行的 ID，然后再根据这些 ID 关联获取其他信息，减少了不必要的 JOIN 操作。

**②、书签（Seek Method）**

**书签方法通过记住上一次查询返回的最后一行的某个值，然后下一次查询从这个值开始**，避免了扫描大量不需要的行。

假设需要对用户表进行分页，根据用户 ID 升序排列。

```mysql
SELECT id, name
FROM users
ORDER BY id
LIMIT 1000, 20;
```

书签方式：

```mysql
SELECT id, name
FROM users
WHERE id > last_max_id  -- 假设last_max_id是上一页最后一行的ID
ORDER BY id
LIMIT 20;
```

**优化后的查询不再使用`OFFSET`**，而是直接从上一页最后一个用户的 ID 开始查询。这里的`last_max_id`是上一次查询返回的最后一行的用户 ID。这种方法有效避免了不必要的数据扫描，提高了分页查询的效率。



#### 如何进行索引优化？

**①、利用覆盖索引**

**使用非主键索引查询数据时需要回表，但如果索引的叶节点中已经包含要查询的字段，那就不会再回表查询**了，这就叫覆盖索引。

举个例子，现在要从 test 表中查询 city 为上海的 name 字段。

```mysql
select name from test where city='上海'
```

如果仅在 city 字段上添加索引，那么这条查询语句会先通过索引找到 city 为上海的行，然后再回表查询 name 字段，这就是回表查询。

为了**避免回表查询**，可以在 city 和 name 字段上建立联合索引，这样查询结果就可以直接从索引中获取。

```mysql
alter table test add index index1(city,name);
```

**②、避免使用 != 或者 <> 操作符**

**`!=` 或者 `<>` 操作符会导致 MySQL 无法使用索引，从而导致全表扫描**。

例如，可以把`column<>'aaa'`，改成`column>'aaa' or column<'aaa'`，就可以使用索引了。

优化策略就是尽可能使用 `=`、`>`、`<`、`BETWEEN`等操作符，它们能够更好地利用索引。

**③、适当使用前缀索引**

适当使用**前缀索引**可以降低索引的空间占用，提高索引的查询效率。

比如，邮箱的后缀一般都是固定的`@xxx.com`，那么类似这种后面几位为固定值的字段就非常适合定义为前缀索引：

```mysql
alter table test add index index2(email(6));
```

需要注意的是，**MySQL 无法利用前缀索引做 order by 和 group by 操作**。

**④、避免列上使用函数**

**在 where 子句中直接对列使用函数会导致索引失效**，因为数据库需要对每行的列应用函数后再进行比较，无法直接利用索引。

```mysql
select name from test where date_format(create_time,'%Y-%m-%d')='2021-01-01';
```

可以改成：

```mysql
select name from test where create_time>='2021-01-01 00:00:00' and create_time<'2021-01-02 00:00:00';
```

通过日期的范围查询，而不是在列上使用函数，可以利用 create_time 上的索引。

**⑤、正确使用联合索引**

正确地使用联合索引可以极大地提高查询性能，联合索引的创建应遵循**最左前缀原则**，即索引的顺序应根据列在查询中的使用频率和重要性来安排。

```mysql
select * from messages where sender_id=1 and receiver_id=2 and is_read=0;
```

那就可以为 sender_id、receiver_id 和 is_read 这三个字段创建联合索引，但是要注意索引的顺序，应该按照查询中的字段顺序来创建索引。

```mysql
alter table messages add index index3(sender_id,receiver_id,is_read);
```



#### 如何进行 JOIN 优化？

**①、优化子查询**

子查询，特别是在 select 列表和 where 子句中的子查询，往往会导致性能问题，因为它们可能会为每一行外层查询执行一次子查询。

使用子查询：

```mysql
select name from A where id in (select id from B);
```

**使用 JOIN 代替子查询**：

```mysql
select A.name from A join B on A.id=B.id;
```

**②、小表驱动大表**

在执行 JOIN 操作时，应尽量让**行数较少的表（小表）驱动行数较多的表（大表）**，这样可以减少查询过程中需要处理的数据量。

比如 **left join，左表是驱动表**，所以 A 表应小于 B 表，这样建立连接的次数就少，查询速度就快了。

```mysql
select name from A left join B;
```

**③、适当增加冗余字段**

在某些情况下，通过在表中**适当增加冗余字段来避免 JOIN 操作**，可以提高查询效率，尤其是在高频查询的场景下。

比如，我们有一个订单表和一个商品表，查询订单时需要显示商品名称，如果每次都通过 JOIN 操作查询商品表，会降低查询效率。这时可以在订单表中增加一个冗余字段，存储商品名称，这样就可以避免 JOIN 操作。

```mysql
select order_id,product_name from orders;
```

**④、避免使用 JOIN 关联太多的表**

《[阿里巴巴 Java 开发手册](https://javabetter.cn/pdf/ali-java-shouce.html)》上就规定，**不要使用 join 关联太多的表，最多不要超过 3 张表**。

因为 join 太多表会降低查询的速度，返回的数据量也会变得非常大，不利于后续的处理。

如果业务逻辑允许，**可以考虑将复杂的 JOIN 查询分解成多个简单查询，然后在应用层组合这些查询的结果。**



#### 如何进行排序优化？

MySQL 生成有序结果的方式有两种：一种是**对结果集进行排序**操作，另外一种是**按照索引顺序扫描得出的自然有序结果**。

因此在设计索引的时候要充分考虑到排序的需求。

```mysql
select id, name from users order by name;
```

如果 name 字段上有索引，那么 MySQL 可以**直接利用索引的有序性**，避免排序操作。



**①、条件下推**

**条件下推是指将 where、limit 等子句下推到 union 的各个子查询中，以便优化器可以充分利用这些条件进行优化。**

假设我们有两个查询分支，需要合并结果并过滤：

```mysql
SELECT * FROM (
    SELECT * FROM A
    UNION
    SELECT * FROM B
) AS sub
WHERE sub.id = 1;
```

可以改写成：

```mysql
SELECT * FROM A WHERE id = 1
UNION
SELECT * FROM B WHERE id = 1;
```

通过将查询条件下推到 UNION 的每个分支中，每个分支查询都只处理满足条件的数据，减少了不必要的数据合并和过滤。





### 怎么看执行计划 explain，如何理解其中各个字段的含义？

explain 是 MySQL 提供的一个用于查看查询执行计划的工具，可以帮助我们分析查询语句的性能瓶颈，找出慢 SQL 的原因。

使用方式在 select 语句前加上 `explain` 关键字就可以了。

```mysql
explain select * from students where id =9
```

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202044681.jpeg" alt="三分恶面渣逆袭：EXPLAIN" style="zoom:50%;" />

explain 的输出结果中包含了很多字段，下面是一些常见的字段含义：

①、**id** 列：查询的标识符。

②、**select_type** 列：查询的类型。常见的类型有：

- SIMPLE：简单查询，不包含子查询或者 UNION 查询。
- PRIMARY：查询中如果包含子查询，则最外层查询被标记为 PRIMARY。
- SUBQUERY：子查询。
- DERIVED：派生表的 SELECT，FROM 子句的子查询。

③、**table** 列：查的哪个表。

④、**type** 列：表示 MySQL 在表中找到所需行的方式，性能从最优到最差分别为：system > const > eq_ref > ref > range > index > ALL。

- system，表只有一行，一般是系统表，往往不需要进行磁盘 IO，速度非常快
- const：表中只有一行匹配，或通过主键或唯一索引获取单行记录。通常用于使用主键或唯一索引的精确匹配查询，性能非常高。
- eq_ref：对于每个来自上一张表的记录，最多只返回一条匹配记录，通常用于多表关联且使用主键或唯一索引的查询。效率非常高，适合多表关联查询。
- ref：使用非唯一索引或前缀索引查询的情况，返回符合条件的多行记录。通常用于普通索引或联合索引查询，效率较高，但不如 const 和 eq_ref。
- range：只检索给定范围的行，使用索引来检索。在`where`语句中使用 `bettween...and`、`<`、`>`、`<=`、`in` 等条件查询 `type` 都是 `range`。
- index：全索引扫描，即扫描整个索引而不访问数据行。
- ALL：全表扫描，效率最低。

⑤、**possible_keys** 列：可能会用到的索引，但并不一定实际被使用。

⑥、**key** 列：实际使用的索引。如果为 NULL，则没有使用索引。

⑦、**key_len** 列：MySQL 决定使用的索引长度（以字节为单位）。当表有多个索引可用时，key_len 字段可以帮助识别哪个索引最有效。通常情况下，更短的 key_len 意味着数据库在比较键值时需要处理更少的数据。

⑧、**ref** 列：用于与索引列比较的值来源。

- const：表示常量，这个值是在查询中被固定的。例如在 WHERE `column = 'value'`中。
- 一个或多个列的名称，通常在 JOIN 操作中，表示 JOIN 条件依赖的字段。
- NULL，表示没有使用索引，或者查询使用的是全表扫描。

⑨、**rows** 列：估算查到结果集需要扫描的数据行数，原则上 rows 越少越好。

⑩、**Extra** 列：附加信息。

- Using index：表示只利用了索引。
- Using where：表示使用了 WHERE 过滤。
- Using temporary ：表示使用了临时表来存储中间结果。



#### type的执行效率等级，达到什么级别比较合适？

从高到低的效率排序是 system、const、eq_ref、ref、range、index 和 ALL。

一般情况下，建议 type 值达到 const、eq_ref 或 ref，因为这些类型表明查询使用了索引进行精确匹配，效率较高。

如果是范围查询，range 类型也是可以接受的。

通常要**避免出现 ALL 类型**，因为它表示全表扫描，性能最低。





## 索引

### 为什么使用索引会加快查询？

数据库文件是存储在磁盘上的，**磁盘 I/O** 是数据库操作中最耗时的部分之一。没有索引时，数据库会进行**全表扫描**（Sequential Scan）有了索引，就可以直接跳到索引指示的数据位置，而不必扫描整张表，从而大大减少了磁盘 I/O 操作的次数。

MySQL 的 InnoDB 存储引擎默认使用 **B+ 树**来作为索引的数据结构，而 B+ 树的查询效率非常高，时间复杂度为 O(logN)。

可通过 `create index` 创建索引，比如：

```mysql
create index idx_name on students(name);
```

#### 索引优化举例？

在实际开发中，我们可以通过合理使用单字段索引、复合索引和覆盖索引来优化查询。例如，如果要加速查询 age 字段的条件，我们可以在 age 字段上创建索引。

```mysql
CREATE INDEX idx_age ON users(age);
```

如果查询涉及多个字段 age 和 name，可以使用**复合索引**来提高查询效率。

```mysql
CREATE INDEX idx_age_name ON users(age, name);
```

当我们只需要查询部分字段时 `SELECT name FROM users WHERE age = 30;`，**覆盖索引**可以提升查询效率。

```mysql
CREATE INDEX idx_age_name ON users(age, name);
```

由于 age 和 name 字段都在索引中，MySQL 直接从索引中获取结果，无需回表查找。



### 能简单说一下索引的分类吗？

![二哥的 Java 进阶之路：索引类型](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202112915.png)

#### 说说从功能上的分类？

①、**主键索引**: 表中每行数据唯一标识的索引，强调**列值的唯一性和非空性**。

当创建表的时候，可以直接指定主键索引：

```mysql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    email VARCHAR(255)
);
```

id 列被指定为主键索引，同时，MySQL 会自动为这个列创建一个**聚簇索引**（**主键索引一定是聚簇索引**）。

②、**唯一索引**: 保证数据**列中每行数据的唯一性，但允许有空值**。

可以通过下面的语句创建唯一索引：

```mysql
CREATE UNIQUE INDEX idx_username ON users(username);
```

③、**普通索引**: 基本的索引类型，用于加速查询。

可以通过下面的语句创建普通索引：

```mysql
CREATE INDEX idx_email ON users(email);
```

④、**全文索引**：特定于**文本数据的索引**，用于提高文本搜索的效率。

假设有一个名为 articles 的表，下面这条语句在 content 列上创建了一个全文索引。

```mysql
CREATE FULLTEXT INDEX idx_article_content ON articles(content);
```



#### 说说从数据结构上分类？

①、**B+树索引**：

[![一颗剽悍的种子：B+树的结构](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202117491.png)](https://camo.githubusercontent.com/5812a4567768205a3f97f6d3e97522b4e22e1044dfd12c66a4fa96c71822c8d1/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303331323039323734352e706e67)

②、**Hash 索引**：基于哈希表的索引，查询效率可以达到 O(1)。

Hash 索引在原理上和 Java 中的 [HashMap](https://javabetter.cn/collection/hashmap.html) 类似，当发生哈希冲突的时候也是通过拉链法来解决。

[![业余码农：哈希索引](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202117631.png)](https://camo.githubusercontent.com/e5250ee8ac0255cd70d855999e7b97fcfe22e7c757b23b20ee889b460734e489/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303331323039343533372e706e67)

注意，我们这里创建的是 MEMORY 存储引擎，**InnoDB 并不提供直接创建哈希索引的选项**，因为 B+ 树索引能够很好地支持范围查询和等值查询，满足了大多数数据库操作的需要。

不过，InnoDB 存储引擎内部使用了一种名为“**自适应哈希索引**”（Adaptive Hash Index, AHI）的技术。

自适应哈希索引并不是由用户显式创建的，而是 InnoDB 根据数据访问的模式**自动**建立和管理的。当 InnoDB 发现某个索引**被频繁访问**时，会在内存中创建一个哈希索引，以加速对这个索引的访问。



#### 说说从存储位置上分类：

①、聚簇索引：聚簇索引的叶子节点保存了一行记录的所有列信息。也就是说，聚簇索引的叶子节点中，包含了一个完整的记录行。

②、非聚簇索引：它的叶子节点只包含一个主键值，通过非聚簇索引查找记录要先找到主键，然后通过主键再到聚簇索引中找到对应的记录行，这个过程被称为**回表**。

InnoDB 存储引擎的主键使用的是聚簇索引，MyISAM 存储引擎不管是主键索引，还是二级索引使用的都是非聚簇索引。





### 创建索引有哪些注意点？

**①、选择合适的列作为索引**

- 经常作为**查询条件**（WHERE 子句）、**排序条件**（ORDER BY 子句）、**分组条件**（GROUP BY 子句）的列是建立索引的好选项。
- **区分度低的字段，例如性别，不要建索引**
- **频繁更新的字段，不要建索引**

**②、避免过多的索引**

- 因为每个索引都需要占用额外的磁盘空间。
- **更新表（INSERT、UPDATE、DELETE 操作）的时候，索引都需要被更新。**

**③、利用前缀索引和索引列的顺序**

- 对于字符串类型的列，可以考虑使用前缀索引来减少索引大小。
- 在创建联合索引时，应该根据查询条件将最常用的放在前面，遵守**最左前缀原则。**



### 索引哪些情况下会失效呢？

- **在索引列上使用函数或表达式**：索引可能无法使用，因为 MySQL 无法预先计算出函数或表达式的结果。例如：`SELECT * FROM table WHERE YEAR(date_column) = 2021`。
- **使用不等于（`<>`）或者 NOT 操作符**：因为它们会扫描全表。
- **使用 LIKE 语句，但通配符在前面**：以“%”或者“_”开头，索引也无法使用。例如：`SELECT * FROM table WHERE column LIKE '%abc'`。
- **联合索引，但 WHERE 不满足最左前缀原则**，索引无法起效。例如：`SELECT * FROM table WHERE column2 = 2`，联合索引为 `(column1, column2)`。



### 索引不适合哪些场景呢？

- **数据表较小**：当表中的数据量很小，或者查询需要扫描表中大部分数据时，数据库优化器可能会选择全表扫描而不是使用索引。在这种情况下，维护索引的开销可能大于其带来的性能提升。
- **频繁更新的列**：对于经常进行更新、删除或插入操作的列，使用索引可能会导致性能下降。因为每次数据变更时，索引也需要更新，这会增加额外的写操作负担。

#### 性别字段要建立索引吗？

性别字段通常不适合建立索引。因为性别字段的选择性（区分度）较低，独立索引效果有限。

如果性别字段又很少用于查询，表的数据规模较小，那么建立索引反而会增加额外的存储空间和维护成本。

如果性别字段确实经常用于查询条件，数据规模也比较大，**可以将性别字段作为复合索引的一部分，与选择性较高的字段一起加索引**，会更好一些。

#### 什么是区分度？

区分度（Selectivity）是衡量**一个字段在数据库表中唯一值的比例**，用来表示该字段在索引优化中的有效性。

区分度 = 字段的唯一值数量 / 字段的总记录数；接近 1，字段值大部分是唯一的。例如，用户的唯一 ID，一般都是主键索引。接近 0，则说明字段值重复度高。

例如，一个表中有 1000 条记录，其中性别字段只有两个值（男、女），那么性别字段的区分度只有 0.002。

高区分度的字段更适合拿来作为索引，因为索引可以更有效地缩小查询范围。

可以通过 `COUNT(DISTINCT column_name)` 和 `COUNT(*)` 的比值来计算字段的区分度



#### 什么样的字段适合加索引？什么不适合？

适合加索引的字段包括：

- 经常出现在 WHERE 子句中的字段，如 `SELECT * FROM users WHERE age = 30` 中的 age 字段，加上索引后可以快速定位到满足条件的记录。
- 经常用于 JOIN 的字段，如 `SELECT * FROM users u JOIN orders o ON u.id = o.user_id` 中的 user_id 字段，加上索引后可以避免多表扫描。
- 经常出现在 ORDER BY 或 GROUP BY 子句中的字段，如 `SELECT * FROM users ORDER BY age` 中的 age 字段。加上索引后可以避免额外的排序操作。
- 高区分度的字段，查询时可以有效减少返回的数据行，比如用户 ID、邮箱等。

对应的，不适合加索引的字段包括：

- 低区分度字段，如性别、状态等。
- 经常更新的字段，如用户的登录时间、登录次数等。
- 不经常出现在查询条件中的字段，如用户的生日、地址等。
- 使用函数、运算符的字段。





### 索引是不是建的越多越好？

- **索引会占据磁盘空间**
- **索引虽然会提高查询效率，但是会降低更新表的效率**。每次对表进行增删改操作，MySQL 不仅要更新数据，还要更新对应的索引文件。

#### 说说索引优化的思路？

①、选择合适的索引类型

- 如果需要等值查询和范围查询，请选择 B+树索引。
- 如果是用于处理文本数据的全文搜索，请选择全文索引。

②、创建适当的索引

- 创建组合索引时，应将查询中最常用、**区分度高的列放在前面**。对于查询条件 `WHERE age = 18 AND gender = '女' AND city = '洛阳'`，如果 age 列的值相对较为分散，可以优先考虑将 age 放在组合索引的第一位。
- 使用 SELECT 语句时，尽量选择覆盖索引来避免不必要的回表操作，也就是说，索引中包含了查询所需的所有列；但要注意，覆盖索引的列数不宜过多，否则会增加索引的存储空间。





### :star:为什么 InnoDB 要使用 B+树作为索引？

B 树是一种自平衡的多路查找树，和红黑树、二叉平衡树不同，B 树的每个节点可以有 m 个子节点，而红黑树和二叉平衡树都只有 2 个。

内存和磁盘在进行 IO 读写的时候，有一个最小的逻辑单元，叫做页（Page），页的大小一般是 4KB。

[![二哥的 Java 进阶之路：IO 读写](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202138637.png)](https://camo.githubusercontent.com/d41aa6c9db2af2b728400eb4f1c51461ea249be4d3e6b8d1c80d5e5a03f0b51c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303332323133333635302e706e67)

那为了提高读写效率，从磁盘往内存中读数据的时候，一次会读取至少一页的数据，比如说读取 2KB 的数据，实际上会读取 4KB 的数据；读取 5KB 的数据，实际上会读取 8KB 的数据。**我们要尽量减少读写的次数**。

树越高，意味着查找数据时就需要更多的磁盘 IO，因为每一层都可能需要从磁盘加载新的节点。



**B 树的节点大小通常与页的大小对齐**，这样每次从磁盘加载一个节点时，可以正好是一个页的大小。因为 B 树的节点可以有多个子节点，可以填充更多的信息以达到一页的大小。

正是因为 B 树的每个节点上都存了数据，就导致每个节点能存储的键值和指针变少了，因为每一页的大小是固定的，于是 B+树就来了，B+树的非叶子节点只存储键值，不存储数据，而叶子节点存储了所有的数据，并且构成了一个有序链表。

这样做的好处是，**非叶子节点上由于没有存储数据，就可以存储更多的键值对，树就变得更加矮胖了**，再加上叶子节点构成了一个有序链表，范围查询时就**可以直接通过叶子节点间的指针顺序访问整个查询范围内的所有记录**，而无需对树进行多次遍历。



#### 为什么 MongoDB 索引用 B树，而 MySQL 用 B+ 树？

- B树的特点是每个节点都存储数据，相邻的叶子节点之间没有指针链接。

- B+树的特点是非叶子节点只存储索引，叶子节点存储数据，并且相邻的叶子节点之间有指针链接。


MySQL 属于关系型数据库，所以范围查询会比较多，所以采用了 B+树；但 MongoDB 属于非关系型数据库，在大多数情况下，只需要查询单条数据，所以 MongoDB 选择了 B 树。





### 一棵 B+树能存储多少条数据呢？

![image-20241220214428199](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202144359.png)

假如我们的主键 ID 是 bigint 类型，长度为 8 个字节。指针大小在 InnoDB 源码中设置为 6 字节，这样一共 14 字节。所以非叶子节点(一页)可以存储 **（16KB）16384/14=1170**个这样的单元(键值+指针)。

一个指针指向一个存放记录的页，**一页可以放 16 条数据**，**树深度为 2** 的时候，可以存放 1170*16=**18720** 条数据。

同理，树深度为 3 的时候，可以存储的数据为 1170乘1170乘16=**21902400**条记录。

理论上，在 InnoDB 存储引擎中，B+树的高度一般为 2-4 层，就可以满足千万级数据的存储。查找数据的时候，一次页的查找代表一次 IO，当我们通过主键索引查询的时候，最**多只需要 2-4 次 IO** 就可以了。



#### innodb 使用数据页存储数据？默认数据页大小 16K，我现在有一张表，有 2kw 数据，我这个 b+树的高度有几层？

如果有 2KW 条数据，那么这颗 B+树的高度为 3 层。



#### 为什么用 B+树不用跳表呢？

- 跳表基于链表，节点分布不连续，会**频繁触发随机磁盘访问**，性能较差。
- 跳表需要**逐节点遍历链表**，范围查询性能不如 B+ 树。





### Hash 索引和 B+ 树索引区别是什么？

- B+ 树索引可以进行范围查询，Hash 索引不能。
- B+ 树索引支持联合索引的最左侧原则，Hash 索引不支持。
- B+ 树索引支持 order by 排序，Hash 索引不支持。
- Hash 索引在等值查询上比 B+ 树索引效率更高。
- B+ 树使用 like 进行模糊查询的时候，`LIKE 'abc%'` 的话可以起到索引优化的作用，**Hash 索引无法进行模糊查询。**

#### MySQL 模糊查询怎么查，什么情况下模糊查询不走索引？

MySQL 中进行模糊查询主要使用 LIKE 语句，结合**通配符 %（代表任意多个字符）和 _（代表单个字符）**来实现。

```mysql
SELECT * FROM table WHERE column LIKE '%xxx%';
```

这个查询会返回所有 column 列中包含 xxx 的记录。

但是，如果模糊查询的**通配符 % 出现在搜索字符串的开始位置，如 `LIKE '%xxx'`，MySQL 将无法使用索引**，因为数据库必须扫描全表以匹配任意位置的字符串。



### 回表了解吗？

回表是指在数据库查询过程中，**通过非聚簇索引（secondary index）查找到记录的主键值后，再根据这个主键值到聚簇索引（clustered index）中查找完整记录的过程。**

回表操作通常发生在使用非聚簇索引进行查询，但查询的字段不全在该索引中，必须通过主键进行再次查询以获取完整数据。换句话说，数据库需要先查找索引，然后再根据索引回到数据表中去查找实际的数据。

**索引覆盖**（Covering Index）可以减少回表操作，**将查询的字段都放在索引中**，这样不需要回表就可以获取到查询结果了。



### 联合索引了解吗？

#### 联合索引的叶子节点存的什么内容?

比如说有这样一个联合索引 idx_c2_c3（c2 和 c3 列，主键是 c1），那么**叶子节点存储的是 c2、c3 索引列的值和c1 主键列的值**。这样，当查询时，可以先通过联合索引找到对应的主键值，然后再通过主键值找到完整的数据行。





### 覆盖索引了解吗？

覆盖索引（Covering Index）指的是一种索引能够**“覆盖”查询中所涉及的所有列**，换句话说，查询所需的数据全部都可以从索引中直接获取，而无需访问数据表的行数据（也就是**无需回表**）。

假设有一张用户表 users，包含以下字段：id、name、email、age。执行下面的查询：

```mysql
SELECT age, email FROM users WHERE name = "张三";
```

如果在 name 列上创建了索引，但没有在 age 和 email 列上创建索引，那么数据库引擎会：

1. 使用 name 列的索引查找到满足条件的记录的 id。
2. 根据 id 回表查询 age 和 email 字段的数据。

如果创建了一个覆盖索引 idx_users_name_email_age 包含 name、email、age 列：

```mysql
CREATE INDEX idx_users_name_email_age ON users (age, name, email);
```





### 什么是最左前缀原则？

在使用联合索引时，应当遵守最左前缀原则，或者叫最左匹配原则（最左前缀匹配原则）。

指的是**在使用联合索引时，查询条件从索引的最左列开始并且不跳过中间的列。**

如果一个复合索引包含`(col1, col2, col3)`，那么它可以支持 `col1`、`col1,col2` 和 `col1, col2, col3` 的查询优化，但不支持只有 col2 或 col3 的查询。

**不满足最左前缀法则时候，MySQL会进行全表扫描。**



#### 为什么不从最左开始查，就无法匹配呢？

联合索引在 B+ 树中是复合的数据结构，按照从左到右的顺序依次建立搜索树 (name 在左边，age 在右边)。

[![三分恶面渣逆袭：联合索引](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202211232.jpeg)](https://camo.githubusercontent.com/2eb42b94a8ee56eebf764d8b4575a20580c1feb94ff783068e2d47e87ca092b8/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d65333438323033632d663030612d343261342d613734352d6232313964393865613433352e6a7067)

注意，name 是有序的，age 是无序的。当 name 相等的时候，age 才有序。

当我们使用 `where name= '张三' and age = '20'` 去查询的时候， B+ 树会优先比较 name 来确定下一步应该搜索的方向，往左还是往右。

如果 name 相同的时候再比较 age。

但如果查询条件没有 name，就不知道应该怎么查了，因为 name 是 B+树中的前置条件，没有 name，索引就派不上用场了。



#### （联合索引）下面怎么走的索引？

```mysql
select * from t where a = 2 and b = 2;
select * from t where b = 2 and c = 2;
select * from t where a > 2 and b = 2;
```

**第三条，只有前面 a > 2 用到了联合索引，b = 2是得出结果后再额外进行过滤。**

**`a > 2` 是范围查询（`>`），MySQL 在遇到范围查询后，后续的索引字段（如 `b`）不能用于索引查找，只能作为普通的过滤条件！**

**`a > 2` 可以使用索引，但 `b = 2` 不能直接利用索引。**



**范围查询 (`>`, `<`, `BETWEEN`) 会影响索引的使用**：

- **后续的索引字段无法用于索引查找，但可能仍可用于排序（如果有序）或索引下推（ICP）**。



#### 下面代码怎么创建索引？

```mysql
mysqlSELECT * FROM table   WHERE age > 90 AND address = 'sss'   ORDER BY created_at DESC;
```

这个语句怎么进行索引优化？

根据**最左前缀原则**，最优索引应该按照**等值查询列、范围查询列、排序列**的顺序来排列：

```mysql
CREATE INDEX idx_table ON table (address, age, created_at);
```





#### **索引失效 & 隐式类型转换**

假设 `students` 表有如下索引：

```mysql
CREATE INDEX idx_students ON students (student_id, class_id);
```

问：**以下 SQL 是否能正确使用索引？如果不能，如何优化？**

```mysql
SELECT * FROM students WHERE student_id = '12345';
```

**分析**

- `student_id` 是 `INT` 类型，而 `'12345'` 是 **字符串**，MySQL 会**隐式转换 `student_id` 为字符串**，导致**索引失效**。

- 优化方法

  - 确保查询时类型匹配

    ```mysql
    SELECT * FROM students WHERE student_id = 12345;
    ```

  - **如果 `student_id` 是字符串类型（如 `VARCHAR`），则索引仍然有效**。







### 什么是索引下推 (ICP) 优化？

索引下推`（Index Condition Pushdown (ICP) ）`是 MySQL 5.6 时添加的，它允许在访问索引时对数据进行过滤，从而减少回表的次数。

例如有一张 user 表，建了一个联合索引（name, age），查询语句：`select * from user where name like '张%' and age=10;`，没有索引下推优化的情况下：

MySQL 会先根据 `name like '张%'` 查找条件匹配的数据，对于符合索引条件的每一条记录，都会去访问对应的数据行，并在 Server 层过滤 `age=10` 这个条件。

这样就等于说即使 age 不等于 10，MySQL 也会执行回表操作。

[![三分恶面渣逆袭：没有使用 ICP](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202218158.jpeg)](https://camo.githubusercontent.com/9febfd34d840f814a651b33067369d6783f1bc75034aa870f45e0716f18edd71/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d63353866353965302d383530622d346466642d383132392d3264666335316366343736382e6a7067)

有索引下推的情况下，MySQL 可以在存储引擎层检查 `name like '张%' and age=10` 的条件，而不仅仅是 `name like '张%'`。

[![三分恶面渣逆袭：使用 ICP](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202219600.jpeg)](https://camo.githubusercontent.com/476d1e6464224c39880ae5ec628682c455b2eec7e86ac5970307111b44d6ca96/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d61383532356366332d326431362d343961392d613764612d6131393736326564313664662e6a7067)

这就意味着不符合 age = 10 条件的记录将会在索引扫描时被过滤掉，从而**减少了回表的次数**。





### 如何查看是否用到了索引？

可以通过 `EXPLAIN` 关键字来查看是否使用了索引。

```mysql
EXPLAIN SELECT * FROM table WHERE column = 'value';
```

其结果中的 **key 值**显示了查询是否使用索引，如果使用了索引，会显示索引的名称。





## :star:锁

### MySQL 中有哪几种锁，列举一下？

[
![三分恶面渣逆袭：MySQL 中的锁](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202224780.jpeg)](https://camo.githubusercontent.com/7b4ad56e0ba3d145b18771908e1e317f5a5dd98608a60b684dfd501298408639/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d61303765343532352d636363312d343238372d616563352d6562663366323737383537632e6a7067)

**按锁粒度划分的话，MySQL 的锁有：**

| **特性**         | **表锁**                                | **页锁**                                         | **行锁**                                 |
| ---------------- | --------------------------------------- | ------------------------------------------------ | ---------------------------------------- |
| **锁的粒度**     | 锁定整张表                              | 锁定一页数据（通常是 16KB 或 8KB）               | 锁定单行数据                             |
| **并发性**       | 并发性最差，其他线程/事务无法访问整个表 | 并发性较差，锁定的页中的其他行可能被其他事务访问 | 并发性较好，其他事务可以访问不受锁定的行 |
| **性能影响**     | 对于频繁操作的表，性能差，容易引起阻塞  | 性能比表锁好，但不如行锁灵活                     | 性能较好，适用于高并发的场景             |
| **锁的开销**     | 锁定整个表，开销较大                    | 锁定一页的多个行，开销适中                       | 锁定具体行，开销较小                     |
| **适用场景**     | 小表、少并发、简单查询或更新操作        | 大数据量、行数据密集、需要提高性能的查询         | 高并发环境、大量并发修改单行数据         |
| **常见数据库**   | MySQL（MyISAM 存储引擎）、SQLite        | MySQL（InnoDB 存储引擎）、SQL Server             | MySQL（InnoDB 存储引擎）、PostgreSQL     |
| **锁的粒度控制** | 无法细粒度控制                          | 锁定一页数据，可以锁定多行数据                   | 精细控制，锁定单行数据                   |
| **死锁可能性**   | 低，通常不会发生死锁（因锁定整个表）    | 死锁风险介于行锁和表锁之间                       | 高，多个事务同时修改不同的行时容易死锁   |

**按兼容性划分的话，MySQL 的锁有：**

- **共享锁**（S Lock），也叫**读锁**（read lock），**相互不阻塞。**
- **排他锁**（X Lock），也叫**写锁**（write lock），排它锁是**阻塞**的，在一定时间内，只有一个请求能执行写入，并阻止其它锁读取正在写入的数据。

**按加锁机制划分的话，MySQL 的锁有：**

①、**乐观锁**

乐观锁基于这样的假设：冲突在系统中出现的频率较低，因此在数据库事务执行过程中，不会频繁地去锁定资源。相反，它在提交更新的时候才检查是否有其他事务已经修改了数据。

可以通过在数据表中使用版本号（Version）或时间戳（Timestamp）来实现，每次读取记录时，同时获取版本号或时间戳，更新时检查版本号或时间戳是否发生变化。

如果没有变化，则执行更新并增加版本号或更新时间戳；如果检测到冲突（即版本号或时间戳与之前读取的不同），则拒绝更新。

②、**悲观锁**

悲观锁假设冲突是常见的，因此在数据处理过程中，它会主动锁定数据，防止其他事务进行修改。

可以直接使用数据库的锁机制，如行锁或表锁，来锁定被访问的数据。常见的实现是 `SELECT FOR UPDATE` 语句，它在读取数据时就加上了锁，直到当前事务提交或回滚后才释放。



### 全局锁和表级锁了解吗？

全局锁就是对整个数据库实例进行加锁，在 MySQL 中，可以使用 `FLUSH TABLES WITH READ LOCK` 命令来获取全局读锁。

全局锁的作用是保证在**备份数据库时**，数据不会发生变化【数据更新语句（增删改）、数据定义语句（建表、修改表结构等）和更新事务的提交语句】。当我们需要备份数据库时，可以**先获取全局读锁，然后再执行备份操作。**



表锁（Table Lock）就是锁住整个表。在 MySQL 中，可以使用 `LOCK TABLES` 命令来锁定表。

表锁可以分为读锁（共享锁）和写锁（排他锁）。

```mysql
LOCK TABLES your_table READ;
-- 执行读操作
UNLOCK TABLES;
```

读锁允许多个事务同时读取被锁定的表，但不允许任何事务进行写操作。

```mysql
LOCK TABLES your_table WRITE;
-- 执行写操作
UNLOCK TABLES;
```

写锁允许一个事务对表进行读写操作，其他事务不能对该表进行任何操作（读或写）。

在进行**大规模的数据导入、导出或删除操作时**，为了防止其他事务对数据进行并发操作，可以使用表锁。

或者在**进行表结构变更（如添加列、修改列类型）时**，为了确保变更期间没有其他事务访问或修改该表，可以使用表锁。



### 说说 MySQL 的行锁？

行级锁（Row Lock）是数据库锁机制中最细粒度的锁，主要用于**对单行数据进行加锁**，以确保数据的一致性和完整性。在高并发环境中，行级锁能够**提高系统的并发性能**，因为锁定的粒度较小，只会锁住特定的行，不会影响其他行的操作。

通过 `SELECT ... FOR UPDATE` 可以加排他锁，通过 `LOCK IN SHARE MODE` 可以加共享锁。

#### select for update 加锁有什么需要注意的？

- 如果**查询条件使用了索引（特别是主键索引或唯一索引），SELECT FOR UPDATE 会锁定特定的行，即行级锁**，这样锁的粒度较小，不会影响未涉及的行或其他并发操作。
- 但**如果查询条件未使用索引，SELECT FOR UPDATE 可能锁定整个表或大量的行，因为查询需要执行全表扫描。**



#### 说说 InnoDB 的行锁实现？

1. **Record Lock 记录锁**

   记录锁就是直接锁定某行记录。当我们**使用唯一性的索引(包括唯一索引和聚簇索引)进行等值查询且精准匹配到一条记录时**，此时就会直接将这条记录锁定。例如`select * from t where id =6 for update;`就会将`id=6`的记录锁定。

   [![三分恶面渣逆袭：记录锁](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202245948.jpeg)](https://camo.githubusercontent.com/206afb093d7d500933b2a37497d785f638ba8438907f24cec62ed31d1e6e5dbe/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d38393839616332372d653434322d346331342d383161642d3662633133336437386266642e6a7067)

2. **Gap Lock 间隙锁**

   间隙锁(Gap Locks) 的间隙指的是两个记录之间逻辑上尚未填⼊数据的部分,是⼀个左开右开空间。

   ![image-20241220224828762](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202248935.png)

   间隙锁就是锁定某些间隙区间的。当我们**使用等值查询或者范围查询，并且没有命中任何一 个record** ，此时就会将对应的间隙区间锁定。例如 select * from t where id =3 for update; 或 者select * fr om t where id > 1 and id < 6 for update; 就会将(1,6)区间锁定。

3. **Next-key Lock 临键锁**

   临键指的是间隙加上它右边的记录组成的**左开右闭区间**。比如上述的`(1,6]、(6,8]`等。

   [![三分恶面渣逆袭：临键锁](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202245335.jpeg)](https://camo.githubusercontent.com/5569c021407783c5a4a0fb92d237ad92751338f8d56f4d499e0574dae09bc904/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d61653861323163632d386235322d343637642d393137332d3465303162323465303462392e6a7067)

   **临键锁就是记录锁(Record Locks)和间隙锁(Gap Locks)的结合**，即除了锁住记录本身，还要再锁住索引之间的间隙。当我们**使用范围查询，并且命中了部分`record`记录时**，此时锁住的就是临键区间。

   注意，临键锁锁住的区间会包含最后一个 record 的右边的临键区间。

   例如 `select * from t where id > 5 and id <= 7 for update;` 会锁住`(4,7]、(7,+∞)`。

   **MySQL 默认行锁类型就是`临键锁(Next-Key Locks)`**。**当使用唯一性索引，等值查询匹配到一条记录的时候，临键锁(Next-Key Locks)会退化成记录锁；没有匹配到任何记录的时候，退化成间隙锁。**

   `间隙锁(Gap Locks)`和`临键锁(Next-Key Locks)`都是用来**解决幻读**问题的，**在`已提交读（READ COMMITTED）`隔离级别下，`间隙锁(Gap Locks)`和`临键锁(Next-Key Locks)`都会失效！**

   上面是行锁的三种实现算法，除此之外，在行上还存在插入意向锁。

4. **Insert Intention Lock 插入意向锁**

   一个事务**在插入一条记录时需要判断一下插入位置是不是被别的事务加了意向锁** ，如果有的话，插入操作需要等待，直到拥有 gap 锁 的那个事务提交。但是事务在等待的时候也需要在内存中生成一个 锁结构 ，表明有事务想在某个 间隙 中插入新记录，但是现在在等待。这种类型的锁命名为 Insert Intention Locks ，也就是插入意向锁 。

   假如我们有个 T1 事务，给(1,6)区间加上了意向锁，现在有个 T2 事务，要插入一个数据，id 为 4，它会获取一个（1,6）区间的插入意向锁，又有有个 T3 事务，想要插入一个数据，id 为 3，它也会获取一个（1,6）区间的插入意向锁，但是，这两个插入意向锁锁不会互斥。

   [![三分恶面渣逆袭：插入意向锁](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202245171.jpeg)](https://camo.githubusercontent.com/c41589b7abe714947c37db0ca5457310cdd87ae5890ad6f0a210da60b3123d0a/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d37353134323563622d646162612d346461312d626162362d6638343332353463616433642e6a7067)





### 意向锁是什么知道吗？

**意向锁是一个表级锁，不要和插入意向锁搞混。**

意向锁的出现是为了支持 InnoDB 的**多粒度锁**，它解决的是**表锁和行锁共存的问题**。

当我们需要给一个表加表锁的时候，我们需要根据去判断表中有没有数据行被锁定，以确定是否能加成功。

假如没有意向锁，那么我们就得遍历表中所有数据行来判断有没有行锁；

有了意向锁这个表级锁之后，则我们**直接判断一次就知道表中是否有数据行被锁定了**。

有了意向锁之后，要执行的事务 A 在申请行锁（写锁）之前，数据库会自动先给事务 A 申请表的意向排他锁。当事务 B 去申请表的互斥锁时就会失败，因为表上有意向排他锁之后事务 B 申请表的互斥锁时会被阻塞。

[![意向锁](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202303827.jpeg)](https://camo.githubusercontent.com/d1e6901e07e8c8b4b08f97a7e9110a3821f4a6177f9c03d9dfcd4a25d5f34532/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d33316637663439632d316535612d346434322d623862332d6530323262336261383261652e6a7067)



### MySQL 的乐观锁和悲观锁了解吗？

MySQL 中的行锁、表锁都是悲观锁。

悲观锁是 MySQL 自带的，而乐观锁通常需要开发者自己去实现。



### 遇到过死锁问题吗，你是如何解决的？

两个事务分别更新两张表，但是更新顺序不一致，导致了死锁。

第一步，使用 `SHOW ENGINE INNODB STATUS\G;` 查看死锁信息。

第二步，调整事务的资源访问顺序，保持一致。





## 事务

### MySQL 事务的四大特性说一下？

- **A 原子性**：事务的所有操作要么全部提交成功，要么全部失败回滚，对于一个事务中的操作不能只执行其中一部分。
- **C 一致性**：事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致性与业务规则有关，比如银行转账，不论事务成功还是失败，转账双方的总金额应该是不变的。
- **I 隔离性**：多个并发事务之间需要相互隔离，即一个事务的执行不能被其他事务干扰。
- **D 持久性**：一旦事务提交，则其所做的修改将永久保存到数据库中。即使发生系统崩溃，修改的数据也不会丢失。



### ACID 靠什么保证的呢？

MySQL 通过事务、undo log、redo log 来确保 ACID。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202306870.png" alt="二哥的 Java 进阶之路：ACID 的保证机制" style="zoom:50%;" />](https://camo.githubusercontent.com/f87ff0d551da3a993c955b20953c95893d07caed8447077085408c4a2b2d4276/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303233303931393130333032352e706e67)

#### 如何保证原子性？

MySQL 通过 undo log 来确保原子性（Atomicity）。

当事务开始时，MySQL 会在`undo log`中记录事务开始前的旧值。如果事务执行失败，MySQL 会使用`undo log`中的旧值来回滚事务开始前的状态；如果事务执行成功，MySQL 会在某个时间节点将`undo log`删除。

#### 如何保证一致性？

如果其他三个特性都得到了保证，那么一致性就自然而然得到保证了。

#### 如何保证隔离性？

MySQL 定义了多种隔离级别，通过 MVCC 来确保每个事务都有专属自己的数据版本，从而实现隔离性（Isolation）。

在 MVCC 中，每行记录都有一个版本号，当事务尝试读取记录时，会根据事务的隔离级别和记录的版本号来决定是否可以读取。

#### 如何保证持久性？

redo log 是一种物理日志，当执行写操作时，MySQL 会先将更改记录到 redo log 中。当 redo log 填满时，MySQL 再将这些更改写入数据文件中。

如果 MySQL 在写入数据文件时发生崩溃，可以通过 redo log 来恢复数据文件，从而确保持久性（Durability）。



#### 事务会不会自动提交？

在 MySQL 中，默认情况下事务是 自动提交 的。每执行一条 SQL 语句（如 INSERT、UPDATE），都会被当作一个事务自动提交。

如果需要手动控制事务，可以使用 START TRANSACTION 开启事务，并通过 COMMIT 或 ROLLBACK 完成事务。



### :star:事务的隔离级别有哪些？

MySQL 支持四种隔离级别，分别是：读未提交、读已提交、可重复读和串行化。

[![三分恶面渣逆袭：事务的四个隔离级别](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202307876.jpeg)](https://camo.githubusercontent.com/56912b0714853134193090ba617985bbbbdd37c41e81e0c0c3d3706fff92382b/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d39393934323532392d346139312d343230622d396365322d3431343965373437663634642e6a7067)

| 隔离级别     | 脏读   | 不可重复读 | 幻读   | 特点                                                         |
| ------------ | ------ | ---------- | ------ | ------------------------------------------------------------ |
| **读未提交** | 允许   | 允许       | 允许   | 当前事务可以读取未被其他事务提交的数据                       |
| **读已提交** | 不允许 | 允许       | 允许   | 只能读取已经被其他事务提交的数据                             |
| **可重复读** | 不允许 | 不允许     | 允许   | 一个事务在执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的。 |
| **串行化**   | 不允许 | 不允许     | 不允许 | 强制事务串行执行                                             |



#### A 事务未提交，B 事务上查询到的是旧值还是新值？

在 **MySQL 的默认隔离级别（可重复读）**下，如果事务 A 修改了数据但未提交，事务 B 将看到修改之前的数据。

这是因为在可重复读隔离级别下，MySQL 将通过多版本并发控制（MVCC）机制来保证一个事务不会看到其他事务未提交的数据，从而确保读一致性。

#### 怎么更改事务的隔离级别？

使用 `SET SESSION TRANSACTION ISOLATION LEVEL` 可以修改当前连接的隔离级别，只影响当前会话。

使用 `SET GLOBAL TRANSACTION ISOLATION LEVEL` 可以修改全局隔离级别，影响新的连接，但不会改变现有会话。



### 什么是脏读、不可重复读、幻读呢？

- 脏读指的是**一个事务能够读取另一个事务尚未提交的数据**。如果读到的数据在之后被回滚了，那么第一个事务读取到的就是无效的数据。
- 不可重复读指的是在**同一事务中执行相同的查询时，返回的结果集不同**。这是由于在事务过程中，另一个事务修改了数据并提交。
- 幻读指的是在**同一事务中执行相同的查询时，返回的结果集中出现了之前没有的数据行**。这是因为在事务执行过程中，另外一个事务插入了新的数据。



#### 如何避免幻读？

①、直接使用最高的隔离级别串行化，但会导致大量的锁竞争。

②、使用**间隙锁**，防止其他事务在间隙范围内插入数据。

③、在可重复读的隔离级别下，MVCC 机制会为**每个事务维护一个快照**，事务在读取数据时，只能读取到快照中的数据，而不会读取到其他事务插入的数据。



#### 可重复读级别怎么保证不出现幻读

ChatGPT 说：

在 **可重复读（REPEATABLE READ）** 隔离级别下，MySQL 主要依赖 **MVCC（多版本并发控制）** 和 **Next-Key Lock（间隙锁）** 机制来防止 **幻读（Phantom Read）**，从而确保事务的串行化。下面详细分析：

------

##### **1. 幻读是什么？**

**幻读** 指的是在同一个事务中，多次执行相同的查询语句时，结果集却不同。
 例如：

1. 事务 **T1** 读取 `salary > 5000` 的所有记录，假设返回 5 条数据。
2. 事务 **T2** **插入** 一条 `salary = 6000` 的新记录，并 **提交**。
3. 事务 **T1** 再次执行相同查询时，结果集变成了 6 条数据，这就发生了**幻读**。

------

##### **2. 可重复读如何防止幻读？**

在 **InnoDB** 存储引擎中，**可重复读（REPEATABLE READ）** 主要依靠 **MVCC + 间隙锁（Next-Key Lock）** 来防止幻读。

##### **(1) MVCC 机制**

MVCC 通过**快照读（Snapshot Read）**，保证同一事务中的**所有查询都基于事务开始时的一致性视图**。这意味着：

- 事务 T1 在开始时，会创建一个一致性快照。
- 任何其他事务的**新插入数据**，对于 T1 来说都不可见。

🔹 **MVCC 可以防止 `SELECT` 语句的幻读**，但它无法阻止 `UPDATE` 或 `DELETE` 时的幻读。例如：

```
sql


复制编辑
SELECT * FROM employees WHERE salary > 5000;
```

- 即使事务 T2 **插入** 一条 `salary = 6000` 的记录，事务 T1 仍然看不到（因为 MVCC 只会读取事务开始时的快照）。
- 但如果 T1 执行 `UPDATE employees SET salary = salary + 1000 WHERE salary > 5000;`，就可能影响 T2 插入的数据，导致幻读问题。

##### **(2) Next-Key Lock（间隙锁 + 行锁）**

为了防止 `UPDATE` 或 `DELETE` 时的幻读，**InnoDB** 在 **可重复读** 隔离级别下，会使用 **Next-Key Lock**，它结合了：

- **记录锁（Record Lock）**：锁住查询到的**行数据**。
- **间隙锁（Gap Lock）**：锁住索引之间的**间隙**，防止其他事务在该范围内插入新数据。

📌 **示例**（防止幻读的过程）：

```
sql复制编辑-- 事务 T1
START TRANSACTION;
SELECT * FROM employees WHERE salary > 5000 FOR UPDATE;
```

**事务 T1 执行 `SELECT ... FOR UPDATE`：**

- 它会对 `salary > 5000` 的所有记录加上**记录锁**。
- 同时，它会对 `salary > 5000` 的**索引区间**加上**间隙锁**，防止事务 T2 在这个范围内插入新数据。

**如果事务 T2 试图插入：**

```
sql复制编辑-- 事务 T2
INSERT INTO employees (id, name, salary) VALUES (10, 'Jack', 6000);
```

MySQL 发现 **T1 已经锁住了 `salary > 5000` 的间隙**，T2 必须等待 T1 提交或回滚后才能执行插入操作，从而避免了幻读。

------

##### **3. 可重复读是否完全等价于串行化？**

**可重复读 ≠ 串行化（Serializable）**，但在 MySQL 的 **InnoDB** 存储引擎下，由于 **Next-Key Lock 机制**，可重复读在 **大部分情况下能达到事务串行化的效果**，只是在某些情况下（如非索引列的查询）仍可能存在一定的并发影响。

##### **可重复读 vs 串行化**

| 隔离级别               | MVCC | 记录锁 | 间隙锁 | 幻读     |
| ---------------------- | ---- | ------ | ------ | -------- |
| 读已提交（RC）         | ✅    | ✅      | ❌      | 可能出现 |
| 可重复读（RR）         | ✅    | ✅      | ✅      | 避免     |
| 串行化（SERIALIZABLE） | ❌    | ✅      | ✅      | 完全避免 |

- 在 **串行化** 隔离级别下，每个事务会强制使用**表锁**，完全阻止并发访问，但性能较低。
- **可重复读（RR）** 通过 **Next-Key Lock** 在保证性能的同时，避免了**大多数情况**的幻读。

------

##### **4. 关键总结**

1. **MVCC 通过快照读防止 `SELECT` 发生幻读**，但无法阻止 `UPDATE/DELETE` 产生幻读。
2. **Next-Key Lock（间隙锁 + 记录锁）防止 `UPDATE`/`DELETE` 发生幻读**，确保事务间的可串行化执行。
3. **MySQL 的可重复读（RR）能避免幻读**，但在某些情况下仍可能不如 `SERIALIZABLE` 彻底。

如果你的业务允许较高并发，但又希望防止幻读，**可重复读（RR）+ 显式加锁（SELECT ... FOR UPDATE）** 是一个很好的方案。





### 事务的隔离级别是如何实现的？

- 读未提交**不提供任何锁机制**来保护读取的数据，允许读取未提交的数据（即脏读）。

- 读已提交：**每次读取数据前**都生成一个**MVCC 机制中 ReadView**，保证每次读操作都是最新的数据。
- 可重复读：**只在第一次读**操作时生成一个**MVCC 机制中 ReadView**，后续读操作都使用这个 ReadView，保证事务内读取的数据是一致的。

- 串行化级别下，事务在**读操作时，必须先加表级共享锁**，直到事务结束才释放；事务在**写操作时，必须先加表级排他锁**，直到事务结束才释放。





### MVCC 了解吗？怎么实现的？

MVCC 指的是多版本并发控制，简单来说，就是给我们的 MySQL 数据拍个“**快照**”

在 MySQL 中，MVCC 是通过**版本链和 ReadView 机制**来实现的。

[![天瑕：undo log 版本链和 ReadView](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202313183.png)](https://camo.githubusercontent.com/c373475d66aec437908aa997a9c86a9fd2b96ac4c1877793af056cb00b1ac79d/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234313231353037333833372e706e67)

1. 每条记录包含两个隐藏列：最近修改的事务 ID 和指向 Undo Log 的指针，用于构成版本链。
2. 每次更新数据时，会生成一个新的数据版本，并将旧版本的数据保存到 Undo Log 中。
3. 每次读取数据时，会生成一个 ReadView，用于判断哪个版本的数据对当前事务可见。



#### 什么是版本链？

在 InnoDB 中，每一行数据都有两个隐藏的列：一个是 **DB_TRX_ID**，另一个是 **DB_ROLL_PTR**。

- `DB_TRX_ID`，保存创建这个版本的**事务 ID**。
- `DB_ROLL_PTR`，**指向 undo 日志记录的指针**，这个记录包含了该行的前一个版本的信息。通过这个指针，可以访问到该行数据的历史版本。



当事务更新一行数据时，**InnoDB 不会直接覆盖原有数据，而是创建一个新的数据版本，并更新 DB_TRX_ID 和 DB_ROLL_PTR，使得它们指向前一个版本和相关的 undo 日志**。这样，老版本的数据不会丢失，可以通过版本链找到。

由于 undo 日志会记录每一次的 update，并且新插入的行数据会记录上一条 undo 日志的指针，所以可以通过这个指针找到上一条记录，这样就形成了一个版本链。

[![三分恶面渣逆袭：版本链](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202336292.jpeg)](https://camo.githubusercontent.com/9c965f600cff271f6a18bb21b7c1e3003283059819f3daba3e9c0bb61f5417ab/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d37363562336438332d313465622d346235362d383934302d3964363062666166313733372e6a7067)

#### 说说什么是 ReadView？

ReadView（读视图）是 InnoDB 为了实现一致性读而创建的数据结构，它用于确定在特定事务中哪些版本的行记录是可见的。

ReadView 主要用来处理隔离级别为"可重复读"和"读已提交"的情况。因为在这两个隔离级别下，事务在读取数据时，需要保证读取到的数据是一致的，即读取到的数据是在事务开始时的一个快照。

[![二哥的 Java 进阶之路：ReadView](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202337072.png)](https://camo.githubusercontent.com/1df361a4c59eca9cc88db56d96a2be8e3cbd924b3d500ed1a42805d9208c3412/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303431353039333730332e706e67)

当事务开始执行时，InnoDB 会为该事务创建一个 ReadView，这个 ReadView 会记录 4 个重要的信息：

- creator_trx_id：创建该 ReadView 的事务 ID。
- m_ids：所有活跃事务的 ID 列表，活跃事务是指那些已经开始但尚未提交的事务。
- min_trx_id：所有活跃事务中最小的事务 ID。它是 m_ids 数组中最小的事务 ID。
- max_trx_id ：事务 ID 的最大值加一。换句话说，它是下一个将要生成的事务 ID。

#### ReadView 是如何判断记录的某个版本是否可见的？

[![二哥的 Java 进阶之路：](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202337137.png)](https://camo.githubusercontent.com/ed80612431fd414dad0dbb42a4c20f157e977e8d1ffdd92fa8d96733b0e535e0/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d7973716c2d32303234303431353039343933392e706e67)

读事务开启了一个 ReadView，这个 ReadView 里面记录了当前活跃事务的 ID 列表（444、555、665），以及最小事务 ID（444）和最大事务 ID（666）。当然还有自己的事务 ID 520，也就是 creator_trx_id。

它要读的这行数据的写事务 ID 是 x，也就是 DB_TRX_ID。

- 如果 x = 110，显然在 ReadView 生成之前就提交了，所以这行数据是可见的。
- 如果 x = 667，显然是未知世界，所以这行数据对读操作是不可见的。
- 如果 x = 519，虽然 519 大于 444 小于 666，但是 519 不在活跃事务列表里，所以这行数据是可见的。因为 519 是在 520 生成 ReadView 之前就提交了。
- 如果 x = 555，虽然 555 大于 444 小于 666，但是 555 在活跃事务列表里，所以这行数据是不可见的。因为 555 不确定有没有提交。





#### 可重复读和读已提交在 ReadView 上的区别是什么？

区别在于生成 ReadView 的时机不同。

- 可重复读：在**第一次读取数据时**生成一个 ReadView，这个 ReadView 会一直保持到事务结束，这样可以保证在事务中多次读取同一行数据时，读取到的数据是一致的。
- 读已提交：**每次读取数据前**都生成一个 ReadView，这样就能保证每次读取的数据都是最新的。



#### 如果两个 AB 事务并发修改一个变量，那么 A 读到的值是什么，怎么分析。

当两个事务 A 和 B 并发修改同一个变量时，A 事务读取到的值取决于多个因素，包括事务的隔离级别、事务的开始时间和提交时间等。

- 读未提交：在这个级别下，事务可以看到其他事务尚未提交的更改。如果 B 更改了一个变量但尚未提交，A 可以读到这个更改的值。
- 读提交：A 只能看到 B 提交后的更改。如果 B 还没提交，A 将看到更改前的值。
- 可重复读：在事务开始后，A 总是读取到变量的相同值，即使 B 在这期间提交了更改。这是通过 MVCC 机制实现的。
- 可串行化：A 和 B 的操作是串行执行的，如果 A 先执行，那么 A 读到的值就是 B 提交前的值；如果 B 先执行，那么 A 读到的值就是 B 提交后的值。







## 高可用/性能

### 数据库读写分离了解吗？

读写分离的基本原理是将数据库读写操作分散到不同的节点上，下面是基本架构图：

[![读写分离](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202341802.jpeg)](https://camo.githubusercontent.com/53e2a88a64615d48ac0627be3fe2c99e5e09fa70ec2ffc93b80d141a6227b4cd/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d33316466373637632d646230352d346465342d613035622d6134356263663736633162662e6a7067)

读写分离的基本实现是:

- 数据库服务器搭建主从集群，一主一从、一主多从都可以。
- 数据库**主机负责读写操作，从机只负责读操作**。
- 数据库主机通过复制将数据同步到从机，每台数据库服务器都存储了所有的业务数据。
- 业务服务器将写操作发给数据库主机，将读操作发给数据库从机。



### 读写分离的分配怎么实现呢？

一般有两种方式：程序代码封装和中间件封装。

1. 程序代码封装

程序代码封装指在代码中抽象一个**数据访问层**（所以有的文章也称这种方式为 "中间层封装" ） ，实现读写操作分离和数据库服务器连接的管理。例如，基于 Hibernate 进行简单封装，就可以实现读写分离：

![业务代码封装](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202345371.jpeg)

目前开源的实现方案中，淘宝的 TDDL （Taobao Distributed Data Layer, 外号：头都大了）是比较有名的。



​    2.中间件封装

中间件封装指的是独立一套系统出来，实现读写操作分离和数据库服务器连接的管理。中间件对业务服务器提供 SQL 兼容的协议，业务服务器无须自己进行读写分离。

对于业务服务器来说，访问中间件和访问数据库没有区别，事实上在业务服务器看来，中间件就是一个数据库服务器。

其基本架构是：

[![数据库中间件](https://camo.githubusercontent.com/3efd1a5ddc3e75cae0bb9658c0ea6e1f249c10cded38bf830b921906da562950/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d66323331333631332d323562642d343036352d386636332d3936396134623537353761372e6a7067)](https://camo.githubusercontent.com/3efd1a5ddc3e75cae0bb9658c0ea6e1f249c10cded38bf830b921906da562950/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d66323331333631332d323562642d343036352d386636332d3936396134623537353761372e6a7067)





### 主从复制原理了解吗？

MySQL 的主从复制（Master-Slave Replication）是一种**数据同步机制**，用于将数据从一个主数据库（master）复制到一个或多个从数据库（slave）。

广泛用于数据备份、灾难恢复和数据分析等场景。

[![三分恶面渣逆袭：主从复制](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202347632.jpeg)](https://camo.githubusercontent.com/df356d5db06da7497d9cb3d15681462150fe7ee037bc0d70405599811e5c3f82/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d31626662666362352d323339322d346639382d626531622d6136363230346461303965352e6a7067)

复制过程的主要步骤有：

- 在主服务器上，所有修改数据的语句（如 INSERT、UPDATE、DELETE）会被记录到**二进制日志**中。
- 主服务器上的一个线程（二进制日志转储线程）负责读取二进制日志的内容并发送给从服务器。
- 从服务器接收到二进制日志数据后，会将这些数据写入自己的**中继日志**（Relay Log）。中继日志是从服务器上的一个本地存储。
- 从服务器上有一个 SQL 线程会读取中继日志，并在本地数据库上执行，从而将更改应用到从数据库中，完成同步。



### 主从同步延迟怎么处理？

**主从同步延迟的原因**

一个服务器开放Ｎ个链接给客户端来连接的，这样有会有大并发的更新操作, 但是**从服务器的里面读取 binlog 的线程仅有一个**，当某个 SQL 在从服务器上执行的时间稍长 或者由于某个 SQL 要进行**锁表**就会导致，**主服务器的 SQL 大量积压**，未被同步到从服务器里。这就导致了主从不一致， 也就是主从延迟。

**主从同步延迟的解决办法**

解决主从复制延迟有几种常见的方法:

1. **写操作后的读操作指定发给数据库主服务器**

   例如，注册账号完成后，登录时读取账号的读操作也发给数据库主服务器。这种方式和业务强绑定，对业务的侵入和影响较大，如果哪个新来的程序员不知道这样写代码，就会导致一个 bug。

2. **读从机失败后再读一次主机**

   这就是通常所说的 "**二次读取**" ，二次读取和业务无绑定，只需要对底层数据库访问的 API 进行封装即可，实现代价较小，不足之处在于如果有很多二次读取，将大大增加主机的读操作压力。例如，黑客暴力破解账号，会导致大量的二次读取操作，主机可能顶不住读操作的压力从而崩溃。

3. **关键业务读写操作全部指向主机，非关键业务采用读写分离**

   例如，对于一个用户管理系统来说，注册 + 登录的业务读写操作全部访问主机，用户的介绍、爰好、等级等业务，可以采用读写分离，因为即使用户改了自己的自我介绍，在查询时却看到了自我介绍还是旧的，业务影响与不能登录相比就小很多，还可以忍受。



### 你们一般是怎么分库的呢？

①、垂直分库：按照业务模块将不同的表拆分到不同的库中，例如，用户表、订单表、商品表等分到不同的库中。

②、水平分库：按照一定的策略将一个表中的数据拆分到多个库中，例如，按照用户 id 的 hash 值将用户表拆分到不同的库中。



### 那你们是怎么分表的？

当单表数据增量过快，比如说单表超过 **500 万条数据**，就可以考虑分表了。

在[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)中，我们将文章表和文章详情表做了分表处理，因为文章的详情数据量会比较大，而文章的基本信息数据量会比较小。

垂直拆分可以减轻只查询文章基本数据，不需要附带文章详情时的查询压力。

当然了，当表的数据量过大时，仍然要考虑水平分表，将一个表的数据分散到多个表中，以减轻单表的查询压力。



### 水平分表有哪几种路由方式？

具体哪个表，由分片键（Sharding Key）来决定，分片键的选择应满足以下条件：

- **高区分度**：分片键的值应尽量均匀分布，以避免数据倾斜。
- **查询频率高**：选择经常在查询条件中使用的字段作为分片键，有助于提高查询效率。
- **写入频率高**：选择经常被写入的字段，可以均匀分布写入负载。

那常见的路由策略有三种，分别是范围路由、Hash 路由和配置路由。

#### 什么是范围路由？

![三分恶面渣逆袭：范围路由](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202352755.jpeg)

范围路由的优点是实现简单，可以随着数据的增加平滑地扩充新的表。适用于按时间或按顺序增长的字段（如时间戳、订单号等）。缺点是可能出现**数据倾斜**问题，导致某些表的数据量明显大于其他表。

#### 什么是 Hash 路由？

哈希路由是通过对分片键进行哈希计算，然后取模来确定数据存储的表。哈希值决定了数据分布，通常能较好地平衡数据量。

[![三分恶面渣逆袭：Hash 路由](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202352190.jpeg)](https://camo.githubusercontent.com/fa9c00d20a7f01964b91305fa5e12bccde6343022050c0b0f403891a304bb686/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d65303165373735372d633333372d343863382d393564622d3266376366643262633033362e6a7067)

哈希路由的优点是数据可以均匀分布，避免了数据倾斜，但范围查询时可能会涉及多个表，**性能较差**。

#### 什么是配置路由？

配置路由是通过配置表来确定数据存储的表，适用于分片键不规律的场景。

[![三分恶面渣逆袭：配置路由](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202353990.jpeg)](https://camo.githubusercontent.com/5f747443d589fbd8bc8b416037623c9f52f376801936d4f8cee72d7ad60c5b32/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d66636433343333322d643338642d343535612d383735642d6434616664333763616337322e6a7067)

配置路由的优点是可以根据实际情况灵活配置。缺点是需要额外的配置表，维护成本较高。





### 不停机扩容怎么实现？

- **第一阶段：在线双写，查询走老库**

1. 建立好新的库表结构，数据写入久库的同时，也写入拆分的新库
2. 数据迁移，使用数据迁移程序，将旧库中的历史数据迁移到新库
3. 使用定时任务，新旧库的数据对比，把差异补齐

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412202353927.jpeg" alt="img" style="zoom:50%;" />](https://camo.githubusercontent.com/799dca57c54ed1ddae160d63b900b2ed9530523d22d182bff747b13cc77dc474/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d32643464393463392d653831362d343766632d393364642d6138333562313331383039392e6a7067)

- **第二阶段：在线双写，查询走新库**

1. 完成了历史数据的同步和校验
2. 把对数据的读切换到新库

[<img src="https://camo.githubusercontent.com/4444c3d8bf0f038b64ce80c454c16f50c7c0cb58c1f5daa76cf3d4d2b862ee92/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d35636630313438362d373263312d346561622d396636652d6131396333313536396634362e6a7067" alt="img" style="zoom:50%;" />](https://camo.githubusercontent.com/4444c3d8bf0f038b64ce80c454c16f50c7c0cb58c1f5daa76cf3d4d2b862ee92/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d35636630313438362d373263312d346561622d396636652d6131396333313536396634362e6a7067)

- **第三阶段：旧库下线**

1. 旧库不再写入新的数据
2. 经过一段时间，确定旧库没有请求之后，就可以下线老库

[<img src="https://camo.githubusercontent.com/6cdb1259f64ed6e5dba398db34f0e40e2318b750fceb882137097abf9a02626c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d61313232643664352d666666322d346363642d386464622d6139323832656232653264612e6a7067" alt="img" style="zoom:50%;" />](https://camo.githubusercontent.com/6cdb1259f64ed6e5dba398db34f0e40e2318b750fceb882137097abf9a02626c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7973716c2d61313232643664352d666666322d346363642d386464622d6139323832656232653264612e6a7067)

### 怎么分库分表？

如果表的字段过多，我会按字段的访问频率或功能相关性拆分成多个表，减少单表宽度。

如果单表数据量过大，我会按 ID 或时间将数据分到多张表中。比如订单表可以按 `user_id % N` 进行水平分表。

如果业务模块较多，我会将不同模块的表分布到不同的数据库中，比如用户相关的表放在一个库，订单相关的表放在另一个库。

如果单库数据量过大，我会按用户 ID 范围或哈希值将数据分布到多个库中。



#### 常用的分库分表中间件有哪些？

常用的分库分表中间件有 Sharding-JDBC 和 Mycat。

①、Sharding-JDBC

主要在 Java 的 JDBC 层提供额外的服务。无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。

②、Mycat 

可以把它看作一个数据库代理，其核心功能是分表分库，即将一个大表切片为多个小表，一个大库切片成多个小库。



### 你觉得分库分表会带来什么问题呢？

从分库的角度来讲：

- **事务的问题**

使用关系型数据库，有很大一点在于它保证事务完整性。

而**分库之后单机事务就用不上了，必须使用分布式事务来解决**。

- **跨库 JOIN 问题**

在一个库中的时候我们还可以利用 JOIN 来连表查询，而**跨库了之后就无法使用 JOIN 了**。

此时的解决方案就是**在业务代码中进行关联**，也就是先把一个表的数据查出来，然后通过得到的结果再去查另一张表，然后利用代码来关联得到最终的结果。

这种方式实现起来稍微比较复杂，不过也是可以接受的。

还有可以**适当的冗余一些字段**。比如以前的表就存储一个关联 ID，但是业务时常要求返回对应的 Name 或者其他字段。这时候就可以把这些字段冗余到当前表中，来去除需要关联的操作。

还有一种方式就是**数据异构**，通过 binlog 同步等方式，把需要跨库 join 的数据异构到 ES 等存储结构中，通过 ES 进行查询。



从分表的角度来看：

- **跨节点的 count,order by,group by 以及聚合函数问题**

只能由业务代码来实现或者**用中间件将各表中的数据汇总、排序、分页然后返回**。

- **数据迁移，容量规划，扩容等问题**

数据的迁移，容量如何规划，未来是否可能再次需要扩容，等等，都是需要考虑的问题。

- **ID 问题**

数据库表被切分后，不能再依赖数据库自身的主键生成机制，所以需要一些手段来保证全局主键唯一。

1. 还是自增，只不过自增步长设置一下。比如现在有三张表，步长设置为 3，三张表 ID 初始值分别是 1、2、3。这样第一张表的 ID 增长是 1、4、7。第二张表是 2、5、8。第三张表是 3、6、9，这样就不会重复了。
2. UUID，这种最简单，但是不连续的主键插入会导致严重的页分裂，性能比较差。
3. 分布式 ID，比较出名的就是 Twitter 开源的 sonwflake 雪花算法
4. leaf 美团 [Leaf——美团点评分布式ID生成系统 - 美团技术团队](https://tech.meituan.com/2017/04/21/mt-leaf.html)



## 运维

### 百万级别以上的数据如何删除？

在我们删除数据库百万级别数据的时候，查询 MySQL 官方手册得知删除数据的速度和创建的索引数量是成正比的。

1. 所以我们想要删除百万数据的时候可以**先删除索引**
2. 然后**删除其中无用数据**
3. 删除完成后**重新创建索引**创建索引也非常快



### 百万千万级大表如何添加字段？

当线上的数据库数据量到达几百万、上千万的时候，加一个字段就没那么简单，因为可能会长时间锁表。

大表添加字段，通常有这些做法：

1. **通过中间表转换过去**

   创建一个临时的新表，把旧表的结构完全复制过去，添加字段，再把旧表数据复制过去，删除旧表，新表命名为旧表的名称，这种方式可能回丢掉一些数据。

2. **用 pt-online-schema-change**

   `pt-online-schema-change`是 percona 公司开发的一个工具，它可以在线修改表结构，它的原理也是通过中间表。

3. **先在从库添加 再进行主从切换**

   如果一张表数据量大且是热表（读写特别频繁），则可以考虑先在从库添加，再进行主从切换，切换后再将其他几个节点上添加字段。



### MySQL cpu 飙升的话，要怎么处理呢？

排查过程：

（1）使用 top 命令观察，确定是 mysqld 导致还是其他原因。

（2）如果是 mysqld 导致的，show processlist，查看 session 情况，确定是不是有消耗资源的 sql 在运行。

（3）找出消耗高的 sql，**看看执行计划是否准确， 索引是否缺失，数据量是否太大**。

处理：

（1）kill 掉这些线程 (同时观察 cpu 使用率是否下降)，

（2）进行相应的调整 (比如说**加索引、改 sql、改内存参数**)

（3）重新跑这些 SQL。

其他情况：

也有可能是每个 sql 消耗资源并不多，但是突然之间，有大量的 session 连进来导致 cpu 飙升，这种情况就需要跟应用一起来分析为何连接数会激增，再做出相应的调整，比如说**限制连接数**等





