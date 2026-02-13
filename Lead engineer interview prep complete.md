# 🚀 Lead Engineer Interview Preparation Guide
## VIDIZMO - Complete Study Resource

**Your 6+ Years Experience + This Guide = Interview Success**

---

## 📋 Table of Contents
1. [React & JavaScript](#react--javascript)
2. [Database Fundamentals](#database-fundamentals)  
3. [.NET & Microservices](#net--microservices)
4. [CI/CD, Docker & Kubernetes](#cicd-docker--kubernetes)
5. [Azure Cloud](#azure-cloud)
6. [Quick Reference Cheatsheet](#cheatsheet)

---

# React & JavaScript

## 🎥 YouTube Study Resources

### Essential Playlists
1. **React Complete Course** - Programming with Mosh  
   https://www.youtube.com/watch?v=SqcY0GlETPk
   
2. **React Hooks Deep Dive** - Codevolution  
   https://www.youtube.com/watch?v=cF2lQ_gZeA8
   
3. **JavaScript Advanced Concepts** - Traversy Media  
   https://www.youtube.com/watch?v=R9I85RhI7Cg

4. **React Performance Optimization** - Jack Herrington  
   https://www.youtube.com/watch?v=5fLW5Q5ODiE

## 💡 Top 10 Interview Questions

### Q1: Explain React Hooks and their use cases
**Answer:** Hooks let you use state and lifecycle in functional components.

**Key Hooks:**
- `useState`: Manage local state
- `useEffect`: Side effects (API calls, subscriptions)
- `useContext`: Access context
- `useMemo`: Memoize calculations
- `useCallback`: Memoize functions
- `useRef`: Access DOM or persist values

**Example:**
\`\`\`javascript
function UserProfile() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser().then(setUser);
  }, []); // Empty deps = run once
  
  return <div>{user?.name}</div>;
}
\`\`\`

### Q2: What is Virtual DOM and reconciliation?
**Answer:** Virtual DOM is a lightweight JS representation of the real DOM. React uses it to:
1. Create virtual tree on state change
2. Compare (diff) with previous tree  
3. Calculate minimum changes needed
4. Batch update real DOM

**Performance benefit:** Only updates what changed, not entire page.

### Q3: useMemo vs useCallback
- `useMemo`: Memoizes VALUES (computation result)
- `useCallback`: Memoizes FUNCTIONS

\`\`\`javascript
const total = useMemo(() => 
  items.reduce((sum, item) => sum + item.price, 0), 
  [items]
);

const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
\`\`\`

### Q4: Context API vs Redux
| Context | Redux |
|---------|-------|
| Built-in | External library |
| Simple | More features |
| Good for: theme, auth | Good for: complex state |

### Q5: JavaScript Closures in React
A closure is when a function remembers variables from outer scope.

\`\`\`javascript
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1); // Closure captures count
    }, 1000);
    return () => clearInterval(timer);
  }, []);
}
\`\`\`

---

# Database Fundamentals

## 🎥 YouTube Study Resources

1. **SQL Complete Tutorial** - Programming with Mosh  
   https://www.youtube.com/watch?v=7S_tz1z_5bA
   
2. **Database Indexing** - Hussein Nasser  
   https://www.youtube.com/watch?v=-qNSXK7s7_w
   
3. **Window Functions** - Alex The Analyst  
   https://www.youtube.com/watch?v=Ww71knvhQ-s

4. **Database Sharding** - ByteByteGo  
   https://www.youtube.com/watch?v=5faMjKuB9bc

## 💡 Top 10 Interview Questions

### Q1: Types of Indexes
1. **Clustered**: Physical order of data (one per table)
2. **Non-Clustered**: Separate structure with pointers
3. **Unique**: Ensures uniqueness
4. **Filtered**: Index subset of rows
5. **Covering**: Includes all columns needed

\`\`\`sql
-- Covering index
CREATE INDEX IX_Orders_CustomerId 
ON Orders(CustomerId) 
INCLUDE (OrderDate, Total);
\`\`\`

### Q2: Common Table Expressions (CTEs)
Temporary named result sets.

\`\`\`sql
WITH TopCustomers AS (
  SELECT CustomerId, SUM(Total) as TotalSpent
  FROM Orders
  GROUP BY CustomerId
  HAVING SUM(Total) > 10000
)
SELECT * FROM TopCustomers;
\`\`\`

**Recursive CTE:**
\`\`\`sql
WITH OrgChart AS (
  -- Anchor
  SELECT EmpId, ManagerId, Name, 1 AS Level
  FROM Employees WHERE ManagerId IS NULL
  UNION ALL
  -- Recursive
  SELECT e.EmpId, e.ManagerId, e.Name, oc.Level + 1
  FROM Employees e
  JOIN OrgChart oc ON e.ManagerId = oc.EmpId
)
SELECT * FROM OrgChart;
\`\`\`

### Q3: Window Functions
Perform calculations across related rows without GROUP BY.

\`\`\`sql
SELECT 
  EmployeeId,
  Salary,
  -- Ranking
  ROW_NUMBER() OVER (ORDER BY Salary DESC) as RowNum,
  RANK() OVER (ORDER BY Salary DESC) as Rank,
  -- Running total
  SUM(Salary) OVER (ORDER BY EmployeeId) as RunningTotal,
  -- Previous row
  LAG(Salary, 1) OVER (ORDER BY EmployeeId) as PrevSalary
FROM Employees;
\`\`\`

### Q4: Temp Tables vs Table Variables

| Feature | #TempTable | @TableVariable |
|---------|-----------|----------------|
| Scope | Session | Batch |
| Size | Large data | Small data |
| Indexes | Yes | Limited |
| Statistics | Yes | No |

\`\`\`sql
-- Temp table
CREATE TABLE #Orders (
  OrderId INT,
  Total DECIMAL(10,2)
);

-- Table variable  
DECLARE @Orders TABLE (
  OrderId INT,
  Total DECIMAL(10,2)
);
\`\`\`

### Q5: Query Optimization
1. **Use indexes on WHERE/JOIN columns**
2. **Select only needed columns** (not SELECT *)
3. **Avoid functions on indexed columns**
4. **Use EXISTS instead of IN for large sets**
5. **Update statistics regularly**

\`\`\`sql
-- ❌ Bad
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2024;

-- ✅ Good
SELECT OrderId, Total 
FROM Orders 
WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
\`\`\`

### Q6: Database Scaling

**Vertical Scaling:** Add more CPU/RAM (simple but limited)

**Horizontal Scaling:** Add more servers
- **Replication**: Master-Slave (read replicas)
- **Sharding**: Split data across databases
- **Partitioning**: Split within same database

\`\`\`csharp
// Sharding example
int shardId = userId % 3; // 3 shards
var connectionString = GetShardConnection(shardId);
\`\`\`

---

# .NET & Microservices

## 🎥 YouTube Study Resources

1. **ASP.NET Core Complete** - freeCodeCamp  
   https://www.youtube.com/watch?v=C5cnZ-gZy2I
   
2. **Microservices with .NET** - Milan Jovanović  
   https://www.youtube.com/watch?v=DgVjEo3OGBI
   
3. **gRPC in .NET** - Nick Chapsas  
   https://www.youtube.com/watch?v=QyxCX2GYHxk

4. **Ocelot API Gateway** - Code with Mukesh  
   https://www.youtube.com/watch?v=Bfk7XkWL5r0

## 💡 Top 10 Interview Questions

### Q1: .NET Framework vs .NET Core vs .NET 5+
| Feature | .NET Framework | .NET Core/.NET 5+ |
|---------|---------------|-------------------|
| Platform | Windows only | Cross-platform |
| Performance | Good | Excellent |
| Open Source | No | Yes |
| Future | Maintenance only | Active development |

### Q2: Dependency Injection Lifetimes
\`\`\`csharp
// Transient: New instance every request
builder.Services.AddTransient<IEmailService, EmailService>();

// Scoped: One instance per HTTP request
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// Singleton: One instance for application
builder.Services.AddSingleton<ICacheService, CacheService>();
\`\`\`

### Q3: Microservices Communication

**1. REST (HTTP/JSON):**
\`\`\`csharp
[HttpGet("{id}")]
public async Task<OrderDto> GetOrder(int id) {
  return await _service.GetOrderAsync(id);
}
\`\`\`

**2. gRPC (Binary, HTTP/2):**
\`\`\`protobuf
service OrderService {
  rpc GetOrder (OrderRequest) returns (OrderResponse);
}
\`\`\`

**3. Message Queue (Async):**
\`\`\`csharp
// Publish event
await _eventBus.PublishAsync(new OrderCreatedEvent {
  OrderId = order.Id
});
\`\`\`

| Method | Use Case |
|--------|----------|
| REST | Public APIs, simple CRUD |
| gRPC | Internal, high-performance |
| Queue | Async, event-driven |

### Q4: Ocelot API Gateway
Routes requests to microservices.

\`\`\`json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/orders/{everything}",
      "DownstreamHostAndPorts": [
        { "Host": "order-service", "Port": 5001 }
      ],
      "UpstreamPathTemplate": "/orders/{everything}",
      "LoadBalancerOptions": {
        "Type": "RoundRobin"
      },
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer"
      }
    }
  ]
}
\`\`\`

### Q5: Async/Await Best Practices
\`\`\`csharp
// ✅ Good - Async all the way
public async Task<Order> GetOrderAsync(int id) {
  return await _repository.GetOrderAsync(id);
}

// ❌ Bad - Blocking
public Order GetOrder(int id) {
  return _repository.GetOrderAsync(id).Result; // Deadlock risk!
}

// ✅ Parallel execution
var orderTask = GetOrderAsync(orderId);
var userTask = GetUserAsync(userId);
await Task.WhenAll(orderTask, userTask);
\`\`\`

### Q6: IEnumerable vs IQueryable
\`\`\`csharp
// ❌ Bad - Loads all to memory
IEnumerable<Order> orders = _context.Orders.ToList()
  .Where(o => o.Total > 1000);

// ✅ Good - Filters in database
IQueryable<Order> orders = _context.Orders
  .Where(o => o.Total > 1000);
var result = await orders.ToListAsync();
// SQL: SELECT * FROM Orders WHERE Total > 1000
\`\`\`

---

# CI/CD, Docker & Kubernetes

## 🎥 YouTube Study Resources

1. **Docker Tutorial** - TechWorld with Nana  
   https://www.youtube.com/watch?v=3c-iBn73dDE
   
2. **Kubernetes for Beginners** - freeCodeCamp  
   https://www.youtube.com/watch?v=X48VuDVv0do
   
3. **GitHub Actions CI/CD** - freeCodeCamp  
   https://www.youtube.com/watch?v=R8_veQiYBjI

4. **Azure DevOps Pipeline** - Pragmatic Works  
   https://www.youtube.com/watch?v=NuYDAs3kNV8

## 💡 Top 10 Interview Questions

### Q1: Docker vs Virtual Machines
| Docker | VM |
|--------|-----|
| Shares host OS | Full OS per VM |
| MBs | GBs |
| Seconds startup | Minutes |
| Lightweight | Heavy |

### Q2: Dockerfile Optimization
\`\`\`dockerfile
# Multi-stage build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp.csproj", "./"]
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app

# Runtime image (smaller)
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApp.dll"]
\`\`\`

**Optimization tips:**
1. Use multi-stage builds
2. Layer caching (COPY package files first)
3. Use Alpine images
4. .dockerignore file
5. Run as non-root user

### Q3: Kubernetes Core Concepts
\`\`\`yaml
# Deployment - manages pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 80

---
# Service - exposes pods
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
\`\`\`

### Q4: CI/CD Pipeline Stages
\`\`\`
1. Code Commit (Git push)
2. Build (Compile, Restore)
3. Test (Unit, Integration)
4. Security Scan (SAST, Dependencies)
5. Docker Build & Push
6. Deploy to Dev
7. Deploy to Staging
8. Deploy to Production (Manual approval)
\`\`\`

### Q5: Deployment Strategies

**1. Blue-Green:** Two identical environments, switch traffic
**2. Canary:** Gradually roll out to subset of users
**3. Rolling:** Update pods one by one

\`\`\`bash
# Rolling update
kubectl set image deployment/myapp myapp=myapp:2.0.0

# Rollback
kubectl rollout undo deployment/myapp
\`\`\`

---

# Azure Cloud

## 🎥 YouTube Study Resources

1. **Azure Fundamentals** - freeCodeCamp  
   https://www.youtube.com/watch?v=NKEFWyqJ5XA
   
2. **Azure Kubernetes Service** - Microsoft  
   https://www.youtube.com/watch?v=4BwyqmRTrx8
   
3. **Deploy Microservices to Azure** - dotnet  
   https://www.youtube.com/watch?v=CqCDOosvZIk

## 💡 Top 10 Interview Questions

### Q1: Azure Core Services
- **Compute:** VMs, App Service, AKS, Functions
- **Storage:** Blob, Files, Queue, Table
- **Database:** SQL Database, Cosmos DB, Redis
- **Networking:** VNet, Load Balancer, Application Gateway
- **Security:** Key Vault, Active Directory

### Q2: Deploy Microservices to Azure

**Option 1: Azure Kubernetes Service (AKS)**
\`\`\`bash
# Create AKS cluster
az aks create --name myAKS --resource-group myRG --node-count 3

# Get credentials
az aks get-credentials --name myAKS --resource-group myRG

# Deploy
kubectl apply -f deployment.yaml
\`\`\`

**Option 2: Azure App Service (Containers)**
\`\`\`bash
az webapp create \
  --name myApp \
  --resource-group myRG \
  --plan myPlan \
  --deployment-container-image-name myregistry.azurecr.io/myapp:1.0
\`\`\`

### Q3: Azure Security Best Practices
1. Use Managed Identities (no passwords in code)
2. Store secrets in Key Vault
3. Enable Azure AD authentication
4. Use Private Endpoints
5. Configure Network Security Groups (NSG)
6. Enable encryption at rest and in transit
7. Implement RBAC (least privilege)
8. Enable Azure Security Center

\`\`\`csharp
// Use Managed Identity
var credential = new DefaultAzureCredential();
var keyVaultClient = new SecretClient(
  new Uri("https://myvault.vault.azure.net"), 
  credential
);
var secret = await keyVaultClient.GetSecretAsync("ConnectionString");
\`\`\`

### Q4: Azure DevOps Pipeline
\`\`\`yaml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: DotNetCoreCLI@2
      inputs:
        command: 'build'

- stage: Deploy
  jobs:
  - deployment: DeployJob
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            inputs:
              action: 'deploy'
              manifests: 'k8s/deployment.yaml'
\`\`\`

---

# 📝 Quick Reference Cheatsheet

## React Essentials
\`\`\`javascript
// Hooks
useState, useEffect, useContext, useReducer, 
useCallback, useMemo, useRef

// Performance
React.memo(), useMemo(), useCallback(), 
React.lazy(), Suspense
\`\`\`

## SQL Quick Reference
\`\`\`sql
-- Index
CREATE INDEX IX_Col ON Table(Col);

-- CTE
WITH MyCTE AS (SELECT ...) 
SELECT * FROM MyCTE;

-- Window Function
ROW_NUMBER() OVER (PARTITION BY X ORDER BY Y)

-- Temp Table
CREATE TABLE #Temp (Id INT);
\`\`\`

## .NET Patterns
\`\`\`csharp
// DI Lifetimes
AddTransient // New every time
AddScoped    // One per request
AddSingleton // One forever

// Async
public async Task<T> MethodAsync() {
  return await OperationAsync();
}
\`\`\`

## Docker Commands
\`\`\`bash
# Build
docker build -t image:tag .

# Run
docker run -d -p 8080:80 image:tag

# Compose
docker-compose up -d
docker-compose down

# Clean
docker system prune -a
\`\`\`

## Kubernetes Commands
\`\`\`bash
# Apply
kubectl apply -f file.yaml

# Get resources
kubectl get pods
kubectl get services

# Logs
kubectl logs pod-name

# Scale
kubectl scale deployment name --replicas=5

# Update
kubectl set image deployment/name container=image:v2

# Rollback
kubectl rollout undo deployment/name
\`\`\`

## Azure CLI
\`\`\`bash
# Login
az login

# AKS
az aks create --name myAKS --resource-group myRG
az aks get-credentials --name myAKS --resource-group myRG

# ACR
az acr create --name myACR --resource-group myRG
az acr login --name myACR

# App Service
az webapp create --name myApp --resource-group myRG
\`\`\`

## Architecture Patterns
- **Microservices:** Small, independent services
- **Event-Driven:** Services communicate via events
- **CQRS:** Separate read/write models
- **API Gateway:** Single entry point
- **Saga:** Distributed transactions

## Interview Tips ✨
1. **Understand WHY** technologies exist
2. **Real examples** from your 6+ years experience
3. **Trade-offs** - discuss pros/cons
4. **Scaling** - show you understand performance
5. **Be honest** - say "I don't know" but show willingness to learn
6. **Ask questions** - show curiosity

---

## 🎯 Final Preparation Checklist

### Week Before Interview
- [ ] Review this guide thoroughly
- [ ] Watch key YouTube videos
- [ ] Practice coding on whiteboard/screen
- [ ] Review your past projects
- [ ] Prepare questions to ask interviewer
- [ ] Test your setup (camera, mic for video interview)

### Day Before
- [ ] Relax and get good sleep
- [ ] Review cheatsheet one more time
- [ ] Prepare examples from your experience
- [ ] Set up interview space

### Interview Day
- [ ] Be 10 minutes early
- [ ] Have this guide nearby for quick reference
- [ ] Stay calm and confident
- [ ] Remember: They want you to succeed!

---

**You've got this! Your 6+ years of Microsoft stack experience + this preparation = Success! 🚀**

Good luck with your VIDIZMO Lead Engineer interview!
