# System Design 

## Good resource:  
1. https://github.com/donnemartin/system-design-primer
2. Youtube seriouse:
    1. [东欧哥们](https://www.youtube.com/watch?v=bUHFg8CZFws)
    2. [mock interview](https://www.youtube.com/watch?v=TUhbXHRLf0o)
3. 一亩三分地:
    1. [帖子1](https://www.1point3acres.com/bbs/forum.php?mod=viewthread&tid=771667&ctid=233194)
    2. [帖子2](https://www.1point3acres.com/bbs/thread-490412-1-1.html)
    3. [帖子3](https://www.1point3acres.com/bbs/thread-786280-1-1.html)

## 一些总结
1. Message Queue  
Message queues receive, hold, and deliver messages. If an operation is too slow to perform inline, you can use a message queue with the following workflow:  
    1. An application publishes a job to the queue, then notifies the user of job status
    1. A worker picks up the job from the queue, processes it, then signals the job is complete  
Pros:  
    1. can handle the work async
cons:  
    1. queue could be large, and slow the whole process. 解决方法，limit queue size, thereby maintaining a high throughput rate and good rasponse times for jobs already in the queue. On,ce fills up, client will get a service busy code and try again later.

2. Too many request, scalability
    1. 加 deuplicated database， clones， master and slave database. 
    2. sharding the database, do the join from the application code. 
    3. Add cache, go to the database only if there is no cache there. Redis can do several hundreds of thousands of read operations per second when being hosted on a standard server. Also writes, especially increments, are very, very fast. 
        1. cache 方法 1. cache by query, 只存result，不好，2. cache by object, 把query的object整个存下来，效率较高，比较好delete
        2. 试着async as much as possible


3. consistency
    1. Weak consistency After a write, reads may or may not see it. A best effort approach is taken.  
    This approach is seen in systems such as memcached. Weak consistency works well in real time use cases such as VoIP, video chat, and realtime multiplayer games. For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

    2. Eventual consistency After a write, reads will eventually see it (typically within milliseconds). Data is replicated asynchronously.  
    This approach is seen in systems such as DNS and email. Eventual consistency works well in highly available systems  
    3. Strong consistency After a write, reads will see it. Data is replicated synchronously.
    This approach is seen in file systems and RDBMSes. Strong consistency works well in systems that need transactions.

4. Fail-over
    1. active-passive. passive send heart beat to active worker, if heart beat interuppted, passive replace down active machine. problem: the passive could start cold, down-time depends on how long passive become hot
    2. active-active. both server are managing traffic, spreading the load between them. If servers are public facing, DNS need to know about IP for both servers. If server is internal facing, application logic would know for both servers.  

5. load balancer
    1. Round Robin, send traffic equally to server A, B and C
    2. Weight round Robin, send the traffic proportion to weight to each server. Could have a more complicated algorithm to compute weight. Weight example: cluster size,
    3. Latency-based
    4. Geolocation based

6. Microservice
    1. A microservice communicate via network, small size, could implemente using different language, independent.  
    is not a layer within a monolithic application (example, the web controller, or the backend-for-frontend).[7] Rather it is a self-contained piece of business functionality with clear interfaces, and may, through its own internal components, implement a layered architecture. From a strategy perspective, microservices architecture essentially follows the Unix philosophy of "Do one thing and do it well"
7. Zoo Keeper
    Coordinating distributed system, try to avoid the problem single point failure. 主要是用作consistency and partition. 
8. Kafka, Message Queue, lock database, through put
9. Database
    1. Master-slave replication: master-> read and write. slave-> only read
        1. disadvantage: additinal logic to promote slave into master.
        2. Need replication. potential loss data if master fails.
        3. lots of write, the replication can be blocked
        4. more slaves, more you have to replicate which leads to greater replication lag
        5. replication add more hardware and additional complexity
    2. Master-master replication: 2 master to read and write
        1. disadvantage: need a load balancer to determine with one to write
        2. loss consistency, need time to replicate the nodes
    3. sharding
        1. advantage, only need to read and write part of the traffic, less replication, more cache hits. no single central master, allowing you to write in parallel with increased throughput
        2. disadvantage, need to update business logic with shards
        3. power user in a shard, could result in frequent read/write in a shard
        4. rebalance add additional complexity. Sharding function based on consistent hashing can reduce the amount of data transfered.
    4. Denormalization 
        1. Denormalization attempts to improve read performance at the expense of some write performance. Add redundent data, but no need to do the data join
        2. disadvantage: data is duplicated
        3. heavy write load may perform wrose
    5. NoSQL
        1. Most NoSQL stores lack true ACID transactions and favor eventual consistency.
        2. NoSQL is a collection of data items represented in a key-value store, document store, wide column store, or a graph database.
        3. Key-value store database.
            1. disdavantage: no way to use SQL data type verify the data type
            2. No way to ensure attribute names are spelled consistently
            3. A document store is centered around documents (XML, JSON, binary, etc), where a document stores all information for a given object. Document stores provide APIs or a query language to query based on the internal structure of the document itself.  
            4. Wide column store: A wide column store's basic unit of data is a column (name/value pair). A column can be grouped in column families (analogous to a SQL table). Super column families further group column families. You can access each column independently with a row key, and columns with the same row key form a row. 
    6. Cache
        1. Cache-aside: Look for entry in cache, resulting in a cache miss, Load entry from the database, Add entry to cache. return entry
            1. cache-aside means lazy loading
            2. disadvantage: Each cache miss results in three trips, which can cause a noticeable delay.
            3. Data can become stale if it is updated in the database.  solve by write through cache
            4. When a node fails, it is replaced by a new, empty node, increasing latency.
        2. Write through cache:
            1. Application adds/updates entry in cache
            2. Cache synchronously writes entry to data store
            3. advantage: the data read is up to date
            3. disadvantage: slow due to write operation, but subsequent reads is fast.User more tolarent latency when updating the data
            4. Most data might never be read
        3. Write behind:
            1. update cache, then async write entry to data store
            2. disadvantage: could be data loss if cache goes down
        4. refresh-ahead
            1. predict which one will use, and the refresh
            2. disadvantage:need accurate prediction
        5. overall disadvantage:
            1. Need to maintain consistency between caches and source of truth
            2. Need cache invalidation
            3. need to add redis and other logic
    7. HTTP, TCP
        1. HTTP : HTTP is a method for encoding and transporting data between a client and a server.
        2. TCP is a connection-oriented protocol over an IP network. Connection is established and terminated using a handshake. 
        3. RPC Remote procedure call: RPC is focused on exposing behaviors. a client causes a procedure to execute on a different address space, usually a remote server. RPCs are often used for performance reasons with internal communications, as you can hand-craft native calls to better fit your use cases.
            1. disadvantage: tightly coupled to service implementation
        4. REST API: REST is focused on exposing data. It minimizes the coupling between client/server and is often used for public HTTP APIs. REST uses a more generic and uniform method of exposing resources through URIs,
            1. disadvantage: too simple, need lots of parameters to represent a certain state