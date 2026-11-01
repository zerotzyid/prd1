# AGENT.md — ZeroHost.ID

## Mission
Build **ZeroHost.ID**, a production-oriented Indonesian hosting and digital-services storefront at `zerohost.biz.id`, with customer accounts, an operator/admin dashboard, multi-Pterodactyl management, a complete SEO toolkit, billing, notifications, and automated provisioning.

Read `PRD.md` first. Follow `PROMPT.md` as the execution brief. These three documents are the product and engineering source of truth.

## Non-negotiable decisions
- **Deployment target: Vercel.** The application must be compatible with Vercel's Next.js deployment model. Use Next.js App Router and TypeScript. Do not require a permanently running local server for core web requests.
- **Database: Firebase Realtime Database (RTDB).** RTDB is the primary application database. Do not replace it with Supabase, PostgreSQL, or Firestore.
- **Authentication:** Firebase Authentication. Verify ID tokens on the server before authorizing protected operations.
- **Server-side Firebase access:** Firebase Admin SDK only. Admin SDK credentials must never enter client bundles.
- **Brand/domain:** ZeroHost.ID / `zerohost.biz.id`.
- **Language:** Indonesian-first UI; architecture should allow future localization.
- **Products:** Pterodactyl game servers, VPS, digital services, and other configurable products.
- **Multiple Pterodactyl panels:** each product/plan is manually assigned to a target panel by an administrator.
- **Panel identities:** one ZeroHost customer identity, but separate Pterodactyl user accounts on each panel where required. Generate independent credentials for each panel; never reuse passwords across panels.
- **Subscription lifecycle:** active → suspended at expiry → 7-day grace period → eligible for deletion. Deletion must be guarded, auditable, and revalidated.
- **Notifications:** in-app, Telegram, and WhatsApp through provider adapters.
- **Payments:** Midtrans and PixPayz adapters. Implement Midtrans only against its official current documentation. PixPayz must remain disabled until official API documentation and credentials are supplied. Never invent endpoints or signatures.
- **Existing Pterodactyl installations:** do not reinstall, upgrade, or modify their OS, web server, database, Wings, or panel configuration. Integrate through supported APIs only.
- **VPS provider:** do not use a free/trial platform for commercial resale unless its terms explicitly permit the intended workload and resale model. Provider integration must be replaceable.

## Vercel deployment requirements
1. Build as a standard Next.js App Router application deployable through Vercel Git integration.
2. Use Node.js runtime for Firebase Admin SDK and provider API routes. Explicitly set `runtime = "nodejs"` on routes that use Node-only libraries.
3. Do not rely on persistent local filesystem writes, in-memory state, long-running processes, custom listening ports, or background daemons.
4. Store configuration in Vercel Environment Variables. Separate Development, Preview, and Production values.
5. Use Vercel Cron only for short, bounded, idempotent scheduled tasks. Protect cron endpoints with a secret and design for retries.
6. For long-running provisioning, reconciliation, or bulk jobs, use a durable external queue/worker or a managed task service compatible with Vercel. Do not assume a Vercel Function can run indefinitely.
7. Ensure build succeeds with `npm run build`; add lint/typecheck/test scripts.
8. Never expose Firebase Admin credentials, Pterodactyl API keys, payment server keys, webhook secrets, or messaging tokens to the browser.
9. Do not use Edge Runtime for Firebase Admin SDK or integrations requiring Node APIs.
10. Document Vercel setup, environment variables, domain configuration, preview deployments, cron configuration, and rollback.

## Architecture rules
- Separate UI, domain services, repositories, and provider adapters.
- Define typed interfaces for Pterodactyl, payments, notifications, and VPS providers.
- Validate all external input with a schema library (e.g. Zod).
- Centralize authentication, authorization, and role checks.
- Use RTDB transactions or atomic multi-location updates where concurrency matters.
- Define RTDB paths and security rules as version-controlled artifacts. Deny access by default; clients may access only their own permitted data. Privileged writes happen server-side.
- Use idempotency keys for checkout, payment webhooks, provisioning, renewal, suspend, and deletion.
- Use audit logs for all administrative and destructive actions.
- Store secrets in a managed secret store or encrypted server-side configuration. Do not store plaintext credentials in ordinary RTDB records.
- Avoid logging secrets, tokens, passwords, payment payload secrets, or personal data beyond operational necessity.
- Add rate limiting and abuse controls to authentication, checkout, public forms, and admin APIs.
- Add structured logs with correlation IDs and safe provider error summaries.

## Pterodactyl integration rules
- Use documented Pterodactyl API endpoints and least-privilege keys.
- Distinguish Application API operations from Client API operations.
- Keep one panel adapter instance/configuration per panel.
- Support connection health checks, panel metadata, nodes, nests/eggs, locations, allocations, users, servers, resource limits, power/status views where supported, suspend/unsuspend, and deletion.
- Validate resource availability before provisioning.
- Product-to-panel assignment is explicit and manual by admin; never silently choose a different panel.
- Never perform a destructive operation from a page render or GET request.
- Require confirmation, permission check, eligibility check, audit event, and idempotency for deletion.
- Do not assume all panels have identical versions, features, or resource layouts; expose compatibility/health state.

## SEO requirements
Implement an admin-managed SEO system for public pages:
- Per-page title, meta description, canonical URL, robots directives, Open Graph, and social metadata.
- Product/category SEO fields and editable slugs.
- XML sitemap generation and `robots.txt`.
- Redirect manager (301/302) with loop and conflict validation.
- Structured data (JSON-LD) for Organization, Product, Breadcrumb, Article, and FAQ where appropriate.
- Image alt text and social preview metadata.
- Technical audit for missing/duplicate titles, descriptions, headings, broken internal links, canonical conflicts, and indexability.
- Search Console and Analytics integration configuration without exposing credentials.
- Blog/content publishing support if included in the release scope.
- `noindex` customer account, checkout, invoice, admin, and other private pages.
- Do not promise rankings or fabricate SEO scores. Explain audit rules and limitations.

## Security and reliability
- Threat-model authentication, privilege escalation, webhook spoofing/replay, SSRF, secret leakage, IDOR, race conditions, and destructive automation.
- Verify payment webhook signatures using provider documentation and raw request bodies as required.
- Reconcile payment state server-side; never trust a browser redirect as proof of payment.
- Validate panel URLs against SSRF: reject loopback, private, link-local, metadata, and unsupported schemes unless explicitly allowlisted for a controlled deployment.
- Use timeouts, bounded retries with backoff, and circuit-breaker/health states for external APIs.
- Encrypt sensitive provider credentials at rest; restrict decryption to server-side service code.
- Provide backups/export and documented recovery procedures for RTDB.
- Add tests for authorization boundaries, webhook verification, idempotency, expiry grace period, and panel routing.

## Required delivery quality
- Do not leave fake success flows, hardcoded secrets, mock payment success, or nonfunctional buttons in production paths.
- If credentials/docs are unavailable, ship a clearly disabled integration with setup instructions, not a guessed implementation.
- Include `.env.example` with names and descriptions only; never real secrets.
- Include RTDB schema/path documentation and Firebase Security Rules.
- Include a README with local setup, Firebase setup, Vercel deployment, environment variables, tests, and operations.
- Keep changes incremental. At the end of each milestone report files changed, commands run, test/build results, known limitations, and next milestone.
