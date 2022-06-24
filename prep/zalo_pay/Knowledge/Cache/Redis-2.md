# Caching

## 1. Memcached vs Redis

Redis is better:

- More data structure type (SET, HASH, STRING, LIST, ... )

- Support Data persistence

- Support horizontal scaling

## 2. Redis Sentinel vs Redis cluster

Redis sentinel:

- Using when there is one master node fail over, Redis sentinel support automatically back up.

- Redis sentinel: monitor, elect new node as master.

Redis cluster:

- Support sharding data, and master - replica configuration, when master node fail, elect secondary node as backup 

## 3. Redis Sentinel configuration

**<u>Cấu hình master slave redis:</u>**

Master:

```ini
bind 223.223.223.100 127.0.0.1
```

Slave:

```ini
slaveof 223.223.223.100 6379
```

**<u>Sentine Cơ chế hoạt động</u>**

- Các sentinel sẽ luôn quan sát master server, khi master sập, các sentinels sẽ loan truyền nhau 1 tín hiệu sdown: tao thấy đại ca chết rồi thì phải.
- Khi đủ 1 số lượng n sentinel đồng ý rằng tao cũng thấy master sập rồi, tụi sentinels sẽ loan tiếp tín hiệu odown: nó thực sự chết rồi đó.
- Lúc này, tụi sentinels sẽ bầu chọn ra 1 slave để nâng cấp lên làm master mới, đồng thời cập nhật các cấu hình theo bộ máy chính quyền mới.
- Khi thằng master kia sống lại, nó sẽ được tham gia vào băng nhóm với vai trò slave.

**<u>Master configuration:</u>**

```ini
bind 223.223.223.100
port 26379

sentinel monitor mymaster 223.223.223.100 6379 2
sentinel down-after-milliseconds mymaster 2000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
```

**<u>Slave configuration:</u>**

```ini
bind 223.223.223.101
port 26379

sentinel monitor mymaster 223.223.223.100 6379 2 // Here watching master node 2 is quorum
sentinel down-after-milliseconds mymaster 2000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
```

Quorum = định số tối thiểu.

- The **quorum** is the number of Sentinels that need to agree about the fact the master is not reachable, in order to really mark the master as failing, and eventually start a failover procedure if possible.
- However **the quorum is only used to detect the failure**. In order to actually perform a failover, one of the Sentinels need to be elected leader for the failover and be authorized to proceed. This only happens with the vote of the **majority of the Sentinel processes**.

Sentinel require to talk with the majority to be able to select leader.

When using Lettuce, you can interact with Redis Sentinel and Redis Sentinel-managed nodes in multiple ways:

1. [Direct connection to Redis Sentinel](https://github.com/lettuce-io/lettuce-core/wiki/Redis-Sentinel#direct-connection-redis-sentinel-nodes), for issuing Redis Sentinel commands

2. Using Redis Sentinel to [connect to a master](https://github.com/lettuce-io/lettuce-core/wiki/Redis-Sentinel#sentinel.redis-discovery-using-redis-sentinel)

3. Using Redis Sentinel to connect to master nodes and replicas through the [Master-Replica API](https://github.com/lettuce-io/lettuce-core/wiki/Master-Replica).

```java
RedisURI redisUri = RedisURI.Builder.sentinel("sentinelhost1", "mymaster") 
                                    .withSentinel("sentinelhost2")
                                    .build();
RedisClient client = RedisClient.create(redisUri);
RedisConnection<String, String> connection = client.connect(); 
```

```java
    public String get(String key){
        try {
            RedisClient client = RedisClient.create(redisURI);
            RedisAsyncCommands<String, String> commands = client.connect().async();
            return commands.get(key).get();
        } catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e.getMessage());
        }
    }
```

## 4. Redis Cluster

**<u>Redis Cluster data sharding</u>**

Redis Cluster does not use consistent hashing, but a different form of sharding where every key is conceptually part of what we call a **hash slot**.

There are 16384 hash slots in Redis Cluster, and to compute the hash slot for a given key, we simply take the CRC16 of the key modulo 16384.

Every node in a Redis Cluster is responsible for a subset of the hash slots, so, for example, you may have a cluster with 3 nodes, where:

- Node A contains hash slots from 0 to 5500.
- Node B contains hash slots from 5501 to 11000.
- Node C contains hash slots from 11001 to 16383.

This makes it easy to add and remove cluster nodes. For example, if I want to add a new node D, I need to move some hash slots from nodes A, B, C to D. Similarly, if I want to remove node A from the cluster, I can just move the hash slots served by A to B and C. Once node A is empty, I can remove it from the cluster completely.

Moving hash slots from a node to another does not require stopping any operations; therefore, adding and removing nodes, or changing the percentage of hash slots held by a node, requires no downtime.

**<u>Example configuration:</u>**

In our example cluster with nodes A, B, C, if node B fails the cluster is not able to continue, since we no longer have a way to serve hash slots in the range 5501-11000.

However, when the cluster is created (or at a later time), we add a replica node to every master, so that the final cluster is composed of A, B, C that are master nodes, and A1, B1, C1 that are replica nodes. This way, the system can continue if node B fails.

Node B1 replicates B, and B fails, the cluster will promote node B1 as the new master and will continue to operate correctly.

However, note that if nodes B and B1 fail at the same time, Redis Cluster will not be able to continue to operate.

```java
RedisURI redisUri = RedisURI.Builder.redis("localhost")
  .withPassword("authentication").build(); 

RedisClusterClient clusterClient = RedisClusterClient
  .create(rediUri); 

StatefulRedisClusterConnection<String, String> connection
 = clusterClient.connect(); 

RedisAdvancedClusterCommands<String, String> syncCommands = connection
  .sync();
```

## 5. Redis Cluster

**<u>Cache policy (hot vs cold cache):</u>**

**Now, to your question:**

**Cold cache:** When the `cache` is empty or has irrelevant data, so that `CPU` needs to do a slower read from `main memory` for your program data requirement.

**Hot cache:** When the `cache` contains relevant data, and all the reads for your program are satisfied from the `cache` itself.

So, hot caches are desirable, cold caches are not.

<u>**Cache Penetration, Cache Breakdown, Cache Avalanche**</u>

- Cache penetration: Malicious user intentionally query invalid data, making web service query from both cache and database, degrad over performance. 
  
  - Solution: cache empty result, set low expire time. Perform validation before sending request to cache or database.

- **Cache Avalanche**: after fix time, all cache expire, create a huge workload to database.
  
  - Solution: add random number to expire time.



## 6. Redis store JSON

- Store JSON as string is inefficient.

- User should use RedisJSON



## 7. Redis performance

- Redis is single thread, this can minimize context switching each thread

- Redis is Key value memory database.



**<u>Reduce redis memory usage:</u>**

- Using 32 bit instance

- Using hash map to store object

- The maximum allowed key size is 512 MB.



**<u>What happens if Redis runs out of memory? </u>**

Redis has built-in protections allowing the users to set a max limit on memory usage, using the `maxmemory` option in the configuration file to put a limit to the memory Redis can use.



If this limit is reached, Redis will start to reply with an error to write commands (but will continue to accept read-only commands).



**<u>Redis eviction policy:</u>**

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-01-23-50-image.png)

**<u>Maximum number of keys:</u>**

Redis can handle up to 2^32 keys, and was tested in practice to handle at least 250 million keys per instance.



## 8. Redis sentinel

**<u>Fundamental things to know about Sentinel before deploying</u>**

1. You need at least three Sentinel instances for a robust deployment.
2. The three Sentinel instances should be placed into computers or virtual machines that are believed to fail in an independent way. So for example different physical servers or Virtual Machines executed on different availability zones.
3. Sentinel + Redis distributed system does not guarantee that acknowledged writes are retained during failures, since Redis uses asynchronous replication
4. You need Sentinel support in your clients. Popular client libraries have Sentinel support, but not all.

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-02-13-28-image.png)

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-02-14-54-image.png)

- NEVER SET UP ONLY MASTER AND SLAVE



![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-02-18-39-image.png)

- MAJORITY IS NEED TO ORDER A FAILOVER.



In every Sentinel setup, as Redis uses asynchronous replication, there is always the risk of losing some writes because a given acknowledged write may not be able to reach the replica which is promoted to master. However in the above setup there is a higher risk due to clients being partitioned away with an old master, like in the following picture:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-02-26-20-image.png)

Install sentinel on client:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-02-42-22-image.png)

Sentinel configuration:

![](C:\Users\duyng\AppData\Roaming\marktext\images\2022-06-20-04-29-37-image.png)
