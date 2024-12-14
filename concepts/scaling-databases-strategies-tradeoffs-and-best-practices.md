# Scaling Databases: Strategies, Tradeoffs, and Best Practices

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

**Single-Leader Replication**: All writes occur on one leader node, replicated to follower nodes.

* **Pros**: Simple to implement; scales reads effectively.
* **Cons**: Limited by single-node write throughput; eventual consistency.



<figure><img src="../.gitbook/assets/Screenshot 2024-12-12 at 19.29.12.png" alt=""><figcaption><p>single-leader replication</p></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2024-12-12 at 19.22.12.png" alt=""><figcaption><p>Multi-level replication</p></figcaption></figure>

**Multi-Leader Replication**: Multiple nodes act as leaders to handle writes.

* **Challenges**: Race conditions and ordering conflicts must be resolved.

**Leaderless Replication**: All nodes are equal, and writes are accepted on any node.

* **Quorum** ensures consistency by requiring agreement from a subset of nodes.

**HDFS Replication**: Used in distributed systems like Hadoop. Data is sharded across nodes for fault tolerance.

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

**Single-Tier Aggregation**: A single layer of hosts aggregates events before writing to the database.

<figure><img src="../.gitbook/assets/Screenshot 2024-12-14 at 14.32.29.png" alt=""><figcaption></figcaption></figure>

**Multi-Tier Aggregation**: Multiple layers progressively aggregate events.

<figure><img src="../.gitbook/assets/Screenshot 2024-12-14 at 14.38.15.png" alt=""><figcaption></figcaption></figure>

**Tradeoffs**: Eventual consistency and added latency due to the aggregation process.

***

### Handling Large Key Spaces Effectively

In real-world implementations, your key space will be far more complex than a simple A-Z range. The primary goal is to prevent memory overflow in successive aggregation levels. This requires a strategic approach to distributing and limiting key spaces across different host levels.

Consider a multi-tiered aggregation service where hosts in earlier levels must carefully manage their key space. A practical strategy is to intentionally limit each level's key space to less than its full memory capacity. This approach ensures that subsequent aggregation levels have sufficient memory to accommodate all incoming keys.

For instance, in a two-level aggregation setup:

* First-level hosts should use only half their available memory
* This reservation allows the second-level host to successfully aggregate keys from all first-level hosts

You can also optimize this further by provisioning hosts with varying memory configurations:

* Earlier aggregation levels: Hosts with less memory
* Later aggregation levels: Hosts with more memory

#### Addressing Fault Tolerance Challenges

One of the most significant risks in event aggregation systems is the potential for cascading failures. When a single host goes down, it doesn't just lose its own aggregated events—it can trigger a domino effect that causes data loss across multiple levels.

Traditional solutions like checkpointing and dead letter queues can help, but they're not perfect. Repeating processing for a large number of hosts can waste computational resources and introduce significant latency.

### A Robust Solution: Stateless Service Clusters

A more elegant approach is to convert each aggregation node into an independent service with multiple stateless nodes. Here's how this might look:

* Replace individual nodes with service clusters
* Use a shared in-memory database like Redis for coordination
* Implement load balancing across multiple hosts
* Maintain a small number of hosts (e.g., three) for fault tolerance

This design offers several advantages:

* Improved system resilience
* Reduced risk of complete data loss
* Ability to handle host failures gracefully

***

### Batch vs. Streaming: Choosing the Right Approach

ETL is fundamentally about moving and transforming data between systems, but the approach can vary significantly. Two primary paradigms emerge:

### Batch Processing

* Periodic data processing at fixed intervals
* Ideal for scheduled tasks like monthly billing generation
* Works best when:
  * Data is only available at specific times
  * Large volumes of data need comprehensive processing

### Streaming Processing

* Continuous, real-time data flow
* Processes data immediately as it becomes available
* Advantages include:
  * Near-instantaneous updates
  * Distributed processing costs
  * Easier debugging of smaller data chunks

#### The Messaging Ecosystem: Key Concepts

Understanding messaging systems is critical for building responsive, decoupled architectures:

### Core Terminology

* **Messaging System**: A framework for transferring data between applications
* **Message Queue**: A waiting area for work instructions
* **Producer/Consumer**: An asynchronous system where event producers are separated from event processors
* **Message Broker**: A translation layer between different messaging protocols

### Communication Strategies: Pull vs. Push

The debate between pull and push mechanisms is nuanced:

**Pull (Recommended in Most Cases)**

* Consumer controls message consumption rate
* Prevents system overload
* Provides better load balancing
* Ideal for most enterprise applications

**Push (Specific Use Cases)**

* Better for live streaming applications
* Useful when collecting data from multiple sources
* Provides more timely updates

#### Tools of the Trade

Different tools serve different ETL and streaming needs:

**Batch Processing Tools**

* Airflow
* Luigi

**Streaming Tools**

* Kafka
* Flink
* Flume (specialized for logging)

#### Architectural Patterns: Lambda vs. Kappa

Two prominent architectures for handling complex data processing:

### Lambda Architecture

* Runs parallel batch and streaming pipelines
* Fast pipeline: Lower accuracy, higher speed
* Slow pipeline: High consistency, lower speed
* Uses techniques like:
  * Approximation algorithms
  * In-memory databases
  * Varied replication strategies

### Kappa Architecture

* Single technology stack for batch and streaming
* Uses an append-only log (like Kafka)
* Simplified compared to Lambda
* Processes all data as a continuous stream

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
