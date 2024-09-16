# How to answer system design questions

Table of Contents

- [How to perform well](#how-to-perform-well)
  - [Ask refining questions](#ask-refining-questions)
  - [Handle data](#handle-data)
  - [Discuss the components](#discuss-the-components)
  - [Discuss trade-offs](#discuss-trade-offs)
- [Building blocks](#building-blocks)
  - [Domain Name System (DNS)](#domain-name-system-dns)
  - [Load Balancers](#load-balancers)
  - [Databases](#databases)
  - [Key-Value Store](#key-value-store)
  - [Content Delivery Network (CDN)](#content-delivery-network-cdn)
  - [Sequencer](#sequencer)
  - [Service Monitoring](#service-monitoring)
  - [Distributed Caching](#distributed-caching)
  - [Distributed Messaging Queue](#distributed-messaging-queue)
  - [Publish-Subscribe System](#publish-subscribe-system)
  - [Rate Limiter](#rate-limiter)
  - [Blob Store](#blob-store)
  - [Distributed Search](#distributed-search)
  - [Distributed Logging](#distributed-logging)
  - [Distributed Task Scheduling](#distributed-task-scheduling)
  - [Sharded Counters](#sharded-counters)

## How to perform well

### Ask refining questions

- Functional requirements: These represent the features a user of the designed system will be able to use. For example, the system will allow a user to search for content using the search bar.
- Non-functional requirements (NFRs): The non-functional requirements are criteria based on which the user of a system will consider the system usable. NFR may include requirements like high availability, low latency, scalability, and so on.

### Handle data

- What’s the size of the data right now?  
- At what rate is the data expected to grow over time?  
- How will the data be consumed by other subsystems or end users?  
- Is the data read-heavy or write-heavy?  
- Do we need strict consistency of data, or will eventual consistency work?  
- What’s the durability target of the data?  
- What privacy and regulatory requirements do we require for storing or transmitting user data?

### Discuss the components

- What components to use?  
- Where will they be placed?  
- How will they interact with each other?

### Discuss trade-offs

- Different components have different pros and cons. We’ll need to carefully weigh what works for us.  
- Different choices have different costs in terms of money and technical complexity. We need to efficiently utilize our resources.  
- Every design has its weaknesses. As designers, we should be aware of all of them, and we should have a follow-up plan to tackle them.

## Building blocks

### Domain Name System (DNS)

DNS translates human-readable domain names into machine-readable IP addresses. When designing DNS systems, considerations include:

- Hierarchy: The DNS hierarchy is organized in a tree-like structure with several levels:
  - Root Level: At the top is the root, represented as a dot (.). The root doesn’t have a specific domain name and is managed by a small number of root servers worldwide.
  - Top-Level Domains (TLDs): These are the next level in the hierarchy, divided into:
    - Generic TLDs (gTLDs): .com, .org, .net, etc.
    - Country-Code TLDs (ccTLDs): .uk (United Kingdom), .in (India), etc.
    - Sponsored TLDs (sTLDs): For specific communities or purposes like .edu (education), .gov (government), etc.
  - Second-Level Domains: These are directly below the TLDs. For example, in example.com, example is the second-level domain.
  - Subdomains: Below second-level domains, subdomains allow organizations to create their own internal structure. For instance, mail.example.com might be a subdomain for email services.
  - Authoritative DNS Servers: These servers hold the actual DNS records for domains (e.g., A, CNAME, MX records). Once a query reaches these servers, they provide the IP address or the information the client needs.
- Caching: Local and ISP caches improve lookup performance.
- Load distribution: DNS can distribute client requests across multiple servers (e.g., round-robin DNS).
- Security: DNSSEC (DNS Security Extensions) adds authentication to prevent DNS spoofing attacks.
- Scalability: The DNS system must scale as the number of domains and users grows.

---

### Load Balancers

Load balancers are essential for distributing client requests across multiple servers to improve performance, reliability, and scalability.

- Types of Load Balancers:
  - Layer 4 (Transport Layer): Operates at the transport layer (TCP/UDP). It forwards packets based on IP address and port without understanding the content.
  - Layer 7 (Application Layer): Operates at the application layer and can inspect the content of the requests (e.g., HTTP headers, URLs) before making routing decisions.
- Load Balancing Algorithms:
  - Round-Robin: Each request is sent to the next server in line. This is simple but may not consider the server's load.
  - Least Connections: The load balancer sends requests to the server with the fewest active connections, making it effective in environments where some requests are more resource-intensive than others.
  - IP Hash: The client’s IP address is hashed, and the result is used to assign the client to a particular server, providing session persistence across requests.
  - Weighted Round-Robin: Servers are assigned weights based on their capacity. A higher-capacity server will handle more requests compared to a lower-capacity one.
- Session Persistence (Sticky Sessions):
  - Some applications require that all requests from a single client are routed to the same server to maintain session state (e.g., user login).
  - Methods for Sticky Sessions:
    - IP Hash: Routing based on client IP.
      - Cookie-based: Load balancer adds a session cookie in the client’s browser to ensure future requests from the same session go to the same server.
      - Session Affinity: The application server generates a session ID, and the load balancer keeps track of this session to route requests accordingly.
- Health checks: Load balancers monitor server health and remove unhealthy servers from the pool.
- Scaling: Horizontal scaling by adding more servers or vertical scaling by upgrading existing servers.

---

### Databases

Database design involves choosing the right database type and structure. Consider:

- Types: SQL vs. NoSQL databases (e.g., relational vs. document or column-based).
- Replication: Master-slave replication or multi-master replication to ensure availability.
- Data locality: Choosing appropriate storage regions to minimize latency for geographically distributed users.
- CAP Theorem: The CAP theorem states that a distributed system can provide only two of the following three guarantees at any given time:
  - Consistency (C): All nodes see the same data at the same time.
  - Availability (A): Every request receives a response (success/failure), even if some of the system nodes are down.
  - Partition Tolerance (P): The system continues to function even if communication between nodes is interrupted.

    Most distributed systems choose availability and partition tolerance over strict consistency, but it depends on the application’s needs.

- ACID (SQL Databases):
  - Atomicity: Transactions are all-or-nothing. If any part of a transaction fails, the entire transaction is rolled back.
  - Consistency: Ensures that the database remains in a valid state before and after a transaction.
  - Isolation: Transactions do not interfere with each other.
  - Durability: Once a transaction is committed, it will persist, even in the event of a failure.
- BASE (NoSQL Databases):
  - Basically Available: The system guarantees availability, though it may return stale or incomplete data.
  - Soft State: The system state may change over time, even without new data due to eventual consistency.
  - Eventual Consistency: Over time, the system will become consistent, but there are no guarantees of immediate consistency after an update.
- Sharding in Databases:
  - Sharding: Horizontal partitioning of data where large datasets are split across multiple database instances, called shards. Each shard holds a portion of the data, allowing for greater scalability.
    - Sharding improves performance as queries are distributed across multiple nodes.
    - Examples: In a user database, users with IDs 1-1000 might go to Shard 1, 1001-2000 to Shard 2, etc.
- Sharding vs. Partitioning
  - Partitioning: A broader term referring to dividing a database into smaller, manageable pieces.
    - Horizontal Partitioning (Sharding): Splits the rows of a table across multiple databases.
    - Vertical Partitioning: Splits columns of a table into different databases. For example, frequently accessed columns might be stored on fast SSD storage, while less-accessed columns are stored on slower disks.
  - Sharding: A specific type of partitioning where the dataset is horizontally split across multiple databases or servers. Each shard is a standalone entity that contains a subset of the data.
- Horizontal vs. Vertical Partitioning
  - Horizontal Partitioning (Sharding):
    - Advantages:
      - Scalability: Easily add more shards to handle increased data volume.
      - Load distribution: Balances the load across multiple nodes.
      - Fault isolation: A failure in one shard doesn’t affect others.
    - Disadvantages:
      - Complexity: Requires careful design to ensure data is distributed evenly.
      - Joins across shards can be complex and slow.

  - Vertical Partitioning:
    - Advantages:
      - Simplified management: Easier to manage smaller databases.
      - Improved performance: Access to frequently used columns is faster.
      - Security: Isolates sensitive data from other databases.
    - Disadvantages:
      - Scalability: Not as scalable as horizontal partitioning.
      - Increased complexity: Requires careful design to ensure data is distributed evenly.

---

### Key-Value Store

In a key-value store, data is stored in a simple format with a unique key associated with each value. Additional considerations:

- Consistency: Strong consistency vs. eventual consistency in distributed systems.
- Partitioning: Partitioning key-value pairs across multiple nodes to ensure scalability.
- Durability: Persistence of data through replication or disk-based storage.
- Data structure: Some key-value stores (e.g., Redis) allow more complex structures like lists, sets, and hashes.
- High availability: Techniques like replication and leader election ensure availability.

---

### Content Delivery Network (CDN)

CDNs improve content delivery by caching assets closer to users. Factors include:

- Caching: CDN nodes cache content to reduce load on origin servers and lower latency.
- Geographical distribution: CDN nodes (also called edge servers) are distributed worldwide to serve users from the closest location.
- Failover: In case a CDN node fails, traffic can be rerouted to the nearest available node.
- Cache invalidation: Policies for refreshing or purging cached content to ensure users receive updated data.
- Security: Some CDNs offer DDoS protection, SSL offloading, and protection against attacks on content servers.

---

### Sequencer

A sequencer generates unique identifiers, ensuring order and causality in distributed systems. Key concepts:

- Centralized vs. decentralized: Centralized sequencers (single source of truth) vs. decentralized ones (using distributed algorithms like Lamport timestamps).
- Global vs. local IDs: Global unique IDs (UUIDs) vs. localized IDs generated for specific regions.
- Order preservation: Ensuring IDs are generated in the correct order to maintain consistency across distributed systems.
- Performance: Optimization of sequencer throughput to handle high volumes of requests.

---

### Service Monitoring

Monitoring tracks system health and provides real-time alerts for issues. Considerations:

- Metrics: CPU usage, memory, disk I/O, network latency, and error rates.
- Alerting: Configurable alerts based on thresholds for critical metrics.
- Logs: Collection and analysis of logs for troubleshooting and auditing.
- Uptime: Monitoring server availability, including failure detection and recovery.
- Tools: Tools like Prometheus, Grafana, or ELK stack for visualization and alerting.

---

### Distributed Caching

Distributed caches improve performance by reducing data retrieval times. Key concepts:

- Cache invalidation: Managing cache expiry to ensure fresh data.
- Cache consistency: Maintaining cache coherence across distributed nodes.
- Eviction policies: Policies like LRU (Least Recently Used) or LFU (Least Frequently Used) to remove stale data.
- Cache warming: Preloading frequently accessed data to improve initial performance.
- Scalability: Adding nodes to the cache cluster to handle increased load.

---

### Distributed Messaging Queue

Distributed messaging queues decouple producers (services sending messages) and consumers (services processing messages) to improve scalability and reliability.

- How Distributed Queues Work:
  - Producers: Send messages to the queue without waiting for the consumers to process them.
  - Consumers: Pull messages from the queue at their own pace and process them.
  - Brokers: Queue brokers (e.g., Kafka, RabbitMQ) manage the delivery of messages between producers and consumers.
  - Persistence: Messages are stored reliably in the queue, ensuring that even if consumers are down, the messages are not lost.
  - Fault Tolerance: Messages can be re-queued if a consumer fails to process them, ensuring eventual delivery.
- Key features of distributed messaging queues:
  - Message Ordering: Ensures messages are processed in the same order they were sent.
  - Message Acknowledgment: Consumers acknowledge receipt of a message, at which point the queue broker can delete it from the queue.
  - Message Filtering: Consumers can specify which messages they are interested in, using filters based on message attributes.
  - Message Retries: If a consumer fails to process a message, the queue broker can retry it later.
  - Message Durability: Ensures messages are durable and not lost in case of failures.
  - Message Persistence: Messages are stored reliably in the queue, ensuring that even if consumers are down, the messages are not lost.
  - Message Partitioning: Messages are partitioned across multiple nodes to improve scalability.
  - Message Replication: Messages are replicated across multiple nodes to ensure availability.
  - Message Compression: Messages are compressed to reduce storage and network usage.

---

### Publish-Subscribe System

In a Pub-Sub system, there are two main entities:

- Publishers: Entities that send messages (events) without needing to know who the recipients are.
- Subscribers: Entities that subscribe to topics and receive messages that are published to those topics.

How Pub-Sub Systems Work:

- Topics: Messages are organized into topics or channels. Publishers send messages to a specific topic.
- Subscriptions: Consumers subscribe to topics they are interested in. When a message is published to a topic, all subscribers to that topic receive the message.
- Event-driven: Pub-Sub is commonly used in event-driven architectures where services communicate asynchronously.
- Decoupling: Pub-Sub systems decouple producers (publishers) from consumers (subscribers), meaning they don’t need to know about each other’s existence. This enhances scalability and flexibility.

Key Concepts:

- Asynchronous Communication: Publishers and subscribers operate independently. Publishers don’t wait for subscribers to process messages, and subscribers can process messages as they arrive.
- Fan-out Messaging: A single message sent to a topic can be received by multiple subscribers, allowing for real-time updates across distributed systems.
- Durable Subscriptions: Some Pub-Sub systems (like Apache Kafka) allow subscribers to consume messages at their own pace, with the system keeping track of which messages have been consumed, even if the subscriber is temporarily offline.
- Message Filtering: Subscribers can set filters to receive only specific types of messages within a topic, ensuring they only get relevant data.
- Fault Tolerance: Pub-Sub systems often provide mechanisms to handle failures (e.g., retries, dead-letter queues), ensuring message delivery even in the face of network or system issues.

Common examples of Pub-Sub systems include Google Pub/Sub, Amazon SNS, Apache Kafka, and Redis Pub/Sub.

---

### Rate Limiter

Rate limiters control how frequently users can access services to prevent abuse. Considerations include:

- Algorithm: Token bucket, leaky bucket, or fixed window counters.
- Granularity: Setting limits per user, per IP, or per service.
- Throttling: Gradually slowing down users as they approach their limit, or blocking them once the limit is exceeded.
- Distributed rate limiting: Handling rate limits across multiple servers or regions.
- Burst handling: Allowing short bursts of high traffic while maintaining long-term limits.

---

### Blob Store

A blob (binary large object) store handles unstructured data. Key factors:

- Storage: Efficient storage for large files, such as images, videos, and backups.
- Scalability: Support for scaling storage as data volumes grow.
- Metadata: Associating metadata with each blob to improve organization and retrieval.
- Access control: Ensuring secure access to stored blobs, including public/private ACLs.
- Versioning: Tracking changes to blobs and maintaining previous versions if needed.

---

### Distributed Search

Distributed search engines index and query data across multiple servers. Key design points:

- Crawling: Crawling the web or dataset to collect relevant content.
- Indexing: Creating an inverted index for efficient lookups.
- Ranking: Algorithms for ranking results based on relevance.
- Fault tolerance: Ensuring search continues even if a node fails.
- Sharding: Partitioning the index to distribute the load across multiple servers.

---

### Distributed Logging

Distributed logging captures events from across the system. Key considerations:

- Log aggregation: Centralized collection of logs from multiple services.
- Log indexing: Making logs searchable for real-time troubleshooting.
- Scalability: Handling large volumes of log data efficiently.
- Retention policies: Managing log storage by removing older logs or archiving them.
- Security: Encrypting logs and ensuring access control to protect sensitive information.

---

### Distributed Task Scheduling

A distributed task scheduler allocates resources to tasks across nodes. Important points:

- Task dependencies: Managing task dependencies and execution order.
- Resource allocation: Efficiently distributing resources (e.g., CPU, memory) to tasks.
- Fault tolerance: Handling task retries and failure recovery.
- Concurrency: Executing multiple tasks in parallel where possible.
- Load balancing: Ensuring no single node is overloaded with tasks.

---

### Sharded Counters

Sharded counters distribute counting load across multiple nodes. Key features:

- Concurrency: Handling high volumes of concurrent updates (e.g., likes on social media posts).
- Sharding strategy: Distributing counters across shards to avoid bottlenecks.
- Consistency: Ensuring counters are accurate despite being updated across multiple nodes.
- Latency: Reducing the time it takes to increment/decrement counters.
- Recovery: Ensuring counters are recoverable after failures.
