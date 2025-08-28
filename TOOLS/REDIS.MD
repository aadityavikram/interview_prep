# 1. What is Redis?

1. Redis = REmote DIctionary Server  
  a. Open-source, in-memory key-value database.  
  b. Often called a data structure server, because values aren’t just strings but rich data types: lists, sets, sorted sets, hashes, streams, etc.  
  c. It is single-threaded, but extremely fast (handles millions of ops/sec).  
  d. Data lives in RAM, but Redis supports persistence (can store snapshots or logs to disk).  

2. Used for:  
  a. Caching (most common use case)  
  b. Message queues / Pub-Sub  
  c. Real-time analytics  
  d. Leaderboards, sessions, counters, rate limiting  
  e. Primary datastore in some high-performance systems.  

3. Key-Value Store  
  a. Everything in Redis is stored as key → value.  
  b. Keys: always strings.  
  c. Values: can be strings, lists, sets, hashes, sorted sets, streams, bitmaps, HyperLogLog, geospatial indexes.  
  d. Example:  
```java
SET user:1 "Aaditya"
GET user:1
```

4. Internal Data Structures  
  a. Redis stores values using efficient C-level data structures:  

  | Redis Type        | Backed By (Internal)                               | Notes                                                    |
  | ----------------- | -------------------------------------------------- | -------------------------------------------------------- |
  | String            | **SDS (Simple Dynamic String)**                    | Safer & more efficient than C strings                    |
  | List              | **QuickList** (linked list + ziplist optimization) | Used for pushing/popping                                 |
  | Hash              | **Hashtable** or **Ziplist/Zipmap**                | Depends on size (small hashes use compact memory layout) |
  | Set               | **Hashtable** or **Intset**                        | Intset for small integer-only sets                       |
  | Sorted Set (ZSet) | **Skiplist + Hash table**                          | O(log n) lookups, supports ranking                       |
  | Stream            | Radix-tree + listpacks                             | Used for event logs                                      |
  | Bitmap            | Bit operations on Strings                          | Space-efficient counters                                 |
  | HyperLogLog       | Probabilistic data structure                       | Approximates cardinality                                 |

Redis dynamically switches internal encodings:  
Example: A small hash (few fields) is stored in a compact array (ziplist) → when it grows, Redis converts it to a hash table.  

5. Single-threaded Event Loop  
  a. Redis uses one thread per instance for executing commands:  
  b. Relies on epoll/kqueue event loop.  
  c. Handles millions of ops/sec because:  
  d. All operations are O(1) or O(log n)  
  e. No context-switching overhead  
  f. Memory access is in RAM (no disk I/O bottleneck)  
  g. Heavy work (e.g. persistence, cluster I/O) can be handled by background threads.  

6. Persistence  
Since Redis is in-memory, it needs durability options: 

  | Method | Features |
  |--------|----------|
  | RDB (Redis Database File) Snapshotting | Saves a snapshot of the dataset at intervals (SAVE, BGSAVE). |
  | | Fast, compact (binary dump). |
  | | Risk: Data between snapshots may be lost. |
  | AOF (Append Only File) | Every write is logged. |
  | | Replays log at restart. |
  | | Can appendfsync=always / everysec / no. |
  | | Slower than RDB, but safer. |
  | Hybrid | RDB + AOF together for balance. |

6. Replication  
  a. Leader-Follower (Master-Slave) replication.  
  b. Followers do async replication.  
  - c. Replication is used for:  
    - 1. High availability  
    - 2. Read scaling  
    - 3. Failover setups  

7. High Availability & Clustering  
  a. Sentinel: Monitors Redis masters & auto-promotes a replica if master fails.  
  - b. Cluster Mode:  
    - 1. Data sharded across nodes.  
    - 2. Uses hash slots (0–16383) distributed among masters.  
    - 3. Provides auto-rebalancing and failover.  

8. Memory Management  
  - a. Redis is RAM-first, so memory handling is critical:  
    - 1. Uses jemalloc allocator.  
    - 2. Supports maxmemory policies:  
    - 3. noeviction → reject writes when full.  
    - 4. allkeys-lru → evict least recently used keys.  
    - 5. volatile-ttl → evict expiring keys first.  
    - 6. allkeys-random → evict random keys.  
  - b. Expiration & TTL:  
    - 1. Keys can have expiry (EXPIRE, SETEX).  
    - 2. Expiry is handled lazily + periodic scan.  

9. Pub/Sub & Streams  
  a. Pub/Sub → lightweight message passing system.  
```java
SUBSCRIBE news
PUBLISH news "Redis 7 Released!"
```

10. Streams (Redis ≥5.0):  
  a. Append-only log for event sourcing, Kafka-like.

11. Internal Workings Example  
  a. Let’s walk through SET key value:  
  b. Client sends SET mykey "hello".  
  c. Redis parses command using its RESP (REdis Serialization Protocol) parser.  
  d. Event loop picks request from client socket.  
  e. Redis checks memory, allocates an SDS string for "hello".  
  f. Stores entry in dict (hashtable):  
  g. mykey → SDS("hello")  
  h. Reply +OK to client.  
  i. If AOF is enabled: Appends SET mykey "hello" to AOF file asynchronously.  
  j. If RDB snapshotting is enabled, nothing happens immediately (unless save point reached).  

12. Why Redis is So Fast?  
  a. In-memory (RAM-based).  
  b. Single-threaded → no locking overhead.  
  c. Efficient C data structures (SDS, ziplist, skiplist).  
  d. Optimized I/O using epoll/kqueue.  
  e. Pipelining: Multiple requests in one round-trip.  

13. Typical Use Cases  
  a. Caching layer in front of databases.  
  b. Session store in web apps.  
  c. Leaderboards (sorted sets).  
  d. Queues (lists, streams).  
  e. Rate limiting (INCR with expiry).  
  f. Real-time counters/analytics.  

<br />
<br />
<br />




# 2. JedisConnectionFactory vs LettuceConnectionFactory

1. Background  
  a. Both are Spring Data Redis connection factories (implementing RedisConnectionFactory).  
  b. They provide low-level connections to Redis.  
  c. The difference lies in the underlying Redis client library:  
  d. Jedis → Old, synchronous, thread-unsafe (per-connection model).  
  e. Lettuce → Modern, asynchronous, thread-safe (netty-based).  

2. JedisConnectionFactory  
  a. Uses Jedis client internally.  
  b. Blocking / synchronous API → each command blocks until result arrives.  

3. Connection model:  
  a. Each thread needs its own connection.  
  b. Uses connection pool (Commons Pool2).  
  c. Pooling is mandatory for scalability.  
  d. Thread-safety: Jedis connections are not thread-safe.  
  e. Performance: Good for small workloads, but limited under high concurrency due to pool contention.  
  f. Cluster support: Added later, not as smooth as Lettuce.  
  g. Pub/Sub: Requires dedicated connections (one per subscriber).  

4. Pros:  
  a. Simple, stable, widely used for years.  
  b. Easy to debug.  

5. Cons:  
  a. Pool management overhead.  
  b. Blocking model only.  
  c. Weak cluster and async support.  

6. LettuceConnectionFactory  
  a. Uses Lettuce client internally.  
  b. Built on Netty → asynchronous & event-driven.  

7. Connection model:  
  a. A single connection can be shared by multiple threads.  
  b. No pooling needed (thread-safe).  
  c. Much more scalable under high concurrency.  
  d. Thread-safety: Safe across multiple threads.  
  e. Performance: Typically faster than Jedis under load.  
  f. Cluster support: Native, first-class, very stable.  
  g. Pub/Sub: More efficient; can multiplex connections.  
  h. Reactive support: Works with Spring WebFlux (non-blocking).  

8. Pros:  
  a. Thread-safe, no pool overhead.  
  b. Async & reactive APIs.  
  c. Strong cluster support.  
  d. Better performance for high-concurrency apps.  

9. Cons:  
  a. Slightly higher memory footprint (because Netty).  
  b. More complex for debugging low-level netty issues.  

| Feature                 | JedisConnectionFactory       | LettuceConnectionFactory       |
| ----------------------- | ---------------------------- | ------------------------------ |
| **Client**              | Jedis (blocking)             | Lettuce (Netty, async)         |
| **Thread Safety**       | No (needs pool per thread)   | Yes (multi-threaded, no pool)  |
| **Connection Pooling**  | Required                     | Not required                   |
| **API Style**           | Synchronous only             | Synchronous + Async + Reactive |
| **Performance**         | Lower under high concurrency | Better scalability             |
| **Cluster Support**     | Basic, limited               | Native, robust                 |
| **Pub/Sub**             | Dedicated connections needed | Multiplexed connections        |
| **Spring Boot Default** |  (deprecated)                |  (default since Spring 2.0+)   |

10. Current Best Practice  
  a. Spring Boot 2.x+ → Lettuce is the default, Jedis is optional.  

11. Use LettuceConnectionFactory unless:  
  a. You’re maintaining legacy code with Jedis.  
  b. Or you specifically need Jedis-specific behavior (rare).  
  c. Example config with Lettuce:  

```java
@Configuration
public class RedisConfig {
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory("localhost", 6379);
    }
}
```

12. Conclusion:  
For new projects → always prefer LettuceConnectionFactory.  
For legacy apps with Jedis → fine to use, but consider migrating for performance, scalability, and reactive support.  

<br />
<br />
<br />





# 3. Jedis vs Lettuce in real application issue

1. What Happened  
Spring Boot pod restarted → all beans (including JedisConnectionFactory) were reinitialized.  
But the Redis server connection wasn’t being restored when you used JedisConnectionFactory, while it was automatically recovered when you used LettuceConnectionFactory.  

2.  Why Jedis Fails to Reconnect  
  - a. Jedis is synchronous + blocking.  
    - 1. It opens a raw TCP socket to Redis.  
    - 2. If that connection breaks (because the pod restarted, or Redis restarted, or network blipped), the Jedis connection is dead.  
  - b. Jedis does not auto-reconnect.  
    - 1. Once the socket is closed, the connection object is useless.  
    - 2. You need to manually recreate connections (or rely on a connection pool manager to do it).  
  - c. JedisConnectionFactory in Spring Data Redis  
    - 1. Wraps a Jedis pool (Commons Pool2).  
    - 2. When the connection is broken, Jedis doesn’t retry or reestablish automatically — the pool just hands out broken connections until you restart the app or pool is refreshed.  
  - d. Result: When the pod came back up, Jedis still had stale/broken connections in the pool, so your app couldn’t talk to Redis.  

3. Why Lettuce Works  
  - a. Lettuce is built on Netty (asynchronous, event-driven I/O).  
    - 1. Connections are stateful but resilient.  
    - 2. If the TCP socket drops, Lettuce will attempt reconnects automatically (configurable).  
  - b. Connection re-establishment logic:  
    - 1. Lettuce monitors I/O channels and transparently reconnects when Redis comes back online.  
    - 2. Even in cluster mode, Lettuce can auto-refresh topology and recover nodes.  
  - c. Thread-safe single connection:  
    - 1. No pool handing out broken connections.  
    - 2. One LettuceConnection can be shared safely across multiple threads, and when it detects disconnection → it reestablishes.  
  - d. Result: When the pod restarted, Lettuce automatically reconnected to Redis, so your app kept working without manual intervention.  

4. Summary  
  a. JedisConnectionFactory → No auto-reconnect. Connections remain broken unless explicitly recreated. After pod restart, your app had dangling sockets.  
  b. LettuceConnectionFactory → Has built-in auto-reconnect via Netty event loop. So when Redis became available again, your app’s Redis connection was transparently reestablished.  

5. Best Practice  
  a. For cloud-native / containerized environments (Kubernetes, pods, ephemeral restarts) → Always use Lettuce because it’s designed to handle transient network failures gracefully.  
  b. Jedis might still be fine for small legacy apps running on stable VMs with no frequent restarts.  

6. Links to read  
  a. [Why is Redis So Fast Despite Being Single-Threaded?][REDIS_SINGLE_THREADED_LINK]  
  b. [What is Redis?][WHAT_IS_REDIS_LINK]  


<br />
<br />
<br />



[REDIS_SINGLE_THREADED_LINK]: https://medium.com/@aditimishra_541/why-is-redis-so-fast-despite-being-single-threaded-dc06ba33fc75
[WHAT_IS_REDIS_LINK]: https://medium.com/@humberto521336/what-is-redis-94c1930bd3c3
