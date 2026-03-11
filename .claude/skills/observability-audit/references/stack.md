# Expert Workspace Stack — Observability Patterns

Stack-specific observability knowledge for the Expert Workspace platform. Apply these alongside the general guidelines.

---

## Log Level Standard

The canonical reference is `docs/log-levels-standards.md` in `expert-workspace` (also linked from `expert-workspace-api`). Key points:

- **FATAL**: App must stop — startup failure, unrecoverable state (e.g., Agent SDK unavailable)
- **ERROR**: Correctness/reliability at risk — GAIA/API integration failures, invalid session state, auth blocking the app. Log retries at `warn`; `error` only after all retries fail.
- **WARN**: Expected, handled — 4xx handled gracefully, eligibility debounce, rate limiting, fallback triggered. Validation handled in UI → `debug` or below.
- **INFO**: Lifecycle/business events for ops — session lifecycle, GAIA websocket open/close, routing decisions, enrollment. Not high-frequency or UI-only events.
- **DEBUG**: Dev troubleshooting — queue state, compliance logic, cache checks, skip decisions.
- **TRACE**: Very verbose — session entry/exit points, loop/branch evaluation.

---

## Approved Loggers

### API: `@expert-workspace-api/logger` (Pino-based)

All services in `expert-workspace-api` must use this logger.

Usage patterns:
```ts
// Initialize at handler/server level with correlation context
const logger = rootLogger.child({ requestId, sessionId });

// Structured: always include context object before message
logger.info({ userId, action }, 'Enrollment completed');

// Errors: always include the err object
logger.error({ err, requestId }, 'getCustomerDetails API call failed');
```

Flag:
- `console.log`, `console.error`, `console.warn` in non-test code
- Logger initialized without correlation context on a Lambda handler (`requestId`, `sessionId`, or equivalent)
- `logger.error('message')` without `{ err }` — stack trace lost
- `logger.info({ ...request.body })` on endpoints accepting sensitive data

### Frontend: `@expert/logging` (pino-based)

All apps in `expert-workspace` must use this package for logging.

Flag:
- `console.log`, `console.error` in production application code
- Logging customer PII (account numbers, SSN, health info) in structured fields
- Sensitive data passed as log context

---

## `withMetrics` — Instrumentation Wrapper

Two implementations exist in `expert-workspace-api`:

1. **`@expert-workspace-api/runtime-utils-aws`** — `packages/runtime-utils-aws/metrics/with-metrics.ts`
   - Sends CloudWatch Embedded Metrics Format (EMF) via `sendCloudWatchMetrics`
   - Emits: `Start`, `Success`, `Failure`, `Duration` metrics with `actionName` dimension

2. **`@expert-workspace-api/open-telemetry`** — `packages/open-telemetry/lib/with-metrics.ts`
   - Sends OpenTelemetry metrics via `sendDurationMetric`, `sendEventMetric`, `sendErrorMetric`

Both wrappers:
- Log `debug` on start: `Starting ${actionName}...`
- Log `info` on success with duration and result (when `logResult: true`, the default)
- Log `error` on failure with `{ err }`
- Accept `rethrow` (default: `true`) and `logResult` (default: `true`) options

**Correct usage:**
```ts
const result = await withMetrics(
    'getCustomerDetails',
    () => customerDetailsClient.get(customerId),
    { logger }
);
```

**Flag:**
- Significant async operations (API calls, DB reads/writes, Kafka publishing) added without `withMetrics`
- `withMetrics` with `rethrow: false` and no compensating error handling in the caller — failures silently return `undefined`
- `logResult: true` (the default) on operations that return sensitive data — the result object is logged at `info`. Set `logResult: false` for sensitive returns.
- `withMetrics` wrapping trivial synchronous or near-instant operations (unnecessary noise)

---

## CloudWatch Metrics and Alerts

- CloudWatch alarms are defined in CDK stacks (e.g., `services/backoffice/cdk/alerts/`).
- `withMetrics` in `runtime-utils-aws` emits metrics via EMF — these feed CloudWatch dashboards and alarms.
- Flag: new critical-path operations added without corresponding CloudWatch alarm configuration in the CDK stack when the operation is user-facing or revenue-impacting.

---

## Lambda Handlers

Lambda handlers have specific observability requirements:

**Flag in Lambda handlers (`.lambda.ts` files):**
- No logger initialized at the handler level
- Logger initialized without `requestId` or a correlation identifier from the Lambda event context
- No `withMetrics` or equivalent on the primary handler action
- Missing X-Ray tracing setup (check for `AWSXRay.captureAWSv3Client` or X-Ray middleware)
- No handling of Lambda timeout — consider logging near timeout using `context.getRemainingTimeInMillis()`

---

## Event-Driven Services

### Kafka (`@expert-workspace-api/kafka-client`, `kafka-listener-worker`)

- Kafka consumer handlers use `withMetrics` in many services — flag new handlers that don't.
- Flag: message handlers that catch parse errors and `continue` without logging.
- Flag: processing errors that are swallowed in the consumer loop (silent message drop).

### SQS / SNS

- SQS Lambda triggers must have a DLQ configured in the CDK stack.
- Flag: new `SqsEventSource` or `SnsEventSource` in a CDK stack without a `deadLetterQueue` property.

### DynamoDB Streams, EventBridge, Kinesis

- Same principle: flag event source additions in CDK without DLQ/error destination configuration.
- Flag: stream processor Lambda handlers that don't log failed records.

---

## Fastify Servers

- Fastify servers should use a request-scoped logger (Fastify provides `request.log` by default with Pino).
- Flag: route handlers that initialize a new top-level logger instead of using `request.log` — breaks correlation.
- Flag: Fastify error handlers that return the raw `error.message` or `error.stack` to the client (observability + security issue).

---

## Frontend (expert-workspace)

### Logging — `@expert/logging` + `useLogger`

The logging package wraps Pino for browser use. It has two configurable transports: a console transport (for local dev) and an API backend transport (ships logs to the backend). Both have independent log level thresholds set at `initLogging` time.

**The preferred pattern inside React components and hooks is `useLogger`**, not `getLogger` directly. `useLogger` reads from `LoggerContext`, which has session-level context (`sessionId`, `partner`, `callSid`, `chatId`) already bound. This means every log call automatically carries the full session context without repeating it.

Source: `apps/expert-ui/src/sdk/logging/LoggerProvider.tsx`

**Correct usage:**
```ts
// In a component or hook — picks up all session context automatically
const logger = useLogger({ module: 'IntegratedFlow', supportTeam: 'sales' });

// Logging — bound fields (sessionId, partner, etc.) are included automatically
logger.info({ someData }, 'Eligibility check completed');
logger.error({ error }, 'Enrollment failed');
```

**`getLogger` is for code outside the React tree** (module-level utilities, non-component service functions). It bypasses `LoggerContext` and will not have session context bound.

**Flag:**
- `getLogger(...)` used inside a React component or hook — should be `useLogger(...)` to pick up session context
- `useLogger()` called without `module` — harder to trace logs to their source in production
- `console.log`, `console.error`, `console.warn` in component or service code — use `useLogger` instead
- `logger.error('message')` without `{ error }` — stack trace is lost; should be `logger.error({ error }, 'message')`
- Logging customer PII in structured fields: account numbers, SSN, phone, email (unless the module explicitly handles this data and logging is intentional and reviewed)
- `logger.info(...)` for high-frequency events (every message received, every render trigger) — use `debug`

### `withMetrics` — `@expert/cloudwatch-metrics`

Source: `packages/cloudwatch-metrics/src/withMetrics.ts`

Key differences from the API version:
- `failureLogLevel` option (default: `"warn"`, not `"error"`) — failures log at warn by default since many frontend failures are expected/handled
- Uses `@expert/logging` Logger type
- Sends metrics via `sendMetrics` (CloudWatch via backend relay)

Emits: `Start`, `Success`, `Failure`, `Duration` metrics with `actionName` dimension.

**Correct usage:**
```ts
const result = await withMetrics(
    'fetchEligibility',
    () => eligibilityClient.check(customerId),
    { logger }
);
```

**Flag:**
- Significant async operations added without `withMetrics` — API calls, SDK calls (GAIA, Amazon Connect, Twilio), eligibility checks, enrollment actions
- `rethrow: false` with no compensating error handling in the caller
- `logResult: true` (the default) on operations that return sensitive customer data — the full result is logged at `info`. Pass `logResult: false`.
- `failureLogLevel: "error"` used where `"warn"` is more appropriate (handled failure, expected condition), or vice versa — `"error"` should align with the log-level standard (correctness/reliability at risk)

### Session Recording — `@expert/monitoring` (FullStory, Appcues)

**FullStory** (`trackFullStoryEvent`, `setFullStoryIdentifier`):
- `trackFullStoryEvent(name, properties)` — session event tracking
- Flag: PII in event properties (`ssn`, `accountNumber`, `dateOfBirth`, full name beyond `displayName`, raw phone numbers)
- Flag: `setFullStoryIdentifier` called with properties that include unfiltered customer data objects — spread PII into FullStory unintentionally
- `devMode: true` suppresses FullStory in local dev — flag if this guard is removed or bypassed

**Appcues** (`trackAppcuesEvent`, `setAppcuesIdentifier`):
- Same PII concerns as FullStory — flag event properties containing sensitive customer fields

### TanStack Query Error Handling

TanStack Query mutations and queries should surface errors to observability, not just UI state.

**Flag:**
- `mutation.onError` or `query.onError` callbacks that update UI state without logging the error
- `useQuery`/`useMutation` with no `onError` handler on a critical-path query (session data, eligibility, enrollment)
- `throwOnError: false` (or equivalent) on a query where the error is silently swallowed with no logging

### React Error Boundaries

Error boundaries are the last line of defense for unhandled rendering errors.

**Flag:**
- Error boundary `componentDidCatch` or equivalent hook that catches errors without logging them (`logger.error({ error }, 'Unhandled render error')`)
- New feature areas that render complex async data without an error boundary wrapping them
- Error boundaries that render a fallback UI but don't emit a metric or log — the failure becomes invisible in production

### SDK Event Handlers (Amazon Connect, Twilio)

Amazon Connect and Twilio emit events from their SDKs. Error and disconnect events must be logged.

**Flag:**
- Connect or Twilio error event handlers that catch and ignore: `sdk.on('error', () => {})` — complete blind spot
- Event handlers that update Zustand state on error without logging — state is visible in devtools but not in production logs
- Missing error handlers on critical SDK events (connection failures, media errors, task failures)
