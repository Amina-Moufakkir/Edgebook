# 0009 — Row-Level Security Strategy

- **Status:** Draft — proposed, not approved. Documents the decision; do not implement until reviewed and approved.
- **Date:** 2026-07-11
- **Deciders:** Founder
- **Related:** [0008 Supabase Integrated Infrastructure], [0004 Authentication Strategy], [0006 Private Object Storage Strategy]

---

> This ADR explains *why* Edgebook AI adopts Row-Level Security and what architectural boundary it creates. It is not implementation documentation; it defines reasoning and constraints before any policy is written.

## Context

Edgebook AI stores private, security-sensitive data — trades, executions, journal entries, screenshots, reviews, and AI conversations. Every one of these records belongs to a single user, and the product's credibility depends on the guarantee that no user can ever see another user's data.

That guarantee cannot rest on a single layer of code. Application authorization is where isolation is *intended* to happen, but application code is also where mistakes happen: a forgotten `where user_id = …`, a new endpoint that skips a shared guard, a background job that trusts its payload, a refactor that quietly widens a query. In a financial, privacy-sensitive product operated by a solo founder, one such mistake is a cross-user data leak.

Row-Level Security (RLS) lets the database itself enforce that a query can only ever return or modify rows belonging to the authenticated user — independently of whether the application remembered to scope the query. Adopting Supabase (0008) makes RLS a first-class, natively supported control tied to the same identity that authentication (0004) establishes.

## Decision Principles

- **Isolation is too important to depend on one layer.** The database must be able to protect users even when application code is wrong.
- **Explicit over implicit.** Access rules are declared, not inferred from defaults.
- **Least privilege by default.** A table exposes nothing until a policy grants a specific, scoped path to it.
- **Enforcement, not behavior.** The database enforces *who may touch a row*; it never decides *what the product does* with it.
- **Verified, not trusted.** Every isolation guarantee is backed by an automated test, not by belief that the policy is correct.
- **Defense in depth over perfect code.** Assume application mistakes will eventually happen, and design the system so they do not become data breaches.

### Default Deny

- Every private table begins with **no access** — nothing can be read or modified until access is deliberately granted.
- Access is granted **only** through explicit, reviewed policies.
- The **absence** of a policy always results in denied access, never accidental exposure. A table that someone forgot to write a policy for fails closed, not open.

## Decision

Adopt **PostgreSQL Row-Level Security as a mandatory defense-in-depth layer** on every private table, enforcing per-user isolation at the database level beneath the application's own authorization.

### Three distinct layers

Edgebook AI's access model has three layers, each with a separate job:

1. **Authentication (proves identity).** Supabase Auth (0004) establishes *who* is making the request via a stable user UUID (`auth.uid()`).
2. **Application authorization (determines permission).** The application server decides *whether* that identity may perform the requested action on the requested resource — including business rules, ownership-chain checks, and rate limits. This is the **primary** authorization mechanism.
3. **Row-Level Security (database-level enforcement).** RLS independently guarantees that any query executed on behalf of a user can only reach that user's rows. This is **defense in depth**, not the primary mechanism.

### Why RLS is defense in depth, not the primary mechanism

- RLS answers only one question — *does this row belong to the current user?* It cannot express business rules, workflow state, or multi-step authorization, and it must not try to.
- The application server must authorize every private **operation** — read, insert, update, or delete — **before** it reaches the database. RLS then independently verifies that only authorized rows may be accessed or modified. The two act on every operation, not just queries: a read returns nothing it shouldn't, and an insert, update, or delete cannot touch a row the caller does not own.
- RLS is the net that catches an operation the application should have scoped but didn't — not a substitute for scoping it.
- Treating RLS as primary would push authorization logic into the database (see constraints below), couple the product to Postgres, and blur the boundary between access enforcement and business behavior.

RLS exists precisely for the case where application logic fails: a missing filter, a bad refactor, an unguarded new route. When that happens, RLS turns a would-be data breach into a returned-empty-set or a denied write.

## Ownership Model

Every private record belongs to an authenticated user. Ownership is expressed two ways:

- **Direct ownership.** The record carries the owner's identity itself (e.g. a trading account owned by a user).
- **Ownership chains.** Most records inherit ownership through a parent relationship:

  ```text
  Authenticated User
      → Trading Account
          → Trade
              → Execution / Review / Screenshot / Note
  ```

  A screenshot belongs to a user because it belongs to a trade, which belongs to an account, which belongs to that user.

RLS policies enforce this isolation at the database level: a policy scopes each private table to rows the current identity owns, directly or by following the chain to an owning record it already controls. Conceptually, the deeper a record sits in the chain, the more its policy relies on the integrity of the relationships above it — which is why the application must also verify the complete ownership chain, and why foreign keys and constraints matter as much as policies. (This ADR describes the model conceptually; concrete policy definitions belong to implementation.)

## Architectural Constraints

- **RLS enabled on every private table.** No private table is ever exposed without it.
- **Policies are explicit.** Access is granted by named, reviewed policies — never left to defaults or implicit table grants.
- **Service-role credentials never reach the browser.** The service role **intentionally bypasses RLS**, so it is a full-access credential and must live only in trusted server-side infrastructure.
- **Service-role access requires stronger controls than ordinary requests.** Because it bypasses the database's isolation layer entirely, its use demands stronger operational controls than a normal application request — confined to specific trusted infrastructure paths (migrations, system jobs), justified case by case, and never used to serve user-facing requests.
- **No business logic in the database.** Policies must not encode financial rules, workflow state, or application behavior.
- **Calculations and rules stay in application code.** P&L, analytics, and domain rules are deterministic application logic (per the architecture's source-of-truth hierarchy), never database policy.
- **Policies enforce access, not behavior.** If a rule decides *what the product does* rather than *which rows a user may touch*, it does not belong in RLS.

## Testing Requirements

Isolation is a security assumption, and every security assumption is verified by tests rather than trusted.

- **Every private table has automated cross-user isolation tests.**
- **Tests verify both allowed and denied access** — a user can reach their own rows, and provably cannot reach another user's.
- **Tests exercise the ownership chain**, not just top-level tables, so inherited ownership is validated at every depth.

Examples of the isolation failures these tests exist to prevent (described, not implemented):

- User B reading User A's trade by guessing or iterating an identifier.
- A newly added endpoint or query that forgets to scope by user and returns cross-user rows.
- A nested record (screenshot, execution, review) reachable even though its parent trade belongs to another user.
- An update or delete that mutates a row the caller does not own.
- A background job that processes another user's data because it trusted its input payload.
- A service-role code path that unintentionally handles a user-facing request and bypasses isolation.

If a policy is added, changed, or removed, its isolation tests are updated in the same change.

## Alternatives Considered

### A. Server-side authorization without RLS

- Rely solely on application code to scope every query.
- Weaker: a single missed filter or unguarded route is an immediate cross-user leak, with no second layer to contain it. This is the *primary* mechanism we already use — the point of RLS is to not stand alone on it.

### B. Database views or a custom permission system

- Expose per-user views, or build a bespoke row-permission table and enforce it in queries.
- Weaker and heavier: views multiply as the schema grows and still depend on the application selecting the right one; a custom permission system reinvents, less rigorously, what RLS provides natively — and becomes its own source of bugs.

### C. No database-level authorization

- Trust application logic entirely; the database enforces nothing.
- Weakest: zero defense in depth. Any application mistake, misconfigured client, or leaked non-service credential can read across users.

RLS is chosen because it provides genuine, database-enforced defense in depth with the least custom machinery, natively supported by the infrastructure we are adopting.

## Consequences

**Positive**

- A second, independent isolation layer that holds even when application code is wrong.
- Isolation guarantees are declarative and reviewable at the table level.
- Aligns with least privilege: tables expose nothing until an explicit policy grants scoped access.

**Negative / risks**

- Every new private table adds a policy-authoring and test obligation; skipping it is a security gap, so process discipline matters.
- The service role bypasses RLS — mishandling it defeats the layer, so its protection is critical.
- Policies can create hard-to-diagnose "empty result" behavior when application intent and policy disagree; this must be understood as the layer working, not a bug to loosen.
- RLS is Postgres-specific and part of the infrastructure boundary.

## Security Implications

- **Cross-user data leakage** is the primary threat RLS mitigates; it is the difference between an application bug and a breach.
- **Defense in depth:** authorization and isolation are independent layers, and neither is allowed to depend on the other to fail safely.
- **Least privilege:** default-deny at the row level; access is granted only by explicit policy.
- **Policy review:** every policy change is security-relevant and reviewed as such.
- **Policy testing:** isolation is proven by allowed/denied tests, not assumed correct.
- **Administrative access:** admin paths are explicit, audited, and never a blanket bypass of isolation for ordinary operations; service-role use is not a substitute for authorization.
- **Service-role protection:** the RLS-bypassing credential is confined to trusted server infrastructure and never shipped to the client.
- **Policy change control:** policy changes are treated as security-sensitive infrastructure changes and reviewed with the same rigor as application authorization changes.
- **Security audits:** RLS coverage (every private table enabled, every policy tested) is a standing item in periodic security review.

## Portability Strategy

- **Authorization logic belongs to the application** and is provider-neutral; it does not move if the database does.
- **RLS policies are infrastructure-specific.** They live in the migration/infrastructure layer, not in domain code.
- **Database isolation is treated as an infrastructure concern**, owned by the infrastructure layer rather than any product domain.
- **Business domains remain unaware of how isolation is implemented.** Feature code relies on the `db` adapter and application authorization; it does not know whether isolation is enforced by RLS or by some future mechanism.
- If PostgreSQL or the infrastructure provider changes, **only the infrastructure layer should require significant modification** — policies are re-expressed for the new platform while application authorization is untouched.

## Review Triggers

Revisit this decision if:

- the product adopts multi-user organizations/tenancy, requiring richer ownership than per-user rows;
- RLS becomes a measured performance bottleneck for specific access patterns;
- the underlying database or infrastructure provider (0008) changes;
- policy management at scale becomes error-prone enough to warrant additional tooling or a different enforcement model;
- a security review reveals a class of access RLS cannot adequately express;
- application authorization and RLS responsibilities begin to overlap or duplicate one another.

## Guiding Principle

> Authorization belongs to the application.
>
> Isolation belongs to the database.
>
> Each layer should remain independently trustworthy.
>
> Neither is permitted to rely on the other to fail safely.
