# Screen Inventory

Status: Draft
Last Reviewed: 2026-07-12

---

# Purpose

This document identifies every screen required to support the approved user flows.

It defines each screen's responsibility within the product.

It does **not** define:

- visual design
- layouts
- navigation
- components
- implementation details
- architecture

Those belong to `design-system.md` and `architecture.md`.

---

# Related Documents

This inventory derives from, and defers to, the following documents. When this document and one of them disagree, the owning document wins.

- `user-flows.md` — **what must work.** Screens are derived from flows (core `F-#` and supporting `SF-#`); this is the upstream source of truth.
- `design-principles.md` / `design-system.md` — **how screens look and are built** (layout, navigation, components, the visual form of every state).
- `architecture.md` — **how the system behind the screens works** (boundaries, data, security).
- `ai-philosophy.md` — **how the AI behaves** on AI-backed screens.
- `glossary.md` — shared vocabulary (Screen, State, Core Flow, Supporting Flow, Confirmed Capture, …).

---

# Principles

A screen exists only if it supports an approved user flow — core (`F-#`) or supporting (`SF-#`).

A screen exists because the user has a destination — not because the implementation benefits from another page.

Screens are derived from user flows — never the reverse.

Never invent screens because they "might be useful."

Every screen has one primary responsibility.

A **screen** is a routable destination. A **state** is a condition that appears *within* a screen (a confirmation, an empty result, an unavailable dependency). States are not screens and do not receive screen IDs; they are recorded in each screen's `Required States` and, when shared across screens, in [Cross-Cutting States](#cross-cutting-states).

---

# Decisions Applied

_This draft made three judgment calls that are yours to overturn. Each was re-validated in review against `architecture.md` and `design-system.md`._

1. **Pure inventory.** The inventory lists only routable screens. Confirmations and dependency-failure conditions are recorded as **states**, not screens — reclassifying the proposed **S-13 Trade Confirmation**, **S-16 Delete Trade Confirmation**, and **S-22 AI Unavailable** into [Cross-Cutting States](#cross-cutting-states). _(Confirmed by `architecture.md` Error Handling: AI failure is degradation behavior, not a destination.)_
2. **Removed S-09 Application Shell and S-10 Navigation.** `design-system.md` → Layout System owns navigation and shell dimensions; they are the frame screens render into, not destinations. See [Removed / Reclassified](#removed--reclassified).
3. **Authentication homed under Supporting Flows.** Auth and account-management screens support no *core* product flow, but the product still requires them. They now trace to **Supporting Flows** (`SF-#`) in `user-flows.md` — a category kept separate from the core coaching flows so the product's focus stays on coaching. The named flows exist; **SF-02 and SF-05 are drafted**, and SF-01, SF-03 and SF-04 are not yet. See [Cross-Document Consistency](#cross-document-consistency).

Retired IDs (S-09, S-10, S-13, S-16, S-22) are **not reused**.

---

# Screen Statuses

MVP · Planned · Future · Deprecated

---

# Standard States

To keep `Required States` at the product altitude — *which conditions a screen must handle*, not *how they look or are implemented* — screens draw from one shared vocabulary rather than inventing terms. The visual form of each state belongs to `design-system.md`; this list only names which conditions exist. This vocabulary is aligned with the browser-state language already defined in `architecture.md` (loading, empty, success, error).

**Only applicable states are documented.** No screen must implement every state; an omitted state simply does not apply.

- **Loading** — awaiting data or a server response.
- **Empty** — no data yet. Must teach the next action (`design-principles.md` → Empty States).
- **Populated** — normal state, content present.
- **Validation Error** — user input rejected before persistence; entered data is preserved (GR-2, GR-6).
- **Authorization Error** — the action or resource is not permitted or not owned (GR-1).
- **Server Error** — an unexpected failure; a safe, plain-language message (GR-2).
- **Recoverable Failure** — a dependency or partial operation failed, but the user's work is preserved and retryable (GR-6). Covers draft recovery and upload failure.
- **Offline** — network unavailable. Rarely applicable in MVP; listed only where it genuinely applies.

Screens may also host **cross-cutting product states** — named, rule-bearing states that recur across screens: Confirmed Capture, Destructive Confirmation, AI Unavailable, Insufficient Evidence. These are defined once in [Cross-Cutting States](#cross-cutting-states).

---

# Screen Template

Each screen records:

- **Status**
- **Supported Flow(s)** — the approved flow(s) this screen serves, core (`F-#`) or supporting (`SF-#`). A screen with none is orphaned (⚠).
- **Primary Actor**
- **Primary Purpose** — one responsibility.
- **Entry Points**
- **Exit Points**
- **Required States** — drawn from [Standard States](#standard-states) plus any cross-cutting states hosted. Only applicable states.
- **Security & Privacy Notes** — reference GR-#.
- **Open Questions**

_Not every screen needs every field. An omitted field must be intentional — name it and give a one-line reason. A missing field with no note reads as an oversight._

---

# Public

## S-01 — Landing Page

- **Status:** MVP
- **Supported Flow(s):** F-01 (public entry point)
- **Primary Actor:** Unauthenticated visitor
- **Primary Purpose:** Explain what the product is and route the visitor to sign up or sign in.
- **Entry Points:** Direct / organic arrival
- **Exit Points:** → S-03 Sign Up · → S-02 Sign In
- **Required States:** Populated (static — no data dependency)
- **Security & Privacy Notes:** No authentication; exposes no private data.
- **Open Questions:** Confirm marketing scope is out of MVP (this is the app's public door, not a marketing site).

## S-02 — Sign In

- **Status:** MVP
- **Supported Flow(s):** SF-01 Sign In (supporting flow — detailed journey not yet drafted)
- **Primary Actor:** Returning unauthenticated trader
- **Primary Purpose:** Authenticate a returning trader into their session.
- **Entry Points:** S-01 · deep link to a private route · expired session
- **Exit Points:** → S-08 Dashboard · → S-04 Password Recovery · → S-03 Sign Up
- **Required States:** Loading · Populated · Validation Error · Recoverable Failure (rate-limited / locked)
- **Security & Privacy Notes:** GR-1 — authentication server-side. No account enumeration in error copy. Rate limiting on attempts.
- **Open Questions:** SF-01's journey is undrafted; confirm session/MFA scope when it is defined.

## S-03 — Sign Up

- **Status:** MVP
- **Supported Flow(s):** F-01 (First-Time Onboarding — account creation step)
- **Primary Actor:** New visitor
- **Primary Purpose:** Create an account so onboarding can begin.
- **Entry Points:** S-01 · S-02
- **Exit Points:** → S-05 Welcome
- **Required States:** Loading · Populated · Validation Error
- **Security & Privacy Notes:** GR-1 — creation authorized server-side. No enumeration of existing accounts.
- **Open Questions:** Is email verification in MVP? That would add a verification-pending state.

## S-04 — Password Recovery

- **Status:** MVP
- **Supported Flow(s):** SF-02 Password Recovery (supporting flow — **journey drafted** in `user-flows.md`)
- **Primary Actor:** Locked-out user
- **Primary Purpose:** Let a user securely regain access.
- **Entry Points:** S-02
- **Exit Points:** → S-02 Sign In
- **Required States:** Loading · Populated · Validation Error · Recoverable Failure (expired / invalid token)
- **Security & Privacy Notes:** GR-1. No account enumeration. Time-limited, single-use reset tokens.
- **Open Questions:** Owned by `user-flows.md` → SF-02 Open Questions now that the journey is drafted. Still live there: whether recovery requires an MFA challenge (an unresolved gap in ADR 0004 — recovery that bypasses MFA defeats it), whether a new request invalidates outstanding tokens, and whether a password-change notification is sent. Note the **uniform-response requirement**: this screen's Validation Error and Recoverable Failure states must not distinguish invalid from expired from already-used tokens, nor reveal whether an address has an account.

---

# Onboarding

## S-05 — Welcome

- **Status:** MVP
- **Supported Flow(s):** F-01
- **Primary Actor:** Newly registered trader
- **Primary Purpose:** Orient the trader and set expectations before first setup.
- **Entry Points:** S-03 Sign Up
- **Exit Points:** → S-06 Create Trading Account
- **Required States:** Populated (informational — no data dependency)
- **Security & Privacy Notes:** Authenticated; no sensitive data shown.
- **Open Questions:** Is Welcome shown once, or re-entrant if onboarding is abandoned and resumed?

## S-06 — Create Trading Account

- **Status:** MVP
- **Supported Flow(s):** F-01 (onboarding step) · F-02 (Create a Trading Account)
- **Primary Actor:** Authenticated trader (during onboarding, or adding an account later)
- **Primary Purpose:** Create a trading account so trades can be logged against it.
- **Entry Points:** S-05 (onboarding) · S-19 Trading Accounts (later additions)
- **Exit Points:** → S-07 Onboarding Complete (onboarding path) · → S-19 (settings path)
- **Required States:** Loading · Populated · Validation Error
- **Security & Privacy Notes:** GR-1 — account creation authorized server-side and bound to the owning user.
- **Open Questions:** Whether an account may hold more than one currency — **still open**, pending ADR 0010 Open Question 2. _(The money representation itself is settled by ADR 0010, Accepted 2026-08-01; every monetary value carries an explicit currency regardless of how the multi-currency question resolves.)_

## S-07 — Onboarding Complete

- **Status:** MVP
- **Supported Flow(s):** F-01 (completion state)
- **Primary Actor:** Trader finishing onboarding
- **Primary Purpose:** Confirm setup is complete and route the trader into the product.
- **Entry Points:** S-06
- **Exit Points:** → S-08 Dashboard · → S-12 Log Trade
- **Required States:** Populated (terminal confirmation — no data dependency)
- **Security & Privacy Notes:** Authenticated.
- **Open Questions:** None.

---

# Application

## S-08 — Dashboard

- **Status:** MVP
- **Supported Flow(s):** ⚠ No single flow — the authenticated home surface that routes into F-03, F-04, F-07. Borderline; see [Cross-Document Consistency](#cross-document-consistency).
- **Primary Actor:** Authenticated trader
- **Primary Purpose:** Orient the returning trader and provide entry into the core flows, presenting computed facts with clear provenance (GR-4).
- **Entry Points:** Post-authentication (S-02) · S-07 Onboarding Complete
- **Exit Points:** → S-11 Trade History · → S-12 Log Trade · → S-17 Trade Reflection · → S-18 Profile
- **Required States:** Loading · Empty (no trades yet — teach and encourage the first log) · Populated
- **Security & Privacy Notes:** GR-1 — only the trader's own data; database-level isolation as defense in depth.
- **Open Questions:** ⚠ Dashboard risks becoming a catch-all. What is its **one** primary responsibility, and what does the MVP version actually show (which computed metrics)? _Candidate responsibility (pending implementation): help the trader understand where they are today and choose the next meaningful action. Revisit the "Dashboard" label then — the responsibility matters more than the name._

_S-09 Application Shell and S-10 Navigation were proposed here and removed — see [Removed / Reclassified](#removed--reclassified)._

---

# Trades

## S-11 — Trade History

- **Status:** MVP
- **Supported Flow(s):** F-04 (Browse Trade History)
- **Primary Actor:** Authenticated trader
- **Primary Purpose:** Browse the record of logged trades.
- **Entry Points:** S-08 Dashboard
- **Exit Points:** → S-14 Trade Details · → S-12 Log Trade
- **Required States:** Loading · Empty (no trades yet — coach the first log) · Populated · Server Error
- **Security & Privacy Notes:** GR-1 — only trades the trader owns; RLS as defense in depth.
- **Open Questions:** Filter & search is a Planned flow, not MVP — confirm the MVP list has no filtering.

## S-12 — Log Trade

- **Status:** MVP
- **Supported Flow(s):** F-03 (Log a Trade) — reference flow
- **Primary Actor:** Authenticated trader recording a completed trade
- **Primary Purpose:** Accurately capture a completed trade and its executions so the application computes performance deterministically.
- **Entry Points:** S-08 Dashboard · S-11 Trade History · S-07 Onboarding Complete
- **Exit Points:** → Confirmed Capture → S-14 Trade Details or S-11 · → S-15 Edit Trade (to correct)
- **Required States:** Populated · Validation Error · Recoverable Failure (draft recovery, upload failure — input preserved) · **AI Unavailable** · **Confirmed Capture** (flow-end)
- **Security & Privacy Notes:** GR-1 (server-authorized creation and every association) · GR-3 (metrics application-computed) · GR-6 (drafts/failures never lose input) · GR-8 (AI never writes the record) · GR-9 (discarding a draft is confirmed).
- **Open Questions:** Is structured reflection captured inline here or deferred to S-17 (F-03's open reflection-boundary question)? How are still-open positions / partial exits represented at log time?

## S-14 — Trade Details

- **Status:** MVP
- **Supported Flow(s):** F-04 (view a trade) · entry point to F-05, F-06, F-07
- **Primary Actor:** Authenticated trader
- **Primary Purpose:** Present a single trade's full record — executions, computed metrics, notes, reflection — with provenance visually distinct (GR-4).
- **Entry Points:** S-11 Trade History
- **Exit Points:** → S-15 Edit Trade · → Destructive Confirmation (F-06) · → S-17 Trade Reflection
- **Required States:** Loading · Populated · Authorization Error (not found / not owned) · **Destructive Confirmation** (delete) · **AI Unavailable** (if reflection is surfaced here)
- **Security & Privacy Notes:** GR-1 — owned trade only; private screenshots served via server-authorized, time-limited access.
- **Open Questions:** Is reflection displayed here, or only in S-17? Depends on the F-03 reflection-boundary decision.

## S-15 — Edit Trade

- **Status:** MVP
- **Supported Flow(s):** F-05 (Edit a Trade)
- **Primary Actor:** Authenticated trader correcting a trade
- **Primary Purpose:** Change a persisted trade through the explicit edit path — never silently, never as a side effect.
- **Entry Points:** S-14 Trade Details · S-12 Confirmed Capture (correct-now path)
- **Exit Points:** → Confirmed Capture → S-14 Trade Details
- **Required States:** Loading · Populated · Validation Error · **Confirmed Capture** (flow-end)
- **Security & Privacy Notes:** GR-1 · GR-3 (metrics recomputed deterministically on change) · GR-8 · GR-9.
- **Open Questions:** Pressure-test the confirmed-outcome boundary here and in F-06 (see `NOTES.local.md` Resume item #2) before promoting it to a rule.

---

# Reflection

## S-17 — Trade Reflection

- **Status:** MVP
- **Supported Flow(s):** F-07 (Reflect on a Trade with the AI Coach)
- **Primary Actor:** Authenticated trader reflecting on a trade
- **Primary Purpose:** Support evidence-grounded reflection on decision quality along the separate structural and behavioral dimensions (GR-5, GR-7); AI coaching is labeled and non-authoritative (GR-4, GR-8).
- **Entry Points:** S-14 Trade Details · S-08 Dashboard
- **Exit Points:** → S-14 Trade Details
- **Required States:** Loading · Populated · **Insufficient Evidence** (coach admits uncertainty rather than diagnosing) · **AI Unavailable** · Server Error
- **Security & Privacy Notes:** GR-1 · GR-7 (coaching traceable to authorized evidence only) · GR-8 (read-only) · GR-10 (usable degraded).
- **Open Questions:** The reflection-boundary question — is raw reflection captured inline in S-12, with the evidence model living here? (This is the first decision in `NOTES.local.md`.)

_Named "Trade Reflection," not "AI Coach": the coach is a capability, the screen is the trader's purpose. The glossary defines **AI Coach** as a capability, not a screen._

---

# Settings

## S-18 — Profile

- **Status:** MVP
- **Supported Flow(s):** SF-04 Manage Profile (supporting flow — detailed journey not yet drafted)
- **Primary Actor:** Authenticated trader
- **Primary Purpose:** View and update personal profile.
- **Entry Points:** S-08 Dashboard
- **Exit Points:** → S-08 Dashboard
- **Required States:** Loading · Populated · Validation Error
- **Security & Privacy Notes:** GR-1.
- **Open Questions:** Which fields are in MVP? Confirm when SF-04 is drafted.

## S-19 — Trading Accounts

- **Status:** MVP
- **Supported Flow(s):** F-02 (manage / add trading accounts)
- **Primary Actor:** Authenticated trader
- **Primary Purpose:** View and manage the trader's trading accounts.
- **Entry Points:** S-08 Dashboard
- **Exit Points:** → S-06 Create Trading Account
- **Required States:** Loading · Empty · Populated · **Destructive Confirmation** (account removal)
- **Security & Privacy Notes:** GR-1 · GR-9 (account deletion is destructive and confirmed).
- **Open Questions:** What happens to trades owned by a deleted account? Account-deletion semantics are undefined.

## S-20 — Security

- **Status:** MVP
- **Supported Flow(s):** SF-03 Manage Security & Sessions (supporting flow — detailed journey not yet drafted)
- **Primary Actor:** Authenticated trader
- **Primary Purpose:** Manage credentials and sessions (password change, sign out of devices).
- **Entry Points:** S-08 Dashboard
- **Exit Points:** → S-08 Dashboard
- **Required States:** Loading · Populated · Validation Error · Authorization Error (re-authentication required for sensitive changes)
- **Security & Privacy Notes:** GR-1 — re-authentication required for sensitive changes.
- **Open Questions:** Is 2FA in MVP? Confirm when SF-03 is drafted.

## S-21 — Data & Privacy

- **Status:** MVP _(hosts account deletion, which is MVP; data export remains Planned.)_
- **Supported Flow(s):** SF-05 Delete Account (MVP — supporting flow, **journey drafted** in `user-flows.md`) · Export User Data (Planned)
- **Primary Actor:** Authenticated trader
- **Primary Purpose:** Give the trader control over their own data — delete their account and private data (MVP); export their data (Planned).
- **Entry Points:** S-08 Dashboard
- **Exit Points:** → S-08 Dashboard
- **Required States:** Populated · **Destructive Confirmation** (account deletion; and export when built)
- **Security & Privacy Notes:** GR-1 · GR-9 (account deletion is destructive, confirmed, and cascades to all private data — trades, executions, screenshots, AI conversations, and stored files per ADR 0006).
- **Open Questions:** Owned by `user-flows.md` → SF-05 Open Questions now that the journey is drafted. Still live there: the unresolved deletion and backup-expiry windows, whether deletion is immediate or reversible, and whether MVP deletion without export is acceptable. Placement — could alternatively live as a "danger zone" in S-20 Security — remains this document's call. Export scope deferred with its Planned flow.

---

# Cross-Cutting States

_These are **states**, not screens. Each is owned by the screens it appears in; it is listed here because it recurs and carries a governing rule, so its behavior is defined once. Retired proposed IDs are noted for traceability._

## AI Unavailable  _(was proposed as S-22)_

- **Governing rule:** GR-10 (Graceful AI failure)
- **Appears in:** S-12 Log Trade · S-14 Trade Details · S-17 Trade Reflection
- **Behavior:** An in-context document state — never a full-page error. Core journaling and review remain fully usable. The trader's own reflection input is preserved, a later retry is offered, and the system never fabricates or partially invents a coaching result. Mirrors the "AI Unavailable State" already defined inside F-03 in `user-flows.md` and the AI-failure behavior in `architecture.md` → Error Handling.

## Insufficient Evidence

- **Governing rule:** GR-7 (Evidence-grounded coaching)
- **Appears in:** S-17 Trade Reflection (and future coaching surfaces)
- **Behavior:** When authorized evidence is insufficient, the coach acknowledges uncertainty rather than diagnosing or fabricating a conclusion. Distinct from AI Unavailable: the AI *is* available, but the evidence is not sufficient to support a claim.

## Confirmed Capture  _(was proposed as S-13 Trade Confirmation)_

- **Governing rule:** GR-2 (Clear outcome)
- **Appears in:** S-12 Log Trade · S-15 Edit Trade
- **Behavior:** The flow-end state at which the trader has clear confirmation of what was captured and leaves with confidence — F-03/F-05 end here, not at persistence. Defined as **Confirmed Capture** in the glossary. Not a route.

## Destructive Confirmation  _(was proposed as S-16 Delete Trade Confirmation)_

- **Governing rule:** GR-9 (Safe destructive actions)
- **Appears in:** S-14 Trade Details / S-15 Edit Trade (delete a trade, F-06) · S-19 Trading Accounts (remove an account) · S-21 Data & Privacy (export / delete data)
- **Behavior:** An explicit, clearly-worded confirmation before a destructive action, with recovery or undo where practical. A state/overlay of its host screen, not a route.

_The **Empty** state is part of the [Standard States](#standard-states) vocabulary; its coaching behavior is governed by `design-principles.md` → Empty States._

---

# Removed / Reclassified

_Nothing is silently dropped — the Evolution Notes forbid orphaned screens, and that cuts both ways._

- **S-09 — Application Shell** — **Removed.** The shell is the frame screens render into, not a destination. `design-system.md` → Layout System owns shell and page dimensions.
- **S-10 — Navigation** — **Removed.** `design-system.md` → Layout System explicitly owns "navigation dimensions, sidebar dimensions." Navigation is a design concern, not an inventory item.
- **S-13 — Trade Confirmation** — **Reclassified** as the *Confirmed Capture* state.
- **S-16 — Delete Trade Confirmation** — **Reclassified** as a *Destructive Confirmation* state.
- **S-22 — AI Unavailable** — **Reclassified** as the *AI Unavailable* cross-cutting state.

Retired IDs are not reused.

---

# Flow → Screen Mapping

Every flow maps to at least one screen; every screen traces back to a flow. The screen path preserves order; the flow-end column names the state where the flow completes (per `user-flows.md`).

**Core Flows**

| Flow | Screen path | Flow-end state |
|---|---|---|
| **F-01** First-Time Onboarding | S-01 → S-03 → S-05 → S-06 → S-07 | — |
| **F-02** Create a Trading Account | S-06 (also reachable via S-19) | — |
| **F-03** Log a Trade | S-12 → S-14 | Confirmed Capture |
| **F-04** Browse Trade History | S-11 → S-14 | — |
| **F-05** Edit a Trade | S-15 → S-14 | Confirmed Capture |
| **F-06** Delete a Trade | S-14 (no dedicated screen) | Destructive Confirmation |
| **F-07** Reflect on a Trade with the AI Coach | S-17 | — |

**Supporting Flows** _(infrastructure; SF-02 and SF-05 drafted in `user-flows.md`; SF-01, SF-03, SF-04 not yet)_

| Flow | Screen(s) | Flow-end state |
|---|---|---|
| **SF-01** Sign In | S-02 | — |
| **SF-02** Password Recovery | S-04 | — |
| **SF-03** Manage Security & Sessions | S-20 | — |
| **SF-04** Manage Profile | S-18 | — |
| **SF-05** Delete Account | S-21 | Destructive Confirmation |

S-08 Dashboard maps to no single flow (hub surface). See [Cross-Document Consistency](#cross-document-consistency).

---

# Cross-Document Consistency

_Contradictions are named here, not silently resolved._

- **Authentication — resolved via Supporting Flows.** `architecture.md` assumes authentication and account-recovery capability (auth/security domains, E2E tests) that the *core* flows in `user-flows.md` do not cover. Rather than promote authentication into the core coaching flows, it is homed under a separate **Supporting Flows** (`SF-#`) category in `user-flows.md`, preserving the product's focus on coaching. **S-02, S-04, S-18, S-20 now trace to SF-01…SF-04.** Remaining work: draft those journeys with the per-flow template (tracked in `NOTES.local.md`). Until drafted, the screens are stable but their flow detail is pending.

- **⚠ Dashboard (S-08) supports no single flow.** It is an authenticated hub that routes into F-03/F-04/F-07. Defensible as the product's home surface, but strictly it has no owning flow. Either define a "Return & orient" / "Review performance" flow, or accept it explicitly as the routing hub. Flagged, not resolved.

- **S-21 Data & Privacy — resolved: MVP (account deletion), export Planned.** `architecture.md` and the security docs (`security-baseline.md` §6, `threat-model.md`) treat account deletion as an expected capability. It is promoted to MVP as supporting flow **SF-05 Delete Account** on trust/privacy grounds; **data export remains Planned**. S-21 is therefore an MVP screen hosting the deletion capability, with export deferred. _(Note: `security-baseline.md` §6 also lists user data export as a requirement — now a documented post-MVP target, consistent with export's Planned status.)_

- **State vocabulary — aligned, not contradictory.** The [Standard States](#standard-states) vocabulary is drawn from `architecture.md`'s browser-state language (loading, empty, success, error) so the two documents describe the same conditions in the same words. Visual form remains owned by `design-system.md`.

- **Navigation removal — confirmed by `design-system.md`.** Layout System owns navigation and shell; removing S-09/S-10 resolves a would-be duplication rather than creating one.

---

# Review Summary

_Answers to the review questions, after applying the changes above._

1. **Is the responsibility still crystal clear?** Yes — sharpened. The doc identifies screens supporting approved flows and nothing else; layout, navigation, components, and implementation are explicitly delegated in [Related Documents](#related-documents).
2. **Is anything drifting toward design-system or user-flows responsibilities?** The remaining risk was `Required States` inventing interaction micro-states (`submitting`, `draft-restored`); the [Standard States](#standard-states) vocabulary pulled those back to product-level conditions. No layout, component, or navigation detail remains.
3. **Screens that should exist but don't?** None new for the approved flows. The remaining upstream work is drafting the **Supporting Flow** journeys **SF-01, SF-03 and SF-04** that S-02/S-20/S-18 already trace to; SF-02 (S-04) and SF-05 (S-21) are drafted.
4. **Screens that should be removed as layouts / navigation / states?** Done — S-09, S-10 removed (layout/navigation); S-13, S-16, S-22 reclassified as states.
5. **Ready to become the authoritative MVP screen inventory?** **Yes, for the MVP scope as it stands.** Every MVP screen traces to an approved flow (core or supporting). S-21 is MVP (hosts account deletion, SF-05); data export remains Planned. S-08's responsibility is intentionally deferred to implementation (candidate parked, ⚠ retained as a known decision). The one remaining upstream task — drafting the SF-01, SF-03 and SF-04 journeys (SF-02 and SF-05 are done) — deepens flow detail but doesn't block the inventory.

---

# Evolution Notes

This inventory intentionally reflects the current product scope.

Screens are added only when they support an approved user flow — core or supporting.

Removing a screen requires updating the associated user flow.

Screens should never become orphaned — a screen with no flow is either a gap in `user-flows.md` or a screen that should not exist.

States are not screens. A confirmation, an empty result, or an unavailable dependency is recorded as a state of its host screen; promoting one to a screen (or demoting a real screen to a state) is a deliberate change, documented here.

Changes to this document should be driven by implementation experience rather than theoretical refinement.
