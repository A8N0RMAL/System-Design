# System Design Fundamentals: Scaling from Zero to Millions
This guide outlines the foundational journey of designing a system that evolves from a simple prototype to a platform serving millions.

## Core Philosophy
Effective system design starts with asking the right questions and understanding the trade-offs (pros/cons) of available technologies. The primary driver for architectural change is **user growth**—a system for 100 users will not suffice for millions.

## System Architecture Overview
A scalable application typically involves these core components in its flow:

1.  **Frontend:** The user interface (Web/Mobile App) initiating requests.
2.  **DNS:** Translates domain names to IP addresses. Efficiency is gained through caching (browser, OS, ISP).
3.  **Web Server:** Handles HTTP requests and serves responses (HTML, JSON, etc.).
4.  **Database:** Stores and retrieves data, with a discussion on different database types.

## 💾 Database Selection Guide
Choosing the right database is critical and depends on your data structure and consistency requirements.

| Database Type | Ideal Use Case | Key Characteristics | Examples |
| :--- | :--- | :--- | :--- |
| **Relational (SQL)** | Structured data, complex transactions, strong consistency. | Fixed schema, ACID compliance, relational integrity. | MySQL, PostgreSQL |
| **Non-Relational (NoSQL)** | Flexible, schema-less data, high scalability, eventual consistency. | Dynamic schema, horizontal scaling, varied data formats (documents, key-value). | MongoDB, Cassandra |
| **Graph Databases** | Data with complex relationships and interconnectedness (mentioned for completeness). | Nodes, edges, and properties to model relationships. | Neo4j |

**Decision Framework:**
*   **Relational databases** are preferred for structured, consistent data with complex transactions (e.g., financial systems).
*   **NoSQL** databases suit flexible, evolving data schemas and less stringent consistency needs (e.g., social media feeds, analytics).
<img width="2752" height="1536" alt="systemdesign_1" src="https://github.com/user-attachments/assets/05719857-5d24-45f7-aab2-78e2445fa77b" />

---

## Scaling Strategies & High Availability Architecture

### 1. Vertical Scaling (Scaling Up)
* Increasing the capacity of a single machine by upgrading hardware components such as CPU, RAM, and disk.
* Easy to implement but limited by the maximum capacity of the machine.
* Represents a single point of failure: if the machine crashes, the entire system goes down.
<img width="2752" height="1536" alt="verticlescaling" src="https://github.com/user-attachments/assets/887c25e6-7bba-4513-b528-9890998272b2" />

---
### 2. Horizontal Scaling (Scaling Out)
* Adding more machines of the same size and distributing the load among them.
* Requires a load balancer to efficiently distribute incoming requests.
* Avoids the single point of failure problem inherent in vertical scaling.
<img width="2752" height="1536" alt="horizontalscaling" src="https://github.com/user-attachments/assets/8f49de03-290f-453e-92a0-3cfa3898b9a4" />

---
### 3. Load Balancer
* Acts as the traffic distributor, routing requests to multiple backend servers.
* Backend servers reside on a private network and are not directly accessible to users.
* Load balancers themselves can be a single point of failure; this is mitigated by clustering multiple balancers with one active and others as failovers.
* Health checks (heartbeat protocol) continuously monitor server availability and reroute traffic if a server is down.
<img width="2752" height="1536" alt="loadbalancer" src="https://github.com/user-attachments/assets/646361d8-8bc9-4c8a-a70e-61e32fe980d8" />

---
### 4. Database Scaling Challenges
* As the number of servers increases, the database can become a bottleneck due to limited connections and resource constraints.
* Vertical scaling of databases has similar limitations as servers.

### 5. Database Replication and Leader-Follower Model
* Commonly uses the Leader (Master) - Follower (Slave) replication model.
* The leader handles write operations and propagates changes to follower replicas.
* Followers handle read operations, improving read scalability and availability.
* If a follower fails, it can be replaced with another follwer replica without affecting the system via Catch-up Recovery technique.
* If the Leader fails, it can be replaced with a follwer replica via Failover technique.
* Concepts of consistency and event propagation are critical and have been explained in detail in the creator’s previous series.
<img width="2752" height="1536" alt="databasereplication" src="https://github.com/user-attachments/assets/29ef4170-3ec2-405d-b71e-796bc3f28cab" />

---
### 6. High Availability and Fault Tolerance
* Achieved through horizontal scaling, load balancer clustering, and database replication.
* Systems can continue operating smoothly even if individual components fail.

---

## Caching in System Design

* **Replication**: Multiple copies of the database are maintained to handle increased load.
* **Caching**: Introduced as a memory-based layer (e.g., using Redis) placed between web servers and databases to store frequently or recently accessed data in RAM, which is faster than disk reads.

### Cache Operation:
1. Queries first check the cache.
2. On a cache hit, data is returned immediately.
3. On a cache miss, data is fetched from the database, cached, and then returned.

### Benefits of Caching:
* Reduces database query load.
* Improves response time due to faster memory access.
<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/b45b7f34-4946-4592-b4ea-4823eeab84a9" />

---
### Cache Write Strategies:
* Write-Behind: Write to cache first, then database. Fast response but risk of data loss if write to database fails.
* Write-Around: Write to database first, then update cache asynchronously. Avoids data loss but may serve stale data temporarily.
* Write-Through: Write to both cache and database simultaneously. Suitable when data size is small or cost of RAM is a concern.
<img width="1036" height="1079" alt="Screenshot 2025-12-08 213300" src="https://github.com/user-attachments/assets/c87d1b47-8469-4309-8503-ad01ec61114c" />

---
### Cache Limitations:
* RAM size limits data stored in cache; primary data must reside on disk.
* Cache data can become stale if not properly managed.

---
### Cache Eviction and Expiration Policies:
* Expiration Policy: Cached data has a time-to-live (TTL) to prevent serving outdated data.
* Retention (Eviction) Policy: When cache is full, remove data based on usage patterns:
  * Time-based: Remove data not accessed recently.
  * LFU (Least Frequently Used): Remove data accessed least often.
  * LRU (Least Recently Used): Remove data least recently accessed.
  * LFRU: A hybrid method combining LFU and LRU.
<img width="1919" height="975" alt="Screenshot 2025-12-08 213550" src="https://github.com/user-attachments/assets/8540a757-d045-4ea0-8200-7c5fa3170ea8" />

---
### Cache Failure Handling:
* If cache fails, all requests fall back to the database, which may cause performance degradation or system crashes under heavy load.
* To mitigate this, multiple cache clusters and tuning are recommended.

---

## Caching and CDN
* **CDN(Content Delivery Network)** acts as a geographically distributed caching layer that speeds up file delivery by serving files(static files (e.g. , images, videos, javascript or css files)) from the nearest server to the user.

### Cache Invalidation and TTL
* Explanation of how cached files persist until they expire or are renamed (e.g., by appending timestamps).
* Use of HTTP cache-control headers to specify TTL (Time To Live), which determines how long a file remains cached before expiration.
<img width="1134" height="295" alt="image" src="https://github.com/user-attachments/assets/d944a808-f6bd-4c85-b78d-cb7497a090e2" />

---
### Challenges with CDN
* Dependence on external services such as Amazon CloudFront or Cloudflare, which can be costly and introduce risks related to third-party reliance.
* After cache expiration, the CDN requires permission from the origin server to fetch updated files, which can affect update speed.

---
