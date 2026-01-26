# .NET Aspire Stack

Distributed applications using .NET Aspire for orchestration and Azure for deployment. Use this for microservices, cloud-native .NET applications, and projects targeting Azure Container Apps.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| .NET | 8.x LTS | **Major if 10+** |
| C# | 12 | **Major if 14+** |
| .NET Aspire | 8.x | **Major if 13+** |
| Entity Framework Core | 8.x | Moderate if 10+ |
| ASP.NET Core | 8.x | **Major if 10+** |
| Blazor | 8.x | Moderate if 10+ |

### .NET 10 / Aspire 13 Critical Differences

**Aspire CLI (New in 13):**
- `aspire deploy` - Deploys to configured targets
- `aspire publish` - Generates deployment artifacts
- `aspire run` - Runs the application locally
- `aspire new` - Creates new Aspire projects
- Requires .NET SDK 10.0.100+

**Deployment State Management (New in 13):**
- Deployment info persists locally across runs
- Each environment (dev, staging, prod) isolated
- No more repetitive prompts on redeploy

**Breaking Changes from Aspire 8.x:**
- `dotnet run --publisher manifest` is deprecated
- Use `aspire publish` instead for manifest generation
- `DeployingCallbackAnnotation` required for custom deployment
- No built-in deployment annotations by default

**Recommendation**: Reference the gotchas in `tech/stack.md` for all .NET 10 / Aspire 13 code. Verify patterns against current documentation.

## Core Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Runtime** | .NET 10 LTS | Latest LTS runtime |
| **Language** | C# 14 | Latest language features |
| **Orchestration** | .NET Aspire 13 | Distributed app orchestration |
| **Web Framework** | ASP.NET Core | Web APIs and UI |
| **UI (optional)** | Blazor / React | Interactive UI |
| **ORM** | Entity Framework Core | Database access |
| **Database** | Azure SQL / PostgreSQL | Relational database |
| **Caching** | Redis | Distributed caching |
| **Messaging** | Azure Service Bus | Message queuing |
| **Storage** | Azure Blob Storage | File storage |
| **Deployment** | Azure Container Apps | Container hosting |
| **CI/CD** | GitHub Actions / Azure DevOps | Automation |
| **Source Control** | GitHub | Repository hosting |

## Project Structure

```
{solution}/
├── {Solution}.sln
├── src/
│   ├── {Solution}.AppHost/           # Aspire orchestration
│   │   ├── Program.cs
│   │   └── {Solution}.AppHost.csproj
│   ├── {Solution}.ServiceDefaults/   # Shared service config
│   │   ├── Extensions.cs
│   │   └── {Solution}.ServiceDefaults.csproj
│   ├── {Solution}.Api/               # Web API project
│   │   ├── Program.cs
│   │   ├── Controllers/
│   │   └── {Solution}.Api.csproj
│   ├── {Solution}.Web/               # Blazor/frontend project
│   │   ├── Program.cs
│   │   ├── Components/
│   │   └── {Solution}.Web.csproj
│   └── {Solution}.Data/              # Data access layer
│       ├── DbContext.cs
│       ├── Entities/
│       └── {Solution}.Data.csproj
├── tests/
│   ├── {Solution}.Api.Tests/
│   └── {Solution}.Integration.Tests/
├── .azure/                           # Azure deployment config
├── azure.yaml                        # azd configuration
└── README.md
```

## AppHost Configuration (Aspire 13)

```csharp
// src/{Solution}.AppHost/Program.cs
var builder = DistributedApplication.CreateBuilder(args);

// Add infrastructure
var redis = builder.AddRedis("cache");
var sqlServer = builder.AddAzureSqlServer("sql")
    .AddDatabase("appdb");

// Add services
var api = builder.AddProject<Projects.Api>("api")
    .WithReference(redis)
    .WithReference(sqlServer);

var web = builder.AddProject<Projects.Web>("web")
    .WithReference(api)
    .WithExternalHttpEndpoints();

// Azure-specific configuration
if (builder.ExecutionContext.IsPublishMode)
{
    api.WithDeploymentSlot("staging");
}

builder.Build().Run();
```

## Service Defaults (Aspire 13)

```csharp
// src/{Solution}.ServiceDefaults/Extensions.cs
public static class Extensions
{
    public static IHostApplicationBuilder AddServiceDefaults(
        this IHostApplicationBuilder builder)
    {
        builder.ConfigureOpenTelemetry();
        builder.AddDefaultHealthChecks();
        builder.Services.AddServiceDiscovery();

        builder.Services.ConfigureHttpClientDefaults(http =>
        {
            http.AddStandardResilienceHandler();
            http.AddServiceDiscovery();
        });

        return builder;
    }
}
```

## Testing Stack

| Tool | Purpose |
|------|---------|
| xUnit | Unit testing framework |
| NSubstitute / Moq | Mocking |
| FluentAssertions | Assertion library |
| Aspire.Hosting.Testing | Integration testing |
| Playwright | E2E testing |

## Logging Stack

| Tool | Purpose |
|------|---------|
| Serilog | Structured logging |
| OpenTelemetry | Distributed tracing |
| Application Insights | Azure monitoring |
| Aspire Dashboard | Local development monitoring |

## Deployment Commands

### Azure Developer CLI (Recommended)

```bash
# Initialize (first time)
azd init

# Deploy everything
azd up

# Or step by step:
azd provision    # Create Azure resources
azd deploy       # Deploy application

# Manage environments
azd env new staging
azd env select staging
azd up
```

### Aspire CLI

```bash
# Install Aspire CLI
dotnet tool install -g aspire

# Run locally
aspire run

# Publish deployment artifacts
aspire publish --output ./deploy

# Deploy (requires deployment annotations)
aspire deploy --environment production
```

## Environment Variables

```bash
# Connection strings (managed by Aspire)
ConnectionStrings__cache=redis-connection-string
ConnectionStrings__appdb=sql-connection-string

# Azure-specific
AZURE_SUBSCRIPTION_ID=xxx
AZURE_RESOURCE_GROUP=rg-appname-prod
APPLICATIONINSIGHTS_CONNECTION_STRING=xxx

# App settings
ASPNETCORE_ENVIRONMENT=Production
```

## Polyglot Projects (Node.js, Python, etc.)

Aspire 13 supports polyglot orchestration using `AddDockerfile()`:

**CRITICAL: Buildpacks don't work with Docker Desktop containerd**

Docker Desktop's containerd image store (default since 2024) is incompatible with buildpacks:

```csharp
// DON'T USE - fails with containerd:
builder.AddNpmApp("backend", "../../backend")

// USE THIS - explicit Dockerfile:
builder.AddDockerfile("backend", "../..", "backend/Dockerfile")
```

**Monorepo Docker Context:**

For npm workspaces monorepos, Docker context must be the repository root:

```csharp
// Context is "../.." (monorepo root), Dockerfile is "backend/Dockerfile"
var backend = builder.AddDockerfile("backend", "../..", "backend/Dockerfile")
    .WithHttpEndpoint(port: 3000, targetPort: 3000, name: "api")
    .WithExternalHttpEndpoints();
```

**azure.yaml for polyglot services:**
```yaml
services:
  backend:
    language: js  # Required even with custom Dockerfile
    host: containerapp
    docker:
      path: ./backend/Dockerfile
      context: .  # CRITICAL: monorepo root
```

## When to Use This Stack

| Requirement | Recommendation |
|-------------|---------------|
| Microservices architecture | ✓ Aspire excels here |
| Azure-first deployment | ✓ First-class Azure support |
| Mixed .NET + Node/Python services | ✓ Polyglot orchestration in Aspire 13 |
| Local development experience | ✓ Aspire Dashboard |
| Team familiar with .NET | ✓ Natural fit |
| npm workspaces monorepo | ✓ Use AddDockerfile with root context |
| AWS/GCP deployment | Consider alternatives |
| Simple single-service app | May be overkill |

## When to Deviate

| Requirement | Alternative |
|-------------|-------------|
| AWS deployment | Use AWS CDK or Pulumi |
| GCP deployment | Use Cloud Run directly |
| Non-.NET primary language | Use Docker Compose or Kubernetes |
| Simple API without orchestration | Plain ASP.NET Core |
| Serverless-first | Azure Functions |

## For Non-Technical Users

When the user is non-technical, don't ask about stack choices. Simply state:

> "I'll build this using .NET and Azure - it's Microsoft's enterprise-grade platform for building reliable cloud applications. Everything will be configured to deploy automatically to Azure."

## For Technical Users

Ask about preferences:

> "For .NET distributed apps, I recommend .NET 10 with Aspire 13 for Azure Container Apps. This gives you local orchestration, built-in observability, and streamlined Azure deployment. Any preferences?"

Then dive into:
- Service architecture? (monolith vs microservices)
- UI framework? (Blazor, React, none)
- Database? (Azure SQL, PostgreSQL, Cosmos DB)
- Additional Azure services? (Service Bus, Storage, etc.)

## References

- [Aspire 13 What's New](https://aspire.dev/whats-new/aspire-13/)
- [Azure Container Apps Deployment](https://learn.microsoft.com/en-us/dotnet/aspire/deployment/azd/aca-deployment)
- [Aspire CLI Reference](https://learn.microsoft.com/en-us/dotnet/aspire/cli/overview)
- [.NET 10 Release Notes](https://github.com/dotnet/core/discussions/10157)
