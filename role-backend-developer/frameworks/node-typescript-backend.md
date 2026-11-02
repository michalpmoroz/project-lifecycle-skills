# Node.js 22 LTS & TypeScript 5.7+ Backend Implementation Guide (`frameworks/node-typescript-backend.md`)

> **Baseline Versions**: Node.js `22 LTS`, TypeScript `5.7+` (`strict: true`, `noUncheckedIndexedAccess: true`), Fastify `5.x` / Hono `4.x` / NestJS `11.x`, Zod `3.24+`, Pino `9.x`, Drizzle ORM `0.38+` / Prisma `6.x`.
> **Mandate**: Load this helper when implementing or reviewing backend services, API servers, background workers, or CLI tools in Node.js and TypeScript.

> [!NOTE]
> **Continuous Evolution Mandate**: This guide establishes the baseline engineering floor for Node.js/TypeScript services. Continuously profile memory allocations, event-loop lag, and async context propagation under realistic load.

---

## 1. Request-Scoped Context & Structured Logging (`AsyncLocalStorage` + Pino)

Every incoming HTTP request, RPC call, or background job must propagate a correlation context (`requestId`, `traceId`, `tenantId`, `actorId`) without polluting every function signature.

```typescript
// src/infra/context/request-context.ts
import { AsyncLocalStorage } from 'node:async_hooks';
import { randomUUID } from 'node:crypto';
import pino from 'pino';

export interface RequestContext {
  requestId: string;
  traceId: string;
  tenantId?: string;
  actorId?: string;
}

export const requestContextStorage = new AsyncLocalStorage<RequestContext>();

const baseLogger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  redact: {
    paths: [
      'req.headers.authorization',
      'req.headers.cookie',
      '*.password',
      '*.token',
      '*.secret',
      '*.creditCard',
      '*.ssn',
    ],
    censor: '[REDACTED]',
  },
  mixin() {
    const ctx = requestContextStorage.getStore();
    return ctx
      ? { requestId: ctx.requestId, traceId: ctx.traceId, tenantId: ctx.tenantId, actorId: ctx.actorId }
      : {};
  },
});

export const logger = baseLogger;

export function runWithRequestContext<T>(partial: Partial<RequestContext>, fn: () => T): T {
  const ctx: RequestContext = {
    requestId: partial.requestId ?? randomUUID(),
    traceId: partial.traceId ?? randomUUID(),
    tenantId: partial.tenantId,
    actorId: partial.actorId,
  };
  return requestContextStorage.run(ctx, fn);
}
```

---

## 2. Strict Zod Validation Pipeline (Params, Query, Body, Env)

- **Fail-Fast Environment Validation**: Validate `process.env` at startup using a strict Zod schema; crash immediately before binding ports if required secrets or URLs are missing or malformed.
- **Boundary Parsing (`z.strictObject`)**: Strip or reject unknown keys at API boundaries to eliminate Mass Assignment vulnerabilities:

```typescript
import { z } from 'zod';

// Multi-domain example: Creating a resource order/appointment/filing
export const CreateResourceMutationSchema = z.strictObject({
  idempotencyKey: z.string().uuid(),
  category: z.enum(['healthcare_appointment', 'ecommerce_order', 'civic_permit', 'fintech_transfer']),
  amountMinorUnits: z.number().int().nonnegative().max(100_000_000),
  currency: z.string().length(3).regex(/^[A-Z]{3}$/),
  metadata: z.record(z.string().max(64), z.union([z.string().max(500), z.number(), z.boolean()])).optional(),
});

export type CreateResourceMutationInput = z.infer<typeof CreateResourceMutationSchema>;
```

---

## 3. Event Loop Protection & Stream Processing

- **Never Block the Event Loop**:
  - Avoid synchronous APIs (`fs.readFileSync`, `crypto.pbkdf2Sync`, `zlib.gzipSync`) in request handlers.
  - Guard against Regular Expression Denial of Service (ReDoS) by using linear-time regexes or validating string lengths (`z.string().max(255)`) prior to regex evaluation.
  - Use Node.js **Streams / Web Streams (`pipeline`)** when exporting large datasets (e.g., CSV/JSON audit exports in FinTech or EHR exports in Healthcare) rather than buffering hundreds of megabytes in V8 heap memory:
    ```typescript
    import { pipeline } from 'node:stream/promises';
    import { Transform } from 'node:stream';

    export async function streamAuditCsv(cursorStream: AsyncIterable<Record<string, unknown>>, res: NodeJS.WritableStream) {
      const toCsvRow = new Transform({
        objectMode: true,
        transform(row, _encoding, callback) {
          callback(null, `${row.id},${row.tenantId},${row.occurredAt}\n`);
        },
      });
      await pipeline(cursorStream, toCsvRow, res);
    }
    ```
  - Offload CPU-intensive tasks (image processing, PDF generation, large cryptographic signing) to `worker_threads` or dedicated background queues.

---

## 4. Centralized Domain Error Mapping & Graceful Shutdown

- **Typed Domain Errors to RFC 9457 Problem Details**:
  - Domain services should throw typed domain errors (`ConflictError`, `ForbiddenError`, `ResourceNotFoundError`) rather than HTTP-coupled exceptions.
  - Map all uncaught errors in a single top-level HTTP error handler into `application/problem+json` with `requestContextStorage.getStore()?.traceId`, logging `5xx` errors at `logger.error({ err })` and returning a generic sanitized message to the client.
- **Deterministic Graceful Shutdown (`SIGTERM` / `SIGINT`)**:
  ```typescript
  export function registerGracefulShutdown(server: { close: () => Promise<void> }, closeResources: () => Promise<void>) {
    let shuttingDown = false;
    const shutdown = async (signal: string) => {
      if (shuttingDown) return;
      shuttingDown = true;
      logger.info({ signal }, 'Graceful shutdown initiated');
      const forceExitTimer = setTimeout(() => {
        logger.error('Shutdown timed out; forcing exit');
        process.exit(1);
      }, 20_000).unref();

      try {
        await server.close();
        await closeResources(); // e.g., dbPool.end(), redis.quit(), telemetry.shutdown()
        clearTimeout(forceExitTimer);
        process.exit(0);
      } catch (err) {
        logger.error({ err }, 'Error during graceful shutdown');
        process.exit(1);
      }
    };

    process.once('SIGTERM', () => void shutdown('SIGTERM'));
    process.once('SIGINT', () => void shutdown('SIGINT'));
  }
  ```
