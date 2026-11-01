# PROMPT.md — Implementation Brief for Coding Agent

You are the lead full-stack engineer building **ZeroHost.ID**. Follow `AGENT.md` and `PRD.md` as binding requirements. Do not start by generating a generic template; inspect the existing repository, identify its framework and current state, then implement in small verified milestones.

## Project outcome
Deliver a professional Indonesian hosting/digital-services store at `zerohost.biz.id`, built with Next.js App Router + TypeScript, Firebase Authentication, Firebase Realtime Database, and deployable to Vercel. It must include a customer portal, admin dashboard, multi-Pterodactyl manager, billing/provisioning foundation, notification adapters, and a full SEO management toolkit.

## Mandatory first actions
1. Inspect the repository, package manager, existing source, git status, and current deployment configuration.
2. Read `AGENT.md`, `PRD.md`, and this file completely.
3. Identify reusable code and dependencies; do not overwrite working project code without understanding it.
4. Produce an implementation plan with milestones, risks, RTDB data paths, authorization boundaries, and Vercel runtime decisions.
5. Confirm the project builds before making broad changes. Record baseline errors.
6. If the selected base is a Firebase + Next.js starter, audit its Firebase setup, authentication, RTDB access, rules, and Vercel configuration. Retain useful pieces, remove insecure or irrelevant examples, and do not assume its current state is production-ready.

## Technical baseline
- Next.js App Router, TypeScript strict mode.
- Vercel deployment compatibility is mandatory.
- Firebase Authentication for identity.
- Firebase Realtime Database as the primary database.
- Firebase Admin SDK for trusted server-side operations only.
- Zod (or equivalent) for validation.
- Typed provider adapters for Pterodactyl, payments, notifications, and VPS.
- Use the repository's established package manager and lockfile.
- Do not introduce Supabase, Firestore, PostgreSQL, or another primary database.

## Required application areas
### Public storefront
- Home, product catalog, category pages, product detail pages, pricing, FAQ, blog/content pages, contact/support.
- Search-friendly slugs and server-rendered metadata.
- Product types include Pterodactyl server plans, VPS, digital services, and extensible future products.

### Customer portal
- Firebase Auth login/register and profile.
- Orders, invoices/payment state, active services, expiry, renewal, notifications, and service access details.
- One store account may map to distinct Pterodactyl user accounts on separate panels.

### Admin dashboard
- Overview and operational metrics.
- Products/categories/pricing/coupons/orders/customers/content management.
- Multi-Pterodactyl panel connections and health checks.
- Per-product manual panel assignment.
- Payment/notification settings with masked secrets.
- Subscription/job controls, audit logs, integration health.
- SEO Manager described below.

### Multi-Pterodactyl Manager
- Add/edit/disable/test multiple panel connections.
- Store panel URL, label, supported version/compatibility, and a secure credential reference.
- Discover supported nodes, locations, nests/eggs, and allocations.
- Display health, resource capacity where supported, and panel-specific errors.
- Manually map product/plan to one selected panel and resource preset.
- Provisioning must use server-side documented APIs only.
- Separate panel-specific customer accounts; generate independent credentials per panel and do not log plaintext.
- List/search servers across panels and display panel association.
- Suspend/unsuspend and delete only with authorization, explicit confirmation, eligibility checks, idempotency, and audit log.
- Do not alter existing panel installations or silently reroute a product to a different panel.

### Full SEO toolkit
Create an SEO admin area with:
- Global SEO defaults.
- Per-page, per-product, and per-category title/description/canonical/robots metadata.
- Editable slugs and canonical URL validation.
- Open Graph and social preview metadata.
- XML sitemap generation and robots.txt management.
- Redirect manager (301/302), conflict validation, and redirect-loop prevention.
- JSON-LD structured data for Organization, Product, Breadcrumb, Article, and FAQ where relevant.
- Image alt text management.
- Technical SEO audit for missing/duplicate metadata, heading hierarchy, broken internal links, canonical conflicts, noindex status, missing alt text, and sitemap coverage.
- Google Search Console verification/configuration and Analytics integration settings.
- Content/blog publishing fields and article SEO.
- `noindex` for login, customer portal, checkout, invoices, admin, APIs, and private service pages.
- Never claim guaranteed ranking improvements.

## Data and security
- Use Firebase RTDB paths and rules documented in `PRD.md`.
- Create and maintain RTDB Security Rules in the repository. Deny by default.
- Verify Firebase ID tokens server-side; centralize role/permission checks.
- Never trust client-side admin flags.
- Keep Firebase Admin credentials, Pterodactyl API keys, Midtrans server key, PixPayz secrets, Telegram tokens, WhatsApp tokens, and cron secrets out of client bundles and source control.
- Validate panel URLs and protect against SSRF.
- Verify payment webhooks using official provider signature rules and idempotency.
- Do not mark payment successful from client redirect alone.
- Do not invent PixPayz API details. Keep PixPayz disabled until official docs and credentials are available.
- Do not use Railway Trial for commercial VPS resale or prohibited proxy/VPN workloads.
- Add rate limiting, safe logs, audit trails, timeouts, and bounded retries.

## Vercel-specific implementation rules
- All web pages, route handlers, and server actions must work in Vercel's serverless model.
- Use Node.js runtime for Firebase Admin SDK and Node-dependent provider clients; do not deploy those modules to Edge Runtime.
- No long-lived process, local persistent storage, custom server, or process-memory queue.
- Use Vercel Environment Variables with separate Development, Preview, and Production values.
- Provide `.env.example` with placeholder names only.
- Use Vercel Cron only to trigger short, bounded, authenticated, idempotent tasks.
- Put long-running provisioning/reconciliation/bulk work behind a durable external queue or managed task service. Document the chosen provider and required environment variables; if no provider is selected, implement the queue interface and a safe disabled adapter rather than pretending jobs are durable.
- Add Vercel deployment instructions, domain configuration for `zerohost.biz.id`, Preview setup, cron setup, and rollback notes.
- Ensure CI runs lint, typecheck, tests, and production build.

## Design direction
- Premium, modern, trustworthy hosting brand.
- Dark navy/charcoal foundation with restrained electric-blue or violet accents; keep contrast accessible.
- Clear dashboard navigation and dense but readable data tables.
- Responsive layouts, accessible controls, keyboard support, visible focus, skeletons, empty states, validation, and error feedback.
- Avoid decorative UI that hides operational status.

## Implementation sequence
**Milestone 0 — Audit and foundation**
- Repository audit, app structure, environment validation, Firebase initialization, Vercel deployment baseline, RTDB rules, CI checks.
- Output: architecture note, `.env.example`, successful baseline build.

**Milestone 1 — UI shell and identity**
- Design system, public layout, admin/customer layouts, Firebase Auth, server-side token verification, RBAC.
- Output: working sign-in and protected route tests.

**Milestone 2 — Store and customer portal**
- RTDB repositories, catalog, product pages, customer profile, cart/checkout intent, orders, service list.
- Output: functional catalog and customer order history using RTDB.

**Milestone 3 — Multi-panel management**
- Panel registry, secure credential handling, connection health, resource discovery, product-to-panel assignment, cross-panel server list.
- Output: tested read-only discovery and admin panel assignment before destructive controls.

**Milestone 4 — Payments**
- Midtrans sandbox adapter, verified webhook, idempotent payment events, order state machine.
- Output: sandbox end-to-end payment flow.
- PixPayz stays disabled until docs and credentials arrive.

**Milestone 5 — Provisioning and lifecycle**
- Durable job interface, provisioning, panel-specific accounts, expiry, suspend, 7-day grace, deletion safeguards, notification adapters.
- Output: retry/idempotency tests and operator job console.

**Milestone 6 — SEO Manager**
- Metadata, sitemap, robots, redirects, JSON-LD, audit, Search Console/Analytics configuration, noindex rules.
- Output: inspect rendered public metadata and generated sitemap/robots.

**Milestone 7 — Hardening and production**
- Security review, RTDB rules review, backup/recovery, monitoring, accessibility, performance, Vercel production deployment checklist.

## Definition of done for every milestone
- No TypeScript errors.
- Lint and tests pass.
- `npm run build` (or the repository's equivalent) succeeds.
- No secrets committed or exposed to browser.
- New RTDB paths and rules documented.
- Error, loading, empty, and permission-denied states implemented.
- Vercel Preview deployment is verified where credentials are available.
- Report files changed, commands run, test/build outcomes, known limitations, and next milestone.

## Final instruction
Start with the repository audit and Milestone 0. Do not attempt to implement all modules in one unreviewed code dump. Ask for missing provider credentials or official API documentation only when a milestone genuinely requires them. When blocked, leave a secure, clearly disabled adapter and continue with independent work.
