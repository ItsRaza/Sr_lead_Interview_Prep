# Senior/Lead/Architecture Role Interview Preparation

Comprehensive Q&A Guide for .NET, Azure, and Architecture Interviews

# Table of Contents

1. Object-Oriented Programming (OOP)

2. Database & SQL (RDBMS)

3. C# Programming

4. .NET Core & ASP.NET

5. Microsoft Azure

6. CI/CD & DevOps

7. Docker & Kubernetes

8. System Design & Architecture

9. Behavioral & Leadership Questions

10. Quick Reference Cheat Sheet

# 1. Object-Oriented Programming (OOP)

## Q1: Explain the four pillars of OOP and provide real-world examples from your experience.

Answer:

The four pillars of OOP are Encapsulation, Abstraction, Inheritance, and Polymorphism.

1. Encapsulation: Bundling data and methods that operate on that data within a single unit (class) and restricting direct access to some components.

Example: In my ERP system at Info Access Solutions, I encapsulated payment gateway logic within a PaymentProcessor class, exposing only ProcessPayment() method while keeping gateway-specific credentials and validation logic private.

public class PaymentProcessor
{
    private string _apiKey;
    private string _secretKey;
    
    public PaymentResponse ProcessPayment(decimal amount, string currency)
    {
        ValidatePayment(amount);
        return CallGatewayAPI(amount, currency);
    }
    
    private void ValidatePayment(decimal amount) { /* validation logic */ }
    private PaymentResponse CallGatewayAPI(decimal amount, string currency) { /* API call */ }
}

2. Abstraction: Hiding complex implementation details and showing only essential features.

Example: At Enterprise64, I created an abstract IWarehouseService interface that defined core operations (ReceiveInventory, ShipOrder, TrackLocation), allowing different warehouse implementations (RFID-based, barcode-based) without exposing their internal complexity.

public interface IWarehouseService
{
    Task<InventoryResult> ReceiveInventory(InventoryRequest request);
    Task<ShipmentResult> ShipOrder(OrderRequest order);
}

public class RFIDWarehouseService : IWarehouseService
{
    // RFID-specific implementation
}

public class BarcodeWarehouseService : IWarehouseService
{
    // Barcode-specific implementation
}

3. Inheritance: Mechanism where a new class derives properties and behavior from an existing class.

Example: In the mutual funds super app at DigiTrends, I created a base FundAccount class with common properties and methods, then derived SavingsFund, MoneyMarketFund, and EquityFund classes that added specific features.

public abstract class FundAccount
{
    public string AccountNumber { get; set; }
    public decimal Balance { get; protected set; }
    
    public abstract decimal CalculateReturns();
    public virtual void Deposit(decimal amount) 
    { 
        Balance += amount; 
    }
}

public class EquityFund : FundAccount
{
    public decimal RiskFactor { get; set; }
    
    public override decimal CalculateReturns()
    {
        return Balance * RiskFactor * MarketGrowthRate;
    }
}

4. Polymorphism: Ability of objects to take multiple forms. Method overloading (compile-time) and method overriding (runtime).

Example: At DigiTrends, I implemented a notification system where different notification types (Email, SMS, Push) inherited from INotificationService and overrode the Send() method with their specific implementation.

public interface INotificationService
{
    Task<bool> Send(string recipient, string message);
}

public class EmailNotificationService : INotificationService
{
    public async Task<bool> Send(string recipient, string message)
    {
        // Email-specific sending logic
        return await SendEmail(recipient, message);
    }
}

public class SMSNotificationService : INotificationService
{
    public async Task<bool> Send(string recipient, string message)
    {
        // SMS-specific sending logic
        return await SendSMS(recipient, message);
    }
}

Key Resources:

Microsoft Docs: https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/

OOP Design Patterns: https://refactoring.guru/design-patterns

## Q2: What is the difference between Abstract Classes and Interfaces? When would you use each?

Answer:

Abstract Classes:

Can have implementation for some methods and fields

Supports access modifiers (public, protected, private)

Can have constructors

Single inheritance only (a class can inherit from only one abstract class)

Use when: You have a "is-a" relationship and want to share code among related classes

Interfaces:

No implementation (prior to C# 8.0), only method signatures

All members are public by default

Cannot have constructors

Multiple inheritance supported (a class can implement multiple interfaces)

Use when: You have a "can-do" relationship and want to define a contract

Real-world Example from My Experience:

At DigiTrends, I used an abstract class BaseReportGenerator for common report functionality (pagination, headers, footers), while interfaces like IExportable and ICacheable were used for cross-cutting concerns that any class could implement regardless of hierarchy.

// Abstract class for common report behavior
public abstract class BaseReportGenerator
{
    protected string Title { get; set; }
    protected DateTime GeneratedDate { get; set; }
    
    protected void AddHeader() { /* common header logic */ }
    public abstract byte[] Generate();
}

// Interfaces for capabilities
public interface IExportable
{
    byte[] ExportToPDF();
    byte[] ExportToExcel();
}

public interface ICacheable
{
    string CacheKey { get; }
    TimeSpan CacheDuration { get; }
}

// Concrete implementation
public class SalesReport : BaseReportGenerator, IExportable, ICacheable
{
    public override byte[] Generate() { /* implementation */ }
    public byte[] ExportToPDF() { /* implementation */ }
    public byte[] ExportToExcel() { /* implementation */ }
    public string CacheKey => "sales_report";
    public TimeSpan CacheDuration => TimeSpan.FromHours(1);
}

Key Resources:

Microsoft Docs - Abstract Classes: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members

Microsoft Docs - Interfaces: https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces

## Q3: Explain SOLID principles with examples from your projects.

Answer:

SOLID is an acronym for five design principles that make software designs more understandable, flexible, and maintainable.

1. Single Responsibility Principle (SRP): A class should have only one reason to change.

Example: At Enterprise64, I refactored a monolithic OrderProcessor class into separate classes: OrderValidator, InventoryChecker, PaymentProcessor, and NotificationSender. Each class had one responsibility.

// Before: Violates SRP
public class OrderProcessor
{
    public void ProcessOrder(Order order)
    {
        ValidateOrder(order);
        CheckInventory(order);
        ProcessPayment(order);
        SendNotification(order);
        UpdateDatabase(order);
    }
}

// After: Follows SRP
public class OrderValidator
{
    public ValidationResult Validate(Order order) { /* validation */ }
}

public class InventoryService
{
    public bool CheckAvailability(Order order) { /* inventory check */ }
}

public class PaymentService
{
    public PaymentResult Process(Order order) { /* payment */ }
}

2. Open/Closed Principle (OCP): Classes should be open for extension but closed for modification.

Example: At DigiTrends, I implemented a discount calculation system using strategy pattern. Adding new discount types didn't require modifying existing code.

public interface IDiscountStrategy
{
    decimal CalculateDiscount(decimal amount);
}

public class PercentageDiscount : IDiscountStrategy
{
    private readonly decimal _percentage;
    public PercentageDiscount(decimal percentage) => _percentage = percentage;
    
    public decimal CalculateDiscount(decimal amount) => amount * _percentage / 100;
}

public class FixedAmountDiscount : IDiscountStrategy
{
    private readonly decimal _amount;
    public FixedAmountDiscount(decimal amount) => _amount = amount;
    
    public decimal CalculateDiscount(decimal amount) => _amount;
}

// Adding new discount type doesn't modify existing code
public class TieredDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount)
    {
        if (amount > 10000) return amount * 0.20m;
        if (amount > 5000) return amount * 0.15m;
        return amount * 0.10m;
    }
}

3. Liskov Substitution Principle (LSP): Derived classes must be substitutable for their base classes.

Example: At Info Access Solutions, I ensured that all payment gateway implementations (HBL, Alfalah, 1-Link) could be used interchangeably without breaking the payment flow.

public abstract class PaymentGateway
{
    public abstract Task<PaymentResult> ProcessPayment(PaymentRequest request);
    
    public virtual bool ValidateRequest(PaymentRequest request)
    {
        return request.Amount > 0 && !string.IsNullOrEmpty(request.Currency);
    }
}

public class HBLGateway : PaymentGateway
{
    public override async Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        // HBL-specific implementation
        // Must maintain the same contract as base class
    }
}

public class AlfalahGateway : PaymentGateway
{
    public override async Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        // Alfalah-specific implementation
        // Must maintain the same contract as base class
    }
}

// Client code works with any payment gateway
public class PaymentService
{
    public async Task ProcessOrder(Order order, PaymentGateway gateway)
    {
        var result = await gateway.ProcessPayment(order.PaymentRequest);
        // Works with any PaymentGateway implementation
    }
}

4. Interface Segregation Principle (ISP): Clients should not be forced to depend on interfaces they don't use.

Example: At DigiTrends, instead of one large IRepository interface, I created specific interfaces like IReadRepository, IWriteRepository, and ISearchableRepository.

// Bad: Fat interface
public interface IRepository<T>
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
    IEnumerable<T> Search(Expression<Func<T, bool>> predicate);
    Task BulkInsert(IEnumerable<T> entities);
    Task<bool> ExistsAsync(int id);
}

// Good: Segregated interfaces
public interface IReadRepository<T>
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    Task<bool> ExistsAsync(int id);
}

public interface IWriteRepository<T>
{
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
}

public interface ISearchableRepository<T>
{
    IEnumerable<T> Search(Expression<Func<T, bool>> predicate);
}

public interface IBulkRepository<T>
{
    Task BulkInsert(IEnumerable<T> entities);
}

// Classes implement only what they need
public class ReadOnlyProductRepository : IReadRepository<Product>
{
    // Only implements read operations
}

public class FullProductRepository : IReadRepository<Product>, 
                                    IWriteRepository<Product>, 
                                    ISearchableRepository<Product>
{
    // Implements all operations
}

5. Dependency Inversion Principle (DIP): High-level modules should not depend on low-level modules. Both should depend on abstractions.

Example: At Enterprise64, I used dependency injection throughout the application, ensuring all services depended on interfaces rather than concrete implementations.

// Bad: Direct dependency on concrete class
public class OrderService
{
    private SqlServerRepository _repository = new SqlServerRepository();
    private SmtpEmailService _emailService = new SmtpEmailService();
    
    public void CreateOrder(Order order)
    {
        _repository.Save(order);
        _emailService.Send("New order created");
    }
}

// Good: Dependency on abstractions
public interface IRepository
{
    void Save(Order order);
}

public interface IEmailService
{
    void Send(string message);
}

public class OrderService
{
    private readonly IRepository _repository;
    private readonly IEmailService _emailService;
    
    // Dependencies injected through constructor
    public OrderService(IRepository repository, IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
    
    public void CreateOrder(Order order)
    {
        _repository.Save(order);
        _emailService.Send("New order created");
    }
}

// Startup.cs configuration
services.AddScoped<IRepository, SqlServerRepository>();
services.AddScoped<IEmailService, SmtpEmailService>();
services.AddScoped<OrderService>();

Key Resources:

SOLID Principles: https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design

Clean Architecture: https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures

# 2. Database & SQL (RDBMS)

## Q4: Explain database normalization and denormalization. When would you use each?

Answer:

Normalization: Process of organizing data to reduce redundancy and improve data integrity.

Normal Forms:

1NF (First Normal Form): Eliminate duplicate columns; each cell contains atomic values

2NF: Meet 1NF + all non-key attributes fully dependent on primary key

3NF: Meet 2NF + no transitive dependencies

BCNF (Boyce-Codd): Stronger version of 3NF

Example: In the ERP system at Info Access Solutions, I normalized student fee data:

-- Unnormalized (violates 1NF - repeating groups)
CREATE TABLE StudentFees (
    StudentID INT,
    StudentName VARCHAR(100),
    Fee1 DECIMAL,
    Fee2 DECIMAL,
    Fee3 DECIMAL
);

-- Normalized (3NF)
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(100),
    ClassID INT FOREIGN KEY REFERENCES Classes(ClassID)
);

CREATE TABLE FeeTypes (
    FeeTypeID INT PRIMARY KEY,
    FeeTypeName VARCHAR(50),
    Amount DECIMAL
);

CREATE TABLE StudentFees (
    StudentFeeID INT PRIMARY KEY,
    StudentID INT FOREIGN KEY REFERENCES Students(StudentID),
    FeeTypeID INT FOREIGN KEY REFERENCES FeeTypes(FeeTypeID),
    AmountPaid DECIMAL,
    PaymentDate DATE
);

Denormalization: Intentionally introducing redundancy to improve read performance.

Example: At Enterprise64, I denormalized warehouse reporting tables to avoid expensive joins on millions of records:

-- Normalized (requires joins for reports)
SELECT 
    o.OrderID,
    c.CustomerName,
    p.ProductName,
    w.WarehouseName,
    od.Quantity
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN OrderDetails od ON o.OrderID = od.OrderID
JOIN Products p ON od.ProductID = p.ProductID
JOIN Warehouses w ON od.WarehouseID = w.WarehouseID;

-- Denormalized reporting table (pre-joined for performance)
CREATE TABLE OrderReporting (
    OrderID INT,
    OrderDate DATE,
    CustomerID INT,
    CustomerName VARCHAR(100),
    ProductID INT,
    ProductName VARCHAR(200),
    WarehouseID INT,
    WarehouseName VARCHAR(100),
    Quantity INT,
    TotalAmount DECIMAL,
    -- Indexed for common query patterns
    INDEX idx_date (OrderDate),
    INDEX idx_customer (CustomerID),
    INDEX idx_product (ProductID)
);

When to Use Each:

Normalization: OLTP systems, write-heavy applications, data integrity critical

Denormalization: OLAP/Reporting systems, read-heavy applications, performance critical

Key Resources:

Database Normalization: https://www.guru99.com/database-normalization.html

SQL Server Best Practices: https://docs.microsoft.com/en-us/sql/relational-databases/performance/performance-monitoring-and-tuning-tools

## Q5: Explain indexing strategies and query optimization techniques you've used.

Answer:

Index Types:

1. Clustered Index: Determines physical order of data. One per table.

Use for: Primary keys, frequently sorted columns

Example: StudentID in Students table

2. Non-Clustered Index: Separate structure pointing to data. Multiple per table.

Use for: Frequently queried columns, WHERE clause columns, JOIN columns

Example: Email in Users table for login queries

3. Covering Index: Includes all columns needed by a query.

Eliminates need to access table data

4. Filtered Index: Index on subset of rows.

Use for: Queries on sparse columns or specific conditions

Real-world Optimization Example from Enterprise64:

-- Problem: Slow inventory search query (2-3 seconds on 5M records)
SELECT 
    InventoryID,
    ProductID,
    WarehouseID,
    Quantity,
    LastUpdated
FROM Inventory
WHERE WarehouseID = 123 
  AND ProductID IN (SELECT ProductID FROM Products WHERE CategoryID = 5)
  AND Quantity > 0
  AND LastUpdated >= '2024-01-01';

-- Solution 1: Create covering index
CREATE NONCLUSTERED INDEX IX_Inventory_Warehouse_Product_Quantity
ON Inventory (WarehouseID, ProductID, Quantity, LastUpdated)
INCLUDE (InventoryID);

-- Solution 2: Rewrite query to avoid subquery
SELECT 
    i.InventoryID,
    i.ProductID,
    i.WarehouseID,
    i.Quantity,
    i.LastUpdated
FROM Inventory i
INNER JOIN Products p ON i.ProductID = p.ProductID
WHERE i.WarehouseID = 123 
  AND p.CategoryID = 5
  AND i.Quantity > 0
  AND i.LastUpdated >= '2024-01-01';

-- Solution 3: Use filtered index for active inventory
CREATE NONCLUSTERED INDEX IX_Inventory_Active
ON Inventory (WarehouseID, ProductID)
WHERE Quantity > 0;

-- Result: Query time reduced from 2-3 seconds to 50-100ms

Query Optimization Techniques I've Applied:

1. Avoid SELECT *:

-- Bad
SELECT * FROM Orders WHERE CustomerID = 123;

-- Good
SELECT OrderID, OrderDate, TotalAmount FROM Orders WHERE CustomerID = 123;

2. Use EXISTS instead of COUNT for existence checks:

-- Bad (counts all rows)
IF (SELECT COUNT(*) FROM Orders WHERE CustomerID = 123) > 0

-- Good (stops at first match)
IF EXISTS (SELECT 1 FROM Orders WHERE CustomerID = 123)

3. Avoid functions on indexed columns in WHERE clause:

-- Bad (index not used)
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2024;

-- Good (index used)
SELECT * FROM Orders 
WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';

4. Use appropriate JOIN types:

-- Use INNER JOIN when you need matching rows from both tables
-- Use LEFT JOIN when you need all rows from left table
-- Avoid CROSS JOIN unless intentional Cartesian product needed

5. Pagination optimization:

-- Bad (scans all rows)
SELECT * FROM Orders 
ORDER BY OrderDate DESC
OFFSET 10000 ROWS FETCH NEXT 50 ROWS ONLY;

-- Good (using indexed column for cursor)
SELECT * FROM Orders 
WHERE OrderID < @LastOrderID
ORDER BY OrderID DESC
FETCH NEXT 50 ROWS ONLY;

Key Resources:

SQL Server Indexing: https://docs.microsoft.com/en-us/sql/relational-databases/indexes/indexes

Query Performance Tuning: https://use-the-index-luke.com/

Execution Plans: https://www.brentozar.com/archive/2013/10/how-to-read-execution-plans/

## Q6: Explain ACID properties and transaction isolation levels.

Answer:

ACID Properties:

1. Atomicity: All-or-nothing. Either entire transaction succeeds or fails completely.

Example: In payment processing at Info Access Solutions, either both debit from student account AND credit to school account happen, or neither happens.

BEGIN TRANSACTION;
TRY
    -- Debit student account
    UPDATE StudentAccounts SET Balance = Balance - 5000 
    WHERE StudentID = 123;
    
    -- Credit school account
    UPDATE SchoolAccounts SET Balance = Balance + 5000 
    WHERE SchoolID = 1;
    
    -- Record payment
    INSERT INTO Payments (StudentID, Amount, PaymentDate) 
    VALUES (123, 5000, GETDATE());
    
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    THROW;
END CATCH

2. Consistency: Database moves from one valid state to another. All constraints maintained.

Example: Foreign key constraints, check constraints, triggers ensure data validity.

3. Isolation: Concurrent transactions don't interfere with each other.

Example: Two users booking last available seat shouldn't both succeed.

4. Durability: Once committed, changes are permanent even after system failure.

Example: Transaction logs ensure recovery after crash.

Transaction Isolation Levels (from weakest to strongest):

1. READ UNCOMMITTED:

Allows dirty reads (reading uncommitted changes from other transactions)

Lowest isolation, highest performance

Use case: Dashboard statistics where exact accuracy not critical

2. READ COMMITTED (SQL Server default):

Prevents dirty reads, but allows non-repeatable reads

Can read only committed data

Use case: Most OLTP applications

3. REPEATABLE READ:

Prevents dirty reads and non-repeatable reads

Holds read locks until transaction completes

Can still have phantom reads

Use case: Financial calculations within a transaction

4. SERIALIZABLE:

Prevents all anomalies: dirty reads, non-repeatable reads, phantom reads

Highest isolation, lowest performance

Use case: Critical financial transactions, inventory reservations

5. SNAPSHOT (SQL Server specific):

Readers don't block writers, writers don't block readers

Uses row versioning in tempdb

Use case: Reporting queries that shouldn't block OLTP operations

Real-world Example from Enterprise64:

-- Inventory reservation system - preventing overselling
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;

-- Check availability
DECLARE @Available INT;
SELECT @Available = Quantity 
FROM Inventory 
WHERE ProductID = 123 AND WarehouseID = 1;

IF @Available >= 5
BEGIN
    -- Reserve inventory
    UPDATE Inventory 
    SET Quantity = Quantity - 5,
        ReservedQuantity = ReservedQuantity + 5
    WHERE ProductID = 123 AND WarehouseID = 1;
    
    -- Create reservation record
    INSERT INTO Reservations (ProductID, WarehouseID, Quantity, OrderID)
    VALUES (123, 1, 5, 456);
    
    COMMIT TRANSACTION;
END
ELSE
BEGIN
    ROLLBACK TRANSACTION;
    THROW 50001, 'Insufficient inventory', 1;
END

-- For reporting (don't block operations)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT SUM(Quantity) as TotalInventory FROM Inventory;

Key Resources:

Transaction Isolation Levels: https://docs.microsoft.com/en-us/sql/t-sql/statements/set-transaction-isolation-level-transact-sql

ACID Properties: https://www.databricks.com/glossary/acid-transactions

## Q7: How do you handle database migrations in production environments?

Answer:

My approach to database migrations, refined across multiple production deployments:

1. Version Control & Migration Tools:

Use Entity Framework Migrations or FluentMigrator for .NET projects

Store migration scripts in source control alongside application code

Use naming convention: YYYYMMDDHHMMSS_Description.cs

// Entity Framework Migration Example
public partial class AddOrderStatusIndex : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateIndex(
            name: "IX_Orders_Status_CreatedDate",
            table: "Orders",
            columns: new[] { "Status", "CreatedDate" });
    }
    
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropIndex(
            name: "IX_Orders_Status_CreatedDate",
            table: "Orders");
    }
}

2. Backward Compatibility Strategy:

Make changes in multiple deployments when breaking changes needed

Add new columns as nullable initially, populate data, then make required

Keep old columns during transition period

Example from DigiTrends B2B platform migration:

-- Phase 1: Add new nullable column
ALTER TABLE Customers ADD Email VARCHAR(255) NULL;

-- Phase 2: Deploy application code that writes to both columns
-- (Application handles both old and new schema)

-- Phase 3: Data migration
UPDATE Customers SET Email = OldEmailField WHERE Email IS NULL;

-- Phase 4: Make column required
ALTER TABLE Customers ALTER COLUMN Email VARCHAR(255) NOT NULL;
CREATE UNIQUE INDEX IX_Customers_Email ON Customers(Email);

-- Phase 5: Remove old column (in next release)
ALTER TABLE Customers DROP COLUMN OldEmailField;

3. Zero-Downtime Deployments:

Run migrations before deploying new application version

Test migrations on production-like data

Use online index operations for large tables

-- Create index without locking table
CREATE NONCLUSTERED INDEX IX_Products_CategoryID 
ON Products(CategoryID)
WITH (ONLINE = ON, MAXDOP = 4);

4. Rollback Strategy:

Always have a DOWN migration

Test rollback procedure

Keep database backups before migration

5. Production Migration Checklist:

Test migration on staging environment with production-size data

Estimate migration time for large tables

Schedule during low-traffic window if possible

Have rollback plan ready

Monitor application logs during deployment

Have DBA on standby for large migrations

Real-world Example from Enterprise64:

We had to add a composite index on a 50-million row Orders table. The migration:

Tested on staging with 60M rows - took 45 minutes

Used ONLINE = ON option to avoid blocking

Ran during 2 AM maintenance window

Monitored query performance before/after

Result: Query performance improved by 75% with zero downtime

Key Resources:

EF Core Migrations: https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/

FluentMigrator: https://fluentmigrator.github.io/

Database Refactoring: https://www.martinfowler.com/books/refactoringDatabases.html

# 3. C# Programming

## Q8: Explain async/await and when to use it. What are common pitfalls?

Answer:

Async/await enables asynchronous programming, allowing threads to work on other tasks while waiting for I/O operations.

Key Concepts:

async: Keyword to mark method as asynchronous

await: Keyword to wait for async operation without blocking thread

Task/Task<T>: Represents asynchronous operation

ConfigureAwait(false): Avoid capturing synchronization context (use in library code)

When to Use Async/Await:

I/O-bound operations: Database calls, file operations, HTTP requests

Long-running operations that don't need CPU

Operations that can run concurrently

When NOT to Use:

CPU-bound operations (use Task.Run or parallel processing instead)

Very fast operations (overhead not worth it)

When you need to block (e.g., console app Main before C# 7.1)

Example from DigiTrends - Processing Multiple SAP Integrations:

// Bad: Sequential execution (takes 15+ seconds for 5 APIs)
public async Task<OrderResult> ProcessOrderAsync(Order order)
{
    var inventory = await _inventoryService.CheckInventoryAsync(order.ProductId);
    var pricing = await _pricingService.GetPriceAsync(order.ProductId);
    var customer = await _customerService.GetCustomerAsync(order.CustomerId);
    var discount = await _discountService.CalculateDiscountAsync(order);
    var validation = await _validationService.ValidateAsync(order);
    
    return CreateOrderResult(inventory, pricing, customer, discount, validation);
}

// Good: Parallel execution (takes 3-4 seconds)
public async Task<OrderResult> ProcessOrderAsync(Order order)
{
    var inventoryTask = _inventoryService.CheckInventoryAsync(order.ProductId);
    var pricingTask = _pricingService.GetPriceAsync(order.ProductId);
    var customerTask = _customerService.GetCustomerAsync(order.CustomerId);
    var discountTask = _discountService.CalculateDiscountAsync(order);
    var validationTask = _validationService.ValidateAsync(order);
    
    await Task.WhenAll(inventoryTask, pricingTask, customerTask, 
                      discountTask, validationTask);
    
    return CreateOrderResult(
        await inventoryTask, 
        await pricingTask, 
        await customerTask,
        await discountTask, 
        await validationTask
    );
}

Common Pitfalls and Solutions:

1. Async Void - NEVER use except for event handlers:

// BAD - exceptions can't be caught
public async void ProcessDataAsync()
{
    await _service.ProcessAsync();
}

// GOOD - returns Task for proper exception handling
public async Task ProcessDataAsync()
{
    await _service.ProcessAsync();
}

2. Deadlock with .Result or .Wait():

// BAD - causes deadlock in UI/ASP.NET contexts
public void ProcessData()
{
    var result = GetDataAsync().Result; // DEADLOCK!
}

// GOOD - use async all the way
public async Task ProcessDataAsync()
{
    var result = await GetDataAsync();
}

// If you MUST block (console app), use GetAwaiter().GetResult()
public void ProcessData()
{
    var result = GetDataAsync().GetAwaiter().GetResult();
}

3. Not Using ConfigureAwait(false) in Library Code:

// Library code - don't need UI context
public async Task<Data> GetDataAsync()
{
    var response = await _httpClient.GetAsync(url)
        .ConfigureAwait(false); // Avoid capturing context
    var content = await response.Content.ReadAsStringAsync()
        .ConfigureAwait(false);
    return JsonSerializer.Deserialize<Data>(content);
}

// Application code - usually omit ConfigureAwait
public async Task UpdateUIAsync()
{
    var data = await GetDataAsync(); // Captures UI context by default
    textBox.Text = data.Value; // Safe - on UI thread
}

4. Exception Handling in Parallel Tasks:

// BAD - only catches first exception
try
{
    await Task.WhenAll(task1, task2, task3);
}
catch (Exception ex)
{
    // Only catches first exception
}

// GOOD - handle all exceptions
try
{
    await Task.WhenAll(task1, task2, task3);
}
catch
{
    var exceptions = new List<Exception>();
    if (task1.IsFaulted) exceptions.Add(task1.Exception);
    if (task2.IsFaulted) exceptions.Add(task2.Exception);
    if (task3.IsFaulted) exceptions.Add(task3.Exception);
    
    if (exceptions.Any())
        throw new AggregateException(exceptions);
}

Real-world Performance Impact at Enterprise64:

Converted synchronous warehouse operations to async/await:

Before: Processing 100 orders took 45 seconds (sequential)

After: Processing 100 orders took 8 seconds (parallel async)

Server CPU utilization dropped from 80% to 35%

Throughput increased from 150 orders/min to 750 orders/min

Key Resources:

Async/Await Best Practices: https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming

Task-based Async Pattern: https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap

## Q9: Explain memory management in C#: Stack vs Heap, Garbage Collection, IDisposable.

Answer:

Stack vs Heap:

Stack:

Stores value types, method parameters, local variables, return addresses

LIFO (Last In First Out) structure

Fast allocation and deallocation

Memory automatically reclaimed when method exits

Limited size (~1MB per thread)

Heap:

Stores reference types (objects)

Slower allocation, managed by Garbage Collector

Larger size, shared across application

Objects remain until GC collects them

// Value types on stack
int number = 42;           // Stack
DateTime date = DateTime.Now;  // Stack (struct)

// Reference types on heap
string text = "Hello";     // Reference on stack, object on heap
var order = new Order();   // Reference on stack, object on heap

// Value type containing reference type
struct OrderInfo
{
    public int OrderId;      // On stack
    public string Name;      // Reference on stack, string on heap
}

Garbage Collection:

GC Generations:

Gen 0: Short-lived objects (most objects die here)

Gen 1: Medium-lived objects (buffer between Gen 0 and Gen 2)

Gen 2: Long-lived objects (large object heap for objects >85KB)

GC Triggers:

Gen 0 threshold exceeded (~1MB)

Explicit GC.Collect() call (generally avoid)

Low memory situation

Application entering idle state

IDisposable Pattern:

Used for cleaning up unmanaged resources (file handles, database connections, network sockets).

// Proper IDisposable implementation
public class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;
    private bool _disposed = false;
    
    public DatabaseConnection(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
    }
    
    // Public Dispose method
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this); // Prevent finalizer from running
    }
    
    // Protected Dispose method
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Dispose managed resources
                _connection?.Dispose();
            }
            
            // Clean up unmanaged resources (if any)
            // ...
            
            _disposed = true;
        }
    }
    
    // Finalizer (only if you have unmanaged resources)
    ~DatabaseConnection()
    {
        Dispose(false);
    }
}

// Usage with using statement (preferred)
using (var db = new DatabaseConnection(connString))
{
    // Use connection
} // Dispose called automatically

// C# 8.0+ using declaration
using var db = new DatabaseConnection(connString);
// Dispose called at end of scope

Memory Leak Prevention - Real Example from Enterprise64:

// Problem: Event handler causing memory leak
public class OrderProcessor
{
    public OrderProcessor()
    {
        // BAD - creates memory leak
        EventBus.OrderCreated += HandleOrderCreated;
    }
    
    private void HandleOrderCreated(object sender, OrderEventArgs e)
    {
        // Process order
    }
}

// Solution 1: Unsubscribe in Dispose
public class OrderProcessor : IDisposable
{
    public OrderProcessor()
    {
        EventBus.OrderCreated += HandleOrderCreated;
    }
    
    public void Dispose()
    {
        EventBus.OrderCreated -= HandleOrderCreated;
    }
}

// Solution 2: Use weak event pattern
public class OrderProcessor
{
    public OrderProcessor()
    {
        WeakEventManager<EventBus, OrderEventArgs>
            .AddHandler(EventBus.Instance, "OrderCreated", HandleOrderCreated);
    }
}

Performance Optimization Tips from My Experience:

1. Object Pooling for High-Frequency Allocations:

// At Enterprise64, we processed millions of shipment labels daily
// Using object pooling reduced Gen 2 collections by 60%

public class LabelPrinterPool
{
    private static ObjectPool<LabelPrinter> _pool = 
        new DefaultObjectPool<LabelPrinter>(
            new DefaultPooledObjectPolicy<LabelPrinter>(), 
            maxRetained: 100
        );
    
    public async Task PrintLabelAsync(ShipmentData data)
    {
        var printer = _pool.Get();
        try
        {
            await printer.PrintAsync(data);
        }
        finally
        {
            _pool.Return(printer);
        }
    }
}

2. Avoid Boxing/Unboxing:

// BAD - boxing value types
object obj = 42; // Boxing
int value = (int)obj; // Unboxing

// GOOD - use generics
public class Container<T>
{
    private T _value;
    public void Set(T value) => _value = value;
    public T Get() => _value;
}

3. Use Span<T> and Memory<T> for High-Performance Scenarios:

// Traditional approach - creates multiple string objects
public string ProcessLargeText(string text)
{
    var result = text.Substring(0, 100);
    result = result.Trim();
    result = result.ToUpper();
    return result;
}

// Better - uses Span<T>, no allocations
public string ProcessLargeText(string text)
{
    var span = text.AsSpan(0, Math.Min(100, text.Length));
    span = span.Trim();
    return span.ToString().ToUpper(); // Single allocation
}

Key Resources:

Memory Management: https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

IDisposable Pattern: https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose

Span<T> and Memory<T>: https://docs.microsoft.com/en-us/archive/msdn-magazine/2018/january/csharp-all-about-span-exploring-a-new-net-mainstay

## Q10: Explain LINQ and Expression Trees. How do they work internally?

Answer:

LINQ (Language Integrated Query): Unified syntax for querying different data sources (objects, databases, XML, etc.).

LINQ Providers:

LINQ to Objects: Queries in-memory collections

LINQ to SQL / EF: Translates to SQL queries

LINQ to XML: Queries XML documents

Custom providers: Can create your own

Query Syntax vs Method Syntax:

// Query syntax
var results = from p in products
              where p.Price > 100
              orderby p.Name
              select new { p.Name, p.Price };

// Method syntax (what query syntax compiles to)
var results = products
    .Where(p => p.Price > 100)
    .OrderBy(p => p.Name)
    .Select(p => new { p.Name, p.Price });

Expression Trees:

Expression trees represent code as data structures. Enable analyzing, modifying, or executing code at runtime.

// Lambda expression - compiled to IL
Func<int, int, int> add = (a, b) => a + b;
var result = add(3, 4); // Executes immediately

// Expression tree - represents the lambda as data
Expression<Func<int, int, int>> addExpr = (a, b) => a + b;
// Can analyze the expression
var body = addExpr.Body; // BinaryExpression
var left = ((BinaryExpression)body).Left; // ParameterExpression
var right = ((BinaryExpression)body).Right; // ParameterExpression

// Can compile and execute
var compiled = addExpr.Compile();
var result = compiled(3, 4);

How EF Core Uses Expression Trees:

// This LINQ query
var products = dbContext.Products
    .Where(p => p.Price > 100 && p.CategoryId == 5)
    .OrderBy(p => p.Name)
    .ToList();

// Gets converted to Expression Tree, then to SQL:
SELECT * FROM Products
WHERE Price > 100 AND CategoryId = 5
ORDER BY Name

// The Where() method signature shows it accepts an Expression:
public static IQueryable<T> Where<T>(
    this IQueryable<T> source,
    Expression<Func<T, bool>> predicate)
{
    // EF Core analyzes the expression tree
    // Converts to SQL AST
    // Generates parameterized SQL
}

Real-world Example from DigiTrends - Dynamic Filtering:

// Building dynamic queries using Expression Trees
public class DynamicQueryBuilder<T>
{
    public IQueryable<T> ApplyFilters(
        IQueryable<T> query, 
        Dictionary<string, object> filters)
    {
        var parameter = Expression.Parameter(typeof(T), "x");
        Expression combinedExpression = Expression.Constant(true);
        
        foreach (var filter in filters)
        {
            var property = Expression.Property(parameter, filter.Key);
            var value = Expression.Constant(filter.Value);
            var equals = Expression.Equal(property, value);
            
            combinedExpression = Expression.AndAlso(
                combinedExpression, 
                equals);
        }
        
        var lambda = Expression.Lambda<Func<T, bool>>(
            combinedExpression, 
            parameter);
        
        return query.Where(lambda);
    }
}

// Usage
var builder = new DynamicQueryBuilder<Product>();
var filters = new Dictionary<string, object>
{
    { "CategoryId", 5 },
    { "IsActive", true }
};

var products = builder.ApplyFilters(
    dbContext.Products, 
    filters
).ToList();

// Generates: 
// SELECT * FROM Products 
// WHERE CategoryId = 5 AND IsActive = 1

LINQ Performance Tips:

1. Deferred Execution - Queries don't execute until enumerated:

var query = products.Where(p => p.Price > 100); // Not executed yet
var count = query.Count(); // Executes: SELECT COUNT(*) FROM Products WHERE Price > 100
var list = query.ToList();  // Executes again: SELECT * FROM Products WHERE Price > 100

// Better - materialize once
var list = products.Where(p => p.Price > 100).ToList();
var count = list.Count; // In-memory count

2. N+1 Query Problem:

// BAD - N+1 queries
var orders = dbContext.Orders.ToList(); // 1 query
foreach (var order in orders)
{
    var customer = order.Customer; // N queries (lazy loading)
}

// GOOD - Single query with Include
var orders = dbContext.Orders
    .Include(o => o.Customer)
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
    .ToList(); // Single query with joins

3. Projection - Select Only Needed Columns:

// BAD - loads entire entity
var products = dbContext.Products
    .Where(p => p.CategoryId == 5)
    .ToList();

// GOOD - projects to DTO
var products = dbContext.Products
    .Where(p => p.CategoryId == 5)
    .Select(p => new ProductDto 
    { 
        Id = p.Id, 
        Name = p.Name, 
        Price = p.Price 
    })
    .ToList();

4. AsNoTracking for Read-Only Queries:

// With tracking (default) - EF creates change tracking overhead
var products = dbContext.Products.Where(p => p.CategoryId == 5).ToList();

// Without tracking - better performance for read-only
var products = dbContext.Products
    .AsNoTracking()
    .Where(p => p.CategoryId == 5)
    .ToList();

Key Resources:

LINQ Documentation: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/

Expression Trees: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/

EF Core Performance: https://docs.microsoft.com/en-us/ef/core/performance/