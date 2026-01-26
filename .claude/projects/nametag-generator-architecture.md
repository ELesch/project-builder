# Architecture: BigBadge - AI-Powered Name Tag Generator

## Project Details

- **Project Name**: BigBadge
- **Repository**: big-badge (GitHub)
- **Deployment**: Vercel
- **Database Hosting**: Supabase (PostgreSQL)

## Overview

A Next.js 14+ full-stack web application using the App Router pattern, with a Supabase PostgreSQL database and Google Gemini API integration for AI-generated name tag images. The architecture emphasizes modularity, with clear separation between the template designer (admin), check-in interface (public), and AI/printing integrations.

**Key Architectural Decisions:**
- **Monorepo with Next.js**: Single codebase for both admin and check-in interfaces
- **Server Components + API Routes**: Leverage Next.js server capabilities for database access
- **Abstracted AI Layer**: Provider-agnostic interface for future flexibility
- **Canvas-based Template Designer**: Using Fabric.js for rich template editing
- **WebSocket for Real-time Sync**: Multi-device check-in synchronization

## Directory Structure

```
big-badge/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── (admin)/                  # Admin route group (protected)
│   │   │   ├── dashboard/            # Event management dashboard
│   │   │   ├── events/
│   │   │   │   ├── [eventId]/
│   │   │   │   │   ├── template/     # Template designer
│   │   │   │   │   ├── attendees/    # Attendee management
│   │   │   │   │   └── settings/     # Event settings
│   │   │   │   └── new/              # Create new event
│   │   │   └── layout.tsx            # Admin layout with auth
│   │   ├── (checkin)/                # Public check-in route group
│   │   │   ├── [eventSlug]/          # Event-specific check-in
│   │   │   │   ├── page.tsx          # Search/select attendee
│   │   │   │   ├── register/         # Walk-in registration
│   │   │   │   └── confirm/[id]/     # Confirm & print
│   │   │   └── layout.tsx            # Check-in layout (no auth)
│   │   ├── api/                       # API Routes
│   │   │   ├── auth/                  # Authentication endpoints
│   │   │   ├── events/                # Event CRUD
│   │   │   ├── attendees/             # Attendee management
│   │   │   ├── templates/             # Template operations
│   │   │   ├── generate/              # AI image generation
│   │   │   ├── import/                # Excel import
│   │   │   ├── print/                 # Print job handling
│   │   │   └── ws/                    # WebSocket endpoint
│   │   ├── layout.tsx                 # Root layout
│   │   └── page.tsx                   # Landing/redirect
│   │
│   ├── components/                    # React components
│   │   ├── ui/                        # Base UI components (shadcn/ui)
│   │   ├── admin/                     # Admin-specific components
│   │   │   ├── EventCard.tsx
│   │   │   ├── AttendeeTable.tsx
│   │   │   └── ImportWizard/
│   │   ├── checkin/                   # Check-in specific components
│   │   │   ├── AttendeeSearch.tsx
│   │   │   ├── NameTagPreview.tsx
│   │   │   └── PrintButton.tsx
│   │   └── template-designer/         # Canvas-based editor
│   │       ├── Canvas.tsx             # Fabric.js wrapper
│   │       ├── Toolbar.tsx            # Design tools
│   │       ├── PropertyPanel.tsx      # Element properties
│   │       ├── LayerPanel.tsx         # Layer management
│   │       └── elements/              # Draggable elements
│   │           ├── TextElement.tsx
│   │           ├── ImageElement.tsx
│   │           ├── PlaceholderElement.tsx
│   │           └── AIZoneElement.tsx  # AI generation area
│   │
│   ├── lib/                           # Core library code
│   │   ├── db/                        # Database layer
│   │   │   ├── client.ts              # Prisma client
│   │   │   ├── schema.prisma          # Database schema
│   │   │   └── migrations/            # Prisma migrations
│   │   ├── ai/                        # AI abstraction layer
│   │   │   ├── types.ts               # Provider-agnostic types
│   │   │   ├── provider.ts            # AI provider interface
│   │   │   ├── gemini.ts              # Google Gemini implementation
│   │   │   └── fallback.ts            # Plain-text fallback generator
│   │   ├── printer/                   # Printer integration
│   │   │   ├── types.ts               # Printer interfaces
│   │   │   ├── detection.ts           # Printer discovery
│   │   │   ├── dymo.ts                # Dymo SDK integration
│   │   │   ├── brother.ts             # Brother SDK integration
│   │   │   ├── zebra.ts               # Zebra SDK integration
│   │   │   └── browser-print.ts       # Browser print fallback
│   │   ├── import/                    # Data import
│   │   │   ├── excel-parser.ts        # XLSX parsing
│   │   │   ├── field-mapper.ts        # Field mapping logic
│   │   │   └── validators.ts          # Data validation
│   │   ├── template/                  # Template processing
│   │   │   ├── renderer.ts            # Template to image
│   │   │   ├── serializer.ts          # Canvas state save/load
│   │   │   └── placeholders.ts        # ${field} substitution
│   │   ├── auth/                      # Authentication
│   │   │   ├── config.ts              # Auth configuration
│   │   │   └── session.ts             # Session management
│   │   └── utils/                     # Shared utilities
│   │       ├── api-response.ts        # Consistent API responses
│   │       ├── errors.ts              # Error types
│   │       └── validation.ts          # Zod schemas
│   │
│   ├── hooks/                         # React hooks
│   │   ├── useEvent.ts                # Event data fetching
│   │   ├── useAttendees.ts            # Attendee operations
│   │   ├── useTemplate.ts             # Template state
│   │   ├── usePrinter.ts              # Printer connection
│   │   ├── useWebSocket.ts            # Real-time sync
│   │   └── useAIGeneration.ts         # AI image generation
│   │
│   └── types/                         # TypeScript types
│       ├── event.ts
│       ├── attendee.ts
│       ├── template.ts
│       └── api.ts
│
├── prisma/
│   ├── schema.prisma                  # Database schema (linked)
│   ├── seed.ts                        # Development seed data
│   └── migrations/                    # Migration history
│
├── public/
│   └── assets/                        # Static assets
│
├── tests/
│   ├── unit/                          # Unit tests
│   ├── integration/                   # Integration tests
│   └── e2e/                           # Playwright E2E tests
│
├── docs/
│   ├── DECISIONS/                     # Architecture Decision Records
│   ├── DESIGNS/                       # Design documents
│   └── API.md                         # API documentation
│
├── .claude/                           # Claude Code orchestrator
│   ├── CLAUDE.md                      # Project instructions
│   ├── roster.md                      # Agent selection guide
│   ├── agents/                        # Specialized agents
│   ├── templates/                     # Planning templates
│   ├── plans/                         # Active plans
│   └── results/                       # Completed results
│
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── .env.example
```

## Database Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================
// AUTHENTICATION
// ============================================

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  passwordHash  String
  name          String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  events        Event[]   @relation("EventOrganizer")

  @@map("users")
}

// ============================================
// EVENTS
// ============================================

model Event {
  id            String    @id @default(cuid())
  name          String
  slug          String    @unique  // URL-friendly identifier for check-in
  description   String?
  startDate     DateTime?
  endDate       DateTime?
  status        EventStatus @default(DRAFT)

  organizerId   String
  organizer     User      @relation("EventOrganizer", fields: [organizerId], references: [id])

  template      Template?
  attendees     Attendee[]
  fieldMappings FieldMapping[]
  printerConfig PrinterConfig?

  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@index([slug])
  @@index([organizerId])
  @@map("events")
}

enum EventStatus {
  DRAFT
  ACTIVE
  COMPLETED
  ARCHIVED
}

// ============================================
// TEMPLATES
// ============================================

model Template {
  id            String    @id @default(cuid())
  eventId       String    @unique
  event         Event     @relation(fields: [eventId], references: [id], onDelete: Cascade)

  // Label dimensions
  widthInches   Float     @default(4.0)
  heightInches  Float     @default(3.0)

  // Serialized canvas state (Fabric.js JSON)
  canvasData    Json

  // AI generation zone (coordinates within canvas)
  aiZone        Json?     // { x, y, width, height }

  // Prompt template for AI generation
  aiPromptTemplate String? @default("Create a stylized name tag image with the name '{firstName}' prominently displayed, '{lastName}' smaller below, and '{company}' as subtle text. Use the full image area efficiently.")

  // Fallback template (plain text layout)
  fallbackEnabled Boolean @default(true)

  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@map("templates")
}

// ============================================
// ATTENDEES
// ============================================

model Attendee {
  id            String    @id @default(cuid())
  eventId       String
  event         Event     @relation(fields: [eventId], references: [id], onDelete: Cascade)

  // Core fields (always present)
  firstName     String
  lastName      String
  email         String?
  company       String?

  // Dynamic fields from Excel import (JSON blob)
  customFields  Json      @default("{}")

  // Check-in tracking
  checkedIn     Boolean   @default(false)
  checkedInAt   DateTime?
  checkedInBy   String?   // Device identifier

  // Generated badge info
  badgePrinted  Boolean   @default(false)
  badgeImageUrl String?   // URL to generated AI image
  printCount    Int       @default(0)

  // Source tracking
  source        AttendeeSource @default(MANUAL)
  importBatchId String?

  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@unique([eventId, email])
  @@index([eventId])
  @@index([eventId, lastName, firstName])
  @@map("attendees")
}

enum AttendeeSource {
  MANUAL      // Added manually
  IMPORT      // From Excel import
  WALKIN      // Walk-in registration
}

// ============================================
// FIELD MAPPING (Excel Import)
// ============================================

model FieldMapping {
  id            String    @id @default(cuid())
  eventId       String
  event         Event     @relation(fields: [eventId], references: [id], onDelete: Cascade)

  // Excel column header
  sourceColumn  String

  // Target field (system field or custom)
  targetField   String    // 'firstName', 'lastName', 'email', 'company', or custom field name
  isSystemField Boolean   @default(false)

  // Transformation (optional)
  transform     String?   // 'uppercase', 'lowercase', 'titlecase', etc.

  @@unique([eventId, sourceColumn])
  @@map("field_mappings")
}

// ============================================
// PRINTER CONFIGURATION
// ============================================

model PrinterConfig {
  id            String    @id @default(cuid())
  eventId       String    @unique
  event         Event     @relation(fields: [eventId], references: [id], onDelete: Cascade)

  // Printer identification
  printerType   PrinterType
  printerName   String?   // OS printer name or SDK identifier
  connectionType ConnectionType @default(USB)

  // Print settings
  copies        Int       @default(1)
  quality       PrintQuality @default(STANDARD)

  // Label settings (override template if different)
  labelWidthInches  Float?
  labelHeightInches Float?

  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@map("printer_configs")
}

enum PrinterType {
  DYMO
  BROTHER
  ZEBRA
  BROWSER    // Use browser print dialog
}

enum ConnectionType {
  USB
  NETWORK
  BLUETOOTH
}

enum PrintQuality {
  DRAFT
  STANDARD
  HIGH
}

// ============================================
// AUDIT / LOGGING
// ============================================

model PrintJob {
  id            String    @id @default(cuid())
  eventId       String
  attendeeId    String

  status        PrintJobStatus @default(PENDING)
  printerType   PrinterType

  // Image data
  imageUrl      String?
  usedFallback  Boolean   @default(false)

  // Timing
  queuedAt      DateTime  @default(now())
  startedAt     DateTime?
  completedAt   DateTime?

  // Error tracking
  errorMessage  String?
  retryCount    Int       @default(0)

  @@index([eventId])
  @@index([attendeeId])
  @@map("print_jobs")
}

enum PrintJobStatus {
  PENDING
  GENERATING   // AI generation in progress
  PRINTING
  COMPLETED
  FAILED
}

model AIGenerationLog {
  id            String    @id @default(cuid())
  attendeeId    String

  provider      String    // 'gemini', 'fallback'
  model         String?   // 'gemini-2.0-flash-exp' etc.

  prompt        String
  aspectRatio   String    // '4:3', '3:4', etc.

  success       Boolean
  durationMs    Int?
  errorMessage  String?

  createdAt     DateTime  @default(now())

  @@index([attendeeId])
  @@map("ai_generation_logs")
}
```

## Component Architecture

### Template Designer (Canvas-Based Editor)

```
┌─────────────────────────────────────────────────────────────┐
│  Template Designer                                           │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌────────────────────────┐  ┌─────────────┐ │
│  │ Toolbar  │  │      Canvas            │  │  Property   │ │
│  │          │  │  (Fabric.js)           │  │   Panel     │ │
│  │ - Text   │  │                        │  │             │ │
│  │ - Image  │  │  ┌─────────────────┐   │  │ - Position  │ │
│  │ - Shape  │  │  │   AI Zone       │   │  │ - Size      │ │
│  │ - AI Zone│  │  │   (dashed)      │   │  │ - Font      │ │
│  │ - ${var} │  │  └─────────────────┘   │  │ - Color     │ │
│  │          │  │                        │  │ - Rotation  │ │
│  │          │  │  [Logo]  ${firstName}  │  │             │ │
│  │          │  │          ${company}    │  │             │ │
│  └──────────┘  └────────────────────────┘  └─────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer Panel: [AI Zone] [Logo] [FirstName] [Company]   │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**Key Components:**
- `Canvas.tsx`: Fabric.js wrapper with custom object types
- `AIZoneElement`: Special element marking AI generation area
- `PlaceholderElement`: Text with `${fieldName}` syntax
- `PropertyPanel`: Context-sensitive property editor

### Check-in Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Search    │ --> │   Select    │ --> │  Generate   │ --> │   Print     │
│  Attendee   │     │  Attendee   │     │   Badge     │     │   Badge     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
       │                                       │
       v                                       v
┌─────────────┐                        ┌─────────────┐
│  Walk-in    │                        │  Fallback   │
│ Registration│                        │ (Plain Text)│
└─────────────┘                        └─────────────┘
```

### AI Integration Layer

```typescript
// lib/ai/provider.ts - Provider-agnostic interface

export interface AIImageProvider {
  name: string;
  isAvailable(): Promise<boolean>;
  generateNameTagImage(params: GenerationParams): Promise<GenerationResult>;
}

export interface GenerationParams {
  firstName: string;
  lastName: string;
  company?: string;
  customFields?: Record<string, string>;
  promptTemplate: string;
  aspectRatio: AspectRatio;
  style?: GenerationStyle;
}

export interface GenerationResult {
  success: boolean;
  imageData?: string;  // Base64 encoded
  mimeType?: string;
  error?: string;
  durationMs: number;
  provider: string;
}

// Supported aspect ratios (Gemini constraints)
export type AspectRatio = '1:1' | '4:3' | '3:4' | '16:9' | '9:16' | '3:2' | '2:3';
```

## API Routes

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| **Authentication** |
| POST | `/api/auth/login` | Organizer login | Public |
| POST | `/api/auth/logout` | Logout | Protected |
| GET | `/api/auth/session` | Get current session | Protected |
| **Events** |
| GET | `/api/events` | List organizer's events | Protected |
| POST | `/api/events` | Create new event | Protected |
| GET | `/api/events/[id]` | Get event details | Protected |
| PATCH | `/api/events/[id]` | Update event | Protected |
| DELETE | `/api/events/[id]` | Delete event | Protected |
| GET | `/api/events/slug/[slug]` | Get event by slug (check-in) | Public |
| **Attendees** |
| GET | `/api/events/[id]/attendees` | List attendees (paginated) | Protected |
| POST | `/api/events/[id]/attendees` | Add attendee | Protected |
| GET | `/api/events/[id]/attendees/[attendeeId]` | Get attendee | Mixed |
| PATCH | `/api/events/[id]/attendees/[attendeeId]` | Update attendee | Mixed |
| DELETE | `/api/events/[id]/attendees/[attendeeId]` | Delete attendee | Protected |
| GET | `/api/events/[id]/attendees/search` | Search attendees | Public |
| POST | `/api/events/[id]/attendees/walkin` | Walk-in registration | Public |
| POST | `/api/events/[id]/attendees/[attendeeId]/checkin` | Mark checked in | Public |
| **Templates** |
| GET | `/api/events/[id]/template` | Get template | Protected |
| PUT | `/api/events/[id]/template` | Save template | Protected |
| POST | `/api/events/[id]/template/preview` | Generate preview | Protected |
| **Import** |
| POST | `/api/events/[id]/import/upload` | Upload Excel file | Protected |
| GET | `/api/events/[id]/import/preview` | Preview import data | Protected |
| POST | `/api/events/[id]/import/confirm` | Confirm import | Protected |
| GET | `/api/events/[id]/import/mappings` | Get field mappings | Protected |
| PUT | `/api/events/[id]/import/mappings` | Save field mappings | Protected |
| **Generation** |
| POST | `/api/generate` | Generate AI badge image | Public |
| GET | `/api/generate/status` | Check AI service status | Public |
| **Printing** |
| GET | `/api/print/printers` | Discover available printers | Public |
| POST | `/api/print/job` | Create print job | Public |
| GET | `/api/print/job/[jobId]` | Get job status | Public |
| **WebSocket** |
| WS | `/api/ws` | Real-time sync connection | Public |

## Agents Included

| Agent | Purpose | Customizations |
|-------|---------|----------------|
| `dev-architect` | High-level design decisions | Focus on Next.js App Router patterns, AI integration architecture |
| `dev-frontend` | React components, UI | Fabric.js canvas editor, shadcn/ui components, Tailwind styling |
| `dev-backend` | Business logic, services | AI provider abstraction, template processing, import logic |
| `dev-api` | API route design | Next.js route handlers, Zod validation, consistent responses |
| `dev-database` | Query optimization, seeds | PostgreSQL with Prisma, attendee search optimization |
| `dev-integration` | External APIs | Gemini API integration, printer SDK integration |
| `dev-test` | Testing strategy | Unit tests for AI layer, E2E for check-in flow |
| `dev-reviewer` | Code review | Next.js best practices, TypeScript strictness |

### Agent Customizations

**dev-frontend:**
- Stack: React 18, Next.js App Router, Tailwind CSS, shadcn/ui
- Special: Fabric.js canvas integration for template designer
- Paths: `src/components/`, `src/app/`, `src/hooks/`

**dev-backend:**
- Stack: Next.js API routes, Prisma ORM
- Patterns: Service layer for business logic, repository pattern via Prisma
- Paths: `src/lib/`, `src/app/api/`

**dev-integration:**
- Focus: Google Gemini API (image generation), Label printer SDKs
- Patterns: Provider abstraction, retry with backoff, fallback handling
- Paths: `src/lib/ai/`, `src/lib/printer/`

## Conventions

### File Organization

- **Route groups** for admin vs check-in: `(admin)/` and `(checkin)/`
- **Feature-based components**: Group by feature (template-designer, checkin, admin)
- **Colocation**: Keep hooks, types, and tests near the code they support
- **Lib for shared logic**: Business logic and utilities in `src/lib/`

### Naming

- **Files**: kebab-case for all files (`attendee-search.tsx`, `field-mapper.ts`)
- **Components**: PascalCase (`AttendeeSearch`, `TemplateDesigner`)
- **Hooks**: camelCase with `use` prefix (`useAttendees`, `usePrinter`)
- **Types**: PascalCase with descriptive suffixes (`AttendeeInput`, `GenerationResult`)
- **API routes**: kebab-case matching resource (`/api/events/[id]/attendees`)

### Patterns

- **Server Components**: Default for pages, use `'use client'` only when needed
- **Server Actions**: For form submissions where appropriate
- **React Query**: For client-side data fetching with caching
- **Zod**: For all input validation (API and forms)
- **Error Boundaries**: Wrap major sections for graceful degradation

### Code Style

- **TypeScript strict mode**: No `any`, explicit return types for public functions
- **Async/await**: Prefer over raw promises
- **Early returns**: For cleaner conditionals
- **Destructuring**: For props and function parameters

## Tech Stack Details

### Next.js 14+

- **Version**: 14.x (latest stable)
- **Purpose**: Full-stack React framework
- **Patterns**:
  - App Router for file-based routing
  - Route groups for logical organization
  - API routes for backend endpoints
  - Server components by default

### Supabase + Prisma

- **Version**: PostgreSQL 15+ (Supabase hosted), Prisma 5.x
- **Purpose**: Managed relational database with type-safe ORM
- **Patterns**:
  - Prisma Client for all database access
  - Supabase connection pooling for serverless
  - Migrations for schema changes
  - Seed scripts for development data

### Google Gemini API

- **Model**: `gemini-2.0-flash-exp` (for image generation)
- **Purpose**: AI-generated name tag images
- **Patterns**:
  - Abstracted behind provider interface
  - Automatic fallback to plain-text rendering
  - Rate limiting and retry logic

### Fabric.js

- **Version**: 6.x
- **Purpose**: Canvas-based template designer
- **Patterns**:
  - Custom object types for AI zones and placeholders
  - JSON serialization for template storage
  - Event handling for real-time property updates

### Label Printer SDKs

- **Dymo**: DYMO Label Web SDK
- **Brother**: Brother b-PAC SDK
- **Zebra**: Browser Print API
- **Fallback**: Native browser print dialog

## Quality Standards

### Testing

- **Unit tests**: Jest for lib functions, especially AI and template processing
- **Integration tests**: API route testing with test database
- **E2E tests**: Playwright for critical user flows (check-in, template design)
- **Coverage target**: 80% for `src/lib/`, 60% overall

### Documentation

- **API**: OpenAPI spec generated from Zod schemas
- **Components**: JSDoc for complex components
- **Architecture decisions**: ADRs in `docs/DECISIONS/`

### Review Process

- **PR required**: All changes go through PR
- **CI checks**: Lint, type-check, tests must pass
- **Agent review**: `dev-reviewer` for final check

## Customization Specifications

### CLAUDE.md Customizations

```markdown
# BigBadge - AI-Powered Name Tag Generator

## Quick Reference

- **Stack**: Next.js 14, TypeScript, Supabase (PostgreSQL), Prisma, Tailwind, Fabric.js
- **Deployment**: Vercel + Supabase
- **Repository**: big-badge (GitHub)
- **AI**: Google Gemini API (Nano Banana) with fallback
- **Testing**: Jest + Playwright

## Critical Files

- `src/lib/ai/provider.ts` - AI abstraction interface
- `src/lib/ai/gemini.ts` - Gemini implementation
- `src/components/template-designer/Canvas.tsx` - Template editor
- `prisma/schema.prisma` - Database schema

## Commands

- `npm run dev` - Start development server
- `npm run db:migrate` - Run database migrations
- `npm run db:seed` - Seed development data
- `npm test` - Run tests
- `npm run lint` - Run ESLint

## Environment Variables

- `DATABASE_URL` - Supabase PostgreSQL connection string (with pooling)
- `DIRECT_URL` - Supabase direct connection (for migrations)
- `GEMINI_API_KEY` - Google Gemini API key
- `NEXTAUTH_SECRET` - Authentication secret
- `NEXTAUTH_URL` - Application URL (Vercel deployment URL)
```

### Agent File Customizations

**dev-integration.md additions:**
- Gemini API specifics (aspect ratios, rate limits)
- Printer SDK documentation links
- Fallback generation patterns

**dev-frontend.md additions:**
- Fabric.js patterns and gotchas
- shadcn/ui component usage
- Real-time sync with WebSocket

---

Based on: nametag-generator-brief.md
Created: 2026-01-22
Status: Pending Approval
