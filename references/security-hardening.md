# Security Hardening Deep-Dive

## Authentication & Credentials

### Password Handling
- Hash with bcrypt (cost ≥12), scrypt, or Argon2id. Nothing weaker.
- Unique cryptographically random salt per password.
- Minimum 10 characters, complexity checks, breached-password screening.

### Password Reset
- Tokens: single-use, cryptographically random (≥128-bit entropy), expire 15-30 min.
- No token leakage in referrer headers. Old tokens invalidated on new issuance.

### Sessions & Tokens
- **JWT:** RS256 or ES256 signing. Validate `alg` server-side, reject `none`. Access tokens 15-30 min. Refresh token rotation with family detection.
- **Cookies:** `HttpOnly`, `Secure`, `SameSite=Strict`. Regenerate session ID after login. Absolute + idle timeouts.
- Logout destroys session server-side.
- Account lockout or exponential backoff after 5 failed attempts (per-account AND per-IP).

### MFA
- TOTP: correct parameters, ±1 window drift max, backup codes hashed and single-use.

## Authorization & Access Control
- Every endpoint: auth required (unless explicitly public), correct role/permission enforced.
- IDOR testing: replay User A's request with User B's token — must fail.
- Privilege escalation: regular users can't hit admin endpoints or modify roles.
- Authorization centralized in middleware, not scattered per-handler.
- No endpoint returns: password hashes, internal IDs, other users' PII, stack traces, DB schema, internal URLs.

## Injection Prevention

### SQL Injection
- Zero raw SQL or string interpolation. Parameterized statements or ORM only.
- Test: `' OR 1=1--`, `'; DROP TABLE users;--`, `UNION SELECT`.
- DB user: least-privilege (no DROP, ALTER, GRANT in production).

### XSS
- All user content escaped/sanitized. Server-side and client-side.
- No `dangerouslySetInnerHTML`/`v-html`/`[innerHTML]` with user data without DOMPurify whitelist.
- Test: `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`, `javascript:alert(1)`.
- CSP: no `unsafe-inline` for scripts (use nonces), no `unsafe-eval`.

### CSRF
- Protection on all state-changing endpoints if cookie-based auth.
- Bearer token in Authorization header only: document why CSRF unnecessary.

### Other Vectors
- **Command injection:** No `exec`/`spawn`/`system`/`popen` with user input. Whitelist if unavoidable.
- **Path traversal:** Validate/sanitize file paths. Test `../../etc/passwd`.
- **SSRF:** Allowlist domains, block internal ranges (127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.169.254).
- **XXE:** Disable external entities in XML parsers.
- **ReDoS:** Audit regex on user input for catastrophic backtracking.

## Data Protection

### Encryption in Transit
- HTTPS-only. HTTP 301 → HTTPS. HSTS `max-age ≥ 1 year`, `includeSubDomains`.
- TLS 1.2 minimum (prefer 1.3). DB connections via TLS.

### Encryption at Rest
- AES-256-GCM for PII, financial data, auth secrets. Keys separate from data.

### PII
- Audit every table for PII. Justify collection. Protect appropriately.
- No PII in logs.
- No sensitive data in CDN/browser cache (`Cache-Control` headers).
- Data retention: deletion on account closure, expired token cleanup.

## HTTP Security Headers
- `Content-Security-Policy` — strict, nonces for scripts
- `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy` — disable unused browser features
- Remove `X-Powered-By`, `Server`

### CORS
- Specific origins only, never `*` with credentials.
- Restrict methods and headers.

## Rate Limiting
- Login: 5/15min per account + per IP
- Signup: 3/hour per IP
- Password reset: 3/hour per account
- General API: per-user and per-IP limits
- 429 response with `Retry-After`
- Request size limits, JSON depth limits, pagination enforced.

## Infrastructure
- Debug modes OFF in production.
- Admin/debug/docs routes removed or auth-locked.
- DB, Redis, queues not publicly accessible.
- File uploads: validate by magic bytes, enforce size, store outside webroot, random filenames.
- No `.env`, `.git`, `docker-compose.yml`, backups publicly accessible.

## Security Logging
Log with timestamp, user ID, IP, user-agent:
- Login success/failure, password changes/resets
- Authorization failures (403s), suspicious input validation
- Rate limit hits, account lockouts, admin actions
- No PII/tokens/passwords in logs.
