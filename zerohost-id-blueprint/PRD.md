# PRD.md — ZeroHost.ID

**Product:** ZeroHost.ID  
**Domain:** `zerohost.biz.id`  
**Document status:** Product requirements baseline  
**Primary deployment:** Vercel  
**Primary database:** Firebase Realtime Database (RTDB)

## 1. Product vision
ZeroHost.ID is a professional Indonesian storefront for hosting and digital services. Customers can discover products, pay, manage subscriptions, and access their services. Administrators can operate products, orders, payments, SEO, and multiple independent Pterodactyl panels from one control center.

The product must be built as a Vercel-compatible Next.js application with Firebase Authentication and Firebase RTDB. Pterodactyl panels remain independently hosted and must not be reinstalled or altered by the store.

## 2. Goals
1. Sell configurable hosting and digital products through a responsive storefront.
2. Provide one customer account for the store.
3. Provision Pterodactyl resources across multiple panels selected manually per product by an administrator.
4. Manage subscriptions, renewals, expiration, suspension, and post-grace deletion safely.
5. Provide a comprehensive admin-managed SEO toolkit for public pages and products.
6. Integrate payment and notification providers through replaceable adapters.
7. Deploy previews and production through Vercel with secure environment separation.
8. Use Firebase RTDB as the sole primary application database.

## 3. Non-goals and constraints
- Replacing or migrating existing Pterodactyl panel installations.
- Running privileged Pterodactyl API calls from browser code.
- Using Supabase, Firestore, or PostgreSQL as the primary database.
- Guaranteeing Google rankings or automatic SEO outcomes.
- Enabling PixPayz without official API documentation and credentials.
- Selling VPS resources hosted on a trial platform whose terms prohibit resale or the intended workload.
- Assuming Vercel Functions are persistent workers or unlimited-duration background processes.

## 4. Users and roles
### Visitor
- Browse products, categories, articles, pricing, and public pages.
- Use search and contact forms.
- View SEO-friendly public URLs.

### Customer
- Register/login and manage profile.
- Purchase products and view invoices/payment status.
- View active services, expiry, credentials/instructions, and renewal options.
- View notifications and support links.
- Access service management links appropriate to the assigned panel.

### Administrator
- Manage catalog, pricing, stock/capacity, orders, customers, coupons, and content.
- Configure payment and notification providers.
- Add, test, disable, and manage multiple Pterodactyl panel connections.
- Assign each product/plan to a specific panel.
- Manage panel resources and view servers/customers across panels.
- Manage subscription lifecycle and review destructive actions.
- Manage all SEO settings, redirects, sitemap, robots, structured data, and audits.
- View audit logs, job history, integration health, and operational alerts.

### Operator/support (optional role)
- View customer/service records and perform limited support actions.
- No access to secrets or unrestricted destructive operations.

## 5. Functional requirements

### 5.1 Storefront and catalog
- Responsive landing page with ZeroHost.ID branding.
- Product categories: Pterodactyl game servers, VPS, digital services, and configurable future categories.
- Product detail pages with configurable pricing, billing period, specifications, FAQ, and SEO fields.
- Search, category filtering, featured products, and clear purchase flow.
- Configurable slugs and public content pages.
- Indonesian currency/locale presentation; currency and tax rules must be configurable.

### 5.2 Authentication and customer area
- Firebase Authentication for customer identity.
- Server-side verification of Firebase ID tokens.
- Role-based authorization for admin/operator/customer.
- Customer profile and order/service history.
- One store identity may map to separate Pterodactyl user identities on each panel.
- Customer data isolation must be enforced in server code and RTDB Security Rules.

### 5.3 Admin dashboard
- Overview of revenue, orders, active services, expiring services, panel health, failed jobs, and payment status.
- CRUD for products, categories, pricing, billing cycles, coupons, and content.
- Customer and order management.
- Integration settings with masked secrets and connection tests.
- Audit trail for sensitive actions.
- Search, filters, pagination, loading/empty/error states, and responsive layout.

### 5.4 Multi-Pterodactyl manager
- Register multiple panel endpoints with label, URL, status, version/compatibility metadata, and encrypted API credential reference.
- Test connection and show last health check.
- Discover/manage supported panel resources: locations, nodes, nests/eggs, and allocations.
- Show capacity and resource availability when the API exposes it.
- Map each sellable product/plan to exactly one configured panel and required egg/location/resource preset.
- Admin manually chooses the panel per product. No automatic failover to another panel unless a future explicit policy is configured.
- Provision customer user/server after verified payment.
- Create a separate Pterodactyl account per panel as needed. Generate independent random credentials for each panel; securely deliver initial access and provide reset flow. Never store or log plaintext passwords.
- Display servers across all connected panels with panel identity, customer, status, limits, and expiry.
- Support documented operations: suspend, unsuspend, reinstall only if explicitly approved as a product feature, and delete. Existing panel infrastructure must not be changed.
- Destructive actions require explicit confirmation, authorization, eligibility validation, idempotency, and audit log.
- Handle panel outages and rate limits with visible retryable job status.

### 5.5 Orders, payments, and billing
- Order lifecycle: pending → awaiting payment → paid → provisioning → active; include failed/cancelled/refunded states.
- Midtrans integration using official documentation and sandbox-first verification.
- PixPayz adapter interface and admin configuration placeholder; disabled until official docs/credentials are supplied.
- Verify webhook signatures and event authenticity server-side.
- Idempotent webhook processing and payment reconciliation.
- Never mark an order paid based only on client-side redirect/query parameters.
- Support renewals and configurable billing periods.
- Store transaction references and safe payment metadata; never store card secrets.

### 5.6 Subscription lifecycle and scheduled work
- Track `startsAt`, `expiresAt`, `suspendedAt`, `graceEndsAt`, and lifecycle status.
- At expiry, enqueue suspend operation and notify customer.
- Keep service suspended during a 7-day grace period.
- After grace, mark eligible for deletion; require policy checks and an auditable deletion job.
- Jobs must be retry-safe and idempotent.
- Vercel Cron may trigger bounded scans or enqueue work. Long-running work must execute through a durable external queue/worker or managed task service compatible with Vercel.
- Provide admin retry, pause, and inspect controls for failed jobs.

### 5.7 Notifications
- In-app notification center.
- Telegram bot adapter.
- WhatsApp provider adapter (provider and official API credentials configurable).
- Templates for payment, provisioning, renewal, expiry, suspension, deletion warning, and support events.
- Track delivery status and provider errors without exposing tokens.

### 5.8 Full SEO toolkit
- Global SEO defaults and per-page overrides.
- Product/category SEO title, description, slug, canonical URL, robots directives, Open Graph, and social preview.
- XML sitemap generation, robots.txt editor/generator, and sitemap index support if needed.
- Redirect manager with 301/302, validation, conflict checks, and loop prevention.
- Structured data generator/editor for Organization, Product, Breadcrumb, Article, and FAQ.
- Image alt text and social image metadata.
- Technical SEO audit: missing/duplicate title and descriptions, heading hierarchy, broken internal links, canonical issues, noindex/indexability, missing alt text, and sitemap coverage.
- Google Search Console verification/configuration and Analytics ID configuration.
- Blog/content manager with publish status, author, category, slug, canonical, and article schema (release can be phased).
- Public pages are indexable only when published and intended for search.
- Customer dashboard, authentication, checkout, invoices, admin, API, and private service pages must be `noindex`.
- Show audit findings and recommended fixes; do not claim guaranteed rankings.

### 5.9 VPS and other digital services
- Provider-agnostic VPS provisioning interface.
- Provider must explicitly permit commercial hosting/resale and the intended workload.
- Do not use Railway Trial for commercial VPS resale, proxy, or VPN offerings.
- Where no approved provider is configured, keep VPS fulfillment manual or mark product unavailable for automated provisioning.

## 6. Vercel deployment and runtime requirements
- Next.js App Router project must deploy using Vercel Git integration.
- Support local development, Vercel Preview, and Vercel Production environments.
- Use environment variables for Firebase client config, Firebase Admin credentials, provider secrets, webhook secrets, cron secret, and public site URL.
- Keep Admin SDK and privileged integrations in Node.js server routes/actions; do not use Edge Runtime for these modules.
- No persistent local filesystem assumptions or process-local queues.
- Functions must be bounded and return within configured platform limits.
- Use Vercel Cron for short scheduled triggers only; authenticate cron requests and make runs idempotent.
- Use an external durable worker/queue for slow or high-volume provisioning, billing, and reconciliation.
- Include deployment instructions, domain setup for `zerohost.biz.id`, preview environment guidance, build checks, and rollback instructions.
- Build, lint, typecheck, and tests must run in CI before production deployment.

## 7. Firebase RTDB data model (logical)
Use a documented, version-controlled path layout. Suggested top-level paths:
- `/users/{uid}`
- `/roles/{uid}`
- `/products/{productId}`
- `/categories/{categoryId}`
- `/orders/{orderId}`
- `/payments/{paymentId}`
- `/subscriptions/{subscriptionId}`
- `/panels/{panelId}` (public metadata only; never plaintext secrets)
- `/panelAccounts/{panelId}/{uid}`
- `/servers/{serverId}`
- `/productPanelConfigs/{productId}`
- `/notifications/{uid}/{notificationId}`
- `/provisioningJobs/{jobId}`
- `/webhookEvents/{provider}/{eventId}`
- `/seo/global`
- `/seo/pages/{pageKey}`
- `/seo/redirects/{redirectId}`
- `/content/{contentId}`
- `/auditLogs/{logId}`
- `/system/integrationHealth/{integrationId}`

Sensitive encrypted credentials should be stored outside ordinary client-readable RTDB nodes, or in a server-only encrypted secret store with references in RTDB. Define indexes and `.indexOn` rules where required. Security Rules must deny by default and permit only explicitly authorized access. Never trust client-supplied role claims without server-side verification.

## 8. UX and design requirements
- Professional, modern, fast, trustworthy hosting brand.
- Responsive desktop/tablet/mobile layouts.
- Consistent navigation, sidebar, tables, forms, dialogs, toasts, and status indicators.
- Clear service lifecycle labels and timestamps.
- Confirmation dialogs for destructive actions.
- Accessible contrast, keyboard navigation, semantic labels, and focus states.
- Useful skeleton/loading, empty, validation, and error states.
- Avoid fake data in production. Demo seed data must be clearly marked.

## 9. Non-functional requirements
- Security: least privilege, secret isolation, webhook verification, rate limiting, CSRF/origin protections where applicable, SSRF protection for panel URLs, and audit logs.
- Reliability: idempotent jobs, bounded retries, timeouts, backoff, health checks, and reconciliation.
- Performance: server rendering/static generation for public content where appropriate; avoid unnecessary RTDB reads.
- Maintainability: typed provider interfaces, modular architecture, schema validation, tests, and documented RTDB paths.
- Observability: structured logs, correlation IDs, job status, integration health, and safe error messages.
- Privacy: collect minimum personal data and define retention/export/deletion procedures.

## 10. Acceptance criteria
1. A clean checkout can install dependencies and pass lint, typecheck, tests, and `npm run build`.
2. The app can be deployed to Vercel without a custom server or unsupported persistent process.
3. Firebase Authentication and RTDB work using environment-specific Firebase projects.
4. RTDB Security Rules prevent a customer from reading/writing another customer's private records.
5. Admin can add multiple Pterodactyl panels, test connections, and assign a product to a chosen panel.
6. Privileged panel API calls happen only server-side.
7. A paid sandbox order can be provisioned idempotently to the configured panel.
8. Expiry triggers suspension; deletion is not eligible until the 7-day grace period has elapsed and safeguards pass.
9. Invalid/replayed payment webhooks cannot create duplicate paid orders or provisioning.
10. SEO admin can edit metadata, redirects, sitemap/robots configuration, structured data, and run an audit.
11. Private pages are noindex; published public product/content pages emit valid metadata and canonical URLs.
12. Telegram, WhatsApp, and in-app notification adapters expose delivery status; unavailable providers are visibly disabled.
13. PixPayz remains disabled until official documentation and credentials are configured.
14. No secret is present in source control, browser bundles, client logs, or public RTDB paths.

## 11. Delivery milestones
- M0: repository audit, architecture, Vercel build baseline, Firebase project wiring.
- M1: design system, storefront, Firebase Auth, customer/admin RBAC.
- M2: RTDB repositories, catalog, orders, customer portal.
- M3: multi-panel manager and read-only panel discovery/health.
- M4: payment adapters (Midtrans sandbox first), webhook verification, order state machine.
- M5: provisioning jobs, subscription lifecycle, 7-day grace, notifications.
- M6: SEO Manager and content publishing.
- M7: VPS provider adapter, operational hardening, backups, monitoring, production readiness.

Each milestone must include tests, documentation, and a Vercel Preview deployment before production promotion.
