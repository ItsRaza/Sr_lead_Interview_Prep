# VIDIZMO Interview - Quick Reference Guide

## MOST LIKELY INTERVIEW QUESTIONS (Based on Your HR Screening)

### 1. Session Management
**Q: How do you implement session management in a distributed system?**

**Answer Points:**
- JWT tokens for stateless authentication (recommended)
- Distributed session with Redis for stateful data
- Session data stored with TTL (Time To Live)
- Never use server memory for sessions in distributed systems

**Code Example:**
```csharp
// Distributed session with Redis
var userId = GetCurrentUserId();
var sessionKey = $"session:{userId}:cart";
await _redis.StringSetAsync(sessionKey, data, TimeSpan.FromMinutes(30));
```

---

### 2. HTTP Context
**Q: What is HTTP Context and how do you pass user context across microservices?**

**Answer:**
- HTTPContext contains request/response data for current HTTP request
- In microservices: Extract user info from JWT → Store in scoped service
- Each service gets user context via dependency injection

**Code:**
```csharp
public class UserContextMiddleware
{
    public async Task InvokeAsync(HttpContext context, IUserContextService userContext)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        userContext.SetContext(new UserContext { UserId = Guid.Parse(userId) });
        await _next(context);
    }
}
```

---

### 3. High Availability
**Q: How do you ensure 99.9% uptime?**

**Your Answer (based on Enterprise64 experience):**
- Multi-zone deployment (3 availability zones)
- Auto-scaling (3-10 instances based on CPU)
- Health checks (/health/live and /health/ready)
- Blue-green deployment with auto-rollback
- Database with geo-replication
- CDN for static content
- Circuit breaker pattern for failures

**Key Metric:** 99.9% = max 43.8 minutes downtime per month

---

### 4. Stateless vs Stateful
**Q: Explain the difference and when to use each.**

| Stateless | Stateful |
|-----------|----------|
| Session in Redis/JWT | Session in memory |
| Easy to scale | Hard to scale |
| Any server can handle request | Sticky sessions needed |
| **Use for:** APIs, microservices | **Use for:** Games, WebSockets |

**VIDIZMO Example:**
- Video Upload API: Stateless (JWT auth)
- Live Streaming: Stateful (WebSocket connection)

---

### 5. Static Functions
**Q: When should you use static methods?**

**Use Static:**
- ✅ Utility functions (no state needed)
- ✅ Helper methods (ValidateEmail, GenerateThumbnailUrl)
- ✅ Constants

**Don't Use Static:**
- ❌ Database access (breaks DI, hard to test)
- ❌ Business logic (need instance state)

**Example:**
```csharp
// GOOD
public static class VideoHelper
{
    public static bool IsValidFormat(string extension)
    {
        return new[] { ".mp4", ".avi" }.Contains(extension);
    }
}

// BAD
public static class VideoRepository
{
    private static SqlConnection _connection; // Hard to test!
}
```

---

### 6. Database Indexing
**Q: Your query is slow on a 10 billion row table. How do you fix it?**

**Your Approach:**
1. **Identify slow queries** - Use SQL Server DMVs
2. **Add indexes** - Covering index on WHERE/ORDER BY columns
3. **Partitioning** - Partition by date (most common filter)
4. **Columnstore** - For analytics queries
5. **Archiving** - Move old data to archive table

**Example:**
```sql
-- Before: Full table scan (slow)
SELECT * FROM VideoViews WHERE VideoId = @Id;

-- After: Covering index (fast)
CREATE INDEX IX_VideoViews_VideoId_ViewedAt 
ON VideoViews (VideoId, ViewedAt DESC) 
INCLUDE (WatchDuration);
```

---

### 7. Sharding
**Q: How do you shard a database with 100M videos?**

**Strategy:**
- Shard by VideoId hash (even distribution)
- 32-64 shards (to handle scale)
- Use consistent hashing (easier rebalancing)
- Global secondary index for cross-shard queries

**Example:**
```csharp
public string GetVideoShard(Guid videoId)
{
    var hash = videoId.GetHashCode();
    var shardNumber = Math.Abs(hash % 32); // 32 shards
    return $"VideoShard{shardNumber}";
}
```

---

### 8. Replication
**Q: Explain master-slave replication.**

**Answer:**
- **Master (Primary):** Handles all WRITES
- **Slaves (Replicas):** Handle READS only
- Async replication: Slight lag (1-5 seconds)
- Use case: Offload reporting queries to replicas

**Connection Routing:**
```csharp
public SqlConnection GetConnection(QueryType type)
{
    return type == QueryType.Write 
        ? new SqlConnection(_primaryConnection)
        : new SqlConnection(_replicaConnection);
}
```

---

### 9. Cloud Architecture
**Q: Design a multi-region deployment.**

**Your Architecture:**
```
Primary Region: East US (Active)
├── App Service (10 instances)
├── SQL Database (Primary)
└── Redis Cache

Secondary Region: West US (Warm Standby)
├── App Service (2 instances)
├── SQL Database (Replica)
└── Redis Cache

Failover: Promote secondary in 15 minutes
```

---

### 10. RabbitMQ vs Kafka
**Q: When to use RabbitMQ vs Kafka?**

**RabbitMQ:**
- Task queues (video processing)
- Complex routing
- Slower throughput (10K-20K msg/sec)

**Kafka:**
- Event streaming (analytics)
- High throughput (100K-1M msg/sec)
- Event replay needed

**VIDIZMO Use Case:**
- RabbitMQ: Video transcoding queue
- Kafka: User analytics stream

---

## SCENARIO-BASED QUESTIONS (Most Common)

### Scenario 1: Video Upload Failure
**Q: User's 2GB video upload fails at 80%. How do you handle this?**

**Your Answer:**
1. **Chunked Upload** - Upload in 10 MB chunks
2. **Resume Capability** - Track uploaded chunks in Redis
3. **Retry Logic** - Exponential backoff
4. **Storage** - Direct upload to blob storage (presigned URL)

```csharp
// Track progress in Redis
var uploadState = new
{
    UploadId = Guid.NewGuid(),
    TotalChunks = 200,
    CompletedChunks = 160 // 80% done
};
await _redis.SetAsync($"upload:{userId}:{uploadId}", uploadState);
```

---

### Scenario 2: Database Performance
**Q: Your analytics query takes 30 seconds on 10B rows. Fix it.**

**Your Solution:**
1. **Partitioning** by date
2. **Columnstore index** for analytics
3. **Pre-aggregation** - Hourly summary tables
4. **Caching** - Redis for frequent queries

**Result:** 30 seconds → 200ms (99.3% improvement)

---

### Scenario 3: High Traffic Spike
**Q: Black Friday - 10x traffic surge. How do you handle it?**

**Your Plan:**
1. **Auto-scaling** - 10 → 100 instances
2. **CDN** - Offload static content
3. **Caching** - Aggressive Redis caching
4. **Read Replicas** - 3 read replicas for DB
5. **Rate Limiting** - Protect backend
6. **Queue** - Queue non-critical tasks

---

### Scenario 4: Microservice Failure
**Q: Your payment service is down. How do you prevent cascading failure?**

**Your Implementation:**
1. **Circuit Breaker** - Stop calling failed service
2. **Timeout** - 3-5 second max
3. **Fallback** - Degrade gracefully
4. **Bulkhead** - Isolate failure
5. **Retry** - Exponential backoff

```csharp
var policy = Policy
    .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
    .CircuitBreakerAsync(3, TimeSpan.FromSeconds(30));
```

---

## KEY METRICS TO REMEMBER

**Performance:**
- API Response: <100ms (P95)
- Cache Hit Ratio: >90%
- Database Query: <50ms

**Availability:**
- 99.9% = 43.8 min/month downtime
- 99.95% = 21.9 min/month
- 99.99% = 4.38 min/month

**Scalability:**
- Redis: 100K ops/sec
- Kafka: 1M msg/sec
- SQL Server: 10K-50K queries/sec

---

## YOUR EXPERIENCE HIGHLIGHTS

**When they ask about your experience, mention:**

1. **Enterprise64 (Current):**
   - Modernized legacy systems → 55-60% efficiency gain
   - Led 10-12 member team
   - Selenium QA automation
   - Microservices architecture

2. **DigiTrends:**
   - 75% performance improvement on B2B platform
   - SAP integrations
   - Microservices + micro-frontends
   - Azure Cloud deployment

3. **Info Access Solutions:**
   - Multi-tenant ERP for schools
   - Led 10-member team
   - Payment gateway integrations (HBL, Alfalah)
   - Expanded to Bahrain and Oman

---

## COMMON MISTAKES TO AVOID

❌ **Don't say:**
- "I don't know"
- "We always used X" (show flexibility)
- Over-engineer (keep it simple first)

✅ **Do say:**
- "Here's how I'd approach this..."
- "Trade-off is X vs Y"
- "In my experience at [company]..."

---

## RED FLAGS FOR INTERVIEWER

**They're looking for:**
- Clear communication (can you explain complex topics simply?)
- Problem-solving (not just memorization)
- Real-world experience (not just theory)
- Trade-off analysis (understanding pros/cons)
- Ownership mindset (can you drive projects?)

---

## 5-MINUTE PREP BEFORE INTERVIEW

1. Review VIDIZMO's product (https://www.vidizmo.com)
2. Know their AI features (auto-tagging, OCR, transcription)
3. Prepare 3 STAR stories from your experience
4. Have pen and paper ready (draw diagrams!)
5. Test your video/audio

---

## QUESTIONS TO ASK THEM

1. "What's the current architecture of the video platform?"
2. "What are the biggest technical challenges you're facing?"
3. "How do you handle video processing at scale?"
4. "What's the team structure for this role?"
5. "What does success look like in the first 3-6 months?"

---

## ONE-LINERS FOR TOUGH QUESTIONS

**"Tell me about yourself"**
→ "I'm a Senior Software Engineer with 6+ years building scalable enterprise systems. At Enterprise64, I led modernization of warehouse management systems achieving 55% performance improvement. I specialize in microservices, cloud architecture, and distributed systems - exactly what VIDIZMO needs for its AI-powered video platform."

**"Why VIDIZMO?"**
→ "VIDIZMO combines three things I'm passionate about: microservices architecture, AI/ML integration, and enterprise-scale systems. My experience modernizing legacy systems at Enterprise64 and building B2B platforms at DigiTrends aligns perfectly with VIDIZMO's technical challenges. Plus, I'm excited about working with cutting-edge AI features like auto-tagging and OCR."

**"What's your biggest weakness?"**
→ "I sometimes dive too deep into solving problems, which can delay initial estimates. I've learned to set time-boxes and reassess priorities regularly. For example, at DigiTrends, I initially spent 3 days optimizing a query that impacted 5 users, when I should've focused on the main feature first."

---

## FINAL CONFIDENCE BOOSTERS

✅ You have **6+ years** relevant experience  
✅ You've worked with **microservices, cloud, distributed systems**  
✅ You've **led teams** and delivered enterprise projects  
✅ You've achieved **measurable results** (55-75% improvements)  
✅ You've worked in **US and Gulf markets** (relevant for VIDIZMO)  

**You're qualified. You're prepared. Go ace this interview!**
