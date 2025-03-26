# MyBatis

## 基础

### 说说什么是 MyBatis?

**先吹一下**：

- Mybatis 是一个**半 ORM**（对象关系映射）框架，它**内部封装了 JDBC**，开发时只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。程序员直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高。
- MyBatis 可以使用 **XML 或注解来配置**和映射原生信息，将 POJO 映射成数据库中的记录，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。

**再说一下缺点**

- SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求
- SQL 语句依赖于数据库，导致**数据库移植性差**，不能随意更换数据库



#### ORM 是什么?

[![ORM简单示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412210949927.png)](https://camo.githubusercontent.com/59293a5597b0b9b82260025f7efddfed7e2e9eb58fb7216844787ed51cc6f5fd/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d65613231323835302d353665302d346431322d393866622d3033626234303030376634342e706e67)

- ORM（Object Relational Mapping），**对象关系映射**，是一种为了解决关系型数据库数据与简单 Java 对象（POJO）的映射关系的技术。简单来说，ORM 是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系型数据库中。



#### 为什么说 Mybatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？

- **Hibernate 属于全自动 ORM 映射工具**，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。
- 而 Mybatis 在查询关联对象或关联集合对象时，**需要手动编写 SQL** 来完成，所以，被称之为半自动 ORM 映射工具。





### Hibernate 和 MyBatis 有什么区别？

以下是 **Hibernate** 和 **MyBatis** 的主要区别，以表格形式展示：

| **特性**         | **MyBatis**                                                  | **Hibernate**                                                |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **映射关系**     | **半自动映射**，手动配置 Java 对象与 SQL 结果的对应关系。多表关联配置简单。 | **全自动映射**，配置 Java 对象与数据库表的对应关系。多表关联配置复杂。 |
| **SQL 控制**     | **手动编写 SQL**，灵活性高，支持**动态 SQL**、列表处理、动态表名、存储过程。 | **自动生成 SQL**，支持 **HQL**，灵活性较低，复杂 SQL 优化较困难。 |
| **数据库移植性** | 不支持数据库无关性，SQL 与数据库耦合，**移植性较差。**       | 数据库无关性强，适合支**持多种数据库**，但会增加性能消耗。   |
| **特性支持**     | **缺少内置日志、缓存和级联**等高级特性，需自行实现或依赖外部工具。 | 内置日志、缓存、级联操作（更强大），增强数据库交互体验。     |
| **适用场景**     | 需求变化频繁、快速迭代的项目，如电商网站、大数据量处理。     | 需求相对稳定、中小型项目，如办公自动化系统。                 |
| **开发工作量**   | 手动编写 SQL，开发量大，SQL 优化容易。                       | 减少 SQL 编写量，开发量小，但 SQL 优化困难。                 |
| **性能**         | 性能优异，SQL 优化可控，适合复杂查询和高性能需求场景。       | 对于简单操作性能较好，复杂查询时性能可能受限制。             |



### MyBatis 使用过程？生命周期？

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412210955150.png" alt="image-20241221095502087" style="zoom:50%;" />

SqlSession（会话）可以理解为程序和数据库之间的桥梁

如果是更新、删除语句，我们还需要提交一下事务。

最后一定要记得关闭会话。



#### 说说 MyBatis 生命周期？

- **SqlSessionFactoryBuilder**

**一旦创建了 SqlSessionFactory，就不再需要它了**。 因此 SqlSessionFactoryBuilder 实例的**生命周期只存在于方法的内部。**

![image-20241221100513659](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211005724.png)

**SqlSession 的实例不是线程安全的**，当然，万物皆可集成 Spring，MyBatis 通常也是和 Spring 集成使用，Spring 可以帮助我们创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到我们的 bean 中，我们不需要关心它们的创建过程和生命周期，那就是另外的故事了。



### 在 mapper 中如何传递多个参数？

![mapper传递多个参数方法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211006907.png)

**方法 1：顺序传参法**

```xml
public User selectUser(String name, int deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{0} and dept_id = #{1}
</select>
```

- `\#{}`里面的**数字代表传入参数的顺序。**
- 这种方法不建议使用，sql 层表达不直观，且一旦顺序调整容易出错。

**方法 2：@Param 注解传参法**

```xml
public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

- `\#{}`里面的名称对应的是注解@Param 括号里面修饰的名称。
- 这种方法在参数不多的情况还是比较直观的，（推荐使用）。

**方法 3：Map 传参法**

```xml
public User selectUser(Map<String, Object> params);

<select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

- `\#{}`里面的名称**对应的是 Map 里面的 key 名称**。
- 这种方法适合传递多个参数，且参数易变能灵活传递的情况。

**方法 4：Java Bean 传参法**:white_check_mark:

```xml
public User selectUser(User user);

<select id="selectUser" parameterType="com.jourwon.pojo.User" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

- `\#{}`里面的名称对应的是 User 类里面的成员属性。
- 这种方法直观，需要建一个实体类，扩展不容易，需要加属性，但代码可读性强，业务逻辑处理方便，推荐使用。（推荐使用）。



### 实体类属性名和表中字段名不一样 ，怎么办?

- 第 1 种： 通过在查询的 SQL 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。

```xml
<select id="getOrder" parameterType="int" resultType="com.jourwon.pojo.Order">
       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};
</select>
```

- 第 2 种： 通过 resultMap 中的<result>来映射字段名和实体类属性名的一一对应的关系。

```xml
<select id="getOrder" parameterType="int" resultMap="orderResultMap">
  select * from orders where order_id=#{id}
</select>

<resultMap type="com.jourwon.pojo.Order" id="orderResultMap">
    <!–用id属性来映射主键字段–>
    <id property="id" column="order_id">
    <!–用result属性来映射非主键字段，property为实体类属性名，column为数据库表中的属性–>
  <result property ="orderno" column ="order_no"/>
  <result property="price" column="order_price" />
</resultMap>
```



### Mybatis 是否可以映射 Enum 枚举类？

- Mybatis 当然**可以映射枚举类**，不单可以映射枚举类，Mybatis **可以映射任何对象到表的一列上**。映射方式为自定义一个 TypeHandler，**实现 TypeHandler 的 setParameter()和 getResult()接口方法**。
- TypeHandler 有两个作用，一是完成从 javaType 至 jdbcType 的转换，二是完成 jdbcType 至 javaType 的转换，体现为 setParameter()和 getResult()两个方法，分别代表设置 sql 问号占位符参数和获取列查询结果。



### :star:#{}和${}的区别?

**`#{}` 是预编译处理，`${}` 是字符串替换。**

①、当使用 `#{}` 时，MyBatis 会在 SQL 执行之前，将**占位符替换为问号 `?`**，并使用参数值来替代这些问号。

由于 `#{}` 使用了预处理，所以能**有效防止 SQL 注入**，确保参数值在到达数据库之前被正确地处理和转义。

```xml
<select id="selectUser" resultType="User">
  SELECT * FROM users WHERE id = #{id}
</select>
```

②、当使用 `${}` 时，参数的值会直接替换到 SQL 语句中去，而不会经过预处理。

这就**存在 SQL 注入的风险**，因为参数值会直接拼接到 SQL 语句中，假如参数值是 `1 or 1=1`，那么 SQL 语句就会变成 `SELECT * FROM users WHERE id = 1 or 1=1`，这样就会导致查询出所有用户的结果。

**`${}` 通常用于那些不能使用预处理的场合，比如说动态表名、列名、排序等，要提前对参数进行安全性校验。**

```xml
<select id="selectUsersByOrder" resultType="User">
  SELECT * FROM users ORDER BY ${columnName} ASC
</select>
```





### 模糊查询 like 语句该怎么写?

![image-20241221101440076](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211014139.png)

```xml
<select id="findByName" resultType="User">
    SELECT * FROM users WHERE name LIKE CONCAT('%', #{name}, '%')
</select>
```





### Mybatis 能执行一对一、一对多的关联查询吗？

当然可以，不止支持一对一、一对多的关联查询，还支持多对多、多对一的关联查询。

![image-20241221101819093](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211018153.png)

**association 指的就是一对一，collection 指的就是一对多查询。**





### Mybatis 是否支持延迟加载？原理？

- Mybatis 支持 **association 关联对象和 collection 关联集合对象的延迟加载**，association 指的就是一对一，collection 指的就是一对多查询。在 Mybatis 配置文件中，可以配置是否启用延迟加载 lazyLoadingEnabled=true|false。
- 它的原理是，**使用 CGLIB 创建目标对象的代理对象**，当调用目标方法时，进入**拦截器方法**，比如调用 a.getB().getName()，拦截器 **invoke()方法**发现 a.getB()是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName()方法的调用。这就是延迟加载的基本原理。
- 当然了，不光是 Mybatis，几乎所有的包括 Hibernate，支持延迟加载的原理都是一样的。





### 如何获取生成的主键?

- 新增标签中添加：keyProperty=" ID " 即可，这时候就可以完成回填主键





### MyBatis 支持动态 SQL 吗？

MyBatis 中有一些支持动态 SQL 的标签，它们的原理是**使用 OGNL** 从 SQL 参数对象中计算表达式的值，根据表达式的值**动态拼接 SQL**，以此来完成动态 SQL 的功能。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211023233.png" alt="MyBatis" style="zoom: 67%;" />](https://camo.githubusercontent.com/5bbec4f023ff693ad309a1c466c033c8b38ab51ae9dc34d6c604648b0658b635/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d66353263303237642d323561352d346264392d623564332d3134323136353535343661352e706e67)





### MyBatis 如何执行批量操作？

![image-20241221102420028](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211024089.png)

**第一种方法：使用 foreach 标签**

foreach 的主要用在构建 in 条件中，它可以在 SQL 语句中进行迭代一个集合。foreach 标签的属性主要有 item，index，collection，open，separator，close。

- **item**   表示集合中每一个元素进行迭代时的别名，随便起的变量名；
- **index**   指定一个名字，用于表示在迭代过程中，每次迭代到的位置，不常用；
- **open**   表示该语句以什么开始，常用“(”；
- **separator** 表示在每次进行迭代之间以什么符号作为分隔符，常用“,”；
- **close**   表示以什么结束，常用“)”。

在使用 foreach 的时候最关键的也是最容易出错的就是 **collection 属性**，该属性是**必须指定**的，但是在不同情况下，该属性的值是不一样的，主要有以下 3 种情况：

1. 如果传入的是单参数且参数类型是一个 List 的时候，collection 属性值为 list
2. 如果传入的是单参数且参数类型是一个 array 数组的时候，collection 的属性值为 array
3. 如果传入的参数是多个的时候，我们就需要把它们封装成一个 Map 了，当然单参数也可以封装成 map，实际上如果你在传入参数的时候，在 MyBatis 里面也是会把它封装成一个 Map 的，map 的 key 就是参数名，所以这个时候 collection 属性值就是传入的 List 或 array 对象在自己封装的 map 里面的 key



**第二种方法：使用 ExecutorType.BATCH**

- Mybatis 内置的 ExecutorType 有 3 种，**默认为 simple**，该模式下它为**每个语句的执行创建一个新的预处理语句**，单条提交 sql；而 **batch** 模式**重复使用已经预处理的语句，并且批量执行所有更新语句**，显然 batch 性能将更优； 但 batch 模式也有自己的问题，比如在 Insert 操作时，在事务没有提交之前，是没有办法获取到自增的 id，在某些情况下不符合业务的需求。





### 说说 Mybatis 的一级、二级缓存？

MyBatis 的二级缓存是一个跨 SQLSession 级别的缓存，它可以提高查询效率，减少数据库访问次数。相较于一级缓存（**SQLSession 级别**），二级缓存是**Mapper 级别**的缓存，**多个 SQLSession 共享同一个缓存数据**。



1. **一级缓存**: 基于 **PerpetualCache 的 HashMap 本地缓存**，其**存储作用域为 SqlSession**，各个 SqlSession 之间的缓存**相互隔离**，当 Session flush 或 close 之后，该 SqlSession 中的所有 Cache 就将清空，MyBatis **默认打开一级缓存。**

   [![Mybatis一级缓存](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211027560.png)](https://camo.githubusercontent.com/2545fead6188dd6d758c0a1c296723a4f920b613a02b9d4103a215c47a98a29e/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d35346166623435382d376466632d346434382d396139302d3461643161383733393933372e706e67)

2. **二级缓存**与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同之处在于其**存储作用域为 Mapper(Namespace)**，可以在**多个 SqlSession 之间共享**，并且可自定义存储源，如 Ehcache。**默认不打开二级缓存**，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态),可在它的映射文件中配置。

![Mybatis二级缓存示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211028249.png)



## 原理

### 能说说 MyBatis 的工作原理吗？

![image-20241221102948113](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211029176.png)

#### 构建会话工厂

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211030704.png" alt="image-20241221103008604" style="zoom:50%;" />

- 获取配置

获取配置这一步经过了几步转化，最终由生成了一个配置类 Configuration 实例，这个配置类实例非常重要，主要作用包括：

- 读取配置文件，包括基础配置文件和映射文件
- 初始化基础配置，比如 MyBatis 的别名，还有其它的一些重要的类对象，像插件、映射器、ObjectFactory 等等
- 提供一个**单例**，作为会话工厂构建的重要参数
- 它的构建过程也会初始化一些环境变量，比如数据源

- 构建 SqlSessionFactory



#### 会话运行

![image-20241221103215564](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211032662.png)

- Executor（执行器）

Executor 起到了至关重要的作用，SqlSession 只是一个门面，相当于客服，真正干活的是是 Executor，就像是默默无闻的工程师。它提供了相应的查询和更新方法，以及事务方法。

- StatementHandler（数据库会话器）

StatementHandler，顾名思义，处理数据库会话的。我们以 SimpleExecutor 为例，看一下它的查询方法，先生成了一个 StatementHandler 实例，再拿这个 handler 去执行 query。

这里用的就是 ParameterHandler，setParameters 的作用就是设置预编译 SQL 语句的参数。

里面还会用到 typeHandler 类型处理器，对类型进行处理。

- ParameterHandler （参数处理器）

PreparedStatementHandler 里对 sql 进行了预编译处理

这里用的就是 ParameterHandler，setParameters 的作用就是设置预编译 SQL 语句的参数。

里面还会用到 typeHandler 类型处理器，对类型进行处理。

- ResultSetHandler（结果处理器）

我们前面也看到了，最后的结果要通过 ResultSetHandler 来进行处理，handleResultSets 这个方法就是用来包装结果集的。Mybatis 为我们提供了一个 DefaultResultSetHandler，通常都是用这个实现类去进行结果的处理的。

它会使用 typeHandle 处理类型，然后用 ObjectFactory 提供的规则组装对象，返回给调用者。

整体上总结一下会话运行：

[![会话运行的简单示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211033178.png)](https://camo.githubusercontent.com/387f8aa4cfa45b517db4a0a0532a224ca8052d125bce6e92291fbcb9b3214a94/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d65626430373132612d316636322d343135342d623339312d3263623539363633343731302e706e67)





#### MyBatis 的工作原理

[![MyBatis整体工作原理图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211034025.png)](https://camo.githubusercontent.com/2c8d8007f1540f78cb877fa7ba3a683e824a47caee7a5596b2a73c41a7d79fe9/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d64633134326539342d386537662d346563362d613166362d3164323036363932393261642e706e67)

1. **读取 MyBatis 配置文件**——mybatis-config.xml 、加载映射文件——映射文件即 SQL 映射文件，文件中配置了操作数据库的 SQL 语句。最后生成一个配置对象。
2. **构造会话工厂**：通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。
3. **创建会话对象**：由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。
4. **Executor 执行器**：MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。
5. **StatementHandler**：数据库会话器，串联起参数映射的处理和运行结果映射的处理。
6. **参数处理**：对输入参数的类型进行处理，并预编译。
7. **结果处理**：对返回结果的类型进行处理，根据对象映射规则，返回相应的对象。





### MyBatis 的功能架构是什么样的？

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211035604.png" alt="MyBatis功能架构" style="zoom:67%;" />

我们一般把 Mybatis 的功能架构分为三层：

- **API 接口层**：提供给外部使用的接口 API，开发人员通过这些本地 API 来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。
- **数据处理层**：负责具体的 SQL 查找、SQL 解析、SQL 执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
- **基础支撑层**：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。





### 为什么 Mapper 接口不需要实现类？

四个字回答：**动态代理**，我们来看一下获取 Mapper 的过程：

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211035911.png" alt="Mapper代理" style="zoom:50%;" />](https://camo.githubusercontent.com/5a21866ee535eb600c7647f76abe744cb8c94fa26b97cc3637e8c9386d63866d/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d31356533306131352d663334632d346161342d623133312d3464646338363230333438652e706e67)

- 获取 Mapper

我们都知道定义的 Mapper 接口是没有实现类的，Mapper 映射其实是通过**动态代理**实现的。

七拐八绕地进去看一下，发现获取 Mapper 的过程，需要先获取 MapperProxyFactory——Mapper 代理工厂。

- MapperProxyFactory

MapperProxyFactory 的作用是生成 MapperProxy（Mapper 代理对象）。

- MapperProxy

MapperProxy 里，通常会生成一个 MapperMethod 对象，它是通过 cachedMapperMethod 方法对其进行初始化的，然后执行 excute 方法。

- MapperMethod

MapperMethod 里的 excute 方法，会真正去执行 sql。这里用到了命令模式，其实绕一圈，最终它还是通过 SqlSession 的实例去运行对象的 sql。





### Mybatis 都有哪些 Executor 执行器？

<img src="https://camo.githubusercontent.com/f0b8486ef3ef78a64f9d41ae0143cf709abc0dcfed317bee06cb0ddea53a2c6d/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d35393334303134332d353135352d343731392d383639652d3330346235373338623266322e706e67" alt="Mybatis Executor类型" style="zoom:67%;" />

Mybatis 有三种基本的 Executor 执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

- **SimpleExecutor**：每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。
- **ReuseExecutor**：执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。简言之，就是**重复使用** Statement 对象。
- **BatchExecutor**：执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它**缓存**了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。

作用范围：Executor 的这些特点，都严格限制在 SqlSession 生命周期范围内。

> **Mybatis 中如何指定使用哪一种 Executor 执行器？**

- 在 Mybatis 配置文件中，在设置（settings）可以指定默认的 ExecutorType 执行器类型，也可以手动给 DefaultSqlSessionFactory 的创建 SqlSession 的方法传递 ExecutorType 类型参数，如`SqlSession openSession(ExecutorType execType)`。
- 配置默认的执行器。
  - **SIMPLE 就是普通的执行器；**
  - **REUSE 执行器会重用预处理语句（prepared statements）； **
  - **BATCH 执行器将重用语句并执行批量更新。**





## 插件

### 说说 Mybatis 的插件运行原理，如何编写一个插件？

#### 插件的运行原理？

Mybatis 会话的运行需要 ParameterHandler、ResultSetHandler、StatementHandler、Executor 这四大对象的配合，插件的原理就是在这四大对象调度的时候，插入一些我我们自己的代码。

[![MyBatis插件原理简图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412211039085.png)](https://camo.githubusercontent.com/ba7406723411095f9f35b75793030b916995190569c21e2fcbdffc27dd14c847/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6d7962617469732d30306632353831622d356161652d343431612d383366372d3735363431623362613031302e706e67)

Mybatis 使用 JDK 的动态代理，为目标对象生成代理对象。它提供了一个工具类`Plugin`，实现了`InvocationHandler`接口。

使用`Plugin`生成代理对象，代理对象在调用方法的时候，就会进入 invoke 方法，在 invoke 方法中，如果存在签名的拦截方法，插件的 intercept 方法就会在这里被我们调用，然后就返回结果。如果不存在签名方法，那么将直接反射调用我们要执行的方法。

#### 如何编写一个插件？

我们自己编写 MyBatis 插件，**只需要实现拦截器接口 `Interceptor (org.apache.ibatis. plugin Interceptor ）`**，在实现类中对拦截对象和方法进行处理。

- 实现 Mybatis 的 Interceptor 接口并重写 intercept()方法

- 然后再给插件编写注解，确定要拦截的对象，要拦截的方法

- 最后，再 MyBatis 配置文件里面配置插件



### MyBatis 是如何进行分页的？分页插件的原理是什么？

#### MyBatis 是如何分页的？

MyBatis 使用 RowBounds 对象进行分页，它是**针对 ResultSet 结果集执行的内存分页，而非物理分页**。可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

#### 分页插件的原理是什么？

- 分页插件的基本原理是使用 Mybatis 提供的插件接口，实现自定义插件，**拦截 Executor 的 query 方法**
- 在执行查询的时候，**拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。**
- 举例：`select * from student`，拦截 sql 后重写为：`select t.* from (select * from student) t limit 0, 10`





## 补充

### 说说 JDBC 的执行步骤？

**第一步，加载数据库驱动**

在与数据库建立连接之前，首先需要通过`Class.forName()`方法加载对应的数据库驱动。这一步确保 JDBC 驱动注册到了`DriverManager`类中。

**第二步，建立数据库连接**

使用`DriverManager.getConnection()`方法建立到数据库的连接。这一步需要提供数据库 URL、用户名和密码作为参数。

**第三步，创建`Statement`对象**

通过建立的数据库连接对象`Connection`创建`Statement`、`PreparedStatement`或`CallableStatement`对象，用于执行 SQL 语句。

或者创建`PreparedStatement`对象（预编译 SQL 语句，适用于带参数的 SQL）：

**第四步，执行 SQL 语句**

使用`Statement`或`PreparedStatement`对象执行 SQL 语句。

**第五步，处理结果集**

如果执行的是查询操作，需要处理`ResultSet`对象来获取数据。

**第六步，关闭资源**

最后，需要依次关闭`ResultSet`、`Statement`和`Connection`等资源，释放数据库连接等资源。



### 创建连接拿到的是什么对象？

在 JDBC 的执行步骤中，创建连接后拿到的对象是`java.sql.Connection`对象。这个对象是 JDBC API 中用于表示数据库连接的接口，它提供了执行 SQL 语句、管理事务等一系列操作的方法。

`Connection`对象代表了应用程序和数据库的一个连接会话。

通过调用`DriverManager.getConnection()`方法并传入数据库的 URL、用户名和密码等信息来获得这个对象。

一旦获得`Connection`对象，就可以使用它来创建执行 SQL 语句的`Statement`、`PreparedStatement`和`CallableStatement`对象，以及管理事务等。





### Statement 与 PreparedStatement 的区别

`Statement`和`PreparedStatement`都是**用于执行 SQL 语句的接口**，但它们之间存在几个关键的区别：

①、每次执行`Statement`对象的`executeQuery`或`executeUpdate`方法时，SQL 语句在数据库端都需要**重新编译**和执行。这适用于一次性执行的 SQL 语句。

**Statement** 不支持参数化查询。如果需要在 SQL 语句中插入变量，通常需要通过字符串拼接的方式来实现，这会增加 **SQL 注入攻击的风险**。

②、**PreparedStatement** 代表**预编译**的 SQL 语句的对象。这意味着 SQL 语句在`PreparedStatement`对象创建时就被发送到数据库进行预编译。

之后，可以通过设置参数值来多次高效地执行这个 SQL 语句。这不仅减少了数据库编译 SQL 语句的开销，也提高了性能，尤其是对于重复执行的 SQL 操作。

**PreparedStatement** 支持参数化查询，即可以在 SQL 语句中使用问号（`?`）作为参数占位符。通过`setXxx`方法（如`setString`、`setInt`）设置参数，可以有效防止 SQL 注入。

总的来说，`PreparedStatement`相比`Statement`有着更好的性能和更高的安全性，是执行 SQL 语句的首选方式，尤其是在处理含有用户输入的动态查询时。







### 什么是 SQL 注入？如何防止 SQL 注入？

为了防止 SQL 注入，可以采取以下措施：

①、**使用参数化查询**

使用参数化查询，即使用`PreparedStatement`对象，**通过`setXxx`方法设置参数值，而不是通过字符串拼接 SQL 语句**。这样可以有效防止 SQL 注入。

`?` 是一个参数占位符，userName 是外部输入。这样即便用户输入了恶意的 SQL 语句，也只会被视为参数的一部分，不会改变查询的结构。

②、**限制用户输入**

对用户输入进行验证和过滤，只允许输入预期的数据，不允许输入特殊字符或 SQL 关键字。

③、**使用 ORM 框架**

比如，在 MyBatis 中，使用**`#{}`占位符**来代替直接拼接 SQL 语句，MyBatis 会自动进行参数化处理。

































































































































































































































































