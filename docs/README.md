# Edgebook AI — Handbook

The documentation that defines Edgebook AI: why it exists, what it is, how it looks, how it works, and how it stays secure.

**One navigation rule:** every responsibility has one owning document. When two documents disagree, the one that owns the responsibility wins — screens derive from flows, scope is owned by the flows, and definitions are pinned in the glossary.

---

## Map

**Product — start here, in order**

- `PROJECT_CHARTER.md` — why the product exists (repo root)
- `docs/product/product-spec.md` — product scope, MVP, and boundaries
- `docs/product/user-flows.md` — the journeys that must work; the source of truth for MVP scope
- `docs/product/screen-inventory.md` — the screens that support those flows (screens derive from flows, never the reverse)

**Experience and system — how it looks and works**

- `docs/design/design-principles.md` — user-experience philosophy
- `docs/design/design-system.md` — the visual system: tokens, typography, layout, components
- `docs/ai/ai-philosophy.md` — how the AI reasons and coaches
- `docs/architecture/architecture.md` — system architecture and boundaries
- `docs/architecture/implementation-roadmap.md` — phased delivery plan and the MVP boundary
- `docs/architecture/decisions/` — architecture decision records (ADRs): the reasoning behind major technical choices

**Security**

- `docs/security/security-baseline.md` — minimum security requirements
- `docs/security/threat-model.md` — assets, threats, and controls
- `docs/security/release-checklist.md` — the go/no-go security gate before release
- `docs/security/incident-response.md` — incident severity, roles, and playbooks

**Reference**

- `docs/glossary.md` — shared vocabulary; one concept, one meaning, one canonical source

---

## Conventions

- **Status.** Each document carries `Status:` (Draft · Frozen v1.0 · Deprecated) and `Last Reviewed:` near its top. Status signals lifecycle, not quality. ADRs use `Status / Date / Deciders`.
- **Ownership.** Each concept has one canonical source document; the glossary points to it rather than restating it.
- **Derivation.** Flows define what must work → screens derive from flows → the design system defines how they look → the architecture defines how they work.
