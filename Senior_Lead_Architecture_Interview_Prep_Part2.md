# Senior/Lead/Architecture Role Interview Preparation - Part 2

# 4. .NET Core & ASP.NET

## Q11: Explain the .NET Core middleware pipeline and how to create custom middleware.

Answer:

The middleware pipeline is a series of components that handle HTTP requests and responses in sequence. Each middleware can:

Process an incoming request before the next middleware

Process an outgoing response after the next middleware

Short-circuit the pipeline (not call next middleware)

Middleware Execution Order:

public void Configure(IApplicationBuilder app)
{
    // Request flow: 1 -> 2 -> 3 -> 4 -> 5
    // Response flow: 5 -> 4 -> 3 -> 2 -> 1
    
    app.UseExceptionHandler();      // 1. Exception handling
    app.UseHttpsRedirection();      // 2. HTTPS redirect
    app.UseStaticFiles();          // 3. Static files
    app.UseRouting();              // 4. Routing
    app.UseAuthentication();       // 5. Authentication
    app.UseAuthorization();        // 6. Authorization
    app.UseEndpoints(endpoints =>  // 7. Endpoints
    {
        endpoints.MapControllers();
    });
}

Creating Custom Middleware - Example from Enterprise64:

I created a request logging and correlation ID middleware:

// Middleware class
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    
    public RequestLoggingMiddleware(
        RequestDelegate next, 
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Generate correlation ID
        var correlationId = Guid.NewGuid().ToString();
        context.Items["CorrelationId"] = correlationId;
        context.Response.Headers.Add("X-Correlation-ID", correlationId);
        
        // Log request
        _logger.LogInformation(
            "Request {CorrelationId}: {Method} {Path} started",
            correlationId,
            context.Request.Method,
            context.Request.Path);
        
        var sw = Stopwatch.StartNew();
        
        try
        {
            // Call next middleware
            await _next(context);
        }
        finally
        {
            sw.Stop();
            
            // Log response
            _logger.LogInformation(
                "Request {CorrelationId}: {Method} {Path} completed " +
                "in {ElapsedMs}ms with status {StatusCode}",
                correlationId,
                context.Request.Method,
                context.Request.Path,
                sw.ElapsedMilliseconds,
                context.Response.StatusCode);
        }
    }
}

// Extension method for easy registration
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}

// Usage in Startup.cs
public void Configure(IApplicationBuilder app)
{
    app.UseRequestLogging(); // Custom middleware
    app.UseRouting();
    app.UseEndpoints(endpoints => endpoints.MapControllers());
}

Real-world Middleware Example - API Key Authentication:

public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IConfiguration _configuration;
    
    public ApiKeyMiddleware(RequestDelegate next, IConfiguration configuration)
    {
        _next = next;
        _configuration = configuration;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Skip authentication for health check endpoint
        if (context.Request.Path.StartsWithSegments("/health"))
        {
            await _next(context);
            return;
        }
        
        // Check for API key in header
        if (!context.Request.Headers.TryGetValue("X-API-Key", out var apiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key missing");
            return; // Short-circuit pipeline
        }
        
        // Validate API key
        var validApiKey = _configuration["ApiKey"];
        if (apiKey != validApiKey)
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }
        
        // Authentication successful, continue pipeline
        await _next(context);
    }
}

Key Resources:

ASP.NET Core Middleware: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/

Custom Middleware: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write

## Q12: Explain Dependency Injection in .NET Core and its lifetime scopes.

Answer:

Dependency Injection (DI) is a design pattern built into .NET Core that enables loose coupling and testability.

Service Lifetimes:

1. Transient: Created each time they are requested. New instance per injection.

Use for: Lightweight, stateless services

Example: Logging services, utility services

2. Scoped: Created once per client request (HTTP request in web apps).

Use for: Per-request services, DbContext

Example: Entity Framework DbContext, unit of work

3. Singleton: Created once for the application lifetime.

Use for: Expensive-to-create services, caches, configuration

Example: Configuration, memory cache, app-wide state

// Startup.cs or Program.cs (.NET 6+)
public void ConfigureServices(IServiceCollection services)
{
    // Transient - new instance every time
    services.AddTransient<IEmailService, EmailService>();
    services.AddTransient<INotificationService, NotificationService>();
    
    // Scoped - one instance per request
    services.AddScoped<IOrderService, OrderService>();
    services.AddScoped<IProductRepository, ProductRepository>();
    services.AddDbContext<AppDbContext>(options => 
        options.UseSqlServer(connectionString)); // DbContext is scoped by default
    
    // Singleton - one instance for app lifetime
    services.AddSingleton<IMemoryCache, MemoryCache>();
    services.AddSingleton<IConfiguration>(configuration);
    services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
}

Real-world DI Architecture from DigiTrends:

// Repository Pattern with DI
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T> GetByIdAsync(int id) => 
        await _dbSet.FindAsync(id);
    
    public async Task<IEnumerable<T>> GetAllAsync() => 
        await _dbSet.ToListAsync();
    
    // ... other methods
}

// Service Layer
public interface IProductService
{
    Task<ProductDto> GetProductAsync(int id);
    Task<IEnumerable<ProductDto>> GetAllProductsAsync();
    Task CreateProductAsync(CreateProductDto dto);
}

public class ProductService : IProductService
{
    private readonly IRepository<Product> _productRepo;
    private readonly IMapper _mapper;
    private readonly ILogger<ProductService> _logger;
    private readonly ICacheService _cache;
    
    // Constructor injection
    public ProductService(
        IRepository<Product> productRepo,
        IMapper mapper,
        ILogger<ProductService> logger,
        ICacheService cache)
    {
        _productRepo = productRepo;
        _mapper = mapper;
        _logger = logger;
        _cache = cache;
    }
    
    public async Task<ProductDto> GetProductAsync(int id)
    {
        var cacheKey = $"product_{id}";
        
        // Try cache first
        var cached = _cache.Get<ProductDto>(cacheKey);
        if (cached != null) return cached;
        
        // Get from database
        var product = await _productRepo.GetByIdAsync(id);
        var dto = _mapper.Map<ProductDto>(product);
        
        // Cache result
        _cache.Set(cacheKey, dto, TimeSpan.FromMinutes(10));
        
        return dto;
    }
}

// Registration
services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
services.AddScoped<IProductService, ProductService>();
services.AddAutoMapper(typeof(Startup));
services.AddSingleton<ICacheService, RedisCacheService>();

Common DI Pitfalls and Solutions:

1. Captive Dependencies - Singleton depending on Scoped:

// BAD - Singleton capturing Scoped service
public class CacheService // Singleton
{
    private readonly AppDbContext _context; // Scoped - memory leak!
    
    public CacheService(AppDbContext context)
    {
        _context = context; // Context will never be disposed
    }
}

// GOOD - Use IServiceProvider to resolve per operation
public class CacheService // Singleton
{
    private readonly IServiceProvider _serviceProvider;
    
    public CacheService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public async Task<Product> GetProductAsync(int id)
    {
        using var scope = _serviceProvider.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        return await context.Products.FindAsync(id);
    }
}

2. Constructor Over-Injection:

// BAD - too many dependencies
public class OrderService
{
    public OrderService(
        IOrderRepository orderRepo,
        ICustomerRepository customerRepo,
        IProductRepository productRepo,
        IInventoryService inventoryService,
        IPaymentService paymentService,
        IEmailService emailService,
        ILogger logger,
        IMapper mapper,
        IConfiguration config,
        ICacheService cache) // 10 dependencies!
    {
        // This class is doing too much - violates SRP
    }
}

// GOOD - refactor into smaller services
public class OrderService
{
    private readonly IOrderRepository _orderRepo;
    private readonly IOrderValidationService _validationService;
    private readonly IOrderNotificationService _notificationService;
    
    public OrderService(
        IOrderRepository orderRepo,
        IOrderValidationService validationService,
        IOrderNotificationService notificationService)
    {
        _orderRepo = orderRepo;
        _validationService = validationService;
        _notificationService = notificationService;
    }
}

Key Resources:

Dependency Injection: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection

Service Lifetimes: https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection

## Q13: How do you implement API versioning and what strategies have you used?

Answer:

API Versioning Strategies:

1. URL Path Versioning (Most Common):

// api/v1/products
// api/v2/products

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1() => Ok(new { version = "1.0" });
    
    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetV2() => Ok(new { version = "2.0" });
}

// Startup configuration
services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true; // Add version info to response headers
});

2. Query String Versioning:

// api/products?api-version=1.0
// api/products?api-version=2.0

services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new QueryStringApiVersionReader("api-version");
});

3. Header Versioning:

// Header: X-API-Version: 1.0

services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new HeaderApiVersionReader("X-API-Version");
});

4. Media Type Versioning (Content Negotiation):

// Accept: application/json;version=1.0

services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new MediaTypeApiVersionReader();
});

Real-world Example from DigiTrends - Mutual Funds API:

// V1 - Original implementation
[ApiController]
[Route("api/v{version:apiVersion}/funds")]
[ApiVersion("1.0")]
public class FundsControllerV1 : ControllerBase
{
    private readonly IFundService _fundService;
    
    public FundsControllerV1(IFundService fundService)
    {
        _fundService = fundService;
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<FundDto>>> GetAll()
    {
        var funds = await _fundService.GetAllFundsAsync();
        return Ok(funds);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<FundDto>> Get(int id)
    {
        var fund = await _fundService.GetFundByIdAsync(id);
        if (fund == null) return NotFound();
        return Ok(fund);
    }
}

// V2 - Enhanced with pagination, filtering, and performance metrics
[ApiController]
[Route("api/v{version:apiVersion}/funds")]
[ApiVersion("2.0")]
public class FundsControllerV2 : ControllerBase
{
    private readonly IFundServiceV2 _fundService;
    
    public FundsControllerV2(IFundServiceV2 fundService)
    {
        _fundService = fundService;
    }
    
    [HttpGet]
    public async Task<ActionResult<PagedResult<FundDtoV2>>> GetAll(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20,
        [FromQuery] string category = null,
        [FromQuery] decimal? minReturn = null)
    {
        var result = await _fundService.GetFundsAsync(
            page, pageSize, category, minReturn);
        
        // Add pagination metadata to headers
        Response.Headers.Add("X-Total-Count", result.TotalCount.ToString());
        Response.Headers.Add("X-Page-Number", page.ToString());
        Response.Headers.Add("X-Page-Size", pageSize.ToString());
        
        return Ok(result);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<FundDetailDtoV2>> Get(int id)
    {
        // V2 includes performance history and risk metrics
        var fund = await _fundService.GetFundDetailsAsync(id);
        if (fund == null) return NotFound();
        return Ok(fund);
    }
}

// DTOs
public class FundDto // V1
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal CurrentValue { get; set; }
}

public class FundDtoV2 // V2
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal CurrentValue { get; set; }
    public decimal OneYearReturn { get; set; }
    public decimal ThreeYearReturn { get; set; }
    public string RiskRating { get; set; }
}

public class FundDetailDtoV2 : FundDtoV2
{
    public List<PerformancePoint> PerformanceHistory { get; set; }
    public List<Holding> TopHoldings { get; set; }
}

// Startup
services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(2, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new HeaderApiVersionReader("X-API-Version"),
        new QueryStringApiVersionReader("api-version")
    );
});

services.AddScoped<IFundService, FundService>();
services.AddScoped<IFundServiceV2, FundServiceV2>();

Deprecation Strategy:

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0", Deprecated = true)] // Mark as deprecated
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    [Obsolete("This endpoint is deprecated. Use v2 instead.")]
    public IActionResult GetV1()
    {
        // Add deprecation warning to response
        Response.Headers.Add("X-API-Deprecated", "true");
        Response.Headers.Add("X-API-Sunset-Date", "2024-12-31");
        Response.Headers.Add("X-API-Deprecation-Info", 
            "This version will be removed on 2024-12-31. Please migrate to v2.");
        
        return Ok(new { version = "1.0" });
    }
}

Key Resources:

API Versioning: https://github.com/dotnet/aspnet-api-versioning

Best Practices: https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design

# 5. Microsoft Azure

## Q14: Explain Azure App Service vs Azure Functions vs Azure Kubernetes Service. When would you use each?

Answer:

1. Azure App Service (PaaS):

Managed platform for web apps, REST APIs, mobile backends

Always-on, auto-scaling, easy deployment

Built-in CI/CD, deployment slots, custom domains, SSL

Pricing: Based on App Service Plan (reserved capacity)

Use when:

Traditional web applications or APIs

Need predictable performance

Long-running processes

Require WebSockets or always-on functionality

Example from DigiTrends: Hosted the B2B retail platform on App Service with:

Production and staging slots for zero-downtime deployments

Auto-scaling rules based on CPU and HTTP queue length

Application Insights for monitoring

Handled 10,000+ concurrent users during peak hours

2. Azure Functions (Serverless/FaaS):

Event-driven, serverless compute

Pay only for execution time

Automatic scaling to zero

Multiple trigger types: HTTP, Timer, Queue, Blob, Event Grid, etc.

Use when:

Event-driven or scheduled tasks

Sporadic or unpredictable load

Short-lived operations (< 10 minutes for Consumption plan)

Cost optimization for infrequent operations

Example from Enterprise64: Used Azure Functions for:

Processing shipment label generation (Queue trigger)

Daily inventory reports (Timer trigger - CRON schedule)

Image resizing when uploaded to Blob Storage (Blob trigger)

Webhook endpoints for third-party integrations

// Azure Function - Queue Trigger Example
[FunctionName("ProcessShipmentLabel")]
public async Task Run(
    [QueueTrigger("shipment-labels")] ShipmentMessage message,
    [Blob("labels/{rand-guid}.pdf", FileAccess.Write)] Stream outputBlob,
    ILogger log)
{
    log.LogInformation($"Processing shipment: {message.ShipmentId}");
    
    var labelPdf = await _labelService.GenerateLabelAsync(message);
    await labelPdf.CopyToAsync(outputBlob);
    
    await _notificationService.SendAsync(
        message.Email, 
        "Shipment label ready");
}

// Timer Trigger - Daily Report
[FunctionName("DailyInventoryReport")]
public async Task GenerateReport(
    [TimerTrigger("0 0 2 * * *")] TimerInfo timer, // 2 AM daily
    ILogger log)
{
    log.LogInformation("Generating daily inventory report");
    
    var report = await _reportService.GenerateInventoryReportAsync();
    await _emailService.SendReportAsync(report);
}

3. Azure Kubernetes Service (AKS):

Managed Kubernetes cluster

Container orchestration at scale

Full control over containerized apps

Supports microservices architectures

Use when:

Complex microservices architecture

Need portability across cloud providers

Advanced deployment strategies (canary, blue-green)

Large-scale distributed systems

Example: Would use AKS for:

Multi-tenant SaaS platform with 50+ microservices

Need for service mesh (Istio/Linkerd)

Advanced traffic management and A/B testing

Running mixed Linux/Windows containers

Comparison Table:

Feature          | App Service      | Functions           | AKS
----------------|------------------|---------------------|--------------------
Complexity      | Low              | Low-Medium          | High
Cost            | Fixed            | Pay-per-execution   | Variable
Scaling         | Auto (vertical)  | Auto (to zero)      | Manual/Auto
Cold Start      | No               | Yes (Consumption)   | No
Max Duration    | Unlimited        | 10 min / 230 min    | Unlimited
Control         | Limited          | Limited             | Full
Best For        | Web apps/APIs    | Event-driven tasks  | Microservices

Key Resources:

App Service: https://docs.microsoft.com/en-us/azure/app-service/

Azure Functions: https://docs.microsoft.com/en-us/azure/azure-functions/

AKS: https://docs.microsoft.com/en-us/azure/aks/

## Q15: How do you secure Azure resources and implement identity management?

Answer:

Azure Security Best Practices:

1. Azure Active Directory (Azure AD / Entra ID):

Centralized identity and access management

Multi-factor authentication (MFA)

Conditional Access policies

Single Sign-On (SSO)

// ASP.NET Core with Azure AD Authentication
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApi(Configuration.GetSection("AzureAd"));
    
    services.AddAuthorization(options =>
    {
        options.AddPolicy("AdminOnly", policy =>
            policy.RequireRole("Admin"));
        
        options.AddPolicy("ReadAccess", policy =>
            policy.RequireClaim("scope", "api.read"));
    });
}

// appsettings.json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "yourdomain.onmicrosoft.com",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id",
    "Audience": "api://your-api-id"
  }
}

// Controller with authorization
[Authorize(Policy = "AdminOnly")]
[ApiController]
[Route("api/[controller]")]
public class AdminController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var email = User.FindFirst(ClaimTypes.Email)?.Value;
        return Ok(new { userId, email });
    }
}

2. Managed Identity:

Eliminate credentials in code

System-assigned or User-assigned

Azure resources authenticate to other Azure services

// Using Managed Identity to access Key Vault
var keyVaultUrl = "https://myvault.vault.azure.net/";
var client = new SecretClient(
    new Uri(keyVaultUrl), 
    new DefaultAzureCredential()); // Uses Managed Identity

var secret = await client.GetSecretAsync("DatabasePassword");
var connectionString = secret.Value.Value;

// Access Azure SQL with Managed Identity
var connection = new SqlConnection(
    "Server=myserver.database.windows.net;Database=mydb;" +
    "Authentication=Active Directory Default");

// Access Blob Storage with Managed Identity
var blobServiceClient = new BlobServiceClient(
    new Uri("https://mystorage.blob.core.windows.net"),
    new DefaultAzureCredential());

3. Azure Key Vault:

Securely store secrets, keys, and certificates

Centralized secret management

Access auditing and logging

Real-world Example from DigiTrends Auditing Solution:

// Program.cs - Configure Key Vault
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, config) =>
        {
            var builtConfig = config.Build();
            var keyVaultUrl = builtConfig["KeyVaultUrl"];
            
            // Add Key Vault as configuration source
            config.AddAzureKeyVault(
                new Uri(keyVaultUrl),
                new DefaultAzureCredential());
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });

// Usage in application
public class PaymentService
{
    private readonly IConfiguration _configuration;
    
    public PaymentService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public async Task ProcessPaymentAsync(decimal amount)
    {
        // Secrets automatically retrieved from Key Vault
        var apiKey = _configuration["PaymentGatewayApiKey"];
        var secretKey = _configuration["PaymentGatewaySecret"];
        
        // Use secrets
    }
}

4. Role-Based Access Control (RBAC):

Grant minimum required permissions

Built-in roles: Owner, Contributor, Reader, custom roles

Apply at subscription, resource group, or resource level

5. Network Security:

Virtual Networks (VNets) for isolation

Network Security Groups (NSGs) for traffic filtering

Private Endpoints for Azure services

Azure Firewall / Application Gateway with WAF

6. Data Protection:

Encryption at rest (Azure Storage, SQL Database)

Encryption in transit (TLS 1.2+)

Azure Disk Encryption for VMs

Always Encrypted for sensitive database columns

7. Monitoring and Compliance:

Azure Monitor and Application Insights

Azure Security Center / Defender for Cloud

Azure Policy for compliance enforcement

Azure Sentinel for SIEM

Security Architecture Example:

Internet
   |
   v
Azure Front Door / Application Gateway (WAF enabled)
   |
   v
Azure App Service (VNet Integration)
   |
   v
Private Endpoint
   |
   v
Azure SQL Database (Firewall: Deny all, allow VNet only)
   |
   v
Managed Identity Authentication
   |
   v
Azure Key Vault (Secrets, Connection Strings)
   |
   v
Azure Storage (Private Endpoint, Encryption at rest)

All components:
- Enable Azure AD authentication
- Use Managed Identity (no credentials in code)
- Apply least privilege RBAC
- Enable diagnostic logging to Log Analytics
- Monitor with Azure Security Center

Key Resources:

Azure Security Best Practices: https://docs.microsoft.com/en-us/azure/security/fundamentals/best-practices-and-patterns

Managed Identity: https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/

Key Vault: https://docs.microsoft.com/en-us/azure/key-vault/

# 6. CI/CD & DevOps

## Q16: Explain your CI/CD pipeline setup. What tools and strategies do you use?

Answer:

CI/CD Pipeline Components:

1. Source Control:

Git (Azure DevOps, GitHub, GitLab)

Branch strategy: GitFlow or GitHub Flow

Protected main/master branch

Pull request reviews required

2. Continuous Integration (CI):

Triggered on every commit/PR

Build + Unit Tests + Code Analysis

Static code analysis (SonarQube)

Security scanning (SAST)

3. Continuous Deployment (CD):

Automated deployment to environments

Environment-specific configurations

Blue-Green or Canary deployments

Automated smoke tests

Real-world Azure DevOps Pipeline from DigiTrends:

# azure-pipelines.yml

trigger:
  branches:
    include:
      - main
      - develop
  paths:
    exclude:
      - README.md
      - docs/*

variables:
  buildConfiguration: 'Release'
  azureSubscription: 'MyAzureSubscription'
  appServiceName: 'myapp-$(Build.SourceBranchName)'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    # Restore dependencies
    - task: DotNetCoreCLI@2
      displayName: 'Restore NuGet packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
    
    # Build
    - task: DotNetCoreCLI@2
      displayName: 'Build solution'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '--configuration $(buildConfiguration)'
    
    # Run unit tests
    - task: DotNetCoreCLI@2
      displayName: 'Run unit tests'
      inputs:
        command: 'test'
        projects: '**/*Tests/*.csproj'
        arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage"'
    
    # Publish code coverage
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish code coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    
    # SonarQube analysis
    - task: SonarCloudPrepare@1
      displayName: 'Prepare SonarCloud'
      inputs:
        SonarCloud: 'SonarCloud'
        organization: 'myorg'
        scannerMode: 'MSBuild'
        projectKey: 'myproject'
    
    - task: SonarCloudAnalyze@1
      displayName: 'Run SonarCloud analysis'
    
    - task: SonarCloudPublish@1
      displayName: 'Publish SonarCloud results'
    
    # Publish artifact
    - task: DotNetCoreCLI@2
      displayName: 'Publish application'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      displayName: 'Publish build artifacts'
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

- stage: DeployDev
  displayName: 'Deploy to Development'
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/develop'))
  jobs:
  - deployment: DeployDev
    environment: 'Development'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadBuildArtifacts@0
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: 'drop'
          
          - task: AzureRmWebAppDeployment@4
            displayName: 'Deploy to App Service'
            inputs:
              azureSubscription: '$(azureSubscription)'
              appType: 'webApp'
              WebAppName: 'myapp-dev'
              package: '$(System.ArtifactsDirectory)/drop/**/*.zip'
              appSettings: '-ASPNETCORE_ENVIRONMENT Development'
          
          # Run smoke tests
          - task: PowerShell@2
            displayName: 'Run smoke tests'
            inputs:
              targetType: 'inline'
              script: |
                $response = Invoke-WebRequest -Uri "https://myapp-dev.azurewebsites.net/health"
                if ($response.StatusCode -ne 200) {
                  Write-Error "Health check failed"
                  exit 1
                }

- stage: DeployStaging
  displayName: 'Deploy to Staging'
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployStaging
    environment: 'Staging'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureRmWebAppDeployment@4
            displayName: 'Deploy to staging slot'
            inputs:
              azureSubscription: '$(azureSubscription)'
              appType: 'webApp'
              WebAppName: 'myapp-prod'
              deployToSlotOrASE: true
              resourceGroupName: 'myapp-rg'
              slotName: 'staging'
              package: '$(System.ArtifactsDirectory)/drop/**/*.zip'
          
          # Integration tests
          - task: DotNetCoreCLI@2
            displayName: 'Run integration tests'
            inputs:
              command: 'test'
              projects: '**/*IntegrationTests/*.csproj'

- stage: DeployProduction
  displayName: 'Deploy to Production'
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - deployment: DeployProduction
    environment: 'Production'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          # Swap staging slot to production
          - task: AzureAppServiceManage@0
            displayName: 'Swap staging to production'
            inputs:
              azureSubscription: '$(azureSubscription)'
              action: 'Swap Slots'
              webAppName: 'myapp-prod'
              resourceGroupName: 'myapp-rg'
              sourceSlot: 'staging'
              targetSlot: 'production'
          
          # Monitor for errors (5 minutes)
          - task: PowerShell@2
            displayName: 'Monitor production'
            inputs:
              targetType: 'inline'
              script: |
                Start-Sleep -Seconds 300
                # Check Application Insights for errors
                # If error rate > threshold, rollback

Deployment Strategies:

1. Blue-Green Deployment:

Two identical environments (Blue = current, Green = new)

Deploy to Green, test, then switch traffic

Instant rollback by switching back to Blue

Used at DigiTrends for zero-downtime deployments

2. Canary Deployment:

Route small % of traffic to new version

Monitor metrics, gradually increase traffic

Rollback if issues detected

Implemented at Enterprise64 for warehouse system updates

3. Rolling Deployment:

Update instances one at a time

Maintains availability during deployment

Default for Kubernetes deployments

Key CI/CD Metrics I Track:

Deployment frequency: How often we deploy to production

Lead time: Time from commit to production

Change failure rate: % of deployments causing failures

Mean time to recovery (MTTR): Time to recover from failures

Results Achieved:

Enterprise64: Reduced deployment time from 4 hours to 15 minutes

DigiTrends: Increased deployment frequency from weekly to daily

Zero-downtime deployments using slot swaps

Automated rollback on health check failures

Key Resources:

Azure DevOps: https://docs.microsoft.com/en-us/azure/devops/pipelines/

GitHub Actions: https://docs.github.com/en/actions

Deployment Strategies: https://docs.microsoft.com/en-us/azure/architecture/patterns/deployment-strategies

# 7. Docker & Kubernetes

## Q17: Explain containerization with Docker. How do you optimize Docker images?

Answer:

Docker Fundamentals:

Containers: Lightweight, standalone, executable package of software

Images: Read-only template for creating containers

Dockerfile: Script defining how to build an image

Registry: Repository for storing images (Docker Hub, ACR)

Dockerfile Example for .NET Core API:

# Multi-stage build for optimized image
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy csproj and restore dependencies (cached layer)
COPY ["MyApi/MyApi.csproj", "MyApi/"]
COPY ["MyApi.Core/MyApi.Core.csproj", "MyApi.Core/"]
RUN dotnet restore "MyApi/MyApi.csproj"

# Copy source code and build
COPY . .
WORKDIR "/src/MyApi"
RUN dotnet build "MyApi.csproj" -c Release -o /app/build

# Publish
FROM build AS publish
RUN dotnet publish "MyApi.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime image (smaller, only contains runtime)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Non-root user for security
RUN addgroup --system --gid 1000 appuser && \
    adduser --system --uid 1000 --gid 1000 appuser
USER appuser

# Copy published app
COPY --from=publish /app/publish .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Environment
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080

ENTRYPOINT ["dotnet", "MyApi.dll"]

Docker Image Optimization Techniques:

1. Multi-Stage Builds:

Separate build and runtime stages

Final image only contains runtime dependencies

Reduces image size by 60-80%

2. Layer Caching:

# BAD - entire build invalidated if any code changes
COPY . .
RUN dotnet restore
RUN dotnet build

# GOOD - restore cached unless dependencies change
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet build

3. Use .dockerignore:

# .dockerignore
**/bin/
**/obj/
**/.vs/
**/node_modules/
**/.git/
**/README.md
**/.gitignore
**/docker-compose*.yml
**/.env

4. Minimize Layers:

# BAD - creates 3 layers
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# GOOD - creates 1 layer
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    rm -rf /var/lib/apt/lists/*

5. Use Specific Base Images:

# LARGE - 200+ MB
FROM mcr.microsoft.com/dotnet/sdk:8.0

# SMALLER - 100 MB
FROM mcr.microsoft.com/dotnet/aspnet:8.0

# SMALLEST - 40 MB (Alpine Linux)
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine

Real-world Results from DigiTrends:

Before optimization: 850 MB image

After optimization: 180 MB image

Deployment time reduced from 3 minutes to 45 seconds

Using multi-stage builds + Alpine base image

Docker Compose for Local Development:

# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=db;Database=MyDb;User=sa;Password=YourPassword123;
    depends_on:
      - db
      - redis
    networks:
      - app-network
  
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourPassword123
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql
    networks:
      - app-network
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  sqldata:

Key Resources:

Docker Best Practices: https://docs.docker.com/develop/dev-best-practices/

Multi-stage Builds: https://docs.docker.com/build/building/multi-stage/

.NET Docker Images: https://hub.docker.com/_/microsoft-dotnet

## Q18: Explain Kubernetes architecture and core concepts.

Answer:

Kubernetes (K8s) Architecture:

Control Plane Components:

API Server: Front-end for Kubernetes control plane

etcd: Key-value store for cluster data

Scheduler: Assigns pods to nodes

Controller Manager: Runs controller processes

Cloud Controller Manager: Interacts with cloud provider

Node Components:

Kubelet: Agent running on each node

Container Runtime: Docker, containerd, CRI-O

Kube-proxy: Network proxy on each node

Core Kubernetes Objects:

1. Pod: Smallest deployable unit, one or more containers

# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: myregistry.azurecr.io/myapp:1.0
    ports:
    - containerPort: 8080
    env:
    - name: ASPNETCORE_ENVIRONMENT
      value: "Production"
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5

2. Deployment: Manages ReplicaSets and Pods

# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry.azurecr.io/myapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: connection-string

3. Service: Exposes Pods as network service

# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer  # ClusterIP, NodePort, LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  sessionAffinity: ClientIP  # Sticky sessions

4. ConfigMap: Configuration data

# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  appsettings.json: |
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information"
        }
      },
      "AllowedHosts": "*"
    }
  database-host: "db.example.com"
  cache-size: "100"

5. Secret: Sensitive data (passwords, tokens)

# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
data:
  connection-string: U2VydmVyPWRiO0RhdGFiYXNlPW15ZGI7VXNlcj1zYTtQYXNzd29yZD1TZWN1cmUxMjM=
  api-key: bXlzZWNyZXRhcGlrZXk=

# Use in Pod
spec:
  containers:
  - name: myapp
    env:
    - name: DB_CONNECTION
      valueFrom:
        secretKeyRef:
          name: myapp-secrets
          key: connection-string

6. Ingress: HTTP/HTTPS routing

# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80

Horizontal Pod Autoscaling:

# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

Common kubectl Commands:

# Get resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes

# Describe resource
kubectl describe pod myapp-pod

# View logs
kubectl logs myapp-pod
kubectl logs -f myapp-pod  # Follow logs

# Execute command in pod
kubectl exec -it myapp-pod -- /bin/bash

# Apply configuration
kubectl apply -f deployment.yaml
kubectl apply -f .  # All yamls in directory

# Scale deployment
kubectl scale deployment myapp-deployment --replicas=5

# Rollout management
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment

# Port forwarding (debugging)
kubectl port-forward pod/myapp-pod 8080:8080

# Delete resources
kubectl delete pod myapp-pod
kubectl delete -f deployment.yaml

Key Resources:

Kubernetes Docs: https://kubernetes.io/docs/home/

AKS: https://docs.microsoft.com/en-us/azure/aks/

Kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/