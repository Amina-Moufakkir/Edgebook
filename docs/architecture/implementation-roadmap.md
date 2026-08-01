# Implementation Roadmap

Status: Draft
Last Reviewed: 2026-07-12

_Phased delivery plan. The **MVP boundary** is marked below. MVP scope is defined canonically by `user-flows.md` (F-01–F-07 and the SF-# supporting flows); everything after the boundary is Post-MVP and earns its own flow before implementation._

## Phase 0 — Foundation

Goals:

- repository
- CI
- auth (SF-01–SF-04)
- design system
- architecture
- security baseline

---

## Phase 1 — Core Trading Journal

Phase 1 ships as two slices: the deterministic-truth core, then screenshot uploads as a separately gated increment.

This is a **sequencing** decision, not a scope change — uploads remain in the MVP, and `user-flows.md` remains canonical for scope. The split is available because F-03 already treats screenshots as optional and non-blocking (a trade saves with an upload pending or failed, and S-12 records upload failure as a Recoverable Failure state, not a blocking one), so removing them from the first slice keeps it vertical rather than thinning it into a layer.

### Phase 1a — Core (no uploads)

Deliver:

- trades
- executions
- journal (log, browse, edit, delete)
- core performance metrics (win/loss, strategy performance, rule adherence)
- basic orientation dashboard
- account deletion (removes the account and all private data; SF-05)

### Phase 1b — Screenshot uploads _(gated increment)_

**Entry gate — the dedicated file-upload threat model required by ADR 0006 exists and has been reviewed.** No upload work begins before it: not storage wiring, not signed-URL issuance, not upload UI.

The gate is attached to **this work item, not to a phase number**. If uploads are resequenced, the gate moves with them; it is satisfied by the document existing, never by the phase arriving.

It extends `threat-model.md` **T4 — Malicious or Oversized File Upload** rather than starting from a blank page, adding depth against the storage design ADR 0006 defines — decompression bombs, content-type spoofing, executable disguise, image-parser attack surface, and signed-URL expiry tuning. Writing it before that design existed would have been a guess, which is why it is gated here rather than in Phase 0.

Deliver:

- screenshot upload with server-side type and size validation
- private storage with opaque keys and PostgreSQL-owned ownership metadata (ADR 0006)
- server-authorized, time-limited retrieval

`release-checklist.md` §6 (File Upload Security) applies from this increment onward.

---

## Phase 2 — Per-Trade AI Coach

Deliver:

- AI reflections on a single trade (separate structural and behavioral dimensions)
- evidence model
- reflective questions

**— MVP boundary —** _Phases 0–2 constitute the MVP (F-01–F-07). Everything below is Post-MVP._

---

## Phase 3 — Advanced Analytics

Deliver:

- streaks
- behavioral trend charts
- structural-analysis dashboards (distinct from the basic orientation dashboard shipped in Phase 1)

---

## Phase 4 — Cross-Trade Coaching

Deliver:

- end-of-session reviews
- weekly reviews
- cross-trade behavioral and structural pattern detection
- open-ended "ask the coach" questions

---

## Phase 5 — Learning Experience

Deliver:

- simulated trading (Practice) mode
- practice journal and progress tracking
- Learn pillar (educational content)
