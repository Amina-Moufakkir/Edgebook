# Product Specification

# Edgebook AI

**Version:** 1.0 (Draft)

---

# Product Overview

Edgebook AI is an AI coaching platform that helps traders become better decision-makers through structured journaling, behavioral analysis, evidence-based reflection, and continuous coaching.

Unlike traditional trading journals that primarily measure profit and loss, Edgebook AI focuses on improving the person making the trading decisions.

The platform does not attempt to predict the market.

It helps traders understand themselves.

---

# Product Vision

Most trading software helps traders analyze markets.

Edgebook AI helps traders analyze themselves.

The long-term vision is to become the platform where traders continuously learn, practice, reflect, and improve throughout their entire trading journey.

Rather than optimizing for better trades, Edgebook AI optimizes for becoming a better trader.

---

# Product Principles

The product is guided by five core principles.

## 1. Coach the trader, not the market.

Edgebook AI never attempts to predict prices or generate trading signals.

Its responsibility is improving decision-making.

---

## 2. Evaluate process before outcome.

Good trades lose.

Bad trades win.

Every review begins by evaluating the quality of the decision rather than the financial result.

---

## 3. Separate structure from psychology.

Technical mistakes and behavioral mistakes are different problems.

The platform should help traders distinguish between them rather than treating every losing trade as emotional.

---

## 4. Evidence before interpretation.

Insights should be supported by observable trading behavior.

The platform should avoid speculation when evidence is insufficient.

---

## 5. Long-term habits beat short-term wins.

The objective is not maximizing today's P&L.

The objective is becoming a consistently better decision-maker over thousands of trades.

---

# Problem Statement

Most trading education focuses on understanding the market.

Very little focuses on understanding the decision-maker.

As a result, many traders:

- repeat the same mistakes
- confuse luck with skill
- mistake technical problems for psychological ones
- blame psychology when the issue is structural
- improve slowly despite collecting years of trading history

Existing journals often become spreadsheets.

Existing AI tools often attempt to predict the future.

Neither consistently helps traders understand why they make the decisions they make.

Edgebook AI exists to solve that problem.

---

# Target Users

## Primary Users

Traders who actively review their own performance and want to improve through deliberate practice.

Examples include:

- beginner traders
- intermediate traders
- active day traders
- swing traders

---

## Secondary Users

Aspiring traders who want to build strong trading habits before risking significant real capital through a structured simulation experience.

---

# Core Value Proposition

Edgebook AI helps traders answer questions that are difficult to answer objectively on their own.

Examples include:

- Am I consistently following my trading plan?
- Is my strategy failing?
- Or am I failing to execute it?
- Which mistakes cost me the most?
- Which habits have improved?
- What should I work on next?
- Am I becoming more disciplined over time?

---

# Core Product Pillars

## Learn

Understand trading concepts, market structure, and trading principles.

---

## Practice

Develop skills in a simulated environment before risking real capital.

---

## Journal

Capture trades, executions, notes, screenshots, and observations.

---

## Reflect

Review decisions using objective evidence rather than memory or emotion.

---

## Coach

Receive AI coaching that identifies both structural and behavioral patterns while clearly distinguishing between them.

---

## Improve

Measure long-term progress through better habits, stronger execution, and more consistent decision-making.

---

# Minimum Viable Product

Version 1 includes:

## User Accounts

- Authentication
- User profiles
- Secure account management

---

## Trading Journal

- Trade creation
- Multiple executions
- Position tracking
- Strategy tags
- Rule tracking
- Trade notes
- Screenshot uploads

---

## Analytics

- Trading dashboard
- Performance metrics
- Win/loss statistics
- Strategy performance
- Rule adherence
- Behavioral trends

---

## AI Coaching

- Trade reflections
- Session reviews
- Weekly reviews
- Behavioral pattern detection
- Structural pattern detection
- Evidence-based coaching
- Reflective questions
- Coaching summaries

---

## Learning Experience

- Simulated trading mode (planned within Version 1 roadmap)
- Practice journal
- Progress tracking

---

# Out of Scope

Version 1 will not include:

- Live market data
- Brokerage integrations
- Automatic trade imports
- Real-money trade execution
- AI trading signals
- AI trade recommendations
- AI portfolio management
- Copy trading
- Social trading
- Public leaderboards
- Mobile applications

---

# Success Criteria

A successful user should be able to:

- consistently journal trades
- identify recurring mistakes
- distinguish structural mistakes from behavioral mistakes
- recognize improving habits
- measure long-term progress
- build greater discipline
- understand performance trends
- reflect after every trading session

Success is measured primarily by improvement in decision-making and consistency—not by profit alone.

---

# AI Boundaries

The AI:

- coaches
- teaches
- asks reflective questions
- identifies behavioral patterns
- identifies structural patterns
- explains its reasoning
- summarizes evidence
- acknowledges uncertainty

The AI never:

- predicts markets
- recommends buying or selling securities
- guarantees outcomes
- fabricates evidence
- diagnoses mental health conditions
- replaces the trader's judgment

---

# Product Evolution

Edgebook AI is designed to support the complete learning journey.

Learn

↓

Practice

↓

Journal

↓

Reflect

↓

Receive Coaching

↓

Improve

↓

Trade Live

↓

Continue Improving

The trading journal is not the destination.

It is the foundation of an AI coaching platform that grows alongside the trader throughout their journey.

---

# Dependencies

This specification works alongside the following project documents:

- `PROJECT_CHARTER.md` — why the product exists
- `docs/ai-philosophy.md` — how the AI reasons and coaches
- `docs/design-principles.md` — user experience philosophy
- `docs/design-system.md` — visual system
- `docs/architecture.md` — system architecture
- `docs/security/` — security principles and operational guidance
- `docs/decisions/` — architectural decision records

This document intentionally avoids duplicating those responsibilities.

---

# Definition of Done

A feature is considered complete only when:

- functionality works correctly
- tests pass
- accessibility requirements are satisfied
- responsive layouts are complete
- loading states exist
- empty states exist
- error states exist
- security requirements are satisfied
- documentation is updated
- architecture remains consistent
- AI review passes
- manual review passes
