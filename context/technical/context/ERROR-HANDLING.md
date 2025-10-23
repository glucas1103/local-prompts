# Error Handling & Logging

## Metadata

- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/utils/logger.ts`, `apps/api/src/instrument.ts`, `apps/api/src/server.ts`, `apps/web/src/app/global-error.tsx`

## Technical Overview

Error handling implements a comprehensive strategy combining **structured logging** (Pino), **error tracking** (Sentry), and **standardized error responses**. The system enforces consistent error formatting across all layers with SCREAMING_SNAKE_CASE error codes, detailed logging with event classification, and automatic error capturing for monitoring.

Frontend errors are caught by React error boundaries and Next.js error handlers, logging to Sentry with user context. Backend errors flow through Express error middleware, logging structured data to Pino (pretty-printed in development, JSON in production) and capturing exceptions in Sentry with request context.

All logs follow a strict payload structure with three required fields: `msg` (human-readable message), `event` (event classification), and `metadata` (contextual data). This structure enables efficient log searching, error aggregation, and debugging across environments.

## Architecture & Design

### Error Flow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Error Propagation Flow                    │
└─────────────────────────────────────────────────────────────┘

Error Occurs (Database, External API, Validation, etc.)
     ↓
┌─────────────────────────────────────┐
│  try/catch in Route Handler         │
│  - Capture error                    │
│  - Log with structured format       │
│  - Send to Sentry (if 500)          │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│  Return HTTP Error Response         │
│  - SCREAMING_SNAKE_CASE code        │
│  - Appropriate status code          │
│  - Optional details (validation)    │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│  Express Error Handler              │
│  - Sentry.setupExpressErrorHandler  │
│  - Global error middleware          │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│  Observability                      │
│  - Pino logs (console/file)         │
│  - Sentry dashboard (errors)        │
│  - Request ID for tracing           │
└─────────────────────────────────────┘
```

### Logging Layers

**1. HTTP Request Logging** (`loggerMiddleware`):

- Automatically logs all incoming requests
- Logs response time and status
- Redacts sensitive data (auth headers, passwords)

**2. Application Logging** (`logger`):

- Manual logging for business events
- Structured format enforced
- Different log levels (info, warn, error, debug)

**3. Error Tracking** (Sentry):

- Automatic capture of unhandled errors
- Manual capture with context
- Performance monitoring

## Configuration & Setup

### Logger Configuration

**File**: `apps/api/src/utils/logger.ts`

```typescript
import pino from "pino";

interface LogPayload {
  msg: string;
  event: string;
  metadata?: Record<string, unknown>;
}

const baseLogger = pino({
  level: process.env.LOG_LEVEL || "info",
  ...(process.env.NODE_ENV === "development"
    ? {
        transport: {
          target: "pino-pretty",
          options: {
            colorize: true,
            translateTime: "HH:MM:ss.l",
            ignore: "pid,hostname",
          },
        },
      }
    : {
        formatters: {
          level: (label: string) => ({ level: label }),
        },
        timestamp: () => `,"time":"${new Date().toISOString()}"`,
        messageKey: "msg",
        singleLine: true,
      }),
  serializers: pino.stdSerializers,
});

// Helper to process metadata with error objects
const processMetadata = (metadata?: Record<string, unknown>) => {
  if (!metadata?.error) return metadata;

  const error = metadata.error as Error;
  return {
    ...metadata,
    message: error.message,
    stack: error.stack,
  };
};

// Wrapped logger enforcing payload structure
export const logger = {
  info: (payload: LogPayload) =>
    baseLogger.info({
      ...payload,
      metadata: processMetadata(payload.metadata),
    }),

  error: (payload: LogPayload) =>
    baseLogger.error({
      ...payload,
      metadata: processMetadata(payload.metadata),
    }),

  warn: (payload: LogPayload) =>
    baseLogger.warn({
      ...payload,
      metadata: processMetadata(payload.metadata),
    }),

  debug: (payload: LogPayload) =>
    baseLogger.debug({
      ...payload,
      metadata: processMetadata(payload.metadata),
    }),
};

export { baseLogger };
```

### HTTP Request Logger

**File**: `apps/api/src/middlewares/loggerMiddleware.ts`

```typescript
import pinoHttp from "pino-http";
import { baseLogger } from "../utils/logger";

const isDevelopment = process.env.NODE_ENV === "development";

export const loggerMiddleware = pinoHttp({
  logger: baseLogger,

  // Don't log OPTIONS requests
  autoLogging: {
    ignore: (req) => req.method === "OPTIONS",
  },

  // Custom request serialization
  serializers: {
    req(req) {
      return {
        id: req.id,
        method: req.method,
        url: req.url,
        body: req.method === "POST" ? req.raw.body : undefined,
        query: req.query,
        params: req.params,
        headers: req.headers,
        remoteAddress: req.remoteAddress,
      };
    },
  },

  // Redact sensitive data
  redact: [
    "req.headers.authorization",
    "req.headers.cookie",
    "*.password",
    "*.token",
    "*.secret",
  ],

  // Custom success/error messages
  customSuccessMessage: (req, res) =>
    `${req.method} ${req.url} - ${res.statusCode}`,
  customErrorMessage: (req, res, err) =>
    `${req.method} ${req.url} - ${res.statusCode} - ${err.message}`,

  // Quieter logging in development
  quietReqLogger: isDevelopment,
  quietResLogger: isDevelopment,
});
```

### Sentry Configuration

**Backend** (`apps/api/src/instrument.ts`):

```typescript
import * as Sentry from "@sentry/node";
import { nodeProfilingIntegration } from "@sentry/profiling-node";

Sentry.init({
  dsn: process.env.SENTRY_API_DSN,
  release: process.env.COMMIT_SHA,
  integrations: [nodeProfilingIntegration()],
  tracesSampleRate: 1.0, // Capture 100% of transactions
  environment: process.env.NODE_ENV,
});
```

**Frontend** (`apps/web/sentry.server.config.ts`):

```typescript
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_WEB_DSN,
  environment: process.env.NEXT_PUBLIC_NODE_ENV || "development",
  release: process.env.NEXT_PUBLIC_COMMIT_SHA,
  tracesSampleRate: 1,
  debug: false,
});
```

### Global Error Handler

**File**: `apps/api/src/server.ts`

```typescript
// Sentry error handler (must be before custom error handler)
Sentry.setupExpressErrorHandler(app);

// Custom error handler
app.use(function onError(err, req, res, next) {
  // Sentry ID attached to response
  const sentryId = res.sentry;

  logger.error({
    msg: "Error in server",
    event: "server-error",
    metadata: {
      message: err.message,
      stack: err.stack,
      sentryId,
    },
  });

  res.statusCode = 500;
  res.json({
    message: "Internal server error",
    sentryId, // Include for user support tickets
  });
});
```

## Patterns & Best Practices

### ✅ DO: Use Structured Logging

```typescript
// GOOD: Structured with event classification
logger.info({
  msg: "User created successfully",
  event: "user-create",
  metadata: { userId: user.id, organizationId: org.id },
});

logger.error({
  msg: "Database operation failed",
  event: "database-error",
  metadata: { error, operation: "insert", table: "users" },
});

logger.warn({
  msg: "Rate limit approaching",
  event: "rate-limit",
  metadata: { userId, requestCount: 95, limit: 100 },
});

logger.debug({
  msg: "Cache hit",
  event: "cache-hit",
  metadata: { key: "user:123", ttl: 3600 },
});
```

### ✅ DO: Use SCREAMING_SNAKE_CASE for Error Codes

```typescript
// GOOD: Clear, searchable error codes
res.status(404).json({ error: 'USER_NOT_FOUND' });
res.status(401).json({ error: 'UNAUTHORIZED - USER ID NOT FOUND' });
res.status(500).json({ error: 'FAILED_TO_CREATE_ORGANIZATION' });
res.status(400).json({ error: 'REQUEST_VALIDATION_ERROR', details: [...] });
```

### ✅ DO: Log Before Returning Errors

```typescript
// GOOD: Log then respond
router.post("/organizations", authMiddleware, async (req, res) => {
  try {
    const [org] = await req.tx
      .insert(organizations)
      .values(req.body)
      .returning();
    res.status(201).json(org);
  } catch (error) {
    logger.error({
      msg: "Failed to create organization",
      event: "create-organization",
      metadata: { error, body: req.body },
    });
    res.status(500).json({ error: "CREATE_ORG_FAILED" });
  }
});
```

### ✅ DO: Use Sentry Context Helpers

```typescript
import { captureExceptionWithContext } from "@/utils/sentry-utils";

router.get("/user", authMiddleware, async (req, res) => {
  try {
    // ... operation
  } catch (error) {
    captureExceptionWithContext({
      req,
      error,
      msg: "Error fetching user",
      event: "get-user",
      metadata: { userId: req.auth.userId },
    });
    res.status(500).json({ error: "FAILED_TO_GET_USER" });
  }
});
```

### ✅ DO: Include Request ID for Tracing

```typescript
// Request ID automatically added by pino-http
logger.error({
  msg: "Payment failed",
  event: "payment-error",
  metadata: {
    error,
    userId: req.auth.userId,
    requestId: req.id, // Pino HTTP assigns unique ID
  },
});
```

### ❌ DON'T: Use console.log

```typescript
// BAD: No structure, not searchable
console.log("User created:", user);
console.error("Error:", error);

// GOOD: Structured logging
logger.info({
  msg: "User created",
  event: "user-create",
  metadata: { userId: user.id },
});

logger.error({
  msg: "Operation failed",
  event: "database-error",
  metadata: { error },
});
```

**Rule**: `no-console: error` enforced by ESLint.

### ❌ DON'T: Log Sensitive Data

```typescript
// BAD: Logging passwords, tokens
logger.info({
  msg: "User login",
  event: "login",
  metadata: {
    password: req.body.password, // ❌ NEVER!
    token: req.headers.authorization, // ❌ NEVER!
  },
});

// GOOD: Redact sensitive data
logger.info({
  msg: "User login attempt",
  event: "login",
  metadata: {
    userId: user.id,
    ipAddress: req.ip,
  },
});
```

**Auto-Redaction**: Pino HTTP middleware redacts:

- `req.headers.authorization`
- `req.headers.cookie`
- `*.password`
- `*.token`
- `*.secret`

### ❌ DON'T: Swallow Errors Silently

```typescript
// BAD: Silent failure
try {
  await riskyOperation();
} catch (error) {
  // Nothing - error lost!
}

// GOOD: Log even if you handle it
try {
  await riskyOperation();
} catch (error) {
  logger.warn({
    msg: "Operation failed, using fallback",
    event: "operation-fallback",
    metadata: { error },
  });
  useFallback();
}
```

## Usage Examples

### Complete Error Handling in Route

```typescript
router.post(
  "/organizations",
  validateRequestBody(createOrgSchema),
  validateResponse(organizationSchema),
  authMiddleware,
  async (req, res) => {
    const { name, plan } = req.body;
    const { userId } = req.auth;

    try {
      // Log operation start
      logger.info({
        msg: "Creating organization",
        event: "create-organization-start",
        metadata: { name, userId },
      });

      const [org] = await req.tx
        .insert(organizations)
        .values({ name, plan })
        .returning();

      // Log success
      logger.info({
        msg: "Organization created successfully",
        event: "create-organization-success",
        metadata: { organizationId: org.id, name },
      });

      res.status(201).json(org);
    } catch (error) {
      // Log error with full context
      logger.error({
        msg: "Failed to create organization",
        event: "create-organization-error",
        metadata: {
          error,
          name,
          userId,
          requestId: req.id,
        },
      });

      // Capture in Sentry with context
      captureExceptionWithContext({
        req,
        error,
        msg: "Organization creation failed",
        event: "create-organization-error",
        metadata: { name, userId },
      });

      // Return user-friendly error
      res.status(500).json({ error: "CREATE_ORG_FAILED" });
    }
  }
);
```

### Frontend Error Boundary

**File**: `apps/web/src/app/global-error.tsx`

```typescript
'use client';

import * as Sentry from '@sentry/nextjs';
import { useEffect } from 'react';

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log error to Sentry
    Sentry.captureException(error);
  }, [error]);

  return (
    <html>
      <body>
        <div>
          <h2>Something went wrong!</h2>
          <p>Error ID: {error.digest}</p>
          <button onClick={reset}>Try again</button>
        </div>
      </body>
    </html>
  );
}
```

### Validation Error Handling

```typescript
export function validateRequestBody(schema) {
  return (req, res, next) => {
    try {
      req.body = schema.parse(req.body);
      next();
    } catch (error) {
      if (isZodError(error)) {
        const errorMessages = formatZodError(error);

        logger.error({
          msg: "Request validation error",
          event: "request-validation",
          metadata: { errorMessages, body: req.body },
        });

        res.status(400).json({
          error: "REQUEST_VALIDATION_ERROR",
          details: errorMessages, // Detailed field errors
        });
      } else {
        logger.error({
          msg: "Unexpected validation error",
          event: "request-validation",
          metadata: { error },
        });
        res.status(500).json({ error: "Internal Server Error" });
      }
    }
  };
}
```

### Database Error Handling

```typescript
router.get("/user/:id", authMiddleware, async (req, res) => {
  const { id } = req.params;

  try {
    const [user] = await req.tx
      .select()
      .from(users)
      .where(eq(users.id, id))
      .limit(1);

    if (!user) {
      // Not an error, just not found
      logger.debug({
        msg: "User not found",
        event: "get-user-not-found",
        metadata: { userId: id },
      });
      return res.status(404).json({ error: "USER_NOT_FOUND" });
    }

    res.json(user);
  } catch (error) {
    // Database error - this is serious
    logger.error({
      msg: "Database query failed",
      event: "database-error",
      metadata: { error, userId: id, table: "users" },
    });

    // Sentry will auto-capture if using setupExpressErrorHandler
    res.status(500).json({ error: "FAILED_TO_GET_USER" });
  }
});
```

## Patterns & Best Practices

### ✅ DO: Use Structured Logging

```typescript
logger.info({
  msg: "User created",
  event: "user-create",
  metadata: { userId: user.id },
});

logger.error({
  msg: "Database operation failed",
  event: "database-error",
  metadata: { error },
});
```

### Log Levels Guide

**info**: Normal operations

```typescript
logger.info({
  msg: "Server started",
  event: "server-start",
  metadata: { port: 3001 },
});
logger.info({
  msg: "User logged in",
  event: "user-login",
  metadata: { userId },
});
```

**warn**: Unusual but handled situations

```typescript
logger.warn({
  msg: "Rate limit approached",
  event: "rate-limit-warning",
  metadata: { userId },
});
logger.warn({
  msg: "Deprecated API used",
  event: "deprecated-api",
  metadata: { endpoint },
});
```

**error**: Failures requiring attention

```typescript
logger.error({
  msg: "Payment failed",
  event: "payment-error",
  metadata: { error, userId },
});
logger.error({
  msg: "External API timeout",
  event: "api-timeout",
  metadata: { service: "github" },
});
```

**debug**: Detailed info for debugging

```typescript
logger.debug({
  msg: "Cache hit",
  event: "cache-hit",
  metadata: { key: "user:123" },
});
logger.debug({
  msg: "Query executed",
  event: "db-query",
  metadata: { sql, duration },
});
```

## Technical Dependencies

**Logging**:

- `pino`: ^9.7.0 - Fast logger
- `pino-http`: ^10.5.0 - HTTP request logger
- `pino-pretty`: ^13.0.0 - Dev pretty printer

**Error Tracking**:

- `@sentry/node`: ^9.22.0 - Backend error tracking
- `@sentry/nextjs`: ^9.22.0 - Frontend error tracking
- `@sentry/profiling-node`: ^9.22.0 - Performance profiling

**Utilities**:

- Custom `sentry-utils.ts` - Helper functions

## Tests & Validation

### Testing Logger

```typescript
import { logger } from "@/utils/logger";

describe("Logger", () => {
  it("should log structured info", () => {
    const spy = jest.spyOn(console, "log");

    logger.info({
      msg: "Test message",
      event: "test-event",
      metadata: { key: "value" },
    });

    expect(spy).toHaveBeenCalled();
    // Verify structure in output
  });

  it("should process error metadata", () => {
    const error = new Error("Test error");

    logger.error({
      msg: "Error occurred",
      event: "test-error",
      metadata: { error },
    });

    // Should extract error.message and error.stack
  });
});
```

### Testing Error Handling

```typescript
describe("Error Handling", () => {
  it("should return 500 on database error", async () => {
    // Mock database to throw error
    jest.spyOn(req.tx, "select").mockRejectedValue(new Error("DB Error"));

    const response = await request(app).get("/api/user").expect(500);

    expect(response.body.error).toBe("FAILED_TO_GET_USER");
  });

  it("should log error details", async () => {
    const logSpy = jest.spyOn(logger, "error");

    await request(app).get("/api/user");

    expect(logSpy).toHaveBeenCalledWith({
      msg: expect.stringContaining("Error"),
      event: expect.any(String),
      metadata: expect.objectContaining({ error: expect.any(Error) }),
    });
  });
});
```

## Troubleshooting & FAQ

### Issue: Logs not showing in development

**Cause**: Log level too high or pino-pretty not installed.

**Solution**:

```bash
# Install pino-pretty
npm install --save-dev pino-pretty

# Set log level
export LOG_LEVEL=debug
npm run dev
```

---

### Issue: Sentry not capturing errors

**Cause**: Sentry not initialized or DSN missing.

**Solution**:

```bash
# Verify DSN
echo $SENTRY_API_DSN

# Check initialization order (instrument.ts must be imported first)
# apps/api/src/server.ts
import './instrument';  // Must be first!
```

---

### Issue: Logs missing context

**Cause**: Metadata not included or error object not processed.

**Solution**:

```typescript
// Include relevant context
logger.error({
  msg: "Operation failed",
  event: "operation-error",
  metadata: {
    error, // Will be processed to extract message/stack
    userId: req.auth.userId,
    organizationId: req.auth.orgId,
    requestId: req.id,
    endpoint: req.url,
  },
});
```

---

### Issue: Sensitive data in logs

**Cause**: Logging request body or headers without redaction.

**Solution**:

```typescript
// pino-http redacts automatically, but for manual logs:
logger.info({
  msg: "Request received",
  event: "request",
  metadata: {
    url: req.url,
    method: req.method,
    // DON'T log: req.headers, req.body directly
  },
});
```

## References

- **Main Files**:
  - `apps/api/src/utils/logger.ts` - Logger configuration
  - `apps/api/src/middlewares/loggerMiddleware.ts` - HTTP logging
  - `apps/api/src/instrument.ts` - Sentry initialization
  - `apps/api/src/server.ts` - Global error handler
  - `apps/web/src/app/global-error.tsx` - Frontend error boundary
- **Utilities**:
  - `apps/api/src/utils/sentry-utils.ts` - Sentry helpers
- **External Docs**:
  - [Pino Documentation](https://getpino.io/)
  - [Pino HTTP](https://github.com/pinojs/pino-http)
  - [Sentry Node.js](https://docs.sentry.io/platforms/node/)
  - [Sentry Next.js](https://docs.sentry.io/platforms/javascript/guides/nextjs/)
