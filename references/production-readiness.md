# Production Readiness: Granular Checklist

> This is supplementary detail for Shipwright's SHIP MODE. The SKILL.md is the master — follow its phase numbering and structure. Use this file for granular sub-steps and deeper checklists when you need more specificity on a particular area. The section numbering here is internal to this document and does not correspond to SKILL.md phase numbers.

Fix issues as you find them. Test after every fix.

---

---

# PHASE 1 — CODE HEALTH & STRUCTURAL AUDIT

---

## 1.1 Dead Code & Hygiene

- Walk through every file. Identify and remove: dead code, unused imports, orphaned components, commented-out blocks, TODO/FIXME/HACK comments, and leftover debugging statements (`console.log`, `print`, `debugger`, `dd()`, `var_dump`).
- Search for and remove any placeholder or Lorem Ipsum text that made it into production code.
- Verify consistent naming conventions across the entire project — files, components, functions, DB columns, API routes. Fix inconsistencies.

## 1.2 Dependency Audit

- Verify all dependencies in `package.json` / `requirements.txt` / equivalent are actually used. Remove any that aren't.
- Run `npm audit` / `pip audit` / equivalent. Fix all critical and high vulnerabilities. Document any unfixable medium/low with justification.
- Pin dependency versions. Verify lockfiles are committed and integrity-checked.
- Check for typosquatting: verify all dependency names are correct and from official publishers.
- Review post-install scripts in dependencies for suspicious behavior.

## 1.3 Environment & Configuration

- Verify environment variables, config files, and secrets are properly handled — no hardcoded keys, no `.env` files committed, sensible fallback defaults where appropriate.
- Search the entire codebase AND `git log` for any previously committed secrets (API keys, database passwords, tokens). Flag for immediate rotation.
- Verify `.gitignore` properly covers `.env`, build artifacts, OS files, IDE configs, and any generated files.
- Verify all production environment variables are documented and accounted for.
- Verify build process works cleanly with zero warnings.

---

---

# PHASE 2 — FUNCTIONAL QA & END-TO-END TESTING

---

## 2.1 User Flow Testing

- Trace every user-facing flow end-to-end: signup, login, logout, password reset, onboarding, core CRUD operations, search, filtering, pagination, settings, profile management, and every feature-specific workflow.
- For each flow, test:
  - **Happy path**: Expected inputs, expected results.
  - **Edge cases**: Empty states, very long strings (1000+ chars), special characters (`<script>`, `' OR 1=1`, unicode, emoji, RTL text), boundary values (0, -1, MAX_INT), rapid clicks, double-submits.
  - **Error recovery**: What happens when something fails mid-flow? Can the user recover without losing data?

## 2.2 Form Validation

- Test every form: required field validation, type validation, min/max lengths, email/phone format validation, error message display and positioning, field masking where appropriate.
- Verify double-submit prevention on all forms (disable button after click, debounce, or server-side idempotency).
- Verify form state is preserved when validation fails — the user should never have to re-enter everything.

## 2.3 Navigation & Routing

- Test all navigation: links, routing, browser back/forward buttons, deep-linking to every route, 404 handling, redirect logic after auth.
- Verify protected routes enforce authentication. Unauthenticated users must be redirected to login.
- Verify role-based access is enforced on the frontend — users should never see UI for actions they can't perform.
- Test what happens when a user manually types a URL they shouldn't have access to.

## 2.4 API Testing

- Hit every API endpoint with valid requests and verify: correct status codes, correct response shapes, data accuracy.
- Hit every endpoint with invalid input: missing fields, wrong types, oversized payloads, malformed JSON. Verify graceful error responses with correct codes and messages.
- Test all business logic edge cases: boundary values, race conditions, concurrent writes, duplicate submissions.
- Verify pagination, sorting, and filtering work correctly on all list endpoints. Test with 0 results, 1 result, and many results.

## 2.5 Database Verification

- Verify all migrations run cleanly from scratch on a blank database.
- Check that indexes exist for frequently queried and filtered fields.
- Verify foreign key constraints, unique constraints, and cascade deletes are set up correctly.
- Look for N+1 query problems and optimize where found.
- Verify soft-delete logic (if any) doesn't leak deleted data in queries.

## 2.6 Integration Testing

- Test the full stack together: frontend → API → database → response → UI update for every major workflow.
- Test websocket/real-time features (if any): normal operation, reconnection after disconnect, stale data handling.
- Test file uploads end-to-end: size limits, type validation, storage, retrieval, display. Verify rejection of disallowed file types.
- Verify third-party integrations (payment, email, analytics, OAuth) are functional or gracefully degraded with proper error handling.
- Test what happens when the backend is slow (simulate 5s delay) or returns 500 errors — frontend should show appropriate feedback, never crash or hang indefinitely.

---

---

# PHASE 3 — SECURITY HARDENING

---

## 3.1 Authentication & Credential Security

### Password Handling
- Verify passwords are hashed with bcrypt (cost ≥12), scrypt, or Argon2id. If using anything weaker, migrate immediately.
- Confirm unique, cryptographically random salts per password.
- Enforce minimum 10-character passwords with complexity checks. Screen against a breached-password list if feasible.

### Password Reset
- Tokens must be single-use, cryptographically random (min 128-bit entropy), expire within 15-30 minutes.
- Reset links must not leak tokens in referrer headers. Old tokens must be invalidated when a new one is issued.

### Sessions & Tokens
- **JWT (if used):** Sign with RS256 or ES256. Validate `alg` header server-side, explicitly reject `none`. Short expiry (15-30 min access tokens). Implement refresh token rotation with family detection.
- **Session cookies (if used):** Set `HttpOnly`, `Secure`, `SameSite=Strict` (or `Lax` with justification). Regenerate session ID after login. Implement absolute + idle timeouts.
- Verify logout destroys the session/token server-side, not just client-side.
- Implement account lockout or exponential backoff after 5 failed login attempts (per-account AND per-IP).

### MFA
- If MFA exists, verify TOTP parameters and that backup codes are hashed and single-use.
- If MFA doesn't exist, add a flag/stub for near-term implementation.

## 3.2 Authorization & Access Control

- Audit **every API endpoint**. For each, verify authentication is required (unless explicitly public) and authorization enforces the correct role/permission.
- **Test IDOR explicitly:** Take a valid request from User A and replay it with User B's token. It must fail for every endpoint that returns or modifies user-specific data.
- Check for privilege escalation: Can a regular user hit admin endpoints? Modify their own role? Access other users' data through crafted filter parameters?
- Verify authorization logic is centralized (middleware/decorator), not scattered per-handler.
- Audit every API response — ensure no endpoint returns: password hashes, internal IDs that should be opaque, other users' PII, stack traces, DB schema details, internal URLs, or debug info.

## 3.3 Injection Prevention

### SQL Injection
- Search the entire codebase for raw SQL or string interpolation in queries. Every query must use parameterized statements or a properly configured ORM.
- Test with: `' OR 1=1--`, `'; DROP TABLE users;--`, `UNION SELECT` variants.
- Verify the app's database user follows least-privilege (no `DROP`, `ALTER`, `GRANT` in production).

### XSS
- Verify all user-generated content is escaped/sanitized before rendering (server-side and client-side).
- Search for `dangerouslySetInnerHTML` / `v-html` / `[innerHTML]` with user-controlled data. If found, ensure it passes through DOMPurify with a whitelist-only policy.
- Test with: `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`, `javascript:alert(1)`, event handler injection.

### CSRF
- Verify CSRF protection on all state-changing endpoints if using cookie-based auth.
- If using bearer tokens in Authorization header only, document why CSRF protection is unnecessary.

### Other Vectors
- **Command injection:** Search for `exec`, `spawn`, `system`, `popen`, `subprocess` with user input. Replace with safe alternatives or strict whitelist validation.
- **Path traversal:** Validate/sanitize any endpoint that reads/writes files from user input. Test with `../../etc/passwd`.
- **SSRF:** Any endpoint fetching user-provided URLs must validate against a domain allowlist and block internal network ranges (127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.169.254).
- **XXE:** Disable external entity processing in any XML parser.
- **ReDoS:** Audit all regex patterns on user input for catastrophic backtracking.

## 3.4 Data Protection

### Encryption in Transit
- HTTPS-only. HTTP must 301 redirect. HSTS header with `max-age` ≥ 1 year, `includeSubDomains`.
- TLS 1.2 minimum (prefer 1.3). No SSLv3/TLS 1.0/1.1.
- Database connections must use TLS/SSL.

### Encryption at Rest
- Encrypt PII, financial data, and authentication secrets at rest (AES-256-GCM or equivalent). Keys stored separately from data.

### PII Handling
- Audit every database table. Identify all PII. Verify each field has a justified reason for collection and appropriate protection.
- Verify sensitive data is not logged — search all logging calls for PII, tokens, passwords, request bodies.
- Verify sensitive data is not cached in CDN or browser (appropriate `Cache-Control` headers).
- Verify data retention: user deletion on account closure, cleanup of expired tokens/sessions/OTPs.

## 3.5 HTTP Security Headers

Set and verify on every response:

- `Content-Security-Policy` — strict, no `unsafe-inline` for scripts (use nonces), no `unsafe-eval`
- `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY` (or `SAMEORIGIN` with justification)
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy` — disable unused browser features (camera, mic, geolocation, payment)
- Remove `X-Powered-By`, `Server`, and any stack-revealing headers.

### CORS
- `Access-Control-Allow-Origin`: specific origins only, never `*` with credentials.
- Restrict `Allow-Methods` and `Allow-Headers` to what's actually needed.

## 3.6 Rate Limiting & Abuse Prevention

- Login: 5 attempts per 15 min per account + per IP
- Signup: 3 per hour per IP
- Password reset: 3 per hour per account
- General API: appropriate per-user and per-IP limits
- Return `429 Too Many Requests` with `Retry-After` header.
- Enforce request size limits on all endpoints (body, file upload, JSON depth, array length).
- Enforce pagination on all list endpoints — no unbounded queries.

## 3.7 Infrastructure Security

- Production debug modes OFF (no `DEBUG=True`, no Express detailed errors, no source maps served to client).
- Admin panels, debug routes, docs endpoints (`/swagger`, `/graphql/playground`, `/admin`, `/__pycache__`, `/.git`) must be removed or locked behind auth in production.
- Database ports, Redis, message queues, and internal services must not be publicly accessible.
- File uploads: validate by magic bytes (not just extension), enforce size limits, store outside webroot, serve with `Content-Disposition: attachment`, use random filenames.
- Verify no sensitive files are publicly accessible: `.env`, `.git`, `docker-compose.yml`, backups, SQL dumps.

## 3.8 Security Logging

Log with timestamp, user ID, IP, and user-agent:
- Login success/failure
- Password changes/resets
- Authorization failures (403s)
- Suspicious input validation failures
- Rate limit hits
- Account lockouts
- Admin/privileged actions

Verify logs are tamper-resistant and contain zero sensitive data (no PII, tokens, passwords, session IDs).

---

---

# PHASE 4 — LIGHT MODE DESIGN REFINEMENT

> Only modify light mode styles. Do not touch dark mode.

---

## 4.1 Background & Surface Hierarchy

- Establish a clear 3-level background system:
  - **Level 0 (page):** Subtle off-white (e.g., `#FAFAFA`, `#FAF9F7`) — never pure `#FFFFFF` everywhere.
  - **Level 1 (cards, panels):** Pure or near-pure white for elevation.
  - **Level 2 (modals, dropdowns):** White with refined shadow.
- Remove any grey backgrounds that feel dull or muddy.

## 4.2 Text & Color Audit

- **Primary text:** Rich near-black (`#1A1A1A`, `#111827`) — never pure `#000000`.
- **Secondary text:** Clearly readable mid-grey (`#6B7280`, `#64748B`) meeting WCAG AA (4.5:1 minimum).
- **Tertiary/placeholder:** Lighter but never invisible.
- Verify the accent color works on light backgrounds — it may need to be slightly deeper/muted than the dark mode variant.
- Audit semantic colors (success, warning, error): should be slightly deeper for light backgrounds. Alert backgrounds should use very subtle tints.

## 4.3 Typography

- Verify heading/body font pairing feels premium and intentional.
- Body text: weight 400+, line-height 1.5–1.7. Headings: tighter line-height (1.1–1.3), consider slight negative letter-spacing for premium feel.
- Ensure font-size scale is consistent (ratio-based, no arbitrary sizes).

## 4.4 Shadows, Borders & Depth

- Replace harsh shadows with refined, layered shadows:
  - Cards: `0 1px 3px rgba(0,0,0,0.04), 0 1px 2px rgba(0,0,0,0.06)`
  - Dropdowns: `0 4px 6px -1px rgba(0,0,0,0.05), 0 2px 4px -2px rgba(0,0,0,0.05)`
  - Modals: `0 10px 25px -5px rgba(0,0,0,0.08), 0 8px 10px -6px rgba(0,0,0,0.04)`
- Replace any hard border colors with softer `rgba(0,0,0,0.06)` to `rgba(0,0,0,0.10)`.
- Prefer whitespace and background shifts over borders to separate sections.

## 4.5 Component Polish (Light Mode)

### Buttons
- Primary: confident, solid, with subtle shadow lift. Smooth hover transition (150ms ease).
- Secondary/ghost: clearly interactive but non-competing. Subtle border or light fill.
- Verify hover, focus, active, and disabled states exist on ALL interactive elements.
- Consistent border-radius across all buttons and elements (pick one: 6px, 8px, or rounded).

### Inputs
- Clear resting state (subtle border), focus state (accent ring, smooth transition), and error state (red border + message).
- Focus rings: accent-colored, visible but not chunky.

### Cards
- Clean, airy: sufficient padding, consistent radius, subtle elevation.
- Proper content hierarchy: title → body → actions visually distinct without extra decoration.

### Tables & Lists
- Subtle row hover (`rgba(0,0,0,0.02)`). Very subtle alternate striping if used.
- Table headers distinct through weight/background, not heavy borders.

### Navigation
- Active item clearly indicated with accent tint or indicator + smooth transition.
- Sidebar background slightly different from main content to create natural boundary.

### Modals
- Semi-transparent backdrop (`rgba(0,0,0,0.3)` to `rgba(0,0,0,0.5)`).
- Subtle enter/exit animation (scale + fade, 150-200ms).

## 4.6 States & Loading

- Verify every async operation has a loading state: skeleton loaders or shimmer (not blank screens or hanging spinners).
- Verify empty states have proper UI with helpful messaging.
- Verify error states show user-friendly messages — no raw error dumps.
- Verify favicons, page titles, and meta tags are correct on every route.

---

---

# PHASE 5 — DARK MODE DESIGN REFINEMENT

> Only modify dark mode styles. Do not touch light mode.

---

## 5.1 Surface Hierarchy

Establish 4-5 surface levels using subtle lightness shifts. Never use pure `#000000` as a base:

- **Level -1 (deepest):** `#09090B` to `#0C0C0E`
- **Level 0 (page background):** `#111113` to `#141416`
- **Level 1 (cards, sidebar):** `#1A1A1E` to `#1C1C20`
- **Level 2 (dropdowns, hover states):** `#222228` to `#27272A`
- **Level 3 (modals, command palettes):** `#2A2A30` to `#2C2C32`

Total range should be subtle (~`#0A` to `#2C`). Consider a barely perceptible color tint in the greys (cool blue or warm neutral) to avoid lifeless grey.

## 5.2 Text & Color Tuning

- **Primary text:** Off-white, never pure `#FFFFFF` (causes eye strain). Use `#ECECEF`, `#E4E4E7`.
- **Secondary text:** Mid-grey around `#9898A0`, `#A0A0A8` — must meet WCAG AA contrast.
- **Tertiary:** Around `#5E5E6A`, `#6B6B76`.
- **Accent colors:** Reduce saturation 10-20% from light mode. Colors pop dramatically on dark — what's muted in light becomes neon in dark. Dial back until the accent feels rich, not electric.
- Limit accent usage aggressively. On dark, every color spot draws the eye.
- Semantic status backgrounds: very low-opacity tints (`rgba(color, 0.08)`) not solid panels.

## 5.3 Borders & Edges

- Dark mode borders should be `rgba(255,255,255,0.06)` to `rgba(255,255,255,0.10)`. Avoid hard grey borders like `#333`, `#444`.
- Use borders even more sparingly than light mode — surface-level shifts do most of the work.
- Consider `inset` box-shadows (`inset 0 0 0 1px rgba(255,255,255,0.05)`) on cards for a premium edge treatment.

## 5.4 Shadows & Glow

- Shadows need to be darker and more spread on dark backgrounds:
  - Cards: `0 2px 8px rgba(0,0,0,0.3), 0 1px 3px rgba(0,0,0,0.4)`
  - Dropdowns: `0 8px 24px rgba(0,0,0,0.4), 0 2px 8px rgba(0,0,0,0.3)`
  - Modals: `0 16px 48px rgba(0,0,0,0.5), 0 4px 16px rgba(0,0,0,0.4)`
- **Glow (sparingly):** Subtle colored glow behind accent elements (focus rings, primary buttons on hover, active nav). One or two glowing elements per viewport maximum.

## 5.5 Component Polish (Dark Mode)

### Buttons
- Primary: solid accent, slightly reduced brightness vs light mode. Subtle glow on hover.
- Secondary: `rgba(255,255,255,0.08)` background + semi-transparent border. Hover: `rgba(255,255,255,0.12)`.
- Ghost: hover background `rgba(255,255,255,0.05)`.

### Inputs
- Background slightly darker than surface (recessed well effect). Focus: accent border + subtle glow ring.
- Placeholder: `rgba(255,255,255,0.3)` range.

### Cards
- One surface level above background. Consider subtle top-edge highlight (`border-top: 1px solid rgba(255,255,255,0.05)`) for dimensionality.

### Tables
- Row hover: `rgba(255,255,255,0.03)` to `rgba(255,255,255,0.05)`. Striping barely perceptible.
- Selected rows: `rgba(accent, 0.08)` background.

### Navigation
- Active item: accent text/icon + `rgba(accent, 0.10)` background.
- Hover: `rgba(255,255,255,0.05)` shift.

### Modals
- Heavier backdrop than light mode: `rgba(0,0,0,0.6)` to `rgba(0,0,0,0.75)`.
- Consider `backdrop-filter: blur(8px)` if performance allows.

### Scrollbars
- Style to match: transparent track, `rgba(255,255,255,0.12)` thumb, thin (6-8px).

## 5.6 Imagery & Data Visualization

- Chart backgrounds: transparent or matching surface. Grid lines: `rgba(255,255,255,0.06)`.
- Chart series colors: reduce saturation 10-15% from light mode.
- Verify logos have dark-mode variants. Bright images should have subtle borders/containers.

## 5.7 Token Consolidation

- Walk every page in dark mode. Fix any surface, text, border, or accent inconsistencies.
- Ensure all dark mode values are in CSS variables / theme tokens — no hardcoded values.

---

---

# PHASE 6 — ANIMATIONS, MICRO-INTERACTIONS & POLISH

---

## 6.1 Purposeful Motion

Add transitions and micro-interactions where they improve UX. Every animation must serve a purpose: feedback, orientation, or reducing perceived latency. Remove anything gratuitous.

- **Page/route transitions:** Fast fade or slide (150-300ms).
- **Button feedback:** Subtle scale (`0.98`) or color shift on press. Smooth hover (150ms ease).
- **Skeleton loaders / shimmer:** For all async content. Style appropriately for both light and dark mode.
- **Smooth scroll behavior** across the app.
- **Toast/notification animations:** Enter and exit with slide + fade.
- **Modal/dropdown transitions:** Scale + fade (from `scale(0.97) opacity(0)` → `scale(1) opacity(1)`, 150-200ms).
- **List stagger animations** where appropriate (dashboard cards, search results).
- **Focus-visible outlines:** Accent-colored ring, keyboard-only.

## 6.2 Mobile Responsiveness & Cross-Device QA

### Breakpoint Verification
- Test every page and component at these breakpoints: **320px** (small phones), **375px** (standard phones), **390px** (modern iPhones), **428px** (large phones), **768px** (tablets portrait), **1024px** (tablets landscape / small laptops), **1280px** (laptops), **1440px** (desktops), **1920px** (large screens).
- At every breakpoint, verify: nothing overflows horizontally (no horizontal scrollbar on the page), no content is cut off or unreachable, no overlapping elements, no text truncated without indication (ellipsis or expand).
- Resize the browser continuously from 320px to 1920px — layouts should transition smoothly with no "jump" or breakage at any point, not just at named breakpoints.

### Layout & Navigation (Mobile)
- Verify the navigation collapses properly on mobile: hamburger menu, slide-out drawer, or bottom tab bar — whichever pattern the app uses. Test open/close, backdrop click-to-dismiss, scroll behavior while nav is open.
- Verify the mobile nav includes all routes accessible from the desktop nav — nothing should be missing or hidden.
- If using a sidebar on desktop, verify it either collapses to a drawer on mobile or is replaced by an appropriate mobile pattern. It must not simply shrink and become unusable.
- Verify fixed/sticky elements (headers, CTAs, floating buttons) don't overlap content or each other on small screens. Test with the keyboard open on mobile — fixed elements should not cover the active input.
- Verify modals and drawers are full-screen or near-full-screen on mobile (not a tiny centered box on a phone). Ensure they are scrollable if content exceeds viewport.
- Verify dropdown menus and popovers reposition or transform into bottom sheets/drawers on mobile so they don't overflow off-screen.

### Touch & Interaction
- All interactive elements (buttons, links, form fields, toggles, tabs, cards) must have a minimum touch target of **44×44px** on mobile. If the visual element is smaller, expand the tappable area with padding.
- Verify adequate spacing between adjacent touch targets — users should not accidentally tap the wrong element. Minimum 8px gap between tappable items.
- Remove or replace hover-dependent interactions on mobile. Any information or action only accessible via hover (tooltips, hover menus, hover previews) must have a tap-equivalent on touch devices.
- Verify swipe gestures (if any) work correctly: carousel swipes, swipe-to-delete, pull-to-refresh. They must not conflict with browser back-swipe or scroll.
- Verify long-press doesn't trigger unintended browser context menus on interactive custom elements.
- Test form input on mobile: verify keyboard type matches input (numeric keyboard for phone/zip, email keyboard for email fields, URL keyboard for URLs). Verify `autocomplete` attributes are set correctly for autofill.
- Verify input focus scrolls the field into view above the keyboard — no field should be hidden behind the mobile keyboard.

### Typography & Readability (Mobile)
- Body text must be at least **16px** on mobile to prevent iOS auto-zoom on input focus and ensure readability.
- Verify line length doesn't exceed ~80 characters on mobile — text should have appropriate horizontal margins (minimum 16px on each side).
- Verify headings scale down appropriately on mobile and don't break into awkward single-word lines.
- Verify text isn't relying on font weight alone for hierarchy — on small screens, size differences matter more.

### Images, Media & Tables (Mobile)
- All images must be responsive (`max-width: 100%`, `height: auto`) and must not overflow their containers.
- Verify proper `srcset` / responsive images are served — don't send 2000px images to a 375px screen.
- Images should be lazy-loaded, especially on mobile where bandwidth may be limited.
- Data tables must have a mobile strategy: horizontal scroll with sticky first column, card-based layout, or collapsible rows. A table that simply shrinks to unreadable size is not acceptable.
- Verify charts and visualizations are legible on mobile. If they're not, provide a simplified mobile version or allow pinch-to-zoom.

### Mobile-Specific UI Patterns
- Verify bottom sheets (if used) have proper drag handles, snap points, and dismiss behavior.
- Verify toast notifications on mobile are properly positioned (typically top or bottom, not center where they block content) and dismissible.
- Verify loading states (skeletons, spinners) are appropriately sized for mobile — don't show desktop-width skeletons on a phone.
- Verify empty states have mobile-appropriate sizing and messaging.
- Verify pagination or infinite scroll works correctly on mobile — test rapid scrolling, reaching the end, and "load more" triggers.

### Viewport & Meta Tags
- Verify `<meta name="viewport" content="width=device-width, initial-scale=1">` is set. No `maximum-scale=1` or `user-scalable=no` unless absolutely required by the app (these harm accessibility).
- Verify `theme-color` meta tag is set for both light and dark mode to match the app's header/nav color.
- Verify the app works correctly in mobile browsers' safe areas (notch, home indicator) — use `env(safe-area-inset-*)` padding where content might be obscured.
- Test in both portrait and landscape orientations. The app doesn't need to be optimized for landscape on phone, but it must not break — no overlapping elements, no lost content, no stuck states.

### Performance (Mobile-Specific)
- Verify no layout shift (CLS) during load — content should not jump around as images, fonts, and async content load. Reserve space for dynamic content.
- Verify first meaningful paint is fast on simulated 3G/4G: no massive blocking resources, critical CSS is inlined or loaded first.
- Verify touch response is immediate — no perceptible delay between tap and visual feedback. If using `click` events, ensure no 300ms delay (modern browsers handle this, but verify).
- Verify animations are performant on mobile — use `transform` and `opacity` only for animated properties (not `width`, `height`, `top`, `left` which trigger layout recalculation).
- Verify `prefers-reduced-motion` is respected: when set, disable or simplify all non-essential animations.

### PWA & Mobile Browser Behavior (If Applicable)
- If the app is a PWA: verify the manifest is correct (name, icons, theme color, start URL, display mode), the service worker caches critical assets for offline, and the install prompt works.
- If not a PWA: verify the app handles offline/poor connectivity gracefully — show a clear offline state, don't silently fail.
- Verify the app handles mobile browser chrome correctly: address bar show/hide doesn't cause layout jumps, `100vh` issues are handled (use `100dvh` or JavaScript fallback for mobile viewport height).

## 6.3 Performance Polish

- Check bundle size. Code-split where possible. Remove unnecessarily large dependencies.
- Images: compressed, proper formats (WebP where supported), lazy-loaded.
- Caching: correct headers for static assets and API responses.
- Verify no render-blocking resources delay initial paint.

---

---

# PHASE 7 — ARCHITECTURE & ENGINEERING PRACTICES REVIEW

---

## 7.1 Code Architecture

- Verify clear separation of concerns: routing, business logic, data access, and presentation are not tangled.
- Check that shared logic is properly abstracted — no copy-paste duplication of business rules across files.
- Verify error handling is consistent: centralized error middleware/handler, consistent error response format, no unhandled promise rejections or swallowed exceptions.
- Verify all async operations have proper error handling, timeouts, and cleanup.

## 7.2 API Design

- Verify consistent API conventions: URL structure, HTTP method usage, status codes, request/response shapes, error formats.
- Verify API versioning strategy is in place (URL prefix, header, or documented approach).
- Verify pagination follows a consistent pattern across all list endpoints.
- Verify proper use of HTTP methods (no state changes on GET, no data creation on DELETE).

## 7.3 Database Architecture

- Verify schema design follows normalization best practices (or has justified denormalization).
- Verify migration strategy: migrations are sequential, reversible, and version-controlled.
- Verify connection pooling is configured appropriately.
- Verify database backup strategy is documented and tested.

## 7.4 Observability

- Verify structured logging is in place (JSON format, consistent fields, request IDs for tracing).
- Verify error monitoring/alerting is configured (Sentry, Datadog, or equivalent).
- Verify health check endpoints exist for load balancers and monitoring.
- Verify critical user flows have appropriate metrics/instrumentation.

## 7.5 Type Safety & Validation

- Verify type safety is enforced (TypeScript strict mode, Python type hints, or equivalent).
- Verify runtime validation at API boundaries (Zod, Pydantic, Joi, or equivalent).
- Verify database query types match schema types.

---

---

# PHASE 8 — CI/CD & DEPLOYMENT READINESS

---

## 8.1 CI Pipeline

If CI doesn't exist, set it up. If it exists, verify and harden:

- **Linting:** Runs on every PR. Zero warnings policy (or explicitly documented exceptions).
- **Type checking:** Full type check on every PR (if using TypeScript/typed language).
- **Tests:** Unit tests, integration tests, and any E2E tests run on every PR. Failing tests block merge.
- **Security scanning:** Dependency audit (`npm audit`, `pip audit`) runs automatically. Fail on critical/high vulnerabilities.
- **Build verification:** Production build succeeds with zero warnings on every PR.
- **Branch protection:** Main/production branch requires PR review + passing CI. No direct pushes.

## 8.2 CD Pipeline

- Verify deployment is automated from the main branch (or a release branch) to staging, and from a tag/approval to production.
- Verify zero-downtime deployment strategy (rolling deploy, blue-green, or canary).
- Verify database migrations run automatically as part of deploy (with rollback capability).
- Verify environment-specific configuration is injected at deploy time, not baked into builds.
- Verify deployment rollback procedure exists and is documented — how to revert to the previous version in under 5 minutes.

## 8.3 Environment Strategy

- Verify at minimum two environments: staging (mirrors production) and production.
- Verify staging uses production-like data and configuration (with sanitized PII).
- Verify environment parity: same OS, runtime version, database version across environments.

## 8.4 Docker & Containerization (If Applicable)

- Verify Dockerfiles use trusted, pinned base images (specific digests, not `latest`). Prefer minimal images (Alpine, distroless).
- Verify multi-stage builds: build dependencies are not in the production image.
- Verify no secrets in Docker images or layers (check `docker history`).
- Run a container vulnerability scan.
- Verify `.dockerignore` excludes `.env`, `.git`, `node_modules`, and build artifacts.

## 8.5 Post-Deploy Verification

- Verify smoke tests run automatically after every production deploy (hit key endpoints, verify responses).
- Verify monitoring dashboards exist for: error rate, response latency (p50, p95, p99), database query performance, and resource utilization.
- Verify alerting is configured for: error rate spikes, latency degradation, service downtime, disk/memory thresholds.

## 8.6 Documentation

- Verify a `README.md` exists with: project setup instructions, environment variable reference, deployment process, and architecture overview.
- Verify runbooks exist for: deployment, rollback, database migration issues, and incident response.
- Verify API documentation is accurate and up-to-date (auto-generated from code where possible).

---

---

# PHASE 9 — WORLD-CLASS ENGINEERING STANDARDS

> This phase covers the practices that separate production-grade software from elite-tier software — the things Meta, Stripe, Linear, and Vercel ship with by default.

---

## 9.1 Accessibility (WCAG 2.1 AA Compliance)

This is non-negotiable for any modern software company. Accessibility failures are legal liability, lost users, and bad engineering.

### Semantic HTML & Structure
- Verify the entire app uses semantic HTML: `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<header>`, `<footer>`, `<button>`, `<a>` — not `<div>` and `<span>` for everything.
- Every page must have exactly one `<h1>`. Heading hierarchy must be sequential (`h1` → `h2` → `h3`, never skipping levels).
- Verify all interactive elements are actual `<button>` or `<a>` tags, not `<div onClick>`. If a custom component must be used, verify it has `role`, `tabIndex`, and keyboard event handlers.
- Verify landmark regions are present: at minimum `<main>`, `<nav>`, and if the page has multiple nav/section areas, they have `aria-label` to distinguish them.

### Keyboard Navigation
- Tab through every page end-to-end. Every interactive element must be reachable via Tab/Shift+Tab. Tab order must follow visual order — no jumping around.
- Verify focus is visible on every focusable element (buttons, links, inputs, tabs, cards). The focus ring must be clearly visible in both light and dark mode.
- Verify all custom components (dropdowns, modals, date pickers, carousels, tabs, accordions, command palettes) are fully keyboard-operable: arrow keys for navigation within the component, Escape to close/dismiss, Enter/Space to activate.
- Verify focus trapping in modals and drawers — Tab should cycle within the modal, not escape to content behind it. On close, focus must return to the trigger element.
- Verify skip-to-content link exists (visually hidden, appears on Tab) that jumps past the navigation to `<main>`.

### Screen Reader Compatibility
- Verify all images have meaningful `alt` text (or `alt=""` for purely decorative images). Icons used as buttons must have `aria-label`.
- Verify all form inputs have associated `<label>` elements (via `htmlFor`/`for` attribute, not just visual proximity). If labels are visually hidden, use `sr-only` class, not `display: none`.
- Verify dynamic content updates (toasts, live search results, real-time data) use `aria-live` regions (`polite` for non-urgent, `assertive` for critical) so screen readers announce changes.
- Verify all custom components have correct ARIA roles, states, and properties: `aria-expanded` on toggles/dropdowns, `aria-selected` on tabs, `aria-checked` on custom checkboxes, `aria-haspopup` on menus, `aria-current="page"` on active nav items.
- Verify loading states are announced: use `aria-busy="true"` on the loading container, or an `aria-live` region that announces "Loading..." and "Content loaded."
- Verify error messages on form fields are linked to their input via `aria-describedby`.

### Color & Contrast
- All text must meet WCAG AA contrast ratios: 4.5:1 for normal text, 3:1 for large text (18px+ or 14px bold+).
- UI components and graphical objects (icons, borders, focus rings) must meet 3:1 contrast against adjacent colors.
- Verify information is never conveyed by color alone. Error states must have icons or text in addition to red color. Status badges must have labels, not just color dots. Charts must be distinguishable without color (use patterns, labels, or shapes).

### Motion & Vestibular
- Verify `prefers-reduced-motion` media query is implemented. When active: disable parallax, auto-playing animations, transition effects, and any motion that isn't user-initiated.
- No content should flash more than 3 times per second (seizure risk).

## 9.2 Internationalization (i18n) & Localization Readiness

Even if launching in one language, build the foundation now. Retrofitting i18n is 10x harder later.

### Text & Translation Infrastructure
- Verify no user-facing strings are hardcoded in components. All text should go through a translation function (`t()`, `useTranslation()`, `intl.formatMessage()`, or equivalent), even if only English translations exist today.
- If i18n is not yet set up, install and configure a library (react-intl, next-intl, i18next, or equivalent) and extract all strings into a single locale file as the source of truth.
- Verify no string concatenation for sentences (`"Hello " + name + ", welcome"`) — use interpolation with named placeholders (`t('greeting', { name })`) so translators can reorder words.
- Verify no hardcoded pluralization (`count + " items"`) — use ICU MessageFormat or the i18n library's plural rules.

### Layout Resilience
- Verify UI does not break when text is 30-40% longer (German, French, Finnish expand significantly vs English). Test by temporarily adding padding/filler to all strings.
- Verify no fixed-width containers that would clip or overflow with longer translations. Buttons, labels, tabs, and nav items must flex or wrap.
- Verify text truncation (`text-overflow: ellipsis`) is used where appropriate for bounded containers, with full text available via tooltip or expansion.
- Verify the app does not break with RTL (right-to-left) text direction if Arabic, Hebrew, or Urdu are potential future locales. At minimum, no layouts should be impossible to flip. Use logical CSS properties (`margin-inline-start` vs `margin-left`) where possible.

### Date, Time, Number, and Currency
- Verify all dates and times are formatted using `Intl.DateTimeFormat` or equivalent locale-aware formatter — never hardcoded `MM/DD/YYYY`.
- Verify all numbers use `Intl.NumberFormat` for locale-appropriate thousands separators and decimal marks.
- Verify currency display uses `Intl.NumberFormat` with `style: 'currency'` — never hardcode `$` or currency symbols.
- Verify timezone handling is explicit: store all timestamps in UTC, display in the user's local timezone (or let the user choose). Never assume a timezone.

## 9.3 SEO & Social Sharing (If Publicly Accessible)

### Technical SEO
- Verify every page has a unique, descriptive `<title>` tag (50-60 characters).
- Verify every page has a unique `<meta name="description">` tag (150-160 characters).
- Verify a canonical URL (`<link rel="canonical">`) is set on every page to prevent duplicate content.
- Verify proper heading hierarchy (`h1` → `h2` → `h3`) on every page (overlaps with accessibility — both benefit).
- Verify clean, human-readable URLs (not `/page?id=a8f3b2`). Use slugs where appropriate.
- Verify a `robots.txt` exists and is correct — allows indexing of public pages, blocks private/admin areas.
- Verify an XML sitemap exists and is auto-generated, submitted to search engines, and updates when content changes.
- Verify server-side rendering (SSR) or static generation (SSG) is used for all public-facing pages that need to be indexed. Client-only rendered pages are invisible to most search engines.
- Verify proper `rel="nofollow"` on user-generated links and external links where appropriate.

### Open Graph & Social Cards
- Verify Open Graph tags are set on every shareable page: `og:title`, `og:description`, `og:image` (1200×630px minimum), `og:url`, `og:type`.
- Verify Twitter Card tags are set: `twitter:card` (summary_large_image), `twitter:title`, `twitter:description`, `twitter:image`.
- Test by pasting a URL into Twitter, LinkedIn, Slack, iMessage, and WhatsApp. The preview should render correctly with the correct image, title, and description.
- Verify `og:image` is an absolute URL, not a relative path.

### Structured Data
- Add JSON-LD structured data where applicable (Organization, Product, FAQ, Article, BreadcrumbList). Validate with Google's Rich Results Test.

## 9.4 Error Boundaries & Resilience

### Frontend Error Boundaries
- Verify React Error Boundaries (or framework equivalent) are in place at multiple levels: a top-level boundary that prevents full-app crashes, and granular boundaries around independent features so one broken widget doesn't take down the page.
- Error boundaries should render a helpful fallback UI ("Something went wrong. Please refresh." with a retry button), not a blank screen or raw stack trace.
- Verify error boundaries log the error to the monitoring service (Sentry, etc.) with component stack and user context.

### Graceful Degradation
- Test every third-party integration failure in isolation. When Stripe is down, the rest of the app must work. When analytics fails to load, nothing should break. When the CDN is slow, the app should still render.
- Verify all `fetch`/API calls have timeouts (don't hang forever), retry logic for transient failures (with exponential backoff and jitter), and user-facing fallback states.
- Verify the app handles API responses with unexpected shapes (missing fields, extra fields, null where an object is expected) without crashing. Use runtime validation (Zod, etc.) and fallback to defaults.
- Verify the app handles browser APIs that may be unavailable (service workers, notifications, geolocation, clipboard) with proper feature detection — never assume a capability exists.

### Stale Data & Optimistic Updates
- Verify data freshness strategy: how does the UI handle data that changed while the user was viewing it? Implement stale-while-revalidate patterns, background refetching, or real-time subscriptions where appropriate.
- If using optimistic updates (UI updates before server confirms), verify rollback behavior when the server rejects the change — the UI must revert cleanly with a user-visible notification.

## 9.5 Feature Flags & Controlled Rollout

- If a feature flag system is not in place, set one up (even a simple JSON config or environment variable-based approach). No feature should require a code deploy to enable/disable.
- Verify any experimental or partially-built features are behind flags and OFF by default in production.
- Verify feature flags are cleaned up — no flags for features that have been fully rolled out for 30+ days. Dead flags are dead code.
- Verify the app handles the "flag off" state gracefully: no broken UI, no empty containers, no orphaned routes.

## 9.6 Analytics, Instrumentation & Product Intelligence

### Event Tracking
- Verify analytics is integrated (Mixpanel, Amplitude, PostHog, GA4, or equivalent) and the SDK loads correctly without blocking rendering.
- Verify key events are tracked at minimum:
  - **Acquisition:** Page views, signup started, signup completed, referral source
  - **Activation:** Onboarding steps completed, first key action taken
  - **Engagement:** Feature usage (every major feature), session duration, return frequency
  - **Retention:** Login frequency, feature re-engagement
  - **Revenue (if applicable):** Checkout started, payment completed, plan upgraded/downgraded
- Verify all events include relevant properties: user ID (anonymized if pre-auth), timestamp, platform (web/mobile), viewport size, and feature-specific context.
- Verify no PII is sent to analytics (no emails, names, passwords, or tokens in event properties).

### Error & Performance Tracking
- Verify frontend errors are captured with stack traces and user context (Sentry, Datadog RUM, or equivalent).
- Verify Core Web Vitals are monitored: LCP (Largest Contentful Paint), FID/INP (Interaction to Next Paint), CLS (Cumulative Layout Shift). Set targets: LCP < 2.5s, INP < 200ms, CLS < 0.1.
- Verify API latency is tracked per-endpoint with percentiles (p50, p95, p99).

### Consent & Privacy
- If analytics collects user data, verify a cookie consent banner is implemented where legally required (EU, UK, California at minimum).
- Verify analytics respects user opt-out preferences — when consent is declined or withdrawn, no tracking events should fire and any stored identifiers should be cleared.
- Verify compliance with applicable privacy regulations: GDPR (EU users), CCPA (California), and any Pakistan-specific data protection requirements.

## 9.7 Legal, Compliance & Trust

- Verify a Privacy Policy exists, is accessible from every page (typically footer), and accurately describes data collection, usage, sharing, and retention practices.
- Verify Terms of Service / Terms of Use exist and are accessible.
- Verify a cookie policy exists if cookies are used for anything beyond essential session management.
- Verify data deletion capability: users must be able to request deletion of their account and associated data (GDPR "right to be forgotten"). Test the deletion flow end-to-end — verify data is actually removed from the database, not just soft-deleted with ongoing access.
- Verify all user-facing legal text was reviewed (or flagged for legal review before launch).

## 9.8 Edge Cases & Chaos Engineering

### Connectivity & Network
- Test the app with throttled connections (Slow 3G, Offline). Verify graceful behavior: loading indicators, cached data where available, clear offline messaging, queued actions that sync on reconnect.
- Test what happens when a user starts an action on wifi, loses connectivity mid-request, and reconnects. The app should recover without data loss or corruption.
- Verify API request deduplication — if the same request is fired twice (network retry, double-click), the backend handles it idempotently.

### Concurrent Users & Race Conditions
- If two users edit the same resource simultaneously, what happens? Verify either optimistic concurrency control (version/ETag checks), last-write-wins with notification, or real-time collaboration — but never silent data loss.
- Verify database transactions are used for multi-step operations that must be atomic. A failure mid-operation must roll back completely, not leave partial data.

### Time & Timezone Edge Cases
- Verify the app handles timezone changes correctly (user travels, DST transitions).
- Verify date-boundary logic: what happens at midnight UTC vs midnight local time? Test scheduled actions, expiry logic, and "today" calculations.
- Verify the app handles clock skew between client and server (client time 5+ minutes off).

### Account & Session Edge Cases
- Test concurrent sessions: user logged in on multiple devices/tabs. Session invalidation on one must not corrupt state on others.
- Test session expiry during active use: the user is filling a form when their token expires. The app must not lose their work — either silently refresh the token or save their state and redirect to login with a return path.
- Test account deletion while logged in on another device.
- Test what happens when a user's role/permissions change while they're actively using the app.

### Data Edge Cases
- Test with very large datasets: 10,000+ items in a list, very long text content, deeply nested structures. Verify the UI virtualizes or paginates — never renders 10,000 DOM nodes.
- Test with completely empty accounts (new user, no data). Every screen should have a helpful empty state, not a blank void or an error.
- Test with unicode edge cases: emoji in all text fields, RTL characters, zero-width characters, extremely long single words (no spaces), and mixed-script text.
- Verify the app handles special email formats: `+` tags (`user+test@email.com`), subdomains, and new TLDs.

## 9.9 Open Source & License Compliance

- Verify all third-party dependencies have compatible licenses for your use case (commercial, SaaS, etc.).
- Check for any GPL/AGPL licensed dependencies that would impose copyleft requirements on your codebase. Flag or replace if needed.
- Verify attribution requirements are met for any MIT/Apache/BSD dependencies that require notices (typically in an ACKNOWLEDGMENTS file or about page).
- If using any fonts, icons, or assets, verify the license permits commercial use and is properly attributed.

---

---

# EXECUTION RULES

---

1. **Fix as you go.** Don't just list problems — fix every issue immediately. If a fix requires a judgment call, state reasoning briefly and proceed with the best option.
2. **Test after every fix.** Verify the fix works and doesn't break anything else.
3. **Be surgical.** Don't refactor architecture for its own sake. The goal is stability, security, polish, and deployability — not a rewrite.
4. **Be paranoid on security.** When in doubt, choose the more restrictive option.
5. **Respect design boundaries.** Light mode changes don't touch dark mode. Dark mode changes don't touch light mode.
6. **Log everything.** Produce a final report at the end with:
   - Summary of issues found and fixed, organized by phase
   - Security controls now in place (headers, rate limits, encryption, auth hardening)
   - Design changes made (palette, shadows, animations, component states)
   - Accessibility audit results and fixes
   - i18n readiness status
   - Analytics and instrumentation coverage
   - CI/CD pipeline status and configuration
   - Items requiring manual action (key rotation, DNS config, third-party settings, WAF setup, legal review)
   - Recommended post-launch practices (dependency update cadence, log review, pen testing schedule, backup verification, analytics review cadence)
