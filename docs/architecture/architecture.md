# Architecture

> Edgebook AI should be simple enough for one founder to operate, structured enough for another engineer to join, and secure enough to protect real users.

Status: Draft
Last Reviewed: 2026-07-12

---

## Evolution Strategy

This document describes the **intended architecture** of Edgebook AI.

It intentionally focuses on **where the system is going**, not the order in which every feature will be implemented.

Development will proceed incrementally.

Some architectural capabilities will exist before others, but every implementation should move the system toward the architecture described here.

The implementation order, priorities, and milestones are documented separately in:

`docs/architecture/implementation-roadmap.md`

Keeping the architecture separate from the implementation roadmap allows this document to remain stable while the product evolves.

---

## Purpose

This document defines how Edgebook AI is structured as a software system.

It explains:

- the major parts of the application
- where responsibilities belong
- how data moves through the system
- where security boundaries exist
- how the system remains testable and maintainable
- how the architecture may evolve as the product grows

This document describes architectural direction.

Individual technology decisions should be documented separately in Architecture Decision Records.

---

## Architectural Goals

The architecture should make Edgebook AI:

- secure by default
- maintainable by a small team
- easy to understand
- testable at multiple levels
- reliable with financial calculations
- safe for private user data
- capable of supporting AI features without giving the model excessive authority
- flexible enough to evolve without premature complexity
- observable in production
- efficient to develop and deploy

---

## Architecture Principles

Every architectural decision should support one or more of the following principles:

- Simplicity before cleverness.
- Explicit boundaries before shared assumptions.
- Security by default.
- Deterministic logic before AI interpretation.
- Measure before optimizing.
- Prefer deletion over unnecessary abstraction.
- Build for today's product while leaving room for tomorrow's growth.

---

## Architectural Invariants

The following rules are non-negotiable:

- Every private database query MUST be scoped to the authenticated user.
- Every private mutation MUST verify ownership on the server.
- The client MUST NOT determine authorization.
- The AI model MUST NOT act as an authorization boundary.
- Authoritative financial calculations MUST be deterministic and tested.
- AI output MUST NOT overwrite source-of-truth records without explicit validated user action.
- Production secrets MUST remain server-side.
- Private uploads MUST require authorization for access.
- Cross-user isolation tests MUST exist for every private resource.

---

## Non-Goals

The initial architecture is not designed to:

- support millions of users immediately
- use microservices without a demonstrated need
- introduce event-driven infrastructure everywhere
- optimize before performance has been measured
- duplicate responsibilities across services
- provide real-time brokerage execution
- provide market predictions or trading signals
- make the AI model an authority over application data or user permissions
- solve hypothetical scale problems before real usage exists

---

## Architectural Style

Edgebook AI begins as a **modular monolith**.

That means:

- one primary codebase
- one main deployable application
- one shared TypeScript environment
- one relational source-of-truth database
- clearly separated internal feature domains
- background processing only where necessary

The application should feel internally modular without creating the operational cost of multiple services.

This approach prioritizes:

- fast iteration
- simple deployments
- easier debugging
- fewer network boundaries
- shared types and validation
- lower infrastructure complexity

A feature may later become a separate service only when there is measurable evidence that separation improves reliability, scalability, ownership, or security.

---

## High-Level System

```text
User Browser
    |
    v
Edgebook AI Web Application
    |
    +--> Authentication Provider
    |
    +--> Application Server
             |
             +--> PostgreSQL Database
             |
             +--> Private Object Storage
             |
             +--> Background Job System
             |
             +--> AI Provider
             |
             +--> Logging and Monitoring
```

The browser is not trusted with authorization decisions, secrets, private storage credentials, or financial business logic.

The application server is the primary enforcement boundary.

---

## System Responsibilities

### Browser Client

The browser is responsible for:

- rendering the interface
- collecting user input
- providing immediate validation feedback
- displaying loading, success, empty, and error states
- sending authenticated requests
- presenting approved application data
- maintaining accessible interaction behavior

The browser is not responsible for:

- determining record ownership
- enforcing authorization
- calculating authoritative financial metrics
- choosing which private records the AI may access
- storing secrets
- directly accessing the database
- directly accessing private object storage without controlled authorization

### Application Server

The application server is responsible for:

- authenticating requests
- enforcing authorization
- validating input
- executing business rules
- coordinating database operations
- calculating financial results
- preparing analytics
- constructing authorized AI context
- controlling file upload and download access
- applying rate limits
- recording security-relevant events
- returning safe errors

The server is the main trust boundary of the application.

### Authentication Provider

The authentication provider is responsible for:

- identity verification
- sign-in flows
- session establishment
- account recovery
- supported MFA behavior
- secure token or session management

The application remains responsible for authorization.

Authentication proves identity.

It does not prove permission to access a particular trade, screenshot, account, review, or AI conversation.

### PostgreSQL Database

PostgreSQL is the system of record for structured application data.

It stores:

- users or application user references
- trading accounts
- trades
- executions
- strategies
- tags
- trading rules
- journal entries
- reviews
- metadata for uploaded files
- analytics inputs
- AI conversation metadata
- generated insight records
- audit-relevant application events where appropriate

The database should enforce important invariants through:

- foreign keys
- unique constraints
- required fields
- check constraints
- appropriate numeric types
- transactions

Application validation and database constraints should reinforce each other.

### Private Object Storage

Private object storage holds uploaded files such as:

- trade screenshots
- chart images
- future export artifacts

Storage must be private by default.

Files should be accessed through:

- short-lived signed URLs, or
- an authenticated server-controlled delivery path

The user must never be able to retrieve another user's file by changing an identifier.

### Background Jobs

Background jobs should be introduced only for work that should not block a user request.

Possible uses include:

- AI analysis
- weekly summaries
- export generation
- image processing
- cleanup tasks
- analytics recomputation
- retention and deletion workflows

Background jobs must enforce the same authorization and data boundaries as normal request handlers.

A job payload must not be trusted merely because it was created internally.

### AI Provider

The AI provider processes carefully prepared, minimized, authorized context.

The AI provider is not:

- the source of truth
- the authorization layer
- the financial calculation engine
- the owner of application state
- trusted to produce verified facts
- permitted unrestricted database access

The application decides:

- what data the model may see
- what tools it may call
- what outputs are acceptable
- what actions require confirmation
- what content must be rejected or flagged
- what evidence supports an insight

---

## Core Domains && Ownership

The application should be organized around product domains rather than technical layers alone.

Initial domains include:

- auth
- users
- trading-accounts
- trades
- executions
- strategies
- rules
- journal
- reviews
- analytics
- uploads
- ai-coach
- exports
- security

Each domain should own the logic directly related to its responsibility.

A domain may include:

- schemas
- types
- validation
- data-access functions
- server logic
- UI components
- tests
- documentation

Domains should not reach into one another's internal implementation without a clear public interface.

### Domain Ownership

Every piece of business logic has exactly one owner.

A domain may expose public interfaces.

Other domains consume those interfaces.

They should not reach into another domain's internal implementation.

If ownership becomes unclear, the architecture should be refactored before additional complexity is added.

---

## Idempotency

Operations that may be retried should be safe to execute multiple times.

Examples include:

- background jobs
- payment callbacks
- file uploads
- export generation
- AI analysis requests

Repeated execution should not create duplicate state unless explicitly intended.

---

## Dependency Direction

Dependencies should flow from product-specific code toward stable shared foundations.

```text
Pages and Feature UI
    |
    v
Domain Services
    |
    v
Validation and Business Logic
    |
    v
Data Access and External Adapters
    |
    v
Database, Storage, AI, Providers
```

Rules:

- UI components should not contain database logic.
- Database code should not depend on UI code.
- Business logic should be usable without rendering a page.
- External-provider logic should be wrapped behind application-owned interfaces.
- Financial logic should remain independent from presentation.
- AI prompts should not contain core authorization rules that belong in deterministic code.

---

## Source-of-Truth Hierarchy

Edgebook AI must distinguish between recorded facts, calculated values, and interpretations.

```text
User-entered and imported trade data
    |
    v
Validated source-of-truth records
    |
    v
Deterministic calculations
    |
    v
Aggregated analytics
    |
    v
AI coaching and interpretation
```

### Source-of-Truth Records

Examples:

- executions
- quantities
- prices
- fees
- timestamps
- user-defined rules
- journal responses

### Deterministic Calculations

Examples:

- gross P&L
- net P&L
- average entry
- average exit
- win rate
- expectancy
- profit factor
- drawdown

These calculations must be implemented in application code and covered by tests.

### AI Interpretation

Examples:

- identifying possible behavioral patterns
- generating reflection questions
- summarizing weekly activity
- comparing structural and behavioral observations

AI output must not overwrite source-of-truth facts without explicit, validated user action.

The application preserves source attribution throughout the system so the user interface can distinguish deterministic facts, AI-generated coaching, and user-authored content.

---

## Feature Flags

Incomplete or experimental features should be protected behind feature flags rather than partially integrated into production behavior.

Feature flags should have documented owners and removal plans.

---

## Data Ownership

Every private record must have clear ownership.

A private record should be connected directly or indirectly to:

- an authenticated user, or
- a tenant if multi-user organizations are added later

Ownership must be enforceable through the data model.

The application must never rely solely on identifiers supplied by the browser.

Example:

```text
Authenticated User
    |
    v
Trading Account
    |
    v
Trade
    |
    v
Execution / Note / Screenshot / Review
```

Every operation should verify the complete ownership chain when necessary.

---

## Authentication and Authorization

Authentication and authorization must remain separate concerns.

### Authentication

Determines who is making the request.

### Authorization

Determines whether that identity may perform the requested action on the requested resource.

Every private operation requires server-side authorization.

This includes:

- reading
- creating
- updating
- deleting
- exporting
- uploading
- downloading
- generating analytics
- initiating AI analysis
- retrieving AI context
- running background jobs

The client may request an action.

The server decides whether it is allowed.

---

## Tenant Isolation

The initial product may be single-user-per-account, but the architecture should still enforce tenant-style isolation.

Required rules:

- Every private query is scoped to the authenticated user.
- Every mutation verifies ownership.
- Background jobs preserve ownership context.
- AI retrieval uses only already-authorized records.
- Exports include only authorized data.
- File access verifies ownership.
- Administrative access is explicit and audited.
- Cross-user access tests are mandatory.

The AI model must never be responsible for tenant isolation.

---

## Financial Data and Calculations

Financial calculations require deterministic, testable logic.

Rules:

- Do not use imprecise floating-point arithmetic casually for money.
- Use approved decimal or integer representations.
- Store source values with appropriate precision.
- Keep calculation logic separate from UI formatting.
- Support multiple executions.
- Support long and short positions.
- Include fees and commissions.
- Define handling for partial exits.
- Define handling for open positions.
- Define rounding rules explicitly.
- Never allow the AI to calculate authoritative P&L.

Calculation functions should be:

- pure where possible
- independently testable
- deterministic
- documented
- covered by edge-case tests

---

## Validation

Validation must exist at multiple layers.

### Client Validation

Used for:

- immediate feedback
- reducing user mistakes
- improving form usability

Client validation is not a security boundary.

### Server Validation

Used for:

- security
- data integrity
- business rules
- trusted input handling

### Database Constraints

Used for:

- preventing invalid persisted states
- enforcing relationships
- protecting invariants

The same schema may be reused where appropriate, but server enforcement remains mandatory.

---

## File Upload Architecture

Uploads should follow this flow:

```text
Authenticated User
    |
    v
Server validates upload request
    |
    v
Server checks file policy
    |
    v
Private object storage
    |
    v
Metadata stored in database
```

Requirements:

- strict type and size validation
- generated storage identifiers
- private storage
- ownership metadata
- controlled access
- deletion support
- image-processing limits
- no trust in the original filename
- no public bucket by default

File uploads require a dedicated threat model before implementation.

---

## Analytics Architecture

Analytics should derive from trusted records.

The analytics layer should:

- consume validated source data
- use deterministic formulas
- remain independent from chart components
- expose clear typed results
- document metric definitions
- support recomputation
- handle edits and deletions
- preserve user and account boundaries

Charts visualize analytics.

They do not calculate them.

For early versions, calculate analytics on demand where performance is acceptable.

Introduce cached aggregates only after measuring a real need.

---

## AI Coaching Architecture

The AI coaching system should be built as a controlled pipeline.

```text
Authenticated request
    |
    v
Authorization
    |
    v
Relevant record selection
    |
    v
Data minimization
    |
    v
Structured context construction
    |
    v
AI provider request
    |
    v
Output validation
    |
    v
Safe presentation or storage
```

### AI Context Builder

The context builder is application-owned.

Its primary responsibility is to construct a trusted context for the AI using only authorized and verified application data.

The AI does not calculate authoritative application facts.

It receives deterministic facts produced by the application and uses them as the foundation for interpretation.

The context builder should:

- select only authorized records
- include only verified deterministic metrics
- distinguish deterministic facts from user interpretation
- minimize personal data
- include metric definitions where needed
- attach evidence references
- enforce context limits
- exclude secrets
- preserve structural and behavioral separation

### AI Output

AI output is interpretive, not authoritative.

It may explain verified facts, identify patterns, and generate coaching observations.

It must never replace, modify, or contradict deterministic application data.

AI output should be treated as untrusted.

### AI Evidence Model

Every meaningful AI observation should be traceable to deterministic application data.

The AI should be able to explain not only _what_ it concluded, but _why_ it reached that conclusion.

Important AI observations should be traceable to supporting data.

For example:

- **Observation:** The user exited winners earlier than planned.
- **Evidence:** 6 of the last 8 trades marked as plan-followed had exits before the recorded target.
- **Confidence:** Moderate.
- **Alternative explanation:** Targets may have changed during the trade but were not updated in the journal.

The application should separate:

- observation
- evidence
- confidence
- interpretation
- suggested reflection

This supports intellectual honesty and reduces unsupported psychological claims.

---

## External Providers

Every external provider should be wrapped behind an application-owned adapter.

Examples:

- authentication
- AI model
- object storage
- email
- monitoring
- background jobs

The application should not spread provider-specific logic throughout the codebase.

Each integration should document:

- purpose
- data sent
- data received
- retention
- security settings
- failure behavior
- cost controls
- replacement strategy

---

## Error Handling

Errors should be handled consistently.

### Client-Facing Errors

Should:

- explain what happened in plain language
- provide a useful next step
- avoid blame
- avoid technical internals
- include a support or correlation reference when useful

### Server Errors

Should:

- use typed or categorized error handling
- distinguish expected from unexpected failures
- log sufficient diagnostic context
- exclude secrets and highly sensitive content
- avoid exposing stack traces to users

### AI Failures

Should degrade safely.

Examples:

- show a retry option
- preserve the user's journal entry
- avoid fabricating a result
- state when analysis is unavailable
- avoid treating partial output as complete

---

## Observability

If a production issue cannot be explained from telemetry, the observability system is incomplete.

Observability should include:

- structured logs
- error monitoring
- performance monitoring
- security event logging
- AI usage and cost monitoring
- background job visibility
- correlation IDs
- provider health visibility

Logs must not contain:

- passwords
- API keys
- session tokens
- raw sensitive journal entries
- unnecessary full AI prompts
- another user's private records

Observability should help answer:

- What failed?
- Who was affected?
- When did it begin?
- Is it still happening?
- What changed?
- How expensive is the AI layer?
- Did authorization behave correctly?

---

## Testing Strategy

The architecture must support several testing layers.

### Unit Tests

Used for:

- financial calculations
- validation
- transformations
- authorization helpers
- AI context construction
- metric formulas

### Integration Tests

Used for:

- database behavior
- transactions
- authorization
- storage workflows
- provider adapters
- background jobs

### End-to-End Tests

Used for:

- authentication flows
- trade creation
- editing and deletion
- uploads
- analytics
- user isolation
- AI coaching flows
- account deletion

### Security Tests

Required for:

- cross-user access
- privilege escalation
- input abuse
- upload abuse
- prompt injection
- AI cross-user leakage
- rate limits
- exports
- account recovery

### AI Evaluations

Used for:

- evidence grounding
- structural versus behavioral separation
- unsupported claims
- financial-advice boundaries
- diagnostic language
- uncertainty
- prompt injection
- regression across model or prompt changes

---

## Transaction Boundaries

Operations that must succeed or fail together should use database transactions.

Examples:

- creating a trade with executions
- deleting a trade and related records
- updating trade data and derived state
- creating export metadata and job records
- applying account deletion state

Avoid distributed transactions.

Keep related state changes within one application and one database where practical.

---

## Caching

Caching should not be introduced by default.

Before adding caching, document:

- the measured performance problem
- cache ownership
- invalidation rules
- tenant isolation
- stale-data tolerance
- failure behavior

Private user data must never leak across cache keys.

Correctness is more important than premature optimization.

---

## Background Processing

A task belongs in the background when:

- it is slow
- it is expensive
- it is retryable
- it does not need to block the user
- it can be safely made idempotent

Every background job should define:

- owner
- input schema
- authorization context
- retry policy
- timeout
- idempotency behavior
- failure handling
- observability
- cost limit

---

## Deployment Model

Initial deployment should favor managed services and operational simplicity.

Expected deployment shape:

- managed web hosting
- managed PostgreSQL
- managed private object storage
- managed authentication
- managed error monitoring
- managed AI provider

Deployment requirements:

- isolated environments
- separate credentials
- reproducible builds
- CI checks
- secret management
- HTTPS
- production-safe configuration
- rollback capability
- backup and restore procedures

Specific providers should be selected through Architecture Decision Records.

---

## Environment Separation

The project should maintain:

- local development
- test
- staging when needed
- production

Rules:

- no production data in development
- no shared credentials
- no production secrets in local files
- synthetic test data
- environment-specific configuration
- production debug behavior disabled

---

## Scalability Strategy

Scale only in response to evidence.

### Initial Strategy

- modular monolith
- relational database
- on-demand analytics
- limited background jobs
- managed infrastructure
- no microservices

### Possible Future Evolution

Only if justified:

- cached analytics
- read replicas
- dedicated job workers
- separate AI orchestration service
- specialized search or vector infrastructure
- event-driven workflows
- service extraction

Every extraction should answer:

- What measured problem does this solve?
- Why can the modular monolith no longer handle it?
- What new operational burden does it introduce?
- How will consistency and authorization remain correct?

---

## Architecture Debt

No architecture remains perfect forever.

Temporary architectural compromises are acceptable when they help move the product forward.

However, every compromise must be intentional and documented.

Each architectural debt item should include:

- why the compromise exists
- why it is acceptable today
- what risk it introduces
- what conditions should trigger revisiting it
- the intended long-term solution

Examples include:

- temporary duplication between domains
- an overly large module awaiting decomposition
- synchronous work that should eventually become a background job
- analytics calculated on demand before introducing caching
- temporary provider-specific code awaiting abstraction

Architecture should become simpler as understanding improves.

Complexity should only increase when reality demonstrates that it is necessary.

Every architectural shortcut should have an owner and a plan—not become permanent by accident.

---

## Repository Organization

The exact folder structure should evolve with implementation, but it should reflect domain boundaries.

A possible direction:

```text
src/
├── app/
├── components/
│   ├── ui/
│   └── product/
├── features/
│   ├── trades/
│   ├── executions/
│   ├── analytics/
│   ├── journal/
│   ├── reviews/
│   ├── uploads/
│   └── ai-coach/
├── shared/
│   ├── auth/
│   ├── db/
│   ├── security/
│   ├── validation/
│   ├── observability/
│   └── providers/
└── styles/
```

This is a direction, not a final commitment.

The folder structure should follow real product boundaries rather than forcing every feature into a rigid template.

---

## Shared Code

Shared code should be genuinely shared.

Good candidates:

- validation primitives
- authorization helpers
- financial types
- logging utilities
- provider interfaces
- design-system primitives
- test factories

Avoid a generic `utils` folder that becomes an unstructured collection of unrelated functions.

Prefer specific ownership and naming.

---

## Architecture Decision Records

Every major architectural choice should be documented in:

```text
docs/architecture/decisions/
```

The records currently in `docs/architecture/decisions/`:

- `0004-authentication-strategy.md`
- `0006-private-object-storage.md`
- `0008-supabase-integrated-infrastructure.md`
- `0009-row-level-security-strategy.md`
- `0010-money-representation.md`

Additional ADRs (for example, the AI provider) will be added here as those decisions are made; a number is assigned when the record is written.

Each record should include:

- status
- context
- decision
- alternatives
- consequences
- security implications
- review trigger

Architecture decisions should exist in documentation before they become deeply embedded in code.

---

## Open Questions

The following decisions require further evaluation:

- Which ORM or database-access approach should be used?
- Which AI provider and model strategy should be used?
- Should AI conversations be stored by default?
- Which data fields require field-level encryption?
- When should analytics be calculated on demand versus cached?
- Which background-job system is appropriate?
- What is the initial deployment provider?
- How should AI evidence references be represented?
- What should the account deletion and backup-expiry windows be?

These decisions should be resolved through product requirements, threat modeling, technical evaluation, and Architecture Decision Records.

---

## Architecture Review Checklist

Before introducing a major feature or dependency, ask:

- Which domain owns this?
- What data does it touch?
- Who is authorized to access it?
- Where is validation enforced?
- What is the source of truth?
- Is the logic deterministic or interpretive?
- Does the AI need access to this data?
- What could be abused?
- How will it be tested?
- How will failure be observed?
- What happens after deletion?
- Does this introduce unnecessary operational complexity?
- Does the decision require an ADR?

Every ADR should include:

- Why now?

---

## Guiding Principle

Keep the system simple enough to understand.

Keep the boundaries strong enough to trust.

Prefer clarity over cleverness.

Add complexity only when reality earns it.
