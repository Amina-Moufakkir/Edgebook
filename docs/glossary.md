# Glossary

> **One concept, one meaning, one owner.**
>
> **When a definition and this glossary disagree, the source document wins.**

---

# Using This Glossary

Definitions summarize. They do not replace the source document. If implementation requires changing a definition, update the source document first, then update this glossary.

---

# Purpose

This glossary pins the shared vocabulary the frozen documents already depend on.

It is a **trailing reference**, not a source of truth.

- Each **concept** is defined by **one canonical source document**. This glossary summarizes and points to it.
- It introduces **no new vocabulary**. A term earns an entry only once another document already relies on it.
- If an entry and its source document drift, the **source document is correct** and this entry is the bug.

Status: Draft
Last Reviewed: 2026-07-12

---

# Truth & Provenance

The product distinguishes three kinds of information by where they come from. Every screen and flow must keep them visually distinct (GR-4).

**Deterministic Truth** _(also: Deterministic Fact)_
Facts computed by application logic — P&L, prices, fills, position size, metrics. Never generated, estimated, or recalculated by an LLM.
_Canonical Source: `user-flows.md` (GR-3), `ai-philosophy.md` — Respect Deterministic Truth._

**Source of Truth**
The authoritative persisted record. Once written, it changes only through an explicit flow (Edit / Delete), never as a side effect and never by AI.
_Canonical Source: `user-flows.md` (GR-8, F-03 Completion State)._

**Computed Fact**
A deterministic value presented to the user, labeled as computed. One of the three provenance categories.
_Canonical Source: `design-principles.md` — Make Truth Visible._

**User-Authored Content**
Content the trader writes — notes, reflections. Shown as the trader's own words, distinct from computed facts and AI coaching.
_Canonical Source: `design-principles.md` — Make Truth Visible._

**AI Coaching** _(also: Interpretation)_
The AI's explanations, questions, and pattern observations. Non-authoritative, evidence-grounded, always labeled as coaching. Interpretation belongs to the AI; truth belongs to the application.
_Canonical Source: `ai-philosophy.md` — Respect Deterministic Truth; `user-flows.md` (GR-4, GR-7)._

---

# Reflection

Every trade is analyzed along two **independent** dimensions, kept separate (GR-5). One never automatically explains the other.

**Structural Analysis**
Technical evaluation of the trade: setup validity, entry, risk, position sizing, exit, market conditions. Structural mistakes are technical.
_Canonical Source: `ai-philosophy.md` — Separate Structure from Psychology._

**Behavioral Analysis**
Psychological evaluation of the trade: rule adherence, hesitation, impulsiveness, revenge trading, FOMO, overconfidence. Behavioral mistakes are psychological and require behavioral evidence.
_Canonical Source: `ai-philosophy.md` — Separate Structure from Psychology; Evidence Before Interpretation._

**Evidence**
Observable patterns that justify a behavioral claim — repeated rule violations, impulsive entries, position-size increases after losses, journal notes of distress. Without sufficient evidence, the AI must acknowledge uncertainty rather than diagnose.
_Canonical Source: `ai-philosophy.md` — Evidence Before Interpretation; `user-flows.md` (GR-7)._

---

# Trade Domain

**Trade**
A single recorded position against a trading account, composed of **one or more executions**, plus optional strategy/rule associations, notes, screenshots, reflection, and its deterministic metrics.
_Canonical Source: `user-flows.md` — F-03 Log a Trade._

**Execution**
One fill within a trade — carries quantity, price, timestamp, and fees/commissions. A trade may have multiple executions, partial fills, and partial exits.
_Canonical Source: `user-flows.md` — F-03 Primary Path._

**Trading Account**
The account a trade is logged against. At least one must exist before a trade can be logged (F-02 precondition of F-03).
_Canonical Source: `user-flows.md` — F-02, F-03 Preconditions._

**Direction**
Whether a trade is **long** or **short**. Must be consistent with the recorded executions.
_Canonical Source: `user-flows.md` — F-03 Primary Path, Validation._

**Strategy**
A named approach optionally associated with a trade.
_Canonical Source: `user-flows.md` — F-03 Primary Path._

**Trading Rule**
A discipline constraint optionally associated with a trade; rule adherence is later a subject of behavioral analysis.
_Canonical Source: `user-flows.md` — F-03 Primary Path; `ai-philosophy.md` — Behavioral Analysis._

**Deterministic Metrics**
Application-computed derived values for a trade — position size, average entry, average exit, gross P&L, fees, net P&L, and applicable metrics. A form of Deterministic Truth.
_Canonical Source: `user-flows.md` — F-03 Primary Path (GR-3)._

---

# Trade Lifecycle States

A trade moves through distinct states during and after logging (F-03). These are **not** interchangeable.

**Draft** _(also: Recoverable Draft)_
In-progress trade entry, private to the trader, that survives interruption or recoverable failure. Never silently promoted to a saved trade.
_Canonical Source: `user-flows.md` — F-03 Abandonment & Re-entry (GR-6)._

**Persisted**
The moment the trade is written to source-of-truth, atomically (all-or-nothing). An implementation milestone **inside** the flow — not the flow's end.
_Canonical Source: `user-flows.md` — F-03 System Responses, Completion State._

**Confirmed Capture**
The point at which the trader has clear confirmation the trade was accurately recorded and leaves with confidence. This — not persistence — is where F-03 ends.
_Canonical Source: `user-flows.md` — F-03 Completion State._

---

# AI Behavior

**AI Coach**
The product's coaching capability — an evidence-based coach, not a guru, therapist, or financial advisor. Helps traders reflect on decision quality; never predicts markets or replaces judgment. A **cross-cutting capability**, not a screen — it appears across screens, flows, and domains.
_Canonical Source: `ai-philosophy.md` — Purpose, Coaching Boundaries._

**Non-Authoritative AI**
The AI is read-only by default and never overwrites source-of-truth records without explicit, validated user action.
_Canonical Source: `user-flows.md` (GR-8)._

**Graceful AI Failure**
Core journaling and review remain usable when AI is unavailable; the system never fabricates a coaching result. Presented as a state within a screen, never as a fabricated answer.
_Canonical Source: `user-flows.md` (GR-10)._

---

# Governance Vocabulary

**Global Rule (GR-#)**
An enforceable, testable constraint every applicable flow must satisfy. Referenced by ID (e.g. GR-2). Distinct from Flow Principles, which are aspirational and not individually testable.
_Canonical Source: `user-flows.md` — Global Rules._

**User Flow (F-# / SF-#)**
A journey the product must support, described from the user's perspective — goal and path, independent of screens. Screens are derived from flows, never the reverse.
_Canonical Source: `user-flows.md` — Purpose._

**Core Flow (F-#)**
A journey that delivers the product's coaching value — onboarding, logging, review, reflection. The reason the product exists.
_Canonical Source: `user-flows.md` — MVP Flows._

**Supporting Flow (SF-#)**
An infrastructure journey the product requires to function but which is not itself coaching value — authentication, recovery, security, profile. Kept separate from core flows so the product's focus stays on coaching. May be MVP-required without being core.
_Canonical Source: `user-flows.md` — Supporting Flows._

**Flow Principle**
An aspirational experience goal every journey should strive for (e.g. minimize friction, fail safely). Guides judgment; not individually testable. Contrast with Global Rule.
_Canonical Source: `user-flows.md` — Flow Principles._

---

# Product Structure

The distinction between a screen and a state is load-bearing: it decides what earns a screen ID and what does not. This section also pins the broader structural terms (e.g. capability) that describe *what kind of thing* something is.

**Screen**
A routable destination with one primary responsibility, existing only because it supports an approved user flow (core or supporting). Receives a screen ID (`S-#`).
_Canonical Source: `screen-inventory.md` — Principles._

**State**
A condition that appears *within* a screen — a confirmation, an empty result, a loading period, an unavailable dependency. A state is not a destination: it does not receive a screen ID and is recorded in its host screen's `Required States`.
_Canonical Source: `screen-inventory.md` — Standard States._

**Cross-Cutting State**
A named, rule-bearing state that recurs across multiple screens and is therefore defined once — e.g. Confirmed Capture (GR-2, see Trade Lifecycle States), Destructive Confirmation (GR-9), AI Unavailable (GR-10), Insufficient Evidence (GR-7). Still a state, not a screen.
_Canonical Source: `screen-inventory.md` — Cross-Cutting States._

**Capability**
A product behavior that may appear across multiple screens and flows. Capabilities are not screens, components, or flows. Example: the AI Coach.
_Canonical Source: `screen-inventory.md` — S-17 note (capability vs. screen)._
