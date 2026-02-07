# Azure Container Apps Provider Guide

Container hosting for .NET Aspire and polyglot applications.

## When to Use

- .NET Aspire projects (default)
- Microservices architectures
- Polyglot projects (Node.js, Python with .NET orchestration)
- Azure-first deployments

## Prerequisites

```bash
# 1. .NET 10 SDK (required for Aspire 13)
winget install Microsoft.DotNet.SDK.10
dotnet --version  # Should be 10.0.100+

# 2. Azure Developer CLI
winget install Microsoft.Azd
azd version  # Should be 1.11.0+

# 3. Aspire CLI
dotnet tool install -g aspire.cli
aspire --version  # Should be 13.1.0+

# 4. Docker Desktop
docker --version

# 5. Azure CLI
winget install Microsoft.AzureCLI
az --version

# 6. Login to Azure
azd auth login
azd auth login --check-status
```

## Critical: Buildpacks Don't Work with containerd

Docker Desktop's containerd image store (default since 2024) is **incompatible** with Cloud Native Buildpacks.

```csharp
// DON'T USE (fails with containerd):
builder.AddNpmApp("backend", "../../backend")

// USE THIS (explicit Dockerfile):
builder.AddDockerfile("backend", "../..", "backend/Dockerfile")
```

## Critical: azd Requires Bicep Templates

Even with Aspire, create `infra/` directory:
- `infra/main.bicep` - Infrastructure definition
- `infra/containerApps.bicep` - Container Apps module
- `infra/main.parameters.json` - Parameters

## azure.yaml Configuration

```yaml
name: projectname
metadata:
  template: aspire-starter@13.0.0

infra:
  provider: bicep
  path: ./infra
  module: main

host:
  project: ./aspire/Project.AppHost/Project.AppHost.csproj

services:
  backend:
    language: js  # Required even with Dockerfile
    host: containerapp
    docker:
      path: ./backend/Dockerfile
      context: .  # CRITICAL: monorepo root

  frontend:
    language: js
    host: containerapp
    docker:
      path: ./frontend/Dockerfile
      context: .  # CRITICAL: monorepo root
```

## Deployment Steps

```bash
# 1. Navigate to project
cd "project-name"

# 2. First-time setup
azd auth login
azd env new dev
azd env set AZURE_SUBSCRIPTION_ID "your-subscription-id"
azd env set AZURE_LOCATION "eastus2"

# 3. Deploy (provision + deploy)
azd up --no-prompt

# Or step by step:
azd package    # Build Docker images
azd provision  # Create Azure resources
azd deploy     # Deploy containers

# 4. Verify
azd env get-values
curl https://backend.<env>.azurecontainerapps.io/api/health

# 5. Subsequent deploys
azd deploy

# 6. Cleanup
azd down --force --purge
```

## Local Development

```bash
# Run with Aspire Dashboard
cd aspire/Project.AppHost
dotnet run
# Visit https://localhost:15888

# Or with Aspire CLI
aspire run
```

## Monorepo Docker Builds

For npm workspaces, Docker context must be repo root:

```dockerfile
# backend/Dockerfile
FROM node:20-alpine AS deps
WORKDIR /app

# Copy root workspace files
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
COPY shared/package.json ./shared/

# Install workspace dependencies
RUN npm ci --workspace=backend --workspace=@project/shared
```

## AppHost Configuration

```csharp
var builder = DistributedApplication.CreateBuilder(args);

// Use explicit Dockerfile with monorepo root context
var backend = builder.AddDockerfile("backend", "../..", "backend/Dockerfile")
    .WithHttpEndpoint(port: 3000, targetPort: 3000, name: "api")
    .WithExternalHttpEndpoints()
    .WithEnvironment("NODE_ENV", "production");

var frontend = builder.AddDockerfile("frontend", "../..", "frontend/Dockerfile")
    .WithHttpEndpoint(port: 5173, targetPort: 80, name: "web")
    .WithExternalHttpEndpoints()
    .WaitFor(backend);

builder.Build().Run();
```

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `buildpack requires containerd disabled` | containerd + buildpacks | Use `AddDockerfile()` |
| `npm ci requires package-lock.json` | Wrong Docker context | Set `context: .` in azure.yaml |
| `Must specify language or image` | Missing language | Add `language: js` |
| `PrincipalId has type 'User'` | Role assignment error | Remove `principalType` from Bicep |
| `Could not find infra/main.bicep` | Missing templates | Create infra/ directory |

## Verification

```bash
# View logs
az containerapp logs show -n backend -g rg-<env> --type console

# Monitor
azd monitor

# Check endpoints
curl https://backend.<env>.azurecontainerapps.io/health
```

## Update manifest.json

```json
{
  "hosting": {
    "provider": "azure-container-apps",
    "resourceGroup": "rg-projectname-prod",
    "containerRegistry": "acrXXXXXX.azurecr.io",
    "services": {
      "backend": "https://backend.XXX.azurecontainerapps.io",
      "frontend": "https://frontend.XXX.azurecontainerapps.io"
    },
    "aspireVersion": "13.x"
  }
}
```
