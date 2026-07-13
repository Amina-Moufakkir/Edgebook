# 0008 — Supabase as Integrated Infrastructure

- **Status:** Draft — proposed, not approved. Documents the decision; do not initialize or implement until reviewed and approved.
- **Date:** 2026-07-11
- **Deciders:** Founder
- **Related:** [0004 Authentication Strategy], [0006 Private Object Storage Strategy], [0009 Row-Level Security Strategy]

---

> This ADR documents our **initial** infrastructure decision, not a permanent commitment. It records why we are starting with an integrated provider so the reasoning is explicit before it becomes code, and so a future separation can be evaluated against the original intent.

## Context

Edgebook AI is a modular monolith operated by a single founder. It handles private, security-sensitive data — trades, journal entries, screenshots, AI conversations — and must enforce strict per-user isolation from day one.

The early product needs four foundational capabilities:

- managed PostgreSQL (the source of truth)
- authentication
- private object storage
- a coherent authorization model that ties them together

Assembling these from separate best-of-breed providers is possible, but it multiplies the security-sensitive integration surface: identity mapping, storage authorization, and database access each become their own wiring problem. For a solo founder, that glue is exactly where subtle cross-user isolation bugs are most likely to appear.

## Decision

Adopt **Supabase** as the initial integrated infrastructure provider for managed PostgreSQL, authentication, and private object storage.

Core reasons:

1. Supabase gives Postgres, Auth, Storage, and Row-Level Security a single coherent identity and authorization model — the authenticated user's ID flows consistently through database policies and storage access.
2. It minimizes security-sensitive integration work for a solo founder.
3. It supports the modular-monolith and managed-infrastructure strategy without adding services to operate.
4. PostgreSQL remains the portable source of truth.

This decision is deliberately an **umbrella**. The specific authentication, storage, and RLS strategies are refined in their own ADRs (0004, 0006, 0009).

### What this decision explicitly preserves

- **PostgreSQL remains the source of truth.** Supabase is how we host and access Postgres, not a replacement for it.
- **The application server remains the primary authorization boundary.** Every private operation is authorized server-side.
- **Row-Level Security is defense in depth, not our only security mechanism.** A missing server-side check is a bug even if RLS would have caught it.
- **We own the domain model, SQL migrations, business logic, and provider interfaces.** These live in the repository, not solely in the provider dashboard.
- **Provider-specific logic stays behind application-owned adapters.** Feature code talks to our `auth`, `db`, and `storage` interfaces, never directly to vendor SDKs scattered across the codebase.

### Binding constraints

- RLS is enabled with explicit policies on every private table.
- Every private resource has automated cross-user isolation tests.
- The browser never contains service-role credentials.
- Only private storage buckets are used; upload and download access is authorized and time-limited.
- File ownership and metadata are stored in PostgreSQL.
- Core business logic and financial calculations do not live in RLS policies or provider-specific database functions.

## Alternatives Considered

### A. Neon + Clerk + Cloudflare R2

- **Neon** (serverless Postgres) + **Clerk** (auth) + **Cloudflare R2** (object storage).
- Pros: strong individual products; excellent auth DX; no storage egress fees; lower per-component lock-in.
- Cons: three vendors and billing relationships; identity must be mapped from Clerk into Postgres and into storage authorization by hand; RLS must be wired to an externally issued JWT; more integration surface for a solo founder.

### B. Separate managed PostgreSQL + Auth + S3-compatible storage

- e.g. RDS/Cloud SQL Postgres, a standalone auth service, and S3.
- Pros: maximum control and portability; no bundled-platform lock-in.
- Cons: the most integration and operational burden; the founder must glue identity, storage authorization, and database policy together — the work most prone to isolation defects.

Supabase is chosen because its bundled, coherent identity/authorization model removes the highest-risk integration work while keeping Postgres portable.

## Consequences

**Positive**

- One identity model spans auth, database (RLS), and storage.
- Fast path to a secure baseline with less custom glue code.
- Managed backups, connection pooling, and dashboards reduce operational load.

**Negative / risks**

- Moderate platform lock-in for Auth and Storage (much less for Postgres).
- Dependency on Supabase availability and pricing.
- Temptation to push logic into database functions/policies — explicitly forbidden by the constraints above.
- Connection pooling and cold-start behavior must be validated against real access patterns.

## Security Implications

- The service-role key is a full-access credential; it must live only on the server and never reach the browser or client bundles.
- RLS is a critical control surface — a missing or wrong policy is a cross-user data leak. Automated isolation tests are mandatory (see 0009).
- Storage buckets default to private; any public exposure is a deliberate, reviewed exception (see 0006).
- Auth token handling, session lifetime, and MFA posture are defined in 0004.
- Because RLS is defense in depth, server-side ownership checks must exist independently and be tested independently.

## Portability Strategy

- Postgres schema and migrations are owned in-repo, so the database can be lifted to any Postgres host.
- All Supabase access goes through application-owned adapter interfaces (`auth`, `db`, `storage`), so replacing a component means reimplementing an adapter — not editing feature code.
- Storage object keys and ownership metadata are recorded in Postgres, so files can be re-pointed to another object store.
- Auth is treated as an identity provider behind an adapter; the application owns authorization, so a provider swap does not change permission logic.

## Review Triggers

Revisit this decision if:

- product scale, operational complexity, or provider limitations justify separating infrastructure components;
- monthly infrastructure cost crosses a defined threshold, or Supabase pricing changes materially;
- connection limits, pooling, or latency become a measured bottleneck;
- multi-region or data-residency requirements emerge;
- a security limitation in Supabase Auth, Storage, or RLS blocks a required control;
- the product moves beyond single-tenant-per-account in a way the bundled model does not serve.
