# STEP 15 — AUDIT LOG REQUIREMENTS

## 15.1 Purpose

The audit system shall provide traceability for significant security, administrative, clinical-data, configuration, and business actions performed within the MedReminder system.

**Cross-Document Authority:** Step 4B defines authentication behavior that generates audit events. Step 19 defines security controls including retention policies. Step 14B defines adherence behavior that may involve auditable administrative actions. This document defines the audit log requirements to support accountability and traceability across all system functions.

The audit log shall answer, where applicable:
* Who performed the action
* What action occurred  
* What record/entity was affected
* When it occurred
* Whether it succeeded or failed
* Relevant correlation/request context

---

## 15.2 Audit Scope and Separation

The audit log is distinct from:
* **Application logs** — technical debugging and system monitoring
* **Notification history** — communication delivery events (Telegram/SMS/Voice)  
* **Adherence records** — patient medication response data

Audit records shall focus on significant user actions, system events, and configuration changes that require accountability, security monitoring, or regulatory traceability.

---

## 15.3 Audit Event Structure

Each audit event shall contain, as applicable:

| Field | Purpose |
|-------|---------|
| `audit_event_id` | Unique audit event identifier |
| `timestamp` | When the event occurred (UTC) |
| `actor_type` | Doctor, Admin, System |
| `actor_id` | Internal identifier of the actor |
| `action` | Action type using consistent internal classification |
| `entity_type` | Patient, Medication, Schedule, etc. |
| `entity_id` | Internal identifier of affected record |
| `result` | SUCCESS, FAILURE, or specific result code |
| `request_id` | Correlation identifier for related operations |
| `metadata` | Limited structured additional context |

**Change Summary:** For record modifications, audit events should include a minimal structured summary of changed fields without storing complete sensitive records.

---

# 15.4 Formal Audit Requirements

## Audit Event Creation and Structure

### AUD-001 — Audit Event Creation

The system shall create an audit event for significant actions that require accountability or traceability.

### AUD-002 — Actor Identification

Each audit event shall identify the actor responsible for the action, including Doctor, Admin, or System where applicable.

### AUD-003 — Actor ID

Each audit event shall contain the internal identifier of the actor when an authenticated actor exists.

### AUD-004 — Action Identification

Each audit event shall identify the action that occurred using a consistent internal action type.

### AUD-005 — Target Identification

Where applicable, each audit event shall identify the affected entity type and entity identifier.

### AUD-006 — Timestamp

Each audit event shall contain the event timestamp.

### AUD-007 — UTC Storage

Audit event timestamps shall be stored using UTC.

### AUD-008 — Result

Audit events shall record whether the action succeeded or failed where applicable.

### AUD-009 — Correlation

Audit events shall support correlation with the originating request or operation through a request/correlation identifier where applicable.

### AUD-010 — Authentication Auditing

The system shall audit significant authentication and account-security events, including successful and failed authentication attempts and security-related account changes.

### AUD-011 — Authorization Auditing

The system shall audit significant unauthorized access or modification attempts.

### AUD-012 — Patient Management Auditing

The system shall audit significant patient creation, modification, archival, restoration, assignment, transfer, and communication-account linking changes.

### AUD-013 — Patient Access Auditing

The system shall support auditing of significant access to sensitive patient records in accordance with the system's privacy and authorization policy.

### AUD-014 — Medication Auditing

The system shall audit significant medication creation, modification, discontinuation, and clinically meaningful medication changes.

### AUD-015 — Schedule Auditing

The system shall audit significant medication schedule creation, modification, activation, pausing, discontinuation, and timing/frequency changes.

### AUD-016 — Reminder Configuration Auditing

The system shall audit significant reminder configuration changes.

### AUD-017 — Notification Configuration Auditing

The system shall audit significant notification-channel, escalation, and notification-configuration changes.

### AUD-018 — Adherence Administrative Auditing

The system shall audit authorized administrative actions that modify or materially affect adherence records.

### AUD-019 — Telegram Account Auditing

The system shall audit significant Telegram account linking and unlinking actions.

### AUD-020 — Role and Permission Auditing

The system shall audit significant role, permission, and access-control changes.

### AUD-021 — Administrative Configuration Auditing

The system shall audit significant system and administrative configuration changes.

### AUD-022 — System Actor

The system shall support System as an audit actor for significant system-generated actions.

### AUD-023 — Change Summary

For significant record modifications, the system shall record an appropriate structured summary of the changed fields.

### AUD-024 — Sensitive Data Minimization

Audit events shall not contain passwords, authentication secrets, provider credentials, tokens, OTPs, or other secrets.

### AUD-025 — Clinical Data Minimization

Audit metadata shall contain only the minimum sensitive or clinical information necessary for accountability and traceability.

### AUD-026 — Append-Only

Audit records shall be append-only from normal application users.

### AUD-027 — No User Deletion

Doctors and Admins shall not be permitted to delete audit records through normal application functionality.

### AUD-028 — No User Modification

Doctors and Admins shall not be permitted to modify historical audit records through normal application functionality.

### AUD-029 — Role-Based Audit Access

Access to audit logs shall be controlled by role and authorization.

### AUD-030 — Doctor Audit Scope

Doctors shall only be able to access audit information within their authorized scope.

### AUD-031 — Admin Audit Scope

Authorized Admin users shall be able to access audit information within their administrative scope.

### AUD-032 — Doctor Isolation

A Doctor shall not be able to use audit logs to access unrelated data or activity belonging outside their authorized scope.

### AUD-033 — Notification History Separation

Notification delivery and communication events shall remain distinguishable from audit events.

### AUD-034 — Adherence Separation

Adherence records shall remain separate from audit records while significant administrative actions affecting adherence may also generate audit events.

### AUD-035 — Application Log Separation

Technical application logs shall remain separate from the audit log.

### AUD-036 — System Events

Only significant system-generated business or security events shall require audit events; routine technical background-job execution shall not automatically create audit records.

### AUD-037 — Reliability

Critical audit events shall be persisted reliably and shall not be silently lost.

### AUD-038 — Traceability

Audit events shall provide sufficient information to trace a significant action to its actor, target, time, and result.

### AUD-039 — Historical Integrity

Historical audit records shall remain unchanged after creation through normal application functionality.

### AUD-040 — Privacy

Audit-log access and storage shall comply with the system's privacy and data-protection requirements.

### AUD-041 — Retention Policy

Audit records shall be retained per documented/configurable retention policy.

**Cross-Reference:** Retention policy requirements are defined in Step 19 SEC-022.

### AUD-042 — No Invented Retention Periods  

The system shall not assume specific legal retention durations unless explicitly established by organizational or legal requirements.

### AUD-043 — Controlled Retention Actions

Data retention, archival, and deletion actions affecting audit records shall be:
* Explicitly authorized
* Controlled through documented procedures
* Traceable and auditable
* Not performed silently through ordinary business operations

### AUD-044 — Retention History Integrity

Retention actions shall preserve audit trail integrity required for accountability and regulatory purposes.

### AUD-045 — Monitoring

Critical audit-log failures shall be detectable by system monitoring and operational alerting.

### AUD-046 — Idempotency

The audit mechanism shall prevent unintended duplicate audit events for the same logical operation where duplicate processing can occur.

### AUD-047 — Provider Independence

Audit logging shall not depend on a specific SMS, Voice, Telegram, hosting, or cloud provider.

### AUD-048 — Auditability of Critical Changes

Critical changes affecting patient data, medication, schedules, access control, or notification behavior shall be auditable.

### AUD-049 — No Secret Logging

The system shall prevent security secrets and credentials from being written into audit metadata or audit payloads.

---

# 15.5 Cross-Document Authority

To prevent conflicting requirements:

| Audit Area | Primary Requirement Section |
|------------|----------------------------|
| Authentication event structure | Step 4B |
| Authentication event auditability | Step 15 |
| Adherence business logic | Step 14B |
| Adherence administrative audit | Step 15 |
| Security controls | Step 19 |
| Audit requirements | Step 15 |
| Retention policy framework | Step 19 SEC-022 |
| Audit-specific retention | Step 15 |

**Critical Rule:** Step 15 defines audit requirements to support other documents' business logic. Where audit affects the behavior defined in other documents, those documents remain authoritative for the business behavior while Step 15 defines auditability requirements.

---

# 15.6 Audit Acceptance Criteria

Step 15 shall be considered satisfied for Phase 1 when:

1. Significant authentication and security events are auditable per Step 4B requirements.
2. Patient management actions are auditable with appropriate privacy controls.
3. Medication and schedule changes are auditable for clinical traceability.
4. Reminder and notification configuration changes are auditable.
5. Adherence-affecting administrative actions are auditable.
6. Audit events contain actor, action, entity, timestamp, and result information.
7. Audit timestamps use UTC storage.
8. Doctor access is limited to authorized patient audit records.
9. Admin access follows administrative scope boundaries.
10. Ordinary users cannot edit or delete audit records.
11. Sensitive secrets are excluded from audit metadata.
12. Unnecessary sensitive clinical content is minimized in audit records.
13. Critical audit persistence failures are detectable by monitoring.
14. Audit logging operates independently of communication providers.
15. System-generated significant events are auditable where appropriate.
16. Audit records are distinguishable from application logs and notification history.
17. Change summaries capture relevant modifications without storing complete records.
18. Authorization violations and suspicious access attempts are auditable.
19. Retention follows documented/configurable policy per Step 19 SEC-022.
20. Retention actions are controlled, traceable, and do not silently delete records.

---

# 15.7 Relationship With Other Requirements

Step 15 shall support, not override, requirements established elsewhere.

Where audit events involve authentication behavior, Step 4B remains the authoritative source for authentication requirements.

Where audit events involve adherence behavior, Step 14B remains authoritative for adherence business logic.

Where audit involves security controls, Step 19 remains authoritative for security implementation.

Where audit involves retention policies, Step 19 SEC-022 provides the policy framework while Step 15 defines audit-specific retention requirements.

Step 15 defines the audit logging requirements necessary to ensure accountability and traceability across all system functions.
