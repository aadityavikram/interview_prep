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
<br />
<br />
<br />



# 4. Bloom Filter

1. A Bloom filter is a space-efficient probabilistic data structure used to test whether an element is a member of a set. It operates on a bit array and a set of hash functions.  

2. Initialization:  
  a. A Bloom filter is initialized as a bit array of a fixed size, where all bits are set to 0. It also uses a predetermined number of independent hash functions.  

3. Adding an Element:  
  a. To add an element to the set, the element is passed through each of the k hash functions.  
  b. Each hash function produces an index within the bit array.  
  c. The bits at these k calculated indices in the bit array are then set to 1.  

4. Checking for Membership:  
  a. To check if an element is present in the set, the same k hash functions are applied to the element.  
  b. The bits at the resulting k indices in the bit array are then examined.  
  c. If all of these k bits are set to 1, the Bloom filter indicates that the element might be in the set (a "possibly in the set" result).  
  d. If any of these k bits is found to be 0, the Bloom filter definitively states that the element is not in the set (a "definitely not in the set" result).  

5. Probabilistic Nature:  
  a. Bloom filters can produce false positives (claiming an element is in the set when it's not), but never false negatives (claiming an element is not in the set when it actually is). The probability of false positives increases with the number of elements added to the filter and decreases with a larger bit array and more hash functions.  

6. Space Efficiency:  
  a. They are highly space-efficient as they do not store the actual elements, only a compact representation in a bit array.  

7. No Deletion:  
  a. Elements cannot be reliably removed from a Bloom filter without potentially affecting the membership status of other elements, as removing a bit might incorrectly indicate the absence of another element that also hashed to that bit.  

<br />
<br />
<br />




# 5. How to setup Redis in a distributed environment?

1. Single Redis Server (Baseline)  
  a. Easiest setup → one Redis server (or container).  
  b. All pods connect to the same Redis instance.  

```java
spring:
  data:
    redis:
      host: redis-service   # K8s service for Redis
      port: 6379
```

2. Problems in distributed env:  
  a. Single point of failure.  
  b. Memory limited to one machine.  
  c. No scaling beyond one node.  

3. Redis with Replication (Leader–Follower)  
  a. One master (accepts writes), multiple replicas (read-only).  
  b. Follower auto-syncs from master.  
  - c. Pods can:  
    - 1. Send writes → master  
    - 2. Send reads → followers (manually or via proxy)  
  d. High availability (if master dies, a replica can take over manually).  
  e. Failover is manual unless you add Sentinel.  

4. Redis Sentinel (High Availability)  
  a. Sentinel = monitoring + auto-failover layer.  
  b. Deployed alongside master + replicas.  
  - c. If master goes down:  
    - 1. Sentinel elects a new master.  
    - 2. Clients auto-discover new master.  
  d. Topology:  

```java
           ┌─────────┐
  Writes → │ Master  │
           └─────────┘
             /     \
            /       \
       ┌───────┐ ┌───────┐
       │Replica│ │Replica│
       └───────┘ └───────┘
          ↑           ↑
          |           |
      ┌───────────────┐
      │  Sentinels    │
      └───────────────┘
```

Spring Boot config for Sentinel:

```java
spring:
  data:
    redis:
      sentinel:
        master: mymaster
        nodes:
          - redis-sentinel-0:26379
          - redis-sentinel-1:26379
          - redis-sentinel-2:26379
```

Provides automatic failover.  
Still a single master (no write scaling).  

5. Redis Cluster (Sharding + HA)  
  a. Data is sharded across multiple masters.  
  b. Each master has replicas.  
  c. Cluster uses hash slots (0–16383) to distribute keys.  
  d. Clients (like Lettuce, Redisson) understand cluster topology.  
  e. Topology:  

```java
 Master1(0-5460)  Master2(5461-10922)  Master3(10923-16383)
    | Replica1       | Replica2            | Replica3
```

Spring Boot config for Cluster:
```java
spring:
  data:
    redis:
      cluster:
        nodes:
          - redis-cluster-0:6379
          - redis-cluster-1:6379
          - redis-cluster-2:6379
```

True distributed Redis (scales horizontally).  
Automatic failover (replicas promoted).  
Higher throughput (multiple masters).  
More operational complexity.  
Multi-key ops limited to same hash slot.  

6. Redis Enterprise / Managed Services  
  - a. If you don’t want to manage Sentinel/Cluster yourself:  
    - 1. Redis Enterprise (official commercial solution).  
    - 2. AWS ElastiCache (Redis).  
    - 3. Azure Cache for Redis.  
    - 4. GCP Memorystore.  
  b. Automatic scaling, HA, backups.  
  c. Costs more, vendor lock-in.  

7. In Kubernetes  
  - a. If you’re deploying on K8s:  
    - 1. Use Helm charts or Operators:  
      - a. Bitnami Redis Helm Chart  
      - b. Redis Operator  
      - c. Example (Redis Cluster via Bitnami):  
```java
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install redis-cluster bitnami/redis-cluster --set cluster.nodes=6
```

8. This deploys:  
  a. Multiple masters (clustered)  
  b. Replicas  
  c. Sentinel (optional)  
  d. Your Spring Boot apps just connect using the service DNS name.  

9. Best Practices for Distributed Environments  
  a. Use Lettuce (not Jedis) → auto-reconnect, cluster-aware, Sentinel-aware.  
  b. Externalize config → Use K8s ConfigMap/Secret for host/port/password.  
  c. Enable TLS + AUTH for multi-tenant / internet-facing Redis.  
  d. Use @Cacheable with RedisCacheManager for cache layer (don’t reinvent).  
  e. For distributed locks / rate-limiting → use Redisson (handles cluster correctly).  
  - f. Capacity planning:  
    - 1. Size memory for peak load.  
    - 2. Use eviction policies (allkeys-lru) if using as cache.  
  g. Monitor Redis with Prometheus + Grafana (exporter available).

10. Summary  
  a. Small team / simple setup → Single Redis + persistence.  
  b. Need HA → Redis + Sentinel.  
  c. Need HA + scalability → Redis Cluster.  
  d. Don’t want ops hassle → Redis Enterprise / Cloud provider.  
  e. K8s → Use Helm chart / Operator.  

<br />
<br />
<br />





# 6. Links to Read

1. [Why is Redis So Fast Despite Being Single-Threaded?][REDIS_SINGLE_THREADED_LINK]  
2. [What is Redis?][WHAT_IS_REDIS_LINK]  

[REDIS_SINGLE_THREADED_LINK]: https://medium.com/@aditimishra_541/why-is-redis-so-fast-despite-being-single-threaded-dc06ba33fc75
[WHAT_IS_REDIS_LINK]: https://medium.com/@humberto521336/what-is-redis-94c1930bd3c3
