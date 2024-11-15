# Redis重难点

[TOC]

## 1、缓存

### 1.1、缓存更新策略

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858076.png" alt="image-20241106185200935" style="zoom: 50%;" />

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858077.png" alt="image-20241106191046947" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858078.png" alt="image-20241106191344652" style="zoom:50%;" />

涉及到**数据同步**问题，可以参考数据同步四种方案



### 1.2、缓存穿透

**避免数据库也查不到，还把null存入缓存，那么以后缓存就永远不生效**

![image-20241106191939515](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858079.png)

缓存穿透产生的**原因**是什么?
用户请求的数据在缓存中和数据库中都不存在，不断发起这样的请求给数据库带来巨大压力

缓存穿透的**解决方案**有哪些?

1. **缓存空对象null**

2. **布隆过滤**

3. 增强id的复杂度，避免被猜测id规律

4. **检验参数**

   我们可以对用户id做检验。

   比如你的合法id是15xxxxxx，以15开头的。如果用户传入了16开头的id，比如：16232323，则参数校验失败，直接把相关请求拦截掉。这样可以过滤掉一部分恶意伪造的用户id。

5. 做好数据的基础格式校验

6. 加强用户权限校验

7. 做好热点参数的限流

#### 1.2.1、布隆过滤器

如果数据比较少，我们可以把数据库中的数据，全部放到内存的一个map中。

这样能够非常快速的识别，数据在缓存中是否存在。如果存在，则让其访问缓存。如果不存在，则直接拒绝该请求。

但如果数据量太多了，有数千万或者上亿的数据，全都放到内存中，很显然会占用太多的内存空间。

那么，有没有办法减少内存空间呢？

答：这就需要使用**布隆过滤器**了。

布隆过滤器底层使用**bit数组**存储数据，该数组中的元素默认值是0。

布隆过滤器第一次初始化的时候，会把数据库中所有已存在的key，经过一些列的hash算法（比如：三次hash算法）计算，每个key都会计算出多个位置，然后把这些位置上的元素值设置成1。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858080.png" alt="image-20241106194101548" style="zoom: 67%;" />

之后，有用户key请求过来的时候，再用相同的hash算法计算位置。

- 如果多个位置中的元素值都是1，则说明该key在数据库中已存在。这时允许继续往后面操作。
- 如果有1个以上的位置上的元素值是0，则说明该key在数据库中不存在。这时可以拒绝该请求，而直接返回。

使用布隆过滤器确实可以解决缓存穿透问题，但同时也带来了两个问题：

1. 存在误判的情况。
2. 存在数据更新问题。

先看看为什么会存在误判呢？

上面我已经说过，初始化数据时，针对每个key都是通过多次hash算法，计算出一些位置，然后把这些位置上的元素值设置成1。

但我们都知道hash算法是会出现hash冲突的，也就是说不同的key，可能会计算出相同的位置。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858081.png" alt="image-20241106194112897" style="zoom:67%;" />

上图中的下标为2的位置就出现了hash冲突，key1和key2计算出了一个相同的位置。

如果有几千万或者上亿的数据，布隆过滤器中的hash冲突会非常明显。

如果某个用户key，经过多次hash计算出的位置，其元素值，恰好都被其他的key初始化成了1。此时，就出现了误判，原本这个key在数据库中是不存在的，但布隆过滤器确认为存在。

如果布隆过滤器判断出某个key存在，可能出现误判。如果判断某个key不存在，则它在数据库中一定不存在。

通常情况下，布隆过滤器的误判率还是比较少的。即使有少部分误判的请求，直接访问了数据库，但如果访问量并不大，对数据库影响也不大。

此外，如果想减少误判率，可以适当增加hash函数，图中用的3次hash，可以增加到5次。

其实，布隆过滤器最致命的问题是：如果数据库中的数据更新了，需要同步更新布隆过滤器。但它跟数据库是两个数据源，就可能存在**数据不一致**的情况。

比如：数据库中新增了一个用户，该用户数据需要实时同步到布隆过滤。但由于网络异常，同步失败了。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858082.png" alt="image-20241106194122881" style="zoom:50%;" />

这时刚好该用户请求过来了，由于布隆过滤器没有该key的数据，所以直接拒绝了该请求。但这个是正常的用户，也被**拦截**了。

很显然，如果出现了这种正常用户被拦截了情况，有些业务是无法容忍的。所以，布隆过滤器要看实际业务场景再决定是否使用，它帮我们解决了缓存穿透问题，但同时了带来了新的问题。



#### 1.2.2、缓存空对象null

上面使用布隆过滤器，虽说可以过滤掉很多不存在的用户id请求。但它除了增加系统的复杂度之外，会带来两个问题：

1. 布隆过滤器存在误杀的情况，可能会把少部分正常用户的请求也过滤了。
2. 如果用户信息有变化，需要实时同步到布隆过滤器，不然会有问题。

所以，通常情况下，我们很少用布隆过滤器解决缓存穿透问题。其实，还有另外一种更简单的方案，即：**缓存空值**。

当某个用户id在缓存中查不到，在数据库中也查不到时，也需要将该用户id缓存起来，只不过值是空的。这样后面的请求，再拿相同的用户id发起请求时，就能从缓存中获取空数据，直接返回了，而无需再去查一次数据库。

优化之后的流程图如下：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858083.png"  />

 关键点是不管从数据库有没有查到数据，都将结果放入缓存中，只是如果没有查到数据，缓存中的值是null的罢了。



### 1.3、缓存击穿

缓存击穿问题，也叫 **热点 Key** 问题；就是一个被 高并发访问 并且 缓存中业务较复杂的 Key 突然失效，大量的请求在极短的时间内一起请求这个 Key 并且都未命中，无数的请求访问在瞬间打到数据库上，给数据库带来巨大的冲击。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858085.png" alt="image-20241106194458477" style="zoom: 50%;" />

**解决方案**：

- 互斥锁：查询缓存未命中，获取互斥锁，获取到互斥锁的才能查询数据库重建缓存，将数据写入缓存中后，释放锁。
- 逻辑过期：查询缓存，发现逻辑时间已经过期，获取互斥锁，开启新线程；在新线程中查询数据库重建缓存，将数据写入缓存中后，                   释放锁；在释放锁之前，查询该数据时，都会将过期的数据返回。

#### 1.3.1、互斥锁

核心：利用 Redis 的 **setnx** 方法来表示获取锁。该方法的含义是：如果 Redis 中没有这个 Key，则插入成功；如果有这个 Key，则插入失败。通过插入成功或失败来表示是否有线程插入 Key，插入成功的 Key 则认为是获取到锁的线程；释放锁就是将这个 Key 删除，因为删除 Key 以后其他线程才能再执行 setnx 方法。

stenx lock 1

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858086.png" alt="image-20241106195021768" style="zoom: 50%;" />


```java
try{
    // 获取互斥锁
    Boolean flag = redisTemplate.opsForValue().setIfAbsent(key, "1", TTL_TEN, TimeUnit.SECONDS);
    return BooleanUtil.isTrue(flag);
}finally{
    // 释放互斥锁
    redisTemplate.delete(key);
}
return null;

```



#### 1.3.2、逻辑过期

![image-20241106200222497](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858087.png)



#### 1.3.3、缓存不失效预热

对于很多热门key，其实是可以不用设置过期时间，让其永久有效的。

比如参与秒杀活动的热门商品，由于这类商品id并不多，在缓存中我们可以不设置过期时间。

在秒杀活动开始前，我们先用一个程序提前从数据库中查询出商品的数据，然后同步到缓存中，提前做**预热**。

等秒杀活动结束一段时间之后，我们再**手动删除**这些无用的缓存即可。



#### 1.3.4、自动续期（非缓存击穿）

**（续约第三方平台接口token情况）**

出现缓存击穿问题是由于key过期了导致的。那么，我们换一种思路，在key快要过期之前，就自动给它续期，不就OK了？

答：没错，我们可以用job给指定key自动续期。

比如说，我们有个分类功能，设置的缓存过期时间是30分钟。但有个job每隔20分钟执行一次，自动更新缓存，重新设置过期时间为30分钟。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858088.png" alt="image-20241106200930498" style="zoom: 50%;" />

这样就能保证，分类缓存不会失效。

此外，在很多请求第三方平台接口时，我们往往需要先调用一个获取token的接口，然后用这个token作为参数，请求真正的业务接口。一般获取到的token是有有效期的，比如24小时之后失效。

如果我们每次请求对方的业务接口，都要先调用一次获取token接口，显然比较麻烦，而且性能不太好。

这时候，我们可以把第一次获取到的token缓存起来，请求对方业务接口时从缓存中获取token。

同时，有一个job每隔一段时间，比如每隔12个小时请求一次获取token接口，不停刷新token，重新设置token的过期时间。



### 1.4、缓存雪崩

而缓存雪崩是缓存击穿的升级版，缓存击穿说的是某一个热门key失效了，而缓存雪崩说的是有多个热门key同时失效。看起来，如果发生缓存雪崩，问题更严重。

缓存雪崩目前有两种：

1. 有大量的热门缓存，同时失效。会导致大量的请求，访问数据库。而数据库很有可能因为扛不住压力，而直接挂掉。
2. 缓存服务器down机了，可能是机器硬件问题，或者机房网络问题。总之，造成了整个缓存的不可用。

归根结底都是有大量的请求，透过缓存，而直接访问数据库了。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858089.png" alt="image-20241106192418938" style="zoom: 67%;" />

**解决方案**：

- 给不同的Key的**TTL添加随机值**
- **高可用**利用Redis集群提高服务的可用性
- 给缓存业务添加**降级限流**策略
- 给业务添加**多级缓存**



## 2、乐观锁

### 2.1、CAS法

![image-20241106202707969](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858090.png)

### 2.2、ABA问题

CAS会导致“ABA问题”。

CAS算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。

比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。

线程1准备用CAS将变量的值由A替换为B，在此之前，线程2将变量的值由A替换为C，又由C替换为A，然后线程1执行CAS时发现变量的值仍然为A，所以CAS成功。但实际上这时的现场已经和最初不同了，尽管CAS成功，但可能存在潜藏的问题，例如下面的例子：



部分乐观锁的实现是通过**版本号**（version）的方式来解决ABA问题，乐观锁每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1操作，否则就执行失败。因为每次操作的版本号都会随之增加，所以不会出现ABA问题，因为版本号只会增加不会减少。



**只能保证一个共享变量的原子操作**。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。

**比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij**。

从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，**你可以把多个变量放在一个对象里来进行CAS操作**。



### 2.3、版本号法

![image-20241106202924506](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858091.png)



## 3、分布式锁

### 3.1、常见分布式锁方案对比

| 分类                              | 方案                                                         | 实现原理                                                     | 优点                                                         | 缺点                                                         |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 基于数据库                        | 基于mysql 表唯一索引                                         | 1.表增加唯一索引 2.加锁：执行insert语句，若报错，则表明加锁失败 3.解锁：执行delete语句 | 完全利用DB现有能力，实现简单                                 | 1.锁无超时自动失效机制，有死锁风险 2.不支持锁重入，不支持阻塞等待 3.操作数据库开销大，性能不高 |
| 基于MongoDB findAndModify原子操作 | 1.加锁：执行findAndModify原子命令查找document，若不存在则新增 2.解锁：删除document | 实现也很容易，较基于MySQL唯一索引的方案，性能要好很多        | 1.大部分公司数据库用MySQL，可能缺乏相应的MongoDB运维、开发人员 2.锁无超时自动失效机制 |                                                              |
| 基于分布式协调系统                | 基于ZooKeeper                                                | 1.加锁：在/lock目录下创建临时有序节点，判断创建的节点序号是否最小。若是，则表示获取到锁；否，则则watch /lock目录下序号比自身小的前一个节点 2.解锁：删除节点 | 1.由zk保障系统高可用 2.Curator框架已原生支持系列分布式锁命令，使用简单 | 需单独维护一套zk集群，维保成本高                             |
| 基于缓存                          | 基于redis命令                                                | 1. 加锁：执行setnx，若成功再执行expire添加过期时间 2. 解锁：执行delete命令 | 实现简单，相比数据库和分布式系统的实现，该方案最轻，性能最好 | 1.setnx和expire分2步执行，非原子操作；若setnx执行成功，但expire执行失败，就可能出现死锁 2.delete命令存在误删除非当前线程持有的锁的可能 3.不支持阻塞等待、不可重入 |
| 基于redis Lua脚本能力             | 1. 加锁：执行SET lock_name random_value EX seconds NX 命令  2. 解锁：执行Lua脚本，释放锁时验证random_value  -- ARGV[1]为random_value, KEYS[1]为lock_nameif redis.call("get", KEYS[1]) == ARGV[1] then  return redis.call("del",KEYS[1])else  return 0end | 同上；实现逻辑上也更严谨，除了单点问题，生产环境采用用这种方案，问题也不大。 | 不支持锁重入，不支持阻塞等待                                 |                                                              |



**分布式锁需满足四个条件**

首先，为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：

1. 互斥性。在任意时刻，只有一个客户端能持有锁。
2. 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
3. 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了，即不能误解锁。
4. 具有容错性。只要大多数Redis节点正常运行，客户端就能够获取和释放锁。



### 3.2、Redission

#### 3.2.1、配置环境

导入依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.13.6</version>
</dependency>
```

RedissonClient配置

```java
@Configuration
public class RedissionConfig {
    @Bean
    public RedissonClient redissionClient(){
        //配置
        Config config = new Config();
        config.useSingleServer().setAddress("redis://192.168.200.131:6379").setPassword("1234");
        //创建RedissonClient对象
        return Redisson.create(config);
    }
}
```

#### 3.2.2、可重入锁原理

> 为什么要引入可重入锁这种机制?
>
> 我们知道“对象一把锁，多个对象多把锁”，可重入锁的概念就是：自己可以获取自己的内部锁。
>
> 假如有一个线程 T 获得了对象 A 的锁，那么该线程 T 如果在未释放前再次请求该对象的锁时，如果没有可重入锁的机制，是不会获取到锁的，这样的话就会出现死锁的情况

这里除了记录key和ThreaId外，还需有记录重入次数，所以我们需要用hash, 每次的thead一样时，次数+1；

直到次数为0的时候才可以删除锁

> nx ：判断锁是否存在
>
> ex： 设置过期时间
>
> 但是Redis的hash结构没有nx这个命令，所以我们只能先判是否存在(exist)，再设置过期时间

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858092.png" alt="image-20241106225831769" style="zoom: 67%;" />

#### 3.2.3、锁重试tryLock和看门狗WatchDog

![image-20241106230628141](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858093.png)

**1、锁重试：**获取锁失败后重新获取

**tryLock(waitTime,leaseTime,TimeUnit)**

waitTime:获取锁的等待时长，获取锁失败后等待waitTime再去获取锁

leaseTime: 锁自动失效时间，这里测试锁重试不需要用到



**2、WatchDog**-----超时释放

对抢锁过程进行监听，抢锁完毕后，scheduleExpirationRenewal(threadId) 方法会被调用来对锁的过期时间进行续约，在后台开启一个线程，进行续约逻辑，也就是看门狗线程。

```java
// 续约逻辑
commandExecutor.getConnectionManager().newTimeout(new TimerTask() {... }, 锁失效时间 / 3, TimeUnit.MILLISECONDS);

Method(new TimerTask(){}, 参数2, 参数3)
```

通过参数2、参数3 去描述，什么时候做参数1 的事情。

锁的失效时间为 30s，10s 后这个 TimerTask 就会被触发，于是进行续约，将其续约为 30s；
若操作成功，则递归调用自己，重新设置一个 TimerTask 并且在 10s 后触发；循环往复，不停的续约。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858094.png" alt="image-20241106230736344" style="zoom:50%;" />

#### 3.2.4、MutiLock锁

为了提高redis的可用性，我们会搭建集群或者主从，现在以主从为例

此时我们去写命令，写在主机上， 主机会将数据同步给从机，但是假设在主机还没有来得及把数据写入到从机去的时候，此时主机宕机，哨兵会发现主机宕机，并且选举一个slave变成master，而此时新的master中实际上并没有锁信息，此时锁信息就已经丢掉了，此时如果有另一个线程去从机上获得锁就会成功，造成**一锁被多建多用**。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858095.png" alt="image-20241106232128038" style="zoom:50%;" />

为了解决这个问题，redission提出来了**MutiLock锁**，使用这把锁咱们就**不使用主从**了，**使用多个独立的master**，每个节点的地位都是一样的，这把锁加锁的逻辑需要写入到每一个主丛节点上，只有所有的服务器都写入成功，此时才是加锁成功，假设现在某个节点挂了，那么他去获得锁的时候，只要有一个节点拿不到，都不能算是加锁成功，就保证了加锁的可靠性。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858096.png" alt="image-20241106232059050" style="zoom:50%;" />

那么MutiLock（连锁） 加锁原理是什么呢？笔者画了一幅图来说明

> **MutiLock（连锁）：**当我们去设置了多个锁时，redission会将多个锁添加到一个集合中，然后用while循环去不停去尝试拿锁，但是会有一个总共的加锁时间，这个时间是用 需要加锁的个数 * 1500ms ，假设有3个锁，那么时间就是4500ms，假设在这4500ms内，所有的锁都加锁成功（获取到的锁会放到一个acquiredLocks列表里），那么此时才算是加锁成功，如果在4500ms有线程加锁失败，则会释放掉所有已经获得成功的锁，再次去进行重试。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858097.png" alt="image-20241106232145788" style="zoom:200%;" />

**使用：**

```java
RLock lock1 = redissonClient.getLock("order" + userId);
RLock lock2 = redissonClient.getLock("order" + userId);
RLock lock3 = redissonClient.getLock("order" + userId);
RLock multiLock = redissonClient.getMultiLock(lock1, lock2, lock3);
```



## 4、Feed流的滚动分页

我们需要记录每次操作的最后一条，然后从这个位置开始去读取数据

举个例子：我们从t1时刻开始，拿第一页数据，拿到了10~6，然后记录下当前最后一次拿取的记录，就是6，t2时刻发布了新的记录，此时这个11放到最顶上，但是不会影响我们之前记录的6，此时t3时刻来拿第二页，第二页这个时候拿数据，还是从6后一点的5去拿，就拿到了5-1的记录。我们这个地方可以采用sortedSet来做，可以进行范围查询，并且还可以记录当前获取数据时间戳最小值，就可以实现滚动分页了

[<img src="https://github.com/spongehah/redis/raw/main/%E7%AC%94%E8%AE%B0/image/Redis%E5%AE%9E%E6%88%98%E7%AF%87.assets/1653813462834.png" alt="1653813462834" style="zoom: 80%;" />

**我们需要使用ZSet的ZREVRANGEBYSCORES命令进行滚动分页的查询**

```
ZREVRANGEBYSCORE key max min WITHSCORES LIMIT offset count
```



| 指令             | 是否必须 | 说明                                                    |
| ---------------- | -------- | ------------------------------------------------------- |
| ZREVRANGEBYSCORE | 是       | 指令                                                    |
| key              | 是       | 有序集合键名称                                          |
| max              | 是       | 最大分数值,可使用"+inf"代替                             |
| min              | 是       | 最小分数值,可使用"-inf"代替                             |
| WITHSCORES       | 否       | 将成员分数一并返回                                      |
| LIMIT            | 否       | 返回结果是否分页,指令中包含LIMIT后offset、count必须输入 |
| offset           | 否       | 返回结果起始位置（偏移量）                              |
| count            | 否       | 返回结果数量                                            |

所以我们需要max（lastId，第一次时为当前时间戳）、offset（最小的时间戳相同的个数就是跳过的偏移量）这两个参数

具体操作如下：

1、每次查询完成后，我们要分析出查询出数据的最小时间戳，这个值会作为下一次查询的条件

2、我们需要找到与上一次查询相同的查询个数作为偏移量，下次查询时，跳过这些查询过的数据，拿到我们需要的数据

综上：我们的请求参数中就需要携带 lastId：上一次查询的最小时间戳 和偏移量这两个参数。

这两个参数第一次会由前端来指定，以后的查询就根据后台结果作为条件，再次传递到后台。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858098.png" alt="image-20241106235220641" style="zoom: 67%;" />



**代码实现：**

BlogController

注意：RequestParam 表示接受url地址栏传参的注解，当方法上参数的名称和url地址栏不相同时，可以通过RequestParam 来进行指定

```java
@GetMapping("/of/follow")
public Result queryBlogOfFollow(
    @RequestParam("lastId") Long max, @RequestParam(value = "offset", defaultValue = "0") Integer offset){
    return blogService.queryBlogOfFollow(max, offset);
}
```

BlogServiceImpl

```java
@Override
public Result queryBlogOfFollow(Long max, Integer offset) {
    // 1.获取当前用户
    Long userId = UserHolder.getUser().getId();
    // 2.查询收件箱 ZREVRANGEBYSCORE key Max Min LIMIT offset count
    String key = FEED_KEY + userId;
    Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet()
        .reverseRangeByScoreWithScores(key, 0, max, offset, 2);
    // 3.非空判断
    if (typedTuples == null || typedTuples.isEmpty()) {
        return Result.ok();
    }
    // 4.解析数据：blogId、minTime（时间戳）、offset
    List<Long> ids = new ArrayList<>(typedTuples.size());
    long minTime = 0; // 2
    int os = 1; // 2
    for (ZSetOperations.TypedTuple<String> tuple : typedTuples) { // 5 4 4 2 2
        // 4.1.获取id
        ids.add(Long.valueOf(tuple.getValue()));
        // 4.2.获取分数(时间戳）
        long time = tuple.getScore().longValue();
        if(time == minTime){
            os++;
        }else{
            minTime = time;
            os = 1;
        }
    }
    //若是整页都与上一页的最小值的score相等，则跳过的数量还需增加
	os = minTime == max ? os : os + offset;
    // 5.根据id查询blog
    String idStr = StrUtil.join(",", ids);
    List<Blog> blogs = query().in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list();

    for (Blog blog : blogs) {
        // 5.1.查询blog有关的用户
        queryBlogUser(blog);
        // 5.2.查询blog是否被点赞
        isBlogLiked(blog);
    }

    // 6.封装并返回
    ScrollResult r = new ScrollResult();
    r.setList(blogs);
    r.setOffset(os);
    r.setMinTime(minTime);

    return Result.ok(r);
}
```

## 5、UV统计-HyperLogLog

首先我们搞懂两个概念：

- UV：全称Unique Visitor，也叫独立访客量，是指通过互联网访问、浏览这个网页的自然人。1天内同一个用户多次访问该网站，只记录1次。
- PV：全称Page View，也叫页面访问量或点击量，用户每访问网站的一个页面，记录1次PV，用户多次打开页面，则记录多次PV。往往用来衡量网站的流量。

通常来说UV会比PV大很多，所以衡量同一个网站的访问量，我们需要综合考虑很多因素，所以我们只是单纯的把这两个值作为一个参考值

UV统计在服务端做会比较麻烦，因为要判断该用户是否已经统计过了，需要将统计过的用户信息保存。但是如果每个访问的用户都保存到Redis中，数据量会非常恐怖，那怎么处理呢？

Hyperloglog(HLL)是从Loglog算法派生的概率算法，用于确定非常大的集合的基数，而不需要存储其所有值。相关算法原理大家可以参考：https://juejin.cn/post/6844903785744056333#heading-0 Redis中的**HLL是基于string结构实现**的，单个HLL的内存**永远小于16kb**，**内存占用低**的令人发指！作为代价，其测量结果是概率性的，**有小于0.81％的误差**。不过对于UV统计来说，这完全可以忽略。

[![image-20241107000921311](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858099.png)



## 6、布隆过滤器和bitmap区别

### 6.1、bitmap

由于用到哈希运算，就不可避免地会出现数据碰撞问题，即不同的字符串可能得出相同的哈希值。这样一来，状态判断就可能不准确

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858100.png" alt="image-20241107001404881" style="zoom:50%;" />

**操作命令**

- SETBIT：用于设置指定偏移量上的位值，其时间复杂度为 O (1)。例如，当用户答对了第 7 题时，可以将题号对应的偏移量为 7 的位值设置为 1，以此表示该题已被答对。

- GETBIT：获取指定偏移量上的位值，同样具有高效的时间复杂度。可以快速查询用户对特定题号的回答状态。
- BITCOUNT：用于统计位值中被设置为 1 的数量。比如上边我可以很容易统计答对全部题目的用户，但也仅能知道个数，无法查看具体的哪个题目。
- BITOP：对一个或多个 bitmap 进行位运算，并将结果保存到新的键中，支持 AND、OR、NOT、XOR 四种操作。这个命令的用法是将多个bitmap中相同偏移量的位值进行运算。若我想知道用户 1 和用户 2 都答对的题目，经过 AND 运算后，假如只有题号 1 是两个用户都答对的题目，那么生成新的结果集中就只有题号 1 对应的位值为 1。


**优点**
极高空间效率：bitmap 是真的节省数据存储空间。粗略的算一下，一亿位的 Bitmap 大概才占 12MB 的内存，相比其他数据结构，能极大地节省存储空间；

快速查询：位操作通常比其他数据结构查询速度更快。无论是设置位值还是获取位值，时间复杂度都为 O (1)，能够快速响应查询请求；

易于操作：支持单个位操作、位统计、位逻辑运算等，运算效率高，不需要进行比较和移位；

**缺点**
由于数据结构特点，导致它仅适用于表示两种状态，即 0 和 1。对于需要表示更多状态的情况，Bitmap 就不适用了；

只有当数据比较密集时才有优势，如果我们只设置（20，30，888888888）三个偏移量的位值，则需要创建一个 99999999 长度的 BitMap ，但是实际上只存了3个数据，这时候就有很大的空间浪费，碰到这种问题的话，可以通过引入另一个 Roaring BitMap 来解决；

**应用场景**
看到 Bitmap 还是比较简单的一种数据结构，占用空间小查询效率高，非常适用于记录状态的场景，它的应用场景很常见，比如：

- 用户签到状态（连续签到天数）

- 用户的在线状态（统计活跃用户）

- 问卷答题等等吧！

### 6.2、布隆过滤器

上边咱们提到 bitmap 记录字符元素的状态时，需要先借助哈希运算得出偏移量。但引入哈希运算后可能会出现哈希碰撞的情况，导致状态误判。

布隆过滤器对这个问题做了进一步的优化，做到了可控误判率，当我们将一个邮箱地址添加到集合中，多个不同的哈希函数会将这个邮箱地址映射到 bitmap 中的不同偏移量位置上，且将这些位值置为 1。

要判断邮箱地址是否在集合中，通过相同的哈希函数映射到 bitmap 上的多个位置，如果这些位上的值都为 1，则邮箱可能存在集合中；如果有任何一个位置的值为 0，则元素一定不在集合中。这是布隆过滤器的特点。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858101.png" alt="image-20241107001806248" style="zoom: 50%;" />

虽然但是布隆过滤器还是会发生误判的情况，额～，但好在我们可以通过**调整布隆过滤器的大小和哈希函数的数量来控制误判率**

**操作命令**

布隆过滤器的命令也不多，主要用到的如下几个：

- BF.RESERVE：创建一个新的布隆过滤器，并指定容量 capacity 和误判率 error_rate。
- BF.INFO：获取布隆过滤器的信息，包括容量、误判率等。
- BF.ADD 和 BF.MADD：分别是向布隆过滤器中添加元素和批量添加
- BF.EXISTS 和 BF.MEXISTS：分别是检查布隆过滤器中某个元素和批量检查元素是否存在

**优点**
布隆过滤器的空间占用也是极小，它本身不存储完整的数据，和 bitmap 一样底层也是通过 bit 位来表示数据是否存在。

性能比较稳定，无论集合中元素的数量有多少，插入和查询操作的时间复杂度都非常低，通常为 O (k)，其中 k 是哈希函数的个数。也就是说在处理大规模数据时，布隆过滤器的性能不会随着数据量的增加而急剧下降。

**缺点**
存在一定的误识别率：布隆过滤器存在误判的情况，即当一个元素实际上不在集合中时，有可能被判断为在集合中。这是因为多个元素可能通过哈希函数映射到相同的位置，导致误判。但是，当布隆过滤器判断一个元素不在集合中时，则是 100% 正确的。

删除元素比较困难：一般情况下，不能直接从布隆过滤器中删除元素。这是因为一个位置可能被多个元素映射到，如果直接将该位置的值置为 0，可能会影响其他元素的判断。

**应用场景**
布隆过滤器存在一定的误判，所以使用它的场景就一定要允许不准确的情况发生：

- 解决 Redis 缓存穿透问题：秒杀商品详情通常会被缓存到 Redis 中。如果有大量恶意请求查询不存在的商品，通过布隆过滤器可以快速判断这些商品不存在，从而避免了对数据库的查询，减轻了数据库的压力。

- 邮箱黑名单过滤：在邮件系统中，可以使用布隆过滤器来过滤垃圾邮件和恶意邮件。将已知的垃圾邮件发送者的地址或特征存储在布隆过滤器中，新邮件来时判断发送者是否在黑名单中。

- 对爬虫网址进行过滤：在爬虫程序中，为了避免重复抓取相同的网址，可以使用布隆过滤器来记录已经抓取过的网址。新网址出现时，先判断是否已抓取过。


太多太多了…



## 7、分布式缓存

### 7.1、Redis持久化

#### 7.1.1、RDB

RDB全称Redis Database Backup file（Redis数据备份文件），也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。快照文件称为RDB文件，默认是保存在当前运行目录。

##### 7.1.1.1、执行时机

RDB持久化在四种情况下会执行：

1. 执行save命令
2. 执行bgsave命令
3. Redis停机时
4. 触发RDB条件时

**1）save命令**

执行下面的命令，可以立即执行一次RDB：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858102.png" alt="image-20241107115616688" style="zoom:50%;" />

save命令会导致主进程执行RDB，这个过程中其它所有命令都会被阻塞。只有在数据迁移时可能用到。

**2）bgsave命令**

下面的命令可以异步执行RDB：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858103.png" alt="image-20241107115637845" style="zoom:50%;" />

这个命令执行后会开启独立进程完成RDB，主进程可以持续处理用户请求，不受影响。

**3）停机时**

Redis停机时会执行一次save命令，实现RDB持久化。

**4）触发RDB条件**

Redis内部有触发RDB的机制，可以在redis.conf文件中找到，格式如下：

**redis6.0.16以下默认情况：**

```
# 900秒内，如果至少有1个key被修改，则执行bgsave ， 如果是save "" 则表示禁用RDB
save 900 1  
save 300 10  
save 60 10000 
```

**redis6.2和redis7.0.0默认情况：**

```
save 3600 1
save 300 100
save 60 10000
```

RDB的其它配置也可以在redis.conf文件中设置：

```
# 是否压缩 ,建议不开启，压缩也会消耗cpu，磁盘的话不值钱
rdbcompression yes

# RDB文件名称
dbfilename dump.rdb  

# 文件保存的路径目录
dir ./ 
```



##### 7.1.1.2、RDB原理

bgsave开始时会**fork**主进程得到子进程，子进程**共享**主进程的内存数据。完成fork后读取**共享**内存数据并写入 RDB 文件**替换**旧的RDB文件。

当子进程在访问共享内存数据写RDB文件时，**主进程若要执行写操作，采用的是copy-on-write技术**：

- 当主进程执行**读操作**时，访问共享内存；
- 当主进程执行**写操作**时，将共享数据设置为read-only，**拷贝**一份数据，执行**写**操作，之后的**读**也会在副本上进行。

![image-20241107115832534](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858104.png)

##### 7.1.1.3、小结

RDB方式bgsave的基本流程？

- fork主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并**异步**写入新的RDB文件
- 用新RDB文件**替换**旧的RDB文件

RDB会在什么时候执行？save 60 1000代表什么含义？

- 默认是服务停止时
- 执行save和bgsave
- 代表60秒内至少执行1000次修改则触发RDB

RDB的缺点？

- RDB执行间隔时间长，两次RDB之间写入数据有丢失的风险
- fork子进程、压缩、写出RDB文件都比较耗时



#### 7.1.2、AOF持久化

##### 7.1.2.1、AOF原理

AOF全称为Append Only File（追加文件）。Redis处理的每一个写命令都会记录在AOF文件，可以看做是命令日志文件。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858105.png" alt="image-20241107120359856" style="zoom: 50%;" />

##### 7.1.2.2、AOF配置

AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF：

```
# 是否开启AOF功能，默认是no
appendonly yes
# AOF文件的名称
appendfilename "appendonly.aof"
```



AOF的命令记录的频率也可以通过redis.conf文件来配：

```
# 表示每执行一次写命令，立即记录到AOF文件
appendfsync always 
# 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案
appendfsync everysec 
# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```

三种策略对比：

![image-20241107120630274](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858106.png)



##### 7.1.2.3、AOF文件重写

因为是记录命令，AOF文件会比RDB文件大的多。而且AOF会记录对同一个key的多次写操作，但只有最后一次写操作才有意义。通过执行bgrewriteaof命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。

[![image-20210725151729118](https://github.com/spongehah/redis/raw/main/%E7%AC%94%E8%AE%B0/image/Redis%E9%AB%98%E7%BA%A7-%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%93%E5%AD%98.assets/image-20210725151729118.png)](https://github.com/spongehah/redis/blob/main/笔记/image/Redis高级-分布式缓存.assets/image-20210725151729118.png)

如图，AOF原本有三个命令，但是`set num 123 和 set num 666`都是对num的操作，第二次会覆盖第一次的值，因此第一个命令记录下来没有意义。

所以重写命令后，AOF文件内容就是：`mset name jack num 666`

Redis也会在触发阈值时**自动去重写AOF文件**。阈值也可以在redis.conf中配置：

```
# AOF文件比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF文件体积最小多大以上才触发重写 
auto-aof-rewrite-min-size 64mb 
```

> 满足64M的时候，满足percentage才会触发重写，文件太小重写没意义



##### 7.1.2.4、AOF文件格式（Redis7.0以后）

aof文件保存位置：

- **redis6：和RDB保存的位置相同**
- **redis7：会在RAB文件保存的位置上加上一个自己设定的appenddirname目录，保存在其中**

文件格式：

- redis6：apendfilename.aof
- redis7：**aof文件被拆分为三个**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858107.png" alt="image-20241107121742609" style="zoom:50%;" />

[![image-20231013214141914](https://github.com/spongehah/redis/raw/main/%E7%AC%94%E8%AE%B0/image/Redis%E9%AB%98%E7%BA%A7-%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%93%E5%AD%98.assets/image-20231013214141914.png)](https://github.com/spongehah/redis/blob/main/笔记/image/Redis高级-分布式缓存.assets/image-20231013214141914.png)



#### 7.1.3、RDB与AOF对比

RDB和AOF各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会**结合**两者来使用。

1. RDB是完全的异步操作，AOF除了频率为appendfsync always情况外，也都是异步操作。
2. AOF和RDB同时存在的混合模式也是可以的，如果AOF和RDB同时存在的时候，RDB和AOF的写入互不干扰，但是读取的话，Redis会优先使用从AOF文件来还原数据库状态,如果AOF关闭状态时,则从RDB中恢复。
3. 在Redis版本更新的计划中，计划把RDB和AOF两者融合为一种，因为RDB和AOF混合使用非常常见。
4. AOF的重写命令BGREWRITEAOF会占用大量CPU和内存资源。

[![image-20210725151940515](https://github.com/spongehah/redis/raw/main/%E7%AC%94%E8%AE%B0/image/Redis%E9%AB%98%E7%BA%A7-%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%93%E5%AD%98.assets/image-20210725151940515.png)](https://github.com/spongehah/redis/blob/main/笔记/image/Redis高级-分布式缓存.assets/image-20210725151940515.png)



#### 7.1.4、RDB和AOF混合使用

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858108.png" alt="image-20241107122614254" style="zoom: 67%;" />

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858109.png" alt="image-20241107122810377"  />

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858110.png" alt="image-20241107122831700" style="zoom:50%;" />



**redis数据库RDB和AOF配置，数据库备份的区别：**

**一句话：AOF的优先级高于RDB**

- **情况1：整个redis默认情况下，redis为不设置save参数且未开启了AOF持久化时，当shutdown时会生成一个有效dump文件，整个过程只能在shutdown时保存一个有效的dump文件，以至于下一次打开redis时可恢复数据**
- **情况2：不设置save参数但是开启了AOF持久化时，以AOF为主，当shutdown时也会生成一个无效dump文件，备份数据库由AOF完成，若将AOF文件删除，数据库将无法完成备份**
- **情况3：但是若将save 后写成空串，则是禁用所有dump文件的自动生成方式，shutdown时连空文件也不会生成，若开启了AOF，则可由AOF完成备份，若未开启AOF，则是纯缓存模式，则redis无法自动完成备份**
- **情况4：设置了save参数以开启自动触发RDB，若未开启AOF，则由RDB独自完成备份，若开启了AOF且开启了混合模式，则由RDB和AOF混合完成备份，生成的AOF文件包括RDB头部和AOF混写，若未开启混合模式，则以AOF为主，AOF优先级高**



### 7.2、Redis 主从集群

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858111.png" alt="image-20241107123139802" style="zoom:50%;" />



#### 7.2.1、主从数据同步原理

##### 7.2.1.1、全量同步

主从第一次建立连接时，会执行**全量同步**，将master节点的所有数据都拷贝给slave节点，

这里有一个问题，**master如何得知salve是第一次来连接呢**？？

有几个概念，可以作为判断依据：

- **Replication Id**：简称replid，是数据集的标记，id一致则说明是同一数据集。每一个master都有唯一的replid，slave则会继承master节点的replid
- **offset**：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。如果slave的offset小于master的offset，说明slave数据落后于master，需要更新。

因此slave做数据同步，必须向master声明自己的replication id 和offset，master才可以判断到底需要同步哪些数据。

因为slave原本也是一个master，有自己的replid和offset，当第一次变成slave，与master建立连接时，发送的replid和offset是自己的replid和offset。

master判断发现**slave发送来的replid与自己的不一致**，说明这是一个全新的slave，就知道要做**全量同步**了。

master会将自己的replid和offset都发送给这个slave，slave保存这些信息。以后slave的replid就与master一致了。

因此，**master判断一个节点是否是第一次同步的依据，就是看replid是否一致**。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858112.png" alt="image-20241107123315252"  />

![image-20241107123814348](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858113.png)



**完整流程描述：**

- slave节点请求增量同步
- **master节点判断replid**，发现不一致，拒绝增量同步
- master将完整内存数据生成RDB，发送RDB到slave
- slave清空本地数据，加载master的RDB
- master将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给slave
- slave执行接收到的命令，保持与master之间的同步



##### 7.2.1.2、增量同步

全量同步需要先做RDB，然后将RDB文件通过网络传输个slave，成本太高了。因此除了第一次做全量同步，其它大多数时候slave与master都是做**增量同步**。

什么是增量同步？就是只更新slave与master存在差异的部分数据。如图：

[![image-20210725153201086](https://github.com/spongehah/redis/raw/main/%E7%AC%94%E8%AE%B0/image/Redis%E9%AB%98%E7%BA%A7-%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%93%E5%AD%98.assets/image-20210725153201086.png)](https://github.com/spongehah/redis/blob/main/笔记/image/Redis高级-分布式缓存.assets/image-20210725153201086.png)

那么master怎么知道slave与自己的数据差异在哪里呢?



##### 7.2.1.3、repl_backlog原理

master怎么知道slave与自己的数据差异在哪里呢?

这就要说到全量同步时的repl_baklog文件了。

这个文件是一个固定大小的数组，只不过数组是环形，也就是说**角标到达数组末尾后，会再次从0开始读写**，这样数组头部的数据就会被覆盖。

repl_baklog中会记录Redis处理过的命令日志及offset，包括master当前的offset，和slave已经拷贝到的offset：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858114.png" alt="image-20241107132533532" style="zoom: 67%;" />

slave与master的offset之间的差异，就是salve需要增量拷贝的数据了。

但是，如果slave出现网络阻塞，导致master的offset远远超过了slave的offset：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858115.png" alt="image-20241107132548907" style="zoom:67%;" />

**如果master继续写入新数据，其offset就会覆盖旧的数据**，直到将slave现在的offset也覆盖：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858116.png" alt="image-20241107132559163" style="zoom:67%;" />

棕色框中的红色部分，就是尚未同步，但是却已经被覆盖的数据。此时如果slave恢复，需要同步，却**发现自己的offset都没有了，无法完成增量同步了**。`只能做全量同步`。

![image-20241107132609048](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858117.png)



#### 7.2.2、主从同步优化

主从同步可以保证主从数据的一致性，非常重要。

因为**全量同步非常消耗性能**，repl_baklog写满时还会发生全量同步，为了**减少或优化全量同步**：

可以从以下几个方面来**优化Redis主从集群**：

1. 在master中配置**repl-diskless-sync yes**启用**无磁盘复制**，直接将生成的rdb发送给salve，避免全量同步时的磁盘IO，提高全量同步性能。（磁盘**IO能力较差，网络强**时）
2. 设置Redis单节点上的**内存占用不要太大**，减少RDB导致的过多磁盘IO
3. 适当**提高repl_backlog_buffer的大小**，参数为repl-backlog-size（默认为1MB）发现slave宕机时尽快实现故障恢复，尽可能避免全量同步
4. 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用**主-从-从链式结构，减少master压力**

主从从架构图：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858118.png" alt="image-20241107132720736" style="zoom:50%;" />



#### 7.2.3、小结

简述全量同步和增量同步区别？

- 全量同步：master将完整内存数据生成**RDB**，发送RDB到slave。后续命令则记录在repl_baklog，逐个发送给slave。
- 增量同步：slave提交自己的offset到master，master获取repl_backlog中从offset之后的命令给slave

什么时候执行全量同步？

- slave节点第一次连接master节点时
- slave节点断开时间太久，repl_baklog中的offset已经被覆盖时

什么时候执行增量同步？

- slave节点断开又恢复，并且在repl_baklog中能找到offset时



### 7.3、Redis 哨兵机制

#### 7.3.1、哨兵原理

##### 7.3.1.1、集群结构和作用

哨兵的结构如图：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858119.png" alt="image-20241107145359965" style="zoom:50%;" />

哨兵的作用如下：

- **监控**：Sentinel 会不断检查您的master和slave是否按预期工作
- **自动故障恢复**：如果master故障，Sentinel会将一个slave提升为master。当故障实例恢复后也以新的master为主
- **通知**：Sentinel充当Redis客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端



##### 7.3.1.2、集群监控原理

Sentinel基于心跳机制监测服务状态，**每隔1秒向集群的每个实例发送ping命令**：

•主观下线：如果某sentinel节点发现某实例未在规定时间响应，则认为该实例**主观下线**。

•客观下线：若**超过指定数量（quorum）的sentinel**都认为该实例主观下线，则该实例**客观下线**。quorum值最好超过**Sentinel实例数量的一半**。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858120.png" alt="image-20241107145447848" style="zoom:50%;" />



##### 7.3.1.3、集群故障恢复原理

一旦发现master故障，会在sentinel集群中**选出一个sentinel-leader**，采用**Raft算法**

- Raft算法的基本思路是**先到先得**
- 比如sentinel 1首先发现master宕机，那么他就会向sentinel 2和3发送请求成为leader，若sentinel 2和3没有接受/同意过别人的请求，就会同意sentinel 1的请求

一旦发现master故障，sentinel-leader需要在salve中**选择**一个作为**新的master**，选择依据是这样的：

- 首先会判断slave节点与master节点断开时间长短，如果超过指定值（down-after-milliseconds * 10）则会排除该slave节点
- 然后判断slave节点的**slave-priority**值，越小优先级越高，如果是0则永不参与选举
- 如果slave-prority一样，则判断slave节点的**offset值，越大说明数据越新，优先级越高**
- 最后是判断slave节点的**运行id大小**，越小优先级越高。

当选出一个新的master后，该**如何实现切换**呢？

流程如下：

- sentinel给备选的slave1节点发送**slaveof no one**命令，让该节点成为master
- sentinel给所有其它slave发送slaveof 192.168.111.100 7002 命令，让这些slave成为新master的从节点，开始从新的master上同步数据。
- 最后，sentinel**将故障节点标记为slave**，当故障节点恢复后会自动成为新的master的slave节点

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858121.png" alt="image-20241107145549246" style="zoom:50%;" />



##### 7.3.1.4、Sentinel 可以防止脑裂吗？

> 如果 M1 和 R2、R3 之间的⽹络被隔离，也就是发⽣了脑裂，M1 和 R2 、 R3 隔离在了两个不同的⽹络分区中。这意味着，R2 或者 R3 其中⼀个会被选为 master，这⾥假设为 R2。
>
> 但是！这样会出现问题了！！
>
> 如果客户端 C1 是和 M1 在⼀个⽹络分区的话，从⽹络被隔离到⽹络分区恢复这段时间，C1 写⼊ M1 的数据都会丢失，并且，C1 读取的可能也是过时的数据。这是因为当⽹络分区恢复之后，M1 将会成为 slave 节点。

想要解决这个问题的话也不难，对 Redis 主从复制进⾏配置即可。

下⾯对这两个配置进⾏解释：

- **min-replicas-to-write** **1**：⽤于配置写 master ⾄少写⼊的 slave 数量，设置为 0 表示关闭该功能。3 个节点的情况下，可以配置为 1 ，表示 master 必须写⼊⾄少 1 个 slave ，否则就停⽌接受新的写⼊命令请求。
- **min-replicas-max-lag** **10** ：⽤于配置 master 多⻓时间（秒）⽆法得到从节点的响应，就认为这个节点失联。我们这⾥配置的是 10 秒，也就是说master 10 秒都得不到⼀个从节点的响应，就会认为这个从节点失联，停⽌接受新的写⼊命令请求。

不过，这样配置会降低 Redis 服务的整体可⽤性，如果 2 个 slave 都挂掉，master 将会停⽌接受新的写⼊命令请求



##### 7.3.1.5、小结

Sentinel的三个作用是什么？

- 监控
- 故障转移
- 通知

Sentinel如何判断一个redis实例是否健康？

- 每隔1秒发送一次ping命令，如果超过一定时间没有相向则认为是主观下线
- 如果大多数sentinel都认为实例主观下线，则判定服务下线

故障转移步骤有哪些？

- 首先选定一个slave作为新的master，执行slaveof no one
- 然后让所有节点都执行slaveof 新master
- 修改故障节点配置，添加slaveof 新master

#### 7.3.2、RedisTemplate

在Sentinel集群监管下的Redis主从集群，其节点会因为自动故障转移而发生变化，Redis的客户端必须感知这种变化，及时更新连接信息。Spring的RedisTemplate底层利用lettuce实现了节点的感知和自动切换。

下面，我们通过一个测试来实现RedisTemplate集成哨兵机制。

##### 7.3.2.1、引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

##### 7.3.2.2、配置Redis地址

然后在配置文件application.yml中指定redis的sentinel相关信息：

```yaml
spring:
  redis:
    sentinel:
      master: mymaster
      nodes:
        - 192.168.111.100:27001
        - 192.168.111.100:27002
        - 192.168.111.100:27003
```

##### 7.3.2.3、配置读写分离

在项目的启动类中，添加一个新的bean：

```java
@Bean
public LettuceClientConfigurationBuilderCustomizer clientConfigurationBuilderCustomizer(){
    return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
}
```

这个bean中配置的就是读写策略，包括四种：

- MASTER：从主节点读取
- MASTER_PREFERRED：优先从master节点读取，master不可用才读取replica
- REPLICA：从slave（replica）节点读取
- REPLICA_PREFERRED：优先从slave（replica）节点读取，所有的slave都不可用才读取master



### 7.4、Redis 分片集群

主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决：

- 海量数据存储问题
- 高并发写的问题

使用分片集群可以解决上述问题，如图:

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858122.png" alt="image-20241107152652103" style="zoom:50%;" />

分片集群特征：

- 集群中有多个master，每个master保存不同数据
- 每个master都可以有多个slave节点
- master之间通过ping监测彼此健康状态
- 客户端请求可以访问集群任意节点，最终都会被转发到正确节点



#### 7.4.1、散列插槽 slot

##### 7.4.1.1、插槽原理

Redis会把每一个master节点映射到0~16383共16384个插槽（hash slot）上，查看集群信息时就能看到：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858123.png" alt="image-20241107153042187" style="zoom: 80%;" />

数据key不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况：

- key中包含"{}"，且“{}”中至少包含1个字符，“{}”中的部分是有效部分
- key中不包含“{}”，整个key都是有效部分

**为什么是16384（2^14^）个槽**，CRC16算法产生的hash值有16bit，为什么不使用2^16^=65536个槽位呢？



> **（1）65536个slot槽ping心跳包的消息头太大**

![image-20241107184642647](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858124.png)



> **（2）Redis作者推荐集群1000以内节点，16384个槽够用了，因为集群节点数太多ping心跳包也会很大，消耗带宽，容易造成网络拥堵**

![image-20241107184648535](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858125.png)



> **（3）在节点少的情况下，即小型集群中，因为填充率为slots/N，若采用65536的话，填充率将会很高，压缩比将会很低，不容易传输，但是采用16384的话，填充率低一些，压缩比将会高很多，容易传输些**

![image-20241107184703492](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858126.png)



##### 7.4.1.2、槽位映射算法：一致性哈希算法

> 修正：Redis Cluster使用的是哈希槽算法，在集群数量发生改变时，手动进行哈希槽的分配 理由:
>
> 1. 哈希槽算法更简单
> 2. 哈希槽算法可以根据机器的配置手动分配，一致性哈希不好手动分配
>
> 但是一致性哈希也有优点：比如下面所说的**缓存抖动**现象，即将数据添加到后一台机器上，而使用哈希槽会导致那部分槽位不可用

要将数据缓存在集群的哪一台节点上，采用哈希算法，这里有两种哈希算法：

1. 哈希取余算法：计算公式：hash(key) % N ，N为集群中节点的数量

   - 这种哈希算法虽然简单，但是会有一个缺点：当集群数量发生改变的时候，那么计算公式的分母发生改变，之前存储的所有slot中的数据，都要重新进行排列

2. 一致性哈希算法：计算公式：hash(key) % 2^32^

   - 目的：当服务器个数发生变动时尽量减少影响客户端到服务器的映射关系
   - 实现：采用一致性哈希环，即令 2^32^ = 0 ，让哈希表首位相连成一个环，集群中每个节点落在环上的位置是固定的

   <img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858127.png" alt="image-20241107184959764" style="zoom: 67%;" />

   - 落键规则：hash(key) 并对2^32^ 取模过后，将会落在一致性哈希环的某个位置，然后顺时针寻找第一个Redis节点，那么此key就属于该节点存储
   - 优点：若节点数量发生变化，影响的映射关系也只有出现变化的那个节点逆时针到第一个节点之间的映射关系 例如：新增了一个节点，那么该节点逆时针到上一个节点之间的数据归新节点所有；删除了一个节点，那么该节点逆时针到上一个节点之间的数据归顺时针下一个节点所有
   - 缺点：当节点太少时，容易发生数据倾斜问题

   <img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858128.png" alt="image-20241107185014119" style="zoom: 67%;" />

**解决方法——虚拟节点**

解决哈希环偏斜问题的方法就是，让集群中的节点尽可能的多，从而让各个节点均匀的分布在哈希空间中。在现实情境下，机器的数量一般都是固定的，所以我们只能将现有的物理节通过虚拟的方法复制多个出来，这些由实际节点虚拟复制而来的节点被称为**虚拟节点**。加入虚拟节点后的情况如下图所示：



##### 7.4.1.3、客户端操作

**连接客户端集群模式：**

```
redis-cli -p 7001 -c
```

- -c：集群模式，若不加-c，根据key映射的slot不在此节点时将会报错

例如：key是num，那么就根据num计算，如果是{itcast}num，则根据itcast计算。计算方式是利用CRC16算法得到一个hash值，然后对16384取余，得到的结果就是slot值。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858129.png" alt="image-20241107185816083" style="zoom: 80%;" />

如图，在7001这个节点执行set a 1时，对a做hash运算，对16384取余，得到的结果是15495，因此要存储到103节点。

到了7003后，执行`get num`时，对num做hash运算，对16384取余，得到的结果是2765，因此需要切换到7001节点

如何将同一类数据固定的**保存在同一个Redis实例**？

- 这一类数据使用相同的有效部分，例如key都以**{typeId}为前缀**

**查看集群节点信息：**

```
127.0.0.1:7001> cluster nodes
```

```
redis-cli -a 123456 -p 7001 cluster nodes
```

**查看某个key对应的slot：**

```
cluster keyslot key
```

**查看slot是否被使用：**

```
cluster countkeysinslot 槽位数字编号
```

- 返回0：没被使用
- 返回1：已被使用



##### 7.4.1.4、小结

Redis如何判断某个key应该在哪个实例？

- 将16384个插槽分配到不同的实例
- 根据key的有效部分计算哈希值，对16384取余
- 余数作为插槽，寻找插槽所在实例即可

如何将同一类数据固定的保存在同一个Redis实例？

- 这一类数据使用相同的有效部分，例如key都以{typeId}为前缀





#### 7.4.2、集群伸缩

redis-cli --cluster提供了很多操作集群的命令，可以通过下面方式查看：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858130.png" alt="image-20241107190640227" style="zoom: 67%;" />

比如，添加节点的命令：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858131.png" alt="image-20241107190658031" style="zoom:67%;" />



##### 7.4.2.1、需求分析

需求：向集群中添加一个新的master节点，并向其中存储 num = 10

- 启动一个新的redis实例，端口为7004
- 添加7004到之前的集群，并作为一个master节点
- 给7004节点分配插槽，使得num这个key可以存储到7004实例

这里需要两个新的功能：

- 添加一个节点到集群中
- 将部分插槽分配到新插槽



##### 7.4.2.2、创建新的redis实例

创建一个文件夹：

```
mkdir 7004
```

拷贝配置文件：

```
cp redis.conf /7004
```

修改配置文件：

```
sed /s/6379/7004/g 7004/redis.conf
```

启动

```
redis-server 7004/redis.conf
```



##### 7.4.2.3、添加新节点到redis

我的： **集群扩容全部命令总结：**

```
redis-server /myredis/cluster/redisCluster6387.conf		#启动6387server
redis-server /myredis/cluster/redisCluster6388.conf		#启动6388server
redis-cli -a 111111 --cluster add-node 192.168.111.100:6387 192.168.111.100:6381	#将6387作为新的master加入集群，6381为引荐人
redis-cli -a 111111 --cluster check 192.168.111.100:6381	#第一次检查
redis-cli -a 111111 --cluster reshard 192.168.111.100:6381		#给6387分配slot
redis-cli -a 111111 --cluster check 192.168.111.100:6381		#第二次检查
redis-cli -a 111111 --cluster add-node 192.168.111.100:6388 192.168.111.100:6387 --cluster-slave --cluster-master-id xxxxxxxxxxxxxxxxxxxxx(6387id)		#让6388成为6387的从节点
redis-cli -a 111111 --cluster check 192.168.111.100:6381		#第三c
```



添加节点的语法如下：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858132.png" alt="image-20241107190841720" style="zoom:67%;" />

执行命令：

```
redis-cli --cluster add-node 192.168.111.100:7004 192.168.111.100:7001
```

通过命令查看集群状态：

```
redis-cli -p 7001 cluster nodes
```

我的：也可以使用check命令：

```
redis-cli -a 111111 --cluster check 192.168.111.100:6381
```

如图，7004加入了集群，并且默认是一个master节点：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858133.png" alt="image-20241107190901951" style="zoom:67%;" />

但是，可以看到7004节点的插槽数量为0，因此没有任何数据可以存储到7004上



##### 7.4.2.4、转移插槽

我们要将num存储到7004节点，因此需要先看看num的插槽是多少：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858134.png" alt="image-20241107190918226" style="zoom: 50%;" />

如上图所示，num的插槽为2765.

我们可以将0~3000的插槽从7001转移到7004，命令格式如下：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858135.png" alt="image-20241107190931716" style="zoom: 50%;" />

具体命令如下：

建立连接：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858136.png" alt="image-20241107190956884" style="zoom: 67%;" />

得到下面的反馈：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858137.png" alt="image-20241107191012508" style="zoom: 50%;" />

询问要移动多少个插槽，我们计划是3000个：

新的问题来了：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858138.png" alt="image-20241107191038141" style="zoom: 50%;" />

那个node来接收这些插槽？？

显然是7004，那么7004节点的id是多少呢？

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858139.png" alt="image-20241107191505859" style="zoom:50%;" />

复制这个id，然后拷贝到刚才的控制台后：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858140.png" alt="image-20241107191518431" style="zoom:50%;" />

这里询问，你的插槽是从哪里移动过来的？

- all：代表全部，也就是三个节点各转移一部分
- 具体的id：目标节点的id
- done：没有了

这里我们要从7001获取，因此填写7001的id：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858141.png" alt="image-20241107191536757" style="zoom:50%;" />

填完后，点击done，这样插槽转移就准备好了：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858142.png" alt="image-20241107191546962" style="zoom:50%;" />

确认要转移吗？输入yes：

然后，通过命令查看结果：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858144.png" alt="image-20241107191604160" style="zoom:67%;" />

可以看到：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858145.png" alt="image-20241107191613271" style="zoom: 67%;" />

目的达成。



##### 7.4.2.5、集群缩容

**删除之前先把cluster分配的slot给重新分配一下  (很重要)**

我的： **集群缩容全部命令总结：**

```
redis-cli -a 111111 --cluster check 192.168.111.100:6388	#获得6388的ID
redis-cli -a 111111 --cluster del-node 192.168.111.100:6388 xxxxxxxx(6388id)  #删除6388
redis-cli -a 111111 --cluster reshard 192.168.111.100:6381    #把6387的slot都给6381
redis-cli -a 111111 --cluster check 192.168.111.100:6381    #第二次检查
redis-cli -a 111111 --cluster del-node 192.168.111.100:6387 xxxxxxxx(6387id)  #删除6387
redis-cli -a 111111 --cluster check 192.168.111.100:6381	#第三次检查
```





#### 7.4.3、故障转移

集群初识状态是这样的：

![image-20241107193212376](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858146.png)

其中7001、7002、7003都是master，我们计划让7002宕机。



##### 7.4.3.1、自动故障转移

当集群中有一个master宕机会发生什么呢？

直接停止一个redis实例，例如7002：

```
redis-cli -p 7002 shutdown
```



1）首先是该实例与其它实例失去连接

2）然后是疑似宕机：

![image-20241107193230083](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858147.png)

3）最后是确定下线，自动提升一个slave为新的master：

![image-20241107193238919](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858148.png)

4）当7002再次启动，就会变为一个slave节点了：

![image-20241107193249880](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858149.png)



##### 7.4.3.2、手动故障转移

利用cluster failover命令可以手动让集群中的某个master宕机，切换到执行cluster failover命令的这个slave节点，实现无感知的数据迁移。其流程如下：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858150.png" alt="image-20241107193303987" style="zoom: 67%;" />

这种failover命令可以指定三种模式：

- 缺省：默认的流程，如图1~6歩
- force：省略了对offset的一致性校验
- takeover：直接执行第5歩，忽略数据一致性、忽略master状态和其它master的意见（暴力）
- 

**案例需求**：在7002这个slave节点执行手动故障转移，重新夺回master地位

步骤如下：

1）利用redis-cli连接7002这个节点

2）执行**cluster failover**命令

如图：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858151.png" alt="image-20241107193433348" style="zoom: 67%;" />

效果：

![image-20241107193446978](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858152.png)



##### 7.4.3.4、Redis动态扩/缩容时，若有请求过来怎么办？

Redis动态扩/缩容时，slot槽位会发生变化，若是请求经过hash到达对应的节点后，发现slot槽已经迁移支别的节点上了，该怎么办？

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858153.png" alt="image-20241107194923338" style="zoom:50%;" />

- 1.当客户端向集群中某个节点发送命令，节点向客户端返回moved异常，告诉客户端数据对应目标槽的节点信息。
- 2.客户端再向目标节点发送命令，目标节点中的槽已经迁移出别的节点上了，此时目标节点会返回ask重定向给客户端。
- 2.客户端向新的target节点发送Asking命令，然后再次向新节点发送请求请求命令。
- 3.新节点target执行命令，把命令执行结果返回给客户端。

> moved和ask重定向的区别： 两者都是客户端重定向 moved异常：槽已经确定迁移，即槽已经不在当前节点 ask异常：槽还在迁移中

> 集群节点较多时，客服端大多数请求都会发生重定向，每次重定向都会产生一次无用的请求，严重影响了redis的性能。
>
> smart智能客户端知道由哪个节点负责管理哪个槽，而且某个当节点与槽的映射关系发生改变时，客户端也会进行响应的更新，这是一种非常高效的请求方式。例如Jedis就设置了JedisCluster类作为smart客户端，将映射关系保存在本地，当出现重定向时更新本地的映射关系，连续出现5次未找到目标节点，则抛出异常：Too many cluster redirection



##### 7.4.3.5、RedisCluster的故障选举

集群节点创建时，不管是 master还是slave，都置**currentEpoch为0**。 当前节点在接受其他节点发送的请求时，如果发送者的currentEpoch（消息头部会包含发送者的 currentEpoch）大于当前节点的currentEpoch，那么当前节点会更新currentEpoch。 因此，集群中所有节点的 currentEpoch最终会达成一致，相当于对集群状态的认知达成了一致。

**master节点选举过程：**

- 1.slave发现自己的master变为FAIL。
- 2.发起选举前，slave先给自己的epoch（即currentEpoch）加一，然后请求集群中其它master给自己投票，并广播信息给集群中其他节点。
- 3.slave发起投票后，会等待至少两倍NODE_TIMEOUT时长接收投票结果，不管NODE_TIMEOUT何值，也至少会等待2秒。
- 4.其他节点收到该信息，只有master响应，判断请求者的合法性，并发送结果。
- 5.尝试选举的slave收集master返回的结果，收到超过半投票数master的统一后变成新Master，如果失败会发起第二次选举，选举轮次标记+1继续上面的流程。
- 6.选举成功后，广播Pong消息通知集群其它节点。

之所以强制延迟至少0.5秒选举，是为确保master的fail状态在整个集群内传开，否则可能只有小部分master知晓，而master只会给处于fail状态的master的slaves投票。 如果一个slave的master状态不是fail，则其它master不会给它投票，Redis通过八卦协议（即Gossip协议，也叫谣言协议）传播fail。

而在固定延迟上再加一个随机延迟，是为了避免多个slaves同时发起选举。



#### 7.4.4、RedisTemplate访问分片集群

RedisTemplate底层同样基于lettuce实现了分片集群的支持，而使用的步骤与哨兵模式基本一致：

1）引入redis的starter依赖

2）配置分片集群地址

3）配置读写分离

与哨兵模式相比，1）和3）已在哨兵模式下配置过了，其中只有分片集群的配置方式略有差异，如下：

```yaml
spring:
  redis:
    cluster:
      nodes:
        - 192.168.111.100:7001
        - 192.168.111.100:7002
        - 192.168.111.100:7003
        - 192.168.111.100:8001
        - 192.168.111.100:8002
        - 192.168.111.100:8003
```



## 8、多级缓存

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858154.png" alt="image-20241108132522052"  />



<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858155.png" alt="image-20241108104059992" style="zoom:50%;" />

存在下面的问题：

•请求要经过Tomcat处理，**Tomcat的性能成为整个系统的瓶颈**（之前实战篇学的Redis缓存就是这种情况）

•Redis缓存失效时，会对数据库产生冲击

多级缓存就是充分利用请求处理的每个环节，分别添加缓存，减轻Tomcat压力，提升服务性能：

- 浏览器访问静态资源时，优先读取浏览器本地缓存
- 访问非静态资源（ajax查询数据）时，访问服务端
- 请求到达Nginx后，优先读取Nginx本地缓存
- 如果Nginx本地缓存未命中，则去直接查询Redis（不经过Tomcat）
- 如果Redis查询未命中，则查询Tomcat
- 请求进入Tomcat后，优先查询JVM进程缓存
- 如果JVM进程缓存未命中，则查询数据库



<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858156.png" alt="image-20241108104201275" style="zoom:50%;" />

可见，多级缓存的关键有两个：

- 一个是在nginx中编写业务，实现nginx本地缓存、Redis、Tomcat的查询
- 另一个就是在Tomcat中实现JVM进程缓存

其中Nginx编程则会用到OpenResty框架结合Lua这样的语言。



### 8.1、JVM进程缓存

#### 8.1.1、Caffeine

缓存在日常开发中启动至关重要的作用，由于是存储在内存中，数据的读取速度是非常快的，能大量减少对数据库的访问，减少数据库的压力。我们把缓存分为两类：

- 分布式缓存，例如Redis：
  - 优点：存储容量更大、可靠性更好、可以在集群间共享
  - 缺点：访问缓存有网络开销
  - 场景：缓存**数据量较大**、可靠性要求较高、需要在集群间共享
- 进程本地缓存，例如HashMap、GuavaCache：
  - 优点：读取本地内存，没有网络开销，速度更快
  - 缺点：存储容量有限、可靠性较低、无法共享
  - 场景：性能要求较高，缓存**数据量较小**



##### 8.1.1.1、Caffeine基本API

引入依赖：

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

缓存使用的基本API：

```java
@Test
void testBasicOps() {
    // 构建cache对象
    Cache<String, String> cache = Caffeine.newBuilder().initialCapacity(100).build();

    // 存数据
    cache.put("gf", "迪丽热巴");

    // 取数据
    String gf = cache.getIfPresent("gf");
    System.out.println("gf = " + gf);

    // 取数据，包含两个参数：
    // 参数一：缓存的key
    // 参数二：Lambda表达式，表达式参数就是缓存的key，方法体是查询数据库的逻辑
    // 优先根据key查询JVM缓存，如果未命中，则执行参数二的Lambda表达式
    String defaultGF = cache.get("defaultGF", key -> {
        // 根据key去数据库查询数据
        return "柳岩";
    });
    System.out.println("defaultGF = " + defaultGF);
}
```

- Caffeine.newBuilder().[参数].build(); 构建缓存对象，可以设置参数如初始大小，驱逐策略等
- cache.getIfPresent(key); 如果存在缓存就获取，否则返回null
- cache.get(key, function); 如果存在缓存就返回缓存，否则执行function操作（一般是查询数据库操作）**并存入缓存**

其它API：

- cache.put(key, object) 将object放入缓存
- cache.invalidate(key) 删除指定key的缓存



##### 8.1.1.2、Caffeine三种缓存驱逐策略

Caffeine既然是缓存的一种，肯定需要有缓存的清除策略，不然的话内存总会有耗尽的时候。

Caffeine提供了三种**缓存驱逐策略**：

- **基于容量**：设置缓存的数量上限

  ```
  // 创建缓存对象
  Cache<String, String> cache = Caffeine.newBuilder()
      .maximumSize(1) // 设置缓存大小上限为 1
      .build();
  ```

- **基于时间**：设置缓存的有效时间

  ```
  // 创建缓存对象
  Cache<String, String> cache = Caffeine.newBuilder()
      // 设置缓存有效期为 10 秒，从最后一次写入开始计时 
      .expireAfterWrite(Duration.ofSeconds(10)) 
      .build();
  ```

- **基于引用**：设置缓存为软引用或弱引用，利用GC来回收缓存数据。性能较差，不建议使用。

> **注意**：在默认情况下，当一个缓存元素过期的时候，Caffeine**不会自动立即**将其清理和驱逐。而是在一次读或写操作后，或者在空闲时间完成对失效数据的驱逐。
>
> ```
> /*
> 基于大小设置驱逐策略：
> */
> @Test
> void testEvictByNum() throws InterruptedException {
>  // 创建缓存对象
>  Cache<String, String> cache = Caffeine.newBuilder()
>          // 设置缓存大小上限为 1
>          .maximumSize(1)
>          .build();
>  // 存数据
>  cache.put("gf1", "柳岩");
>  cache.put("gf2", "范冰冰");
>  cache.put("gf3", "迪丽热巴");
>  // 延迟10ms，给清理线程一点时间
>  Thread.sleep(10L);
>  // 获取数据
>  System.out.println("gf1: " + cache.getIfPresent("gf1"));
>  System.out.println("gf2: " + cache.getIfPresent("gf2"));
>  System.out.println("gf3: " + cache.getIfPresent("gf3"));
> }
> ```
>
> 若不sleep，则三个gf都可以得到，要sleep给他一点时间清除缓存
>
> 
>
> ```
> /*
> 基于时间设置驱逐策略：
> */
> @Test
> void testEvictByTime() throws InterruptedException {
>  // 创建缓存对象
>  Cache<String, String> cache = Caffeine.newBuilder()
>          .expireAfterWrite(Duration.ofSeconds(1)) // 设置缓存有效期为 10 秒
>          .build();
>  // 存数据
>  cache.put("gf", "柳岩");
>  // 获取数据
>  System.out.println("gf: " + cache.getIfPresent("gf"));
>  // 休眠一会儿
>  Thread.sleep(1200L);
>  System.out.println("gf: " + cache.getIfPresent("gf"));
> }
> ```
>
> 和大小驱逐同理，要sleep一会



#### 8.1.2、实现JVM进程缓存

##### 8.1.2.1、需求

利用Caffeine实现下列需求：

- 给根据id查询商品的业务添加缓存，缓存未命中时查询数据库
- 给根据id查询商品库存的业务添加缓存，缓存未命中时查询数据库
- 缓存初始大小为100
- 缓存上限为10000

##### 8.1.2.2、实现

首先，我们需要定义两个Caffeine的缓存对象，分别保存商品、库存的缓存数据。

在item-service的`com.heima.item.config`包下定义`CaffeineConfig`类：

```java
@Configuration
public class CaffeineConfig {

    @Bean
    public Cache<Long, Item> itemCache(){
        return Caffeine.newBuilder()
                .initialCapacity(100)
                .maximumSize(10_000) // _只是分隔符，可以不要
                .build();
    }

    @Bean
    public Cache<Long, ItemStock> stockCache(){
        return Caffeine.newBuilder()
                .initialCapacity(100)
                .maximumSize(10_000)
                .build();
    }
}
```



然后，修改item-service中的`com.heima.item.web`包下的ItemController类，添加缓存逻辑：

```java
@RestController
@RequestMapping("item")
public class ItemController {
    
    @Autowired
    private IItemService itemService;
    @Autowired
    private IItemStockService stockService;
    @Autowired
    private Cache<Long, Item> itemCache;
    @Autowired
    private Cache<Long, ItemStock> stockCache;
    
    // ...其它略
    
    @GetMapping("/{id}")
    public Item findById(@PathVariable("id") Long id) {
        return itemCache.get(id, key -> itemService.query()
                .ne("status", 3).eq("id", key)
                .one()
        );
    }

    @GetMapping("/stock/{id}")
    public ItemStock findStockById(@PathVariable("id") Long id) {
        return stockCache.get(id, key -> stockService.getById(key));
    }
}
```



### 8.2、Lua语法入门

Nginx编程需要用到Lua语言，因此我们必须先入门Lua的基本语法。

#### 3.1.初识Lua

Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。官网：https://www.lua.org/

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858157.png" alt="image-20241108105410189" style="zoom:50%;" />

Lua经常嵌入到C语言开发的程序中，例如游戏开发、游戏插件等。

Nginx本身也是C语言开发，因此也允许基于Lua做拓展。

#### 3.1.HelloWorld

CentOS7默认已经安装了Lua语言环境，所以可以直接运行Lua代码。

1）在Linux虚拟机的任意目录下，新建一个hello.lua文件

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858158.png" alt="image-20241108105501198" style="zoom: 50%;" />

2）添加下面的内容

```
print("Hello World!")  
```



3）运行

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858159.png" alt="image-20241108105515147" style="zoom:50%;" />



#### 3.2.变量和循环

学习任何语言必然离不开变量，而变量的声明必须先知道数据的类型。

##### 3.2.1.Lua的数据类型

Lua中支持的常见数据类型包括：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858160.png" alt="image-20241108105835189" style="zoom:50%;" />

另外，Lua提供了type()函数来判断一个变量的数据类型：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858161.png" alt="image-20241108105847528" style="zoom:50%;" />

##### 3.2.2.声明变量

Lua声明变量的时候无需指定数据类型，而是用local来声明变量为局部变量：

```lua
-- 声明字符串，可以用单引号或双引号，
local str = 'hello'
-- 字符串拼接可以使用 ..
local str2 = 'hello' .. 'world'
-- 声明数字
local num = 21
-- 声明布尔类型
local flag = true
```

Lua中的table类型既可以作为数组，又可以作为Java中的map来使用。数组就是特殊的table，key是数组角标而已：

```lua
-- 声明数组 ，key为角标的 table
local arr = {'java', 'python', 'lua'}
-- 声明table，类似java的map
local map =  {name='Jack', age=21}
```

:exclamation: **Lua中的数组角标是从1开始**，访问的时候与Java中类似：

```lua
-- 访问数组，lua数组的角标从1开始
print(arr[1])
```

Lua中的table可以用key来访问：

```lua
-- 访问table
print(map['name'])
print(map.name)
```



##### 3.2.3.循环

对于table，我们可以利用for循环来遍历。不过数组和普通table遍历略有差异。

遍历数组：

```lua
-- 声明数组 key为索引的 table
local arr = {'java', 'python', 'lua'}
-- 遍历数组
for index,value in ipairs(arr) do
    print(index, value) 
end
```

遍历普通table

```lua
-- 声明map，也就是table
local map = {name='Jack', age=21}
-- 遍历table
for key,value in pairs(map) do
   print(key, value) 
end
```



#### 3.3.条件控制、函数

Lua中的条件控制和函数声明与Java类似。

##### 3.3.1.函数

定义函数的语法：

```lua
function 函数名( argument1, argument2..., argumentn)
    -- 函数体
    return 返回值
end
```

例如，定义一个函数，用来打印数组：

```lua
function printArr(arr)
    for index, value in ipairs(arr) do
        print(value)
    end
end
```



##### 3.3.2.条件控制

类似Java的条件控制，例如if、else语法：

```lua
if(布尔表达式)
then
   --[ 布尔表达式为 true 时执行该语句块 --]
else
   --[ 布尔表达式为 false 时执行该语句块 --]
end
```

与java不同，布尔表达式中的**逻辑运算是基于英文单词**：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858162.png" alt="image-20241108110040896" style="zoom:50%;" />

##### 3.3.3.案例

需求：自定义一个函数，可以打印table，当参数为nil时，打印错误信息

```lua
function printArr(arr)
    if not arr then
        print('数组不能为空！')
    end
    for index, value in ipairs(arr) do
        print(value)
    end
end
```



### 8.3、实现多级缓存

#### 8.3.1、OpenResty

OpenResty® 是一个基于 Nginx的高性能 Web 平台，用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。具备下列特点：

- 具备Nginx的完整功能
- 基于Lua语言进行扩展，集成了大量精良的 Lua 库、第三方模块
- 允许使用Lua**自定义业务逻辑**、**自定义库**

官方网站： https://openresty.org/cn/



我们希望达到的多级缓存架构如图：

![image-20241108111048555](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858163.png)

其中：

- windows上的nginx用来做反向代理服务，将前端的查询商品的ajax请求代理到OpenResty集群
- OpenResty集群用来编写多级缓存业务



##### 4.2.1.反向代理流程

现在，商品详情页使用的是假的商品数据。不过在浏览器中，可以看到页面有发起ajax请求查询真实商品数据。

这个请求如下：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858164.png" alt="image-20241108111159618" style="zoom:67%;" />

请求地址是localhost，端口是80，就被windows上安装的Nginx服务给接收到了。然后代理给了OpenResty集群：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858165.png" alt="image-20241108111211904" style="zoom:50%;" />

我们需要在OpenResty中编写业务，查询商品数据并返回到浏览器。

但是这次，我们先在OpenResty接收请求，返回假的商品数据。



##### 4.2.2.OpenResty监听请求

OpenResty的很多功能都依赖于其目录下的Lua库，需要在nginx.conf中指定依赖库的目录，并导入依赖：

1）添加对OpenResty的Lua模块的加载

修改`/usr/local/openresty/nginx/conf/nginx.conf`文件，在其中的http下面，添加下面代码：

```nginx
#lua 模块
lua_package_path "/usr/local/openresty/lualib/?.lua;;";
#c模块     
lua_package_cpath "/usr/local/openresty/lualib/?.so;;";  
```



2）监听/api/item路径

修改`/usr/local/openresty/nginx/conf/nginx.conf`文件，在nginx.conf的server下面，添加对/api/item这个路径的监听：

```nginx
location  /api/item {
    # 默认的响应类型
    default_type application/json;
    # 响应结果由lua/item.lua文件来决定
    content_by_lua_file lua/item.lua;
}
```



这个监听，就类似于SpringMVC中的`@GetMapping("/api/item")`做路径映射。

而`content_by_lua_file lua/item.lua`则相当于调用item.lua这个文件，执行其中的业务，把结果返回给用户。相当于java中调用service。



##### 4.2.3.编写item.lua

1）在`/usr/loca/openresty/nginx`目录创建文件夹：lua

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858166.png" alt="image-20241108111452618" style="zoom:50%;" />

2）在`/usr/loca/openresty/nginx/lua`文件夹下，新建文件：item.lua

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858167.png" alt="image-20241108111500950" style="zoom: 67%;" />

3）编写item.lua，返回假数据

item.lua中，利用ngx.say()函数返回数据到Response中

```nginx
ngx.say('{"id":10001,"name":"SALSA AIR","title":"RIMOWA 21寸托运箱拉杆箱 SALSA AIR系列果绿色 820.70.36.4","price":17900,"image":"https://m.360buyimg.com/mobilecms/s720x720_jfs/t6934/364/1195375010/84676/e9f2c55f/597ece38N0ddcbc77.jpg!q70.jpg.webp","category":"拉杆箱","brand":"RIMOWA","spec":"","status":1,"createTime":"2019-04-30T16:00:00.000+00:00","updateTime":"2019-04-30T16:00:00.000+00:00","stock":2999,"sold":31290}')
```



4）重新加载配置

```nginx
nginx -s reload
```

刷新商品页面：http://localhost/item.html?id=1001，即可看到效果



#### 8.3.2、请求参数处理

上一节中，我们在OpenResty接收前端请求，但是返回的是假数据。

要返回真实数据，必须根据前端传递来的商品id，查询商品信息才可以。

那么如何获取前端传递的商品参数呢？



##### 8.3.2.1、获取参数的API

OpenResty中提供了一些API用来获取不同类型的前端请求参数：

![image-20241108113034393](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858168.png)



##### 8.3.2.2、获取参数并返回

在前端发起的ajax请求如图：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858169.png" alt="image-20241108113046262" style="zoom:50%;" />

可以看到商品id是以路径占位符方式传递的，因此可以利用正则表达式匹配的方式来获取ID



1）获取商品id

修改`/usr/loca/openresty/nginx/nginx.conf`文件中监听/api/item的代码，利用正则表达式获取ID：

```nginx
location ~ /api/item/(\d+) {
    # 默认的响应类型
    default_type application/json;
    # 响应结果由lua/item.lua文件来决定
    content_by_lua_file lua/item.lua;
}
```

2）拼接ID并返回

修改`/usr/loca/openresty/nginx/lua/item.lua`文件，获取id并拼接到结果中返回：

```nginx
-- 获取商品id
local id = ngx.var[1]
-- 拼接并返回
ngx.say('{"id":' .. id .. ',"name":"SALSA AIR","title":"RIMOWA 21寸托运箱拉杆箱 SALSA AIR系列果绿色 820.70.36.4","price":17900,"image":"https://m.360buyimg.com/mobilecms/s720x720_jfs/t6934/364/1195375010/84676/e9f2c55f/597ece38N0ddcbc77.jpg!q70.jpg.webp","category":"拉杆箱","brand":"RIMOWA","spec":"","status":1,"createTime":"2019-04-30T16:00:00.000+00:00","updateTime":"2019-04-30T16:00:00.000+00:00","stock":2999,"sold":31290}')
```

3）重新加载并测试

运行命令以重新加载OpenResty配置：

```nginx
nginx -s reload
```



刷新页面可以看到结果中已经带上了ID



#### 8.3.3、查询Tomcat

拿到商品ID后，本应去缓存中查询商品信息，不过目前我们还未建立nginx、redis缓存。因此，这里我们先根据商品id去tomcat查询商品信息。我们实现如图部分：

![image-20241108114602045](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858170.png)

需要注意的是，我们的OpenResty是在虚拟机，Tomcat是在Windows电脑上。两者IP一定不要搞错了。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858171.png" alt="image-20241108114626648" style="zoom:50%;" />



##### 8.3.3.1、发送http请求的API

nginx提供了内部API用以发送http请求：

```nginx
local resp = ngx.location.capture("/path",{
    method = ngx.HTTP_GET,   -- 请求方式
    args = {a=1,b=2},  -- get方式传参数
})
```

返回的响应内容包括：

- resp.status：响应状态码
- resp.header：响应头，是一个table
- resp.body：响应体，就是响应数据

注意：这里的path是路径，并不包含IP和端口。这个请求会被nginx内部的server监听并处理。

但是我们希望这个请求发送到Tomcat服务器，所以还需要编写一个server来对这个路径做反向代理：

```nginx
 location /path {
     # 这里是windows电脑的ip和Java服务端口，需要确保windows防火墙处于关闭状态
     proxy_pass http://192.168.111.1:8081; 
 }
```

原理如图：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858172.png" alt="image-20241108114811150" style="zoom:50%;" />



##### 8.3.3.2、封装http工具

下面，我们封装一个发送Http请求的工具，基于ngx.location.capture来实现查询tomcat。



**1）添加反向代理，到windows的Java服务**

因为item-service中的接口都是/item开头，所以我们监听/item路径，代理到windows上的tomcat服务。

修改 `/usr/local/openresty/nginx/conf/nginx.conf`文件，添加一个location：

```nginx
location /item {
    proxy_pass http://192.168.111.1:8081;
}
```

以后，只要我们调用`ngx.location.capture("/item")`，就一定能发送请求到windows的tomcat服务。



**2）封装工具类**

之前我们说过，OpenResty启动时会加载以下两个目录中的工具文件：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858173.png" alt="image-20241108115648285" style="zoom:50%;" />

所以，自定义的http工具也需要放到这个目录下。

在`/usr/local/openresty/lualib`目录下，新建一个common.lua文件：

```nginx
vi /usr/local/openresty/lualib/common.lua
```

内容如下:

```lua
-- 封装函数，发送http请求，并解析响应
local function read_http(path, params)
    local resp = ngx.location.capture(path,{
        method = ngx.HTTP_GET,
        args = params,
    })
    if not resp then
        -- 记录错误信息，返回404
        ngx.log(ngx.ERR, "http请求查询失败, path: ", path , ", args: ", args)
        ngx.exit(404)
    end
    return resp.body
end
-- 将方法导出
local _M = {  
    read_http = read_http
}  
return _M
```

这个工具将read_http函数封装到_M这个table类型的变量中，并且返回，这类似于导出。

使用的时候，可以利用`require('common')`来导入该函数库，这里的common是函数库的文件名。



**3）实现商品查询**

最后，我们修改`/usr/local/openresty/lua/item.lua`文件，利用刚刚封装的函数库实现对tomcat的查询：

```lua
-- 引入自定义common工具模块，返回值是common中返回的 _M
local common = require("common")
-- 从 common中获取read_http这个函数
local read_http = common.read_http
-- 获取路径参数
local id = ngx.var[1]
-- 根据id查询商品
local itemJSON = read_http("/item/".. id, nil)
-- 根据id查询商品库存
local itemStockJSON = read_http("/item/stock/".. id, nil)
ngx.say(itemJSON)
```

这里查询到的结果是json字符串，并且包含商品、库存两个json字符串，页面最终需要的是把两个json拼接为一个json：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858174.png" alt="image-20241108115704064" style="zoom: 50%;" />

这就需要我们先把JSON变为lua的table，完成数据整合后，再转为JSON。



##### 8.3.3.3、CJSON工具类

OpenResty提供了一个cjson的模块用来处理JSON的序列化和反序列化。

官方地址： https://github.com/openresty/lua-cjson/

安装OpenResty时已安装了一些lua库，CJSON就在里面

1）引入cjson模块：

```nginx
local cjson = require "cjson"
```



2）序列化：

```lua
local obj = {
    name = 'jack',
    age = 21
}
-- 把 table 序列化为 json
local json = cjson.encode(obj)
```



3）反序列化：

```lua
local json = '{"name": "jack", "age": 21}'
-- 反序列化 json为 table
local obj = cjson.decode(json);
print(obj.name)
```



##### 8.3.3.4、实现Tomcat查询

下面，我们修改之前的item.lua中的业务，添加json处理功能：

```lua
-- 导入common函数库
local common = require('common')
local read_http = common.read_http
-- 导入cjson库
local cjson = require('cjson')

-- 获取路径参数
local id = ngx.var[1]
-- 根据id查询商品
local itemJSON = read_http("/item/".. id, nil)
-- 根据id查询商品库存
local itemStockJSON = read_http("/item/stock/".. id, nil)

-- JSON转化为lua的table
local item = cjson.decode(itemJSON)
local stock = cjson.decode(itemStockJSON)

-- 组合数据
item.stock = stock.stock
item.sold = stock.sold

-- 把item序列化为json 返回结果
ngx.say(cjson.encode(item))
```



##### 8.3.3.5、基于ID负载均衡

刚才的代码中，我们的tomcat是单机部署。而实际开发中，tomcat一定是集群模式：

![image-20241108115744407](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858175.png)

因此，OpenResty需要对tomcat集群做负载均衡。

而默认的负载均衡规则是轮询模式，当我们查询/item/10001时：

- 第一次会访问8081端口的tomcat服务，在该服务内部就形成了JVM进程缓存
- 第二次会访问8082端口的tomcat服务，该服务内部没有JVM缓存（因为JVM缓存无法共享），会查询数据库
- ...

你看，因为轮询的原因，第一次查询8081形成的JVM缓存并未生效，直到下一次再次访问到8081时才可以生效，缓存命中率太低了。

怎么办？

如果能让同一个商品，每次查询时都访问同一个tomcat服务，那么JVM缓存就一定能生效了。

也就是说，我们需要根据商品id做负载均衡，而不是轮询。



**1）原理**

nginx提供了**基于请求路径做负载均衡**的算法：

nginx根据请求路径做hash运算，把得到的数值对tomcat服务的数量取余，余数是几，就访问第几个服务，实现负载均衡。

例如：

- 我们的请求路径是 /item/10001
- tomcat总数为2台（8081、8082）
- 对请求路径/item/1001做hash运算求余的结果为1
- 则访问第一个tomcat服务，也就是8081

**只要id不变，每次hash运算结果也不会变，那就可以保证同一个商品，一直访问同一个tomcat服务，确保JVM缓存生效。**



**2）实现**

修改`/usr/local/openresty/nginx/conf/nginx.conf`文件，实现基于ID做负载均衡。

首先，定义tomcat集群，并设置基于路径做负载均衡：

```nginx
upstream tomcat-cluster {
    hash $request_uri;
    server 192.168.111.1:8081;
    server 192.168.111.1:8082;
}
```



然后，修改对tomcat服务的反向代理，目标指向tomcat集群：

```nginx
location /item {
    proxy_pass http://tomcat-cluster;
}
```



重新加载OpenResty

```nginx
nginx -s reload
```



**3）测试**

启动两台tomcat服务：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858176.png" alt="image-20241108115803711" style="zoom: 50%;" />

同时启动：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858177.png" alt="image-20241108115819402" style="zoom:50%;" />

清空日志后，再次访问页面，可以看到不同id的商品，访问到了不同的tomcat服务：

![image-20241108115829041](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858178.png)

![image-20241108115838131](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858179.png)





#### 8.3.4、Redis缓存预热

Redis缓存会面临冷启动问题：

**冷启动**：服务刚刚启动时，Redis中并没有缓存，如果所有商品数据都在第一次查询时添加缓存，可能会给数据库带来较大压力。

**缓存预热**：在实际开发中，我们可以利用大数据统计用户访问的热点数据，在项目启动时将这些热点数据提前查询并保存到Redis中。

我们数据量较少，并且没有数据统计相关功能，目前可以在启动时将所有数据都放入缓存中。

1）利用Docker安装Redis

```shell
docker run --name redis -p 6379:6379 -d redis redis-server --appendonly yes
```

2）在item-service服务中引入Redis依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

3）配置Redis地址

```yaml
spring:
  redis:
    host: 192.168.111.100
```

4）编写初始化类

缓存预热需要在项目启动时完成，并且必须是拿到RedisTemplate之后。

这里我们利用InitializingBean接口来实现，因为InitializingBean可以在对象被Spring创建并且成员变量全部注入后执行。

```java
@Component
public class RedisHandler implements InitializingBean {

    @Autowired
    private StringRedisTemplate redisTemplate;

    @Autowired
    private IItemService itemService;
    @Autowired
    private IItemStockService stockService;

    private static final ObjectMapper MAPPER = new ObjectMapper();

    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化缓存
        // 1.查询商品信息
        List<Item> itemList = itemService.list();
        // 2.放入缓存
        for (Item item : itemList) {
            // 2.1.item序列化为JSON
            String json = MAPPER.writeValueAsString(item);
            // 2.2.存入redis
            redisTemplate.opsForValue().set("item:id:" + item.getId(), json);
        }

        // 3.查询商品库存信息
        List<ItemStock> stockList = stockService.list();
        // 4.放入缓存
        for (ItemStock stock : stockList) {
            // 2.1.item序列化为JSON
            String json = MAPPER.writeValueAsString(stock);
            // 2.2.存入redis
            redisTemplate.opsForValue().set("item:stock:id:" + stock.getId(), json);
        }
    }
}
```



#### 8.3.5、查询Redis缓存

现在，Redis缓存已经准备就绪，我们可以再OpenResty中实现查询Redis的逻辑了。如下图红框所示：

![image-20241108131157705](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858180.png)

当请求进入OpenResty之后：

- 优先查询Redis缓存
- 如果Redis缓存未命中，再查询Tomcat



##### 8.3.5.1、封装Redis工具

OpenResty提供了操作Redis的模块，我们只要引入该模块就能直接使用。但是为了方便，我们将Redis操作封装到之前的common.lua工具库中。

修改`/usr/local/openresty/lualib/common.lua`文件：

1）引入Redis模块，并初始化Redis对象

```lua
-- 导入redis
local redis = require('resty.redis')
-- 初始化redis
local red = redis:new()
red:set_timeouts(1000, 1000, 1000)
```

2）封装函数，用来释放Redis连接，其实是放入连接池

```lua
-- 关闭redis连接的工具方法，其实是放入连接池
local function close_redis(red)
    local pool_max_idle_time = 10000 -- 连接的空闲时间，单位是毫秒
    local pool_size = 100 --连接池大小
    local ok, err = red:set_keepalive(pool_max_idle_time, pool_size)
    if not ok then
        ngx.log(ngx.ERR, "放入redis连接池失败: ", err)
    end
end
```

3）封装函数，根据key查询Redis数据

```lua
-- 查询redis的方法 ip和port是redis地址，key是查询的key
local function read_redis(ip, port, key)
    -- 获取一个连接
    local ok, err = red:connect(ip, port)
    if not ok then
        ngx.log(ngx.ERR, "连接redis失败 : ", err)
        return nil
    end
    -- 查询redis
    local resp, err = red:get(key)
    -- 查询失败处理
    if not resp then
        ngx.log(ngx.ERR, "查询Redis失败: ", err, ", key = " , key)
    end
    --得到的数据为空处理
    if resp == ngx.null then
        resp = nil
        ngx.log(ngx.ERR, "查询Redis数据为空, key = ", key)
    end
    close_redis(red)
    return resp
end
```

4）导出

```lua
-- 将方法导出
local _M = {  
    read_http = read_http,
    read_redis = read_redis
}  
return _M
```



**附录：完整的common.lua**

```lua
-- 导入redis
local redis = require('resty.redis')
-- 初始化redis
local red = redis:new()
red:set_timeouts(1000, 1000, 1000)

-- 关闭redis连接的工具方法，其实是放入连接池
local function close_redis(red)
    local pool_max_idle_time = 10000 -- 连接的空闲时间，单位是毫秒
    local pool_size = 100 --连接池大小
    local ok, err = red:set_keepalive(pool_max_idle_time, pool_size)
    if not ok then
        ngx.log(ngx.ERR, "放入redis连接池失败: ", err)
    end
end

-- 查询redis的方法 ip和port是redis地址，key是查询的key
local function read_redis(ip, port, key)
    -- 获取一个连接
    local ok, err = red:connect(ip, port)
    if not ok then
        ngx.log(ngx.ERR, "连接redis失败 : ", err)
        return nil
    end
    -- 查询redis
    local resp, err = red:get(key)
    -- 查询失败处理
    if not resp then
        ngx.log(ngx.ERR, "查询Redis失败: ", err, ", key = " , key)
    end
    --得到的数据为空处理
    if resp == ngx.null then
        resp = nil
        ngx.log(ngx.ERR, "查询Redis数据为空, key = ", key)
    end
    close_redis(red)
    return resp
end

-- 封装函数，发送http请求，并解析响应
local function read_http(path, params)
    local resp = ngx.location.capture(path,{
        method = ngx.HTTP_GET,
        args = params,
    })
    if not resp then
        -- 记录错误信息，返回404
        ngx.log(ngx.ERR, "http查询失败, path: ", path , ", args: ", args)
        ngx.exit(404)
    end
    return resp.body
end
-- 将方法导出
local _M = {  
    read_http = read_http,
    read_redis = read_redis
}  
return _M
```



##### 8.3.5.2、实现Redis查询

接下来，我们就可以去修改item.lua文件，实现对Redis的查询了。

查询逻辑是：

- 根据id查询Redis
- 如果查询失败则继续查询Tomcat
- 将查询结果返回

1）修改`/usr/local/openresty/lua/item.lua`文件，添加一个查询函数：

```lua
-- 导入common函数库
local common = require('common')
local read_http = common.read_http
local read_redis = common.read_redis
-- 封装查询函数
function read_data(key, path, params)
    -- 查询本地缓存
    local val = read_redis("127.0.0.1", 6379, key)
    -- 判断查询结果
    if not val then
        ngx.log(ngx.ERR, "redis查询失败，尝试查询http， key: ", key)
        -- redis查询失败，去查询http
        val = read_http(path, params)
    end
    -- 返回数据
    return val
end
```



2）而后修改商品查询、库存查询的业务：

![image-20241108131216866](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858181.png)



**附录：完整的item.lua代码**

实现查询redis和tomcat的完整item.lua代码，若要查询nginx本地缓存，下面还需要加代码

```lua
-- 导入common函数库
local common = require('common')
local read_http = common.read_http
local read_redis = common.read_redis
-- 导入cjson库
local cjson = require('cjson')

-- 封装查询函数
function read_data(key, path, params)
    -- 查询Redis缓存
    local val = read_redis("127.0.0.1", 6379, key)
    -- 判断查询结果
    if not val then
        ngx.log(ngx.ERR, "redis查询失败，尝试查询http， key: ", key)
        -- redis查询失败，去查询http
        val = read_http(path, params)
    end
    -- 返回数据
    return val
end

-- 获取路径参数
local id = ngx.var[1]

-- 查询商品信息
local itemJSON = read_data("item:id:" .. id,  "/item/" .. id, nil)
-- 查询库存信息
local stockJSON = read_data("item:stock:id:" .. id, "/item/stock/" .. id, nil)

-- JSON转化为lua的table
local item = cjson.decode(itemJSON)
local stock = cjson.decode(stockJSON)
-- 组合数据
item.stock = stock.stock
item.sold = stock.sold

-- 把item序列化为json 返回结果
ngx.say(cjson.encode(item))
```



#### 8.3.6、Nginx本地缓存



现在，整个多级缓存中只差最后一环，也就是nginx的本地缓存了。如图：

![image-20241108131246157](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858182.png)

##### 8.3.6.1、本地缓存API

OpenResty为Nginx提供了**shard dict**的功能，可以在nginx的多个worker之间共享数据，实现缓存功能。

1）开启共享字典，在nginx.conf的http下添加配置：

```nginx
 # 共享字典，也就是本地缓存，名称叫做：item_cache，大小150m
 lua_shared_dict item_cache 150m; 
```

2）操作共享字典：

```lua
-- 获取本地缓存对象
local item_cache = ngx.shared.item_cache
-- 存储, 指定key、value、过期时间，单位s，默认为0代表永不过期
item_cache:set('key', 'value', 1000)
-- 读取
local val = item_cache:get('key')
```



##### 8.3.6.2、实现本地缓存查询

1）修改`/usr/local/openresty/lua/item.lua`文件，修改read_data查询函数，添加本地缓存逻辑：

```lua
-- 导入共享词典，本地缓存
local item_cache = ngx.shared.item_cache

-- 封装查询函数
function read_data(key, expire, path, params)
    -- 查询本地缓存
    local val = item_cache:get(key)
    if not val then
        ngx.log(ngx.ERR, "本地缓存查询失败，尝试查询Redis， key: ", key)
        -- 查询redis
        val = read_redis("127.0.0.1", 6379, key)
        -- 判断查询结果
        if not val then
            ngx.log(ngx.ERR, "redis查询失败，尝试查询http， key: ", key)
            -- redis查询失败，去查询http
            val = read_http(path, params)
        end
    end
    -- 查询成功，把数据写入本地缓存
    item_cache:set(key, val, expire)
    -- 返回数据
    return val
end
```

2）修改item.lua中查询商品和库存的业务，实现最新的read_data函数：

![image-20241108131406777](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858183.png)

其实就是多了缓存时间参数，过期后nginx缓存会自动删除，下次访问即可更新缓存。

这里给商品基本信息设置超时时间为30分钟，库存为1分钟。

因为库存更新频率较高，如果缓存时间过长，可能与数据库差异较大。

如果是秒杀商品，就不要加库存的缓存了



**附录：完整的item.lua文件**

```lua
-- 导入common函数库
local common = require('common')
local read_http = common.read_http
local read_redis = common.read_redis
-- 导入cjson库
local cjson = require('cjson')
-- 导入共享词典，本地缓存
local item_cache = ngx.shared.item_cache

-- 封装查询函数
function read_data(key, expire, path, params)
    -- 查询本地缓存
    local val = item_cache:get(key)
    if not val then
        ngx.log(ngx.ERR, "Nginx本地缓存查询失败，尝试查询Redis， key: ", key)
        -- 查询redis
        val = read_redis("127.0.0.1", 6379, key)
        -- 判断查询结果
        if not val then
            ngx.log(ngx.ERR, "Redis查询失败，尝试查询http， key: ", key)
            -- Redis查询失败，去查询http
            val = read_http(path, params)
        end
    end
    -- 查询成功，把数据写入本地缓存
    item_cache:set(key, val, expire)
    -- 返回数据
    return val
end

-- 获取路径参数
local id = ngx.var[1]

-- 查询商品信息
local itemJSON = read_data("item:id:" .. id, 1800,  "/item/" .. id, nil)
-- 查询库存信息
local stockJSON = read_data("item:stock:id:" .. id, 60, "/item/stock/" .. id, nil)

-- JSON转化为lua的table
local item = cjson.decode(itemJSON)
local stock = cjson.decode(stockJSON)
-- 组合数据
item.stock = stock.stock
item.sold = stock.sold

-- 把item序列化为json 返回结果
ngx.say(cjson.encode(item))
```





### 8.4、缓存同步

大多数情况下，浏览器查询到的都是缓存数据，如果缓存数据与数据库数据存在较大差异，可能会产生比较严重的后果。

所以我们必须保证数据库数据、缓存数据的一致性，这就是缓存与数据库的同步。

#### 8.4.1、数据同步策略

缓存数据同步的常见方式有三种：

**设置有效期**：给缓存设置有效期，到期后自动删除。再次查询时更新

- 优势：简单、方便
- 缺点：时效性差，缓存过期之前可能不一致
- 场景：更新频率较低，时效性要求低的业务

**同步双写**：在修改数据库的同时，直接修改缓存

- 优势：时效性强，缓存与数据库强一致
- 缺点：有代码侵入，耦合度高；
- 场景：对一致性、时效性要求较高的缓存数据

**异步通知：**修改数据库时发送事件通知，相关服务监听到通知后修改缓存数据

- 优势：低耦合，可以同时通知多个缓存服务
- 缺点：时效性一般，可能存在中间不一致状态
- 场景：时效性要求一般，有多个服务需要同步

而异步实现又可以基于MQ或者Canal来实现：

1）基于MQ的异步通知：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858184.png" alt="image-20241108132207070" style="zoom:50%;" />

解读：

- 商品服务完成对数据的修改后，只需要发送一条消息到MQ中。
- 缓存服务监听MQ消息，然后完成对缓存的更新

依然有少量的代码侵入。

2）基于Canal的通知

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858185.png" alt="image-20241108132220757" style="zoom:50%;" />

解读：

- 商品服务完成商品修改后，业务直接结束，没有任何代码侵入
- Canal监听MySQL变化，当发现变化后，立即通知缓存服务
- 缓存服务接收到canal通知，更新缓存

代码零侵入



#### 8.4.2、安装Canal

**Canal [kə'næl]**，译意为水道/管道/沟渠，canal是阿里巴巴旗下的一款开源项目，基于Java开发。基于数据库增量日志解析，提供增量数据订阅&消费。GitHub的地址：https://github.com/alibaba/canal

Canal是基于mysql的主从同步来实现的，MySQL主从同步的原理如下：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858186.png" alt="image-20241108132306716" style="zoom: 67%;" />

- 1）MySQL master 将数据变更写入二进制日志( binary log），其中记录的数据叫做binary log events
- 2）MySQL slave 将 master 的 binary log events拷贝到它的中继日志(relay log)
- 3）MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

而Canal就是把自己伪装成MySQL的一个slave节点，从而监听master的binary log变化。再把得到的变化信息通知给Canal的客户端，进而完成对其它数据库的同步。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858187.png" alt="image-20241108132320717" style="zoom:50%;" />



#### 8.4.3、监听Canal

Canal提供了各种语言的客户端，当Canal监听到binlog变化时，会通知Canal的客户端。

![image-20241108132424147](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858188.png)

我们可以利用Canal提供的Java客户端，监听Canal通知消息。当收到变化的消息时，完成对缓存的更新。

不过这里我们会使用GitHub上的第三方开源的**canal-starter****客户端**。地址：https://github.com/NormanGyllenhaal/canal-client

与SpringBoot完美整合，自动装配，比官方客户端要简单好用很多。

> 注意：canal只能监听MySQL，然后同步JVM本地缓存和Redis缓存，无法同步Nginx缓存，Nginx缓存目前只教了过期时间，所以Nginx缓存尽量只缓存静态数据等信息，或者把过期时间设置短一些



##### 8.4.3.1、引入依赖：

```xml
<dependency>
    <groupId>top.javatool</groupId>
    <artifactId>canal-spring-boot-starter</artifactId>
    <version>1.2.1-RELEASE</version>
</dependency>
```

##### 8.4.3.2、编写配置：

```yaml
canal:
  destination: heima # canal的集群名字，要与安装canal时设置的名称一致
  server: 192.168.111.100:11111 # canal服务地址
```

##### 8.4.3.3、修改Item实体类

![image-20241108132439375](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858189.png)

通过@Id、@Column、等注解完成Item与数据库表字段的映射：

```java
@Data
@TableName("tb_item")
public class Item {
    @TableId(type = IdType.AUTO)
    @Id
    private Long id;//商品id
    @Column(name = "name")
    private String name;//商品名称
    private String title;//商品标题
    private Long price;//价格（分）
    private String image;//商品图片
    private String category;//分类名称
    private String brand;//品牌名称
    private String spec;//规格
    private Integer status;//商品状态 1-正常，2-下架
    private Date createTime;//创建时间
    private Date updateTime;//更新时间
    @TableField(exist = false)
    @Transient
    private Integer stock;
    @TableField(exist = false)
    @Transient
    private Integer sold;
}
```



##### 8.4.3.4、编写监听器

通过实现`EntryHandler<T>`接口编写监听器，监听Canal消息。注意两点：

- 实现类通过`@CanalTable("tb_item")`指定监听的表信息
- EntryHandler的泛型是与表对应的实体类

```java
@CanalTable("tb_item")
@Component
public class ItemHandler implements EntryHandler<Item> {

    @Autowired
    private RedisHandler redisHandler;
    @Autowired
    private Cache<Long, Item> itemCache;

    @Override
    public void insert(Item item) {
        // 写数据到JVM进程缓存
        itemCache.put(item.getId(), item);
        // 写数据到redis
        redisHandler.saveItem(item);
    }

    @Override
    public void update(Item before, Item after) {
        // 写数据到JVM进程缓存
        itemCache.put(after.getId(), after);
        // 写数据到redis
        redisHandler.saveItem(after);
    }

    @Override
    public void delete(Item item) {
        // 删除数据到JVM进程缓存
        itemCache.invalidate(item.getId());
        // 删除数据到redis
        redisHandler.deleteItemById(item.getId());
    }
}
```



在这里对Redis的操作都封装到了RedisHandler这个对象中，是我们之前做缓存预热时编写的一个类，内容如下：

```java
@Component
public class RedisHandler implements InitializingBean {

    @Autowired
    private StringRedisTemplate redisTemplate;

    @Autowired
    private IItemService itemService;
    @Autowired
    private IItemStockService stockService;

    private static final ObjectMapper MAPPER = new ObjectMapper();

    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化缓存
        // 1.查询商品信息
        List<Item> itemList = itemService.list();
        // 2.放入缓存
        for (Item item : itemList) {
            // 2.1.item序列化为JSON
            String json = MAPPER.writeValueAsString(item);
            // 2.2.存入redis
            redisTemplate.opsForValue().set("item:id:" + item.getId(), json);
        }

        // 3.查询商品库存信息
        List<ItemStock> stockList = stockService.list();
        // 4.放入缓存
        for (ItemStock stock : stockList) {
            // 2.1.item序列化为JSON
            String json = MAPPER.writeValueAsString(stock);
            // 2.2.存入redis
            redisTemplate.opsForValue().set("item:stock:id:" + stock.getId(), json);
        }
    }

    public void saveItem(Item item) {
        try {
            String json = MAPPER.writeValueAsString(item);
            redisTemplate.opsForValue().set("item:id:" + item.getId(), json);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }

    public void deleteItemById(Long id) {
        redisTemplate.delete("item:id:" + id);
    }
}
```



### 8.5、多级缓存总结

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858154.png" alt="image-20241108132522052"  />

**多级缓存**和前面实战篇学的**Redis缓存**中Redis操作的区别：

1. 为什么要使用多级缓存？
   - 因为前面讲的Redis缓存为Nginx请求Tomcat后，先查询Redis缓存，若缓存未命中才查询MySQL数据库，但是这有一个问题：就是**Tomcat的并发度没有Redis高，所以Tomcat成为了Redis的瓶颈**，所以我们可以将Redis放在Tomcat的前面，由Nginx先查询Redis再查询Tomcat；
   - 另外：Tomcat也可以添加本地JVM缓存，若JVM缓存未命中才再去查询MySQL数据库，并将数据写入JVM缓存
2. 缓存同步策略的区别？
   - ①前面Redis缓存的同步策略是：采用**主动操作为主，过期策略为辅**；主动操作表现为数据发生变化则删除Redis中的数据，并且是先改后删，由客户端访问数据时发现Redis没有缓存才将数据缓存到Redis； ②前面的Redis缓存还需要解决 **缓存穿透**、**缓存雪崩**、**缓存击穿** 等问题；缓存穿透可以使用返回空值和使用布隆过滤器解决；缓存击穿可以使用互斥锁和逻辑过期解决。
   - 多级缓存的缓存同步策略可以是设置有效期、同步双写(即Redis缓存的策略)、异步通知三种方法，而异步通知又可以使用MQ或者Canal监听MySQL的变化，这里采用的方法是使用canal来监听MySQL，然后更改JVM缓存和Redis缓存； 注意：canal只能监听MySQL，然后同步JVM本地缓存和Redis缓存，无法同步Nginx缓存，Nginx缓存目前只教了过期时间，所以Nginx缓存尽量只缓存静态数据等信息，或者把过期时间设置短一些 至于缓存穿透、缓存雪崩、缓存击穿等问题没有讲是否需要实现，我认为是需要的





##  9、Redis高级篇之最佳实践

>* Redis键值设计
>* 批处理优化
>* 服务端优化
>* 集群最佳实践

### 9.1、Redis键值设计

#### 9.1.1、优雅的key结构

Redis的Key虽然可以自定义，但最好遵循下面的几个最佳实践约定：

- 遵循基本格式：[业务名称]:[数据名]:[id]
- 长度不超过44字节
- 不包含特殊字符

例如：我们的登录业务，保存用户信息，其key可以设计成如下格式：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858190.png" alt="image-20241108224809851" style="zoom:50%;" />

这样设计的好处：

- 可读性强
- 避免key冲突
- 方便管理
- 更节省内存： key是string类型，底层编码包含int、embstr和raw三种。
  embstr在小于44字节使用，采用连续内存空间，内存占用更小。当字节数大于44字节时，会转为raw模式存储，在raw模式下，内存空间不是连续的，而是采用一个指针指向了另外一段内存空间，在这段空间里存储SDS内容，这样空间不连续，访问的时候性能也就会收到影响，还有可能产生内存碎片
  （具体看《Redis原理篇》 String数据类类型）

> 这里的底层编码指的是key-value中二者，二者都是属于一个对象，key属于String类型

使用object encoding key 查看key的底层编码

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858191.png" alt="image-20241108224822347" style="zoom:50%;" />

#### 9.1.2、拒绝BigKey

BigKey通常以Key的大小和Key中成员的数量来综合判定，例如：

- Key本身的数据量过大：一个String类型的Key，它的值为5 MB
- Key中的成员数过多：一个ZSET类型的Key，它的成员数量为10,000个
- Key中成员的数据量过大：一个Hash类型的Key，它的成员数量虽然只有1,000个但这些成员的Value（值）总大小为100 MB

那么如何判断元素的大小呢？redis也给我们提供了命令

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858192.png" alt="image-20241108224846555" style="zoom:50%;" />

> 判断元素的大小：
>
> - memory usage key ：查看key使用的字节数（CPU消耗较高，不推荐使用）
> - STRLEN/LLEN key ： 查看集合类型key的长度

**推荐值：**

- 单个key的value小于10KB
- 对于集合类型的key，建议元素数量小于1000

##### 9.1.2.1、BigKey的危害

- 网络阻塞
  - 对BigKey执行读请求时，少量的QPS就可能导致带宽使用率被占满，导致Redis实例，乃至所在物理机变慢
- 数据倾斜
  - BigKey所在的Redis实例内存使用率远超其他实例，无法使数据分片的内存资源达到均衡
- Redis主线程阻塞
  - 对元素较多的hash、list、zset等做运算会耗时较旧，使主线程被阻塞
  - 当进行RDB的bgsave和AOF的bgrewrite时，会fork子进程，fork子进程会复制父进程的页表等数据结构，页表太大会造成主线程阻塞
- CPU压力
  - 对BigKey的数据序列化和反序列化会导致CPU的使用率飙升，影响Redis实例和本机其它应用

##### 9.1.2.2、如何发现BigKey

**①redis-cli --bigkeys**

利用redis-cli提供的--bigkeys参数，可以遍历分析所有key，并返回Key的整体统计信息与每个数据的Top1的big key

命令：`redis-cli -a 密码 --bigkeys`

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858193.png" alt="image-20241108224908942" style="zoom:50%;" />

缺点：**只能**扫描到各个数据类型中**TOP1**的数据信息

**②scan扫描（推荐）**

自己编程，利用scan扫描Redis中的所有key，利用strlen、hlen等命令判断key的长度（此处不建议使用MEMORY USAGE）

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858194.png" alt="image-20241108224919534" style="zoom:50%;" />

scan 命令调用完后每次会返回2个元素，第一个是下一次迭代的光标，第一次光标会设置为0，当最后一次scan 返回的光标等于0时，表示整个scan遍历结束了，第二个返回的是List，一个匹配的key的数组

```java
import com.heima.jedis.util.JedisConnectionFactory;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.ScanResult;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class JedisTest {
    private Jedis jedis;

    @BeforeEach
    void setUp() {
        // 1.建立连接
        // jedis = new Jedis("192.168.150.101", 6379);
        jedis = JedisConnectionFactory.getJedis();
        // 2.设置密码
        jedis.auth("123321");
        // 3.选择库
        jedis.select(0);
    }

    final static int STR_MAX_LEN = 10 * 1024;
    final static int HASH_MAX_LEN = 500;

    @Test
    void testScan() {
        int maxLen = 0;
        long len = 0;

        String cursor = "0";
        do {
            // 扫描并获取一部分key
            ScanResult<String> result = jedis.scan(cursor);
            // 记录cursor
            cursor = result.getCursor();
            List<String> list = result.getResult();
            if (list == null || list.isEmpty()) {
                break;
            }
            // 遍历
            for (String key : list) {
                // 判断key的类型
                String type = jedis.type(key);
                switch (type) {
                    case "string":
                        len = jedis.strlen(key);
                        maxLen = STR_MAX_LEN;
                        break;
                    case "hash":
                        len = jedis.hlen(key);
                        maxLen = HASH_MAX_LEN;
                        break;
                    case "list":
                        len = jedis.llen(key);
                        maxLen = HASH_MAX_LEN;
                        break;
                    case "set":
                        len = jedis.scard(key);
                        maxLen = HASH_MAX_LEN;
                        break;
                    case "zset":
                        len = jedis.zcard(key);
                        maxLen = HASH_MAX_LEN;
                        break;
                    default:
                        break;
                }
                if (len >= maxLen) {
                    System.out.printf("Found big key : %s, type: %s, length or size: %d %n", key, type, len);
                }
            }
        } while (!cursor.equals("0"));
    }
    
    @AfterEach
    void tearDown() {
        if (jedis != null) {
            jedis.close();
        }
    }

}
```

**③第三方工具**

- 利用第三方工具，如 Redis-Rdb-Tools 分析RDB快照文件，全面分析内存使用情况
- https://github.com/sripathikrishnan/redis-rdb-tools
- 缺点：只能离线分析，具有数据延迟性

**④网络监控（推荐，但贵）**

- 自定义工具，监控进出Redis的网络数据，超出预警值时主动告警
- 一般阿里云搭建的云服务器就有相关监控页面

![image-20241108224934649](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858195.png)

##### 9.1.2.3、如何删除BigKey

BigKey内存占用较多，即便是删除这样的key也需要耗费很长时间，导致Redis主线程阻塞，引发一系列问题。

- redis 3.0 及以下版本
  - 如果是集合类型，则遍历BigKey的元素，先逐个删除子元素，最后删除BigKey

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858196.png" alt="image-20241108224947480" style="zoom:50%;" />

- Redis 4.0以后
  - Redis在4.0后提供了异步删除的命令：**unlink**

#### 9.1.3、恰当的数据类型

**例1：比如存储一个User对象，我们有三种存储方式：**

**①方式一：json字符串**

| user:1 | {"name": "Jack", "age": 21} |
| :----: | :-------------------------: |

优点：实现简单粗暴

缺点：数据耦合，不够灵活

**②方式二：字段打散**

| user:1:name | Jack |
| :---------: | :--: |
| user:1:age  |  21  |

优点：可以灵活访问对象任意字段

缺点：占用空间大、没办法做统一控制

**③方式三：hash（推荐）**

<table>
	<tr>
		<td rowspan="2">user:1</td>
        <td>name</td>
        <td>jack</td>
	</tr>
	<tr>
		<td>age</td>
		<td>21</td>
	</tr>
</table>


优点：底层使用ziplist，空间占用小，可以灵活访问对象的任意字段

缺点：代码相对复杂

**例2：假如有hash类型的key，其中有100万对field和value，field是自增id，这个key存在什么问题？如何优化？**

<table>
	<tr style="color:red">
		<td>key</td>
        <td>field</td>
        <td>value</td>
	</tr>
	<tr>
		<td rowspan="3">someKey</td>
		<td>id:0</td>
        <td>value0</td>
	</tr>
    <tr>
		<td>.....</td>
        <td>.....</td>
	</tr>
    <tr>
        <td>id:999999</td>
        <td>value999999</td>
    </tr>
</table>


存在的问题：

- hash的entry数量超过500时，会使用**哈希表**而不是**ZipList**，内存占用较多
  - <img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858197.png" alt="image-20241108225022093" style="zoom:50%;" />
- 可以通过**hash-max-ziplist-entries**配置entry上限。但是如果entry过多就会导致BigKey问题
  - config set/get hash-max-ziplist-entries (默认是512)
  - 前面说过集合类型的长度最好不要超过1000，因此不推荐该参数设置的过大


> 可以使用命令	info memory 查看redis实例使用的内存数量

**方案一：调节hash-max-ziplist-entries大小**

调大hash-max-ziplist-entries参数的值，但最好不要超过1000

**方案二：拆分为String**

拆分为string类型

<table>
	<tr style="color:red">
		<td>key</td>
        <td>value</td>
	</tr>
	<tr>
		<td>id:0</td>
        <td>value0</td>
	</tr>
    <tr>
		<td>.....</td>
        <td>.....</td>
	</tr>
    <tr>
        <td>id:999999</td>
        <td>value999999</td>
    </tr>
</table>


存在的问题：

- string结构底层没有太多内存优化，**内存占用较多**，可以发现比原来hash的内存占用还要高

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858198.png" alt="image-20241108225040965" style="zoom:50%;" />

- 想要批量获取这些数据比较麻烦

**方案三：Hash分片（推荐）**

拆分为小的hash，将 id / 100 作为key， 将id % 100 作为field，这样每100个元素为一个Hash

<table>
	<tr style="color:red">
		<td>key</td>
        <td>field</td>
        <td>value</td>
	</tr>
	<tr>
        <td rowspan="3">key:0</td>
		<td>id:00</td>
        <td>value0</td>
	</tr>
    <tr>
		<td>.....</td>
        <td>.....</td>
	</tr>
    <tr>
        <td>id:99</td>
        <td>value99</td>
    </tr>
    <tr>
        <td rowspan="3">key:1</td>
		<td>id:00</td>
        <td>value100</td>
	</tr>
    <tr>
		<td>.....</td>
        <td>.....</td>
	</tr>
    <tr>
        <td>id:99</td>
        <td>value199</td>
    </tr>
    <tr>
    	<td colspan="3">....</td>
    </tr>
    <tr>
        <td rowspan="3">key:9999</td>
		<td>id:00</td>
        <td>value999900</td>
	</tr>
    <tr>
		<td>.....</td>
        <td>.....</td>
	</tr>
    <tr>
        <td>id:99</td>
        <td>value999999</td>
    </tr>
</table>

数据导入测试案例：

```java
public class JedisTest {
    private Jedis jedis;

    @BeforeEach
    void setUp() {
        // 1.建立连接
        // jedis = new Jedis("192.168.150.101", 6379);
        jedis = JedisConnectionFactory.getJedis();
        // 2.设置密码
        jedis.auth("123321");
        // 3.选择库
        jedis.select(0);
    }

    @Test
    void testSetBigKey() {
        Map<String, String> map = new HashMap<>();
        for (int i = 1; i <= 650; i++) {
            map.put("hello_" + i, "world!");
        }
        jedis.hmset("m2", map);
    }

    @Test
    void testBigHash() {
        Map<String, String> map = new HashMap<>();
        for (int i = 1; i <= 100000; i++) {
            map.put("key_" + i, "value_" + i);
        }
        jedis.hmset("test:big:hash", map);
    }

    @Test
    void testBigString() {
        for (int i = 1; i <= 100000; i++) {
            jedis.set("test:str:key_" + i, "value_" + i);
        }
    }

    @Test
    void testSmallHash() {
        int hashSize = 100;
        Map<String, String> map = new HashMap<>(hashSize);
        for (int i = 1; i <= 100000; i++) {
            int k = (i - 1) / hashSize;
            int v = i % hashSize;
            map.put("key_" + v, "value_" + v);
            if (v == 0) {
                jedis.hmset("test:small:hash_" + k, map);
            }
        }
    }

    @AfterEach
    void tearDown() {
        if (jedis != null) {
            jedis.close();
        }
    }
}
```

#### 9.1.4、总结

- Key的最佳实践
  - 固定格式：[业务名]:[数据名]:[id]
  - 足够简短：不超过44字节
  - 不包含特殊字符
- Value的最佳实践：
  - 合理的拆分数据，拒绝BigKey
  - 选择合适数据结构
  - 单个key的value小于10K，Hash结构的entry数量不要超过1000
  - 设置合理的超时时间

### 9.2、批处理优化

#### 9.2.1、Pipeline

##### 9.2.1.1、我们的客户端与redis服务器是这样交互的

单个命令的执行流程

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858199.png" alt="image-20241108225114510" style="zoom:50%;" />

N条命令的执行流程

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858200.png" alt="image-20241108225551124" style="zoom:50%;" />

redis处理指令是很快的，**主要花费的时候在于网络传输**。于是乎很容易想到将多条指令批量的传输给redis

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858201.png" alt="image-20241108225603599" style="zoom:50%;" />

##### 2.1.2、MSet

Redis提供了很多Mxxx这样的命令，可以实现批量插入数据，例如：

- mset
- hmset

利用mset批量插入10万条数据

> 注意：MSet也要分批，因为不要在一次批处理中传输太多命令，否则单次命令占用带宽过多，会导致网络阻塞
>
> 并且MSet只适用于String数据类型

```java
@Test
void testMxx() {
    String[] arr = new String[2000];
    int j;
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        j = (i % 1000) << 1;
        arr[j] = "test:key_" + i;
        arr[j + 1] = "value_" + i;
        if (j == 0) {
            jedis.mset(arr);
        }
    }
    long e = System.currentTimeMillis();
    System.out.println("time: " + (e - b));
}
```

##### 2.1.3、Pipeline

MSET虽然可以批处理，但是却只能操作部分数据类型，因此如果有对复杂数据类型的批处理需要，建议使用Pipeline

```java
@Test
void testPipeline() {
    // 创建管道
    Pipeline pipeline = jedis.pipelined();
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        // 放入命令到管道
        pipeline.set("test:key_" + i, "value_" + i);
        if (i % 1000 == 0) {
            // 每放入1000条命令，批量执行
            pipeline.sync();
        }
    }
    long e = System.currentTimeMillis();
    System.out.println("time: " + (e - b));
}
```

##### 总结

批量处理的方案：
①原生的M操作
②Pipeline批处理

注意事项：
①批处理时不建议一次携带太多命令
②Pipeline的多个命令之间**不具备原子性**，而M操作具有原子性



#### 9.2.2、集群下的批处理

如MSET或Pipeline这样的批处理需要在一次请求中携带多条命令，而此时如果Redis是一个集群，那**批处理命令的多个key必须落在一个`插槽`中**，否则就会导致执行失败。大家可以想一想这样的要求其实很难实现，因为我们在批处理时，可能一次要插入很多条数据，这些数据很有可能不会都落在相同的节点上，这就会导致报错了

这个时候，我们可以找到4种解决方案

![image-20241108225657609](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858202.png)

- **第一种方案：**串行执行，所以这种方式没有什么意义，当然，执行起来就很简单了，缺点就是耗时过久。
- **第二种方案：**串行slot，简单来说，就是执行前，客户端先计算一下对应的key的slot，一样slot的key就放到一个组里边，不同的，就放到不同的组里边，然后对每个组执行pipeline的批处理，他就能串行执行各个组的命令，这种做法比第一种方法耗时要少，但是缺点呢，相对来说复杂一点，所以这种方案还需要优化一下
- **第三种方案：**并行slot，相较于第二种方案，在分组完成后串行执行，第三种方案，就变成了并行执行各个命令，所以他的耗时就非常短，但是实现呢，也更加复杂。
- **第四种方案：**hash_tag，redis计算key的slot的时候，其实是根据key的有效部分来计算的，通过这种方式就能一次处理所有的key，这种方式耗时最短，实现也简单，但是如果通过操作key的有效部分，那么就会导致所有的key都落在一个节点上，产生数据倾斜的问题，所以我们推荐使用第三种方式。

##### 9.2.2.1 Jedis串行化执行代码实践

```java
public class JedisClusterTest {

    private JedisCluster jedisCluster;

    @BeforeEach
    void setUp() {
        // 配置连接池
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(8);
        poolConfig.setMaxIdle(8);
        poolConfig.setMinIdle(0);
        poolConfig.setMaxWaitMillis(1000);
        HashSet<HostAndPort> nodes = new HashSet<>();
        nodes.add(new HostAndPort("192.168.150.101", 7001));
        nodes.add(new HostAndPort("192.168.150.101", 7002));
        nodes.add(new HostAndPort("192.168.150.101", 7003));
        nodes.add(new HostAndPort("192.168.150.101", 8001));
        nodes.add(new HostAndPort("192.168.150.101", 8002));
        nodes.add(new HostAndPort("192.168.150.101", 8003));
        jedisCluster = new JedisCluster(nodes, poolConfig);
    }

    @Test
    void testMSet() {
        jedisCluster.mset("name", "Jack", "age", "21", "sex", "male");

    }

    @Test
    void testMSet2() {
        Map<String, String> map = new HashMap<>(3);
        map.put("name", "Jack");
        map.put("age", "21");
        map.put("sex", "Male");
        //对Map数据进行分组。根据相同的slot放在一个分组
        //key就是slot，value就是一个组
        Map<Integer, List<Map.Entry<String, String>>> result = map.entrySet()
                .stream()
                .collect(Collectors.groupingBy(
                        entry -> ClusterSlotHashUtil.calculateSlot(entry.getKey()))
                );
        //串行的去执行mset的逻辑
        for (List<Map.Entry<String, String>> list : result.values()) {
            String[] arr = new String[list.size() * 2];
            int j = 0;
            for (int i = 0; i < list.size(); i++) {
                j = i<<2;
                Map.Entry<String, String> e = list.get(0);
                arr[j] = e.getKey();
                arr[j + 1] = e.getValue();
            }
            jedisCluster.mset(arr);
        }
    }

    @AfterEach
    void tearDown() {
        if (jedisCluster != null) {
            jedisCluster.close();
        }
    }
}
```

##### 9.2.2.2 Spring集群环境下批处理代码（推荐）

> Spring已经帮我们集成了 **并行Slot** 的处理方式，可以直接使用，推荐使用Spring的框架来处理集群批处理任务

```java
   @Test
    void testMSetInCluster() {
        Map<String, String> map = new HashMap<>(3);
        map.put("name", "Rose");
        map.put("age", "21");
        map.put("sex", "Female");
        stringRedisTemplate.opsForValue().multiSet(map);


        List<String> strings = stringRedisTemplate.opsForValue().multiGet(Arrays.asList("name", "age", "sex"));
        strings.forEach(System.out::println);

    }
```

**原理分析**

在RedisAdvancedClusterAsyncCommandsImpl 类中

首先根据slotHash算出来一个partitioned的map，map中的key就是slot，而他的value就是对应的对应相同slot的key对应的数据

通过 RedisFuture<String> mset = super.mset(op)；进行异步的消息发送

```Java
@Override
public RedisFuture<String> mset(Map<K, V> map) {

    Map<Integer, List<K>> partitioned = SlotHash.partition(codec, map.keySet());

    if (partitioned.size() < 2) {
        return super.mset(map);
    }

    Map<Integer, RedisFuture<String>> executions = new HashMap<>();

    for (Map.Entry<Integer, List<K>> entry : partitioned.entrySet()) {

        Map<K, V> op = new HashMap<>();
        entry.getValue().forEach(k -> op.put(k, map.get(k)));

        RedisFuture<String> mset = super.mset(op);
        executions.put(entry.getKey(), mset);
    }

    return MultiNodeExecution.firstOfAsync(executions);
}
```



#### 9.2.3、Redis事务

##### 9.2.3.1 Redis事务的含义和与MySQL事务的对比

**数据库事务**：在一次跟数据库的连接会话当中，所有执行的SQL，要么一起成功，要么一起失败

**原子性**：一个事务中的所有操作要么全部成功，要么全部失败回滚，不能只执行其中的一部分操作



**Redis事务**：可以一次执行多个命令，本质是**一组命令的集合**。一个事务中的所有命令都会序列化，按顺序地串行化执行而**不会被其它命令插入，不许加塞**，即一个队列中，一次性、顺序性、排他性的执行一系列命令

Redis事务与MySQL事务的**区别**：

![image-20241108225813258](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858203.png)

> 1. 单独的隔离操作：Redis的事务只是能够**保证一组命令能够连续独占的执行**，因为Redis是单线程架构，所以执行完事务内的所有命令前不能去执行其它请求
> 2. 没有隔离级别的概念：**事务提交前任何命令都不会被实际执行**，所以不会存在并发安全问题
> 3. 不保证原子性：若开始执行事务内的所有命令，若其中有命令执行失败，**不提供回滚机制**
> 4. 排他性：Redis会**保证一个事务内的命令依次执行，而不会被其它命令插入**



##### 9.2.3.2 Redis事务的使用和细节

Redis事务相关的命令：

1. MULTI：标志着一个事务的开始
2. EXEC：执行事务内的所有命令
3. DISCARD：取消事务，放弃执行事务内的所有命令
4. WATCH key [key ...]：监视一个或多个key，如果在事务执行之前这个（或这些）key被其他命令所改动，那么事务将被打断。
5. UNWATCH：取消WATCH命令对所有key的监控

Redis事务的使用：

- 正常执行：

  ```shell
  MULTI
  ...
  EXEC
  ```

- 放弃执行：

  ```shell
  MULTI
  ...
  DISCARD
  ```

- 全体连坐（语法编译错误，该组命令全部被舍弃）：

  ```shell
  MULTi
  ...
  set k3 	#故意写错语法
  ...
  EXEC
  
  # 虽然只有一条语句语法错误，可是所有的命令都不被执行
  ```

- 冤头债主（语法没错，编译时没检查出错误，对的命令执行，不对的不执行）

  ```shell
  MULTI
  ...
  set email "spongehah@163.com"
  INCR email  	#对字符串做自增操作，编译不会出错，但运行会出错
  EXEC
  
  # 语法没错，编译时没检查出错误，对的命令执行，不对的不执行
  ```



##### 9.2.3.3 Redis开启Watch监控（乐观锁的使用）

Redis Watch机制采用**乐观锁**，watch key时**不会上锁**，但**提交时的版本必须大于当前版本才能执行**

当一个会话监听了某个key，然后使用MULTI开启事务后，要对该key进行操作，但是还未使用EXEC进行执行，此时**另一个会话**若是**修改了这个被监控的key**，那么前一个会话执行EXEC时**所有命令将会不被执行**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858204.png" alt="image-20241108225826644"  />



##### 9.2.3.4 Pipeline与原生批处理命令、Redis事务的对比

- pipeline与原生批处理命令对比：
  - **原生批量命令是原子性**（例如：mset,mget),**pipeline是非原子性**
  - 原生批量命令一次**只能执行一种命令**，pipeline支持批量执行**不同命令**
  - 原生批命令是**服务端实现**，而pipeline需要**服务端与客户端共同完成**
- pipeline与事务对比：
  - 事务具有排他性，管道不具有排他性
  - 管道一次性将多条命令发送到服务器，事务是一条一条的发，只有在接收到exec命令后才会执行，**管道**不执行事务时**会被其它命令插入、加塞**，而**Redis事务**中的所有命令**开始执行后不会被插入、加塞**



### 9.3、服务器端优化-持久化配置

**持久化配置：**

Redis的持久化虽然可以保证数据安全，但也会带来很多额外的开销，因此持久化请遵循下列建议：

* 用来做**缓存**的Redis实例尽量**不要开启**持久化功能
* 建议**关闭RDB**持久化功能，**使用AOF**持久化
* 利用**脚本定期在slave节点做RDB**，实现数据备份
* 设置合理的rewrite阈值，**避免频繁的bgrewrite**
* 配置no-appendfsync-on-rewrite = yes，**禁止在rewrite期间做aof**，避免因AOF引起的阻塞

**部署有关建议：**

* Redis实例的物理机要预留足够内存，应对fork和rewrite
* 单个Redis实例内存上限不要太大，例如4G或8G。可以加快fork的速度、减少主从同步、数据迁移压力
* 不要与CPU密集型应用部署在一起
* 不要与高硬盘负载应用一起部署。例如：数据库、消息队列

### 9.4、服务器端优化-慢查询优化

#### 9.4.1 什么是慢查询

并不是很慢的查询才是慢查询，而是：在Redis执行时耗时超过某个阈值的命令，称为慢查询。

慢查询的危害：由于Redis是单线程的，所以当客户端发出指令后，他们都会进入到redis底层的queue来执行，如果此时有一些慢查询的数据，就会导致大量请求阻塞，从而引起报错，所以我们需要解决慢查询问题。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858205.png" alt="image-20241108225843362" style="zoom:50%;" />

- **慢查询的阈值**可以通过配置指定：
  - **slowlog-log-slower-than**：慢查询阈值，单位是微秒。**默认是10000，建议1000**
- 慢查询会被放入**慢查询日志**中，日志的**长度**有上限，可以通过配置指定：

  - **slowlog-max-len**：慢查询日志（本质是一个队列）的长度。**默认是128，建议1000**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858206.png" alt="image-20241108225851397" style="zoom:50%;" />

修改这两个配置可以使用：config set命令：

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858207.png" alt="image-20241108230117461" style="zoom:50%;" />

#### 9.4.2 如何查看慢查询

知道了以上内容之后，那么咱们如何去查看慢查询日志列表呢：

* **slowlog len**：查询慢查询日志长度
* **slowlog get [n]**：读取n条慢查询日志
* **slowlog reset**：清空慢查询列表

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858208.png" alt="image-20241108230132320" style="zoom:50%;" />

### 9.5、服务器端优化-命令及安全配置

 安全可以说是服务器端一个非常重要的话题，如果安全出现了问题，那么一旦这个漏洞被一些坏人知道了之后，并且进行攻击，那么这就会给咱们的系统带来很多的损失，所以我们这节课就来解决这个问题。

Redis会绑定在0.0.0.0:6379，这样将会将Redis服务暴露到公网上，而Redis如果没有做身份认证，会出现严重的安全漏洞.
漏洞重现方式：https://cloud.tencent.com/developer/article/1039000

为什么会出现不需要密码也能够登录呢，主要是Redis考虑到每次登录都比较麻烦，所以Redis就有一种ssh免秘钥登录的方式，生成一对公钥和私钥，私钥放在本地，公钥放在redis端，当我们登录时服务器，再登录时候，他会去解析公钥和私钥，如果没有问题，则不需要利用redis的登录也能访问，这种做法本身也很常见，但是这里有一个前提，前提就是公钥必须保存在服务器上，才行，但是Redis的漏洞在于在不登录的情况下，也能把秘钥送到Linux服务器，从而产生漏洞

漏洞出现的核心的原因有以下几点：

* Redis未设置密码
* 利用了Redis的config set命令动态修改Redis配置
* 使用了Root账号权限启动Redis

所以：如何解决呢？我们可以采用如下几种方案

为了避免这样的漏洞，这里给出一些**建议**：

* Redis一定要设置密码
* 禁止线上使用下面命令：keys、flushall、flushdb、config set等命令。可以利用rename-command禁用。
* bind：限制网卡，禁止外网网卡访问
* 开启防火墙
* 不要使用Root账户启动Redis
* 尽量不是有默认的端口

### 9.6、服务器端优化-Redis内存划分和内存配置

当Redis内存不足时，可能导致Key频繁被删除、响应时间变长、QPS不稳定等问题。当内存使用率达到90%以上时就需要我们警惕，并快速定位到内存占用的原因。

**有关碎片问题分析**

Redis底层分配并不是这个key有多大，他就会分配多大，而是有他自己的分配策略，比如8,16,20等等，假定当前key只需要10个字节，此时分配8肯定不够，那么他就会分配16个字节，多出来的6个字节就不能被使用，这就是我们常说的 碎片问题

**进程内存问题分析：**

这片内存，通常我们都可以忽略不计

**缓冲区内存问题分析：**

一般包括客户端缓冲区、AOF缓冲区、复制缓冲区等。客户端缓冲区又包括输入缓冲区和输出缓冲区两种。这部分内存占用波动较大，所以这片内存也是我们需要重点分析的内存问题。

| **内存占用** | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| 数据内存     | 是Redis最主要的部分，存储Redis的键值信息。主要问题是**BigKey**问题、内存碎片问题 |
| 进程内存     | Redis主进程本身运⾏肯定需要占⽤内存，如代码、常量池等等；这部分内存⼤约⼏兆，在⼤多数⽣产环境中与Redis数据占⽤的内存相⽐可以忽略。 |
| 缓冲区内存   | 一般包括**`客户端缓冲区`、AOF缓冲区、复制缓冲区**等。客户端缓冲区又包括输入缓冲区和输出缓冲区两种。这部分内存占用波动较大，不当使用BigKey，可能导致内存溢出。 |

> 综上所述：当内存使用率过高时，应当考虑 **数据内存** 和 **缓冲区内存** 两部分

于是我们就需要通过一些命令，可以查看到Redis目前的内存分配状态：

* **info memory**：查看内存分配的情况

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858209.png" alt="image-20241108230151144" style="zoom:50%;" />

* **memory xxx**：查看key的主要占用情况
  * memory usage key：查看某个key的内存占用情况
  * memory stats：查看统一的内存占用情况

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858210.png" alt="image-20241108230200921" style="zoom:50%;" />

接下来我们看到了这些配置，最关键的缓存区内存如何定位和解决呢？

内存缓冲区常见的有三种：

* **复制缓冲区**：主从复制的repl_backlog_buf，如果太小可能导致频繁的全量复制，影响性能。通过**repl-backlog-size**来设置，**默认1mb**
* **AOF缓冲区**：AOF刷盘之前的缓存区域，AOF执行rewrite的缓冲区。无法设置容量上限
* **客户端缓冲区**：分为输入缓冲区和输出缓冲区，输入缓冲区最大1G且不能设置。输出缓冲区可以设置

以上复制缓冲区和AOF缓冲区 不会有问题，**最关键就是客户端缓冲区**的问题

客户端缓冲区：指的就是我们发送命令时，客户端用来缓存命令的一个缓冲区，也就是我们向redis输入数据的输入端缓冲区和redis向客户端返回数据的响应缓存区，输入缓冲区最大1G且不能设置，所以这一块我们根本不用担心，如果超过了这个空间，redis会直接断开，因为本来此时此刻就代表着redis处理不过来了，我们需要担心的就是输出端缓冲区

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858211.png" alt="image-20241108232519983"  />

我们在使用redis过程中，处理大量的big value，那么会导致我们的输出结果过多，如果输出缓存区过大，会导致redis直接断开，而默认配置的情况下， 其实他是没有大小的，这就比较坑了，内存可能一下子被占满，会直接导致咱们的redis断开，所以解决方案有两个

1、设置一个大小

2、增加我们带宽的大小，避免我们出现大量数据从而直接超过了redis的承受能力



#### 9.6.1 Redis内存碎片问题

1. Redis为什么会有内存碎片？

   - Redis 存储数据的时候向操作系统申请的内存空间可能会大于数据实际需要的存储空间。比如SDS分配时使用jmalloc分配2的幂次方
   - 频繁修改 Redis 中的数据也会产生内存碎片。
   - （操作系统分配的内存少于128K时使用brk()函数，回收后会放入内存池）

2. 如何查看 Redis 内存碎片的信息？

   - info memory

3. 如何清理 Redis 内存碎片？

   - ```shell
     config set activedefrag yes
     
     # 内存碎片占用空间达到 500mb 的时候开始清理
     config set active-defrag-ignore-bytes 500mb
     # 内存碎片率大于 1.5 的时候开始清理
     config set active-defrag-threshold-lower 50
     
     # 内存碎片清理所占用 CPU 时间的比例不低于 20%
     config set active-defrag-cycle-min 20
     # 内存碎片清理所占用 CPU 时间的比例不高于 50%
     config set active-defrag-cycle-max 50
     ```



### 9.7、服务器端集群优化-集群还是主从

集群虽然具备高可用特性，能实现自动故障恢复，但是如果使用不当，也会存在一些问题：

* 集群完整性问题
* 集群带宽问题
* 数据倾斜问题
* 客户端性能问题
* 命令的集群兼容性问题
* lua和事务问题

 **问题1、在Redis的默认配置中，如果发现任意一个插槽不可用，则整个集群都会停止对外服务：** 

大家可以设想一下，如果有几个slot不能使用，那么此时整个集群都不能用了，我们在开发中，其实最重要的是可用性，所以需要把如下配置修改成no，即有slot不能使用时，我们的redis集群还是可以对外提供服务

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202411091858212.png" alt="image-20241108232536974" style="zoom:50%;" />



**问题2、集群带宽问题**

集群节点之间会不断的互相Ping来确定集群中其它节点的状态。每次Ping携带的信息至少包括：

* 插槽信息
* 集群状态信息

集群中节点越多，集群状态信息数据量也越大，10个节点的相关信息可能达到1kb，此时每次集群互通需要的带宽会非常高，这样会导致集群中大量的带宽都会被ping信息所占用，这是一个非常可怕的问题，所以我们需要去解决这样的问题

**解决途径：**

* 避免大集群，集群节点数不要太多，最好少于1000，如果业务庞大，则建立多个集群。
* 避免在单个物理机中运行太多Redis实例
* 配置合适的cluster-node-timeout值



**问题3、命令的集群兼容性问题**

有关这个问题咱们已经探讨过了，当我们使用批处理的命令时，redis要求我们的key必须落在相同的slot上，然后大量的key同时操作时，是无法完成的，所以客户端必须要对这样的数据进行处理，这些方案我们之前已经探讨过了，所以不再这个地方赘述了。

**问题4、lua和事务的问题**

lua和事务都是要保证原子性问题，如果你的key不在一个节点，那么是无法保证lua的执行和事务的特性的，所以在集群模式是没有办法执行lua和事务的



**那我们到底是集群还是主从**

单体Redis（主从Redis）已经能达到万级别的QPS，并且也具备很强的高可用特性。如果主从能满足业务需求的情况下，所以如果不是在万不得已的情况下，尽量不搭建Redis集群
