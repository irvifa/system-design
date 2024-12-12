# Scaling Databases: Strategies, Tradeoffs, and Best Practices1

We'll explore essential strategies and concepts in scaling databases, delving into the intricacies of various storage services, database replication techniques for scalability and reliability, event aggregation to minimize writes, the nuanced differences between normalization and denormalization, and the strategic implementation of query caching to enhance system performance. Through a comprehensive examination of these topics, we'll uncover their unique tradeoffs and provide guidance on selecting the most appropriate approaches for effective system design.

***

## Prelude on Storage Services

Storage services are **stateful** by nature. Unlike stateless services, stateful services require mechanisms for consistency and redundancy to avoid data loss. Designing stateful systems involves making decisions around:

* **Consistency**: Whether data changes are visible to all nodes immediately (strong consistency) or eventually (eventual consistency).
* **Redundancy**: Strategies like replication and failover to prevent data loss.

### **Why Keep Services Stateless?**

1. **Simplicity**: Avoid implementing sticky sessions or complex failover logic.
2. **Separation of Concerns**: Push all state to stateful storage services like databases, which are better suited for managing data at scale.

### **Categories of Storage Services**

**Databases**

* **SQL**: Relational databases that enforce ACID properties.
* **NoSQL**:
  * **Column-Oriented**: Efficient for filtering large datasets (e.g., Cassandra, HBase).
  * **Key-Value**: High-performance data retrieval (e.g., Redis, Memcached).
  * **Document**: Flexible formats like JSON (e.g., MongoDB).
  * **Graph**: Ideal for relationship-heavy data (e.g., Neo4j, Amazon Neptune).

1. **File Storage**: Data organized into files and directories.
2. **Block Storage**: Fixed-size data chunks for low-level systems.
3. **Object Storage**: Suited for static data with a flat hierarchy (e.g., AWS S3).

***

### When to Use Databases

Choosing between a database and alternatives like file, block, or object storage often depends on the following:

* **Size of Objects**: Small objects (<256KB) suit databases; larger objects (>1MB) suit filesystems.
* **Performance Needs**: Databases load objects into memory, which can be inefficient for streaming large files.
* **Replication**: Large objects slow down database replication processes.

> _Tip_: A Microsoft study suggests files larger than 1MB are best stored in filesystems for efficiency.

***

### Database Replication

Database replication involves copying data across nodes to improve scalability and fault tolerance. **Replication** techniques include:

1. **Single-Leader Replication**: All writes occur on one leader node, replicated to follower nodes.
   * **Pros**: Simple to implement; scales reads effectively.
   * **Cons**: Limited by single-node write throughput; eventual consistency.
2. **Multi-Leader Replication**: Multiple nodes act as leaders to handle writes.
   * **Challenges**: Race conditions and ordering conflicts must be resolved.
3. **Leaderless Replication**: All nodes are equal, and writes are accepted on any node.
   * **Quorum** ensures consistency by requiring agreement from a subset of nodes.
4. **HDFS Replication**: Used in distributed systems like Hadoop. Data is sharded across nodes for fault tolerance.

***

### Scaling Storage with Sharding

If the size of a database exceeds the capacity of a single node, **sharding** becomes necessary. Sharding divides data into subsets distributed across multiple nodes.

**Benefits of Sharding**:

* Increased storage capacity
* Improved query performance through parallel processing
* Reduced memory overhead

**Challenges**:

* Complex management of shard locations and partitioning logic.

***

### Aggregating Events to Minimize Writes

Database writes are expensive and challenging to scale. Techniques like **aggregation** reduce the number of writes by combining events:

1. **Single-Tier Aggregation**: A single layer of hosts aggregates events before writing to the database.
2. **Multi-Tier Aggregation**: Multiple layers progressively aggregate events.

**Tradeoffs**: Eventual consistency and added latency due to the aggregation process.

***

### Normalization vs. Denormalization

* **Normalization**: Organizes data to avoid redundancy and ensure consistency. Best suited for small databases where consistency is critical.
* **Denormalization**: Optimizes for read performance by duplicating data and reducing JOIN operations. Common in large-scale systems.

**Tradeoffs**:

* **Normalization**: Faster writes, smaller database size.
* **Denormalization**: Faster reads, larger storage footprint.

***

### Caching for Performance

Caching reduces average query latency by storing frequent or recent queries in memory. **Caching Strategies** include:

1. **Read Strategies**
   * **Cache-Aside**: Data is loaded into the cache only on a miss (lazy loading).
   * **Read-Through**: Cache automatically fetches data from the database on a miss.
2. **Write Strategies**
   * **Write-Through**: Writes go through the cache to the database (consistent but slower).
   * **Write-Back**: Writes are deferred to the database, reducing latency but risking data loss.

**Why Separate Caching Services?**

* Stateless services benefit from independent caching for better performance and scalability.
* Prevents cache loss during deployments.

***

### Cache Invalidation

Cache invalidation ensures that stale data is removed or updated. Techniques include:

* **Max-Age**: Set a time-to-live (TTL) for cache entries.
* **Fingerprinting**: Use versioned file names or query hashes to force updates.

***

### Further Reading

* _Designing Data-Intensive Applications_ by Martin Kleppmann
* _Web Scalability for Startup Engineers_ by Artur Ejsmont
* _Cloud Native Patterns_ by Cornelia Davis

***
