# Senior .NET Developer Interview Preparation Guide
### Bank Al Habib | Enterprise-Grade Applications | Lead Position Scenarios

---

> **How to use this guide:** Questions are grouped by topic and difficulty. Each section progresses from **Basic → Intermediate → Advanced → Scenario-Based**. Lead-level scenarios are marked with 🏆.

---

## Table of Contents

1. [.NET Framework](#1-net-framework)
2. [.NET Core / .NET 5+](#2-net-core--net-5)
3. [C# Language Deep Dive](#3-c-language-deep-dive)
4. [ASP.NET Core Web API](#4-aspnet-core-web-api)
5. [Entity Framework Core](#5-entity-framework-core)
6. [SQL Server & Databases](#6-sql-server--databases)
7. [IBM Integration Bus (IIB) / IBM ACE](#7-ibm-integration-bus-iib--ibm-ace)
8. [MQ & Messaging](#8-mq--messaging)
9. [Design Patterns & Architecture](#9-design-patterns--architecture)
10. [Security in Enterprise Applications](#10-security-in-enterprise-applications)
11. [Performance & Scalability](#11-performance--scalability)
12. [Lead-Level Scenario Questions 🏆](#12-lead-level-scenario-questions-)

---

## 1. .NET Framework

### Basic

**Q1: What is the .NET Framework and what are its main components?**

**A:** The .NET Framework is a software development platform by Microsoft. Its main components are:
- **CLR (Common Language Runtime):** Manages execution of .NET programs, handles memory management (GC), exception handling, and type safety.
- **FCL (Framework Class Library):** A large collection of reusable classes, interfaces, and value types.
- **CTS (Common Type System):** Defines how types are declared and used in the runtime.
- **CLS (Common Language Specification):** A set of rules that languages must follow to ensure interoperability.

---

**Q2: What is the difference between Value Types and Reference Types?**

**A:**

| Feature | Value Type | Reference Type |
|---|---|---|
| Storage | Stack | Heap |
| Examples | `int`, `float`, `struct`, `enum` | `class`, `string`, `interface`, `delegate` |
| Assignment | Copies the value | Copies the reference |
| Default value | Zero / false | null |
| Nullable | Via `Nullable<T>` or `?` | Naturally nullable |

```csharp
int a = 10;
int b = a;   // b is an independent copy
b = 20;      // a is still 10

var obj1 = new Person { Name = "Ali" };
var obj2 = obj1;   // Both point to same object
obj2.Name = "Zara"; // obj1.Name is also "Zara" now
```

---

**Q3: What is Garbage Collection? How does it work?**

**A:** The GC automatically manages memory. It works in generations:
- **Gen 0:** Short-lived objects (most objects start here). Collected frequently.
- **Gen 1:** Objects that survived Gen 0. Buffer between short and long-lived.
- **Gen 2:** Long-lived objects (static data, large objects). Collected infrequently.

The **Large Object Heap (LOH)** holds objects > 85KB and is only collected during Gen 2 GC.

Best practices:
- Implement `IDisposable` for unmanaged resources.
- Use `using` statements to ensure `Dispose()` is called.
- Avoid finalizers unless necessary — they delay GC.

---

### Intermediate

**Q4: What is the difference between `IDisposable` and a Finalizer (Destructor)?**

**A:**
- **`IDisposable` / `Dispose()`:** Deterministic cleanup. Called explicitly by the developer (or via `using`). Used for managed AND unmanaged resources.
- **Finalizer (~ClassName):** Non-deterministic cleanup. Called by GC before collecting the object. Only for unmanaged resources. Adds overhead — avoid unless absolutely needed.

Full pattern:
```csharp
public class ResourceHolder : IDisposable
{
    private bool _disposed = false;
    private IntPtr _unmanagedResource;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this); // Prevent finalizer from running
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Free managed resources
            }
            // Free unmanaged resources
            _disposed = true;
        }
    }

    ~ResourceHolder() => Dispose(false); // Finalizer as safety net
}
```

---

**Q5: What is the difference between `==` and `.Equals()` in C#?**

**A:**
- For **value types**, both compare by value.
- For **reference types**, `==` compares references (memory address) by default. `.Equals()` can be overridden to compare by value (e.g., `string`).
- `string` overrides both `==` and `.Equals()` to do value comparison.

```csharp
string s1 = new string("hello".ToCharArray());
string s2 = new string("hello".ToCharArray());

Console.WriteLine(s1 == s2);        // True (string overrides ==)
Console.WriteLine(s1.Equals(s2));   // True
Console.WriteLine(ReferenceEquals(s1, s2)); // False (different objects)
```

---

### Advanced

**Q6: Explain AppDomain, and how assembly loading works in .NET Framework.**

**A:** An **AppDomain** is an isolation boundary within a process. Multiple AppDomains can run in a single process, each with its own heap and security context. Used for plugin architectures and fault isolation.

Assembly loading order:
1. Check GAC (Global Assembly Cache).
2. Check application's `bin` directory.
3. Check `probing` paths in config.
4. Fire `AssemblyResolve` event if not found.

> **Note:** AppDomains were removed in .NET Core in favor of process-level isolation and `AssemblyLoadContext`.

---

## 2. .NET Core / .NET 5+

### Basic

**Q7: What are the key differences between .NET Framework and .NET Core?**

**A:**

| Feature | .NET Framework | .NET Core / .NET 5+ |
|---|---|---|
| Platform | Windows only | Cross-platform |
| Deployment | GAC / machine-wide | Self-contained / side-by-side |
| Performance | Moderate | Significantly faster |
| Open Source | Partially | Fully open source |
| Web | ASP.NET (System.Web) | ASP.NET Core (no System.Web) |
| AppDomain | Supported | Removed |
| WCF Server | Full support | Community port only |

---

**Q8: What is the Generic Host in .NET Core?**

**A:** The Generic Host (`IHost`) is the foundation for all .NET applications — web and non-web. It manages:
- **Dependency Injection** container
- **Configuration** (appsettings, env vars, etc.)
- **Logging**
- **Hosted Services** (`IHostedService` / `BackgroundService`)
- **Application lifetime** (start/stop signals)

```csharp
var host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        services.AddHostedService<PaymentProcessorService>();
        services.AddDbContext<AppDbContext>(...);
    })
    .Build();

await host.RunAsync();
```

---

**Q9: Explain Dependency Injection lifetimes: Transient, Scoped, Singleton.**

**A:**

| Lifetime | Created | Destroyed | Use case |
|---|---|---|---|
| **Transient** | Every time it's requested | When consumer is done | Stateless services, utilities |
| **Scoped** | Once per HTTP request (or scope) | End of request/scope | DbContext, Unit of Work |
| **Singleton** | Once per application lifetime | App shutdown | Configuration, Caches, Shared state |

**Captive Dependency Problem:** Never inject a `Scoped` or `Transient` service into a `Singleton`. The shorter-lived dependency will be held alive for the singleton's lifetime, causing stale data or threading issues.

```csharp
services.AddTransient<IEmailService, EmailService>();
services.AddScoped<IUnitOfWork, UnitOfWork>();
services.AddSingleton<ICacheService, RedisCacheService>();
```

---

### Intermediate

**Q10: How does the ASP.NET Core middleware pipeline work?**

**A:** Middleware is a chain of components that each process an HTTP request and response. Each component can:
- Execute code before and after the next component.
- Short-circuit the pipeline (not call `next()`).

Order matters critically:
```csharp
app.UseExceptionHandler("/error");   // Must be first
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();             // Before Authorization
app.UseAuthorization();
app.UseCustomLogging();              // Custom middleware
app.MapControllers();
```

Custom middleware:
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Request: {Method} {Path}", 
            context.Request.Method, context.Request.Path);
        
        await _next(context); // Call next middleware
        
        _logger.LogInformation("Response: {StatusCode}", 
            context.Response.StatusCode);
    }
}
```

---

**Q11: What is `IHostedService` and `BackgroundService`? When would you use them in a bank?**

**A:**
- `IHostedService` defines `StartAsync` and `StopAsync` for background work.
- `BackgroundService` is an abstract class that simplifies this with a `ExecuteAsync` method.

**Banking use cases:**
- Polling a database queue for pending payments to process.
- Consuming messages from IBM MQ continuously.
- Running nightly interest calculation jobs.
- Sending scheduled statement emails.

```csharp
public class MQListenerService : BackgroundService
{
    private readonly IMQService _mqService;
    private readonly ILogger<MQListenerService> _logger;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                var message = await _mqService.GetMessageAsync();
                if (message != null)
                    await ProcessPaymentAsync(message);
                else
                    await Task.Delay(500, stoppingToken); // Avoid tight loop
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error processing MQ message");
                await Task.Delay(5000, stoppingToken); // Back-off on error
            }
        }
    }
}
```

---

### Advanced

**Q12: How does `async`/`await` work under the hood? What is a SynchronizationContext?**

**A:** `async`/`await` is syntactic sugar. The compiler transforms an async method into a **state machine**. When `await` is hit on an incomplete task:
1. The current state is saved.
2. Control returns to the caller.
3. When the awaited task completes, the state machine resumes.

**SynchronizationContext** controls how continuations are scheduled. In ASP.NET Core there is **no SynchronizationContext**, so continuations run on thread pool threads — this is why `.ConfigureAwait(false)` is less critical in ASP.NET Core but still best practice in library code.

**Common mistakes:**
```csharp
// DEADLOCK in UI or ASP.NET Classic — DON'T do this:
var result = GetDataAsync().Result;

// CORRECT — use async all the way:
var result = await GetDataAsync();

// In library code, use ConfigureAwait(false):
var data = await _repo.GetAsync().ConfigureAwait(false);
```

---

**Q13: What is the difference between `Task`, `Task<T>`, `ValueTask`, and `ValueTask<T>`?**

**A:**
- **`Task`/`Task<T>`:** Reference types, always allocated on heap. Best for operations that are genuinely asynchronous.
- **`ValueTask`/`ValueTask<T>`:** Struct-based, avoids heap allocation when the result is available synchronously (common in caching scenarios). Should NOT be awaited multiple times.

```csharp
// Use ValueTask when result is often available synchronously
public ValueTask<User> GetUserAsync(int id)
{
    if (_cache.TryGetValue(id, out var user))
        return new ValueTask<User>(user); // No allocation!
    
    return new ValueTask<User>(FetchFromDbAsync(id));
}
```

---

## 3. C# Language Deep Dive

### Basic

**Q14: What are `ref`, `out`, and `in` parameter modifiers?**

**A:**
- **`ref`:** Caller must initialize. Both caller and method can read/write.
- **`out`:** Caller doesn't need to initialize. Method MUST assign before returning.
- **`in`:** Passes by reference but read-only. Avoids copying large structs.

```csharp
void Process(ref int balance, out string message, in TransactionLimit limit)
{
    balance -= 100;          // Modifies caller's variable
    message = "Processed";   // Must assign out param
    // limit.Amount = 999;   // Compile error — in is read-only
}
```

---

**Q15: What are delegates, events, and how do they relate to Func/Action/Predicate?**

**A:**
- **Delegate:** A type-safe function pointer. Can hold references to methods.
- **Event:** A delegate with publish/subscribe semantics. Only the declaring class can invoke it.
- **`Func<T, TResult>`:** Built-in delegate for methods that return a value.
- **`Action<T>`:** Built-in delegate for void methods.
- **`Predicate<T>`:** Built-in delegate that returns bool (equivalent to `Func<T, bool>`).

```csharp
// Custom delegate
public delegate decimal CalculateFeeDelegate(decimal amount);

// Using Func
Func<decimal, decimal> calcFee = amount => amount * 0.02m;

// Event pattern
public class PaymentProcessor
{
    public event EventHandler<PaymentEventArgs> PaymentProcessed;

    protected virtual void OnPaymentProcessed(PaymentEventArgs e)
        => PaymentProcessed?.Invoke(this, e);
}
```

---

### Intermediate

**Q16: Explain LINQ — deferred vs immediate execution.**

**A:**
- **Deferred execution:** Query is defined but NOT executed until iterated (`foreach`, `ToList()`, `Count()`, etc.). Uses `IEnumerable<T>`.
- **Immediate execution:** Operators like `ToList()`, `ToArray()`, `First()`, `Count()` force execution immediately.

```csharp
var query = accounts
    .Where(a => a.Balance > 10000)    // Deferred
    .OrderByDescending(a => a.Balance); // Deferred

// Query executes HERE
foreach (var account in query) { ... }

// Or force immediately
var list = query.ToList(); // Executes once, stores in memory
```

**Common pitfall:** Multiple enumeration of a deferred query hits the database multiple times. Always materialize with `.ToList()` if you need to iterate more than once.

---

**Q17: What are Expression Trees and how are they used by Entity Framework?**

**A:** Expression trees represent code as data — a tree of objects describing the structure of a lambda expression. EF Core uses them to translate LINQ queries to SQL.

```csharp
// This is an Expression<Func<T>> — NOT compiled code, but a tree
Expression<Func<Account, bool>> filter = a => a.Balance > 10000;

// EF Core reads the tree and generates: WHERE Balance > 10000
var accounts = await _context.Accounts.Where(filter).ToListAsync();

// Func<T> is compiled code — cannot be translated to SQL
Func<Account, bool> func = a => a.Balance > 10000;
var local = accounts.Where(func); // Works only in-memory (LINQ to Objects)
```

---

### Advanced

**Q18: What are `Span<T>` and `Memory<T>` and when should you use them?**

**A:** `Span<T>` is a stack-only, ref struct that provides a view over contiguous memory (array, stack allocation, or unmanaged) **without allocation**. `Memory<T>` is the heap-compatible version usable in async methods.

**Banking use case:** Parsing fixed-width transaction files from mainframes without allocating strings for each field.

```csharp
// Parse a fixed-width banking record without allocating substrings
public static TransactionRecord Parse(ReadOnlySpan<char> line)
{
    var accountId = line.Slice(0, 10).ToString();
    var amount = decimal.Parse(line.Slice(10, 12));
    var currency = line.Slice(22, 3).ToString();
    return new TransactionRecord(accountId, amount, currency);
}
```

---

## 4. ASP.NET Core Web API

### Basic

**Q19: What is the difference between `[FromBody]`, `[FromQuery]`, `[FromRoute]`, `[FromHeader]`?**

**A:**
- `[FromBody]`: Reads from request body (JSON/XML). One per action.
- `[FromQuery]`: Reads from query string (`?key=value`).
- `[FromRoute]`: Reads from URL route template (`/api/accounts/{id}`).
- `[FromHeader]`: Reads from HTTP headers.

```csharp
[HttpPost("transfer")]
public async Task<IActionResult> Transfer(
    [FromHeader(Name = "X-Correlation-Id")] string correlationId,
    [FromQuery] string currency,
    [FromBody] TransferRequest request)
{ ... }
```

---

**Q20: What is the difference between `IActionResult` and `ActionResult<T>`?**

**A:**
- `IActionResult`: Generic, returns any HTTP response. No type information for Swagger.
- `ActionResult<T>`: Typed. Swagger can generate accurate response schemas. Supports implicit casting from T.

```csharp
// Preferred for Swagger/OpenAPI documentation
[HttpGet("{id}")]
[ProducesResponseType(typeof(AccountDto), 200)]
[ProducesResponseType(404)]
public async Task<ActionResult<AccountDto>> GetAccount(int id)
{
    var account = await _service.GetByIdAsync(id);
    if (account == null) return NotFound();
    return account; // Implicit cast to ActionResult<AccountDto>
}
```

---

### Intermediate

**Q21: How do you implement global exception handling in ASP.NET Core?**

**A:** Use a combination of middleware and `ProblemDetails`:

```csharp
// Program.cs
app.UseExceptionHandler(appError =>
{
    appError.Run(async context =>
    {
        context.Response.ContentType = "application/json";
        var feature = context.Features.Get<IExceptionHandlerFeature>();
        var ex = feature?.Error;

        var (statusCode, message) = ex switch
        {
            NotFoundException => (404, ex.Message),
            ValidationException => (400, ex.Message),
            UnauthorizedException => (401, ex.Message),
            _ => (500, "An internal error occurred")
        };

        context.Response.StatusCode = statusCode;
        await context.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Status = statusCode,
            Title = message,
            Instance = context.Request.Path,
            Extensions = { ["correlationId"] = context.TraceIdentifier }
        });
    });
});
```

---

**Q22: How do you implement API versioning in an enterprise banking API?**

**A:**
```csharp
// Install: Microsoft.AspNetCore.Mvc.Versioning

services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new HeaderApiVersionReader("X-API-Version"),
        new QueryStringApiVersionReader("api-version"),
        new UrlSegmentApiVersionReader()
    );
});

[ApiController]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class AccountsController : ControllerBase
{
    [HttpGet("{id}")]
    [MapToApiVersion("1.0")]
    public Task<AccountV1Dto> GetV1(int id) => ...;

    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public Task<AccountV2Dto> GetV2(int id) => ...;
}
```

---

### Advanced

**Q23: How do you implement Idempotency in a payment API?**

**A:** Idempotency ensures that retrying the same request doesn't cause duplicate operations — critical in banking.

```csharp
public class IdempotencyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IDistributedCache _cache;

    public async Task InvokeAsync(HttpContext context)
    {
        if (!context.Request.Headers.TryGetValue("Idempotency-Key", out var key))
        {
            await _next(context);
            return;
        }

        var cacheKey = $"idempotency:{key}";
        var cached = await _cache.GetStringAsync(cacheKey);
        if (cached != null)
        {
            // Return cached response — same result, no double processing
            context.Response.StatusCode = 200;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsync(cached);
            return;
        }

        // Capture response
        var originalBody = context.Response.Body;
        using var buffer = new MemoryStream();
        context.Response.Body = buffer;

        await _next(context);

        buffer.Seek(0, SeekOrigin.Begin);
        var responseBody = await new StreamReader(buffer).ReadToEndAsync();

        // Cache for 24 hours
        await _cache.SetStringAsync(cacheKey, responseBody, 
            new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(24) });

        buffer.Seek(0, SeekOrigin.Begin);
        await buffer.CopyToAsync(originalBody);
    }
}
```

---

## 5. Entity Framework Core

### Basic

**Q24: What is the difference between `Add`, `Attach`, `Update`, and `Entry` in EF Core?**

**A:**
- **`Add`:** Marks entity as `Added`. EF will INSERT it.
- **`Attach`:** Marks entity as `Unchanged`. EF tracks it but won't INSERT or UPDATE.
- **`Update`:** Marks ALL properties as `Modified`. EF will UPDATE entire row.
- **`Entry(entity).Property(x).IsModified = true`:** Marks specific properties for UPDATE. More efficient.

```csharp
// Efficient partial update — only updates Balance
var account = new Account { Id = 1, Balance = 5000 };
_context.Attach(account);
_context.Entry(account).Property(a => a.Balance).IsModified = true;
await _context.SaveChangesAsync();
// SQL: UPDATE Accounts SET Balance = 5000 WHERE Id = 1
```

---

**Q25: What is the N+1 problem and how do you fix it?**

**A:** N+1 occurs when you query for N entities and then make an additional query for each entity's related data — N+1 total database hits.

```csharp
// BAD — N+1 problem:
var accounts = await _context.Accounts.ToListAsync(); // 1 query
foreach (var acc in accounts)
{
    var txns = acc.Transactions; // N queries (lazy load)
}

// GOOD — Eager loading:
var accounts = await _context.Accounts
    .Include(a => a.Transactions)
    .ToListAsync(); // 1 query with JOIN

// BEST — Explicit projection (most efficient):
var result = await _context.Accounts
    .Select(a => new AccountDto
    {
        Id = a.Id,
        Balance = a.Balance,
        TransactionCount = a.Transactions.Count()
    }).ToListAsync();
```

---

### Intermediate

**Q26: How do you handle database transactions in EF Core?**

**A:**
```csharp
// EF Core wraps SaveChangesAsync in a transaction automatically.
// For multi-step operations spanning multiple SaveChanges:

public async Task TransferFundsAsync(int fromId, int toId, decimal amount)
{
    await using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        var from = await _context.Accounts.FindAsync(fromId);
        var to = await _context.Accounts.FindAsync(toId);

        if (from.Balance < amount)
            throw new InsufficientFundsException();

        from.Balance -= amount;
        to.Balance += amount;

        await _context.SaveChangesAsync();

        await _context.AuditLogs.AddAsync(new AuditLog
        {
            Action = "TRANSFER",
            Amount = amount,
            Timestamp = DateTime.UtcNow
        });
        await _context.SaveChangesAsync();

        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

---

**Q27: What is the difference between optimistic and pessimistic concurrency in EF Core?**

**A:**
- **Optimistic:** Assume conflicts are rare. Detect conflicts at save time using a `RowVersion`/`Timestamp` column. Throws `DbUpdateConcurrencyException` on conflict.
- **Pessimistic:** Lock the row during the entire operation (`SELECT FOR UPDATE`). Prevents concurrent access. Supported via raw SQL or `IDbContextTransaction`.

```csharp
// Optimistic Concurrency with RowVersion
public class Account
{
    public int Id { get; set; }
    public decimal Balance { get; set; }

    [Timestamp]  // EF adds WHERE RowVersion = @original in UPDATE
    public byte[] RowVersion { get; set; }
}

try
{
    await _context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    var entry = ex.Entries.Single();
    var dbValues = await entry.GetDatabaseValuesAsync();
    // Resolve conflict: client wins, db wins, or merge
    entry.OriginalValues.SetValues(dbValues);
    await _context.SaveChangesAsync(); // Retry
}
```

---

### Advanced

**Q28: How do you use EF Core with stored procedures and raw SQL safely?**

**A:**
```csharp
// Executing a stored procedure
var result = await _context.Accounts
    .FromSqlRaw("EXEC sp_GetAccountSummary @AccountId", 
        new SqlParameter("@AccountId", accountId))
    .ToListAsync();

// Parameterized DML (safe from SQL injection)
await _context.Database.ExecuteSqlRawAsync(
    "UPDATE Accounts SET Status = @p0 WHERE Id = @p1",
    "FROZEN", accountId);

// Using Interpolated SQL (automatically parameterized)
await _context.Database.ExecuteSqlInterpolatedAsync(
    $"UPDATE Accounts SET Status = {"FROZEN"} WHERE Id = {accountId}");
```

---

## 6. SQL Server & Databases

### Basic

**Q29: What is the difference between clustered and non-clustered indexes?**

**A:**
- **Clustered Index:** Physically reorders the table data. One per table. By default on Primary Key. Leaf pages ARE the data.
- **Non-Clustered Index:** A separate structure pointing to the data rows via a row locator. Up to 999 per table. Leaf pages contain the indexed columns + pointer to the data row.

```sql
-- Table physically sorted by AccountId
CREATE CLUSTERED INDEX IX_Accounts_Id ON Accounts(AccountId);

-- Separate structure for fast lookups by CNIC
CREATE NONCLUSTERED INDEX IX_Accounts_CNIC 
ON Accounts(CNIC) 
INCLUDE (AccountNumber, Balance);  -- Covering index avoids key lookup
```

---

**Q30: What are the SQL Server isolation levels?**

**A:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed (default) | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible |
| Serializable | Prevented | Prevented | Prevented |
| Snapshot | Prevented | Prevented | Prevented (uses versioning) |

**Banking:** Use **Snapshot Isolation** or **Serializable** for financial transactions to prevent phantom reads.

```sql
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
    SELECT Balance FROM Accounts WHERE AccountId = 1001;
    UPDATE Accounts SET Balance = Balance - 500 WHERE AccountId = 1001;
COMMIT;
```

---

### Intermediate

**Q31: How do you optimize a slow query in SQL Server?**

**A:** Step-by-step approach:
1. Check Execution Plan (`SET STATISTICS IO ON; SET STATISTICS TIME ON;`)
2. Look for **Table Scans** vs **Index Seeks**
3. Check for **missing indexes** (green hints in execution plan)
4. Check for **parameter sniffing** issues
5. Analyze **key lookups** — add `INCLUDE` columns to covering index
6. Check for **implicit conversions** (mismatched data types)

```sql
-- Find slow queries via DMVs
SELECT TOP 10
    qs.total_elapsed_time / qs.execution_count AS avg_elapsed_time,
    qs.execution_count,
    SUBSTRING(qt.text, qs.statement_start_offset/2 + 1, 
        (CASE WHEN qs.statement_end_offset = -1
            THEN LEN(CONVERT(NVARCHAR(MAX), qt.text)) * 2
            ELSE qs.statement_end_offset END - qs.statement_start_offset)/2) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY avg_elapsed_time DESC;

-- Fix parameter sniffing
CREATE PROCEDURE sp_GetCustomerTransactions
    @CustomerId INT,
    @StartDate DATE
AS
BEGIN
    DECLARE @LocalCustomerId INT = @CustomerId;
    DECLARE @LocalStartDate DATE = @StartDate;

    SELECT * FROM Transactions 
    WHERE CustomerId = @LocalCustomerId 
    AND TransactionDate >= @LocalStartDate
    OPTION (RECOMPILE);
END
```

---

**Q32: What are CTEs and Window Functions? Give a banking example.**

**A:**
```sql
-- CTE: Running balance for an account
WITH RunningBalance AS (
    SELECT
        TransactionId,
        TransactionDate,
        Amount,
        TransactionType,
        SUM(CASE WHEN TransactionType = 'CR' THEN Amount ELSE -Amount END)
            OVER (
                PARTITION BY AccountId 
                ORDER BY TransactionDate, TransactionId
                ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
            ) AS RunningBalance
    FROM Transactions
    WHERE AccountId = 1001
)
SELECT * FROM RunningBalance ORDER BY TransactionDate;

-- Rank customers by total deposits in last 30 days
SELECT
    CustomerId,
    TotalDeposits,
    RANK() OVER (ORDER BY TotalDeposits DESC) AS Rank,
    NTILE(4) OVER (ORDER BY TotalDeposits DESC) AS Quartile,
    LAG(TotalDeposits) OVER (ORDER BY TotalDeposits DESC) AS PrevCustomerDeposits
FROM (
    SELECT CustomerId, SUM(Amount) AS TotalDeposits
    FROM Transactions
    WHERE TransactionType = 'CR' 
    AND TransactionDate >= DATEADD(DAY, -30, GETDATE())
    GROUP BY CustomerId
) t;
```

---

### Advanced

**Q33: How do you implement audit logging in SQL Server for a banking system?**

**A:**
```sql
-- Option 1: Temporal Tables (SQL Server 2016+) — Best for banking
CREATE TABLE Accounts
(
    AccountId INT PRIMARY KEY,
    Balance DECIMAL(18,2),
    Status VARCHAR(20),
    ModifiedBy VARCHAR(100),
    ValidFrom DATETIME2 GENERATED ALWAYS AS ROW START,
    ValidTo   DATETIME2 GENERATED ALWAYS AS ROW END,
    PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
)
WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.AccountsHistory));

-- Query historical state at any point in time
SELECT * FROM Accounts 
FOR SYSTEM_TIME AS OF '2024-01-15 10:00:00'
WHERE AccountId = 1001;

-- Option 2: Trigger-based audit
CREATE TRIGGER tr_Accounts_Audit
ON Accounts
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    INSERT INTO AuditLog (TableName, Operation, OldData, NewData, ChangedBy, ChangedAt)
    SELECT
        'Accounts',
        CASE 
            WHEN EXISTS(SELECT 1 FROM inserted) AND EXISTS(SELECT 1 FROM deleted) THEN 'UPDATE'
            WHEN EXISTS(SELECT 1 FROM inserted) THEN 'INSERT'
            ELSE 'DELETE'
        END,
        (SELECT * FROM deleted FOR JSON PATH),
        (SELECT * FROM inserted FOR JSON PATH),
        SYSTEM_USER,
        GETUTCDATE();
END;
```

---

**Q34: Explain deadlock detection and prevention in SQL Server.**

**A:**
```sql
-- Detect recent deadlocks from system health extended events
SELECT 
    xdr.value('@timestamp', 'datetime2') AS deadlock_time,
    xdr.query('.') AS deadlock_graph
FROM (
    SELECT CAST(target_data AS XML) AS target_data
    FROM sys.dm_xe_session_targets t
    JOIN sys.dm_xe_sessions s ON t.event_session_address = s.address
    WHERE s.name = 'system_health' AND t.target_name = 'ring_buffer'
) AS data
CROSS APPLY target_data.nodes('//RingBufferTarget/event[@name="xml_deadlock_report"]') AS xdt(xdr);

-- Prevention strategies:
-- 1. Use READ COMMITTED SNAPSHOT ISOLATION (RCSI) to eliminate reader-writer deadlocks
ALTER DATABASE BankDB SET READ_COMMITTED_SNAPSHOT ON;

-- 2. Consistent lock ordering (always access Table A before Table B)
-- 3. Keep transactions short and fast
-- 4. Use UPDLOCK to avoid conversion deadlocks
BEGIN TRANSACTION;
    SELECT Balance FROM Accounts WITH (UPDLOCK, ROWLOCK) WHERE AccountId = 1001;
    UPDATE Accounts SET Balance = Balance - 100 WHERE AccountId = 1001;
COMMIT;

-- 5. Use NOLOCK hint for non-critical reporting queries
SELECT COUNT(*) FROM Transactions WITH (NOLOCK) WHERE BranchId = 5;
```

---

## 7. IBM Integration Bus (IIB) / IBM ACE

### Basic

**Q35: What is IBM Integration Bus (IIB)? What problem does it solve?**

**A:** IIB (now known as **IBM App Connect Enterprise / ACE**) is a middleware integration platform that allows disparate systems to exchange data regardless of:
- **Message format** (XML, JSON, SWIFT, ISO 20022, flat file, binary)
- **Protocol** (MQ, HTTP/REST, SOAP, FTP, JDBC, Email)
- **Technology stack** (.NET, Java, COBOL, mainframe)

In a bank, IIB sits between:
- Core Banking System ↔ Internet Banking Portal
- Payment Gateway ↔ SWIFT Network
- Mobile App ↔ Account Management System
- Internal systems ↔ RAAST / 1LINK / SBP

---

**Q36: What are the core components of an IIB/ACE solution?**

**A:**
- **Integration Node (Broker):** The runtime server that hosts and executes message flows.
- **Integration Server (Execution Group):** An isolated process within the broker. Message flows are deployed here.
- **Message Flow:** A visual pipeline of nodes defining how a message is received, transformed, routed, and sent.
- **Message Flow Nodes:** Individual processing units — Input nodes, Processing nodes (Compute, Filter, Route), Output nodes.
- **BAR File (Broker Archive):** The deployable artifact containing compiled message flows and resources.
- **MQ Queue Manager:** Often coupled with IIB for reliable persistent messaging.
- **IIB Toolkit:** Eclipse-based IDE for developing and testing message flows.

---

**Q37: What is ESQL? Where is it used in IIB?**

**A:** **Extended SQL (ESQL)** is IBM's proprietary language for manipulating messages in IIB. It's used inside:
- **Compute Node:** Transform, enrich, or create a new output message.
- **Filter Node:** Conditionally route messages based on content.
- **Database Node:** Query databases inline.

```esql
-- Compute Node ESQL: Transform XML payment to JSON
CREATE COMPUTE MODULE PaymentTransformer_Compute
    CREATE FUNCTION Main() RETURNS BOOLEAN
    BEGIN
        -- Access input XML fields
        DECLARE accountId CHAR InputRoot.XMLNSC.Payment.AccountId;
        DECLARE amount    DECIMAL InputRoot.XMLNSC.Payment.Amount;
        
        -- Build JSON output
        SET OutputRoot.JSON.Data.transactionId = UUIDASCHAR;
        SET OutputRoot.JSON.Data.accountId     = accountId;
        SET OutputRoot.JSON.Data.amount        = amount;
        SET OutputRoot.JSON.Data.timestamp     = 
            CAST(CURRENT_TIMESTAMP AS CHARACTER FORMAT 'ISOTimestamp');
        
        RETURN TRUE;
    END;
END MODULE;
```

---

### Intermediate

**Q38: What is the difference between `OutputRoot` and `OutputLocalEnvironment` in IIB?**

**A:**
- **`OutputRoot`:** Contains the actual outgoing message (headers + payload) that will be sent downstream.
- **`OutputLocalEnvironment`:** Used to set routing destinations dynamically (e.g., which MQ queue, or which HTTP URL). NOT part of the message itself.

```esql
-- Route to different queues based on transaction type
CREATE COMPUTE MODULE DynamicRouter_Compute
    CREATE FUNCTION Main() RETURNS BOOLEAN
    BEGIN
        DECLARE txnType CHAR InputRoot.XMLNSC.Transaction.Type;
        
        IF txnType = 'IBFT' THEN
            SET OutputLocalEnvironment.Destination.MQ.DestinationData[1].queueName 
                = 'IBFT.PROCESSING.QUEUE';
        ELSEIF txnType = 'BILL_PAY' THEN
            SET OutputLocalEnvironment.Destination.MQ.DestinationData[1].queueName 
                = 'BILLPAY.PROCESSING.QUEUE';
        END IF;
        
        SET OutputRoot = InputRoot; -- Pass message unchanged
        RETURN TRUE;
    END;
END MODULE;
```

---

**Q39: How does error handling work in IIB message flows?**

**A:** IIB provides several mechanisms:
1. **Failure Terminal:** Unhandled exceptions route to the `Failure` terminal of the input node.
2. **Catch Terminal:** Catches exceptions for local handling within the flow segment.
3. **Try-Catch in ESQL:** Handle errors inline inside Compute nodes.
4. **Dead Letter Queue (DLQ):** Unprocessable messages are put here with MQMD error info.

```esql
-- ESQL Try-Catch in Compute Node
CREATE FUNCTION Main() RETURNS BOOLEAN
BEGIN
    DECLARE errorCode INT;
    BEGIN
        PASSTHRU('SELECT Balance FROM Accounts WHERE AccountId = ?', 
                 InputRoot.XMLNSC.Request.AccountId) TO Database.Result;
        SET OutputRoot.JSON.Data.balance = Database.Result.Balance;
    END;
    
    SET errorCode = SQLCODE;
    IF errorCode <> 0 THEN
        SET OutputRoot.JSON.Data.error   = 'Database lookup failed';
        SET OutputRoot.JSON.Data.sqlCode = errorCode;
    END IF;
    
    RETURN TRUE;
END;
```

---

**Q40: How does your .NET application communicate with IIB?**

**A:** Multiple integration patterns:

**Pattern 1 — HTTP/REST (Synchronous):**
```csharp
public class IIBRestClient
{
    private readonly HttpClient _httpClient;

    public async Task<PaymentResponse> SubmitPaymentAsync(PaymentRequest request)
    {
        var response = await _httpClient.PostAsJsonAsync(
            "http://iib-server:7800/PaymentService/v1/submit", request);
        
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<PaymentResponse>();
    }
}
```

**Pattern 2 — IBM MQ (Asynchronous / Fire-and-forget):**
```csharp
public class MQPublisher
{
    public void PublishPayment(PaymentMessage payment)
    {
        var factory = XMSFactoryFactory.GetInstance(XMSC.CT_WMQ);
        var cf = factory.CreateConnectionFactory();
        cf.SetStringProperty(XMSC.WMQ_HOST_NAME, "mq-server");
        cf.SetIntProperty(XMSC.WMQ_PORT, 1414);
        cf.SetStringProperty(XMSC.WMQ_QUEUE_MANAGER, "QM_BANK");
        cf.SetStringProperty(XMSC.WMQ_CHANNEL, "SVRCONN.CHANNEL");

        using var connection = cf.CreateConnection();
        using var session = connection.CreateSession(false, AcknowledgeMode.AutoAcknowledge);
        var destination = session.CreateQueue("PAYMENT.IN.QUEUE");
        using var producer = session.CreateProducer(destination);

        connection.Start();
        var message = session.CreateTextMessage(JsonSerializer.Serialize(payment));
        message.SetStringProperty("CorrelationId", Guid.NewGuid().ToString());
        producer.Send(message);
    }
}
```

---

### Advanced

**Q41: How do you implement the request-reply pattern over MQ with .NET and IIB?**

**A:**
```csharp
public class MQRequestReplyClient
{
    public async Task<string> SendAndReceiveAsync(string payload, TimeSpan timeout)
    {
        // ... setup connection factory ...
        using var connection = cf.CreateConnection();
        using var session = connection.CreateSession(false, AcknowledgeMode.AutoAcknowledge);

        var requestQueue = session.CreateQueue("PAYMENT.REQUEST.QUEUE");
        var replyQueue   = session.CreateTemporaryQueue(); // Auto-deleted after use

        using var producer = session.CreateProducer(requestQueue);
        using var consumer = session.CreateConsumer(replyQueue);

        connection.Start();

        var request = session.CreateTextMessage(payload);
        request.JMSReplyTo        = replyQueue;
        request.JMSCorrelationID  = Guid.NewGuid().ToString();

        producer.Send(request);

        // Block and wait for IIB reply
        var reply = consumer.Receive((long)timeout.TotalMilliseconds);
        if (reply == null)
            throw new TimeoutException("IIB did not respond within the timeout period");

        return ((ITextMessage)reply).Text;
    }
}
```

---

**Q42: What is the difference between IIB v9/v10 and IBM ACE (v11+)?**

**A:**

| Feature | IIB v9/v10 | IBM ACE v11+ |
|---|---|---|
| MQ Requirement | Mandatory | Optional (standalone) |
| Container Support | Limited | Full Docker/Kubernetes |
| Node.js Support | Basic in v10 | Full Node.js runtime |
| REST API Mgmt | Manual | Built-in |
| Configuration | File-based | Kubernetes ConfigMaps |
| Monitoring | Basic | Prometheus/Grafana integration |
| Developer Edition | Not available | Free developer edition |

---

## 8. MQ & Messaging

### Basic

**Q43: What is IBM MQ? What is the difference between a Queue and a Topic?**

**A:**
- **IBM MQ:** Enterprise messaging middleware providing reliable, persistent, asynchronous message delivery. Guarantees **exactly-once** or **at-least-once** delivery with full transaction support.
- **Queue (Point-to-Point):** One producer, one consumer. Message is consumed once. Good for payment processing — ensures only one system processes each payment.
- **Topic (Publish-Subscribe):** One producer, many subscribers. All active subscribers receive the message. Good for broadcast events — e.g., rate change notification to multiple downstream systems.

---

**Q44: What is a Dead Letter Queue (DLQ)?**

**A:** A special queue where messages are deposited when they cannot be delivered or processed. Common reasons:
- Target queue does not exist.
- Queue is full.
- Message TTL expired.
- Application threw an unhandled exception.

In a bank, a dedicated DLQ monitoring service should alert operations immediately, with full context (original queue, error reason, message content) for investigation and reprocessing.

---

### Intermediate

**Q45: How do you ensure exactly-once processing of MQ messages in .NET?**

**A:** Combine transactional MQ sessions with idempotency checks:

```csharp
public async Task ProcessMessageExactlyOnceAsync(ITextMessage message)
{
    var messageId = message.JMSMessageID;
    
    // Idempotency check — already processed?
    if (await _repo.IsProcessedAsync(messageId))
    {
        _logger.LogWarning("Duplicate message {MessageId} — skipping", messageId);
        return;
    }

    await using var dbTx = await _context.Database.BeginTransactionAsync();
    try
    {
        await _repo.MarkProcessingAsync(messageId);
        await ProcessPaymentAsync(message.Text);
        await _repo.MarkProcessedAsync(messageId);
        
        await _context.SaveChangesAsync();
        await dbTx.CommitAsync();
        // ACK the MQ message only after DB commit succeeds
    }
    catch
    {
        await dbTx.RollbackAsync();
        // Not ACKing — message returns to queue for retry
        throw;
    }
}
```

---

## 9. Design Patterns & Architecture

### Intermediate

**Q46: Explain SOLID principles with banking examples.**

**A:**

**S — Single Responsibility:** `PaymentProcessor` should only process payments, not send email confirmations. Create a separate `NotificationService`.

**O — Open/Closed:** Add new payment types (IBFT, RTGS, SWIFT) by creating new classes implementing `IPaymentStrategy` — no changes to existing code.

**L — Liskov Substitution:** `SavingsAccount` and `CurrentAccount` should be substitutable for the `Account` base class without breaking caller behavior.

**I — Interface Segregation:** Don't force a read-only `ReportingService` to implement `IAccountService` with `Debit()` and `Credit()`. Split into `IAccountReader` and `IAccountWriter`.

**D — Dependency Inversion:** `PaymentController` depends on `IPaymentService` (abstraction), not `PaymentService` (concrete class). Swap implementations via DI without changing controller code.

---

**Q47: What is the Repository and Unit of Work pattern?**

**A:**
```csharp
// Repository abstracts data access per aggregate
public interface IAccountRepository
{
    Task<Account> GetByIdAsync(int id);
    Task<IEnumerable<Account>> GetByCustomerAsync(int customerId);
    Task AddAsync(Account account);
    void Update(Account account);
    void Delete(Account account);
}

// Unit of Work groups repositories and manages the transaction boundary
public interface IUnitOfWork : IDisposable
{
    IAccountRepository     Accounts     { get; }
    ITransactionRepository Transactions { get; }
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitAsync();
    Task RollbackAsync();
}

// Service uses UoW — no knowledge of DbContext
public class FundTransferService
{
    private readonly IUnitOfWork _uow;

    public async Task TransferAsync(int fromId, int toId, decimal amount)
    {
        await _uow.BeginTransactionAsync();
        var from = await _uow.Accounts.GetByIdAsync(fromId);
        var to   = await _uow.Accounts.GetByIdAsync(toId);

        from.Debit(amount);
        to.Credit(amount);

        await _uow.SaveChangesAsync();
        await _uow.CommitAsync();
    }
}
```

---

**Q48: What is CQRS and Event Sourcing?**

**A:**
- **CQRS (Command Query Responsibility Segregation):** Separate the write model (commands) from the read model (queries). Allows independent scaling and optimization of each side.
- **Event Sourcing:** Instead of storing current state, store every event that changed state. Current state is derived by replaying events. Provides a complete audit trail — ideal for banking.

```csharp
// Command (write side)
public record DebitAccountCommand(int AccountId, decimal Amount, string Reference);

// Event stored in the event log
public record AccountDebitedEvent(int AccountId, decimal Amount, decimal NewBalance, DateTime Timestamp);

// Event Store — append-only log
public class AccountEventStore
{
    public async Task AppendAsync(AccountDebitedEvent evt)
    {
        await _context.AccountEvents.AddAsync(new AccountEventEntity
        {
            AccountId = evt.AccountId,
            EventType = nameof(AccountDebitedEvent),
            EventData = JsonSerializer.Serialize(evt),
            Timestamp = evt.Timestamp
        });
    }

    // Rebuild current balance by replaying all events
    public async Task<decimal> ReplayBalanceAsync(int accountId)
    {
        var events = await _context.AccountEvents
            .Where(e => e.AccountId == accountId)
            .OrderBy(e => e.Timestamp)
            .ToListAsync();

        decimal balance = 0;
        foreach (var evt in events)
        {
            balance = evt.EventType switch
            {
                "AccountCreditedEvent" => balance + 
                    JsonSerializer.Deserialize<AccountCreditedEvent>(evt.EventData).Amount,
                "AccountDebitedEvent"  => balance - 
                    JsonSerializer.Deserialize<AccountDebitedEvent>(evt.EventData).Amount,
                _ => balance
            };
        }
        return balance;
    }
}
```

---

## 10. Security in Enterprise Applications

**Q49: How do you implement JWT authentication in ASP.NET Core?**

**A:**
```csharp
// Register JWT authentication
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer           = true,
            ValidateAudience         = true,
            ValidateLifetime         = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer              = config["Jwt:Issuer"],
            ValidAudience            = config["Jwt:Audience"],
            IssuerSigningKey         = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(config["Jwt:Key"])),
            ClockSkew = TimeSpan.Zero // No tolerance for expired tokens
        };
    });

// Token generation service
public string GenerateToken(User user)
{
    var claims = new[]
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Name,           user.Username),
        new Claim("branch",                  user.BranchCode),
        new Claim(ClaimTypes.Role,           user.Role)
    };

    var key   = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
    var token = new JwtSecurityToken(
        issuer:             _config["Jwt:Issuer"],
        audience:           _config["Jwt:Audience"],
        claims:             claims,
        expires:            DateTime.UtcNow.AddMinutes(30),
        signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256)
    );

    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

---

**Q50: How do you protect against SQL injection, XSS, and CSRF?**

**A:**
- **SQL Injection:** Always use parameterized queries or EF Core. Never concatenate user input.
- **XSS:** Use `HtmlEncoder.Default.Encode()` for output. Razor auto-encodes. Set Content Security Policy headers.
- **CSRF:** For cookie-based auth, use `[ValidateAntiForgeryToken]`. For JWT Bearer tokens, CSRF is not applicable by design.

```csharp
// Security headers middleware
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Content-Type-Options",    "nosniff");
    context.Response.Headers.Add("X-Frame-Options",           "DENY");
    context.Response.Headers.Add("X-XSS-Protection",          "1; mode=block");
    context.Response.Headers.Add("Strict-Transport-Security", "max-age=31536000; includeSubDomains");
    context.Response.Headers.Add("Content-Security-Policy",   "default-src 'self'");
    await next();
});

// Anti-forgery for MVC forms
services.AddAntiforgery(options =>
{
    options.Cookie.SameSite     = SameSiteMode.Strict;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});
```

---

## 11. Performance & Scalability

**Q51: How would you implement caching in a banking API?**

**A:**
```csharp
// Multi-level caching strategy
public class AccountService
{
    private readonly IMemoryCache     _localCache;  // L1: in-process
    private readonly IDistributedCache _redisCache; // L2: shared across instances
    private readonly IAccountRepository _repo;

    public async Task<AccountDto> GetAccountAsync(int accountId)
    {
        var cacheKey = $"account:{accountId}";

        // L1: In-process memory (microseconds)
        if (_localCache.TryGetValue(cacheKey, out AccountDto cached))
            return cached;

        // L2: Redis (milliseconds)
        var redisData = await _redisCache.GetStringAsync(cacheKey);
        if (redisData != null)
        {
            var account = JsonSerializer.Deserialize<AccountDto>(redisData);
            _localCache.Set(cacheKey, account, TimeSpan.FromSeconds(30));
            return account;
        }

        // L3: Database
        var dbAccount = await _repo.GetByIdAsync(accountId);
        var dto = _mapper.Map<AccountDto>(dbAccount);

        await _redisCache.SetStringAsync(cacheKey, JsonSerializer.Serialize(dto),
            new DistributedCacheEntryOptions 
            { AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5) });

        _localCache.Set(cacheKey, dto, TimeSpan.FromSeconds(30));
        return dto;
    }
}
```

---

**Q52: How do you implement resilience patterns using Polly?**

**A:**
```csharp
// Resilience pipeline for calling IIB or external payment services
services.AddHttpClient<IIBClient>()
    .AddPolicyHandler(GetRetryPolicy())
    .AddPolicyHandler(GetCircuitBreakerPolicy())
    .AddPolicyHandler(GetTimeoutPolicy());

static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy() =>
    HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(
            retryCount: 3,
            sleepDurationProvider: attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)), // 2s, 4s, 8s
            onRetry: (outcome, duration, attempt, _) =>
                Log.Warning("Retry {Attempt} after {Duration}ms", attempt, duration.TotalMilliseconds));

static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy() =>
    HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 5,
            durationOfBreak: TimeSpan.FromSeconds(30),
            onBreak:  (_, dur) => Log.Warning("Circuit OPEN for {Seconds}s", dur.TotalSeconds),
            onReset:  ()       => Log.Information("Circuit CLOSED — service recovered"),
            onHalfOpen: ()     => Log.Information("Circuit HALF-OPEN — testing"));

static IAsyncPolicy<HttpResponseMessage> GetTimeoutPolicy() =>
    Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(10));
```

---

## 12. Lead-Level Scenario Questions 🏆

**Q53 🏆: Your team is building a real-time payment processing system. The business wants 99.99% uptime. How do you architect this?**

**A:**

**99.99% uptime = max 52 minutes of downtime per year. This requires:**
- **RTO (Recovery Time Objective):** < 5 minutes.
- **RPO (Recovery Point Objective):** Zero — no transaction loss is acceptable.

**Architecture decisions:**

1. **Active-Active deployment** across two data centers (Primary DC + Disaster Recovery DC).
2. **Stateless API layer** behind a load balancer — horizontal scaling, no session affinity.
3. **IBM MQ Clustering** — persistent messages survive node failure and are replicated.
4. **IIB High Availability** — multiple integration servers across broker nodes.
5. **SQL Server Always On Availability Groups** — synchronous replication to DR replica.
6. **Redis Sentinel/Cluster** — distributed cache with automatic failover.
7. **Circuit Breakers** on all external calls (RAAST, 1LINK, SWIFT gateway).
8. **Health checks** with Kubernetes liveness/readiness probes + auto-restart.
9. **Blue-Green deployment** for zero-downtime releases — switch traffic after smoke tests.
10. **Idempotency** at every step — all retries are safe.
11. **Chaos engineering / game days** — regularly validate that failover actually works.
12. **Runbooks** — documented, tested procedures for every failure scenario.

---

**Q54 🏆: A critical bug is found in production — payments are being double-processed. What do you do?**

**A:**

**Immediate response (0–15 min):**
1. **Stop the bleeding** — disable the affected endpoint or pause the MQ consumer.
2. **Assess blast radius** — query DB for duplicate transactions in the last N hours.
3. **Communicate** — notify management, operations, and compliance immediately (regulatory obligation in banking).

**Investigation (15–60 min):**
4. Trace the flow using correlation IDs across API logs, IIB logs, and MQ.
5. Identify: Did idempotency checks fail? Was there a race condition? A failed deployment?
6. Review recent code changes and deployments.

**Remediation:**
7. Roll back if a deployment caused it, or deploy an emergency hotfix through the fast-track pipeline.
8. Write and test a SQL script to identify and reverse duplicate transactions safely.
9. Coordinate with operations for any manual reversals or customer notifications.
10. Re-enable processing with the fix confirmed.

**Post-incident:**
11. **Blameless post-mortem** — document root cause, timeline, contributing factors, action items.
12. Add regression test coverage for the exact scenario.
13. Improve monitoring — add alert for duplicate transaction rate > 0%.
14. Review whether this should trigger a SBP/regulatory notification.

---

**Q55 🏆: You inherit a monolithic .NET Framework banking application. The business wants to modernize it. How do you approach this?**

**A:**

**Phase 1 — Understand before touching (2–4 weeks):**
- Map all business capabilities and external integrations (IIB flows, MQ queues, SWIFT, 1LINK, RAAST, SBP).
- Identify the biggest pain points: deployment coupling, performance bottlenecks, testing gaps.
- Set up structured logging and APM (Application Insights / Dynatrace) to get a baseline.

**Phase 2 — Strangler Fig (not a big-bang rewrite):**
- Place an **API Gateway / BFF** (Backend for Frontend) in front of the monolith.
- Extract one clearly-bounded, lower-risk domain at a time into a .NET Core microservice.
- Route traffic for that domain to the new service; monolith remains live for everything else.
- Good first candidates: Notifications, Reporting, Statement Generation.

**Phase 3 — Enable microservices properly:**
- Replace in-process calls with **async messaging via MQ** to decouple services.
- Implement **Saga pattern** for distributed transactions (replaces DB-level transactions).
- Deploy services in **Kubernetes** for independent scaling and deployment cadence.

**Non-negotiables in banking:**
- Every migration step must be auditable, reversible, and tested in UAT with real data volumes.
- Zero customer data disruption — run old and new systems in parallel, validate output matches.
- Compliance and legal sign-off before any customer-facing change goes live.

---

**Q56 🏆: How do you ensure a junior team delivers quality code in an enterprise banking environment?**

**A:**

**Process guardrails:**
- **Definition of Done (DoD):** Unit tests passing, integration tests passing, code review by 2 seniors, no critical SonarQube findings, OWASP dependency scan clean, documentation updated.
- **PR review standards:** All payment-critical code requires senior lead approval. All DB migrations reviewed by the DBA or lead.
- **Architecture Decision Records (ADRs):** Document significant design decisions so context is preserved as the team evolves.

**Technical practices:**
- **Pair programming** on complex modules — IIB integration, cryptographic functions, schema migrations.
- **CI pipeline quality gates:** `dotnet format`, SonarQube, OWASP check — failures block merge to main.
- **Feature flags:** Enable gradual rollout and instant rollback without redeployment.
- **Test pyramid:** Unit tests for business logic, integration tests for DB/MQ, contract tests for IIB APIs.

**Culture:**
- **Blameless post-mortems** — engineers are not afraid to surface issues early.
- **Tech debt sprints** — allocate 20% capacity each sprint to address SonarQube and architectural debt.
- **Weekly knowledge sessions (30 min):** Rotate presenters. Topics: new features, production issues, bank regulations.
- **Mentorship accountability:** Each senior is responsible for the growth of an assigned junior.

---

**Q57 🏆: How would you design a rate-limiting system for Bank Al Habib's public APIs?**

**A:**
```csharp
// Sliding window rate limiter using Redis
public class RateLimiterMiddleware
{
    private readonly RequestDelegate    _next;
    private readonly IDistributedCache  _cache;
    private readonly RateLimitOptions   _options;

    public async Task InvokeAsync(HttpContext context)
    {
        var clientKey  = GetClientKey(context);  // API key or IP
        var windowKey  = $"ratelimit:{clientKey}:{DateTime.UtcNow:yyyyMMddHHmm}";
        var limit      = GetLimit(context);

        var countStr = await _cache.GetStringAsync(windowKey);
        var count    = countStr != null ? int.Parse(countStr) : 0;

        if (count >= limit)
        {
            context.Response.StatusCode = 429;
            context.Response.Headers.Add("Retry-After",       "60");
            context.Response.Headers.Add("X-RateLimit-Limit", limit.ToString());
            await context.Response.WriteAsJsonAsync(new { error = "Rate limit exceeded. Please retry after 60 seconds." });
            return;
        }

        // Increment counter with 2-minute expiry (covers window overlap)
        await _cache.SetStringAsync(windowKey, (count + 1).ToString(),
            new DistributedCacheEntryOptions 
            { AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(2) });

        context.Response.Headers.Add("X-RateLimit-Remaining", (limit - count - 1).ToString());
        await _next(context);
    }

    // Tiered limits based on client type
    private int GetLimit(HttpContext context) =>
        context.User.IsInRole("PremiumPartner") ? 10_000 :
        context.User.IsInRole("Internal")       ?  5_000 : 100;
}
```

---

**Q58 🏆: Describe how you would design end-to-end observability for a payment flow spanning .NET API → IIB → MQ → Core Banking.**

**A:**

**The three pillars: Logging, Metrics, Distributed Tracing.**

**Step 1 — Correlation ID propagation:**
- Generate a `CorrelationId` (UUID) at the API gateway for every incoming payment.
- Propagate via: HTTP header `X-Correlation-Id`, MQ message property `CorrelationId`, IIB `LocalEnvironment`.
- Every single log line across every system must include this ID.

**Step 2 — Structured logging:**
```csharp
Log.ForContext("CorrelationId", correlationId)
   .ForContext("AccountId",     accountId)
   .ForContext("Amount",        amount)
   .ForContext("PaymentType",   paymentType)
   .Information("Payment initiated — entering IIB flow");
```

**Step 3 — Metrics (Prometheus + Grafana dashboards):**
- Payment throughput (payments/second by type).
- End-to-end latency percentiles: p50, p95, p99.
- IIB queue depth per queue (alert threshold: > 1,000 messages).
- Error rate per payment type and per integration point.
- MQ DLQ message count (any message = immediate alert).

**Step 4 — Distributed tracing (OpenTelemetry → Jaeger):**
- Each service creates a child span under the root trace.
- IIB logs trace context via user-defined ESQL properties.
- Visualize the full call chain: API → IIB → MQ → Core Banking → response.

**Step 5 — Alerting rules (PagerDuty):**
- Payment error rate > 0.1% → P1 alert.
- API p99 latency > 5 seconds → P2 alert.
- Any DLQ message → P1 alert.
- IIB queue depth > 1,000 → P2 alert.
- Zero payment throughput for 2 minutes during business hours → P1 alert.

**Tool stack:** OpenTelemetry SDK → Jaeger (distributed tracing) + Prometheus (metrics) + ELK / Splunk (log aggregation) + Grafana (dashboards + alerting).

---

*End of Interview Preparation Guide*

---

> **Good luck with your Bank Al Habib interview!**
> Key focus areas: IIB message flows + ESQL basics, .NET async patterns, MQ integration in C#, SQL transaction handling, and be ready to walk through a production incident or system design you have real experience with.
