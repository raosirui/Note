# JetCache



**项目描述：**本项目是基于Alibaba开源的通用缓存访问框架 JetCache ，新增通过key*前缀进行本地缓存和redis缓存删除功能的增强版本。

**新增功能：**

- 新增**基于PatriciaTrie的本地缓存**功能
- 本地缓存ConcurrentPatriciaTrieCache的**前缀删除功能**
- Redis缓存RedisCache的**前缀删除功能**







从代码来看，这个增强版本的JetCache主要通过以下几个方面实现了新增功能：

------

### 1. **前缀删除功能的实现**

#### **本地缓存（`ConcurrentPatriciaTrieCache`）**

- **核心实现**：`ConcurrentPatriciaTrieCache` 是基于 [Patricia Trie](https://en.wikipedia.org/wiki/Patricia_trie) 数据结构的本地缓存实现。Patricia Trie 是一种前缀树，能够高效地存储和检索具有公共前缀的键。

- **前缀删除**：通过 `delByPrefix` 方法实现前缀删除。该方法利用了 `PatriciaTrie` 的 `prefixMap` 方法，该方法可以获取所有以指定前缀开头的键值对。然后，通过 `removeAll` 方法批量删除这些键。

  java复制

  ```java
  @Override
  public boolean delByPrefix(K keyPrefix) {
      Object realKeyPrefix = buildKey(keyPrefix);
      PatriciaTrie unwrap = this.unwrap(PatriciaTrie.class);
      SortedMap sortedMap = unwrap.prefixMap(realKeyPrefix);
      Set set = sortedMap.keySet();
      this.removeAll(set);
      return true;
  }
  ```

- **线程安全**：`ConcurrentPatriciaTrieCache` 的内部实现使用了 `ReentrantLock` 来保证线程安全。所有对 `PatriciaTrie` 的操作（如 `put`、`get`、`remove` 等）都在锁的保护下完成。

#### **Redis 缓存（`RedisCache`）**

- **核心实现**：`RedisCache` 是基于 Redis 的远程缓存实现。它支持多种 Redis 客户端（如 Jedis、Lettuce、Spring Data Redis 等）。

- **前缀删除**：通过 `delByPrefix` 方法实现前缀删除。该方法使用 Redis 的 `SCAN` 命令扫描所有以指定前缀开头的键，并通过 `DEL` 命令批量删除这些键。

  java复制

  ```java
  public boolean delByPrefix(K keyPrefix) {
      byte[] realKeyPrefix = buildKey(keyPrefix);
      String keyPrefixStr = new String(realKeyPrefix, StandardCharsets.UTF_8);
      keyPrefixStr = keyPrefixStr.endsWith("*") ? keyPrefixStr : keyPrefixStr + "*";
      KeyBinaryCommands jedis = (KeyBinaryCommands) writeCommands();
      try {
          String cursor = "0";
          ScanParams scanParams = new ScanParams();
          do {
              scanParams.count(1000);
              scanParams.match(keyPrefixStr);
              ScanResult<byte[]> scan = jedis.scan(cursor.getBytes(StandardCharsets.UTF_8), scanParams);
              List<byte[]> result = scan.getResult();
              cursor = scan.getCursor();
              if (!result.isEmpty()) {
                  jedis.del(result.toArray(new byte[result.size()][]));
              }
          } while (!"0".equalsIgnoreCase(cursor));
      } finally {
          closeJedis(jedis);
      }
      return true;
  }
  ```

- **效率优化**：为了避免 `SCAN` 命令的性能问题，代码中设置了 `COUNT` 参数（每次扫描返回的最大键数）为 1000，以提高扫描效率。

------

### 2. **新增基于 Patricia Trie 的本地缓存功能**

- **实现方式**：`ConcurrentPatriciaTrieCache` 是通过扩展 `AbstractEmbeddedCache` 实现的。它使用了 Apache Commons Collections 的 `PatriciaTrie` 数据结构，并在其基础上增加了线程安全和过期清理功能。

- **内部结构**：

  - **`LRUMap`**：内部实现了一个基于 LRU（最近最少使用）策略的 `PatriciaTrie`，用于限制缓存的大小。当缓存达到最大容量时，会自动淘汰最久未使用的键值对。
  - **过期清理**：通过 `cleanExpiredEntry` 方法定期清理过期的键值对。该方法会遍历 `PatriciaTrie`，移除所有已过期的条目。

  java复制

  ```java
  void cleanExpiredEntry() {
      lock.lock();
      long t = System.currentTimeMillis();
      try {
          for (Iterator it = entrySet().iterator(); it.hasNext(); ) {
              Map.Entry en = (Map.Entry) it.next();
              Object value = en.getValue();
              if (value != null) {
                  CacheValueHolder h = (CacheValueHolder) value;
                  if (t >= h.getExpireTime()) {
                      it.remove();
                  }
              } else {
                  logger.error("key " + en.getKey() + " is null");
              }
          }
      } finally {
          lock.unlock();
      }
  }
  ```

- **自动清理机制**：`Cleaner` 类通过 `ScheduledExecutorService` 定期调用 `cleanExpiredEntry` 方法，清理过期的缓存条目。

------

### 3. **支持多种缓存系统**

- **本地缓存**
  - **`Caffeine`**：基于本地内存的高性能缓存库，支持 LRU、LFU 等多种淘汰策略。
  - **`LinkedHashMap`**：JetCache 自己实现的简易 LRU 缓存，通过 `LinkedHashMap` 的特性实现。
  - **`ConcurrentPatriciaTrieCache`**：基于 Patricia Trie 的本地缓存，支持前缀删除。
- **远程缓存**
  - **`Redis`**：支持多种 Redis 客户端（Jedis、Lettuce、Spring Data Redis、Redisson），并通过注解和配置灵活地支持前缀删除等高级功能。
- **扩展性**
  - **新增缓存类型**：通过实现 `Cache` 接口和 `CacheConfig`，可以很容易地添加新的缓存类型。例如，`ConcurrentPatriciaTrieCache` 的实现就是通过扩展 `AbstractEmbeddedCache` 并实现 `InnerMap` 接口完成的。
  - **自动配置**：通过 Spring Boot 的自动配置机制（如 `ConcurrentPatriciaTrieCacheAutoConfiguration`），可以自动加载和初始化新的缓存类型。

------

### 4. **代码结构和设计模式**

- **工厂模式**：通过 `ConcurrentPatriciaTrieCacheBuilder` 提供缓存构建器，允许用户通过链式调用配置缓存实例。

  java复制

  ```java
  public static ConcurrentPatriciaTrieCacheBuilderImpl createConcurrentPatriciaTrieCacheBuilder() {
      return new ConcurrentPatriciaTrieCacheBuilderImpl();
  }
  ```

- **装饰器模式**：`AbstractEmbeddedCache` 和 `AbstractCache` 提供了通用的缓存操作逻辑，具体的缓存实现（如 `ConcurrentPatriciaTrieCache` 和 `RedisCache`）通过继承和扩展这些抽象类实现具体功能。

- **策略模式**：通过 `CacheConfig` 和注解（如 `@Cached`、`@CacheInvalidate`）灵活地配置缓存行为（如过期策略、加载策略等）。

------

### 5. **使用场景和优势**

- **前缀删除功能**：适用于需要批量清理缓存的场景，例如在多租户系统中，可以根据租户 ID 删除所有相关缓存。
- **多级缓存**：通过 `MultiLevelCache` 支持本地缓存和远程缓存的结合，能够有效降低远程缓存的压力，同时提高系统的响应速度。
- **扩展性**：通过简单的实现和配置，可以轻松支持新的缓存系统，满足不同场景下的需求。
- **性能优化**：通过线程安全的 Patricia Trie 和 Redis 的批量操作，提高了缓存的读写效率。

------

总结来说，这个增强版本的 JetCache 通过扩展和优化现有的缓存功能，提供了更强大的前缀删除能力，并通过灵活的配置和扩展机制，支持多种缓存系统，满足了复杂业务场景下的需求。





















能不能给出另一种更好的方案优化一下，并且保证性能最佳



![Avatar](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202502242223622.png)

当然可以！在优化 Redis 前缀删除功能时，除了使用 `SCAN` 命令外，还可以考虑一些更高效的方法来提升性能，同时减少对 Redis 的压力。以下是一个更优的方案，结合了 **Redis 的 Lua 脚本** 和 **异步处理机制**，以实现高性能的前缀删除功能。

------

### **优化方案：使用 Lua 脚本结合异步处理**

#### **1. Lua 脚本的优势**

Redis 的 Lua 脚本允许在 Redis 服务器端执行一系列命令，而无需多次往返客户端和服务器。这可以显著减少网络延迟，并且 Lua 脚本是原子性的，可以避免并发问题。

#### **2. 方案设计**

1. **编写 Lua 脚本**：
   编写一个 Lua 脚本，用于扫描和删除匹配前缀的键。Lua 脚本可以在 Redis 服务器端高效地执行批量删除操作。
2. **异步执行**：
   使用 Redis 的异步机制（如 Redisson 的 RBatch 或 Jedis 的 Pipeline）批量执行 Lua 脚本，减少对 Redis 的压力。
3. **分批处理**：
   为了避免单次删除过多键导致 Redis 堵塞，可以将删除操作分批进行，每批处理一定数量的键。

------

### **3. Lua 脚本实现**

以下是一个 Lua 脚本示例，用于扫描和删除匹配前缀的键：

lua复制

```lua
-- Lua 脚本：删除匹配前缀的键
local cursor = "0"
local keys = {}
local count = 0
local limit = tonumber(ARGV[1])  -- 每批处理的最大键数

repeat
    local result = redis.call("SCAN", cursor, "MATCH", ARGV[2], "COUNT", 1000)
    cursor = result[1]
    local batch_keys = result[2]
    for _, key in ipairs(batch_keys) do
        table.insert(keys, key)
        count = count + 1
        if count >= limit then
            break
        end
    end
until cursor == "0" or count >= limit

-- 删除匹配的键
if #keys > 0 then
    redis.call("DEL", unpack(keys))
end

return count  -- 返回删除的键数量
```

### **4. Java 代码实现**

在 Java 中，可以通过 Redis 客户端（如 Jedis 或 Redisson）调用 Lua 脚本。以下是基于 Jedis 的实现：

java复制

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.ScanParams;
import redis.clients.jedis.ScanResult;

public class RedisPrefixDeleter {
    private static final String LUA_SCRIPT = 
        "local cursor = '0'\n" +
        "local keys = {}\n" +
        "local count = 0\n" +
        "local limit = tonumber(ARGV[1])\n" +
        "repeat\n" +
        "    local result = redis.call('SCAN', cursor, 'MATCH', ARGV[2], 'COUNT', 1000)\n" +
        "    cursor = result[1]\n" +
        "    local batch_keys = result[2]\n" +
        "    for _, key in ipairs(batch_keys) do\n" +
        "        table.insert(keys, key)\n" +
        "        count = count + 1\n" +
        "        if count >= limit then\n" +
        "            break\n" +
        "        end\n" +
        "    end\n" +
        "until cursor == '0' or count >= limit\n" +
        "if #keys > 0 then\n" +
        "    redis.call('DEL', unpack(keys))\n" +
        "end\n" +
        "return count";

    public static void delByPrefix(Jedis jedis, String keyPrefix, int batchSize) {
        // 调用 Lua 脚本
        Long deletedCount = jedis.eval(
            LUA_SCRIPT,
            0,  // 不需要额外的键
            batchSize,  // 每批处理的最大键数
            keyPrefix + "*"  // 匹配模式
        );

        System.out.println("Deleted " + deletedCount + " keys with prefix: " + keyPrefix);
    }

    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            delByPrefix(jedis, "yourKeyPrefix", 1000);  // 每批删除 1000 个键
        }
    }
}
```

### **5. 性能优化点**

1. **Lua 脚本的原子性**：
   Lua 脚本在 Redis 服务器端执行，避免了多次往返客户端和服务器的开销，减少了网络延迟。
2. **分批处理**：
   通过设置每批处理的最大键数（`batchSize`），可以避免一次性删除过多键导致 Redis 堵塞。每批处理一定数量的键后，Lua 脚本会自动停止。
3. **异步执行**：
   如果需要进一步优化性能，可以将 Lua 脚本的调用放入异步任务中，例如使用 Redisson 的 `RBatch` 或 Java 的 `CompletableFuture`。
4. **监控与日志**：
   在删除操作完成后，记录删除的键数量和耗时，便于监控和优化。

------

### **6. 异步执行示例**

以下是使用 `CompletableFuture` 实现异步删除的示例：

java复制

```java
import java.util.concurrent.CompletableFuture;

public class RedisPrefixDeleterAsync {
    public static CompletableFuture<Void> delByPrefixAsync(Jedis jedis, String keyPrefix, int batchSize) {
        return CompletableFuture.runAsync(() -> {
            delByPrefix(jedis, keyPrefix, batchSize);
        });
    }

    public static void main(String[] args) {
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            CompletableFuture<Void> future = delByPrefixAsync(jedis, "yourKeyPrefix", 1000);
            future.join();  // 等待异步任务完成
        }
    }
}
```

------

### **7. 总结**

通过使用 Lua 脚本结合异步处理，可以显著提升 Redis 前缀删除功能的性能。Lua 脚本的原子性减少了网络延迟，分批处理避免了 Redis 堵塞，而异步执行则进一步优化了性能。这种方案不仅高效，而且对 Redis 的压力较小，适合在生产环境中使用。

如果你对这个方案有任何疑问或需要进一步优化，请随时告诉我！