# Tech Stack Decision Guide

Use this guide to help users choose appropriate technologies.

## Language Selection

| Project Type | Recommended | Also Consider |
|--------------|-------------|---------------|
| Web API | TypeScript, Python, Go | Rust, Java |
| CLI Tool | TypeScript, Go, Rust | Python |
| Web Frontend | TypeScript | JavaScript |
| Data/ETL | Python | TypeScript |
| Library | Match ecosystem | TypeScript for npm |
| Systems | Rust, Go | C++ |

### TypeScript
- **Pros:** Type safety, npm ecosystem, full-stack capability
- **Cons:** Build step, runtime performance
- **Best for:** Web apps, APIs, CLIs, libraries

### Python
- **Pros:** Readability, ML/data ecosystem, rapid prototyping
- **Cons:** Performance, packaging complexity
- **Best for:** Data, ML, scripting, APIs

### Go
- **Pros:** Performance, simplicity, single binary
- **Cons:** Verbosity, limited generics (improving)
- **Best for:** CLIs, microservices, infrastructure

### Rust
- **Pros:** Performance, safety, no runtime
- **Cons:** Learning curve, compile times
- **Best for:** Performance-critical, systems, CLIs

## Framework Selection

### Web Backend

| Framework | Language | Best For |
|-----------|----------|----------|
| Express | TypeScript | Simple APIs, flexibility |
| Fastify | TypeScript | Performance, schema validation |
| NestJS | TypeScript | Enterprise, structure |
| FastAPI | Python | Modern APIs, auto-docs |
| Django | Python | Full-featured, admin |
| Gin | Go | Performance, simplicity |
| Actix | Rust | Maximum performance |

### Web Frontend

| Framework | Best For |
|-----------|----------|
| React | Complex UI, ecosystem |
| Vue | Simplicity, progressive |
| Svelte | Performance, simplicity |
| SolidJS | React-like, performance |

### CLI

| Tool | Language | Best For |
|------|----------|----------|
| Commander | TypeScript | Simple CLIs |
| Yargs | TypeScript | Complex CLIs |
| Click | Python | Python CLIs |
| Cobra | Go | Go CLIs |
| Clap | Rust | Rust CLIs |

## Database Selection

| Type | Options | Best For |
|------|---------|----------|
| Relational | PostgreSQL, MySQL | Structured data, transactions |
| Document | MongoDB | Flexible schema, JSON |
| Key-Value | Redis | Caching, sessions |
| SQLite | SQLite | Embedded, local, simple |
| Graph | Neo4j | Relationships |

### PostgreSQL
- **Pros:** Feature-rich, reliable, JSON support
- **Best for:** Most applications

### SQLite
- **Pros:** Zero config, embedded, portable
- **Best for:** Local apps, prototypes, edge

### MongoDB
- **Pros:** Flexible schema, scaling
- **Best for:** Document-centric, rapid iteration

## Hosting Selection

| Option | Best For |
|--------|----------|
| Vercel | Frontend, serverless |
| Railway | Full-stack, easy deploy |
| Fly.io | Containers, edge |
| AWS | Enterprise, full control |
| GCP | Data, ML |
| Azure | Enterprise, .NET |

## Common Combinations

### Modern Web API (TypeScript)
- Language: TypeScript
- Runtime: Node.js
- Framework: Express or Fastify
- Database: PostgreSQL
- ORM: Drizzle or Prisma
- Validation: Zod
- Testing: Vitest

### Full-Stack Web (TypeScript)
- Backend: Express + TypeScript
- Frontend: React + TypeScript
- Database: PostgreSQL
- Styling: TailwindCSS
- State: TanStack Query
- Testing: Vitest + Testing Library

### Python API
- Language: Python
- Framework: FastAPI
- Database: PostgreSQL
- ORM: SQLAlchemy
- Testing: pytest

### CLI Tool
- Language: TypeScript or Go
- Framework: Commander or Cobra
- Distribution: npm or single binary
- Testing: Vitest or Go test

### Data Pipeline
- Language: Python
- Framework: Prefect or Dagster
- Storage: PostgreSQL + S3
- Testing: pytest
