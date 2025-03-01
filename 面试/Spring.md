# Spring

## 基础

### Spring 是什么？特性？有哪些模块？

**Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面 (AOP) 的框架。**

#### Spring 有哪些特性呢？

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359624.png" alt="image-20241218213712391" style="zoom:50%;" />

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

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359625.png" alt="image-20241218160524070" style="zoom:50%;" />



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

![三分恶面渣逆袭：Spring常用注解](https://camo.githubusercontent.com/bb3c1c8efff1d279066af93d4067373d362b16e68d3c7a3eea831134943ecf76/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d38643061313531382d613432352d343838372d393733352d3435333231303935643932372e706e67)

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



### Spring 中应用了哪些设计模式呢？

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412182343097.png" alt="三分恶面渣逆袭：Spring中用到的设计模式" style="zoom: 67%;" />](https://camo.githubusercontent.com/d54b1a5c039c8779df21241078cc1acf94615068ee171aca9f1ff7e5b8159a7f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d65653163356365652d383436322d346261652d393365612d6563393336636337373634302e706e67)

①、比如说**工厂模式**用于 **BeanFactory 和 ApplicationContext**，实现 **Bean 的创建和管理**。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
MyBean myBean = context.getBean(MyBean.class);
```

②、比如说**单例模式**，这样可以**保证 Bean 的唯一性**，减少系统开销。

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
MyService myService1 = context.getBean(MyService.class);
MyService myService2 = context.getBean(MyService.class);

// This will print "true" because both references point to the same instance
System.out.println(myService1 == myService2);
```

③、比如说 **AOP** 使用了**代理模式**来实现横切关注点（如事务管理、日志记录、权限控制等）。

```java
@Transactional
public void myTransactionalMethod() {
    // 方法实现
}
```



#### Spring如何实现单例模式？

Spring 通过 IOC 容器实现单例模式，具体步骤是：

单例 Bean 在容器初始化时创建并使用 DefaultSingletonBeanRegistry 提供的 **singletonObjects** 进行缓存。

```java
// 单例缓存
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>();

public Object getSingleton(String beanName) {
    return this.singletonObjects.get(beanName);
}

protected void addSingleton(String beanName, Object singletonObject) {
    this.singletonObjects.put(beanName, singletonObject);
}
```

在**请求 Bean 时，Spring 会先从缓存中获取**。





### Spring 容器、Web 容器之间的区别？

Spring 容器是 Spring 框架的核心部分，负责**管理应用程序中的对象生命周期和依赖注入**。

Web 容器（也称 Servlet 容器），是用于**运行 Java Web 应用程序的服务器环境**，支持 Servlet、JSP 等 Web 组件。**常见的 Web 容器包括 Apache Tomcat、Jetty等**。

Spring MVC 是 Spring 框架的一部分，专门用于处理 Web 请求，基于 MVC（Model-View-Controller）设计模式。





## IoC

### :star:说一说什么是 IoC、DI？

所谓的**IoC**，就是**由容器来控制对象的生命周期和对象之间的关系。控制对象生命周期的不再是引用它的对象，而是容器**，这就叫**控制反转**（Inversion of Control）。

[![三分恶面渣逆袭：控制反转示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412182349720.png)](https://camo.githubusercontent.com/2ada8d91d78cdf573ae2af3dc6b61e0d8d7f21de4ccbb754088b80f7551e43a3/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d34343066356430652d663464622d343632632d393766622d6435343430376133353464352e706e67)

**所有类的创建和销毁都通过 Spring 容器来**，不再是开发者去 new，去 `= null`，这样就实现了对象的解耦。

对于某个对象来说，以前是它控制它依赖的对象，现在是**所有对象都被 Spring 控制**。



#### 说说什么是 DI？

IOC 是一种思想，DI 是实现 IOC 的**具体方式**，比如说利用**注入机制（而非查找）**（如构造器注入、Setter 注入）将依赖传递给目标对象。



#### 为什么要使用 IoC 呢？

在平时的 Java 开发中，如果我们要实现某一个功能，可能至少需要两个以上的对象来协助完成，在没有 Spring 之前，每个对象在需要它的合作对象时，需要自己 new 一个，比如说 A 要使用 B，A 就对 B 产生了依赖，也就是 A 和 B 之间存在了一种耦合关系。

有了 Spring 之后，就不一样了，创建 B 的工作交给了 Spring 来完成，Spring 创建好了 B 对象后就放到容器中，A 告诉 Spring 我需要 B，Spring 就从容器中取出 B 交给 A 来使用。

 IoC 的好处，它**降低了对象之间的耦合度，使得程序更加灵活，更加易于维护。**





### 能简单说一下 Spring IoC 的实现机制吗？

Spring 的 IoC 本质就是一个大**工厂**

![image-20241219000425461](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359626.png)

在 Spring 里，不用 Bean 自己来实例化，而是交给 Spring，应该怎么实现呢？——答案毫无疑问，**反射**。

那么这个厂子的生产管理是怎么做的？你应该也知道——**工厂模式**。

![image-20241219000701673](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359627.png)

- 对象工厂，我们最**核心**的一个类，在它初始化的时候，创建了 bean 注册器，完成了资源的加载。
- 获取 bean 的时候，先从单例缓存中取，如果没有取到，就创建并注册一个 bean





### 说说 BeanFactory 和 ApplicantContext?

- BeanFactory 主要负责配置、创建和管理 bean，为 Spring 提供了基本的依赖注入（DI）支持。
- ApplicationContext 是 BeanFactory 的子接口，在 BeanFactory 的基础上添加了企业级的功能支持。

![image-20241219000821651](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359628.png)



#### 详细说说 BeanFactory

BeanFactory 位于整个 Spring IoC 容器的顶端，ApplicationContext 算是 BeanFactory 的子接口。

![image-20241219000855721](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359629.png)

它最主要的方法就是 `getBean()`，这个方法负责从容器中返回特定名称或者类型的 Bean 实例。



#### 请详细说说 ApplicationContext

ApplicationContext 继承了 HierachicalBeanFactory 和 ListableBeanFactory 接口，算是 BeanFactory 的自动挡版本，是 Spring 应用的默认方式。

![image-20241219000954229](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359630.png)

ApplicationContext 会在**启动时预先创建和配置所有的单例 bean**，并支持如 JDBC、ORM 框架的集成，内置面向切面编程（AOP）的支持，可以配置声明式事务管理等。





### 你知道 Spring 容器启动阶段会干什么吗？

Spring 的 IoC 容器工作的过程，其实可以划分为两个阶段：**容器启动阶段**和**Bean 实例化阶段**。

其中容器启动阶段主要做的工作是加载和解析配置文件，保存到对应的 Bean 定义中。

![image-20241219001435551](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359631.png)

容器启动开始，首先会通过某种途径加载 Configuration MetaData，在大部分情况下，容器需要依赖某些工具类（BeanDefinitionReader）对加载的 Configuration MetaData 进行解析和分析，并将分析后的信息组为相应的 BeanDefinition。

![image-20241219001458456](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359632.png)

最后把这些保存了 Bean 定义必要信息的 BeanDefinition，注册到相应的 BeanDefinitionRegistry，这样容器启动就完成了。



#### 说说 Spring 的 Bean 实例化方式

Spring 提供了 4 种不同的方式来实例化 Bean，以满足不同场景下的需求。

#### 构造方法的方式

在类上使用@Component（或@Service、@Repository 等特定于场景的注解）标注类，然后通过构造方法注入依赖。

#### 静态工厂的方式

在这种方式中，Bean 是由一个静态方法创建的，而不是直接通过构造方法。

```java
public class ClientService {
    private static ClientService clientService = new ClientService();

    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

#### 实例工厂方法实例化的方式

与静态工厂方法相比，实例工厂方法依赖于某个类的实例来创建 Bean。这通常用在需要通过工厂对象的非静态方法来创建 Bean 的场景。

```java
public class ServiceLocator {
    public ClientService createClientServiceInstance() {
        return new ClientService();
    }
}
```

#### FactoryBean 接口实例化方式

FactoryBean 是一个特殊的 Bean 类型，可以**在 Spring 容器中返回其他对象的实例**。通过实现 FactoryBean 接口，可以自定义实例化逻辑，这对于构建复杂的初始化逻辑非常有用。

```java
public class ToolFactoryBean implements FactoryBean<Tool> {
    private int factoryId;
    private int toolId;

    @Override
    public Tool getObject() throws Exception {
        return new Tool(toolId);
    }

    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    // setter and getter methods for factoryId and toolId
}
```



### :star:能说一下 Bean 的生命周期吗？

Bean 的生命周期大致分为五个阶段：

![image-20241219003041848](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359633.png)



#### 可以从源码角度讲一下吗？

![image-20241219003222213](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359634.png)

- **实例化**：Spring 容器根据 Bean 的定义创建 Bean 的实例，相当于执行构造方法，也就是 new 一个对象。
- **属性赋值**：相当于执行 setter 方法为字段赋值。
- **初始化**：初始化阶段允许执行自定义的逻辑，比如设置某些必要的属性值、开启资源、执行预加载操作等，以确保 Bean 在使用之前是完全配置好的。
- **销毁**：相当于执行 `= null`，释放资源。

![image-20241219003257189](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359635.png)

至于**销毁**，是在容器关闭的时候调用的，详见 `ConfigurableApplicationContext` 的 `close` 方法。



#### 请在一个已有的 Spring Boot 项目中通过单元测试的形式来展示 Spring Bean 的生命周期？





#### Aware 类型的接口有什么作用？

通过实现 Aware 接口，Bean 可以获取 Spring 容器的相关信息，如 BeanFactory、ApplicationContext 等。

常见 Aware 接口有：

| 接口                    | 作用                                                         |
| ----------------------- | ------------------------------------------------------------ |
| BeanNameAware           | 获取当前 Bean 的名称。                                       |
| BeanFactoryAware        | 获取当前 Bean 所在的 BeanFactory 实例，可以直接操作容器。    |
| ApplicationContextAware | 获取当前 Bean 所在的 ApplicationContext 实例。               |
| EnvironmentAware        | 获取 Environment 对象，用于获取配置文件中的属性或环境变量。  |
| ServletContextAware     | 在 Web 环境下获取 ServletContext 实例，访问 Web 应用上下文。 |
| ResourceLoaderAware     | 获取 ResourceLoader 对象，用于加载资源文件（如类路径文件或 URL）。 |

#### 如果配置了 init-method 和 destroy-method，Spring 会在什么时候调用其配置的方法？

**init-method** 在 Bean **初始化**阶段调用，**依赖注入完成后且 postProcessBeforeInitialization 调用之后**执行。

**destroy-method** 在 Bean **销毁**阶段调用，**容器关闭时**调用。





### Bean 定义和依赖定义有哪些方式？

有三种方式：**直接编码方式**、**配置文件方式**、**注解方式**。

[![Bean依赖配置方式](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191358868.png)](https://camo.githubusercontent.com/61f675167c801b3cb0a7093d0a301fa3211a3e89e3cb621ac04a11a0a87ff2f9/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d38396630663530642d613965342d346465632d623236372d6362316135323663623334302e706e67)

- 直接编码方式：我们一般接触不到直接编码的方式，但其实其它的方式最终都要通过直接编码来实现。
- 配置文件方式：通过 xml、propreties 类型的配置文件，配置相应的依赖关系，Spring 读取配置文件，完成依赖关系的注入。
- 注解方式：注解方式应该是我们用的最多的一种方式了，在相应的地方使用注解修饰，Spring 会扫描注解，完成依赖关系的注入。





### 有哪些依赖注入的方法？

Spring 支持**构造方法注入**、**属性注入**、**工厂方法注入**,其中工厂方法注入，又可以分为**静态工厂方法注入**和**非静态工厂方法注入**。

[![Spring依赖注入方法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191359085.png)](https://camo.githubusercontent.com/493a4ba237b6a5839351438014fc25df66cfca7ed7412283e868e251a69088d1/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d34393166383434342d353462612d343632382d623865622d3834313861323139373039362e706e67)

- **构造方法注入**

  通过调用类的构造方法，将接口实现类通过构造方法变量传入

  ```java
   public CatDaoImpl(String message){
     this. message = message;
   }
  ```

- **属性注入**

  通过 Setter 方法完成调用类所需依赖的注入

  ```java
   public class Id {
      private int id;
  
      public int getId() { return id; }
  
      public void setId(int id) { this.id = id; }
  }
  ```

- **工厂方法注入**

  - **静态工厂注入**

    静态工厂顾名思义，就是通过调用静态工厂的方法来获取自己需要的对象，为了让 Spring 管理所有对象，我们不能直接通过"工程类.静态方法()"来获取对象，而是依然通过 Spring 注入的形式获取：

    ```java
    public class DaoFactory { //静态工厂
    
       public static final FactoryDao getStaticFactoryDaoImpl(){
          return new StaticFacotryDaoImpl();
       }
    }
    
    public class SpringAction {
    
     //注入对象
     private FactoryDao staticFactoryDao;
    
     //注入对象的 set 方法
     public void setStaticFactoryDao(FactoryDao staticFactoryDao) {
         this.staticFactoryDao = staticFactoryDao;
     }
    
    }
    ```

  - **非静态工厂注入**

    非静态工厂，也叫实例工厂，意思是工厂方法不是静态的，所以我们需要首先 new 一个工厂实例，再调用普通的实例方法。

    ```java
    //非静态工厂
    public class DaoFactory {
       public FactoryDao getFactoryDaoImpl(){
         return new FactoryDaoImpl();
       }
     }
    
    public class SpringAction {
      //注入对象
      private FactoryDao factoryDao;
    
      public void setFactoryDao(FactoryDao factoryDao) {
        this.factoryDao = factoryDao;
      }
    }
    ```





### Spring 有哪些自动装配的方式？

#### 什么是自动装配？

Spring IoC 容器知道所有 Bean 的配置信息，此外，通过 Java 反射机制还可以获知实现类的结构信息，如构造方法的结构、属性等信息。掌握所有 Bean 的这些信息后，Spring IoC 容器就可以按照某种规则对容器中的 Bean 进行自动装配，而**无须通过显式的方式进行依赖配置**。

Spring 提供的这种方式，可以按照某些规则进行 Bean 的自动装配，`<bean>`元素提供了一个指定自动装配类型的属性：`autowire="<自动装配类型>"`

#### Spring 提供了哪几种自动装配类型？

Spring 提供了 4 种自动装配类型：

[![Spring四种自动装配类型](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191404572.png)](https://camo.githubusercontent.com/1961b158657f508d23e8e2ff0f73cb55350f5459eea167df04b4998eac775a31/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d30333431323064392d383863372d343930622d616630372d3764343866336236623762632e706e67)

- **byName**：**根据名称**进行自动匹配，假设 Boss 有一个名为 car 的属性，如果容器中刚好有一个名为 car 的 bean，Spring 就会自动将其装配给 Boss 的 car 属性
- **byType**：**根据类型**进行自动匹配，假设 Boss 有一个 Car 类型的属性，如果容器中刚好有一个 Car 类型的 Bean，Spring 就会自动将其装配给 Boss 这个属性
- **constructor**：与 byType 类似， 只不过它是**针对构造函数注入**而言的。如果 Boss 有一个构造函数，构造函数包含一个 Car 类型的入参，如果容器中有一个 Car 类型的 Bean，则 Spring 将自动把这个 Bean 作为 Boss 构造函数的入参；如果容器中没有找到和构造函数入参匹配类型的 Bean，则 Spring 将抛出异常。
- **autodetect**：根据 Bean 的**自省机制决定采用 byType 还是 constructor** 进行自动装配，**如果 Bean 提供了默认的构造函数，则采用 byType，否则采用 constructor。**





### Bean 的作用域有哪些?

在 Spring 中，Bean 默认是单例的，即在整个 Spring 容器中，每个 Bean 只有一个实例。

可以通过在配置中指定 scope 属性，将 Bean 改为**多例（Prototype）**模式，这样**每次获取的都是新的实例**。

```java
@Bean
@Scope("prototype")  // 每次获取都是新的实例
public MyBean myBean() {
    return new MyBean();
}
```

除了**单例**和**多例**，Spring 还支持其他作用域，如**请求作用域**（Request）、**会话作用域**（Session）等，适合 Web 应用中特定的使用场景。

[![三分恶面渣逆袭：Spring Bean支持作用域](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191406023.png)](https://camo.githubusercontent.com/1d51682dede5c6f08ed98aa2a7dd69ea7b0b8fb7d3833d702816274079712cad/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d30386139636233312d356134662d343232342d393463642d3063306636343361353765612e706e67)

- **request**：每一次 HTTP 请求都会产生一个新的 Bean，该 Bean 仅在当前 HTTP Request 内有效。
- **session**：同一个 Session 共享一个 Bean，不同的 Session 使用不同的 Bean。
- **globalSession**：同一个全局 Session 共享一个 Bean，只用于基于 Protlet 的 Web 应用，Spring5 中已经移除。





### Spring 中的单例 Bean 会存在线程安全问题吗？

Spring Bean 的默认作用域是单例（Singleton），这意味着 **Spring 容器中只会存在一个 Bean 实例，并且该实例会被多个线程共享。**

如果单例 Bean 是**无状态**的，也就是**没有成员变量**，那么这个单例 Bean 是**线程安全**的。比如 Spring MVC 中的 Controller、Service、Dao 等，基本上都是无状态的。

但如果 Bean 的内部状态是可变的，且没有进行适当的同步处理，就可能出现线程安全问题。

[![三分恶面渣逆袭：Spring单例Bean线程安全问题](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191407604.png)](https://camo.githubusercontent.com/fffa35d129ccf1308ced86ddf138c66a77dfb671cec8603814eb03e289f1b4f7/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d33356461636566342d316139652d343565312d623366322d3561393132323765623234342e706e67)



#### 单例 Bean 线程安全问题怎么解决呢？

1. 第一，使用**局部变量**。局部变量是线程安全的，因为每个线程都有自己的局部变量副本。尽量使用局部变量而不是共享的成员变量。

2. 第二，尽量使用**无状态的 Bean**，即不在 Bean 中保存任何可变的状态信息。

3. 第三，**同步访问**。如果 Bean 中确实需要保存可变状态，可以通过 [synchronized 关键字](https://javabetter.cn/thread/synchronized-1.html)或者 [Lock 接口](https://javabetter.cn/thread/reentrantLock.html)来保证线程安全。

   或者将 Bean 中的成员变量保存到 ThreadLocal 中，[ThreadLocal](https://javabetter.cn/thread/ThreadLocal.html) 可以保证多线程环境下变量的隔离。

   再或者使用线程安全的工具类，比如说 [AtomicInteger](https://javabetter.cn/thread/atomic.html)、[ConcurrentHashMap](https://javabetter.cn/thread/ConcurrentHashMap.html)、[CopyOnWriteArrayList](https://javabetter.cn/thread/CopyOnWriteArrayList.html) 等。

4. 第四，将 Bean 定义为**原型作用域**（Prototype）。原型作用域的 Bean 每次请求都会创建一个新的实例，因此不存在线程安全问题。





### 说说循环依赖?

A 依赖 B，B 依赖 A，或者 C 依赖 C，就成了循环依赖。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191411600.png" alt="三分恶面渣逆袭：Spring循环依赖" style="zoom: 67%;" />](https://camo.githubusercontent.com/a06aa71a93fd69b90c0ee548beef190e924ed0ebc3efe5f7426e78245e453032/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d66386665613533662d353666612d346363612d393139392d6563376636343864613632352e706e67)

循环依赖**只发生在 Singleton 作用域**的 Bean 之间，因为如果是 **Prototype 作用域**的 Bean，Spring 会**直接抛出异常。**

原因很简单，AB 循环依赖，A 实例化的时候，发现依赖 B，创建 B 实例，创建 B 的时候发现需要 A，创建 A1 实例……无限套娃。。。。



#### Spring 可以解决哪些情况的循环依赖？

看看这几种情形（AB 循环依赖）：

[![三分恶面渣逆袭：循环依赖的几种情形](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191412856.png)](https://camo.githubusercontent.com/44eb0147ccd26c8101e679f034c1985df320fe552a94ccf77fc979c69f4119c6/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d33376262353736642d623461662d343265642d393166342d6438343663656230313262362e706e67)

也就是说：

- AB 均采用构造器注入，不支持
- AB 均采用 setter 注入，支持
- AB 均采用属性自动注入，支持
- A 中注入的 B 为 setter 注入，B 中注入的 A 为构造器注入，支持
- B 中注入的 A 为 setter 注入，A 中注入的 B 为构造器注入，不支持

第四种可以，第五种不可以的原因是 Spring 在创建 Bean 时**默认会根据自然排序进行创建**，所以 A 会先于 B 进行创建。

简单总结下，**当循环依赖的实例都采用 setter 方法注入时，Spring 支持，都采用构造器注入的时候，不支持；构造器注入和 setter 注入同时存在的时候，看天**（😂）。





### Spring 怎么解决循环依赖呢？

Spring 通过**三级缓存机制**来解决循环依赖：

1. 一级缓存：存放**完全初始化好的单例 Bean。**
2. 二级缓存：存放**正在创建但未完全初始化的 Bean 实例。**
3. 三级缓存：存放 **Bean 工厂对象，用于提前暴露 Bean。**

[![三分恶面渣逆袭：三级缓存](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191413863.png)](https://camo.githubusercontent.com/08e56d2eb51166bab407df967f89c19b0c5e0fec0a2774af4c714830965a963a/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d30316439323836332d613263622d346636312d386438642d3330656366303237396232382e706e67)

#### 三级缓存解决循环依赖的过程是什么样的？

1. 实例化 Bean 时，将其早期引用放入三级缓存。
2. 其他依赖该 Bean 的对象，可以从缓存中获取其引用。
3. 初始化完成后，将 Bean 移入一级缓存。

假如 A、B 两个类发生循环依赖：

![image-20241219141408886](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191414957.png)

A 实例的初始化过程：

①、创建 A 实例，实例化的时候把 A 的对象工厂放入三级缓存，表示 A 开始实例化了，虽然这个对象还不完整，但是先曝光出来让大家知道。

![image-20241219141421394](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191414479.png)

②、A 注⼊属性时，发现依赖 B，此时 B 还没有被创建出来，所以去实例化 B。

③、同样，B 注⼊属性时发现依赖 A，它就从缓存里找 A 对象。依次从⼀级到三级缓存查询 A。

**发现可以从三级缓存中通过对象⼯⼚拿到 A，虽然 A 不太完善，但是存在，就把 A 放⼊⼆级缓存，同时删除三级缓存中的 A，此时，B 已经实例化并且初始化完成了，把 B 放入⼀级缓存。**

![image-20241219141429532](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191414610.png)

④、接着 A 继续属性赋值，顺利从⼀级缓存拿到实例化且初始化完成的 B 对象，A 对象创建也完成，删除⼆级缓存中的 A，同时把 A 放⼊⼀级缓存

⑤、最后，⼀级缓存中保存着实例化、初始化都完成的 A、B 对象。

![image-20241219141444725](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191414806.png)



### 为什么要三级缓存？二级不⾏吗？

- 二级缓存只存储未完成初始化的对象，但在某些框架（例如 Spring）的场景中，**对象需要经过代理（例如 AOP、事务管理）才能完成初始化。**
- 二级缓存无法很好地区分“原始对象”和“代理对象”，可能会导致依赖注入的对象不是最终需要的代理版本。
- 三级缓存通过存储生成代理对象的工厂，可以延迟生成最终的代理对象，在任何阶段都能保证依赖注入的是正确版本。

不行，主要是为了 **⽣成代理对象**。如果是没有代理的情况下，使用二级缓存解决循环依赖也是 OK 的。但是如果存在代理，三级没有问题，二级就不行了。

因为三级缓存中放的是⽣成具体对象的匿名内部类，获取 Object 的时候，它可以⽣成代理对象，也可以返回普通对象。**使⽤三级缓存主要是为了保证不管什么时候使⽤的都是⼀个对象**。

假设只有⼆级缓存的情况，往⼆级缓存中放的显示⼀个普通的 Bean 对象，Bean 初始化过程中，通过 BeanPostProcessor 去⽣成代理对象之后，**覆盖**掉⼆级缓存中的普通 Bean 对象，那么可能就导致取到的 Bean 对象不一致了。

[![三分恶面渣逆袭：二级缓存不行的原因](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191418019.png)](https://camo.githubusercontent.com/ba9dd855d66d8048b27a63a38acd2128f4030fb7c062dc5893531ef725720860/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d36656365386134362d323562312d343539622d386366612d3139666336393664643764362e706e67)

#### 如果缺少第二级缓存会有什么问题？

如果没有二级缓存，Spring 无法在未完成初始化的情况下暴露 Bean。会导致代理 Bean 的循环依赖问题，因为某些代理逻辑无法在三级缓存中提前暴露。最终可能抛出 BeanCurrentlyInCreationException。





### @Autowired 的实现原理？

实现@Autowired 的关键是：**AutowiredAnnotationBeanPostProcessor**

在 Bean 的初始化阶段，会通过 Bean 后置处理器来进行一些前置和后置的处理。

实现@Autowired 的功能，也是通过后置处理器来完成的。这个后置处理器就是 AutowiredAnnotationBeanPostProcessor。

- Spring 在创建 bean 的过程中，最终会调用到 doCreateBean()方法，在 doCreateBean()方法中会调用 populateBean()方法，来为 bean 进行属性填充，完成自动装配等工作。
- 在 populateBean()方法中一共调用了两次后置处理器，第一次是为了判断是否需要属性填充，如果不需要进行属性填充，那么就会直接进行 return，如果需要进行属性填充，那么方法就会继续向下执行，后面会进行第二次后置处理器的调用，这个时候，就会调用到 AutowiredAnnotationBeanPostProcessor 的 postProcessPropertyValues()方法，在该方法中就会进行@Autowired 注解的解析，然后实现自动装配。

- postProcessorPropertyValues()方法的源码如下，在该方法中，会先调用 findAutowiringMetadata()方法解析出 bean 中带有@Autowired 注解、@Inject 和@Value 注解的属性和方法。然后调用 metadata.inject()方法，进行属性填充。





## AOP

### :star:说说什么是 AOP？

AOP，也就是面向切面编程，简单点说，AOP 就是把一些业务逻辑中的相同代码抽取到一个独立的模块中，让业务逻辑更加清爽。

#### AOP 有哪些核心概念？

- **切面**（Aspect）：类是对物体特征的抽象，切面就是对横切关注点的抽象
- **连接点**（Join Point）：被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring 中，连接点指的是被拦截到的方法，实际上连接点还可以是字段或者构造方法
- **切点**（Pointcut）：对连接点进行拦截的定位
- **通知**（Advice）：指拦截到连接点之后要执行的代码，也可以称作**增强**
- **目标对象** （Target）：代理的目标对象
- **引介**（introduction）：一种特殊的增强，可以动态地为类添加一些属性和方法
- **织入**（Weabing）：织入是将增强添加到目标类的具体连接点上的过程。



#### 织入有哪几种方式？

①、**编译期织入**：切面在目标类编译时被织入。

②、**类加载期织入**：切面在目标类加载到 JVM 时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。

③、**运行期织入**：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP 容器会为目标对象动态地**创建一个代理对象**。

**Spring AOP 采用运行期织入**，而 AspectJ 可以在编译期织入和类加载时织入。

#### AspectJ 是什么？

AspectJ 是一个 AOP 框架，它可以做很多 Spring AOP 干不了的事情，比如说**支持编译时、编译后和类加载时织入切面**。并且提供更复杂的切点表达式和通知类型。



#### AOP 有哪些环绕方式？

AOP 一般有 **5 种**环绕方式：

- **前置通知** (@Before)
- **返回通知** (@AfterReturning)
- **异常通知** (@AfterThrowing)
- **后置通知** (@After)
- **环绕通知** (@Around)

[![三分恶面渣逆袭：环绕方式](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191433767.png)](https://camo.githubusercontent.com/e1671effe042c637169d33340212dd72b5f985940bdedb3bfd9a6a6408fdc387/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d33323066613334662d363632302d343139632d623137612d3466353136613833636165622e706e67)

多个切面的情况下，可以通过 `@Order` 指定先后顺序，数字越小，优先级越高。代码示例如下：



#### Spring AOP 发生在什么时候？

Spring AOP 基于**运行时代理机制**，这意味着 Spring AOP 是在运行时通过动态代理生成的，而不是在编译时或类加载时生成的。

在 Spring 容器初始化 Bean 的过程中，Spring AOP 会检查 Bean 是否需要应用切面。如果需要，Spring 会为该 Bean 创建一个代理对象，并在代理对象中织入切面逻辑。这一过程发生在 Spring 容器的后处理器（BeanPostProcessor）阶段。



#### 简单总结一下 AOP

① 像日志打印、事务管理等都可以抽离为切面，可以声明在类的方法上。像 `@Transactional` 注解，就是一个典型的 AOP 应用，它就是通过 AOP 来实现事务管理的。我们只需要在方法上添加 `@Transactional` 注解，Spring 就会在方法执行前后添加事务管理的逻辑。

② Spring AOP 是**基于代理**的，它默认**使用 JDK 动态代理和 CGLIB 代理**来实现 AOP。

③ Spring AOP 的织入方式是运行时织入，而 AspectJ 支持编译时织入、类加载时织入。



### AOP和 OOP 的关系？

AOP 和 OOP 是互补的编程思想：

1. OOP 通过类和对象封装数据和行为，专注于**核心业务逻辑**。
2. AOP 提供了解决**横切关注点**（如日志、权限、事务等）的机制，将这些逻辑集中管理。





### AOP的使用场景有哪些？

AOP 的使用场景有很多，比如说日志记录、事务管理、权限控制、性能监控等。

我在项目中主要利用 AOP 来打印接口的入参和出参日志、执行时间，方便后期 bug 溯源和性能调优。

第一步，自定义注解作为切点

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MdcDot {
    String bizCode() default "";
}
```

第二步，配置 AOP 切面：

- `@Aspect`：标识切面
- `@Pointcut`：设置切点，这里以自定义注解为切点
- `@Around`：环绕切点，打印方法签名和执行时间

第三步，在使用的地方加上自定义注解

第四步，当接口被调用时，就可以看到对应的执行日志。





### 说说 JDK 动态代理和 CGLIB 代理？

AOP 是通过[动态代理](https://mp.weixin.qq.com/s/aZtfwik0weJN5JzYc-JxYg)实现的，代理方式有两种：**JDK 动态代理**和 **CGLIB 代理**。



①、JDK 动态代理是**基于接口**的代理，**只能代理实现了接口的类**。

使用 JDK 动态代理时，Spring AOP 会创建一个代理对象，该代理对象实现了目标对象所实现的接口，并在方法调用前后插入横切逻辑。

优点：只需依赖 JDK 自带的 `java.lang.reflect.Proxy` 类，不需要额外的库；缺点：只能代理接口，不能代理类本身。



②、CGLIB 动态代理是基于继承的代理，可以代理没有实现接口的类。

使用 CGLIB 动态代理时，Spring AOP 会生成目标类的子类，并在方法调用前后插入横切逻辑。

[![图片来源于网络](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191514454.png)](https://camo.githubusercontent.com/ec0764835d6af253d87dadd271b02c82f38374d5813abe94e899b305a9d226c3/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f737072696e672d32303234303332313130353635332e706e67)

优点：可以代理没有实现接口的类，灵活性更高；缺点：需要依赖 CGLIB 库，创建代理对象的开销相对较大。



#### 选择 CGLIB 还是 JDK 动态代理？

- 如果目标对象**没有实现任何接口，则只能使用 CGLIB 代理**。如果目标对象实现了接口，通常**首选 JDK 动态代理。**
- 虽然 CGLIB 在代理类的生成过程中可能消耗更多资源，但在运行时具有较高的性能。对于性能敏感且代理对象创建频率不高的场景，可以考虑使用 CGLIB。
- JDK 动态代理是 Java 原生支持的，不需要额外引入库。而 CGLIB 需要将 CGLIB 库作为依赖加入项目中。



#### 你会用 JDK 动态代理和 CGLIB 吗？

①、JDK 动态代理实现：

[![三分恶面渣逆袭：JDK动态代理类图](https://camo.githubusercontent.com/596c8c9b33b7ea46589b30d53b107b74b28f280e580c1ca365cc14bd1fda70a1/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d36356231346133662d323635332d343633652d616637372d6138383735643364363335632e706e67)](https://camo.githubusercontent.com/596c8c9b33b7ea46589b30d53b107b74b28f280e580c1ca365cc14bd1fda70a1/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d36356231346133662d323635332d343633652d616637372d6138383735643364363335632e706e67)

第一步，创建接口

第二步，实现对应接口

第三步，动态代理工厂:ProxyFactory，直接用**反射方式生成一个目标对象的代理**，这里用了一个匿名内部类方式重写 InvocationHandler 方法。

第五步，客户端：Client，生成一个代理对象实例，通过代理对象调用目标对象方法





②、CGLIB 动态代理实现：

[![三分恶面渣逆袭：CGLIB动态代理类图](https://camo.githubusercontent.com/34f0ee3918d07862906f3d398c78f8258eaab5aa9ce8f73188f3c4f41967950b/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d37346461383761662d323064312d346135622d613231322d3338333761313566306261622e706e67)](https://camo.githubusercontent.com/34f0ee3918d07862906f3d398c78f8258eaab5aa9ce8f73188f3c4f41967950b/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d37346461383761662d323064312d346135622d613231322d3338333761313566306261622e706e67)

第一步：定义目标类（Solver），目标类 Solver 定义了一个 solve 方法，模拟了解决问题的行为。目标类不需要实现任何接口，这与 JDK 动态代理的要求不同。

第二步：动态代理工厂（ProxyFactory），ProxyFactory 类实现了 MethodInterceptor 接口，这是 CGLIB 提供的一个方法拦截接口，用于定义方法的拦截逻辑。

- ProxyFactory 接收一个 Object 类型的 target，即目标对象的实例。
- 使用 CGLIB 的 Enhancer 类来生成目标类的子类（代理对象）。通过 setSuperclass 设置代理对象的父类为目标对象的类，setCallback 设置方法拦截器为当前对象（this），最后调用 create 方法生成并返回代理对象。
- 重写 MethodInterceptor 接口的 intercept 方法以提供方法拦截逻辑。在目标方法执行前后添加自定义逻辑，然后通过 method.invoke 调用目标对象的方法。

第三步：客户端使用代理，首先创建目标对象（Solver 的实例），然后使用 ProxyFactory 创建该目标对象的代理。通过代理对象调用 solve 方法时，会先执行 intercept 方法中定义的逻辑，然后执行目标方法，最后再执行 intercept 方法中的后续逻辑。



### 说说 Spring AOP 和 AspectJ AOP 区别?

![Spring AOP和AspectJ对比](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191529677.png)

在性能上，由于 Spring AOP 是基于**动态代理**来实现的，在容器启动时需要生成代理实例，在方法调用上也会增加栈的深度，使得 Spring AOP 的性能不如 AspectJ 的那么好。

AspectJ 属于**静态织入**，通过修改代码来实现，在实际运行之前就完成了织入，所以说它生成的类是没有额外运行时开销的





### 说说 AOP 和反射的区别？

1. 反射：用于**检查和操作类的方法和字段，动态调用方法或访问字段**。反射是 Java 提供的内置机制，直接操作类对象。
2. 动态代理：通过**生成代理类来拦截方法调用**，通常用于 AOP 实现。动态代理使用反射来调用被代理的方法。





## 事务

Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，Spring 是无法提供事务功能的。Spring 只提供统一事务管理接口，具体实现都是由各数据库自己实现，数据库事务的提交和回滚是通过数据库自己的事务机制实现。

### Spring 事务的种类？

在 Spring 中，事务管理可以分为两大类：声明式事务管理和编程式事务管理。

[![三分恶面渣逆袭：Spring事务分类](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191538195.png)](https://camo.githubusercontent.com/06310858f8b9ebc9c4cb52592fe3749ea916d89eb4787ec8e24af28655b7add9/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d64336565373766612d393236642d346333392d393166382d6138623135343461393133342e706e67)

#### 介绍一下编程式事务管理？

编程式事务可以使用 **TransactionTemplate** 和 **PlatformTransactionManager** 来实现，需要显式执行事务。允许我们在代码中直接控制事务的边界，通过编程方式明确指定事务的开始、提交和回滚。**编程式事务是代码块级别的**

#### 介绍一下声明式事务管理？

声明式事务是建立在 **AOP** 之上的。其本质是通过 AOP 功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前启动一个事务，在目标方法执行完之后根据执行情况提交或者回滚事务。

相比较编程式事务，优点是不需要在业务逻辑代码中掺杂事务管理的代码，Spring 推荐通过 **@Transactional 注解的方式来实现声明式事务管理**，也是日常开发中最常用的。

不足的地方是，声明式事务管理**最细粒度只能作用到方法级别**，无法像编程式事务那样可以作用到代码块级别。



#### 说说两者的区别？

- **编程式事务管理**：需要在代码中显式调用事务管理的 API 来控制事务的边界，比较灵活，但是代码**侵入性较强**，不够优雅。
- **声明式事务管理**：这种方式使用 Spring 的 AOP 来声明事务，将事务管理代码从业务代码中分离出来。优点是代码简洁，易于维护。但缺点是**不够灵活**，只能在预定义的方法上使用事务。



### 说说 Spring 的事务隔离级别？

①、ISOLATION_DEFAULT：使用**数据库默认的隔离级别**（你们爱咋咋滴 😁），**MySQL 默认的是可重复读**，Oracle 默认的读已提交。

②、ISOLATION_READ_UNCOMMITTED：**读未提交，允许事务读取未被其他事务提交的更改**。这是隔离级别最低的设置，**可能会导致“脏读”**问题。

③、ISOLATION_READ_COMMITTED：**读已提交**，**确保事务只能读取已经被其他事务提交的更改**。这可以**防止“脏读”，但仍然可能发生“不可重复读”和“幻读”问题**。

④、ISOLATION_REPEATABLE_READ：**可重复读**，**确保事务可以多次从一个字段中读取相同的值**，即在这个事务内，其他事务无法更改这个字段，从而**避免了“不可重复读”，但仍可能发生“幻读”问题**。

⑤、ISOLATION_SERIALIZABLE：**串行化**，这是最高的隔离级别，它完全隔离了事务，确保事务序列化执行，以此来**避免“脏读”、“不可重复读”和“幻读”问题，但性能影响也最大**。





### Spring 的事务传播机制

事务的传播机制定义了方法在被另一个事务方法调用时的事务行为，这些行为定义了事务的边界和事务上下文如何在方法调用链中传播。

[![三分恶面渣逆袭：6种事务传播机制](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191557806.png)](https://camo.githubusercontent.com/44a866e5b3d1a317740f22266e6ad99e69cd89eab487f98c0efd8944fde127b9/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d61366532613864632d393737312d346438622d396439312d3736646465653938616631612e706e67)

Spring 的**默认传播行为是 PROPAGATION_REQUIRED**，即如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

**事务传播机制是使用 [ThreadLocal](https://javabetter.cn/thread/ThreadLocal.html) 实现的，所以，如果调用的方法是在新线程中，事务传播会失效。**

Spring 默认的事务传播行为是 PROPAFATION_REQUIRED，即如果多个 `ServiceX#methodX()` 都工作在事务环境下，且程序中存在这样的调用链 `Service1#method1()->Service2#method2()->Service3#method3()`，那么这 3 个服务类的 3 个方法都通过 Spring 的事务传播机制工作在同一个事务中。



#### protected 和 private 加事务会生效吗

不会，在 Spring 中，**只有通过 Spring 容器的 AOP 代理调用的公开方法（public method）上的`@Transactional`注解才会生效**。

如果在 protected、private 方法上使用`@Transactional`，这些事务注解将不会生效，原因：Spring 默认使用基于 JDK 的动态代理（当接口存在时）或基于 CGLIB 的代理（当只有类时）来实现事务。这两种**代理机制都只能代理公开的方法。**





### 声明式事务实现原理了解吗？

Spring 的声明式事务管理是通过 AOP（面向切面编程）和代理机制实现的。

第一步，**在 Bean 初始化阶段创建代理对象**：

Spring 容器在初始化单例 Bean 的时候，会遍历所有的 BeanPostProcessor 实现类，并执行其 postProcessAfterInitialization 方法。

在执行 postProcessAfterInitialization 方法时会遍历容器中所有的切面，查找与当前 Bean 匹配的切面，这里会获取事务的属性切面，也就是 `@Transactional` 注解及其属性值。

然后根据得到的切面创建一个代理对象，默认使用 JDK 动态代理创建代理，如果目标类是接口，则使用 JDK 动态代理，否则使用 CGLIB。

第二步，**在执行目标方法时进行事务增强操作**：

当通过代理对象调用 Bean 方法的时候，会触发对应的 AOP 增强拦截器，声明式事务是一种环绕增强，对应接口为`MethodInterceptor`，事务增强对该接口的实现为`TransactionInterceptor`，类图如下：

[![图片来源网易技术专栏](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191600323.png)](https://camo.githubusercontent.com/f7bffdb36f97a20b1f184b04cfc7d942a46cec9259ac870e3fad463dd68eee98/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d39373439336337662d633539362d346539382d613661382d6461623235346436643161622e706e67)

事务拦截器`TransactionInterceptor`在`invoke`方法中，通过调用父类`TransactionAspectSupport`的`invokeWithinTransaction`方法进行事务处理，包括开启事务、事务提交、异常回滚等。



### 声明式事务在哪些情况下会失效？

[![三分恶面渣逆袭：声明式事务的几种失效的情况](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191604248.png)](https://camo.githubusercontent.com/5d45202f7205794da56d3cecc2c37a5a567571dcf83e40d556388a6a32f94d75/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d33383165346563392d613233352d346366612d396234642d3531383039356137353032612e706e67)

#### 2、@Transactional 注解属性 propagation 设置错误

- TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务方式执行；错误使用场景：在业务逻辑必须运行在事务环境下以确保数据一致性的情况下使用 SUPPORTS。
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED：总是以非事务方式执行，如果当前存在事务，则挂起该事务。错误使用场景：在需要事务支持的操作中使用 NOT_SUPPORTED。
- TransactionDefinition.PROPAGATION_NEVER：总是以非事务方式执行，如果当前存在事务，则抛出异常。错误使用场景：在应该在事务环境下执行的操作中使用 NEVER。

#### 3、@Transactional 注解属性 rollbackFor 设置错误

rollbackFor 用来指定能够触发事务回滚的异常类型。Spring 默认抛出未检查 unchecked 异常（继承自 RuntimeException 的异常）或者 Error 才回滚事务，其他异常不会触发回滚事务。

[![三分恶面渣逆袭：Spring默认支持的异常回滚](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191605140.png)](https://camo.githubusercontent.com/3a42ca55cc7154931f63cbf924f5c16733989748689ab35a9c47c1deec039a16/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d30343035336230322d333236342d346437662d623836382d3536306130333333663038642e706e67)

#### 4、同一个类中方法调用，导致@Transactional 失效

开发中避免不了会对同一个类里面的方法调用，**比如有一个类 Test，它的一个方法 A，A 调用本类的方法 B（不论方法 B 是用 public 还是 private 修饰），但方法 A 没有声明注解事务，而 B 方法有。**

则**外部调用方法 A 之后，方法 B 的事务是不会起作用的。**这也是经常犯错误的一个地方。

那为啥会出现这种情况呢？其实还是**由 Spring AOP 代理造成的，因为只有事务方法被当前类以外的代码调用时，才会由 Spring 生成的代理对象来管理。**

如果 B 方法内部抛了异常，而 A 方法此时 try catch 了 B 方法的异常，那这个事务就不能正常回滚了，会抛出异常：





## MVC

### Spring MVC 的核心组件？

1. **DispatcherServlet**：前置控制器，是整个流程控制的**核心**，控制其他组件的执行，进行统一调度，降低组件之间的耦合性，相当于总指挥。
2. **Handler**：处理器，完成具体的业务逻辑，相当于 Servlet 或 Action。
3. **HandlerMapping**：DispatcherServlet 接收到请求之后，通过 HandlerMapping 将不同的请求映射到不同的 Handler。
4. **HandlerInterceptor**：处理器拦截器，是一个接口，如果需要完成一些拦截处理，可以实现该接口。
5. **HandlerExecutionChain**：处理器执行链，包括两部分内容：Handler 和 HandlerInterceptor（系统会有一个默认的 HandlerInterceptor，如果需要额外设置拦截，可以添加拦截器）。
6. **HandlerAdapter**：处理器适配器，Handler 执行业务方法之前，需要进行一系列的操作，包括表单数据的验证、数据类型的转换、将表单数据封装到 JavaBean 等，这些操作都是由 HandlerApater 来完成，开发者只需将注意力集中业务逻辑的处理上，DispatcherServlet 通过 HandlerAdapter 执行不同的 Handler。
7. **ModelAndView**：装载了模型数据和视图信息，作为 Handler 的处理结果，返回给 DispatcherServlet。
8. **ViewResolver**：视图解析器，DispatcheServlet 通过它将逻辑视图解析为物理视图，最终将渲染结果响应给客户端。



### :star:Spring MVC 的工作流程？

首先，客户端发送请求，DispatcherServlet 拦截并通过 HandlerMapping 找到对应的控制器。

DispatcherServlet 使用 HandlerAdapter 调用控制器方法，执行具体的业务逻辑，返回一个 ModelAndView 对象。

然后 DispatcherServlet 通过 ViewResolver 解析视图。

最后，DispatcherServlet 渲染视图并将响应返回给客户端。

[![三分恶面渣逆袭：Spring MVC的工作流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191611841.png)](https://camo.githubusercontent.com/faed098232cd03d00278c70976aa3f1dc97f013f69ef5fd7aa42c0955d99dc72/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d65323961313232622d646230372d343862382d383238392d3732353130333265383761312e706e67)

[![图片来源于网络：SpringMVC工作流程图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191611865.png)](https://camo.githubusercontent.com/2652ee98d03297c626be8c707ca645063d2bee30a4654b757fe236be8005441b/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f737072696e672d32303234303530363130323435362e706e67)

[![_未来可期：SpringMVC工作流程图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191611322.png)](https://camo.githubusercontent.com/58c2ecdfb3cbf555bbb46f1bd0bac2222019e7c4bb9a15564c184a9a83a45ae8/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f737072696e672d32303234303530363130333032322e706e67)

①、**发起请求**：客户端通过 HTTP 协议向服务器发起请求。

②、**前端控制器**：这个请求会先到前端控制器 DispatcherServlet，它是整个流程的入口点，负责接收请求并将其分发给相应的处理器。

③、**处理器映射**：DispatcherServlet 调用 HandlerMapping 来确定哪个 Controller 应该处理这个请求。通常会根据请求的 URL 来确定。

④、**处理器适配器**：一旦找到目标 Controller，DispatcherServlet 会使用 HandlerAdapter 来调用 Controller 的处理方法。

⑤、**执行处理器**：Controller 处理请求，处理完后返回一个 ModelAndView 对象，其中包含模型数据和逻辑视图名。

⑥、**视图解析器**：DispatcherServlet 接收到 ModelAndView 后，会使用 ViewResolver 来解析视图名称，找到具体的视图页面。

⑦、**渲染视图**：视图使用模型数据渲染页面，生成最终的页面内容。

⑧、**响应结果**：DispatcherServlet 将视图结果返回给客户端。

**Spring MVC** 虽然整体流程复杂，但是实际开发中很简单，大部分的组件不需要我们开发人员创建和管理，真正需要处理的只有 **Controller** 、**View** 、**Model**。

在前后端分离的情况下，步骤 ⑥、⑦、⑧ 会略有不同，后端通常只需要处理数据，并将 JSON 格式的数据返回给前端就可以了，而不是返回完整的视图页面。



#### 这个 Handler 是什么东西啊？为什么还需要 HandlerAdapter

**Handler 一般就是指 Controller**，Controller 是 Spring MVC 的核心组件，负责处理请求，返回响应。

Spring MVC 允许使用多种类型的处理器。不仅仅是标准的`@Controller`注解的类，还可以是实现了特定接口的其他类（如 HttpRequestHandler 或 SimpleControllerHandlerAdapter 等）。这些处理器可能有不同的方法签名和交互方式。

**HandlerAdapter 的主要职责就是调用 Handler 的方法来处理请求，并且适配不同类型的处理器。**HandlerAdapter 确保 DispatcherServlet 可以以统一的方式调用不同类型的处理器，无需关心具体的执行细节。





### SpringMVC Restful 风格的接口的流程是什么样的呢？

我们都知道 Restful 接口，响应格式是 json，这就用到了一个常用注解：**@ResponseBody**

```java
    @GetMapping("/user")
    @ResponseBody
    public User user(){
        return new User(1,"张三");
    }
```



加入了这个注解后，整体的流程上和使用 ModelAndView 大体上相同，但是细节上有一些不同：

[![Spring MVC Restful请求响应示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191621074.png)](https://camo.githubusercontent.com/6e5d67c6d2c6b37fc359479e1110d2efb55f63ea69d44293bc901f638a866b3b/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d32646139363361302d356461392d346233612d616166642d6664386462633765313830372e706e67)

1. 客户端向服务端发送一次请求，这个请求会先到前端控制器 DispatcherServlet

2. DispatcherServlet 接收到请求后会调用 HandlerMapping 处理器映射器。由此得知，该请求该由哪个 Controller 来处理

3. DispatcherServlet 调用 HandlerAdapter 处理器适配器，告诉处理器适配器应该要去执行哪个 Controller

4. Controller 被封装成了 ServletInvocableHandlerMethod，HandlerAdapter 处理器适配器去执行 invokeAndHandle 方法，完成对 Controller 的请求处理

5. HandlerAdapter 执行完对 Controller 的请求，会调用 HandlerMethodReturnValueHandler 去处理返回值，主要的过程：

   5.1. 调用 RequestResponseBodyMethodProcessor，创建 ServletServerHttpResponse（Spring 对原生 ServerHttpResponse 的封装）实例

   5.2.使用 HttpMessageConverter 的 write 方法，将返回值写入 ServletServerHttpResponse 的 OutputStream 输出流中

   5.3.在写入的过程中，会使用 JsonGenerator（默认使用 Jackson 框架）对返回值进行 Json 序列化

6. 执行完请求后，返回的 ModealAndView 为 null，ServletServerHttpResponse 里也已经写入了响应，所以不用关心 View 的处理





## Spring Boot

### 介绍一下 SpringBoot，有哪些优点？

Spring Boot 提供了一套默认配置，它通过**约定大于配置**的理念，来帮助我们快速搭建 Spring 项目骨架。

以前的 Spring 开发需要配置大量的 xml 文件，并且需要引入大量的第三方 jar 包，还需要手动放到 classpath 下。现在只需要引入一个 Starter，或者一个注解，就可以轻松搞定。

Spring Boot 的优点非常多，比如说：

1. Spring Boot 内嵌了 Tomcat、Jetty、Undertow 等容器，直接运行 jar 包就可以启动项目。
2. Spring Boot 内置了 Starter 和自动装配，避免繁琐的手动配置。例如，如果项目中添加了 spring-boot-starter-web，Spring Boot 会自动配置 Tomcat 和 Spring MVC。
3. Spring Boot 内置了 Actuator 和 DevTools，便于调试和监控。

#### Spring Boot常用注解有哪些？

1. **@SpringBootApplication**：Spring Boot 应用的入口，用在启动类上。
2. 还有一些 Spring 框架本身的注解，比如 **@Component**、**@RestController**、**@Service**、**@ConfigurationProperties**、**@Transactional** 等。





### :star:SpringBoot 自动配置原理了解吗？

在 Spring 中，自动装配是指容器利用**反射**技术，根据 Bean 的类型、名称等自动注入所需的依赖。

[![三分恶面渣逆袭：SpringBoot自动配置原理](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191624739.png)](https://camo.githubusercontent.com/eb2541c62159152c1c0364b608e1dff3c74440d37f71216f5f798069b0c3c98a/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d64663737656531352d326666302d346563372d386536352d6534656262386261383866312e706e67)

在 Spring Boot 中，开启自动装配的注解是`@EnableAutoConfiguration`。

**`@SpringBootApplication` 是一个复合注解，包含了 `@EnableAutoConfiguration`、`@ComponentScan` 和 `@SpringBootConfiguration`。**

其中，`@EnableAutoConfiguration` 是自动配置的核心。

它通过 **`@Import`** 注解引入了 `AutoConfigurationImportSelector` 类。

`AutoConfigurationImportSelector` 会从 **`META-INF/spring.factories` 文件中读取所有自动配置类的全限定类名**。



总结：Spring Boot 的自动装配原理依赖于 Spring 框架的**依赖注入和条件注册**，通过这种方式，Spring Boot 能够智能地配置 bean，并且**只有当这些 bean 实际需要时才会被创建和配置**。



#### 自动配置优先级与排除

- **排除自动配置**：可以使用 `@SpringBootApplication(exclude = {类名.class})` 或 `spring.autoconfigure.exclude` 来排除不需要的自动配置类。
- **优先级管理**：通过 `@Order` 或 `@Priority` 调整 Bean 的装配顺序。





### 如何自定义一个 SpringBoot Starter?

Spring Boot Starter 是一组预先配置好的依赖和自动配置的集合。它们使得开发者在使用常见的框架、库或技术时，可以免去繁琐的配置和集成工作，只需引入相应的 Starter 依赖即可。

- 第一步，创建一个新的 Maven 项目，例如命名为 my-spring-boot-starter。在 pom.xml 文件中添加必要的依赖和配置：
- 第二步，在 `src/main/java` 下创建一个自动配置类，比如 MyServiceAutoConfiguration.java：（通常是 autoconfigure 包下）。
- 第三步，创建一个配置属性类 MyStarterProperties.java：
- 第四步，创建一个简单的服务类 MyService.java：
- 第五步，配置 spring.factories，在 `src/main/resources/META-INF` 目录下创建 spring.factories 文件，并添加：
- 第六步，使用 Maven 打包这个项目：
- 第七步，在其他的 Spring Boot 项目中，通过 Maven 来添加这个自定义的 Starter 依赖，并通过 application.properties 配置欢迎消息：



#### Spring Boot Starter 的原理了解吗？

Spring Boot Starter 主要**通过起步依赖和自动配置机制来简化项目的构建和配置过程**。

起步依赖是 Spring Boot 提供的一组预定义依赖项，它们将一组相关的库和模块打包在一起。比如 `spring-boot-starter-web` 就包含了 Spring MVC、Tomcat 和 Jackson 等依赖。

自动配置机制是 Spring Boot 的核心特性，通过自动扫描类路径下的类、资源文件和配置文件，自动创建和配置应用程序所需的 Bean 和组件。

比如有了 `spring-boot-starter-web`，我们开发者就不需要再手动配置 Tomcat、Spring MVC 等，Spring Boot 会自动帮我们完成这些工作。





### Spring Boot 启动原理了解吗？

在启动类中，main 方法会调用 **`SpringApplication.run()`** 启动项目。

Spring Boot 的启动由 SpringApplication 类负责：

- 第一步，**创建 SpringApplication 实例**，负责应用的启动和初始化；
- 第二步，从 **application.yml 中加载配置文件**和环境变量；
- 第三步，创建上下文环境 ApplicationContext，并加载 Bean，完成**依赖注入**；
- 第四步，启动内嵌的 **Web 容器**。
- 第五步，发布启动完成事件 ApplicationReadyEvent，并**调用 ApplicationRunner 的 run 方法**完成启动后的逻辑。

[![SpringBoot 启动大致流程-图片来源网络](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191907345.png)](https://camo.githubusercontent.com/98b0abba59bcafeb9e0e93c41ae17d70066ce75d81d4c30a4cc5812ef01534fa/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d36383734343535362d613162612d346531662d613039322d3135383238373566306461362e706e67)

SpringApplication.run() 方法负责 Spring 应用的上下文环境（ApplicationContext）准备，包括：

- 扫描配置文件，添加依赖项
- 初始化和加载 Bean 定义
- 启动内嵌的 Web 容器等
- 发布启动完成事件



#### 了解@SpringBootApplication 注解吗？

`@SpringBootApplication`是 Spring Boot 的核心注解，经常用于主类上，作为项目启动入口的标识。它是一个组合注解：

- `@SpringBootConfiguration`：继承自 `@Configuration`，标注该类是一个配置类，相当于一个 Spring 配置文件。
- `@EnableAutoConfiguration`：告诉 Spring Boot 根据 pom.xml 中添加的依赖自动配置项目。例如，如果 spring-boot-starter-web 依赖被添加到项目中，Spring Boot 会自动配置 Tomcat 和 Spring MVC。
- `@ComponentScan`：扫描当前包及其子包下被`@Component`、`@Service`、`@Controller`、`@Repository` 注解标记的类，并注册为 Spring Bean。



#### 为什么 Spring Boot 在启动的时候能够找到 main 方法上的@SpringBootApplication 注解？

Spring Boot 在启动时能够找到主类上的`@SpringBootApplication`注解，是因为它利用了 Java 的**反射机制**和**类加载机制**，结合 Spring 框架内部的一系列处理流程。

当运行一个 Spring Boot 程序时，通常会调用主类中的`main`方法，这个方法会执行`SpringApplication.run()`，

`SpringApplication.run(Class<?> primarySource, String... args)`方法接收两个参数：第一个是主应用类（即包含`main`方法的类），第二个是命令行参数。`primarySource`参数提供了一个起点，Spring Boot 通过它来加载应用上下文。

Spring Boot 利用 Java 反射机制来读取传递给`run`方法的类（`MyApplication.class`）。它会检查这个类上的注解，包括`@SpringBootApplication`。



#### Spring Boot 默认的包扫描路径是什么？

Spring Boot 的默认包扫描路径是以启动类 `@SpringBootApplication` 注解所在的包为根目录的，即默认情况下，Spring Boot 会**扫描启动类所在包及其子包下的所有组件**。

`@SpringBootApplication` 是一个组合注解，它里面的`@ComponentScan`注解可以指定要扫描的包路径，默认扫描启动类所在包及其子包下的所有组件。

比如说带有 `@Component`、`@Service`、`@Controller`、`@Repository` 等注解的类都会被 Spring Boot 扫描到，并注册到 Spring 容器中。

如果需要自定义包扫描路径，可以在`@SpringBootApplication`注解上添加`@ComponentScan`注解，指定要扫描的包路径。





### SpringBoot 和 SpringMVC 的区别？

- Spring MVC 是基于 Spring 框架的一个模块，提供了一种 Model-View-Controller（模型-视图-控制器）的开发模式。
- Spring Boot 旨在简化 Spring 应用的配置和部署过程，提供了大量的自动配置选项，以及运行时环境的内嵌 Web 服务器，这样就可以更快速地开发一个 SpringMVC 的 Web 项目。



### Spring Boot 和 Spring 有什么区别？

Spring Boot 是 **Spring Framework** 的一个扩展，提供了一套快速配置和开发的机制，可以帮助我们快速搭建 Spring 项目的骨架，提高生产效率。

| 特性           | Spring Framework                      | Spring Boot                                   |
| -------------- | ------------------------------------- | --------------------------------------------- |
| **目的**       | 提供企业级的开发工具和库              | 简化 Spring 应用的开发、配置和部署            |
| **配置方式**   | 主要通过 XML 和注解等手动配置         | 提供开箱即用的自动配置                        |
| **启动和运行** | 需要打成 war 包到 Tomcat 等容器下运行 | 已嵌入 Tomcat 等容器，打包成 JAR 文件直接运行 |
| **依赖管理**   | 手动添加和管理依赖                    | 使用 `spring-boot-starter` 简化依赖管理       |





## Spring Cloud

### 对 SpringCloud 了解多少？

Spring Cloud 是一个基于 Spring Boot，提供构建分布式系统和微服务架构的工具集。用于解决分布式系统中的一些常见问题，如配置管理、服务发现、负载均衡等等。

#### 有哪些主流微服务框架？

1. Spring Cloud Netflix
2. Spring Cloud Alibaba
3. SpringBoot + Dubbo + ZooKeeper

#### SpringCloud 有哪些核心组件？

[![SpringCloud](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191922494.png)](https://camo.githubusercontent.com/0f7273ab325c4ecd35c56964e5e8117b05e5cf26cc1ed4a594854a6d6b823ad0/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f737072696e672d32623938386137322d303733392d346665642d623237312d6561663132353839343434662e706e67)





### Spring Cache 了解吗？

Spring Cache 是 Spring 框架提供的一个缓存抽象，它通过统一的接口来支持多种缓存实现（如 Redis、Caffeine 等）。

它通过注解（如 `@Cacheable`、`@CachePut`、`@CacheEvict`）来实现缓存管理，极大简化了代码实现。

- @Cacheable：缓存方法的返回值。
- @CachePut：用于更新缓存，每次调用方法都会将结果重新写入缓存。
- @CacheEvict：用于删除缓存。
- @Caching：组合多个缓存操作。

#### Spring Cache 和 Redis 有什么区别？

1. **Spring Cache** 是 Spring 框架提供的一个缓存抽象，它通过注解来实现缓存管理，支持多种缓存实现（如 Redis、Caffeine 等）。
2. **Redis** 是一个分布式的缓存中间件，支持多种数据类型（如 String、Hash、List、Set、ZSet），还支持持久化、集群、主从复制等。

Spring Cache 适合用于单机、轻量级和短时缓存场景，能够通过注解轻松控制缓存管理。

Redis 是一种分布式缓存解决方案，支持多种数据结构和高并发访问，适合分布式系统和高并发场景，可以提供数据持久化和多种淘汰策略。

在实际开发中，Spring Cache 和 Redis 可以结合使用，Spring Cache 提供管理缓存的注解，而 Redis 则作为分布式缓存的实现，提供共享缓存支持。



#### 有了 Redis 为什么还需要 Spring Cache？

虽然 Redis 非常强大，但 Spring Cache 提供了一层缓存抽象，简化了缓存的管理。我们可以直接在方法上通过注解来实现缓存逻辑，减少了手动操作 Redis 的代码量。

Spring Cache 还能灵活切换底层缓存实现。此外，Spring Cache 支持事务性缓存和条件缓存，便于在复杂场景中确保数据一致性。



#### 说说Spring Cache 的底层原理？

Spring Cache 是基于 **AOP 和缓存抽象层（代理模式）**实现的。它通过 AOP 拦截被 @Cacheable、@CachePut 和 @CacheEvict 注解的方法，在方法调用前后自动执行缓存逻辑。

[![铿然架构：Spring Cache 架构](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412191929500.png)](https://camo.githubusercontent.com/f0f7fa3c5fe2f6ae5d5fcf3a1f964b6b82cb4f55a390946daf313102b637a19f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f737072696e672d32303234313033313131333734332e706e67)

其提供的 CacheManager 和 Cache 等接口，不依赖具体的缓存实现，因此可以灵活地集成 Redis、Caffeine 等多种缓存。

- ConcurrentMapCacheManager：基于 Java ConcurrentMap 的本地缓存实现。
- RedisCacheManager：基于 Redis 的分布式缓存实现。
- CaffeineCacheManager：基于 Caffeine 的缓存实现。















































































