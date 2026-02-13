# VIDIZMO Lead Engineer Interview Prep Guide

**Prepared for:** Sr. Software Engineer (6+ years, Microsoft stack)  
**Role:** Lead Engineer at VIDIZMO  
**Company:** USA-based, Tysons VA — Microsoft Solutions Partner (Data & AI, Infrastructure, Digital & App Innovation)

---

## Table of Contents

1. [React & JavaScript](#1-react--javascript)
2. [Database (SQL Server, Indexes, CTEs, Window Functions, Scaling)](#2-database)
3. [.NET, Microservices, Ocelot, gRPC](#3-net-microservices-ocelot-grpc)
4. [CI/CD, Docker, Kubernetes](#4-cicd-docker-kubernetes)
5. [Azure Cloud](#5-azure-cloud)
6. [Cheat Sheets (Quick Reference)](#6-cheat-sheets-quick-reference)

---

# 1. React & JavaScript

## YouTube Links

| Topic | Link | Description |
|-------|------|-------------|
| Top 100 React JS Interview Q&A | https://www.youtube.com/watch?v=5it_Uv7pGFg | Comprehensive React interview questions with answers |
| Top 20 React 2025 Interview | https://www.youtube.com/watch?v=NkWOzTEEcco | Focused React interviewer Q&A (Intellipaat) |
| Build 25 React Projects (9+ hrs) | https://www.youtube.com/watch?v=5ZdHfJVAY-s | Projects: accordion, weather, cart, custom hooks |
| 25 React Projects Part 2 (8+ hrs) | https://www.youtube.com/watch?v=QZXBylaAk7s | Pagination, timers, forms, Firebase, API calls |
| React Machine Coding: Quiz App | https://www.youtube.com/watch?v=TF1FKrzsRDM | Frontend machine coding + system design |
| JavaScript Interview Prep | https://www.youtube.com/watch?v=8ext9G7xspg | freeCodeCamp JS fundamentals |
| React Hooks Deep Dive | https://www.youtube.com/watch?v=TNhaISOUy6Q | useState, useEffect, custom hooks |
| JavaScript Event Loop | https://www.youtube.com/watch?v=8aGhZQkoFbQ | Philip Roberts: event loop, call stack, queue |

---

## Questions & Answers (Lead Level)

### React

**Q1: Explain React’s reconciliation and the virtual DOM. How would you optimize re-renders in a large app?**

**A:** React keeps an in-memory representation of the DOM (virtual DOM). On state/props change, it diffs the new virtual tree with the previous one and applies minimal updates to the real DOM (reconciliation). For optimization: use `React.memo()` for pure components, `useMemo`/`useCallback` to avoid unnecessary recalculations and callback identity changes, code-splitting with `React.lazy()` and `Suspense`, and avoid inline objects/functions in JSX when they are used as dependencies or props to memoized children.

**Q2: When would you use Context API vs state management (Redux/Zustand)? How do you avoid Context over-renders?**

**A:** Use Context for theme, locale, auth (low-change, app-wide). Use Redux/Zustand when you have complex shared state, need time-travel debugging, middleware, or granular subscriptions. To avoid Context over-renders: split contexts by domain (AuthContext, ThemeContext), wrap the value in `useMemo`, or use state libraries that support selective subscriptions so only components that use changed data re-render.

**Q3: How do you structure a scalable React app (folders, patterns)?**

**A:** Feature-based or hybrid: `src/features/<feature>/components`, `hooks`, `api`, `types`, `utils`. Shared: `components`, `hooks`, `lib`, `styles`. Use custom hooks for data fetching and side effects. Co-locate tests. Consider a design system for reusable components. Use path aliases and clear naming (e.g. `UserProfile.tsx` vs `index.tsx` for public API).

**Q4: Explain useEffect cleanup and how you’d prevent memory leaks with subscriptions/timers.**

**A:** `useEffect` can return a cleanup function that runs before the next effect run and on unmount. For subscriptions/timers: create the subscription or timer inside the effect, and in the return function call `unsubscribe()` or `clearInterval`/`clearTimeout`. Never update state on an unmounted component—use a ref or an “isMounted” guard, or (in modern React) rely on Strict Mode and cleanup to avoid setting state after unmount.

**Q5: What are React Server Components (RSC) and when would you use them vs client components?**

**A:** RSC run only on the server and send rendered output to the client (no JS for that tree by default). Use them for data fetching, heavy dependencies, and static content to reduce bundle size and improve TTFB. Use client components for interactivity, hooks, browser APIs, and event handlers. Lead-level answer: understand the boundary (client vs server), serialization limits, and that RSC are part of the React/Next.js evolution for better performance and DX.

### JavaScript

**Q6: Explain the event loop, macro-tasks vs micro-tasks. What’s the order of execution?**

**A:** The event loop takes tasks from the call stack; when the stack is empty, it runs micro-tasks (e.g. Promise callbacks, queueMicrotask), then one macro-task (e.g. setTimeout, setInterval, I/O). Micro-tasks are drained before the next macro-task. So: sync code → all micro-tasks → one macro-task → repeat. This explains why `Promise.then` runs before `setTimeout(0)`.

**Q7: What’s the difference between `var`, `let`, and `const`? What is temporal dead zone (TDZ)?**

**A:** `var` is function-scoped and hoisted (initialized as `undefined`). `let` and `const` are block-scoped and hoisted but not initialized until their declaration line (TDZ). In the TDZ, accessing the variable throws. `const` must be assigned once and can’t be reassigned; for objects, the reference is constant but properties can change.

**Q8: How does `this` work in JavaScript? How would you bind it in class components vs functional?**

**A:** `this` is determined by how a function is called: default (non-strict: global/window, strict: undefined), implicit (object method), explicit (`call`/`apply`/`bind`), or `new`. In React class components, bind in constructor or use class fields: `onClick = () => this.handler()`. In function components there’s no `this`; use closures and pass callbacks.

**Q9: Explain closure and a practical use (e.g. private state, factory).**

**A:** A closure is when a function retains access to variables from its outer scope after that scope has finished. Uses: private state (module pattern), factories, partial application, and in React (hooks and callbacks capturing state in event handlers).

**Q10: How would you improve performance of a large list (e.g. 10k rows)?**

**A:** Virtualization (windowing): render only visible rows (e.g. react-window, react-virtualized). Use stable keys (e.g. ID). Avoid inline styles/objects. Memoize list items if they’re heavy. Consider pagination or infinite scroll. For tables, use CSS containment and avoid layout thrashing.

---

# 2. Database

## YouTube Links

| Topic | Link | Description |
|-------|------|-------------|
| SQL Window Functions + CTE | https://www.youtube.com/watch?v=SatZc2Fq8-4 | Complex SQL interview with window functions & CTE |
| Tricky SQL Interview Query | https://www.youtube.com/watch?v=6UAU79FNBjQ | Solving tricky SQL with datasets |
| CTE, Join, Window, Rank | https://www.youtube.com/watch?v=knv8Aj6ykNs | Oxane Partners style SQL interview |
| SQL Indexing Explained | https://www.youtube.com/watch?v=HcRcCWVqJ-c | Index types and when to use |
| Database Sharding | https://www.youtube.com/watch?v=5faMjKuB9bc | Sharding concepts and patterns |
| SQL Server Performance Tuning | https://www.youtube.com/watch?v=8GpPWZyFqCQ | Query optimization basics |

---

## Questions & Answers (Lead Level)

### Indexes

**Q1: What are clustered vs non-clustered indexes? When do you use each?**

**A:** **Clustered:** One per table; determines physical order of data. The table is the leaf level. Use on the column(s) used for range scans and as the main access path (e.g. primary key, often identity). **Non-clustered:** Separate structure with key + pointer to data. Use for other search/join/sort columns. Covering index includes all columns needed by the query (index-only scan). Too many indexes hurt write performance; balance read vs write workload.

**Q2: What is a covering index? What’s index selectivity and why does it matter?**

**A:** A covering index includes all columns required by the query so the engine doesn’t need to look up the base table. **Selectivity:** ratio of distinct values to rows. High selectivity (e.g. unique IDs) is good for indexes; low (e.g. boolean) often isn’t. The optimizer uses selectivity to choose indexes; low selectivity can lead to table scans.

**Q3: How would you find missing indexes and duplicate/unused indexes in SQL Server?**

**A:** Use DMVs: `sys.dm_db_missing_index_details` (and related) for suggested indexes; `sys.dm_db_index_usage_stats` for usage. Cross-reference with existing indexes to avoid duplicates. Remove or consolidate unused or overlapping indexes. Test in non-prod and monitor impact.

### CTEs & Temp Tables

**Q4: When do you use a CTE vs a temp table vs a table variable?**

**A:** **CTE:** Readability, recursion, single-statement scope; no statistics; can be inlined or materialized (version-dependent). **Temp table (#):** Session scope, statistics, indexes, good for large intermediate data and multiple references. **Table variable (@):** Single batch/session, no statistics (older versions), good for small sets; can cause poor cardinality estimates. Lead choice: CTE for clarity in one query; temp table for heavy reuse or when statistics help the plan.

**Q5: Write a recursive CTE to get a hierarchy (e.g. employee manager).**

**A:**
```sql
WITH EmployeeHierarchy AS (
    SELECT Id, Name, ManagerId, 1 AS Level
    FROM Employee WHERE ManagerId IS NULL
    UNION ALL
    SELECT e.Id, e.Name, e.ManagerId, eh.Level + 1
    FROM Employee e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerId = eh.Id
)
SELECT * FROM EmployeeHierarchy;
```
Anchor: top-level rows. Recursive member: join to CTE. Set MAXRECURSION if needed to avoid infinite loops.

### Window Functions

**Q6: Explain ROW_NUMBER, RANK, DENSE_RANK, NTILE. When is each used?**

**A:** **ROW_NUMBER:** Unique per partition/order (e.g. “first N per group”). **RANK:** Same value → same rank, gap after ties (e.g. 1,2,2,4). **DENSE_RANK:** Same rank, no gap (1,2,2,3). **NTILE(n):** Buckets rows into n groups. Use ROW_NUMBER for deduplication or top-N; RANK/DENSE_RANK for rankings; NTILE for percentiles or equal groups.

**Q7: What are frame clauses (ROWS vs RANGE)? Give an example of a running total.**

**A:** Frame defines which rows are used for the aggregate in the current row. **ROWS:** Physical rows (e.g. preceding 2 rows). **RANGE:** Logical range by order key (e.g. same value = same frame). Running total:
```sql
SUM(Amount) OVER (ORDER BY Date ROWS UNBOUNDED PRECEDING)
```
Use ROWS for deterministic performance; RANGE when you need logical grouping by value.

### Keys & Normalization

**Q8: Explain primary key, unique key, foreign key. What’s a composite key?**

**A:** **Primary key:** Unique, not null, one per table; often clustered. **Unique:** Unique values, allows one NULL (in SQL Server). **Foreign key:** References primary/unique key; enforces referential integrity. **Composite key:** Key made of multiple columns (e.g. OrderId + LineNumber). Use when a single column doesn’t uniquely identify the row.

**Q9: How do you approach query optimization? What’s in your checklist?**

**A:** (1) Measure: execution plan, IO stats, duration. (2) Find expensive operations: scans, high cost, key lookups. (3) Add/improve indexes (covering, selective). (4) Rewrite: avoid SELECT *, reduce joins, use EXISTS instead of IN when appropriate, fix SARGability (no functions on indexed columns). (5) Consider partitioning, stats updates, parameter sniffing. (6) Test and monitor.

### Scaling, Sharding, Multiple DBs (Microservices)

**Q10: How would you scale a database for read-heavy vs write-heavy workloads?**

**A:** **Read-heavy:** Read replicas, caching (Redis), read-only routing, connection pooling, CDN for static data. **Write-heavy:** Partition tables (by key or time), archive old data, optimize indexes (fewer/secondary), consider async writes or queue-based writes, scale-up then shard if needed.

**Q11: What is sharding? What are the main challenges?**

**A:** Sharding is horizontal partitioning of data across multiple DB instances (shards). Each shard holds a subset of data (e.g. by user_id hash or range). **Challenges:** Cross-shard queries and joins, rebalancing when adding shards, global uniqueness (e.g. IDs), transactions across shards, operational complexity. Mitigate with application-level routing, ID generation (e.g. Snowflake), and avoiding cross-shard transactions where possible.

**Q12: In a microservices architecture, how do you manage databases (one DB per service)?**

**A:** **Database per service:** Each service owns its DB schema; no direct DB access from other services. **Benefits:** Loose coupling, independent scaling and deployment, technology diversity. **Challenges:** Data consistency (use saga, eventual consistency, events), no cross-DB joins (use APIs or events to aggregate), reporting (CQRS, data warehouse, or read replicas fed by events).

---

# 3. .NET, Microservices, Ocelot, gRPC

## YouTube Links

| Topic | Link | Description |
|-------|------|-------------|
| .NET Microservices + Ocelot | https://www.youtube.com/watch?v=T5JY7I2elPs | Full microservices demo with Ocelot API Gateway |
| Ocelot API Gateway Part 1 (Routing) | https://www.youtube.com/watch?v=s3DyxLb5a_o | Ocelot routing in ASP.NET Core |
| gRPC in .NET | https://www.youtube.com/watch?v=XL4sWVtQYSU | gRPC with C# and Protocol Buffers |
| Microservices with ASP.NET Core | https://www.youtube.com/watch?v=0TvGf4lG0qY | Clean architecture microservices |
| .NET 8 Minimal APIs & Microservices | https://www.youtube.com/watch?v=W5g2t1s4_2c | Modern .NET microservices patterns |

---

## Questions & Answers (Lead Level)

### .NET

**Q1: What’s the difference between .NET Framework and .NET Core / .NET 5+? When would you choose one?**

**A:** .NET Framework is Windows-only, full framework. .NET Core / .NET 5+ are cross-platform, open-source, modular, and support side-by-side. For new apps and microservices, use .NET 8 (LTS). Use .NET Framework only for legacy Windows-only apps or dependencies that haven’t been ported.

**Q2: Explain dependency injection in ASP.NET Core. What are service lifetimes?**

**A:** DI is built-in; register services in `Program.cs` or `Startup.cs`. **Lifetimes:** **Transient:** New instance per request. **Scoped:** One per scope (e.g. per HTTP request). **Singleton:** One for the app. Use Scoped for DbContext, Transient for lightweight stateless services, Singleton for caches and shared state (thread-safe).

**Q3: How does middleware work in ASP.NET Core? What’s the order of execution?**

**A:** Middleware is a pipeline of delegates. Each can invoke the next with `next()` or short-circuit. Order matters: e.g. Exception handling → HTTPS redirect → Static files → Routing → Auth → Endpoints. Configure in `UseMiddleware` or extension methods (UseAuthentication, UseAuthorization).

### Microservices

**Q4: How do microservices communicate? When do you use sync vs async?**

**A:** **Sync:** HTTP/REST or gRPC when you need an immediate response (e.g. user request, simple CRUD). **Async:** Message broker (RabbitMQ, Kafka, Azure Service Bus) for decoupling, resilience, and event-driven flows. Use async for cross-service workflows, events, and when the caller doesn’t need to wait. Prefer async for inter-service when possible to reduce coupling and improve resilience.

**Q5: What are the challenges of distributed systems (microservices)? How do you handle them?**

**A:** **Partial failure:** Circuit breaker (Polly), retries with backoff, timeouts. **Consistency:** Saga (choreography or orchestration), eventual consistency, idempotency. **Observability:** Distributed tracing (OpenTelemetry, Zipkin), structured logging, metrics. **Discovery:** Service registry (Consul) or platform (Kubernetes DNS). **Configuration:** Central config (Azure App Configuration) and feature flags.

**Q6: What is the API Gateway pattern? What does Ocelot do?**

**A:** API Gateway is a single entry point for clients. It handles routing, aggregation, auth, rate limiting, and translation. **Ocelot:** .NET library that provides routing (map routes to downstream services), load balancing, authentication (JWT, etc.), caching, and request/response transformation. Configured via `ocelot.json` (routes, aggregates, auth).

**Q7: How do Ocelot and gRPC work together?**

**A:** Ocelot can route HTTP requests to gRPC backends. Configure a route with a downstream scheme (e.g. grpc), host, and port. For HTTP-to-gRPC, use gRPC-Web or a transcoding layer if clients are browsers. Service-to-service can use gRPC directly for performance; Ocelot sits at the edge for external clients and can route to gRPC services.

**Q8: What is gRPC? When would you choose gRPC over REST?**

**A:** gRPC is an RPC framework using HTTP/2 and Protocol Buffers. **Advantages:** Binary serialization, multiplexing, streaming (client/server/bidi), strong typing via .proto, code generation. **Use gRPC** for internal service-to-service, low latency, streaming, and polyglot contracts. **Use REST** for public APIs, browser clients (without gRPC-Web), and when you need human-readable payloads and wide tooling.

**Q9: How do you secure microservices (auth, API keys, mTLS)?**

**A:** **Auth:** JWT validated at gateway or each service; OAuth2/OpenID Connect with an identity provider. **API keys:** For service-to-service or partners; validate at gateway. **mTLS:** Mutual TLS for service-to-service in zero-trust; each service has a certificate. Use Azure AD, API Management, or Ocelot with auth middleware for tokens and keys.

---

# 4. CI/CD, Docker, Kubernetes

## YouTube Links

| Topic | Link | Description |
|-------|------|-------------|
| Azure DevOps CI/CD for AKS | https://www.youtube.com/watch?v=U6n6NzGKyRI | Full pipeline for Kubernetes on Azure |
| Docker to Azure Web App CI/CD | https://www.youtube.com/watch?v=kIRc5J-0mfc | YAML pipeline for Docker → Azure Web App |
| Deploy to AKS and ACR | https://www.youtube.com/watch?v=tRjhTcXMAbo | Azure DevOps → ACR → AKS |
| Docker + Azure DevOps on K8s | https://www.youtube.com/watch?v=B0n_JZ7N028 | Build and deploy containerized apps to K8s |
| Getting started CI/CD & AKS (VSTS) | https://www.youtube.com/watch?v=HMIxLaisKiI | Beginner AKS + VSTS pipeline |
| Dockerfile Best Practices | https://www.youtube.com/watch?v=JofsaZ3H1qM | Multi-stage, layers, security |
| Kubernetes in 100 Seconds | https://www.youtube.com/watch?v=PziYfluuGVs | Quick K8s concepts |

---

## Questions & Answers (Lead Level)

### CI/CD

**Q1: What’s the difference between CI and CD? What does a typical pipeline include?**

**A:** **CI:** Build, test, and merge code frequently; run on every commit/PR (build, unit tests, lint, maybe integration tests). **CD:** Deploy to environments (release, staging, prod) automatically or with approval. Pipeline: checkout → restore → build → test → (optional) security scan → publish artifact → deploy (to container registry, then to K8s/Web App). Use YAML pipelines (Azure DevOps, GitHub Actions) for versioning and reuse.

**Q2: How do you implement branch strategy and environments (dev, staging, prod)?**

**A:** **Branches:** main/production, develop, feature/*. PR from feature to develop; release branch from develop to main. **Environments:** Separate subscriptions or resource groups; deployment stages with approvals for prod. Use environment-specific config (ARM, Bicep, or config service) and secrets in Key Vault or pipeline variables (secret).

**Q3: How do you secure the pipeline (secrets, SAST/DAST)?**

**A:** **Secrets:** Never in code; use pipeline secret variables, Azure Key Vault integration, or managed identities. **SAST:** Static analysis (e.g. SonarQube, CodeQL) in CI. **DAST:** Dynamic scan in staging. **Containers:** Scan images for vulnerabilities (Trivy, Azure Defender). Sign artifacts where possible; limit who can approve prod.

### Docker

**Q4: How is a Dockerfile structured? What are best practices?**

**A:** **Structure:** Base image (FROM), optional ARG, set WORKDIR, COPY/ADD, RUN (minimize layers; combine commands), EXPOSE, CMD/ENTRYPOINT. **Best practices:** Use specific tags (not `latest`). Multi-stage build: build in one stage, copy artifact to slim runtime image. Order layers by change frequency (dependencies before app code). Run as non-root. Use .dockerignore to exclude unneeded files. Prefer COPY over ADD unless you need URL or tar extraction.

**Q5: What’s the difference between COPY and ADD? What’s a multi-stage build?**

**A:** **COPY:** Copy files/dirs from build context. **ADD:** Can also fetch URLs and extract archives (less predictable). Prefer COPY. **Multi-stage build:** Multiple FROM in one Dockerfile. First stage(s) build and compile; final stage copies only artifacts (e.g. binary) into a small image. Reduces image size and hides build tools from production.

**Q6: How do Docker and Kubernetes connect? What does Kubernetes get from a Docker image?**

**A:** Docker builds images (layers + metadata). Images are pushed to a registry (e.g. ACR, Docker Hub). Kubernetes doesn’t run Docker directly; it uses a container runtime (containerd, CRI-O) that can run OCI images. Kubernetes pulls the image by reference (e.g. `acr.io/myapp:v1`), runs it in a Pod, and manages lifecycle (restart, scale, rollout). So: Dockerfile → image → registry → Kubernetes pulls and runs containers.

### Kubernetes

**Q7: Explain Pod, Deployment, Service, Ingress. How do they work together?**

**A:** **Pod:** Smallest unit; one or more containers, shared network/storage. **Deployment:** Declares desired state for Pods (replicas, image, strategy); creates ReplicaSets. **Service:** Stable network identity and load balancing to Pods (ClusterIP, NodePort, LoadBalancer). **Ingress:** HTTP routing (host/path) to Services; TLS termination. Flow: Ingress → Service → Pods; Deployment ensures Pods exist and are updated (rolling update).

**Q8: What’s the difference between rolling update and blue-green? How do you do a rollback?**

**A:** **Rolling:** New pods come up as old ones are terminated; no full downtime; mixed versions briefly. **Blue-green:** Two full environments; switch traffic at once; instant rollback by switching back. In K8s, rolling is default (Deployment); blue-green can be two Deployments + switch Service selector. **Rollback:** `kubectl rollout undo deployment/<name>` or revert to previous ReplicaSet.

**Q9: How do you manage config and secrets in Kubernetes?**

**A:** **ConfigMap:** Non-sensitive config (env vars, files). **Secret:** Sensitive data (base64 or external providers). Mount as env or files. Prefer external secret stores (e.g. Azure Key Vault with CSI driver) so secrets aren’t stored in etcd. Use namespaces and RBAC to limit access; avoid putting secrets in image or source.

---

# 5. Azure Cloud

## YouTube Links

| Topic | Link | Description |
|-------|------|-------------|
| AKS Overview | https://www.youtube.com/live/c4nTKMU6fBU | Azure Kubernetes Service concepts |
| AKS Deployment | https://www.youtube.com/watch?v=oBB6J3XA3m8 | Deploying to AKS |
| Azure Fundamentals (AZ-900 style) | https://www.youtube.com/watch?v=NKEFWyqJ5XA | Core Azure concepts |
| Microservices on Azure | https://www.youtube.com/watch?v=wCwMhR2JnA0 | Architecture patterns on Azure |
| Azure Security Best Practices | https://www.youtube.com/watch?v=UtfQOTVnBSo | Security and compliance |

---

## Questions & Answers (Lead Level)

### Azure Terminology & Services

**Q1: Explain IaaS, PaaS, SaaS. Give Azure examples.**

**A:** **IaaS:** You manage OS and above; provider gives VM, network, storage. Azure: VMs, VNet, Managed Disks. **PaaS:** You manage app and data; provider runs runtime and OS. Azure: App Service, AKS, Azure SQL, Functions. **SaaS:** Full application. Azure: Office 365, Dynamics. For microservices, PaaS (App Service, AKS, serverless) reduces ops.

**Q2: What are the main Azure compute options for microservices?**

**A:** **App Service:** Web apps, containers; easy scaling and slots. **AKS:** Kubernetes; full control, multi-container, scaling, ecosystem. **Container Apps:** Serverless containers; scale to zero, KEDA. **Azure Functions:** Serverless functions; event-driven. **ACI:** Run a container without managing clusters. Choice depends on orchestration needs, scale-to-zero, and team skills.

**Q3: What is Azure Container Registry (ACR)? How does it integrate with AKS?**

**A:** ACR is a private Docker registry in Azure. Store and version images; use with AKS by attaching ACR to the cluster (admin enable or K8s pull secret / managed identity). AKS nodes pull images from ACR; pipelines push after build. Use geo-replication and retention policies for production.

**Q4: How do you deploy microservices on Azure? What are the main patterns?**

**A:** **Options:** (1) **AKS:** Each microservice = Deployment + Service; use Ingress or Application Gateway. (2) **App Service:** One app per microservice; deployment slots, scaling. (3) **Container Apps:** Per-service containers, scale to zero. **Patterns:** API Management or App Gateway at edge; service mesh (optional) for mTLS and traffic; Azure Service Bus or Event Grid for async; Key Vault and Managed Identity for secrets and auth.

**Q5: What are the different ways to deploy to Azure (manual vs automated)?**

**A:** **Manual:** Portal, Azure CLI, PowerShell. **Infrastructure as Code:** ARM, Bicep, Terraform. **CI/CD:** Azure DevOps Pipelines, GitHub Actions; build → push to ACR → deploy to AKS/App Service. **GitOps:** Flux or Argo CD on AKS for Git as source of truth. Prefer IaC + pipelines for consistency and audit.

**Q6: Explain Azure networking: VNet, subnet, NSG, private endpoints.**

**A:** **VNet:** Isolated network in Azure; can peer with other VNets. **Subnet:** Segment within VNet for resources. **NSG:** Firewall rules (inbound/outbound) at subnet or NIC. **Private endpoint:** Private IP in your VNet for a PaaS service; traffic stays on Microsoft backbone, no public endpoint. Use for Azure SQL, Storage, ACR, etc., to reduce exposure.

**Q7: How do you secure Azure workloads (identity, network, data)?**

**A:** **Identity:** Managed Identity for apps (no secrets in code); Azure AD for users and app registration; RBAC and least privilege. **Network:** NSGs, private endpoints, disable public access where possible; WAF on Application Gateway. **Data:** Encryption at rest (default); TLS in transit; Key Vault for keys and secrets; sensitivity labels and audit logs. **Governance:** Azure Policy, Defender for Cloud, secure score.

**Q8: What is Azure API Management? When do you use it vs Ocelot?**

**A:** **API Management:** Managed API gateway with throttling, policies, developer portal, analytics, and multi-backend. Use for external/partner APIs and when you need a full API platform. **Ocelot:** Lightweight, config-driven gateway in your .NET app. Use for internal microservices when you want a simple, code-centric gateway. Can use both: APIM at edge, Ocelot or direct calls internally.

---

# 6. Cheat Sheets (Quick Reference)

## React & JavaScript Cheat Sheet

| Concept | One-liner |
|--------|-----------|
| Virtual DOM | In-memory copy of DOM; diff then patch for minimal updates |
| useEffect cleanup | Return function from useEffect; runs before next run and on unmount |
| useMemo vs useCallback | useMemo = memoized value; useCallback = memoized function |
| Context over-render | Split contexts; useMemo on value; or use subscription-based state |
| Event loop order | Sync → all micro-tasks (Promise) → one macro-task (setTimeout) → repeat |
| Closure | Function that “remembers” outer scope variables |
| this | Determined by call site: default, implicit, explicit (bind/call/apply), new |

---

## Database Cheat Sheet

| Concept | One-liner |
|--------|-----------|
| Clustered index | One per table; physical order of data; usually PK |
| Non-clustered | Separate structure; key + pointer; use for other search/sort columns |
| Covering index | Includes all columns needed by query; no key lookup |
| CTE | Named result set in single statement; can be recursive |
| Temp table | #table; session scope; has statistics; good for large intermediates |
| ROW_NUMBER | Unique per partition/order; top-N, dedup |
| RANK / DENSE_RANK | Same value same rank; RANK has gaps, DENSE_RANK doesn’t |
| Sharding | Split data across DBs by key; no cross-shard joins/transactions |
| DB per service | Each service owns its DB; communicate via API/events; eventual consistency |

---

## .NET & Microservices Cheat Sheet

| Concept | One-liner |
|--------|-----------|
| DI lifetimes | Transient (per resolve), Scoped (per request), Singleton (app lifetime) |
| Middleware order | Exception → HTTPS → Static → Routing → Auth → Endpoints |
| Sync vs async (services) | Sync: HTTP/gRPC for immediate response; Async: messaging for decoupling |
| API Gateway (Ocelot) | Single entry; routing, auth, load balancing; ocelot.json config |
| gRPC vs REST | gRPC: binary, HTTP/2, streaming, typed; REST: widely supported, human-readable |
| Circuit breaker | Stop calling failing service; retry after cooldown; use Polly |

---

## Docker & Kubernetes Cheat Sheet

| Concept | One-liner |
|--------|-----------|
| Dockerfile | FROM → WORKDIR → COPY → RUN → EXPOSE → CMD/ENTRYPOINT |
| Multi-stage | Multiple FROM; build in early stages; copy artifact to final small image |
| Image → K8s | Build image → push to registry → K8s pulls and runs via Deployment |
| Pod | Smallest unit; one or more containers; shared network/storage |
| Deployment | Desired state for Pods; replicas, image, rollout strategy |
| Service | Stable DNS + load balancing to Pods (ClusterIP, NodePort, LB) |
| Ingress | HTTP routing (host/path) to Services; TLS |

---

## Azure Cheat Sheet

| Term | One-liner |
|-----|-----------|
| IaaS | VM, network, storage; you manage OS and above |
| PaaS | Runtime managed; you bring app and data (e.g. App Service, AKS) |
| AKS | Managed Kubernetes in Azure; you manage nodes or use serverless |
| ACR | Private container registry; store images for AKS/pipelines |
| VNet | Isolated network; subnets, peering |
| NSG | Firewall rules for subnet/NIC |
| Private Endpoint | Private IP for PaaS; traffic stays on backbone |
| Managed Identity | No secrets; Azure AD identity for app to access Azure resources |
| Key Vault | Secrets, keys, certs; access via RBAC and policies |

---

## Interview Tips (Lead Level)

1. **Align with JD:** Emphasize Angular/React, .NET, APIs, SQL Server, Agile, security, scalability, and ownership. Mention experience with AI tools (e.g. ChatGPT) for productivity.
2. **Behavioral:** Prepare STAR examples: leading a technical decision, driving adoption of a pattern (e.g. microservices, CI/CD), handling production issues, mentoring.
3. **System design:** Be ready to sketch microservices for a domain (e.g. video platform like VIDIZMO): services, DB per service, API Gateway, events, scaling, security.
4. **Ask back:** Ask about their stack (Angular vs React ratio), how they use AI/LLMs in the product, deployment frequency, and how Lead engineers influence architecture.

---

*Good luck with your VIDIZMO Lead Engineer interview.*  
*Tailor examples to your 6+ years on the Microsoft stack and any experience with video, media, or data platforms.*
