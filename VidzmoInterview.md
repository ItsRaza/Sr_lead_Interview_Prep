# VIDIZMO Senior Software Engineer / Tech Lead Interview Preparation

**Role:** Senior Software Engineer / Technical Lead  
**Focus:** AI, Video Processing, High Availability, Distributed Systems, .NET Core, Azure.  
**Candidate:** Syed Raza Abbas

---

## 📖 Table of Contents
1. [Part 1: Advanced .NET Core & Internals](#part-1-advanced-net-core--internals)
2. [Part 2: Database Engineering & High Volume](#part-2-database-engineering--high-volume)
3. [Part 3: Microservices Architecture](#part-3-microservices-architecture)
4. [Part 4: Cloud, DevOps & High Availability](#part-4-cloud-devops--high-availability)
5. [Part 5: Messaging & Distributed Data (Kafka/RabbitMQ)](#part-5-messaging--distributed-data)
6. [Part 6: Observability & Tooling](#part-6-observability--tooling)
7. [Part 7: Behavioral & Leadership](#part-7-behavioral--leadership)

---

## Part 1: Advanced .NET Core & Internals
**Key Concepts:** `HttpContext`, Thread Safety, Async/Await, Memory Management.

### Q1: The `HttpContext` Trap
**Scenario:** "You see a Pull Request where a developer has assigned `IHttpContextAccessor` to a `static` field to easily access the `CurrentUser` ID in a background service. What is the danger here?"

**The Technical Answer:**
* **Thread Safety:** `HttpContext` is **not thread-safe**. It is scoped strictly to the incoming HTTP request.
* **Async/Await Issues:** If the thread switches (e.g., after an `await` call) or if the logic runs on a background thread (`Task.Run`), the `HttpContext` might be null, disposed, or worse—pointing to a *different* user's request (Thread Borrowing/Context Switching).
* **The Fix:** 1.  Never use static for Request-Scoped services.
    2.  Extract the necessary data (like `UserId` or `TenantId`) in the Controller and pass it as a simple parameter (primitive/DTO) to the background service.

### Q2: Singleton vs. Scoped vs. Transient
**Scenario:** "We have a `VideoProcessingService` that maintains a list of `CurrentJobs` in memory. It is registered as `AddTransient`. What happens when 100 users hit the service simultaneously?"

**The Technical Answer:**
* **The Bug:** If registered as `Transient`, a **new instance** is created for *every* injection. 
* **Consequence:** The `CurrentJobs` list will always be empty or contain only the job of the current request. The data is lost immediately after the request ends.
* **Fix:** It should likely be a **Singleton** (if shared across all users) or use a distributed store (Redis) if running on multiple servers. **Scoped** would create a new instance per HTTP request, which also fails to share state across users.

### Q3: Async void vs. Async Task
**Scenario:** "Why is `async void` strictly forbidden in our backend services, except for Event Handlers?"

**The Technical Answer:**
* **Error Handling:** Exceptions thrown in an `async void` method **crash the process** immediately. They cannot be caught by `try-catch` blocks in the calling code because there is no `Task` object to observe the exception.
* **Best Practice:** Always return `async Task`. This allows the exception to bubble up and be handled by the Global Exception Middleware.

> 📺 **Recommended Watch:** [Async/Await Best Practices in .NET (Raw Coding)](https://www.youtube.com/watch?v=5KfT6j4eWqU)

---

## Part 2: Database Engineering & High Volume
**Key Concepts:** Indexing, Sharding, Replication, Performance Tuning.

### Q1: Improving Query Performance
**Scenario:** "The `VideoAuditLogs` table has 500 million rows. A query filtering by `UploadDate` is taking 15 seconds. The column has an index. Why is it still slow?"

**The Technical Answer:**
* **Key Lookup Cost:** If the query selects columns *not* in the index (e.g., `SELECT *`), the DB engine must do a "Key Lookup" to fetch the rest of the row data from the Clustered Index (Disk I/O).
* **Solution:** Use a **Covering Index**. Create an index that `INCLUDES` the columns you need to select.
    * *SQL:* `CREATE INDEX IX_Date ON Logs(UploadDate) INCLUDE (VideoId, Status);`
* **Fragmentation:** Check if the index is fragmented. If inserts are random (GUIDs), fragmentation kills performance. Rebuild/Reorganize the index.

### Q2: Sharding Strategies
**Scenario:** "Our primary SQL database has reached 4TB. Backups take 12 hours. We need to implement Sharding. How do we approach this for a Multi-Tenant SaaS app?"

**The Technical Answer:**
* **Strategy:** **Tenant-Based Sharding**.
* **Implementation:** 1.  Route requests based on `TenantId`.
    2.  Shard A holds Tenants 1-100; Shard B holds Tenants 101-200.
* **Pros:** Data isolation (security), smaller backups per shard, infinite horizontal scale.
* **Cons:** Cross-tenant analytics become hard (require a separate Data Warehouse/ETL).

### Q3: Database Replication (Read/Write Split)
**Scenario:** "The CPU on our primary database is at 95% due to heavy reporting queries. How do we fix this without code optimization?"

**The Technical Answer:**
* **Architecture:** Implement **Read Replicas** (Master-Slave architecture).
* **Mechanism:** * Point all `INSERT/UPDATE/DELETE` (Command) queries to the **Primary** DB.
    * Point all `SELECT` (Query) operations (especially reports) to the **Read Replica**.
* **Latency Note:** Be aware of *Replication Lag*. A user might create a video and not see it immediately in the list if the replica hasn't synced yet (Eventual Consistency).

> 📺 **Recommended Watch:** [Database Indexing & Tuning (Brent Ozar)](https://www.youtube.com/watch?v=HhqIxX1R0M0)

---

## Part 3: Microservices Architecture
**Key Concepts:** API Gateway, Circuit Breaker, gRPC, Aggregation.

### Q1: Handling Service Failures (Circuit Breaker)
**Scenario:** "Service A calls Service B. Service B is down or slow. Service A keeps retrying, consuming all its threads waiting for answers, eventually crashing itself. How do we prevent this 'Cascading Failure'?"

**The Technical Answer:**
* **Pattern:** **Circuit Breaker** (using **Polly** in .NET).
* **Logic:**
    1.  **Closed:** Traffic flows normally.
    2.  **Open:** If failure rate > 50%, "Open" the circuit. Fail immediately without calling Service B.
    3.  **Half-Open:** After a timeout, let 1 request through to test Service B. If success, Close circuit.
* **Benefit:** Gives Service B time to recover and prevents Service A from hanging.

### Q2: API Gateway (Ocelot)
**Scenario:** "Our frontend needs data from User, Video, and Billing services to show the dashboard. Making 3 calls over the internet is slow. How can Ocelot help?"

**The Technical Answer:**
* **Pattern:** **Request Aggregation**.
* **Implementation:** Configure Ocelot `ocelot.json` to define an aggregate route.
    * Client calls `/dashboard`.
    * Ocelot calls User, Video, and Billing services *internally* (fast network).
    * Ocelot merges the JSON responses into one object and returns it to the client.

### Q3: gRPC vs. REST
**Scenario:** "We are building an internal video transcoder. The Video Service needs to send 10,000 frames per second to the AI Service. Should we use REST?"

**The Technical Answer:**
* **Choice:** **gRPC**.
* **Why:** 1.  **Protobuf (Binary):** Much smaller payload than JSON (REST).
    2.  **Streaming:** gRPC supports bi-directional streaming. You can stream frames continuously without opening/closing connections repeatedly.
    3.  **Strict Contracts:** `.proto` files ensure type safety between services.

> 📺 **Recommended Watch:** [Microservices Design Patterns (Hussein Nasser)](https://www.youtube.com/watch?v=kCwCq3lG-8w)

---

## Part 4: Cloud, DevOps & High Availability
**Key Concepts:** Azure, Docker, Kubernetes, HA/DR.

### Q1: Stateless vs. Stateful Architecture
**Scenario:** "We are moving to Kubernetes with 10 replicas of our app. Users report they have to log in again every time they refresh the page. Why?"

**The Technical Answer:**
* **Root Cause:** You are likely using **In-Memory Sessions**. When a user hits Replica A, their session is created there. The next request hits Replica B, which knows nothing about that session.
* **The Fix:** **Distributed Caching (Redis)**.
    * Store Session/Auth data in Redis.
    * All K8s replicas read/write to the same Redis instance.
    * This makes the application **Stateless**.

### Q2: Azure HA (Region Failover)
**Scenario:** "Design a deployment that survives a total outage of the 'East US' Azure region."

**The Technical Answer:**
* **Traffic:** **Azure Front Door** (Global Load Balancer) to route traffic to healthy regions.
* **App:** Run active clusters in East US and West US.
* **Data:** * **SQL:** Use **Failover Groups** with Auto-Failover enabled.
    * **Storage:** Use **RA-GRS** (Read-Access Geo-Redundant Storage) for blobs/videos.
* **RPO/RTO:** Define how much data loss (RPO) and downtime (RTO) is acceptable. For RPO=0, you need synchronous replication (slower performance).

---

## Part 5: Messaging & Distributed Data
**Key Concepts:** RabbitMQ vs Kafka, Redis.

### Q1: RabbitMQ vs. Apache Kafka
**Scenario:** "We have two use cases: 1) A task queue for video rendering (must be reliable). 2) A stream of user clicks for analytics (millions/sec). Which tool for which?"

**The Technical Answer:**
* **Video Rendering -> RabbitMQ:**
    * **Why:** Smart broker. Supports complex routing, priority queues, and individual message Acknowledgement (Ack/Nack). If a worker dies, the message is re-queued.
* **Click Stream -> Kafka:**
    * **Why:** Dumb broker / Smart consumer. Designed for high throughput (log-based). Messages are persisted on disk for X days. You can "replay" the stream later to train new AI models.

### Q2: Redis Caching Strategies
**Scenario:** "We cache video metadata in Redis. A video is updated in SQL, but users still see the old title. How do we handle Cache Invalidation?"

**The Technical Answer:**
* **Strategy:** **Write-Through** or **Cache-Aside with TTL**.
    * *Cache-Aside:* When code updates the DB, it *must* also delete the key from Redis (`_cache.Remove("video:123")`).
    * *TTL (Time To Live):* Always set an expiry (e.g., 5 mins) so stale data eventually fixes itself if the explicit delete fails.

---

## Part 6: Observability & Tooling
**Key Concepts:** Distributed Tracing, Zipkin, ELK.

### Q1: Debugging Distributed Latency
**Scenario:** "A user complains 'Upload is slow'. The request goes through Gateway -> Auth -> Upload -> S3. How do we know *which* step is slow?"

**The Technical Answer:**
* **Tool:** **Distributed Tracing (Zipkin / Jaeger / OpenTelemetry)**.
* **How it works:**
    1.  Gateway generates a unique `TraceId`.
    2.  This ID is passed in headers (`x-b3-traceid`) to all downstream services.
    3.  Each service sends a "Span" (start/end time) to Zipkin.
    4.  Visualizer shows a waterfall graph: "Auth took 5ms, Upload took 15s".

### Q2: Service Discovery (Consul/mDNS)
**Scenario:** "We deploy on-premise for a high-security client without Azure. How do services find each other without hardcoded IPs?"

**The Technical Answer:**
* **Consul:** Use Consul for Service Discovery. Services register themselves (`I am VideoService at 192.168.1.5`). Clients query Consul to get the IP.
* **mDNS:** For local device discovery (e.g., finding cameras on the same LAN), use Multicast DNS.

---

## Part 7: Behavioral & Leadership
**Based on CV & JD:**
* **Mentorship:** "Tell me about a time you upskilled a junior engineer. How did you handle code reviews?"
    * *Tip:* Mention your experience at *Enterprise64* leading 10+ engineers. Focus on constructive feedback and "Why" not just "What".
* **Ownership:** "Describe a production failure you caused. How did you fix it and prevent it from happening again?"
    * *Tip:* Be honest. Focus on the **Post-Mortem** process. "We fixed the bug, then added a regression test, then added a metric to the dashboard to catch it earlier next time."

---
*Prepared by Gemini for Syed Raza Abbas*
