# Design Principles

> **Edgebook AI is not designed to make trading feel exciting.**
>
> **It is designed to help traders think more clearly.**

---

# Purpose

The interface is part of the coaching experience.

Every visual decision should reduce cognitive load, encourage thoughtful reflection, and reinforce disciplined decision-making.

The goal is not to impress users with visual complexity.

The goal is to help them understand themselves.

---

# Design Philosophy

Edgebook AI is most often opened after uncertainty, frustration, hesitation, or loss.

The interface should become a calm place to think.

It should never amplify emotional intensity.

Every screen should communicate:

- clarity
- confidence
- honesty
- patience
- focus

The product should feel like an experienced mentor sitting beside the trader—not another trading platform shouting for attention.

---

# Emotional Design Principles

The emotional tone of the product is intentional.

### Calm over urgency

Avoid visual noise.

Avoid unnecessary alerts.

Avoid flashing animations.

Avoid exaggerated colors.

The interface should lower emotional intensity rather than increase it.

---

### Clarity over decoration

Every visual element should have a purpose.

If something exists only because it looks impressive, remove it.

Beauty comes from clarity, hierarchy, spacing, and restraint.

---

### Reflection over speed

The interface should encourage thoughtful decisions rather than rapid interaction.

Reducing impulsive behavior is part of the product experience.

Sometimes slowing a user down is the correct design decision.

---

### Confidence without arrogance

The product should feel professional.

Never intimidating.

Never childish.

Never flashy.

Users should trust the software because it is clear—not because it is loud.

---

# Information Hierarchy

Every screen should answer one primary question.

Avoid presenting every piece of information at once.

Reveal complexity progressively.

Important information should become obvious through hierarchy—not through larger buttons or brighter colors.

Whitespace is a design tool.

Not empty space.

---

# Make Truth Visible

The interface should clearly distinguish between:

- deterministic facts produced by the application
- AI-generated coaching
- user-authored content

Each serves a different purpose.

Their visual presentation should reinforce that distinction.

Users should never need to guess where information originated.

---

# Typography Principles

Typography should invite reading rather than scanning.

The interface favors reflection over urgency.

Metrics should be immediately readable.

Long-form journal writing should remain comfortable over extended sessions.

Financial values should prioritize alignment and precision.

Reading should never feel like work.

---

# Color Philosophy

Color communicates meaning.

It should never exist simply for decoration.

Use restrained neutral surfaces as the foundation.

Reserve strong colors for meaningful moments.

Examples:

- profit
- loss
- warnings
- confirmations
- destructive actions

Never use green or red as ambient interface colors.

Those colors should retain their semantic meaning.

Profit and loss should never rely on color alone.

Use icons, labels, signs, or supporting context to reinforce meaning.

---

# Motion Principles

Motion should explain.

Never entertain.

Animations should:

- reinforce spatial relationships
- communicate state changes
- confirm completed actions
- guide attention
- reduce uncertainty

Animations should never exist purely for decoration.

If removing an animation improves clarity, remove it.

Subtle motion is preferred over dramatic motion.

---

# Data Visualization

Charts exist to reveal patterns.

Never decorate charts.

Avoid unnecessary gradients.

Avoid excessive animation.

Avoid chart junk.

Gridlines should remain subtle.

Labels should always be readable.

Every chart should answer a specific question.

If a chart requires explanation before it becomes useful, reconsider the visualization.

---

# Empty States

An empty state should teach.

It should explain:

- why the screen is empty
- what the user can do next
- why that action matters

Never leave users staring at an empty table without guidance.

Every empty state is an opportunity to coach.

---

# Error Messages

Errors should reduce anxiety.

Explain:

- what happened
- what the user can do next
- what the system is doing

Avoid technical language.

Avoid blame.

Never expose implementation details.

Users should leave an error message feeling informed—not confused.

---

# Accessibility

Accessibility is not an enhancement.

It is part of quality.

Every interface should support:

- keyboard navigation
- visible focus indicators
- readable contrast
- screen readers
- reduced motion preferences
- meaningful labels
- scalable typography

Accessibility should be built into the system—not added afterward.

---

# Consistency Before Novelty

Prefer existing patterns over inventing new ones.

Consistency reduces cognitive load.

Novelty should solve a genuine usability problem—not satisfy a desire for visual variety.

If two screens solve similar problems, they should behave similarly.

---

# Component Philosophy

Components consume the design system.

Components do not define it.

Visual decisions belong to the design system.

Business logic belongs to components.

When a component requires a visual value that does not exist:

1. Identify its purpose.
2. Look for an existing design token.
3. If no suitable token exists, propose a new token.
4. Document why it exists before implementing it.

Implementation follows the design system.

Not the other way around.

---

# Design Tokens

Edgebook AI owns its design language.

Components should never hardcode:

- colors
- spacing
- typography
- border radius
- shadows
- motion values

Every component consumes approved design tokens.

The implementation details of those tokens are defined in `design-system.md`.

---

# Generated Code Rules

AI-generated UI must follow the same standards as hand-written UI.

### Token-only styling

Never introduce arbitrary colors, spacing values, radii, shadows, typography, or motion values.

Use approved design tokens.

---

### Existing patterns first

Before creating a new component, check whether an existing pattern solves the problem.

Prefer extending the system over creating another variation.

---

### Extract on the third repeat

Do not build abstractions prematurely.

When the same structure appears a third time, extract it into a shared component.

Grow the design system from real usage.

---

### Preserve semantic color

Positive and negative colors exist for meaningful system states.

Never use them as decorative accents.

---

### Respect accessibility

Never remove keyboard support, focus states, or accessible labels.

Do not sacrifice usability for visual appearance.

---

# Extending the System

Every new shared component or design token is a product decision.

Document why it exists before introducing it.

Prefer extending semantic concepts over inventing new primitives.

The design system should grow deliberately.

Not accidentally.

---

# Guiding Principle

**The interface should never compete with the trader's emotions.**

**It should help them see more clearly, think more calmly, and make better decisions.**
