# Designing Data-Intensive Applications

---

https://dataintensive.net/

---

## Foundations of Data Systems

---

### Reliable, Scalable, and Maintainable Applications

---

- functional requirements
    - store, retrieve, search, process
- nonfunctional requirements
    - security, reliability, compliance, scalability, compatibility, maintainability

---

- Reliability means making systems work correctly, even when faults occur
    - faults: hardware, software, human
- Scalability means having strategies for keeping performance good, even when load increases
    - we first need ways of describing load and performance quantitatively
    - percentiles(p50,p95,p99,p999) are open used in SLO and SLA
    - measure response times on the client side
    - scaling up (vertical) / scaling out (horizontal), shared-nothing architecture
    - elastic: automatically add computing resources when they detect a load increase
- Maintainability (in essence) is about making life better for the engineering and operations teams who need to work with the system
    - good abstractions can help reduce complexity and make the system easier to modify and adapt for new use cases
    - good operability means having good visibility into the system’s health, and having effective ways of managing it

---


### Data Models and Query Languages

---

由于不同的数据关系，发展出了不同的存储模型

- one-to-many
    - hierarchical model
    - document databases
- many-to-one, many-to-many
    - network model
    - relational model
- many-to-many
    - graph model
        - property graph model
        - triple-store model

---

> If the data in your application has a document-like structure, then it’s probably a good idea to use a document model.
> if your application does use many-to-many relationships, the document model becomes less appealing.
> For highly interconnected data, the document model is awkward, the relational model is acceptable, and graph models are the most natural.

---

> Document databases target use cases where data comes in self-contained documents and relationships between one document and another are rare.
> Graph databases target use cases where anything is potentially related to everything.

---

### Storage and Retrieval

---

这一章是介绍存储引擎。
数据间是什么关系，数据被怎么使用，决定了使用什么数据库。
（虽然这种话是老生常谈，但是通过书里的介绍，对如何选择能带来不少帮助

---

这个章节主要介绍两类引擎

- log-structured storage engines
- page-oriented storage engines

---

> well-chosen indexes speed up read queries, but every index slows down writes.

这里介绍的一种策略是用 hash 做索引
- 写入都用 append，然后在 hash 里记录最后一次写入的位置（Bitcask 使用了这种策略）
    - 用于 key-value 的数据，hash 里面是 key-location
- 为了避免浪费大量空间，定期进行 compaction（删除被覆盖了的记录
    - 一开始写入数据时划分成一个个 segment
    - 因为写入都是 append，所以可以在不影响读写的情况下进行 compaction
    - 将旧的 segment 压缩完之后进行替换，整个过程对读写都不可见
- 一些实现上要关注的点
    - File format
    - Deleting records
    - Crash recovery
    - Partially written records
    - Concurrency control
- 优势：append 的策略是顺序读写，更快；并发和错误恢复更简单；通过合并能减少碎片
- 不足：要把整个 hash 装入内存来加速；不支持 range 的查询

---

- SSTables
    - 在内存里用 balanced tree 来存储数据
    - 内存使用增长后，将有序的数据存储到硬盘
    - 为了避免出现数据丢失，同时将一份未排序的数据直接写入硬盘
- LSM-trees are typically faster for writes, B-trees are faster for reads

---

- online transaction processing, OLTP
    - high availability
    - low latency
- online analytic processing, OLAP
    - data warehouse
    - ETL (extract from OLTP, transform, load into data warehouse

---

### Encoding and Evolution

---

- backword compatibility
- forward compatibility

---

- encoding
    - JSON, XML
    - Thrift, Protocol Buffers, Avro
        - schema
- dataflow
    - database
    - RPC, REST
    - message passing

---

## Distributed Data

---

- scalability
- fault tolerance / high availability
- latency

---

- shared-memory architecture
- shared-disk architecture
- shared-nothing architecture
    - horizontal scaling
    - scaling out

---

- replication
- partitioning

---

### Replication

---

- replicate data
    - reduce latency
    - increase availability
    - increase read throughput

---

> all of the difficulty in replication lies in handling changes to replicated data.

- single-leader
- multi-leader
- leaderless

---

- leader-based relication
    - writes only accepted on the loader (followers are read-only
- synchronous vs asynchronous
    - semi-synchronous, one of the followers is synchronous, and the others are asynchronous.
- how to setup new followers
- how to handle node outages
    - follower, catch-up recovery
    - leader, failover
- problems in distributed system
    - node failures
    - unreliable networks
    - trade-offs around replica consistency, durability, latency
- replication logs
- guarantee
    - strong consistency
    - eventual consistency
    - read-after-write consistency
    - monotonic read
    - comsistent prefix read

---

- multi-leader
    - allow more than one node to accept writes.
    - each leader simultaneously acts as a follower to the other leaders.
    - pros: performance, tolerance of datacenter outages, tolerance of network problems
    - cons: the same data may be concurrently modified in two different datacenters
        - those write conflicts must be resolved
- use case
    - multi-datacenter operation
    - clients with offline operation
        - each device is a datacenter, and the network connection between them is extermely unreliable
    - collaborative editing
- handle write conflicts
    - ensure that requests from a user are always routed to the same datacenter
    - database must resolve the conflit ina convergent way.
        - all replicas must arrive at the same final value when all changes have been replicated
    - custom conflict resolution logic
- replication topology
    - all-to-all
    - circular
    - star

---

