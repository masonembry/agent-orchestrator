# Security Audit Guidelines

General security principles to apply when auditing TypeScript full-stack applications. These apply across both the API (Node.js/Lambda/Fastify) and frontend (React SPA) repos.

---

## 1. Secrets Management

**Never hardcode credentials, API keys, tokens, or secrets in source code.**

Flag:
- Hardcoded passwords, API keys, JWT secrets, or tokens in any file
- Secrets read from environment variables directly in application code (should go through a secrets management layer)
- Secrets committed via `.env` files or config files that should be gitignored
- Secrets passed as plaintext in environment variable names like `process.env.SECRET_KEY`

---

## 2. Input Validation

**All untrusted input must be validated at the entry point before use.**

Untrusted input includes: HTTP request bodies, query parameters, path parameters, event payloads (SQS, Kafka, EventBridge), and any data from external services.

Flag:
- Route handlers that use `req.body` or `req.query` without schema validation
- Parsing external event payloads without validation (e.g., raw `JSON.parse` on Kafka messages without Zod)
- Type assertions (`as SomeType`) applied to unvalidated external data
- Missing validation on path parameters used in database queries

---

## 3. Authentication and Authorization

**Every endpoint that accesses user or tenant data must verify identity and permissions.**

Flag:
- Route handlers with no auth middleware or token validation
- JWT tokens used without signature verification
- Authorization checks that only test for presence of a field, not its value
- Endpoints that accept a user/tenant ID from the request body without verifying it matches the authenticated identity (horizontal privilege escalation)
- Missing authz checks when accessing resources by ID

---

## 4. Injection

**Never interpolate untrusted data into queries, shell commands, or dynamic code.**

Flag:
- String interpolation in SQL queries (`` sql`SELECT * FROM ${table}` `` where `table` is user-supplied)
- NoSQL query injection: building DynamoDB FilterExpressions from unvalidated strings
- Template literal injection in shell commands
- `eval()` or `Function()` called with dynamic strings
- Dynamic `require()` or `import()` with user-supplied paths

---

## 5. Sensitive Data Exposure

**Do not expose internal system details, stack traces, or sensitive data in API responses or logs.**

Flag:
- Returning raw `Error` objects or `err.stack` in HTTP responses
- Logging request bodies wholesale on endpoints that accept sensitive data (passwords, tokens, PII)
- Logging sensitive fields in structured log objects: `logger.info({ token, password, ssn, creditCard })`
- Including internal database IDs, implementation details, or service names in client-facing error messages
- Returning more data than the client requested (over-fetching sensitive fields)

---

## 6. CORS Configuration

**CORS should whitelist specific trusted origins, not use wildcard.**

Flag:
- `origin: '*'` or `origin: true` on Fastify CORS for non-public endpoints
- CORS configuration that reflects back any origin from the request (`origin: (origin) => origin`)
- Missing `credentials: true` on endpoints that rely on cookie-based auth

---

## 7. IAM and Cloud Permissions

**AWS IAM policies should follow least privilege.**

Flag:
- `'*'` as the resource in an IAM policy action
- Wildcard actions (`iam:*`, `s3:*`, `dynamodb:*`) where specific actions suffice
- Lambda execution roles with broad managed policies (e.g., `AmazonDynamoDBFullAccess`)
- CDK constructs granting more permissions than the service needs
- S3 bucket policies or DynamoDB resource policies that allow public access

---

## 8. Cross-Site Scripting (XSS) — Frontend

**Never render untrusted HTML or execute dynamic strings as code.**

Flag:
- `dangerouslySetInnerHTML={{ __html: userContent }}` without sanitization
- `eval()`, `new Function()`, or `setTimeout(string)` with user-supplied strings
- Dynamic `<script>` tag injection
- Rendering user-supplied URLs in `href` or `src` without validation (could be `javascript:` URLs)

---

## 9. Sensitive Data in Client Storage — Frontend

**Do not store auth tokens or sensitive data in localStorage or sessionStorage.**

Flag:
- `localStorage.setItem('token', ...)` or `sessionStorage.setItem('token', ...)`
- Persisting auth tokens or PII in Zustand stores with persistence middleware
- API keys or credentials referenced in frontend code (even via `import.meta.env.VITE_*`)

---

## 10. Rate Limiting

**Public or unauthenticated endpoints should have rate limiting to prevent abuse.**

Flag:
- Fastify routes accessible without authentication that have no rate limit plugin applied
- Lambda functions triggered directly (not through API Gateway) without throttling configuration
- Endpoints that trigger expensive operations (AI, external APIs) without request rate controls

---

## 11. Dependency and Supply Chain

**Flag obvious supply chain risks in changed `package.json` or lock files.**

Flag:
- New direct dependencies added without a recognizable source or legitimate reason
- Downgraded dependencies that may reintroduce known CVEs
- `postinstall` scripts in newly added packages
- Dependencies that closely mimic the names of popular packages (typosquatting)
