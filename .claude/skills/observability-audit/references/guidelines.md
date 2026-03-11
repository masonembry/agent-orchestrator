# Observability Audit Guidelines

General observability principles to apply when auditing TypeScript applications. These apply across both the API (Node.js/Lambda/Fastify) and frontend (React SPA) repos.

---

## 1. Log Levels — Apply the Standard

Use the log-level decision framework from `docs/log-levels-standards.md`. The key rule: **level by impact, not vibes.**

| Level | Use when |
|-------|----------|
| `fatal` | App cannot continue. Startup failure, unrecoverable state. |
| `error` | Correctness or reliability at risk. Integration failures, invalid state, auth blocking the app. |
| `warn` | Expected operational issue, handled gracefully. 4xx handled, fallback triggered, rate limited. |
| `info` | Lifecycle and business events needed for ops. Session start/end, routing decisions, state transitions. |
| `debug` | Dev troubleshooting. Internal state, skip/branch logic, cache hits. Disable in prod unless needed. |
| `trace` | Very verbose. Function entry/exit, loop details. Disable in prod. |

**Common mis-levels to flag:**

- `error` used for a handled validation failure or expected 4xx → should be `warn` or `debug`
- `error` used for retry attempts (not the final failure) → should be `warn`; `error` only after all retries fail
- `info` used for high-frequency events (every message processed, every loop iteration) → should be `debug`
- `info` used for UI-only events that have no operational value → should be `debug`
- `warn` used when the system is actually in an incorrect state → should be `error`
- `fatal` used for a recoverable error (app can still reload/retry) → should be `error`

---

## 2. Structured Logging — Always Include Context

Log calls must be structured (object + message), not just strings.

Flag:
- `logger.error('Failed to fetch user')` with no `err` object — the stack trace and error details are lost: should be `logger.error({ err }, 'Failed to fetch user')`
- `logger.info('Request received')` with no correlation context — missing `requestId`, `sessionId`, `userId` or equivalent
- String concatenation instead of structured fields: `logger.info('User ' + userId + ' enrolled')` → should be `logger.info({ userId }, 'User enrolled')`
- Log calls that include the entire request or response body on sensitive endpoints (leaks PII)

---

## 3. Error Swallowing

Every `catch` block must do at least one of:
1. Re-throw the error
2. Log it with the `err` object included
3. Record a failure metric
4. All three (preferred for significant operations using `withMetrics`)

Flag:
- Empty catch blocks: `catch (err) { }` — complete blind spot
- Catch blocks that only update state without logging: `catch (err) { setError(true) }` — error detail lost
- Catch blocks that log without the error object: `catch (err) { logger.error('Something failed') }` — stack trace lost
- Catch blocks that swallow and return a default silently — especially in event handlers where failure should be surfaced

---

## 4. Metrics — Wrap Significant Operations

Significant async operations should be wrapped with `withMetrics` to get start/success/failure/duration metrics automatically.

Significant operations include:
- Calls to external APIs or third-party services
- Database reads and writes (DynamoDB, PostgreSQL, Redis)
- Kafka or SQS message publishing
- Any operation on a critical path where duration or failure rate matters

Flag:
- New service functions or handlers that call external dependencies without `withMetrics`
- Operations that are clearly on the critical path (user-facing, revenue-impacting) without instrumentation
- `withMetrics` used with `rethrow: false` and no compensating error handling or logging in the caller — failure may go undetected

---

## 5. Missing Logger

Every Lambda handler, Fastify route handler, and background worker must have access to a logger.

Flag:
- Lambda handlers that never initialize or use a logger
- Functions that perform side effects (DB writes, external API calls) with no logging at all
- New service modules that take no `logger` parameter when they perform meaningful operations

---

## 6. PII and Sensitive Data in Logs

Logs must not contain PII, credentials, or sensitive data.

Flag:
- Logging full request bodies on endpoints that accept passwords, tokens, or PII
- Structured log fields containing: `password`, `token`, `secret`, `ssn`, `creditCard`, `dob`, `accountNumber`, phone numbers, email addresses (unless the service explicitly handles these and logging is intentional)
- `logResult: true` (the default for `withMetrics`) on operations that return sensitive data — the result is logged at `info` level

---

## 7. Console Anti-patterns

In production code, use the approved logger — not `console.*`.

Flag:
- `console.log(...)`, `console.error(...)`, `console.warn(...)` in any non-test file
- `console.log` used for debugging left in after development
- Exception: test files (`*.test.ts`, `*.spec.ts`) and scripts are acceptable

---

## 8. Event-Driven Reliability

Event consumers (Kafka, SQS, SNS, DynamoDB Streams, Kinesis) must handle errors explicitly.

Flag:
- Kafka or SQS message handlers that catch errors and continue without logging or incrementing a failure metric
- Lambda functions triggered by queues without a DLQ configured in the CDK stack (check CDK files for `deadLetterQueue`)
- Event processor handlers that silently drop messages on parse failure (missing Zod validation + logging on invalid payloads)

---

## 9. Distributed Tracing

Services should propagate trace context for distributed tracing.

Flag:
- New Lambda handlers that don't initialize X-Ray tracing (check for `AWSXRay` or equivalent setup)
- Outgoing HTTP calls to other internal services that don't forward trace headers
- New services added to CDK stacks without X-Ray active tracing enabled
