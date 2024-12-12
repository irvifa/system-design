# Distributed Transactions

## **Creating Data Consistency Across Multiple Services**

In a distributed system, a single unit of work often involves multiple services, each requiring separate writes. Failures in any of these writes, whether due to bugs, network issues, or host failures, can cause data inconsistency.

**Goal:** Maintain atomicity and consistency across distributed services, ensuring that all operations succeed (commit) or fail (abort).

***

## **Techniques for Data Consistency**

**Event-Driven Architecture (EDA)**

**Definition:** EDA uses asynchronous, non-blocking communication where events are published to a log. Event consumers process these events to perform actions, promoting scalability, availability, and responsiveness.

**Benefits of EDA:**

1. **Loose Coupling:** Services don’t directly depend on each other.
2. **High Scalability:** Events can be processed later, avoiding traffic spikes.
3. **Low Latency:** Event publishing is lightweight and fast.

**Tradeoff:**

* May require validation upfront to avoid propagating invalid events.

***

## **Event Sourcing**

**Definition:** Event sourcing records all state-changing events in an append-only log. The log becomes the source of truth, and other systems derive their state from this log.

**Workflow:**

1. Events are published to an event log.
2. Subscribers consume and process these events asynchronously.

**Advantages:**

* Provides a complete audit trail for debugging and analytics.
* Allows replaying events to derive past states.
* Facilitates business logic changes without affecting existing data.

**Challenges:**

* Requires more storage and event versioning.
* Replay becomes costly as logs grow.

**What if a subscriber fails?**

* Event logs ensure events can be reprocessed after recovery.

***

## **Change Data Capture (CDC)**

**Definition:** CDC monitors changes in a database and publishes these changes as events to a change log, which downstream services can consume.

**Workflow:**

1. Changes are written to a database.
2. A transaction log miner detects and publishes these changes as events.
3. Downstream systems consume and process these events.

**Advantages:**

* Near real-time data synchronization.
* Reduces latency compared to event sourcing.

**Example Tools:**

* **Debezium**: Open-source CDC platform
* **DynamoDB Streams**: AWS service for database change streaming

**Challenges:**

* May generate duplicate events, requiring idempotent processing or exactly-once guarantees.

***

## **Saga Pattern**

A saga is a long-lived distributed transaction that consists of multiple steps (transactions). If any step fails, compensating transactions are executed to roll back changes.

**Choreography vs. Orchestration**

| Feature                     | Choreography                            | Orchestration                            |
| --------------------------- | --------------------------------------- | ---------------------------------------- |
| **Coordination**            | Services communicate via events         | Orchestrator coordinates all steps       |
| **Flow**                    | Parallel requests                       | Linear, step-by-step requests            |
| **Complexity**              | Harder to maintain as services increase | Easier to understand with a central flow |
| **Performance**             | Lower latency due to parallel execution | Higher latency due to linear execution   |
| **Failure Handling**        | Compensating transactions in services   | Orchestrator triggers compensating steps |
| **Service Independence**    | Tight coupling between services         | Services are decoupled and independent   |
| **Single Point of Failure** | None                                    | Orchestrator must be highly available    |

**Example: Booking a Tour Package**

1. **Choreography**:
   * Services directly produce/consume events via Kafka.
   * Parallel execution minimizes latency.
   * Complex and difficult to debug.
2. **Orchestration**:
   * A central orchestrator manages steps sequentially.
   * Easier to understand but introduces higher latency.

***

## **Transaction Supervisor**

A transaction supervisor ensures transactions complete successfully or are compensated. It periodically synchronizes destinations in case of failures and can be implemented as:

* Manual compensating transactions.
* Automated compensations (after extensive testing).

**Key Consideration:** Always log compensating transactions for traceability.

***

####
