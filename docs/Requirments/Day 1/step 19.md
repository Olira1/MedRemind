# STEP 19 — SECURITY REQUIREMENTS

## 19.1 Purpose

The MedReminder system shall implement security controls appropriate for a clinical patient medication-reminder and adherence system.

Security requirements shall protect:

* Staff accounts
* Patient information
* Medication and schedule information
* Reminder and adherence information
* Communication-channel identifiers
* Notification infrastructure
* System configuration
* Audit records
* Provider credentials and secrets
* APIs and webhooks
* Application infrastructure and stored data

**Cross-Document Authority:** Step 4B defines required authentication behavior. Step 15 defines audit requirements. Steps 10B-13 define communication channel behavior. Step 14B defines adherence decision authority. This document defines security controls to protect those requirements.

This section defines the broader security controls required to implement, operate, and protect the system securely.

---

# 19.2 Security Principles

### SEC-001 — Security by Design

Security shall be considered throughout application design, implementation, deployment, and operation.

### SEC-002 — Least Privilege

Users, services, background workers, providers, and system components shall receive only the permissions necessary for their responsibilities.

### SEC-003 — Defense in Depth

The system shall use multiple complementary security controls rather than relying on a single security mechanism.

### SEC-004 — Secure Defaults

Security-sensitive configuration shall use secure defaults.

Features shall not be enabled in an insecure state merely because an administrator has not configured them.

### SEC-005 — Fail Securely

When a security-sensitive operation fails, the system shall fail in a manner that does not grant unauthorized access or permissions.

---

# 19.3 Authentication Security

Authentication behavior is defined in Step 4B.

### SEC-006 — Step 4B Authentication Compliance

The implementation shall comply with all applicable authentication requirements defined in Step 4B.

This includes:

* Staff authentication
* Admin MFA (mandatory)
* Doctor MFA (architecturally supported)
* Password protection
* Password recovery (secure email-based)
* Session behavior
* Failed-login protection
* Account status
* Authentication recovery
* Role separation
* First Admin bootstrap security properties

### SEC-007 — Secure Authentication Implementation

Authentication mechanisms shall be implemented using established secure practices and shall not expose credentials, authentication secrets, reset tokens, or MFA secrets unnecessarily.

### SEC-008 — Authentication Secret Protection

Passwords, password-reset tokens, MFA secrets, session credentials, and other authentication secrets shall be protected from unauthorized access.

### SEC-009 — No Authentication Bypass

The application shall not contain undocumented authentication bypasses, universal credentials, hidden master passwords, or equivalent mechanisms.

**Cross-Reference:** This reinforces Step 4B requirement AUTH-033 (no backdoor/master password in First Admin bootstrap).

---

# 19.4 Authorization and Access Control

### SEC-010 — Server-Side Authorization

Authorization shall be enforced by the backend.

The system shall not rely solely on frontend visibility or UI restrictions to protect resources.

**Critical Invariant:** All authorization decisions must be server-side enforced, consistent with notification orchestrator authority (NOTIF-REQ-017) and adherence decision authority (ADH-PRINCIPLE-004).

### SEC-011 — Role Separation

Admin and Doctor permissions shall remain distinct.

A Doctor shall not obtain Admin privileges by manipulating requests, URLs, identifiers, frontend state, or API parameters.

### SEC-012 — Doctor Patient Isolation

A Doctor shall only access patients within the Doctor's authorized scope.

A Doctor shall not be able to access another Doctor's patients by changing a patient ID or other request parameter.

**Cross-Reference:** This enforces the authorization boundaries referenced in Step 4B and supports the patient assignment model.

### SEC-013 — Administrative Scope

Admin access shall follow the permissions and administrative scope defined for the Admin role.

Administrative privileges shall not automatically grant unrestricted access to every operation unless that operation is explicitly authorized.

**Critical Rule:** Admin operational authority does not automatically grant clinical modification authority over Doctor-managed treatment records except through explicitly authorized, narrowly scoped, audited workflows.

### SEC-014 — Object-Level Authorization

The backend shall validate authorization for individual protected resources, including where applicable:

* Patients
* Medications
* Medication schedules
* Reminder occurrences
* Adherence records
* Notification records
* Communication settings
* Audit records

### SEC-015 — Privilege Escalation Protection

Users shall not be able to elevate their own permissions or modify protected role/permission information without appropriate authorization.

### SEC-016 — Deactivated Staff Accounts

**Requirement:** Inactive/deactivated staff accounts cannot access protected functionality.

**Server-Side Enforcement Required:** Do not rely solely on frontend hiding. Server must reject requests from deactivated accounts.

**Session Handling:** Existing sessions/tokens from deactivated accounts shall be rejected or revoked per authentication/session design defined in Step 4B.

**Audit Event:** Account deactivation and subsequent access attempts shall be audited per Step 15 requirements.

---

# 19.5 Patient Data Protection

### SEC-017 — Patient Data Confidentiality

Patient information shall only be accessible to authorized users and system components.

### SEC-018 — Data Minimization

The system shall collect, store, process, transmit, and expose only information necessary for the intended Phase 1 functionality.

### SEC-019 — Sensitive Data Exposure Prevention

Patient information shall not unnecessarily appear in:

* Application logs
* Error messages
* URLs
* Browser-visible technical data
* Notification-provider metadata
* Debug output
* Monitoring systems

**Provider Integration Rule:** Callback payloads and webhook data must not contain sensitive clinical data (TEL-043, SMS-049, VOI-052).

### SEC-020 — API Response Minimization

API responses shall return only the information required by the requesting operation and authorized user.

### SEC-021 — Patient Data in Communications

SMS, Telegram, and Voice reminders shall minimize unnecessary sensitive information while still providing enough information for the intended reminder function.

**Cross-Reference:** Consistent with notification content principles in Steps 11-13.

### SEC-022 — Data Retention Policy

Patient, reminder, adherence, notification, and audit history shall be retained per documented/configurable retention policy.

**Legal Compliance:** Specific retention duration shall be specified before production based on applicable legal/organizational requirements.

**No Silent Deletion:** Data retention actions must be controlled, documented, and auditable.

**Retention Scope:** Applies to all historical records including those required for audit trail integrity per Step 15.

---

# 19.6 API Security

### SEC-023 — Protected API Endpoints

Protected API endpoints shall require appropriate authentication and authorization.

### SEC-024 — Input Validation

Backend endpoints shall validate and sanitize incoming data according to the expected data type, format, length, and business constraints.

### SEC-025 — Malformed Requests

Malformed or invalid requests shall be rejected safely without exposing internal implementation details.

### SEC-026 — Resource Ownership Validation

Requests involving patient, medication, reminder, adherence, or notification identifiers shall verify that the authenticated actor is authorized to access the referenced resource.

### SEC-027 — Mass Assignment Protection

Clients shall not be able to modify protected fields simply by including additional fields in an API request.

Examples include:

* User role
* Account status
* Ownership
* Authorization scope
* Audit metadata
* System-generated identifiers

### SEC-028 — Error Response Security

API errors shall not expose:

* Passwords
* Secrets
* Tokens
* Database credentials
* Internal stack traces
* Sensitive infrastructure information

---

# 19.7 Rate Limiting and Abuse Prevention

### SEC-029 — Authentication Rate Limiting

Authentication endpoints shall be protected against excessive repeated attempts, consistent with Step 4B.

### SEC-030 — Password Recovery Rate Limiting

Password-recovery endpoints shall be protected against abuse and automated enumeration attempts.

**Cross-Reference:** Supports Step 4B requirements AUTH-024 and AUTH-025 for secure password recovery.

### SEC-031 — API Abuse Protection

Appropriate rate limiting shall be applied to sensitive or abuse-prone API operations.

### SEC-032 — Notification Abuse Protection

The system shall prevent uncontrolled repeated SMS, Voice, Telegram, or other notification attempts caused by:

* Duplicate requests
* Retry failures
* Concurrent workers
* Malformed jobs
* Provider callbacks
* Application errors

**Cross-Reference:** Supports notification idempotency requirements in Steps 10B-13.

---

# 19.8 Notification and Communication Security

### SEC-033 — Provider Credential Protection

SMS, Voice, Telegram, and other communication-provider credentials shall be stored securely and shall not be embedded in source code.

### SEC-034 — Provider Abstraction

Security-sensitive provider credentials and implementation details shall remain isolated from business logic.

Replacing a notification provider shall not require exposing provider credentials to unrelated application components.

**Cross-Reference:** Supports provider abstraction requirements NOTIF-REQ-016, SMS-002, SMS-003, SMS-040, VOI-015, VOI-018, VOI-026.

### SEC-035 — Communication Identity Validation

Patient communication identifiers shall be validated and associated with the correct internal patient context before they are used for reminder delivery or response processing.

**Critical Rule:** Internal Patient ID is primary identity, not external communication identifiers (Telegram chat ID, phone number).

### SEC-036 — Response Authorization

Incoming Telegram, SMS, and Voice responses shall be validated against the applicable reminder, patient context, channel, and response state before they can modify adherence.

**Cross-Reference:** Enforces adherence decision authority from Step 14B (ADH-PRINCIPLE-004: only eligible response channel can change adherence).

### SEC-037 — No Trust in Client-Supplied Adherence

The backend shall determine whether a response is valid and authorized.

A client or communication channel shall not be able to directly submit an arbitrary adherence result without backend validation.

**Closed Channel Rule:** Responses through closed channels may be preserved as events but cannot change adherence (ADH-REQ-002).

---

# 19.9 Webhook Security

### SEC-038 — Webhook Authentication

Provider webhooks shall use appropriate verification mechanisms supported by the provider.

**Critical Requirement:** State-changing webhooks must use provider-supported authenticity verification whenever provider offers verification mechanism.

**Trust Boundary:** If webhook cannot be authenticated sufficiently, must not be trusted for state-changing operations.

### SEC-039 — Webhook Validation

Incoming webhook requests shall be validated before processing.

Validation shall include, where applicable:

* Provider authenticity
* Expected event structure
* Relevant identifiers
* Event type
* Request integrity

### SEC-040 — Webhook Idempotency

Webhook processing shall be idempotent.

Repeated delivery of the same provider event shall not create duplicate business actions.

**Cross-Reference:** Supports idempotency requirements TEL-045, TEL-046, SMS-057, SMS-058, VOI-060, VOI-061.

### SEC-041 — Webhook Authorization

A webhook shall only be permitted to affect the internal record associated with the validated provider event.

### SEC-042 — Webhook Payload Minimization

Full provider webhook payloads shall not be unnecessarily stored or exposed.

Sensitive fields shall be minimized in logs and operational records.

**Callback Security:** Callback payloads must use opaque/internal references and must not contain sensitive clinical data (TEL-043, SMS-049, VOI-052).

---

# 19.10 Secrets and Configuration

### SEC-043 — Secret Management

Secrets shall be stored using secure deployment/environment secret mechanisms rather than committed to source control.

This includes:

* Database credentials
* Telegram bot credentials
* SMS provider credentials
* Voice provider credentials
* Authentication secrets
* Encryption keys where applicable
* Webhook secrets
* Other service credentials

### SEC-044 — No Secrets in Source Control

Production secrets shall never be committed to Git repositories.

### SEC-045 — No Secrets in Client Code

Server-side secrets shall never be exposed to frontend/browser code.

### SEC-046 — Secret Rotation

The system shall support replacement/rotation of provider credentials and other security-sensitive secrets without requiring changes to business logic.

### SEC-047 — Environment Separation

Development, staging, and production environments shall use appropriately separated credentials and configuration.

Development credentials shall not provide unintended production access.

---

# 19.11 Database and Data Storage Security

### SEC-048 — Database Access Restriction

Database access shall be restricted to authorized application components and operational personnel.

### SEC-049 — Database Credentials

Database credentials shall not be exposed to frontend clients or stored in application source code.

### SEC-050 — Referential and Integrity Controls

Security-sensitive records shall use appropriate database constraints and transactional behavior to prevent unauthorized or inconsistent state changes.

### SEC-051 — Historical Data Protection

Historical adherence, reminder, notification, and audit information shall not be silently modified or deleted as a consequence of ordinary business operations.

**Append-Only Rule:** Audit records are append-only for ordinary users (Step 15). Reminder/adherence history preservation supports consistency audit requirements.

### SEC-052 — Production Data Separation

Production patient data shall not be copied into development or test environments without appropriate authorization and protection.

---

# 19.12 Logging and Sensitive Information

### SEC-053 — Secure Logging

Application logs shall support troubleshooting and security monitoring without unnecessarily exposing sensitive information.

### SEC-054 — Credential Exclusion

Logs shall never contain:

* Passwords
* Password hashes
* MFA secrets
* Reset tokens
* API keys
* Provider credentials
* Session secrets

### SEC-055 — Sensitive Payload Minimization

Logs shall avoid unnecessary full patient records, medication information, message content, or provider payloads.

### SEC-056 — Security Event Logging

Important security events shall be logged and/or audited according to the requirements of Step 15.

### SEC-057 — Log Access Control

Logs containing operational or security information shall only be accessible to authorized personnel.

---

# 19.13 Audit and Security Monitoring

### SEC-058 — Audit Integration

Security-sensitive events shall comply with the Audit Log Requirements defined in Step 15.

### SEC-059 — Security Monitoring

The system shall provide operational visibility into important security-related failures and suspicious activity where practical for Phase 1.

Examples include:

* Repeated failed authentication
* Excessive password-reset requests
* Repeated authorization failures
* Webhook verification failures
* Abnormal notification activity
* Repeated provider authentication failures

### SEC-060 — Audit Integrity

Audit records shall not be editable by ordinary application users.

Security controls shall prevent unauthorized modification of audit history.

**Cross-Reference:** Enforces Step 15 append-only audit requirements.

### SEC-061 — Critical Security Failure Visibility

Critical security-control failures shall be detectable by authorized operators.

---

# 19.14 Frontend Security

### SEC-062 — Frontend Is Not a Security Boundary

Frontend controls shall be treated as usability controls, not the primary security boundary.

All important authorization decisions shall be enforced by the backend.

### SEC-063 — Sensitive Data Exposure

The frontend shall not unnecessarily expose secrets or sensitive internal information.

### SEC-064 — Secure Session Handling

Frontend authentication/session handling shall follow the secure session requirements established by Step 4B and the application's security architecture.

---

# 19.15 Deployment and Infrastructure Security

### SEC-065 — Secure Deployment Configuration

Production deployment shall use secure configuration appropriate to the hosting platforms selected for the system.

The Phase 1 architecture shall support the selected Vercel frontend and Render backend deployment model.

### SEC-066 — Production Secret Configuration

Production secrets shall be supplied through secure deployment configuration rather than source code.

### SEC-067 — Debug Mode

Production deployments shall not expose development/debug functionality that could disclose sensitive internal information.

### SEC-068 — Dependency Security

Application dependencies shall be monitored for known security vulnerabilities and updated appropriately.

### SEC-069 — Secure Dependency Management

Dependencies shall be obtained from trusted sources and managed using reproducible dependency configuration.

---

# 19.16 Transport and Network Security

### SEC-070 — Encrypted Transport

Sensitive application communication shall use encrypted transport.

### SEC-071 — Secure External Communication

Communication with external providers and services shall use secure provider-supported transport mechanisms.

### SEC-072 — Insecure Transport Protection

The production system shall not intentionally expose authenticated or sensitive application functionality through insecure transport.

---

# 19.17 Backup and Recovery Security

### SEC-073 — Protected Backups

Backups containing patient or system-sensitive information shall be protected against unauthorized access.

### SEC-074 — Backup Access Control

Only authorized personnel or systems shall access production backups.

### SEC-075 — Recovery Security

Recovery procedures shall preserve applicable access controls, authorization boundaries, and data integrity.

### SEC-076 — Backup Testing

Backup and recovery procedures shall be tested sufficiently to establish that critical data can be recovered.

---

# 19.18 Secure Development

### SEC-077 — Code Review

Security-sensitive changes shall be reviewed before production deployment.

### SEC-078 — Security-Sensitive Areas

Additional review shall be applied to changes involving:

* Authentication
* Authorization
* Patient data
* Adherence processing
* Notification orchestration
* Webhooks
* Provider credentials
* Database access
* Audit records

### SEC-079 — No Hard-Coded Credentials

Source code shall not contain production credentials, secrets, tokens, or private keys.

### SEC-080 — Security Testing

Phase 1 shall include security testing appropriate to the application's risk, including authentication, authorization, input validation, webhook handling, and sensitive-data exposure testing.

---

# 19.19 Security Incident Handling

### SEC-081 — Security Incident Identification

The system shall provide sufficient logging and monitoring to help identify significant security incidents.

### SEC-082 — Credential Compromise Response

The operational design shall support revocation or replacement of compromised credentials and provider secrets.

### SEC-083 — Account Compromise Response

The system shall support appropriate administrative actions for compromised staff accounts, including account deactivation and credential reset.

### SEC-084 — Security Incident Auditability

Significant security incidents and security-response actions shall be appropriately documented or audited.

---

# 19.20 Phase 1 Security Boundaries

The following are not required as dedicated Phase 1 security infrastructure unless later approved:

* Multi-region security architecture
* Kubernetes security infrastructure
* Service mesh security
* Dedicated SIEM platform
* Advanced behavioral threat detection
* AI-based security monitoring
* Zero-trust enterprise network architecture
* Hardware security modules
* Full enterprise identity-provider integration
* Advanced penetration-testing infrastructure

This does not remove the underlying security requirements that are applicable to the Phase 1 system.

---

# 19.21 Security Requirement Ownership

To prevent conflicting requirements:

| Security Area                | Primary Requirement Section |
| ---------------------------- | --------------------------- |
| Login behavior               | Step 4B                     |
| Admin MFA requirement        | Step 4B                     |
| Doctor MFA extensibility     | Step 4B                     |
| Password recovery behavior   | Step 4B                     |
| Session behavior             | Step 4B                     |
| Account status               | Step 4B                     |
| First Admin bootstrap        | Step 4B                     |
| Role authorization           | Step 19                     |
| Patient-data protection      | Step 19                     |
| API security                 | Step 19                     |
| Webhook security             | Step 19                     |
| Provider credential security | Step 19                     |
| Database security            | Step 19                     |
| Sensitive-data logging       | Step 19                     |
| Security monitoring          | Step 19                     |
| Authentication event audit   | Step 15                     |
| Business audit history       | Step 15                     |
| Notification behavior        | Step 10B                    |
| Telegram behavior            | Step 11                     |
| SMS behavior                 | Step 12                     |
| Voice behavior               | Step 13                     |
| Adherence behavior           | Step 14B                    |

---

# 19.22 Security Acceptance Criteria

Step 19 shall be considered satisfied for Phase 1 when:

1. Step 4B authentication requirements are implemented as specified.
2. Admin MFA is mandatory and securely implemented.
3. Doctor and Admin authorization boundaries are enforced server-side.
4. Doctors cannot access patients outside their authorized scope.
5. Patient, medication, schedule, reminder, and adherence APIs enforce object-level authorization.
6. Production secrets are not stored in source control or frontend code.
7. Provider credentials are protected.
8. Incoming communication webhooks are verified using provider-supported authenticity verification and processed idempotently.
9. Invalid or unauthorized patient responses cannot modify adherence.
10. Sensitive information is minimized in logs and API responses.
11. Authentication and security-sensitive events are appropriately auditable.
12. Rate limiting protects authentication, recovery, and abuse-prone endpoints.
13. Production deployment does not expose debug information or development credentials.
14. Dependencies are checked for known security vulnerabilities.
15. Security-sensitive functionality has been tested before release.
16. Backup and recovery controls protect production data.
17. No hidden authentication bypass, universal credential, or undocumented security backdoor exists.
18. Deactivated staff accounts cannot access protected functionality (server-side enforced).
19. Data retention follows documented retention policy with controlled deletion procedures.
20. Webhook callbacks use opaque references and do not contain sensitive clinical data.

---

# 19.23 Relationship With Other Requirements

Step 19 shall reinforce, not override, requirements established elsewhere.

Where a security control affects authentication behavior, Step 4B remains the authoritative source for the required authentication behavior.

Where a security control affects notification behavior, Steps 10B–13 remain authoritative for the communication workflow.

Where a security control affects adherence decisions, Step 14B remains authoritative for adherence behavior.

Where a security control affects audit records, Step 15 remains authoritative for audit requirements.

Step 19 defines the security controls necessary to ensure those requirements are implemented and operated securely.

**Cross-Document Consistency:** Security controls must support critical invariants established in normalized requirements:
- Communication grouping never merges adherence records (NOTIF-INVARIANT-001)
- Provider failure never becomes NOT_TAKEN (NOTIF-INVARIANT-002) 
- Delivery never equals adherence (NOTIF-INVARIANT-003)
- Only eligible response channel can change adherence (ADH-PRINCIPLE-004)
- Closed channels cannot change adherence (ADH-REQ-002)
- Webhook authenticity verification for state-changing operations (TEL-025, SMS-025, VOI-031)
