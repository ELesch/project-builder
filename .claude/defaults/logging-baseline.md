# Logging Baseline

> Reference for logging requirements during discovery and initialization.
> Focus: Make logs AI-readable for efficient debugging across all platforms.

## Why Logging Matters for AI-Assisted Development

Claude Code and other AI tools can diagnose issues faster when logs are:
- **Structured** (JSON or similar, not free-form text)
- **Contextual** (include request IDs, user context, operation names)
- **Actionable** (clear error messages with stack traces)
- **Correlated** (trace requests across services)

These principles apply to ALL languages and platforms.

## Universal Logging Principles

### Log Levels (All Platforms)

| Level | Use For | AI Debugging Value |
|-------|---------|-------------------|
| `error` | Failures requiring attention | High - shows what broke |
| `warn` | Degraded but functional | Medium - shows risks |
| `info` | Key business events | Medium - shows flow |
| `debug` | Development details | High in dev - shows state |

### Structured Format (Language-Agnostic)

**Required fields for AI readability:**

```json
{
  "timestamp": "2026-01-23T10:30:00.000Z",
  "level": "error",
  "message": "Failed to create user",
  "requestId": "req_abc123",
  "context": "UserService.create",
  "error": {
    "type": "ValidationError",
    "message": "Email already exists",
    "code": "USER_EMAIL_DUPLICATE",
    "stack": "..."
  },
  "data": {
    "action": "createUser",
    "userId": "usr_xyz789"
  }
}
```

**Why this helps AI:**
- `requestId` - Correlate related log entries across services
- `context` - Know which module/function generated the log
- `error.code` - Machine-readable error type for pattern matching
- `error.stack` - Pinpoint exact failure location
- `data` - Understand what was being attempted

### Never Log (All Platforms)

- Passwords or secrets
- Full credit card numbers
- Personal health information (PHI)
- API keys or tokens
- Session secrets

## Logging Libraries by Language

### Node.js / TypeScript

| Library | Best For | Notes |
|---------|----------|-------|
| **Pino** | Performance, structured JSON | Recommended default |
| Winston | Flexibility, multiple transports | More configuration |
| Bunyan | JSON logging | Older but stable |

```typescript
// Pino example
import pino from 'pino';
const logger = pino({ level: 'info' });
logger.info({ userId, action: 'create' }, 'User created');
```

### Python

| Library | Best For | Notes |
|---------|----------|-------|
| **structlog** | Structured logging | Recommended default |
| loguru | Simple API, pretty output | Good for smaller projects |
| logging (stdlib) | No dependencies | Requires more setup |

```python
# structlog example
import structlog
logger = structlog.get_logger()
logger.info("user_created", user_id=user_id, action="create")
```

### Go

| Library | Best For | Notes |
|---------|----------|-------|
| **zerolog** | Zero-allocation, JSON | Recommended default |
| zap | High performance | Uber's library |
| logrus | Structured, mature | Widely used |

```go
// zerolog example
import "github.com/rs/zerolog/log"
log.Info().Str("userId", userID).Msg("User created")
```

### Rust

| Library | Best For | Notes |
|---------|----------|-------|
| **tracing** | Async, spans, structured | Recommended default |
| log | Simple facade | Use with env_logger |
| slog | Structured logging | Mature |

```rust
// tracing example
use tracing::{info, instrument};
#[instrument]
fn create_user(user_id: &str) {
    info!(user_id, "User created");
}
```

### Java / Kotlin

| Library | Best For | Notes |
|---------|----------|-------|
| **SLF4J + Logback** | Standard, JSON encoder | Recommended default |
| Log4j2 | Performance, async | Watch for vulnerabilities |
| kotlin-logging | Kotlin wrapper | Uses SLF4J |

```java
// SLF4J example
import org.slf4j.Logger;
Logger logger = LoggerFactory.getLogger(UserService.class);
logger.info("User created: userId={}", userId);
```

### C# / .NET

| Library | Best For | Notes |
|---------|----------|-------|
| **Serilog** | Structured, sinks | Recommended default |
| NLog | Flexible targets | Mature |
| Microsoft.Extensions.Logging | Built-in | Abstraction layer |

```csharp
// Serilog example
Log.Information("User created {@User}", new { UserId = userId });
```

### Ruby

| Library | Best For | Notes |
|---------|----------|-------|
| **Semantic Logger** | Structured, enterprise | Recommended default |
| Ougai | JSON structured | Bunyan-compatible |
| Logger (stdlib) | Simple | Basic |

```ruby
# Semantic Logger example
logger.info(message: 'User created', user_id: user_id)
```

### Swift / iOS

| Library | Best For | Notes |
|---------|----------|-------|
| **os.log** | Apple unified logging | Built-in, recommended |
| SwiftyBeaver | Multiple destinations | Cross-platform |
| CocoaLumberjack | Mature, fast | Objective-C heritage |

```swift
// os.log example
import os.log
let logger = Logger(subsystem: "com.app", category: "user")
logger.info("User created: \(userId)")
```

### Kotlin / Android

| Library | Best For | Notes |
|---------|----------|-------|
| **Timber** | Simple, extensible | Recommended default |
| kotlin-logging | Multiplatform | SLF4J wrapper |
| android.util.Log | Built-in | Basic |

```kotlin
// Timber example
Timber.i("User created: userId=%s", userId)
```

## Error Tracking Services (Cross-Platform)

All major error tracking services support multiple languages:

| Service | Languages Supported | Best For |
|---------|---------------------|----------|
| **Sentry** | JS, Python, Go, Java, .NET, Ruby, Rust, iOS, Android, + more | Most projects (free tier) |
| Datadog | All major | Full observability stack |
| New Relic | All major | Enterprise APM |
| Rollbar | All major | Error focus |
| Bugsnag | All major | Mobile apps |
| Raygun | All major | Crash reporting |

### Sentry SDK by Platform

| Platform | Package |
|----------|---------|
| Node.js | `@sentry/node` |
| Browser | `@sentry/browser` |
| React | `@sentry/react` |
| Next.js | `@sentry/nextjs` |
| Python | `sentry-sdk` |
| Go | `github.com/getsentry/sentry-go` |
| Java | `io.sentry:sentry` |
| .NET | `Sentry` |
| Ruby | `sentry-ruby` |
| Rust | `sentry` |
| iOS | `Sentry` (CocoaPods/SPM) |
| Android | `io.sentry:sentry-android` |

## Stack-Specific Recommendations

### Web (Next.js / TypeScript) - Default Stack
- **Logger**: Pino
- **Error Tracking**: Sentry (`@sentry/nextjs`)
- **Log Aggregation**: Vercel (built-in) or Datadog

### Python Backend (FastAPI / Django)
- **Logger**: structlog
- **Error Tracking**: Sentry (`sentry-sdk`)
- **Log Aggregation**: Datadog or CloudWatch

### Go Backend
- **Logger**: zerolog
- **Error Tracking**: Sentry (`sentry-go`)
- **Log Aggregation**: Datadog or stdout to container logs

### Mobile (React Native)
- **Logger**: react-native-logs
- **Error Tracking**: Sentry (`@sentry/react-native`)
- **Crash Reporting**: Sentry or Crashlytics

### Mobile (iOS Native)
- **Logger**: os.log (unified logging)
- **Error Tracking**: Sentry or Crashlytics
- **Analytics**: Firebase Analytics

### Mobile (Android Native)
- **Logger**: Timber
- **Error Tracking**: Sentry or Crashlytics
- **Analytics**: Firebase Analytics

### Desktop (Electron)
- **Logger**: electron-log
- **Error Tracking**: Sentry (`@sentry/electron`)

### CLI Tools
- **Logger**: Language-specific (Pino for Node, structlog for Python, zerolog for Go)
- **Error Tracking**: Usually not needed (stderr is sufficient)

## Discovery Questions

### For Non-Technical Users

> "I'll set up logging so errors are easy to find and fix. This is standard practice."

Only ask:
- "Should I include error tracking (recommended for catching issues early)?" → Sentry

### For Technical Users

- Logging library preference? (or use stack default)
- Error tracking service? (Sentry recommended)
- Log aggregation? (Vercel/Datadog/CloudWatch/none)

## Template Variables

The initializer uses these based on tech stack:

| Variable | Values |
|----------|--------|
| `{{LOGGING_LIBRARY}}` | pino, structlog, zerolog, tracing, serilog, etc. |
| `{{LOGGING_PACKAGE}}` | npm package, pip package, go module, etc. |
| `{{ERROR_TRACKING}}` | sentry, datadog, none |
| `{{ERROR_TRACKING_PACKAGE}}` | Platform-specific Sentry SDK |
| `{{LOG_AGGREGATION}}` | vercel, datadog, cloudwatch, none |

## Logger Template by Stack

The initializer creates the appropriate logger file based on detected/selected stack:

- **Node.js/TypeScript**: `src/lib/logger.ts`
- **Python**: `app/core/logging.py` or `src/logging_config.py`
- **Go**: `internal/logger/logger.go`
- **Rust**: `src/logging.rs`
- **Java**: `src/main/resources/logback.xml`
- **C#**: Logger configured in `Program.cs`

## AI Debugging Workflow (All Platforms)

When Claude Code encounters an error:

1. **Check logs** for the error with context
2. **Find requestId/traceId** to trace the full request
3. **Read stack trace** to identify failure point
4. **Check context data** to understand inputs
5. **Fix and verify** with new log output

Well-structured logs make this process fast and accurate regardless of language.

## Logging by Layer (Universal Pattern)

| Layer | What to Log | Example |
|-------|-------------|---------|
| API/Controller | Request received, response sent, validation errors | `{ action: "POST /users", status: 201 }` |
| Service | Business logic decisions, external calls | `{ action: "createUser", result: "success" }` |
| Repository/DAO | Database operations (in dev), errors | `{ action: "insert", table: "users" }` |
| External API | Request/response (sanitized), latency | `{ api: "stripe", latency_ms: 234 }` |

## Environment Configuration

| Environment | Log Level | Stack Traces | Sensitive Data |
|-------------|-----------|--------------|----------------|
| Development | debug | Full | Redacted (still) |
| Staging | info | Full | Redacted |
| Production | info/warn | Full | Redacted |

Always redact sensitive data, even in development, to prevent accidental leaks.
