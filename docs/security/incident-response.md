# Edgebook AI Incident Response Plan

**Status:** Initial plan  
**Owner:** Founder & Lead Engineer  
**Purpose:** Provide a calm, repeatable response when a security or privacy incident is suspected

> During an incident, protect users first, preserve evidence, communicate accurately, and avoid making irreversible changes without understanding their impact.

This plan does not replace legal counsel, a professional incident-response team, cyber-insurance support, or regulatory advice.

## 1. What Counts as an Incident?

Examples include:

- suspected or confirmed unauthorized access
- cross-user data exposure
- leaked secret or credential
- account takeover
- malicious file upload or code execution
- database or object-storage exposure
- lost or stolen administrator device
- compromised dependency or deployment pipeline
- AI response exposing another user's data
- prompt injection causing unauthorized action
- accidental logging of private journal or AI content
- unauthorized export
- ransomware or destructive activity
- major denial of service or uncontrolled API cost
- deletion or corruption of user data
- third-party provider breach affecting Edgebook AI
- privacy-policy or retention failure

A report does not need to be proven before it is treated seriously.

---

## 2. Severity Levels

### SEV-1 — Critical

Examples:

- active broad data breach
- cross-user access affecting multiple users
- compromised production administrator or signing key
- active code execution
- destructive attack
- confirmed exposure of highly sensitive data
- AI or tool capable of unauthorized high-impact actions

**Response:** Immediate. Stop normal work.

### SEV-2 — High

Examples:

- limited confirmed account takeover
- exposed production secret with meaningful access
- unauthorized access limited to one user
- serious vulnerability with likely exploitation
- private storage misconfiguration without confirmed access
- prompt injection causing unauthorized information disclosure

**Response:** Begin immediately and escalate.

### SEV-3 — Medium

Examples:

- suspicious behavior without confirmed access
- vulnerability with meaningful prerequisites
- sensitive data accidentally logged internally
- temporary failure of a security control
- dependency vulnerability with no known exploitation

**Response:** Investigate promptly.

### SEV-4 — Low

Examples:

- failed attack with controls working
- low-impact misconfiguration
- informational scanner finding
- policy or documentation gap

**Response:** Track and remediate through normal engineering work.

When uncertain, assign the higher severity until evidence supports lowering it.

---

## 3. Incident Roles

At the current project stage, one person may hold multiple roles. Name them explicitly during an incident.

### Incident Commander

Coordinates response, sets priorities, records decisions, and prevents fragmented action.

### Technical Lead

Investigates systems, contains the issue, and implements remediation.

### Communications Lead

Coordinates accurate messages to users, providers, and stakeholders.

### Privacy or Legal Contact

Determines notification, contractual, and regulatory obligations.

### External Security Contact

Provides independent expertise when the incident exceeds internal capability.

### Incident Record

- Incident ID:
- Severity:
- Incident Commander:
- Technical Lead:
- Communications Lead:
- Legal or privacy contact:
- External contact:
- Start time:
- Current status:

---

## 4. Immediate Response: First 15 Minutes

1. Stay calm.
2. Open an incident record.
3. Record the current time and reporter.
4. Capture the initial evidence.
5. Assign provisional severity.
6. Stop further exposure when a safe containment step is obvious.
7. Preserve logs and relevant system state.
8. Avoid deleting evidence.
9. Do not speculate publicly.
10. Contact professional help for SEV-1 or unfamiliar high-risk situations.

### Initial Questions

- What was observed?
- Who reported it?
- When did it begin?
- Is it still happening?
- Which users, systems, data, and providers may be affected?
- What is the worst credible impact?
- What safe containment action can be taken now?
- What evidence must be preserved?

---

## 5. Containment

Containment limits harm without destroying evidence.

Possible actions:

- disable affected feature
- revoke or rotate compromised credentials
- invalidate affected sessions
- temporarily disable an account
- remove public access from storage
- block malicious IPs or tokens
- disable a vulnerable integration
- disable AI tools or retrieval
- place the application in maintenance mode
- reduce privileges
- pause exports
- pause background jobs
- roll back a release when safe

### Containment Rules

- [ ] Record every action and timestamp.
- [ ] Preserve the old configuration before changing it.
- [ ] Do not rotate or delete evidence until necessary copies are secured.
- [ ] Avoid broad destructive commands.
- [ ] Confirm that containment did not create additional exposure.
- [ ] Verify whether the attacker retains another access path.

---

## 6. Evidence Preservation

Preserve only what is necessary and protect it as sensitive.

Potential evidence:

- authentication logs
- authorization failure logs
- application and infrastructure logs
- deployment history
- Git history
- audit events
- database access records
- storage access logs
- AI request and tool-call metadata
- relevant provider alerts
- affected request IDs and correlation IDs
- configuration snapshots
- timestamps and screenshots
- reports from affected users

### Evidence Requirements

- [ ] Store evidence in a restricted location.
- [ ] Record who collected it and when.
- [ ] Do not paste sensitive evidence into general AI chats.
- [ ] Do not expose secrets while sharing evidence.
- [ ] Preserve original files when creating redacted copies.
- [ ] Follow professional or legal instructions when provided.

---

## 7. Investigation

### Establish a Timeline

Document:

- earliest known indicator
- first unauthorized action
- detection time
- containment actions
- credential rotations
- user impact
- recovery steps
- communications

### Determine Root Cause

Ask:

- What failed?
- Was the failure technical, operational, or both?
- Was the control missing, misconfigured, bypassed, or untested?
- Was AI involved?
- Did authorization fail?
- Did a third party contribute?
- Was sensitive data logged or retained elsewhere?
- Did the attacker establish persistence?
- Was the issue exploited or only theoretically possible?

### Determine Scope

Identify:

- affected users
- affected records
- affected environments
- exposed fields
- access duration
- actions taken
- downstream providers
- backups or exports involved

Do not claim certainty until supported by evidence.

---

## 8. Special Playbooks

### A. Secret or API Key Exposure

1. Determine the secret and permissions.
2. Revoke or rotate it immediately.
3. Review provider logs for misuse.
4. Replace it in every environment.
5. Search repository history, logs, CI, issues, and AI sessions.
6. Remove exposed copies after preserving required evidence.
7. Add or improve secret scanning.
8. Document cost or data impact.

Deleting the secret from Git is not sufficient.

---

### B. Cross-User Data Exposure

1. Disable the affected route, query, export, job, or AI retrieval path.
2. Identify affected resource types.
3. Test whether exposure is ongoing.
4. Determine which users and records were accessible.
5. Preserve request and authorization logs.
6. Correct the authorization boundary.
7. Add cross-user regression tests.
8. Review similar resources for the same pattern.
9. Assess notification obligations with qualified help.

---

### C. Account Takeover

1. Invalidate affected sessions.
2. suspend or protect the account when appropriate.
3. Reset compromised credentials.
4. Review account recovery and MFA events.
5. Identify viewed, changed, exported, or deleted data.
6. Restore safe account state.
7. Notify the affected user through a trusted channel.
8. Investigate broader credential stuffing or session theft.

---

### D. Malicious Upload

1. Disable access to the file.
2. Preserve a restricted evidence copy.
3. identify all processing and storage paths it touched.
4. inspect related logs.
5. check for code execution or resource exhaustion.
6. isolate affected systems if needed.
7. patch validation or processing.
8. test neighboring upload paths.
9. add regression fixtures safely.

---

### E. AI Cross-User Leakage or Prompt Injection

1. Disable the affected AI feature or tool.
2. preserve request metadata and relevant authorized context.
3. identify whether the leak came from retrieval, prompt construction, provider behavior, logs, or model output.
4. verify whether another user's data actually entered context.
5. revoke AI tool access if unauthorized action was possible.
6. inspect all users and requests potentially affected.
7. fix deterministic authorization outside the model.
8. add adversarial regression tests.
9. review provider retention and deletion procedures.
10. assess user notification requirements.

Never attempt to fix a real authorization failure only by strengthening the prompt.

---

### F. Public Database or Storage Exposure

1. Remove public access.
2. rotate credentials.
3. preserve access logs.
4. determine exposure duration.
5. identify accessible objects or tables.
6. check for downloads, enumeration, or modification.
7. verify backups and replicas.
8. correct infrastructure policy.
9. add automated configuration checks.

---

### G. Dependency or Build-Pipeline Compromise

1. stop deployments.
2. identify affected package, action, or build step.
3. preserve lockfiles and build logs.
4. rotate secrets available to the pipeline.
5. rebuild from a trusted source and known-good commit.
6. inspect produced artifacts.
7. remove the compromised dependency.
8. assess whether production or developer machines were affected.
9. notify relevant providers and users when required.

---

## 9. Eradication

After containment:

- remove malicious access
- patch the vulnerability
- remove persistence
- rotate affected secrets
- update dependencies
- correct configuration
- repair authorization
- clean or rebuild affected systems
- add automated regression tests
- review similar code and infrastructure
- validate that the root cause is removed

A temporary block is not eradication.

---

## 10. Recovery

- [ ] Restore service gradually.
- [ ] Use known-good code and configuration.
- [ ] Verify authentication and authorization.
- [ ] Verify logs and alerts.
- [ ] Verify backups and data integrity.
- [ ] Test affected user flows.
- [ ] Watch for recurrence.
- [ ] Increase monitoring temporarily.
- [ ] Confirm cost and provider usage are normal.
- [ ] Communicate service status accurately.

For serious incidents, obtain independent confirmation before declaring recovery complete.

---

## 11. Communication

### Principles

- Be accurate.
- Be timely.
- Be plainspoken.
- Do not minimize.
- Do not speculate.
- Do not blame users.
- Do not disclose details that create additional risk.
- Coordinate legal and regulatory advice.

### Internal Update Template

- incident ID:
- severity:
- current status:
- confirmed facts:
- unknowns:
- affected systems:
- containment completed:
- next actions:
- next update time:

### User Notification Draft Structure

- what happened
- when it happened
- what information may be affected
- what Edgebook AI has done
- what the user should do
- how to obtain help
- when more information will be provided

Do not use this template without reviewing applicable legal and contractual obligations.

---

## 12. Legal, Regulatory, and Provider Escalation

- [ ] Identify user locations and affected data.
- [ ] Contact qualified legal or privacy counsel when notification may be required.
- [ ] Review provider contracts and reporting windows.
- [ ] Notify insurance provider when applicable.
- [ ] Preserve evidence according to professional advice.
- [ ] Track every external notification and deadline.

Do not guess about breach-notification obligations.

---

## 13. Post-Incident Review

Hold a blameless post-incident review after containment and recovery.

### Questions

- What happened?
- Why was it possible?
- Why was it not prevented?
- Why was it not detected sooner?
- What reduced harm?
- What made response harder?
- Which assumptions were wrong?
- Which neighboring systems may share the weakness?
- Which documentation, tests, alerts, or training must change?

### Required Outputs

- [ ] root-cause analysis
- [ ] impact summary
- [ ] complete timeline
- [ ] remediation owners and deadlines
- [ ] regression tests
- [ ] updated threat model
- [ ] updated security baseline
- [ ] updated release checklist
- [ ] updated incident plan
- [ ] decision on professional follow-up review

---

## 14. Incident Log Template

### Summary

- Incident ID:
- Title:
- Severity:
- Status:
- Started:
- Detected:
- Contained:
- Recovered:
- Closed:

### Confirmed Impact

### Potential Impact

### Affected Users

### Affected Data

### Timeline

| Time | Event | Source | Decision or action |
| ---- | ----- | ------ | ------------------ |
|      |       |        |                    |

### Containment Actions

### Evidence Locations

### Root Cause

### Remediation

### Communications

### Residual Risk

### Follow-Up Owners

| Action | Owner | Priority | Due date | Status |
| ------ | ----- | -------- | -------- | ------ |
|        |       |          |          |        |

---

## 15. Preparation Checklist

Complete before public launch.

- [ ] Incident contacts are current.
- [ ] Authentication-provider emergency procedures are known.
- [ ] Hosting and database support paths are known.
- [ ] Secret-rotation procedure is documented and tested.
- [ ] Session-revocation procedure is documented and tested.
- [ ] Feature-disable or maintenance-mode procedure exists.
- [ ] Backup restoration has been tested.
- [ ] Log access works.
- [ ] User communication channel exists.
- [ ] External security and legal contacts are identified.
- [ ] A tabletop incident exercise has been completed.

---

## Guiding Rule

Protect users first.

Preserve evidence.

Contain carefully.

Communicate truthfully.

Fix the system, not merely the symptom.
