# 0004 — Authentication Strategy

- **Status:** Draft — proposed, not approved. Documents the decision; do not implement until reviewed and approved.
- **Date:** 2026-07-11
- **Deciders:** Founder
- **Related:** [0008 Supabase Integrated Infrastructure], [0009 Row-Level Security Strategy], [0006 Private Object Storage Strategy]

---

> Authentication proves **identity**. It does not grant **permission**. Authorization is enforced by the application server. This ADR covers only how we establish and verify identity, and how that identity anchors data ownership.

## Context

Edgebook AI must identify users and manage sessions before it can enforce any per-user data isolation. Identity is foundational: the authenticated user's ID is the key that flows into database Row-Level Security (0009) and storage authorization (0006).

Because we are adopting Supabase as integrated infrastructure (0008), its authentication component is available with no additional integration and shares one identity model with the database and storage layers. The decision here is whether to use it, and under what security constraints.

## Decision

Use **Supabase Auth** as the initial authentication provider, behind an application-owned `auth` adapter. Identity issuance is delegated; authorization is not.

### Relationship between Supabase Auth and the application server

- **Supabase Auth** issues and verifies identity and sessions, and runs recovery/verification/MFA flows.
- **The application server** treats the verified identity as an *input* and makes every authorization decision itself. It re-verifies the session on each private request and never trusts a client-asserted user ID.
- The **`auth` adapter** is the only place that imports the Supabase auth SDK; feature code depends on a provider-neutral interface (`getCurrentUser`, `requireUser`, `verifySession`, recovery hooks).

Authentication answers *who is calling*. It never proves permission to access a particular trade, screenshot, account, review, or AI conversation — that is decided server-side and enforced again by RLS as defense in depth.

### How identity maps to PostgreSQL ownership

- Supabase Auth users live in the `auth.users` table and are identified by a stable UUID (`auth.uid()`).
- The application maintains its own `profiles` (or `app_users`) row in the public schema, keyed by that UUID, as the identity anchor for application data. Application code references the profile, not provider internals.
- Every private table carries an owner column (e.g. `user_id`) that references this identity, directly or through an ownership chain (account → trade → execution/note/screenshot/review).
- RLS policies (0009) scope rows to `auth.uid()`; the server also verifies the ownership chain independently. Identity is therefore the single value that ties authentication, database isolation, and storage authorization together.

## Security Requirements

**Session handling**

- Sessions are verified server-side on every private request; short-lived access tokens with refresh, and server-side revocation on sign-out and password/MFA change.
- Session lifetime and idle timeout are explicitly configured, not left at defaults.

**Cookie security**

- Session cookies are `HttpOnly`, `Secure`, and `SameSite` (Lax or Strict), scoped to the application domain, over HTTPS only.
- No access/refresh tokens are exposed to client JavaScript beyond what the provider's session model strictly requires.

**Account recovery**

- Password reset and recovery use single-use, time-limited tokens.
- Recovery invalidates existing sessions.
- Recovery attempts are rate-limited and logged as security events.

**MFA (admin-required)**

- **MFA is required for all administrative access** and strongly encouraged for regular users.
- Admin authentication without a passed MFA challenge is rejected server-side, not just hidden in the UI.
- MFA enrolment/reset changes are logged as security events.

**Rate limiting**

- Sign-in, sign-up, password reset, and MFA challenge endpoints are rate-limited per IP and per account to resist brute force and credential stuffing.

**Account-enumeration resistance**

- Sign-in, sign-up, and password-reset responses do not reveal whether an email exists (uniform messaging and timing; reset always responds "if an account exists, an email was sent").
- Error messages avoid distinguishing "unknown user" from "wrong password."

## Alternatives Considered

### A. Clerk (separate auth provider)

- Pros: excellent DX, polished pre-built UI, strong MFA/session features.
- Cons: identity lives outside Postgres and must be mapped into the database and storage authorization by hand; RLS must be wired to an externally issued JWT; adds a second vendor.

### B. Self-managed authentication (e.g. Auth.js with our own credential store)

- Pros: maximum control and portability; no auth vendor.
- Cons: we take on the full security burden of credential storage, session management, recovery, and MFA — high-risk work for a solo founder and a poor early time investment.

### C. Auth bundled with a different platform

- Deferred; only relevant if the umbrella infrastructure decision (0008) is revisited.

Supabase Auth is chosen because it shares one identity model with the database and storage we are already adopting — exactly where auth integration bugs are most costly.

## Consequences

**Positive**

- Identity, RLS, and storage authorization share one user UUID with no custom mapping.
- Managed recovery, verification, and MFA reduce security-sensitive custom code.
- Fastest path to a secure identity baseline.

**Negative / risks**

- Auth is the most lock-in-prone Supabase component; migration means re-homing user accounts.
- Session/token handling must be validated for server-rendered access patterns.
- Provider outages affect sign-in; degradation behavior must be defined.

## Security Implications

- A compromised or mis-scoped session must never grant cross-user access; RLS (0009) and server-side ownership checks are the backstops.
- The service-role key is never used to impersonate a user in request handling.
- All authentication events (sign-in, reset, recovery, MFA changes, admin access) are recorded in the security event log without secrets.
- Enumeration resistance and rate limiting are verified by tests, not assumed.

## Portability Strategy

- The `auth` adapter exposes a provider-neutral interface; feature code depends only on it.
- Application user/profile records in Postgres reference identity by a stable UUID, so application data is not owned by the provider.
- Because the application owns authorization, swapping providers changes identity issuance only, not permission logic.
- A migration path (export accounts, re-issue credentials via reset flows) is documented before this component would be replaced.

## Review Triggers

Revisit this decision if:

- Supabase Auth lacks a required capability (specific MFA method, SSO, enterprise SAML);
- authentication UX or reliability becomes a measured problem;
- the umbrella infrastructure decision (0008) is revisited;
- compliance or data-residency requirements constrain where identity may live;
- the product adds multi-user organizations/tenancy the current identity model does not serve.
