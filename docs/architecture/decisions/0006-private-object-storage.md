# 0006 — Private Object Storage Strategy

- **Status:** Draft — proposed, not approved. Documents the decision; do not implement until reviewed and approved.
- **Date:** 2026-07-11
- **Deciders:** Founder
- **Related:** [0008 Supabase Integrated Infrastructure], [0004 Authentication Strategy], [0009 Row-Level Security Strategy]

---

> This ADR explains *why* Edgebook AI adopts a private-by-default object storage strategy and what security boundaries it establishes. It is architectural reasoning, not a description of how uploads are wired.

## Context

Edgebook AI lets traders attach files to their records — primarily trade screenshots and chart images, and later export artifacts. These files are private, security-sensitive user content: a screenshot can reveal a user's positions, broker, size, and psychology. They deserve the same isolation guarantees as the structured records they belong to.

Object storage is a different trust surface from the database. Files are large binary blobs served over their own access paths, often directly to browsers, and object stores frequently default to conveniences — public buckets, guessable URLs, trusting client-supplied filenames — that quietly undermine isolation. A storage strategy must therefore be decided deliberately, not inherited from defaults.

Adopting Supabase (0008) gives us managed object storage that shares the same identity model as authentication (0004) and the database (0009). The decision here is how that storage is used so that a file is exactly as private, and as owned, as the record it belongs to.

## Decision Principles

- **Private by default.** User-generated files are inaccessible until access is deliberately and specifically granted.
- **Ownership before storage.** A file should never exist without a clearly defined owner in the application's domain model.
- **The database owns truth; storage holds bytes.** Ownership, identity, and metadata live in PostgreSQL; the object store holds content addressed by opaque keys.
- **Authorize every access, on its own terms.** Uploading and downloading are distinct operations and are authorized separately.
- **Never trust the client's file.** Filenames, content types, and sizes claimed by the browser are inputs to validate, not facts.
- **Storage authorization is not business authorization.** The store enforces *may this identity touch this object*; the application decides *what the product does* with it.
- **Defense in depth.** A file's privacy should survive a single mistake in any one layer.

## Decision

Adopt a **private-by-default object storage strategy** on Supabase Storage, in which every user-generated file is owned through PostgreSQL metadata and reachable only through short-lived, server-authorized access.

### Files are private by default

All user-generated files live in **private buckets**. There is no public bucket for user content. A file is never served by virtue of knowing its location; it is served only after the application authorizes the specific access and issues time-limited credentials for it. Any public exposure would be a deliberate, reviewed exception for genuinely public assets — never user content.

### Storage and PostgreSQL metadata

The object store holds bytes; **PostgreSQL holds the truth about those bytes**:

- Each stored object is addressed by a **generated, opaque storage key**, never by anything user-supplied.
- A metadata row in PostgreSQL records ownership, the owning record, the original filename (as data, not as the storage path), content type, size, and timestamps.
- Because ownership and access decisions read from this metadata, files inherit the same isolation guarantees as the rest of the database — including RLS (0009) on the metadata itself.

The object store is therefore never the authority on who owns a file. The database is.

### Metadata is authoritative

The object store is never queried to determine ownership.

All ownership decisions originate from PostgreSQL metadata.

If metadata and stored objects become inconsistent, the metadata remains authoritative until reconciliation occurs.

### Ownership chains for files

An uploaded file is owned the same way every other private record is — through an ownership chain rooted in the authenticated user:

```text
Authenticated User
    → Trading Account
        → Trade
            → Screenshot / Export Artifact
```

A screenshot is the user's because it belongs to their trade, which belongs to their account. Authorizing access to a file means verifying that chain, not merely checking that a row exists. The file's metadata carries the link; the application follows it.

### Upload authorization and download authorization are separate concerns

These are two different questions and are answered independently:

- **Upload authorization** asks: *may this identity attach a file to this target record, and does the file satisfy policy?* It validates the owning record, enforces type and size limits, and generates the storage key.
- **Download authorization** asks: *may this identity retrieve this specific object right now?* It re-verifies ownership of the file through its metadata and the ownership chain, every time, regardless of how the file was created.

Being allowed to upload never implies open-ended permission to download, and a valid download path is never a shortcut around ownership. Each access is authorized on its own terms.

### Signed URLs and time-limited access

Files are delivered either through short-lived **signed URLs** or an authenticated server-controlled delivery path:

- Access credentials are **time-limited** and scoped to a single object.
- A signed URL is minted only after the application authorizes that specific download; it is not a durable, shareable handle to the content.
- Expiry is short enough that a leaked URL has a small window and cannot substitute for authorization.

### Original filenames are never trusted

The filename the browser supplies is untrusted input:

- It never becomes the storage key or any part of a path.
- It is stored only as display metadata, sanitized for presentation.
- Content type and size are validated server-side rather than taken from client claims, and path-traversal or executable-disguise attempts in the name are neutralized by never using the name for storage or retrieval.

## Alternatives Considered

### A. Public buckets (with obscure URLs)

- Serve files from a public bucket and rely on unguessable paths.
- Rejected: "unguessable" is not authorization. URLs leak through history, referrers, and sharing; obscurity provides no isolation and no revocation. Incompatible with private user content.

### B. Direct browser access to storage with broad credentials

- Let the browser talk to storage using long-lived or broadly scoped credentials.
- Rejected: puts trust and credentials on the client, exactly where they cannot be protected. Any leaked credential becomes cross-user file access. The browser must never hold storage authority.

### C. Self-hosted / self-managed object storage

- Run our own storage (e.g. self-hosted MinIO or a raw S3 bucket we fully operate).
- Rejected for now: adds operational and security burden (patching, access policy, backups, encryption management) that a solo founder should not carry early, and forgoes the shared identity model Supabase provides. Reconsidered only if the umbrella decision (0008) changes.

### D. Best-of-breed managed storage decoupled from the platform (e.g. Cloudflare R2 / S3)

- Use a standalone managed object store behind our adapter.
- Viable and portable, but at this stage it reintroduces the identity-mapping and authorization glue that adopting the integrated platform (0008) was chosen to avoid. Kept as the natural migration target if storage needs diverge from the platform.

## Consequences

**Positive**

- User files are exactly as private and as owned as their parent records.
- Ownership and access decisions live in one place (PostgreSQL metadata), reviewable and testable.
- Time-limited, per-object access limits the blast radius of a leaked link.

**Negative / risks**

- Every file access requires an authorization step and URL minting — more work than serving a static public path, by design.
- Metadata and stored objects can drift (orphaned objects, dangling rows). Orphan detection and reconciliation become operational responsibilities and should be automated.
- Signed-URL expiry must be tuned: too long weakens security, too short harms UX.
- Storage remains a distinct trust surface that must be covered by isolation tests alongside the database.

## Security Implications

- **Cross-user file access** is the primary threat: it is prevented by database-owned ownership metadata, per-access authorization, and RLS on the metadata — not by URL secrecy.
- **Uploads** validate type and size server-side, generate opaque keys, record ownership metadata, and never trust the client-supplied filename or content type.
- **Downloads** re-verify ownership on every request and are served only via short-lived, single-object credentials.
- **Deletion** removes both the stored object and its metadata, cascades with the owning record's deletion, and is authorized like any other private mutation; retention and account-deletion behavior are explicit.
- **Metadata** is itself private data under RLS (0009); leaking metadata (filenames, links, sizes) is treated as seriously as leaking the file.
- **Least privilege:** service-role/storage-admin credentials stay server-side (consistent with 0009) and are never exposed to the browser.
- **Opaque keys:** object keys contain no user-identifiable information or business meaning — they leak nothing if observed and cannot be reverse-engineered into ownership or record identity.
- **Auditability:** file access and deletion are recorded as security-relevant events without leaking file contents or sensitive names.
- **Threat model:** file upload is a high-risk surface and requires its own dedicated threat model before implementation, covering malicious content, oversized uploads, and content-type abuse.

## Portability Strategy

- **All storage access goes through an application-owned `storage` adapter** exposing a provider-neutral interface (store, authorize-download, delete); feature code never calls a storage SDK directly.
- **Ownership and metadata live in PostgreSQL, not the provider**, so files can be re-pointed to another object store by migrating bytes and rewriting keys — without touching domain logic.
- **Storage authorization is application-owned**; changing providers changes how bytes are stored and signed, not who is allowed to access them.
- Because keys are opaque and generated by us, they are portable across providers by design.
- Replacing the storage provider should not require changes to feature modules or business logic.

## Review Triggers

Revisit this decision if:

- storage cost, egress, or performance becomes a measured problem;
- file access patterns (e.g. large exports, streaming, CDN needs) outgrow the platform's storage;
- the umbrella infrastructure decision (0008) is revisited;
- compliance or data-residency requirements constrain where files may live;
- a security review reveals a storage-authorization gap the current model cannot express;
- storage authorization and business authorization begin to overlap or duplicate one another.

## Guiding Principle

> The database owns who a file belongs to.
>
> Storage owns only the bytes.
>
> Every access is authorized on its own terms.
>
> Privacy is enforced by architecture — not obscurity.
