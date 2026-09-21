# Step 20 — Requirement Acceptance Criteria

## 20.1 Purpose

The Phase 1 acceptance criteria define the conditions that must be satisfied before the system can be considered to have fulfilled the approved Phase 1 requirements.

Acceptance criteria must be:

* Observable
* Testable
* Traceable to approved requirements
* Independent of implementation details where possible
* Applicable to the complete Phase 1 system
* Used during integration testing, staging verification, and final release review

A feature is not considered complete merely because the code exists. It must satisfy the approved requirement, behave correctly under expected and failure conditions, preserve required history, and pass the applicable security and reliability checks.

---

## 20.2 General Acceptance Rules

### ACC-001 — Requirements Traceability

Every approved Phase 1 requirement must be traceable to:

**Requirement → Design → Implementation → Test → Verification**

No approved requirement may be silently omitted.

### ACC-002 — Scope Compliance

The implementation must remain within the approved Phase 1 scope.

Features not approved for Phase 1 must not be introduced as production functionality merely because they are technically possible.

### ACC-003 — Requirement Changes

If an approved requirement changes during implementation:

1. The requirement change must be explicitly identified.
2. The affected design must be reviewed.
3. The implementation impact must be identified.
4. The updated requirement must be approved before it becomes the new implementation target.
5. Related tests and traceability must be updated.

### ACC-004 — Functional Correctness

Each approved workflow must perform the required business operation correctly under normal conditions.

### ACC-005 — Failure Handling

Important failure conditions defined by the requirements must produce the specified safe behavior rather than silently succeeding, losing history, or producing an incorrect clinical/adherence result.

### ACC-006 — Historical Integrity

Required historical records must remain available and must not be incorrectly rewritten because of later changes to patients, medications, schedules, reminder configuration, notification channels, or assignments.

### ACC-007 — Authorization

A user must only be able to perform actions and access data permitted by their role and scope.

### ACC-008 — Security

The implementation must satisfy the approved authentication, authorization, privacy, secret-management, API, webhook, logging, and other security requirements.

### ACC-009 — Reliability

Critical background operations must remain reliable across normal application restarts, worker restarts, provider failures, retries, and other defined failure conditions.

### ACC-010 — Idempotency

Operations explicitly requiring idempotency must not create duplicate business outcomes when the same request, callback, webhook, or processing attempt is received more than once.

---

# 20.3 Authentication and Account Acceptance

### ACC-011 — Admin Authentication

An Admin can authenticate using the approved authentication flow and mandatory MFA.

### ACC-012 — Doctor Authentication

A Doctor can authenticate using the approved email/password authentication flow.

### ACC-013 — Account Separation

Admin and Doctor permissions remain separated throughout the system.

### ACC-014 — Doctor Account Creation

An authorized Admin can create Doctor accounts.

A Doctor cannot create an Admin account.

### ACC-015 — Initial Admin

The first Admin account can only be established through the approved secure initial setup process.

### ACC-016 — Inactive Accounts

Inactive/deactivated accounts cannot authenticate or access protected application functionality.

### ACC-017 — Password Recovery

Approved password-recovery behavior works using secure, time-limited, single-use reset mechanisms without exposing whether an account exists.

### ACC-018 — Recovery Restrictions

Forgotten/lost email recovery follows the approved controlled administrative/organizational recovery process.

There must be no hidden master password or backdoor.

### ACC-019 — Authentication Protection

Failed authentication attempts and other defined authentication-abuse conditions are handled according to the approved security requirements.

---

# 20.4 Authorization and Patient Isolation Acceptance

### ACC-020 — Server-Side Authorization

Authorization is enforced by the backend and cannot be bypassed by modifying frontend requests.

### ACC-021 — Doctor Patient Isolation

A Doctor can access only patients within the Doctor's authorized scope.

### ACC-022 — Cross-Doctor Protection

A Doctor cannot access, modify, or configure another Doctor's patients unless explicitly authorized by the approved access model.

### ACC-023 — Admin Scope

Admin functionality follows the approved administrative scope and does not unnecessarily expose clinical information.

### ACC-024 — Object-Level Authorization

Protected patient, medication, schedule, reminder, adherence, and related resources are checked for authorization at the object/resource level.

---

# 20.5 Patient Acceptance

### ACC-025 — Patient Creation

An authorized user can create a patient record according to the approved patient requirements.

### ACC-026 — Patient Updates

Authorized users can update permitted patient information while preserving required history.

### ACC-027 — Patient Assignment

Patient assignment/transfer follows the approved authorization and audit requirements.

### ACC-028 — Patient Communication Information

Patient phone number, preferred language, Telegram linkage, and communication configuration are handled according to the approved requirements.

### ACC-029 — Patient Historical Integrity

Archiving, restoration, transfer, or communication changes do not incorrectly destroy historical reminder, notification, or adherence information.

---

# 20.6 Medication and Schedule Acceptance

### ACC-030 — Medication Management

Authorized users can create, update, and discontinue medications according to the approved requirements.

### ACC-031 — Schedule Management

Authorized users can create and manage medication schedules according to the approved requirements.

### ACC-032 — Schedule State

Schedules correctly support their required active, paused, discontinued, or equivalent lifecycle states.

### ACC-033 — Schedule Timezone

Schedule timing is interpreted using the schedule's configured timezone while persistent timestamps use the approved UTC model.

### ACC-034 — Schedule Changes

Changes to schedules affect future reminder occurrences according to the approved rules without incorrectly rewriting historical occurrences.

---

# 20.7 Reminder Acceptance

### ACC-035 — Reminder Generation

A valid medication schedule produces the required reminder occurrence at the configured time.

### ACC-036 — Reminder Persistence

Reminder processing does not depend on an in-memory scheduler that would lose work after application restart.

### ACC-037 — Duplicate Prevention

A reminder occurrence is not created multiple times because of retries, worker restarts, or repeated processing.

### ACC-038 — Reminder Traceability

Every reminder occurrence can be traced to its patient, medication, and schedule.

### ACC-039 — Reminder History

Reminder history remains available even when the underlying schedule or medication later changes.

### ACC-040 — Reminder State Separation

Reminder state remains distinct from notification delivery state and adherence state.

---

# 20.8 Notification Acceptance

### ACC-041 — Escalation Sequence

Notifications follow the configured escalation sequence rather than being sent to all channels simultaneously unless explicitly configured otherwise.

### ACC-042 — Channel Transition

When the configured escalation condition is reached, the system transitions to the next eligible channel according to the approved policy.

### ACC-043 — Response Stops Escalation

A valid accepted adherence response stops future escalation for that reminder occurrence.

### ACC-044 — Delivery Is Not Adherence

A successfully sent or delivered notification must never automatically be interpreted as Taken or Not Taken.

### ACC-045 — Notification State

Notification status is maintained separately from reminder and adherence status.

### ACC-046 — Retry Handling

Retryable notification failures are retried according to the approved retry policy.

### ACC-047 — Non-Retryable Failures

Non-retryable failures do not cause uncontrolled retry loops.

### ACC-048 — Duplicate Notification Prevention

The same notification attempt is not unintentionally sent multiple times because of retries, duplicate jobs, worker restarts, or repeated provider callbacks.

### ACC-049 — Provider Failure

A provider failure is handled by the notification orchestration and fallback rules without corrupting the reminder or adherence state.

### ACC-050 — Provider Independence

Replacing the SMS or Voice provider does not require rewriting the core reminder/adherence business logic.

---

# 20.9 Telegram Acceptance

### ACC-051 — Secure Linking

A patient can link Telegram through the approved secure temporary linking process.

### ACC-052 — Telegram Identity

The system uses the appropriate Telegram chat/user identifier as the communication identity rather than relying on a patient name or username as the primary identity.

### ACC-053 — Telegram Reminder

A valid reminder can be delivered through Telegram using the centralized localized message/template system.

### ACC-054 — Telegram Response

The patient can provide the approved Taken or Not Taken response using the Telegram interface.

### ACC-055 — Telegram Callback Validation

Telegram callbacks are validated against the appropriate patient, reminder, channel, and eligibility state.

### ACC-056 — Telegram Idempotency

Duplicate Telegram callbacks do not create duplicate adherence outcomes.

### ACC-057 — Telegram Failure

Telegram delivery/processing failure is mapped to the internal notification model and handled by the notification orchestration.

---

# 20.10 SMS Acceptance

### ACC-058 — SMS Provider Abstraction

SMS functionality operates through the approved provider abstraction.

### ACC-059 — SMS Reminder

A valid reminder can be sent through SMS using the centralized localization/template system.

### ACC-060 — SMS Response

The approved SMS responses are interpreted as:

* `1` → Taken
* `2` → Not Taken

### ACC-061 — Invalid SMS Response

Unrecognized SMS text does not automatically modify adherence.

### ACC-062 — SMS Response Association

An incoming SMS response is accepted only when it can be unambiguously associated with the appropriate active reminder context.

### ACC-063 — SMS Delivery Separation

Outgoing SMS delivery status is kept separate from incoming patient adherence responses.

### ACC-064 — SMS Provider Failure

SMS provider failures are handled through the approved retry and escalation mechanisms.

---

# 20.11 Voice Acceptance

### ACC-065 — Automated Voice Call

The system can initiate an automated outbound reminder call through the approved Voice Provider abstraction.

### ACC-066 — Voice Reminder

The call can deliver the required reminder audio/content in the supported patient language.

### ACC-067 — DTMF Response

The approved keypad responses are interpreted as:

* `1` → Taken
* `2` → Not Taken

### ACC-068 — Answered Call Is Not Adherence

An answered call without a valid DTMF response does not produce a Taken or Not Taken result.

### ACC-069 — No Answer

No-answer conditions are represented distinctly from answered calls where required.

### ACC-070 — Invalid DTMF

Invalid keypad input does not automatically modify adherence.

### ACC-071 — Voice Idempotency

Duplicate voice callbacks or DTMF events do not create duplicate adherence outcomes.

### ACC-072 — Voice Provider Independence

The production Voice Provider can be replaced without changing core reminder/adherence business logic.

---

# 20.12 Adherence Acceptance

### ACC-073 — One Reminder, One Adherence Decision

Each reminder occurrence produces no more than one accepted adherence decision.

### ACC-074 — Active Response Channel

Only the currently active response channel can produce an accepted adherence response.

### ACC-075 — Closed Channel Protection

A closed channel cannot later modify the accepted adherence result.

### ACC-076 — Valid Responses

The approved Taken/Not Taken responses from Telegram, SMS, and Voice are processed correctly.

### ACC-077 — No Response

If the overall response window expires without a valid accepted response, the system records `NO_RESPONSE` according to the approved rules.

### ACC-078 — Unrecognized Response

Unrecognized responses do not silently become adherence results.

### ACC-079 — Late Response

Late responses are preserved as events and marked according to the approved late-response policy without incorrectly changing a closed adherence decision.

### ACC-080 — Conflicting Responses

Conflicting responses are resolved deterministically using reminder/channel state and transaction/idempotency controls.

### ACC-081 — Adherence History

Adherence history remains preserved despite later patient assignment, medication, or schedule changes.

---

# 20.13 Audit Acceptance

### ACC-082 — Significant Actions Audited

Required security, administrative, clinical-data, configuration, and business actions generate audit records.

### ACC-083 — Audit Actor

Audit records identify the responsible user or system actor.

### ACC-084 — Audit Context

Required audit context includes the appropriate timestamp, actor, action, entity, result, correlation/request identifier, and limited metadata.

### ACC-085 — Sensitive Data Protection

Passwords, reset tokens, OTPs, API credentials, provider credentials, and other prohibited secrets are not stored in audit records.

### ACC-086 — Audit Immutability

Ordinary users cannot edit or delete audit records.

### ACC-087 — Audit Authorization

Users can only view audit information permitted by their approved role and scope.

### ACC-088 — Critical Audit Reliability

Critical audit events are reliably persisted and failures are visible to the appropriate monitoring/operational mechanisms.

---

# 20.14 Dashboard Acceptance

### ACC-089 — Role-Specific Dashboard

The dashboard presents information appropriate to the logged-in role.

### ACC-090 — Doctor Dashboard

Doctor dashboard information is limited to authorized patients and includes the approved operational reminder/adherence information.

### ACC-091 — Admin Dashboard

Admin dashboard provides the approved broader operational/system visibility while minimizing unnecessary clinical exposure.

### ACC-092 — Reminder States

Dashboard reminder information distinguishes the approved states, including Pending, Taken, Not Taken, No Response, and notification-related failure conditions where applicable.

### ACC-093 — Attention Conditions

Patients/reminders requiring attention are surfaced using the approved transparent rules.

### ACC-094 — Error States

Dashboard failures do not misleadingly appear as successful empty/zero results.

---

# 20.15 Settings Acceptance

### ACC-095 — Role-Appropriate Settings

Users can only modify settings permitted by their role.

### ACC-096 — Communication Language

Patient communication language is independent from the user's own interface language.

### ACC-097 — System Boundaries

Admin controls system-wide available channels, defaults, and policy boundaries according to the approved model.

### ACC-098 — Patient-Level Configuration

Doctors can configure reminder/escalation behavior for assigned patients within Admin-defined limits.

### ACC-099 — Fundamental Rules

Configurable settings cannot override the approved fundamental adherence, authorization, audit, and security rules.

### ACC-100 — Settings History

Significant settings changes are audited.

### ACC-101 — Existing Reminder Integrity

Settings changes affect future reminder occurrences according to the approved policy without rewriting historical reminder decisions.

### ACC-102 — Provider Secrets

Provider credentials and other secrets are not exposed through ordinary settings interfaces or stored as ordinary application data.

---

# 20.16 Localization Acceptance

### ACC-103 — Supported Languages

Phase 1 communication supports:

* English
* Amharic
* Afaan Oromoo

### ACC-104 — Centralized Templates

Notification content is managed through the approved centralized localization/template mechanism.

### ACC-105 — Consistent Language

The selected patient communication language is consistently respected across supported notification channels where the provider/channel supports the required content.

---

# 20.17 Non-Functional Acceptance

### ACC-106 — Reliability

Critical reminder and notification processing survives expected application/worker restarts without losing required work.

### ACC-107 — Performance

Normal user-facing API and dashboard operations meet the approved Phase 1 performance expectations.

### ACC-108 — Availability and Recovery

The system can recover from expected service interruptions without corrupting reminder, notification, adherence, or audit history.

### ACC-109 — Data Integrity

Database transactions, constraints, and application logic preserve required relationships and business invariants.

### ACC-110 — Scalability Baseline

The architecture supports the approved Phase 1 scale without requiring fundamental redesign of the reminder/notification processing model.

### ACC-111 — Observability

Important application, background-job, notification, and failure conditions provide sufficient operational visibility.

### ACC-112 — Backup and Recovery

Approved backup and recovery procedures are defined and verified to the extent required for Phase 1.

### ACC-113 — Maintainability

Core business logic is sufficiently separated from external notification providers to permit provider replacement.

### ACC-114 — Testability

Critical business workflows and failure conditions can be automated or otherwise reproducibly tested.

---

# 20.18 Docker and Deployment Acceptance

### ACC-115 — Reproducible Development Environment

The approved Docker configuration can reproduce the required development services without undocumented machine-specific dependencies.

### ACC-116 — Backend Container

The backend can be built and run from the approved container configuration.

### ACC-117 — Container Configuration

Container configuration does not contain production credentials or secrets.

### ACC-118 — Production Readiness

The backend container/build can be verified for deployment to the approved Render environment.

### ACC-119 — Frontend Deployment

The frontend can be built and deployed through the approved Vercel workflow.

### ACC-120 — Environment Separation

Development, staging/testing, and production configuration/secrets are appropriately separated.

### ACC-121 — Deployment Recovery

Application restart/redeployment does not cause loss of persisted reminder, notification, adherence, or audit history.

---

# 20.19 Security Acceptance

### ACC-122 — Authentication Security

Authentication follows the approved Step 4 requirements.

### ACC-123 — Authorization Security

Backend authorization prevents unauthorized access and privilege escalation.

### ACC-124 — Secret Protection

Application secrets, provider credentials, tokens, and sensitive configuration are not committed to source control or exposed to the frontend.

### ACC-125 — API Security

Protected APIs validate input, authorization, resource ownership, and relevant state before performing operations.

### ACC-126 — Webhook Security

External provider webhooks are authenticated/verified where supported, validated, authorized, and processed idempotently.

### ACC-127 — Sensitive Logging Protection

Logs do not expose passwords, tokens, provider credentials, or unnecessary sensitive patient/communication content.

### ACC-128 — Transport Security

Production communication uses the approved encrypted transport mechanisms.

### ACC-129 — Security Failure Visibility

Important security failures and repeated abuse conditions are visible through the approved monitoring/audit mechanisms.

---

# 20.20 Final Phase 1 Acceptance Gate

Phase 1 can proceed to final release review only when:

1. All approved requirements have corresponding acceptance criteria.
2. All required acceptance criteria have been tested.
3. Failed criteria have been resolved or explicitly dispositioned through the approved change process.
4. Critical security requirements pass.
5. Critical reminder/notification reliability requirements pass.
6. Adherence invariants pass.
7. Authorization and Doctor patient isolation pass.
8. Required audit behavior passes.
9. Required historical integrity passes.
10. Docker/build/deployment verification passes.
11. Frontend and backend integration passes.
12. Production-like staging/simulation passes.
13. No unapproved scope has been introduced.
14. Requirement traceability is complete.
15. The final requirements and release review is approved by the human project owner.

## 20.21 Acceptance Principle

The final decision is not:

> "Does the application appear to work?"

The final decision is:

> **"Does the implemented system demonstrably satisfy the approved Phase 1 requirements, including normal behavior, failure behavior, security, reliability, authorization, historical integrity, and deployment readiness?"**

Only after this acceptance gate is satisfied can Phase 1 be considered ready for release.
