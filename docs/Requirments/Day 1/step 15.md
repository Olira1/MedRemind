Great. **Step 14 — Adherence Requirements is now locked** with the channel-window model we just agreed on.

We now move to **Step 15 — Audit Log Requirements**. We will work on this step only and will not combine it with Step 16.

## Step 15 — Audit Log Requirements

### 15.1 What is the purpose of the audit log?

The audit log answers:

> **Who did what, to which record, when, and what happened?**

For a clinical system, this is important for **security, accountability, troubleshooting, privacy, and historical traceability**.

It is different from:

* **Application logs** → technical debugging
* **Notification history** → Telegram/SMS/Voice delivery events
* **Adherence records** → patient's medication response
* **Audit logs** → important user/system actions and changes

For example:

> Doctor changes a patient's medication dose.

That should create an audit event:

```text
Actor: Doctor
Action: UPDATE_MEDICATION
Patient: Patient #123
Medication: Medication #456
Time: 2026-09-14 08:30 UTC
Result: SUCCESS
```

---

# 15.2 What actions should be audited?

I recommend auditing the following categories.

### A. Authentication and account security

Audit:

* Successful login
* Failed login
* Logout where useful
* Password change
* Password reset/request
* Account activation
* Account deactivation
* Account lock/unlock
* Role change
* Permission/security-setting changes

We should **never** store passwords, password hashes, reset tokens, OTPs, or authentication secrets in the audit log.

---

### B. Patient management

Audit important patient actions:

* Patient created
* Patient information updated
* Patient archived/deactivated
* Patient restored/reactivated
* Patient transferred between doctors
* Patient assignment changed
* Patient communication/contact information changed
* Telegram linked
* Telegram unlinked

For sensitive patient-data access, I recommend also auditing **important patient record views**, especially if the system is intended for real clinical use.

We don't necessarily need to record every UI click such as opening a menu. The audit system should focus on meaningful access and actions.

---

### C. Medication management

Audit:

* Medication created
* Medication updated
* Medication discontinued
* Medication reactivated, if supported
* Medication instructions changed
* Dosage/frequency changes
* Medication associated with a patient
* Medication removed/discontinued

This is particularly important because medication changes can affect future reminders.

---

### D. Schedule management

Audit:

* Schedule created
* Schedule updated
* Schedule activated
* Schedule paused
* Schedule discontinued
* Schedule timing changed
* Schedule frequency changed
* Schedule-related configuration changes

Example:

```text
Doctor
→ changes medication reminder
→ 08:00 daily → 08:00 + 20:00 daily
```

That change should be traceable.

---

### E. Reminder and notification actions

Important reminder-related changes should be audited:

* Reminder/schedule configuration changed
* Reminder cancelled
* Reminder manually triggered, if we support manual triggering
* Notification configuration changed
* Escalation policy changed
* Notification channel enabled/disabled
* Provider configuration changed

However, **we should not put every SMS/Telegram/Voice delivery event into the audit log**.

Those belong primarily in notification history.

The audit log can record significant administrative changes to notification configuration.

---

### F. Adherence-related administrative actions

Because adherence is clinically important, we should audit significant administrative actions involving it.

For example:

* Authorized user manually corrects an adherence record, if manual correction is allowed
* An adherence record is marked/reclassified
* An administrative action changes adherence-related configuration

The underlying patient response event itself remains part of the **adherence data**, not merely an audit record.

---

### G. Authorization/security violations

Audit events such as:

* Unauthorized patient access attempt
* Unauthorized medication access attempt
* Unauthorized modification attempt
* Access to another doctor's patient
* Invalid/expired authorization
* Suspicious repeated failed access attempts

This will be useful for security monitoring.

---

# 15.3 What information should every audit event contain?

I recommend each audit event contain at least:

| Field                         | Purpose                             |
| ----------------------------- | ----------------------------------- |
| `id`                          | Unique audit event ID               |
| `timestamp`                   | When the event occurred             |
| `actor_type`                  | Doctor, Admin, System               |
| `actor_id`                    | Who performed it                    |
| `action`                      | What happened                       |
| `entity_type`                 | Patient, Medication, Schedule, etc. |
| `entity_id`                   | Which record was affected           |
| `result`                      | Success/failure                     |
| `request_id` / correlation ID | Connect related operations          |
| `metadata`                    | Limited additional context          |

For example:

```text
Audit Event

Actor: Doctor
Actor ID: doctor_123
Action: UPDATE_MEDICATION
Entity: Medication
Entity ID: med_456
Patient ID: patient_789
Timestamp: 2026-09-14T08:30:12Z
Result: SUCCESS
Request ID: req_abc123
```

---

# 15.4 Should we store "before" and "after" values?

This needs careful handling.

For important configuration changes, knowing **what changed** is very useful.

For example:

```text
Dose:
Before: 1 tablet
After: 2 tablets
```

However, we should **not blindly copy the entire patient record into the audit log**.

That creates unnecessary duplication of sensitive clinical information.

My recommendation:

> Store a **minimal structured change summary** for important modifications.

For example:

```text
changed_fields:
  - dosage
  - frequency
```

And, where clinically necessary and appropriate, the relevant old/new values.

We should establish privacy rules around exactly which fields may be stored in audit metadata.

---

# 15.5 Who can view audit logs?

This is particularly important because we have separate **Admin** and **Doctor** roles.

My recommendation:

### Admin

Admin can view audit activity within the scope they are authorized to administer.

For a system-wide administrator:

> Organization/system-wide audit visibility.

### Doctor

A doctor should **not** be able to see another doctor's audit activity or unrelated patients.

A doctor can view audit history related to:

* Their own actions
* Patients they are authorized to access
* Records within their authorized scope

This preserves doctor-to-doctor isolation.

---

# 15.6 Can users modify or delete audit logs?

**No.**

The application should treat audit records as **append-only**.

A Doctor or Admin should not have a normal UI/API operation such as:

> Edit Audit Log
> Delete Audit Log

This prevents someone from changing the historical record of what happened.

If we eventually need archival or retention deletion for legal/privacy reasons, that should be a controlled system-level process with its own traceability—not an ordinary user action.

---

# 15.7 Audit log vs application log

We should explicitly keep these separate.

### Application log

Example:

```text
Database connection timeout
Redis connection failed
BullMQ worker restarted
HTTP 500 error
```

Primarily for developers/operators.

### Audit log

Example:

```text
Doctor changed patient's medication dosage.
Admin disabled doctor account.
Doctor transferred patient to another doctor.
```

Primarily for accountability/security/clinical traceability.

### Notification history

Example:

```text
Telegram sent
SMS delivered
Voice call failed
SMS response received
```

### Adherence

Example:

```text
TAKEN
NOT_TAKEN
NO_RESPONSE
```

These four systems should not be mixed together.

---

# 15.8 System-generated audit events

Not every audit event comes from a human.

The system itself may perform important actions.

For example:

```text
Actor: SYSTEM
Action: AUTO_ARCHIVE_EXPIRED_LINK
```

or:

```text
Actor: SYSTEM
Action: ESCALATION_TRIGGERED
```

However, we shouldn't turn every background-job execution into an audit record.

Only **meaningful security/business actions** should be audited.

---

# 15.9 Time standard

I recommend that all audit timestamps be stored in:

> **UTC**

The frontend can display the appropriate local time to the user.

This avoids problems with:

* Different users/time zones
* Server locations
* Daylight-saving changes
* Distributed services

---

# 15.10 Privacy and sensitive data

Audit logs can themselves become sensitive.

Therefore:

**Never store:**

* Passwords
* Password hashes
* Authentication tokens
* Telegram bot tokens
* SMS/Voice provider credentials
* OTPs
* API keys
* Full secret webhook payloads
* Unnecessary full medical records
* Unnecessary full SMS/Telegram message contents

We should store enough information to establish accountability without creating a second uncontrolled copy of sensitive information.

---

# 15.11 Reliability

An important production requirement:

> Critical audit events must not silently disappear.

For example, if a doctor changes a medication and the medication update succeeds but the audit event is lost, we have a serious traceability problem.

During architecture/design we'll therefore need a reliable mechanism for critical audit writes—potentially transactionally or through an appropriate event/outbox pattern.

We don't need to decide the implementation mechanism in the requirements phase; the requirement is that critical audit events are reliably persisted.

---

# 15.12 Proposed formal Step 15 requirements

## 15. Audit Log Requirements

### Purpose

The system shall maintain an audit log that provides traceability of significant security, administrative, clinical-data, configuration, and business actions.

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

### AUD-041 — Retention

Audit-log retention shall follow the retention policy defined by the system's security, privacy, and operational requirements.

### AUD-042 — Monitoring

Critical audit-log failures shall be detectable by system monitoring and operational alerting.

### AUD-043 — Idempotency

The audit mechanism shall prevent unintended duplicate audit events for the same logical operation where duplicate processing can occur.

### AUD-044 — Provider Independence

Audit logging shall not depend on a specific SMS, Voice, Telegram, hosting, or cloud provider.

### AUD-045 — Auditability of Critical Changes

Critical changes affecting patient data, medication, schedules, access control, or notification behavior shall be auditable.

### AUD-046 — No Secret Logging

The system shall prevent security secrets and credentials from being written into audit metadata or audit payloads.

### The main decision I need from you

Before I lock Step 15, I want you to review **one particularly important policy**:

> **Should Doctors be able to view the audit history of patient records they are authorized to access, while Admins can view audit history within their administrative scope?**

That is my recommended model.

If you accept Step 15 as written, including that access model, I will **lock Step 15** and move to **Step 16 — Dashboard Requirements**.
