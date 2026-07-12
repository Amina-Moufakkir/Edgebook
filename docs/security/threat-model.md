# Edgebook AI Threat Model

**Status:** Living document  
**Method:** Lightweight, feature-driven threat modeling  
**Owner:** Founder & Lead Engineer  
**Review triggers:** New sensitive feature, new integration, architecture change, major release, or security incident

> A threat model is not a list of everything that could go wrong. It is a structured way to decide what must be protected, from whom, and how we will verify the protection.

## 1. System Summary

Edgebook AI is an AI coaching platform for traders. It may store:

- user identity and account information
- trading accounts
- trades and executions
- P&L and analytics
- strategies and trading rules
- screenshots
- journal entries
- emotional and behavioral reflections
- AI conversations and generated insights

The system is expected to include:

- web client
- server application
- authentication provider
- relational database
- private object storage
- background jobs
- AI provider
- logging and monitoring services
- deployment platform

Update this section as architecture decisions become final.

---

## 2. Security Objectives

Edgebook AI must protect:

### Confidentiality

Private user data must be visible only to the authorized user and explicitly authorized services.

### Integrity

Trades, journal entries, analytics, AI context, and account settings must not be altered without authorization.

### Availability

The system should resist abuse that makes the application unavailable or creates uncontrolled third-party cost.

### Privacy

Sensitive journal and coaching content should be collected, retained, shared, and deleted according to documented expectations.

### Safety and Trust

The AI should not fabricate evidence, expose another user's data, provide prohibited financial guidance, or make unsupported psychological claims.

---

## 3. Assets

| Asset                              | Classification                   | Impact if compromised                              |
| ---------------------------------- | -------------------------------- | -------------------------------------------------- |
| Session tokens                     | Highly Sensitive                 | Account takeover                                   |
| Authentication and reset tokens    | Highly Sensitive                 | Account takeover                                   |
| Trade history and P&L              | Confidential                     | Financial privacy harm                             |
| Journal entries                    | Highly Sensitive                 | Personal and emotional privacy harm                |
| AI conversations                   | Highly Sensitive                 | Personal privacy and trust harm                    |
| Trade screenshots                  | Confidential or Highly Sensitive | May expose account details or personal information |
| User email and profile             | Confidential                     | Identity and phishing risk                         |
| Encryption keys and API secrets    | Highly Sensitive                 | Broad system compromise                            |
| Authorization rules                | Highly Sensitive                 | Cross-user data exposure                           |
| Analytics and derived insights     | Confidential                     | Misleading decisions or privacy harm               |
| Logs                               | Internal to Highly Sensitive     | May expose operations or private data              |
| Backups                            | Highly Sensitive                 | Bulk data exposure                                 |
| AI prompts and evaluation datasets | Internal to Highly Sensitive     | Product leakage or private-data exposure           |

---

## 4. Actors

### Legitimate Actors

- authenticated trader
- founder or authorized administrator
- background worker
- authentication provider
- database and storage services
- AI provider
- monitoring and deployment services

### Threat Actors

- unauthenticated external attacker
- malicious authenticated user
- compromised user account
- compromised administrator
- malicious or compromised dependency
- compromised third-party provider
- automated bot
- attacker using prompt injection
- accidental insider or developer mistake

---

## 5. Trust Boundaries

Document every point where data crosses between systems with different trust levels.

Expected boundaries:

1. Browser to Edgebook AI server
2. Server to authentication provider
3. Server to database
4. Server to private object storage
5. Server to AI provider
6. Server to monitoring provider
7. Server to background job system
8. Development environment to cloud services
9. Administrator to production environment

For each boundary, document:

- authentication
- authorization
- encryption
- data transmitted
- retention
- failure behavior
- logging
- rate limits

---

## 6. Entry Points

- registration and sign-in
- account recovery
- API routes and server actions
- trade forms
- filters and search
- screenshot uploads
- exports
- account deletion
- AI chat and coaching prompts
- background job payloads
- webhooks
- administrative tools
- third-party callbacks
- support workflows

---

## 7. Global Threats and Controls

### T1 — Cross-User Data Access

**Scenario:** User B changes an identifier or manipulates a request to access User A's trades, screenshots, journal entries, AI conversations, or exports.

**Impact:** Critical confidentiality breach.

**Controls:**

- server-side ownership checks
- authenticated-user scoping in every query
- deny-by-default authorization
- non-enumerable public identifiers where useful
- row-level security as defense in depth
- cross-user tests for every private resource

**Verification:**

- User B cannot read, update, delete, export, or reference User A's resource.
- Background jobs and AI retrieval enforce the same boundary.

---

### T2 — Account Takeover

**Scenario:** An attacker obtains credentials, session tokens, or reset links.

**Impact:** Full access to private user data.

**Controls:**

- vetted authentication provider
- secure session cookies
- MFA for administrators
- rate limiting
- account-enumeration resistance
- short-lived, single-use recovery tokens
- session revocation
- suspicious login monitoring

**Verification:**

- reset tokens cannot be reused
- old sessions are invalidated after security-sensitive events
- repeated login attempts are throttled

---

### T3 — Secret Exposure

**Scenario:** A secret is committed, logged, pasted into an AI tool, or exposed in an error.

**Impact:** Provider compromise, data access, or financial loss.

**Controls:**

- secret manager
- `.gitignore`
- local and CI secret scanning
- GitHub push protection where available
- redacted logs
- separate environment credentials
- documented rotation procedure

**Verification:**

- test secret is blocked by scanning
- no production secrets exist in repository history or logs

---

### T4 — Malicious or Oversized File Upload

**Scenario:** An attacker uploads executable, malformed, deceptive, or resource-exhausting content.

**Impact:** Code execution, denial of service, storage abuse, or data exposure.

**Controls:**

- allowlisted formats
- file signature validation
- strict size and dimension limits
- private storage
- generated storage names
- safe image processing
- malware scanning when feasible
- signed URLs
- authorization on every file access

**Verification:**

- disallowed types are rejected
- oversized and malformed files fail safely
- one user cannot access another user's file

---

### T5 — SQL or Command Injection

**Scenario:** User input alters database queries or system commands.

**Impact:** Data theft, corruption, or system compromise.

**Controls:**

- parameterized queries
- safely configured ORM
- schema validation
- no shell command construction from user input
- least-privileged database account

**Verification:**

- injection payload tests do not alter query behavior
- application database role cannot perform unnecessary administrative operations

---

### T6 — Stored or Reflected XSS

**Scenario:** User or AI content executes script in another user's browser.

**Impact:** Session theft, data theft, or malicious actions.

**Controls:**

- framework output encoding
- sanitized Markdown/HTML
- no raw HTML by default
- Content Security Policy
- URL validation

**Verification:**

- script payloads render as inert content or are removed
- AI output cannot inject executable markup

---

### T7 — CSRF or Unauthorized State Change

**Scenario:** A browser automatically submits an authenticated state-changing request.

**Impact:** Unwanted data changes or account actions.

**Controls:**

- appropriate `SameSite` cookie policy
- CSRF tokens where required
- origin validation
- reauthentication for sensitive actions

**Verification:**

- cross-origin state-changing requests fail

---

### T8 — Business-Logic Manipulation

**Scenario:** A user submits impossible trades, manipulates quantities, bypasses workflow rules, or duplicates requests.

**Impact:** Incorrect analytics, misleading coaching, uncontrolled cost, or data corruption.

**Controls:**

- server-side invariants
- numeric and temporal bounds
- idempotency
- deterministic financial calculations
- server-enforced workflow rules

**Verification:**

- invalid states are rejected
- repeated requests do not create duplicates
- edits and deletions correctly update analytics

---

### T9 — Prompt Injection

**Scenario:** User content or retrieved text attempts to override the AI's instructions, reveal hidden data, call tools, or bypass safety rules.

**Impact:** Privacy breach, unsafe output, unwanted tool action, or cost abuse.

**Controls:**

- treat all content as untrusted data
- deterministic authorization before retrieval
- no secrets in prompts
- tool allowlists
- independent authorization in every tool
- output validation
- explicit confirmation for high-impact actions
- prompt-injection regression tests

**Verification:**

- malicious instructions cannot retrieve another user's data
- model output cannot directly execute privileged actions
- system behaves safely when the model follows malicious content

---

### T10 — AI Sensitive-Information Disclosure

**Scenario:** The model reveals another user's records, hidden prompt content, provider data, or confidential context.

**Impact:** Privacy and trust failure.

**Controls:**

- per-user retrieval boundaries
- minimized context
- redaction
- no secrets in system prompts
- output filters for known sensitive patterns
- provider terms review
- tenant-isolation testing

**Verification:**

- adversarial prompts cannot surface cross-user content
- responses cite only records authorized for the current user

---

### T11 — AI Excessive Agency

**Scenario:** The AI is given more tools or permissions than required and performs damaging actions.

**Impact:** Data modification, deletion, external actions, or privilege escalation.

**Controls:**

- read-only AI by default
- least-privileged tools
- deterministic server authorization
- explicit user confirmation
- no unrestricted database or network access
- action logs and rate limits

**Verification:**

- the model cannot bypass application permissions
- sensitive actions cannot occur solely because the model requested them

---

### T12 — AI Misinformation or Unsupported Psychological Claims

**Scenario:** The AI fabricates evidence, confuses outcome with decision quality, or diagnoses a user.

**Impact:** Harmful coaching, lost trust, or unsafe user behavior.

**Controls:**

- evidence-linked responses
- uncertainty requirements
- separation of structure and psychology
- no diagnosis
- evaluation datasets
- human-readable explanation of supporting observations
- safe fallback when evidence is insufficient

**Verification:**

- the AI states uncertainty when data is insufficient
- claims are traceable to authorized records
- the AI does not diagnose mental-health conditions

---

### T13 — Unbounded AI or API Consumption

**Scenario:** A user or bot triggers excessive model, export, upload, or analytics usage.

**Impact:** Denial of service or significant financial cost.

**Controls:**

- authentication
- rate limits
- quotas
- token and timeout limits
- cost budgets and alerts
- request deduplication
- abuse monitoring

**Verification:**

- limits trigger predictably
- cost spikes generate alerts
- failure does not cascade through the system

---

### T14 — Dependency or Supply-Chain Compromise

**Scenario:** A malicious or compromised package, build script, action, or SDK enters the project.

**Impact:** Secret theft, code execution, or production compromise.

**Controls:**

- dependency review
- lockfile
- reproducible CI
- minimal dependencies
- vulnerability scanning
- pinned GitHub Actions by immutable reference where practical
- controlled update process

**Verification:**

- dependency additions are visible in review
- unused packages are removed
- install and build scripts are examined for risky behavior

---

### T15 — Logging or Monitoring Leakage

**Scenario:** Private data, prompts, tokens, or screenshots appear in logs or third-party monitoring.

**Impact:** Secondary data breach.

**Controls:**

- structured logging
- field allowlists
- redaction
- restricted log access
- retention limits
- processor inventory

**Verification:**

- test workflows do not place secrets or raw sensitive content in logs

---

### T16 — Backup Exposure or Failed Recovery

**Scenario:** Backups are stolen, unavailable, or restored incorrectly.

**Impact:** Bulk data breach, data loss, or reappearance of deleted data.

**Controls:**

- encrypted backups
- restricted access
- tested restores
- deletion reconciliation
- recovery objectives

**Verification:**

- scheduled restore test succeeds
- restored environment preserves authorization boundaries

---

## 8. Feature Threat Model Template

Copy this section for each security-sensitive feature.

### Feature

`<Feature name>`

### Purpose

`<Why the feature exists>`

### Data Involved

| Data | Classification | Source | Destination | Retention |
| ---- | -------------- | ------ | ----------- | --------- |
|      |                |        |             |           |

### Actors

- authorized:
- unauthorized:
- third parties:

### Entry Points

-

### Trust Boundaries

-

### Abuse Cases

1.
2.
3.

### Security Requirements

- [ ]
- [ ]
- [ ]

### Controls

| Threat | Preventive control | Detective control | Recovery control |
| ------ | ------------------ | ----------------- | ---------------- |
|        |                    |                   |                  |

### Verification

- automated tests:
- manual tests:
- monitoring:
- release gate:

### Residual Risk

`<What remains possible after controls?>`

### Decision

- accepted
- mitigated
- transferred
- avoided
- requires professional review

### Owner and Review Date

- owner:
- last reviewed:
- next review:

---

## 9. Initial Feature Threat Models Required

Create dedicated sections or separate files before implementing:

- authentication and account recovery
- trade and execution CRUD
- screenshot uploads
- exports
- account deletion
- AI coaching and retrieval
- weekly summaries
- administrator access
- analytics aggregation
- third-party webhooks or integrations

---

## 10. Risk Rating

Use a simple initial model.

### Likelihood

- Low: difficult, rare, or requires privileged access
- Medium: plausible with moderate effort
- High: easy, automatable, or exposed to all users

### Impact

- Low: minor inconvenience with limited exposure
- Medium: meaningful user or operational harm
- High: private-data exposure, account takeover, major cost, or data corruption
- Critical: broad compromise, cross-tenant breach, or serious user harm

Prioritize all High and Critical risks before release.

---

## 11. Review Rule

A feature cannot be called complete when:

- its trust boundaries are unknown
- its authorization model is unclear
- its data retention is undefined
- its abuse cases are untested
- the AI receives data without deterministic authorization
- residual High or Critical risk lacks explicit human acceptance
