# System Design Interview Flow

**System design interviews** are a critical part of technical hiring for senior engineering roles. In this post, we’ll explore the key principles, structure, and topics you need to master to excel in a 1-hour system design interview. The discussion includes essential steps like clarifying requirements, API design, data modeling, failure design, and post-interview reflection.

***

### Key Steps in a System Design Interview

#### 1. Clarify Requirements and Discuss Tradeoffs

Start by clarifying **functional and non-functional requirements**, such as:

* **Functional**: User roles, API endpoints, and system behavior.
* **Non-functional**: Queries Per Second (QPS), P99 latency, scalability, and fault-tolerance.

**Approach:**

* Spend **≤10 minutes** to brainstorm and discuss requirements.
* Estimate numbers like daily active users and request rates.
* Ask about scalability: Should we start simple and scale up or design for scale immediately?
* Explore edge cases and ensure nothing is missed.

> _"Everything is a tradeoff. Improving one aspect (like latency) may increase cost or complexity."_

***

#### 2. Draft the API Specification

* Quickly sketch out **GET, POST, PUT, and DELETE endpoints**.
* Define the **request/response structure**.
* Mention endpoints like `/health` (for monitoring), `/signup`, and `/login` but keep it concise.

**Tip**: Propose an API draft **within 5 minutes** to leave room for deeper discussions.

***

#### 3. Design the Data Model

* Decide whether to design from scratch or use existing databases.
* Avoid shared databases across services to prevent schema conflicts and resource contention.
* **Key considerations**:
  * Table schema.
  * Data access patterns (CRUD operations).
  * Techniques for preventing concurrent update conflicts.

***

#### 4. Discuss Failures, Logging, and Monitoring

**Failure Design**

Explore graceful degradation and fault-tolerance:

* Silent failures, retry strategies, and monitoring for failures.

**Observability**

Logging, monitoring, and alerting ensure system health:

* **Golden Signals**:
  1. Latency
  2. Traffic, measured in HTTP requests per second
  3. Errors
  4. Saturation, depending on whether our system’s constraint is CPU, memory, or I/O, we can set utilization targets that should not be exceeded
* Use structured logs and standardized time formats. Avoid logging **sensitive information** (PII). Some useful things which can be tracked:
  * Contain a unique identifier to trace requests across services and share between users and developers.
  * Timestamps should use the same time zone and time format.
  * Should contain a unique identifier to trace requests across services .
  * Categorize log entries. Start with debug, info, and error.

**Tools**: Prometheus, Grafana, ELK (Elasticsearch, Logstash, Kibana).

***

#### 5. Explore Search Implementation

Search is a common requirement for most systems. Three techniques include:

1. **SQL + LIKE queries** (not scalable).
2. **Client-side libraries** (for small datasets).
3. **Search Engines like Elasticsearch** (recommended for PB-scale text data).

**Steps to Add Elasticsearch**

1. Create indices for the documents.
2. Use the Bulk API to keep indices updated.
3. Handle fuzzy searches using Elasticsearch's query DSL.

***

#### 6. Time Management and Driving the Discussion

* Focus on what matters most: requirements, API design, data model, and system components.
* Use diagrams (high-level and low-level) as visual aids.
* Keep the interviewer engaged by suggesting **directions** to explore further.

**Remember**: A 50-minute interview cannot cover everything. Communicate **breadth and depth** effectively.

***

### Post-Interview Reflection and Self-Assessment

#### Write a Reflection

Immediately after the interview:

1. **Document** everything:
   * What was asked.
   * What you said.
   * Diagrams drawn.
2. Assess **what went well** and what didn’t.

#### Analyze Your Performance

* Did you **miss any details**?
* Were there any gaps in **communication**?
* Were you able to manage **time effectively**?

#### Research and Improve

* Compare your design to existing resources:
  * High Scalability Blog
  * YouTube discussions
  * System design books
* Identify knowledge gaps and refine your approach.

***

### Interviewing the Company

System design interviews are **two-way**. You are also evaluating the company:

* What’s the **technology stack**?
* How competent is the **engineering leadership**?
* Are their development and CI/CD practices solid?
* What opportunities exist for **learning and growth**?

***

### Summary: Principles to Follow

1. **Clarify** requirements and tradeoffs early.
2. **Time-box** discussions: API specification, data model, failures, and observability.
3. Use **diagrams** to illustrate designs effectively.
4. Cover logging, monitoring, alerting, and observability.
5. Reflect and **self-assess** after interviews to improve.
6. Interview the company to ensure it aligns with your goals.

***

**Additional Resources**:

* [High Scalability Blog](http://highscalability.com)
* Google SRE Book: [Four Golden Signals](https://sre.google/sre-book/monitoring-distributed-systems/#xref_monitoring_golden-signals)
* [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
