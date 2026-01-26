# Backend API Stack

REST, GraphQL, or gRPC services without user-facing frontend. Use this for APIs that power other applications or services.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

### Node.js / Express

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Node.js | 20.x LTS | Minor |
| Express | 4.x | Minor |
| TypeScript | 5.3 | Minor |
| Prisma | 5.x | Major if 6+ |

### Python / FastAPI

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Python | 3.11 | Minor |
| FastAPI | 0.109 | Minor |
| SQLAlchemy | 2.x | Minor |
| Pydantic | 2.x | Minor |

### Go

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Go | 1.21 | Minor |
| Gin | 1.9 | Minor |
| GORM | 1.25 | Minor |

**Recommendation**: For maximum code reliability, use AI-confident versions unless latest features are required.

## Default Stack by Language

### Node.js / TypeScript (Default)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Runtime** | Node.js 20 LTS | JavaScript runtime |
| **Language** | TypeScript (strict mode) | Type safety |
| **Framework** | Express.js | Minimal, flexible API framework |
| **Validation** | Zod | Runtime validation + TypeScript |
| **Database** | PostgreSQL | Relational database |
| **ORM** | Prisma | Type-safe database queries |
| **Auth** | JWT / Passport | Token-based authentication |
| **Docs** | OpenAPI/Swagger | API documentation |
| **Deployment** | Railway / Render | Simple Node.js hosting |
| **Source Control** | GitHub | Repository hosting |

### Python / FastAPI

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Runtime** | Python 3.11+ | Python runtime |
| **Framework** | FastAPI | Modern async API framework |
| **Validation** | Pydantic | Data validation and settings |
| **Database** | PostgreSQL | Relational database |
| **ORM** | SQLAlchemy | Database toolkit |
| **Auth** | JWT / OAuth2 | Token-based authentication |
| **Docs** | OpenAPI (built-in) | Auto-generated API docs |
| **Deployment** | Railway / AWS Lambda | Python hosting |
| **Source Control** | GitHub | Repository hosting |

### Go

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Go 1.21+ | Compiled language |
| **Framework** | Gin | High-performance HTTP framework |
| **Validation** | go-playground/validator | Struct validation |
| **Database** | PostgreSQL | Relational database |
| **ORM** | GORM | Go ORM |
| **Auth** | JWT | Token-based authentication |
| **Docs** | Swaggo | Swagger docs for Go |
| **Deployment** | Railway / AWS ECS | Container hosting |
| **Source Control** | GitHub | Repository hosting |

## Testing Stack

### Node.js
| Tool | Purpose |
|------|---------|
| Jest | Unit testing |
| Supertest | API endpoint testing |

### Python
| Tool | Purpose |
|------|---------|
| pytest | Unit testing |
| httpx | Async API testing |

### Go
| Tool | Purpose |
|------|---------|
| testing (stdlib) | Unit testing |
| httptest (stdlib) | HTTP handler testing |

## Logging Stack

| Language | Logger | Error Tracking |
|----------|--------|----------------|
| Node.js | Pino | Sentry |
| Python | structlog | Sentry |
| Go | zerolog | Sentry |

## When to Deviate

| Requirement | Alternative |
|-------------|-------------|
| GraphQL needed | Apollo Server (Node), Strawberry (Python), gqlgen (Go) |
| gRPC needed | @grpc/grpc-js (Node), grpcio (Python), grpc-go (Go) |
| Real-time needed | Socket.IO (Node), WebSockets |
| High performance | Go or Rust |
| ML/AI integration | Python FastAPI |
| Enterprise Java shop | Spring Boot |

## Database Options

| Service | Best For | Notes |
|---------|----------|-------|
| Supabase | Quick setup | PostgreSQL with dashboard |
| AWS RDS | Enterprise | Managed PostgreSQL/MySQL |
| PlanetScale | Serverless MySQL | Auto-scaling |
| MongoDB Atlas | Document data | Flexible schema |
| Redis | Caching, queues | In-memory storage |

## Development Database Strategy

> **Cost Optimization**: Using separate databases per project during development can significantly increase costs. Consider schema-based isolation instead.

### Shared Database with Separate Schemas (Recommended for Development)

Instead of creating a new database instance for each project, use a single development database with separate schemas:

| Approach | Cost | Isolation | Best For |
|----------|------|-----------|----------|
| **Shared DB + schemas** | Low | Schema-level | Development, staging |
| Separate databases | High | Full | Production, compliance |

**Benefits:**
- Significant cost savings (one database instance vs. many)
- Same isolation patterns you'll use in production
- Easier to manage across multiple projects
- Works with Supabase, Neon, PlanetScale, and most PostgreSQL providers

**Implementation (PostgreSQL):**

```sql
-- Create a schema for each project
CREATE SCHEMA project_api_v1;
CREATE SCHEMA project_api_v2;

-- Set search_path in connection string
-- postgresql://user:pass@host:5432/db?schema=project_api_v1
```

**SQLAlchemy Configuration (Python):**
```python
# In your database URL or engine config
engine = create_engine(
    "postgresql://user:pass@host:5432/db",
    connect_args={"options": "-csearch_path=project_api_v1"}
)
```

**Prisma Configuration (Node.js):**
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL") // includes ?schema=project_name
}
```

**GORM Configuration (Go):**
```go
// Use search_path in connection string
dsn := "host=localhost user=user password=pass dbname=db port=5432 search_path=project_api_v1"
```

**Tradeoffs:**
- Requires discipline around migrations (per-schema)
- Shared resource limits (connections, storage)
- Not suitable for production workloads with strict isolation needs

### When to Use Separate Databases

| Scenario | Recommendation |
|----------|---------------|
| Development/prototyping | Shared + schemas |
| Staging (team shared) | Shared + schemas |
| Production | Separate database |
| Compliance requirements | Separate database |
| Performance isolation needed | Separate database |

### Provider-Specific Notes

| Provider | Schema Support | Notes |
|----------|---------------|-------|
| Supabase | Full | Use SQL editor to create schemas |
| Neon | Full | Branch per project is also cost-effective |
| PlanetScale | Limited | Use database branching instead |
| AWS RDS | Full | Standard PostgreSQL schemas |
| MongoDB Atlas | N/A | Use separate databases or collections |

## Hosting Options

| Service | Best For | Notes |
|---------|----------|-------|
| Railway | Default choice | Simple deploys, DB included |
| Render | Simple APIs | Good free tier |
| AWS Lambda | Serverless | Pay-per-invocation |
| AWS ECS/Fargate | Containers | Full control |
| DigitalOcean | Budget-friendly | Simple VPS or App Platform |
| Fly.io | Edge deployment | Global distribution |

## For Non-Technical Users

When the user is non-technical, don't ask about stack choices. Simply state:

> "I'll use our standard API stack with Node.js and TypeScript - it's reliable and easy to maintain. The specific technologies are documented in the project for any developers who work on it."

## For Technical Users

Ask about preferences:

> "For backend APIs, I typically recommend Node.js/Express, Python/FastAPI, or Go depending on your needs. Any preference?"

Then dive into:
- REST vs GraphQL vs gRPC?
- Database preference?
- Deployment target?
