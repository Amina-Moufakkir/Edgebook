# Edgebook AI Security Baseline

**Status:** Draft  
**Last Reviewed:** 2026-07-12  
**Applies to:** Development, staging, production, AI features, infrastructure, and operational workflows  
**Owner:** Founder & Lead Engineer  
**Review Cadence:** At least quarterly and before every major release

> Security is a design constraint, not a deployment checklist.

## Purpose

This document defines the minimum security expectations for Edgebook AI.

It is a working baseline, not a guarantee of security and not a substitute for an independent professional review. Requirements should evolve with the product, architecture, user count, integrations, and sensitivity of stored data.

These requirements describe the **target security posture**. Some capabilities are delivered incrementally according to the product roadmap (for example, account deletion ships in the MVP while user-data export is Planned) — but scheduling a capability later never lowers the security bar for what already ships.

The project should use OWASP ASVS as the verification backbone and NIST SSDF as the secure-development process reference.

## Core Principles

- Deny by default.
- Apply least privilege.
- Authenticate users with a vetted system.
- Authorize every private operation on the server.
- Minimize collected data.
- Treat all external input as untrusted.
- Treat AI model output as untrusted.
- Keep secrets out of code, prompts, logs, fixtures, and screenshots.
- Fail securely.
- Prefer simple, testable controls over clever security.
- Document significant security decisions before implementing them.
- Never weaken a security control without explicit human approval.

---

## 1. Data Classification

Every stored or transmitted field must be classified before production use.

### Classification Levels

#### Public

Information intentionally available to anyone.

Examples:

- public marketing content
- public documentation
- non-sensitive product metadata

#### Internal

Information intended for the engineering or operations team.

Examples:

- internal architecture notes
- non-sensitive operational metrics
- feature flags

#### Confidential

Private user or business information.

Examples:

- email addresses
- trading account metadata
- trade history
- P&L records
- strategy notes
- uploaded screenshots

#### Highly Sensitive

Information whose exposure could cause substantial harm or reveal deeply personal behavior.

Examples:

- emotional journal entries
- AI coaching conversations
- authentication secrets
- password-reset tokens
- session tokens
- encryption keys
- security logs containing sensitive context

### Requirements

- [ ] Every new data type has a documented classification.
- [ ] Collection has a clear product purpose.
- [ ] Retention and deletion behavior are defined.
- [ ] Access is restricted according to classification.
- [ ] Confidential and highly sensitive data are excluded from logs by default.
- [ ] Highly sensitive data sent to AI providers is minimized and documented.

---

## 2. Secrets and Credentials

- [ ] No API keys, passwords, private keys, tokens, or connection strings appear in source code.
- [ ] `.env`, `.env.*`, private keys, certificates, and credential files are ignored before the first commit.
- [ ] A safe `.env.example` documents variable names without real values.
- [ ] Production secrets are stored in the deployment platform's secret store or a dedicated secrets manager.
- [ ] Development, staging, and production use separate credentials.
- [ ] Every human and service uses its own revocable identity.
- [ ] Secrets have an owner, purpose, and rotation procedure.
- [ ] Compromised or accidentally committed secrets are rotated immediately.
- [ ] Deleting a secret from Git does not count as remediation.
- [ ] Secret scanning runs locally or in CI.
- [ ] GitHub secret scanning and push protection are enabled when available.
- [ ] Secrets never appear in AI prompts, screenshots, logs, test fixtures, or issue descriptions.

---

## 3. Authentication

- [ ] Use a vetted authentication library or provider.
- [ ] Do not implement custom password hashing, token signing, or session management.
- [ ] MFA is required for administrative accounts and supported for users when practical.
- [ ] Authentication responses avoid revealing whether an account exists.
- [ ] Account recovery is treated as a security-sensitive authentication flow.
- [ ] Reset tokens are single-use, short-lived, and invalidated after use.
- [ ] Sessions expire and can be revoked.
- [ ] Password changes, account compromise, and global sign-out invalidate relevant sessions.
- [ ] Sensitive actions require recent authentication when appropriate.
- [ ] Cookie-based sessions use `Secure`, `HttpOnly`, and appropriate `SameSite` settings.
- [ ] Authentication endpoints have rate limiting and abuse detection.
- [ ] Sign-in failures and account recovery events are logged without exposing secrets.

If Edgebook AI ever stores passwords directly, use a modern password hashing algorithm with appropriate parameters. Prefer Argon2id where supported.

---

## 4. Authorization and Tenant Isolation

Authentication answers who the user is. Authorization answers what that user may access.

This is a critical security boundary for Edgebook AI.

- [ ] Every private read, create, update, delete, export, upload, analytics query, background job, and AI retrieval checks authorization on the server.
- [ ] Never trust a user ID, owner ID, role, or account ID supplied by the client.
- [ ] Every user-owned query is scoped to the authenticated user or tenant.
- [ ] Object-level authorization is verified for trades, executions, accounts, screenshots, reviews, exports, AI conversations, and insights.
- [ ] Authorization failures do not reveal whether another user's resource exists.
- [ ] Admin permissions are explicitly modeled and denied by default.
- [ ] Background workers enforce the same authorization boundaries as request handlers.
- [ ] AI retrieval can only receive records already authorized by deterministic application code.
- [ ] Row-level security is considered as defense in depth where supported.
- [ ] Cross-user access tests exist for every private resource.

### Required Cross-User Test Pattern

1. Create User A.
2. Create User B.
3. Create a private resource owned by User A.
4. Authenticate as User B.
5. Attempt to read, update, delete, export, or reference the resource.
6. Expect denial every time.

The AI model must never be treated as an authorization layer.

---

## 5. Data in Transit and at Rest

- [ ] Public traffic uses HTTPS only.
- [ ] Secure cookies are never sent over plaintext connections.
- [ ] Service-to-service traffic crossing a trust boundary is encrypted and authenticated.
- [ ] Production databases, object storage, logs, and backups use provider-supported encryption at rest.
- [ ] Field-level encryption is evaluated for highly sensitive fields based on a documented threat model.
- [ ] Encryption keys are stored separately from encrypted data.
- [ ] Key rotation and recovery procedures are documented.
- [ ] Backups are encrypted and access-controlled.
- [ ] TLS and certificate renewal are managed and monitored by the platform or operations process.

---

## 6. Data Minimization, Retention, Export, and Deletion

- [ ] Collect only data required for a documented product purpose.
- [ ] Do not collect sensitive fields merely because they may be useful later.
- [ ] Real user data never enters development or test environments.
- [ ] Tests and demos use synthetic data or the founder's intentionally provided test data.
- [ ] Each data category has a retention period or a documented reason for indefinite retention.
- [ ] Users can export their own data.
- [ ] Users can delete their account and private data.
- [ ] Deletion removes active production copies promptly.
- [ ] Deleted records expire from backups within a documented retention window.
- [ ] Backup restoration includes a process to reapply deletion records when required.
- [ ] Third-party deletion behavior is documented.
- [ ] Users are informed about realistic deletion timelines.
- [ ] Logs exclude or mask personal and highly sensitive data.

---

## 7. Input Validation, Output Handling, and Injection Prevention

- [ ] All untrusted input is validated on the server.
- [ ] Client-side validation exists for usability, not as a security boundary.
- [ ] Validation uses allowlists, clear schemas, length limits, and sensible numeric bounds.
- [ ] Database operations use parameterized queries or a safely configured ORM.
- [ ] Never construct SQL by concatenating user input.
- [ ] Output is contextually encoded before rendering.
- [ ] User-generated or AI-generated Markdown/HTML is sanitized with a vetted policy.
- [ ] Raw HTML rendering is avoided and requires explicit security review.
- [ ] URLs and redirects are validated.
- [ ] Spreadsheet exports defend against CSV/formula injection.
- [ ] File paths are never built directly from user input.
- [ ] State-changing browser requests include CSRF protection when the authentication architecture requires it.
- [ ] User-facing errors are useful but do not reveal stack traces, SQL, secrets, infrastructure details, or private resource existence.
- [ ] Client-visible errors include correlation IDs when operationally useful.

---

## 8. File Uploads

Trade screenshots are a high-risk feature and require a dedicated threat model.

- [ ] Enforce strict file-size limits.
- [ ] Validate the actual file content, not only the extension or browser-provided MIME type.
- [ ] Allow only explicitly approved formats.
- [ ] Generate server-controlled storage names.
- [ ] Do not trust or expose original filenames without sanitization.
- [ ] Store uploads outside executable application paths.
- [ ] Storage buckets are private by default.
- [ ] Access uses short-lived signed URLs or an authenticated proxy.
- [ ] Image metadata is stripped where practical.
- [ ] Uploaded files are scanned when feasible.
- [ ] Image processing uses maintained libraries and resource limits.
- [ ] Decompression bombs and oversized dimensions are rejected.
- [ ] Users can access only their own uploaded files.
- [ ] Deleting a trade or account applies the documented file-retention policy.
- [ ] Upload failures do not leak storage configuration details.

---

## 9. Business Logic and Abuse Prevention

- [ ] Financial calculations run on trusted server-side data.
- [ ] Impossible or contradictory trade states are rejected.
- [ ] Quantity, price, time, fees, and account values have documented bounds.
- [ ] Workflow restrictions are enforced server-side, not merely hidden in the UI.
- [ ] Duplicate requests cannot create unintended duplicate trades, charges, exports, or AI jobs.
- [ ] Use idempotency where duplicate execution would cause harm.
- [ ] Expensive AI, upload, analytics, and export operations have quotas and rate limits.
- [ ] Cost alerts and hard budgets exist for third-party APIs.
- [ ] Public identifiers are non-sequential when enumeration would create risk.
- [ ] Analytics and AI summaries are updated correctly after edits and deletions.
- [ ] Abuse cases are tested alongside normal workflows.

---

## 10. AI Application Security

- [ ] User text, uploaded content, retrieved records, and external content are treated as untrusted data.
- [ ] Prompt injection is included in the threat model.
- [ ] System instructions are never assumed to be secret.
- [ ] Prompts contain no secrets.
- [ ] Records are authorized before they enter model context.
- [ ] The model cannot request or retrieve another user's data.
- [ ] Model output is untrusted and validated before rendering or using it in tools, code, databases, or workflows.
- [ ] The model has no unrestricted database, filesystem, network, or administrative access.
- [ ] Every AI tool call independently enforces authorization and input validation.
- [ ] High-impact actions require deterministic application checks and explicit user confirmation where appropriate.
- [ ] The AI cannot override financial-safety, mental-health, privacy, or tenant-isolation boundaries.
- [ ] Provider retention, training, regional processing, and deletion terms are reviewed and documented.
- [ ] Sensitive context sent to providers is minimized and redacted where possible.
- [ ] AI conversation retention and deletion rules are documented.
- [ ] Prompt and model changes are versioned.
- [ ] Regression tests cover prompt injection, cross-user leakage, fabricated evidence, unsafe financial guidance, unsupported psychological claims, and unbounded usage.
- [ ] Failures degrade safely: the system admits uncertainty instead of fabricating an answer.
- [ ] Rate limits, token limits, timeouts, and cost controls prevent unbounded consumption.

---

## 11. Dependencies and Software Supply Chain

- [ ] The lockfile is committed.
- [ ] CI uses reproducible installs.
- [ ] New dependencies require a documented need.
- [ ] Package names are checked carefully to prevent typosquatting.
- [ ] Maintenance status, ownership, release history, and security posture are reviewed.
- [ ] Dependency vulnerability scanning runs automatically.
- [ ] Dependency changes receive review.
- [ ] Security updates are triaged and applied through a defined process.
- [ ] Unused dependencies are removed.
- [ ] Build scripts and install hooks are treated as executable code.
- [ ] Third-party SDK permissions and transmitted data are documented.
- [ ] Vulnerability scan results are treated as signals, not proof that the system is secure.

---

## 12. Infrastructure and Configuration

- [ ] Development, staging, and production are separated.
- [ ] Environments do not share production data or credentials.
- [ ] Production debug mode is disabled.
- [ ] Verbose errors and source maps are not publicly exposed unless intentionally configured.
- [ ] Databases and internal services are not publicly exposed unless strictly necessary.
- [ ] Firewalls, private networking, and platform access controls are configured.
- [ ] Storage buckets are private by default and periodically verified.
- [ ] Default credentials are removed.
- [ ] Security headers are configured and tested, including CSP, HSTS, and `X-Content-Type-Options`.
- [ ] CORS is explicitly configured and not broadly permissive.
- [ ] Administrative consoles are restricted.
- [ ] Infrastructure configuration is reviewed before production changes.

---

## 13. Logging, Monitoring, and Privacy

- [ ] Log security-relevant events such as sign-ins, failures, account recovery, permission changes, exports, deletions, and admin actions.
- [ ] Logs record event type, time, actor, outcome, and correlation ID where practical.
- [ ] Logs exclude passwords, tokens, full prompts containing sensitive content, and raw private journal entries.
- [ ] Production log access is restricted and audited.
- [ ] Logs are protected from tampering.
- [ ] Suspicious activity alerts exist for authentication abuse, unusual exports, repeated authorization failures, and cost spikes.
- [ ] Alert failures are detectable.
- [ ] Log retention is documented.
- [ ] Monitoring vendors and transmitted data are included in the data inventory.

---

## 14. Backups and Recovery

- [ ] Backups are automatic.
- [ ] Backups are encrypted and access-controlled.
- [ ] Restore procedures are tested on a schedule.
- [ ] Recovery Point Objective and Recovery Time Objective are documented when production use begins.
- [ ] Backup restoration preserves tenant isolation.
- [ ] Restored data is reconciled with deletion requests where required.
- [ ] Backup failures generate alerts.

---

## 15. Privacy, Legal, and Trust

- [ ] Maintain a data inventory and third-party processor list.
- [ ] Publish a plain-language privacy policy before collecting real user data.
- [ ] Publish terms of service before public use.
- [ ] Explain what is collected, why, where it is processed, who receives it, and how long it is retained.
- [ ] Do not claim legal or security compliance without verifying scope and implementation.
- [ ] Determine applicable laws based on actual users, geography, data, business model, and integrations.
- [ ] Security and privacy claims in marketing match reality.
- [ ] Obtain qualified legal advice before making regulatory claims or entering high-risk markets.

---

## 16. Secure Development Workflow

For every security-relevant feature:

1. Define the feature and data involved.
2. Classify the data.
3. Create or update the threat model.
4. Add security acceptance criteria.
5. Implement the smallest safe design.
6. Add tests for normal, failure, and abuse cases.
7. Run automated checks.
8. Perform manual verification.
9. Document residual risk.
10. Obtain explicit approval before weakening any control.

### Minimum Automated Checks

- lint
- type checking
- unit tests
- integration tests
- cross-user authorization tests
- secret scanning
- dependency scanning
- production build

### Professional Review Trigger

Obtain an independent professional security review before:

- handling meaningful volumes of real user data
- launching paid plans
- adding brokerage or financial-account integrations
- allowing high-impact AI actions
- expanding administrative access
- processing data classified as highly sensitive at scale
- making strong security or compliance claims

---

## 17. Reference Frameworks

This baseline should be progressively mapped to:

- OWASP Application Security Verification Standard, target Level 2 where applicable
- OWASP Cheat Sheet Series
- OWASP Top 10 for Web Applications
- OWASP Top 10 for LLM and Generative AI Applications
- NIST Secure Software Development Framework
- Provider-specific security guidance for authentication, hosting, database, storage, and AI services

---

## Security Decision Rule

When uncertain:

1. Stop.
2. Identify the asset and threat.
3. Choose the simplest control that fails safely.
4. Test the control.
5. Document the decision.
6. Escalate when the risk exceeds current expertise.
