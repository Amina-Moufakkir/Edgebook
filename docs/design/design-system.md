# Design System

> The Design System is the single source of truth for every visual decision in Edgebook AI.

Status: Draft
Last Reviewed: 2026-07-12

It exists to ensure that every screen, component, and interaction feels like part of the same product.

Design decisions belong here.

Components consume the system.

They never redefine it.

---

# Purpose

The purpose of this document is to define how Edgebook AI is built visually.

This document governs:

- design tokens
- theming
- typography
- spacing
- layout
- motion
- elevation
- iconography
- charts
- component architecture
- accessibility implementation

It does **not** explain _why_ the interface should feel the way it does.

That philosophy lives in:

`docs/design/design-principles.md`

---

# Design System Philosophy

The Design System owns the visual language.

Components consume the visual language.

This separation exists so the product remains:

- consistent
- scalable
- maintainable
- themeable
- AI-friendly

No component should make visual decisions independently.

---

# Rule of One Source of Truth

Every visual value has exactly one owner.

Colors belong to the color system.

Typography belongs to the typography system.

Spacing belongs to the spacing system.

Motion belongs to the motion system.

Components reference those systems.

They never redefine them.

---

# Technology Stack

The current UI stack consists of:

- Tailwind CSS v4
- Native CSS variables
- shadcn/ui
- Radix UI primitives
- class-variance-authority (CVA)
- Lucide icons

These technologies are implementation tools.

They are not the design system itself.

The system should remain portable even if implementation technologies change.

---

# Design Token Architecture

The design system is organized into three layers.

## Layer 1 — Primitive Tokens

Primitive tokens contain raw values.

Examples include:

- color values
- spacing scale
- typography scale
- radius values
- shadow definitions
- motion timings

Primitive tokens should never be referenced directly by components.

---

## Layer 2 — Semantic Tokens

Semantic tokens describe purpose.

Examples:

- background
- surface
- border
- text-primary
- text-muted
- action-primary
- success
- warning

Semantic tokens allow the product to change appearance without changing component code.

Components should primarily consume this layer.

---

## Layer 3 — Component Tokens

Component tokens describe reusable component behavior.

Examples:

- button variants
- card variants
- dialog variants
- table variants

Component tokens are built from semantic tokens.

They never introduce raw values.

---

# Ownership

The Design System owns:

- colors
- typography
- spacing
- layout
- elevation
- shadows
- radius
- motion
- iconography
- chart styling
- focus styles
- interaction states

Components own:

- behavior
- composition
- business logic

This boundary should remain strict.

---

# Color System

The color system is semantic.

Color names describe purpose.

Never appearance.

Example:

Good

- primary
- surface
- border
- warning

Avoid

- blue
- gray
- orange

Actual color values are defined in the token layer.

---

# Typography System

Typography defines:

- font families
- type scale
- weights
- line heights
- numeric typography
- reading rhythm

Typography exists to improve comprehension.

Not decoration.

Financial values should prioritize precision and alignment.

---

# Spacing System

Spacing is intentional.

Components should never invent spacing values.

Spacing tokens define:

- internal spacing
- external spacing
- layout spacing
- section spacing

Whitespace is considered part of the interface.

Not unused space.

---

# Layout System

The layout system defines:

- page width
- content width
- navigation dimensions
- sidebar dimensions
- panel spacing
- grid behavior
- responsive breakpoints

Every page should follow the same layout language.

---

# Radius System

Border radius communicates personality.

Edgebook should feel calm and modern.

Radius values should remain consistent throughout the application.

Only approved radius tokens may be used.

---

# Elevation System

Elevation communicates hierarchy.

Use borders before shadows.

Use subtle shadows before strong shadows.

Avoid creating depth purely for decoration.

Every elevated surface should have a reason to exist.

---

# Motion System

Motion communicates change.

It should:

- explain
- guide
- confirm

Motion should never distract.

Animation values belong to the token layer.

Components consume those values.

---

# Icon System

Lucide is the default icon library.

Icons should support meaning.

Never replace readable text.

Decorative icons should remain visually secondary.

---

# Chart System

Charts are analytical tools.

Not decoration.

Charts should prioritize:

- readability
- consistency
- comparison
- clarity

Avoid unnecessary gradients.

Avoid excessive animation.

Avoid chart junk.

Profit and loss should never rely on color alone.

---

# Component Architecture

The Design System provides primitives.

Business components compose those primitives.

Example:

shadcn Button

↓

Edgebook Button

↓

Trade Form

↓

Trade Review

↓

Journal Session

The further up the hierarchy, the more product-specific the component becomes.

---

# Component Growth

Do not build abstractions prematurely.

When the same UI pattern appears three times, extract it into a shared component.

Grow the component library from actual usage.

Not prediction.

---

# Accessibility Standards

Accessibility is part of quality.

Every component should support:

- keyboard navigation
- screen readers
- visible focus
- readable contrast
- reduced motion
- semantic HTML

Accessibility should never be sacrificed for aesthetics.

---

# AI Generated UI Rules

AI-generated code follows exactly the same standards as human-written code.

AI must never:

- invent colors
- invent spacing
- invent typography
- invent shadows
- invent radius values
- invent motion timings

If a required token does not exist:

1. Identify the design need.
2. Check for an existing semantic token.
3. Propose a new token.
4. Document why it exists.
5. Obtain approval before adding it.

---

# Design Debt

Temporary visual inconsistency is acceptable only when documented.

Every temporary deviation should include:

- why it exists
- expected removal date
- replacement plan

The Design System should become more consistent over time.

Never less.

---

# Future Sections

The following sections will be completed as the product evolves:

- Color Palette
- Typography Scale
- Spacing Scale
- Radius Scale
- Shadow Tokens
- Motion Tokens
- Layout Tokens
- Icon Specifications
- Chart Specifications
- Component Inventory

---

# Guiding Principle

A component should never answer:

> "What should I look like?"

It should only answer:

> "Which part of the Design System should I use?"

The Design System defines the language.

Components speak it.
