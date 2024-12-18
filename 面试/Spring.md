# Spring

## 基础

### Spring 是什么？特性？有哪些模块？

**Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面 (AOP) 的框架。**

#### Spring 有哪些特性呢？

![image-20241218171919314](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181719396.png)

1. **IoC** 和 **DI** 的支持

Spring 的核心就是一个大的工厂容器，可以维护所有对象的创建和依赖关系，Spring 工厂用于生成 Bean，并且管理 Bean 的生命周期，实现**高内聚低耦合**的设计理念。

1. AOP 编程的支持

Spring 提供了**面向切面编程**，可以方便的实现对程序进行权限拦截、运行监控等切面功能。

1. **声明式事务的支持**

支持通过配置就来完成对事务的管理，而不需要通过硬编码的方式，以前重复的一些事务提交、回滚的 JDBC 代码，都可以不用自己写了。

1. 快捷测试的支持

Spring 对 Junit 提供支持，可以通过**注解**快捷地测试 Spring 程序。

1. 快速集成功能

方便集成各种优秀框架，Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Hibernate、MyBatis、Quartz 等）的直接支持。

1. 复杂 API 模板封装

Spring 对 JavaEE 开发中非常难用的一些 API（JDBC、JavaMail、远程调用等）都提供了模板化的封装，这些封装 API 的提供使得应用难度大大降低。



#### 简单说一下什么是AOP 和 IoC？

**AOP**：面向切面编程，是一种编程范式，它的主要作用是将那些与核心业务逻辑无关，但是对多个对象产生影响的公共行为封装起来，如日志记录、性能统计、事务等。

**IoC**：控制反转，是一种设计思想，它的主要作用是将对象的创建和对象之间的调用过程交给 Spring 容器来管理。



#### Spring源码看过吗？

看过一些，主要就是针对 Spring 循环依赖、Bean 声明周期、AOP、事务、IOC 这五部分。

![image-20241218160524070](C:/Users/raosirui/AppData/Roaming/Typora/typora-user-images/image-20241218160524070.png)



### Spring 有哪些模块呢？

Spring 框架是分模块存在，除了最核心的`Spring Core Container`是必要模块之外，其他模块都是`可选`，大约有 20 多个模块。

[![Spring模块划分](https://camo.githubusercontent.com/bacc91775fb91f34f1dc81cd025d064934f86be5b6d8805c8eae75c3f0c0895e/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d62623763313365612d333137342d346233322d383462382d3832313834396464633337372e706e67)](https://camo.githubusercontent.com/bacc91775fb91f34f1dc81cd025d064934f86be5b6d8805c8eae75c3f0c0895e/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d62623763313365612d333137342d346233322d383462382d3832313834396464633337372e706e67)

最主要的七大模块：

1. **Spring Core**：Spring 核心，它是框架最基础的部分，提供 IoC 和依赖注入 DI 特性。
2. **Spring Context**：Spring 上下文容器，它是 BeanFactory 功能加强的一个子接口。
3. **Spring Web**：它提供 Web 应用开发的支持。
4. **Spring MVC**：它针对 Web 应用中 MVC 思想的实现。
5. **Spring DAO**：提供对 JDBC 抽象层，简化了 JDBC 编码，同时，编码更具有健壮性。
6. **Spring ORM**：它支持用于流行的 ORM 框架的整合，比如：Spring + Hibernate、Spring + iBatis、Spring + JDO 的整合等。
7. **Spring AOP**：即面向切面编程，它提供了与 AOP 联盟兼容的编程实现。





### Spring 有哪些常用注解呢？

![image-20241218172352171](C:/Users/raosirui/AppData/Roaming/Typora/typora-user-images/image-20241218172352171.png)

#### Web 开发方面有哪些注解呢？

①、`@Controller`：用于标注控制层组件。

②、`@RestController`：是`@Controller` 和 `@ResponseBody` 的结合体，返回 JSON 数据时使用。

③、`@RequestMapping`：用于映射请求 URL 到具体的方法上，还可以细分为：

- `@GetMapping`：只能用于处理 GET 请求
- `@PostMapping`：只能用于处理 POST 请求
- `@DeleteMapping`：只能用于处理 DELETE 请求

④、`@ResponseBody`：直接将返回的数据放入 HTTP 响应正文中，一般用于返回 JSON 数据。

⑤、`@RequestBody`：表示一个方法参数应该绑定到 Web 请求体。

⑥、`@PathVariable`：用于接收路径参数，比如 `@RequestMapping(“/hello/{name}”)`，这里的 name 就是路径参数。

⑦、`@RequestParam`：用于接收请求参数。比如 `@RequestParam(name = "key") String key`，这里的 key 就是请求参数。

#### 容器类注解有哪些呢？

- `@Component`：标识一个类为 Spring 组件，使其能够被 Spring 容器自动扫描和管理。
- `@Service`：标识一个业务逻辑组件（服务层）。比如 `@Service("userService")`，这里的 userService 就是 Bean 的名称。
- `@Repository`：标识一个数据访问组件（持久层）。
- `@Autowired`：按类型自动注入依赖。
- `@Configuration`：用于定义配置类，可替换 XML 配置文件。
- `@Value`：用于将 Spring Boot 中 application.properties 配置的属性值赋值给变量。

#### AOP 方面有哪些注解呢？

`@Aspect` 用于声明一个切面，可以配合其他注解一起使用，比如：

- `@After`：在方法执行之后执行。
- `@Before`：在方法执行之前执行。
- `@Around`：方法前后均执行。
- `@PointCut`：定义切点，指定需要拦截的方法。

#### 事务注解有哪些？

主要就是 `@Transactional`，用于声明一个方法需要事务支持。



















































































































































































































































































































































