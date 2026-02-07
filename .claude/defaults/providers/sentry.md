# Sentry Provider Guide

Error tracking and performance monitoring service.

## When to Use

- Default for error tracking (all project types)
- When you need crash reporting
- When you need performance monitoring

## Setup Steps

1. Go to sentry.io
2. Create new project for your platform
3. Copy the DSN (Data Source Name)

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_SENTRY_DSN=https://xxx@xxx.ingest.sentry.io/xxx
SENTRY_AUTH_TOKEN=sntrys_xxx  # For source maps
SENTRY_ORG=your-org
SENTRY_PROJECT=your-project
```

## SDK by Platform

| Platform | Package | Installation |
|----------|---------|--------------|
| Next.js | `@sentry/nextjs` | `npx @sentry/wizard@latest -i nextjs` |
| Node.js | `@sentry/node` | `npm install @sentry/node` |
| React | `@sentry/react` | `npm install @sentry/react` |
| Python | `sentry-sdk` | `pip install sentry-sdk` |
| Go | `sentry-go` | `go get github.com/getsentry/sentry-go` |
| Rust | `sentry` | Add to Cargo.toml |
| iOS | `Sentry` | SPM or CocoaPods |
| Android | `sentry-android` | Gradle |

## Next.js Setup (Recommended)

```bash
# Automatic setup
npx @sentry/wizard@latest -i nextjs
```

Or manual:

```typescript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});
```

```typescript
// sentry.server.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0,
});
```

## Node.js Setup

```typescript
import * as Sentry from "@sentry/node";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 1.0,
});

// Capture errors
try {
  // code
} catch (error) {
  Sentry.captureException(error);
}
```

## Python Setup

```python
import sentry_sdk

sentry_sdk.init(
    dsn=os.environ["SENTRY_DSN"],
    traces_sample_rate=1.0,
)

# Capture errors
try:
    # code
except Exception as e:
    sentry_sdk.capture_exception(e)
```

## Verification

```typescript
// Test error capture
Sentry.captureException(new Error("Test error"));
```

Then check Sentry dashboard for the error.

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Events not appearing | DSN wrong | Verify DSN in Sentry dashboard |
| Source maps missing | Auth token not set | Add SENTRY_AUTH_TOKEN |
| Too many events | No sampling | Set tracesSampleRate < 1.0 |
| Performance data missing | Traces disabled | Enable tracesSampleRate |

## Sample Rates (Production)

```typescript
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 0.1,  // 10% of transactions
  replaysSessionSampleRate: 0.1,  // 10% of sessions
  replaysOnErrorSampleRate: 1.0,  // 100% of error sessions
});
```

## Alternatives

| Service | Best For |
|---------|----------|
| **Sentry** | Default, free tier |
| Datadog | Full observability stack |
| Rollbar | Error focus |
| Bugsnag | Mobile apps |
