# Expert Workspace Stack — Security Patterns

Stack-specific security knowledge for the Expert Workspace platform. Apply these in addition to the general guidelines.

---

## expert-workspace-api (Node.js / Lambda / Fastify)

### Secrets

- Secrets are stored in **AWS Secrets Manager** and accessed via `@expert-workspace-api/shared-secrets`.
- Flag: secrets read directly from `process.env` without going through the shared-secrets layer (e.g., `process.env.JWT_SECRET`, `process.env.API_KEY`).
- Flag: credentials committed in any config file not covered by `.gitignore`.
- The `local-secrets/` directory is gitignored and checked by the pre-commit hook (Lefthook). Flag: anything that looks like it should be in `local-secrets/` being committed to a tracked file.

### Sensitive Data Encryption

- `@expert-workspace-api/shared-kms` provides KMS encryption helpers for sensitive fields at rest.
- Flag: sensitive fields (PII, financial data, health data) stored in DynamoDB or PostgreSQL in plaintext when they were previously encrypted or are clearly sensitive.

### Input Validation — Fastify + Zod

- All Fastify route handlers must declare a Zod schema for `body`, `querystring`, and `params`.
- Flag: `fastify.post(...)` or `fastify.put(...)` routes without a `schema: { body: zodSchema }` or equivalent.
- Flag: routes that cast `request.body as SomeType` without validation.
- Zod schemas from the `@expert-workspace-api/shared-types` package are validated — trust them. Flag only when schema is absent or bypassed.

### Authentication

- Services use JWT tokens validated via middleware. Flag: Lambda handlers or Fastify routes that skip auth middleware on non-public endpoints.
- Flag: JWT tokens used without explicit `verify()` (not just `decode()`).
- Flag: endpoints that accept `userId` or `tenantId` from the request body without checking it matches the authenticated principal.

### Database — Drizzle ORM (PostgreSQL)

- Drizzle ORM parameterizes queries by default — safe.
- Flag: use of the `sql` tagged template literal (`` sql`...` ``) with string-interpolated untrusted values. Drizzle's `sql` helper supports parameterization (`` sql`WHERE id = ${userId}` `` is safe); flag cases where the interpolation is structural (table names, column names) rather than parameterized values.

### Database — DynamoDB (dynamodb-toolbox)

- dynamodb-toolbox is safe by default.
- Flag: constructing FilterExpression strings by concatenating untrusted input.
- Flag: `KeyConditionExpression` with string interpolation from request parameters.

### CORS — Fastify

- Flag: `origin: '*'` on services that handle authenticated requests or contain sensitive data.
- Flag: `origin: (origin) => origin` (reflects any origin).

### Lambda IAM Roles — CDK

- CDK stacks define Lambda execution roles. Flag in CDK files:
  - `PolicyStatement` with `actions: ['*']` or `resources: ['*']`.
  - Broad managed policies like `AmazonDynamoDBFullAccess` or `AmazonS3FullAccess` when scoped policies suffice.
  - `grant*` methods called on a broader scope than the Lambda needs (e.g., `table.grantFullAccess` when `grantReadData` would suffice).

### Redis (ioredis / cache-client)

- Cache data via `@expert-workspace-api/cache-client`.
- Flag: sensitive PII or auth tokens cached without TTL (unbounded retention).
- Flag: cache keys that use user-supplied input without sanitization.

### Event-Driven Surfaces

- Lambda handlers triggered by Kafka, SQS, SNS, EventBridge, DynamoDB Streams, or Kinesis receive external data — treat all payload fields as untrusted.
- Flag: payload fields from events used in database queries or external calls without Zod validation.

---

## expert-workspace (React 19 / Vite / Mantine)

### XSS

- Flag: `dangerouslySetInnerHTML` used with any non-static, non-sanitized content.
- Flag: `eval()` or dynamic `import()` with runtime-constructed strings.
- Flag: user-supplied strings rendered into `href` or `src` without protocol validation (could be `javascript:` URLs).

### Auth Token Storage

- Flag: auth tokens or session identifiers stored in `localStorage` or `sessionStorage`. These are accessible to any JavaScript on the page; prefer httpOnly cookies or in-memory storage.
- Flag: tokens stored in Zustand state with a persistence adapter writing to localStorage.

### Sensitive Data in State / Devtools

- The convention is `devtools` middleware disabled in production: `{ enabled: import.meta.env.VITE_ENV_TYPE !== "production" }`.
- Flag: Zustand stores using `devtools` without the production disable check — this leaks full store state to Redux DevTools in prod.
- Flag: sensitive fields (tokens, PII, account numbers) stored as top-level fields in Zustand stores where they'd be visible in devtools during development.

### API Keys / Credentials in Client Code

- `import.meta.env.VITE_*` vars are embedded in the client bundle — do not put secrets here.
- Flag: `import.meta.env.VITE_*` variables used for anything that is a credential or secret (API keys, private tokens).
- Flag: service credentials for Amazon Connect, Twilio, or any third party hardcoded in frontend source.

### Amazon Connect / Twilio

- Credentials for Amazon Connect and Twilio must come from the backend, not be embedded in the frontend bundle.
- Flag: Twilio auth tokens, AccountSID, or Amazon Connect credentials present in frontend code or environment variables.
- Flag: WebRTC or voice stream configurations that expose backend credentials client-side.

### Open Redirects

- Flag: `window.location.href = userInput` or `navigate(userInput)` without validating the URL is an internal path or a whitelisted external domain.
- Flag: `<a href={userInput}>` where `userInput` is not validated.
