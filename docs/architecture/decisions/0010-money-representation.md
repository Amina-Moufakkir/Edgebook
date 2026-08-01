# 0010 — Money Representation

- **Status:** Accepted — approved 2026-08-01. Implementation may proceed against this record. **Open Question 2 (multi-currency) is explicitly *not* approved and remains open.**
- **Date:** 2026-07-31 (drafted) · 2026-08-01 (accepted)
- **Deciders:** Founder
- **Scope recorded at approval:** **Equities-first MVP, no crypto.** See Open Question 1 — this answer is load-bearing and the reasoning it forecloses is recorded there.
- **Related:** [0008 Supabase Integrated Infrastructure], [0009 Row-Level Security Strategy]

---

> This ADR explains *how Edgebook AI represents money, prices, and quantities*, and where rounding is permitted. It defines the numeric contract that deterministic truth depends on — not the formulas that consume it, and not how values are displayed.

## Context

Edgebook AI's central promise is that computed facts are exact. `user-flows.md` F-03 states it as a success metric: computed P&L and metrics must match an independent recomputation **100% of the time**. GR-3 makes those facts application-owned and forbids the AI from producing them. If the numbers drift, the product's credibility goes with them — a coaching platform that miscounts a loss has nothing left to coach with.

`architecture.md` → Financial Data and Calculations already sets the direction: do not use imprecise floating-point arithmetic casually for money, use approved decimal or integer representations, store source values with appropriate precision, and define rounding rules explicitly. It stops short of choosing them, and lists "How should monetary values be represented?" as an Open Question. Three documents are waiting on that answer:

- `user-flows.md` F-03 rejects "fees or prices that violate the approved money representation (**no silent rounding**)" — a validation rule with no defined referent.
- `screen-inventory.md` S-06 ties trading-account currency fields to this ADR.
- `release-checklist.md` gates release on "money and decimal values use the approved representation."

The decision is not merely "avoid floats." Trading data mixes three numeric kinds that behave differently: **prices** (tick sizes vary by instrument, often more than two decimal places), **quantities** (fractional for crypto and fractional shares, whole for futures contracts), and **money** (fees, P&L, balances). Their product — price × quantity — routinely lands on more precision than any currency can settle. Deciding where that precision is allowed to be lost, and where it must not be, is the substance of this ADR.

## Decision Principles

- **The user's input is source of truth.** What the trader typed is stored exactly as typed. The system never quietly improves it.
- **Exactness over convenience.** Binary floating point is convenient and wrong; a representation that cannot represent `0.1` exactly has no place near money.
- **Rounding is an event, not a side effect.** Every rounding has a named boundary, a documented rule, and a test. Precision is never lost incidentally.
- **Reject rather than adjust.** Input the representation cannot hold is a validation failure the trader sees and fixes — not something the system rounds away (GR-2, and F-03's "no silent rounding").
- **Calculation and formatting are separate concerns.** Exact values are computed once; how many decimals a screen shows is a display decision and changes nothing stored.
- **Currency is part of the value.** An amount without a currency is not a monetary value; it is a number that will eventually be added to the wrong thing.

## Decision

Represent all financial values as **exact decimals** — `numeric` in PostgreSQL, an arbitrary-precision decimal type in application code — with a fixed set of value classes, documented scales, and rounding permitted only at named boundaries. **Binary floating point is never used for a price, quantity, fee, or monetary amount, at any layer.**

### Value classes

| Class | Storage | Scale rationale |
|---|---|---|
| **Price** | `numeric(28, 10)` | Covers equity ticks, forex pipettes (5 dp), and crypto (8 dp) without per-instrument schema changes. |
| **Quantity** | `numeric(28, 10)` | Fractional shares and crypto require fractional quantities; whole-contract instruments are a subset. |
| **Money** — fees, commissions, gross/net P&L, balances | `numeric(28, 8)` | Checked against published fee schedules, not assumed. FINRA's Trading Activity Fee is $0.000195/share (2026), so a 10-share sale assesses $0.00195 — five decimals — and FINRA applies no rounding rule, calculating in aggregate. Crypto fees charged in the traded asset reach eight (one satoshi = 1e-8). Scale 4 would round the first and zero the second. |
| **FX rate** — only if multi-currency is approved | `numeric(28, 10)` | **Placeholder, not a pinned scale.** Convention only, and some conventions exceed 10. Unused until multi-currency is approved; pin it against a real rate source at that point, along with the rounding boundary FX arithmetic introduces (see Open Question 2). |

Scales are generous deliberately: widening a `numeric` scale later is a migration over live user data, and the cost of unused precision is nil.

**Scale and precision are anchored differently, and the difference is worth stating.** Each *scale* — the fractional half — is pinned to a referent: 8 to the finest real fee and settlement granularity found in published schedules, 10 to instrument tick and quote precision (IBKR activity statements display prices to seven decimals). Each *precision* — the total digit count, and therefore the integer half — is sized by headroom rather than by a referent: `numeric(28, 8)` leaves 20 integer digits, which holds ~1e20 units, roughly ten orders of magnitude beyond the largest lifetime aggregate notional any individual trader could accumulate. The check, if it is ever worth re-running: the largest total notional aggregated across an account's full history, expressed in the smallest unit, must fit in 20 integer digits.

That asymmetry is acceptable because **the column type is a container, not the constraint**. Absurd magnitudes are rejected by documented bounds validated server-side (see Boundary rules), not by running out of digits. A generous integer half costs nothing and is never the thing that says no.

### Where rounding is permitted

Rounding happens in exactly two places, and nowhere else:

1. **When a computed result becomes a money value.** `price × quantity` produces more precision than the Money class holds; the result is rounded to scale 8 at that boundary. Intermediate steps within a single calculation are **not** rounded — a sum of executions rounds once, at the end, never per execution.
2. **At display.** The interface formats to the currency's minor unit (two decimals for USD/EUR). This is a formatting concern owned by the design system; it never changes a stored or computed value.

**Rounding rule: half away from zero.** At the Money scale of 8, a gain of `0.000000005` rounds to `0.00000001`; a loss of `-0.000000005` rounds to `-0.00000001`. The rule is symmetric so that rounding introduces no directional bias between wins and losses — in a journal whose purpose is honest self-assessment, a rounding mode that flatters one side of the ledger is a correctness problem, not a rounding preference.

### Boundary rules

- **No binary floating point anywhere in the chain.** No `float`/`double`/`real` columns, and no JavaScript `number` holding a price, quantity, fee, or amount — including "just for a moment" during parsing, sorting, or charting.
- **Decimals cross every boundary as strings.** JSON has no exact decimal type; serializing through a JSON number silently destroys precision before any validation can catch it. Requests, responses, form payloads, and AI context all carry decimals as strings.
- **The server re-validates magnitude and scale.** Client validation is not a security boundary (`architecture.md` → Validation). Input exceeding the documented bounds or scale is **rejected with a plain-language message** (GR-2) and the entered data is preserved for correction (GR-6) — it is never truncated, and never rounded to fit.
- **Documented bounds.** Every price, quantity, fee, and account value has a documented maximum magnitude, satisfying `security-baseline.md` §9. Bounds are validated server-side and enforced again by database constraints (`architecture.md` → Database Constraints).
- **Currency travels with the amount.** Every stored monetary value carries an explicit currency. Arithmetic across two currencies is **forbidden** unless an explicit, recorded conversion is applied — the system never sums mixed currencies into a single figure.
- **The AI never computes or restates a monetary value.** Money reaches the AI as an already-computed string, as a fact it may interpret and must not recalculate (GR-3, GR-8). A number the AI produced is never persisted as a monetary value.

### Where this lives

The money type and its arithmetic are **shared financial types** (`architecture.md` → Shared Code), provider-neutral and independent of any database or UI concern. Feature domains consume the type; they do not define their own numeric handling. Formatting lives in the presentation layer and never in the calculation layer.

### Testing requirements

Exactness is an assumption, so it is verified rather than trusted. `architecture.md` → Testing Strategy already requires unit coverage of financial calculations; this ADR adds the edge cases that must appear in it:

- fractional quantities (crypto, fractional shares) and sub-cent fees at real granularity — a TAF-sized `0.00195` and an in-asset crypto fee at `0.00000123`, both of which scale 4 would have damaged;
- multiple executions, partial fills, and partial exits summing to an exact total;
- short-direction sign handling, so a profitable short is not sign-flipped;
- values landing exactly on a rounding boundary **at the Money scale**, in both signs — `0.000000005` and `-0.000000005`. A four-decimal stand-in would pass while never exercising the range the scale was widened to;
- input exceeding maximum scale or magnitude — rejected, not rounded;
- a round-trip through serialization proving no value passed through a floating-point type, using a value binary floating point **cannot** hold — a 19-significant-digit amount such as `12345678901.12345678`, which exceeds float64's ~15 reliable significant digits and so fails loudly if any layer parses it into a `number`. Verified, not assumed: the nearest float64 is `12345678901.1234569549560546875`, an error of `1.75e-7` — about 17 units at the Money scale, so the assertion still fails *after* the value is rounded to scale 8. That second property is the one that matters. A test value whose float error rounds away at the stored scale passes while proving nothing, which is the failure mode this test exists to rule out.

## Open — founder decision required

Question 1 was **answered at approval**. Question 2 remains **open and unapproved**.

1. **Which instrument classes does the MVP support? — ANSWERED 2026-08-01: equities-first MVP, no crypto.**

   Recorded as *"no crypto,"* deliberately, rather than the narrower *"no arbitrary on-chain tokens."* The scales below were pinned to crypto referents — satoshi for money, token decimals for the blocker analysis — during a discussion in which crypto was live scope. A future reader seeing only "no arbitrary tokens" beside a satoshi-anchored scale could reasonably conclude that major crypto was in scope and that **"we're already satoshi-safe, so adding BTC is free."** It is not, and this paragraph exists to foreclose that:

   - **The crypto anchoring is design margin, not commitment.** With an equities-first MVP, money scale 8 and price scale 10 are pure headroom — nothing sits on an edge. Satoshi is why the number is 8; it is not evidence that BTC is supported.
   - **Asset tier does not bound precision; denomination does.** Scale 8 fits BTC exactly, with zero headroom — one satoshi *is* 1e-8. It does **not** fit ETH, whose smallest native unit is wei — **18 decimals, fixed by the protocol**, a property of the base unit rather than a value any contract declares. The figure coincides with the ERC-20 convention cited below; the mechanism does not. An ERC-20's precision is a per-contract `decimals()` field, which is *why* arbitrary tokens are unbounded; ETH's is simply deeper than 8 and always has been. "Major crypto" is therefore not a safe category: it holds for the first asset anyone would name and breaks on the second.
   - **Adding any crypto is a conscious scope decision**, not a capability these scales already grant. The question to re-ask at that point is not *"which coins?"* but *"are amounts ever recorded in token-native units, or always converted to fiat at the moment of recording?"* Fiat-denominated throughout, and this ADR is untouched. Token-native, and the token's own `decimals` sets the ceiling — for ERC-20 a per-token `uint8` up to 255, where **no fixed scale is safe by construction** and the correct model is per-asset scale metadata carried with the amount, as token contracts do it. That is a different representation model. Adopting it later means rewriting settled financial records, not widening a column.

2. **Does the MVP expose multi-currency? — STILL OPEN. Not covered by the 2026-08-01 approval.** `user-flows.md` F-03 Open Questions and `screen-inventory.md` S-06 both defer to this ADR, and both remain **partially** deferred because of it: this record resolves the representation half of each; the currency half waits on this question. The architectural constraint above — currency stored on every monetary value from day one — holds either way and is cheap now, expensive later. Recommended: **carry currency from day one, restrict the MVP to a single currency per trading account, and defer cross-currency aggregation and FX rates entirely.** That keeps the option open without building it. This recommendation needs your approval before F-02/S-06 field definitions are settled.

   **Approving multi-currency later introduces a third rounding boundary this ADR does not define.** FX rate (scale 10) × money (scale 8) produces scale 18, and where that result rounds — before or after conversion, per execution or per converted total, and in which currency the rounded value is authoritative — is a named boundary that does not exist yet. It is recorded here so it is designed at approval rather than discovered in a reconciliation discrepancy. Until then the deferral holds precisely because no FX arithmetic occurs. Pin the FX rate scale against a real rate source at the same time: 10 is convention, some conventions exceed it, and it is the one row in the value-class table sized by neither a referent nor headroom.

A third F-03 open question — whether an in-progress/open trade is recordable — does **not** affect this ADR. It changes which values exist (an unrealized P&L would need a mark price), not how any of them are represented.

## Alternatives Considered

### A. Integer minor units (store everything in cents)

- The conventional payments pattern: an amount is an integer count of the currency's smallest unit.
- Rejected. It fits fixed-2dp settlement amounts, not price × quantity arithmetic. Prices and quantities need more than two decimals, so each would carry its own implied scale — and implied scales are invisible in the type system, so mixing them is a silent wrong-by-10³ bug rather than a compile error. For a solo-founder codebase, a bug class that fails quietly is the worst kind.

### B. Binary floating point with careful rounding

- Use `double` and round at the end.
- Rejected outright. `0.1 + 0.2 ≠ 0.3` in binary floating point; errors accumulate across executions and are not reliably reproducible. This directly contradicts F-03's requirement that an independent recomputation match 100% of the time, and `architecture.md`'s existing rule against casual float arithmetic.

### C. Exact decimals with a single universal scale

- One `numeric(28, 10)` type for every financial value, money included.
- Rejected, though close. Money is genuinely different in kind: it settles, it is reported, and it needs one defined place where precision stops. A universal scale defers that decision forever and pushes rounding into whichever caller happens to need it — precisely the incidental rounding this ADR exists to prevent.

### D. Store the string the user typed, parse on demand

- Keep raw input; interpret at read time.
- Rejected. Validation and arithmetic would happen everywhere rather than once, database constraints could not enforce bounds, and the source of truth would be an unvalidated string.

## Consequences

**Positive**

- Deterministic, reproducible arithmetic — the precondition for F-03's correctness metric and for GR-3 meaning anything.
- The trader's input is preserved exactly; the system never quietly changes a recorded fact.
- Sub-cent fees survive, so net P&L reconciles against a broker statement.
- Rounding is auditable: named boundaries, one rule, tested in both signs.
- Multi-currency stays a product decision rather than becoming a migration.

**Negative / risks**

- Decimal arithmetic is more verbose than native numbers, and every boundary must resist the temptation to parse into a `number` — an ongoing discipline, not a one-time setup. This is the most likely place the decision erodes.
- Charting and sorting libraries expect floats; converting for **display or plotting only** is acceptable, but the converted value must never flow back into a calculation or a stored record.
- Scales chosen before real instrument data exists may need one migration if an instrument exceeds them.
- Exact decimal arithmetic is slower than floating point. Irrelevant at this scale, and worth stating so it is never optimized away without measurement.

## Security Implications

- **Input abuse.** Unbounded precision or magnitude is a denial-of-service and storage-abuse vector; documented bounds, validated server-side, close it (`security-baseline.md` §9).
- **Silent truncation is a data-integrity failure.** A value quietly rounded to fit is a corrupted user record. Rejection is the secure behavior.
- **Client input is untrusted.** Precision and bounds are re-validated server-side regardless of what the client enforced.
- **AI boundary.** Money values are application-computed and enter AI context as facts; a monetary figure originating from model output is never persisted (GR-3, GR-8).
- **Release gate.** `release-checklist.md` verifies that money and decimal values use the approved representation; this ADR is what "approved" refers to.

## Portability Strategy

- The money type is **application-owned and provider-neutral** — it does not depend on Supabase, on any ORM, or on PostgreSQL specifics beyond exact `numeric` semantics, which every serious relational database provides.
- Exactness is a **domain guarantee**, not an infrastructure feature. If the database or provider (0008) changes, the money type and its tests are unaffected; only column definitions are re-expressed.
- Formatting stays in the presentation layer, so a change to how values are displayed can never alter how they are computed.

## Review Triggers

Revisit this decision if:

- an instrument requires precision beyond the documented scales;
- multi-currency accounts or cross-currency aggregation are approved for the product;
- broker imports are introduced and arrive with their own precision and rounding conventions (currently out of scope per `product-spec.md`);
- a reconciliation discrepancy against a broker statement is traced to representation or rounding;
- aggregate analytics over large trade counts show rounding bias that half-away-from-zero does not adequately control (the point at which banker's rounding would be reconsidered);
- exact decimal arithmetic becomes a **measured** performance problem — measured, per the architecture's own rule, not assumed.

## Follow-on documentation

On approval, in the same change:

- resolve "How should monetary values be represented?" in `architecture.md` → Open Questions (leaving it listed would re-open a decision this record closes);
- confirm the currency-field answer for `screen-inventory.md` S-06 and the money-representation dependency in `user-flows.md` F-03 Open Questions.

## Guiding Principle

> The trader's numbers are recorded exactly as given.
>
> Every calculation is reproducible by hand.
>
> Precision is lost only where we said it would be, in a way we tested.
