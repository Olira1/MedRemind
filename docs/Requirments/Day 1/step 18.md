# STEP 18 — NON-FUNCTIONAL REQUIREMENTS

## 18.1 Purpose

The system shall satisfy non-functional requirements covering reliability, performance, availability, scalability, data integrity, security, privacy, maintainability, observability, recoverability, deployment readiness, compatibility, and usability.

**Cross-Document Authority:** Step 4B defines authentication security requirements. Step 10B-13 define notification business logic. Step 14B defines adherence business logic. Step 15 defines audit requirements. Step 19 defines detailed security controls. This document defines system-wide quality attributes and operational constraints that support those functional requirements.

These requirements apply to the Doctor/Admin web application, backend services, reminder engine, notification orchestration, communication channel integrations, database, and supporting infrastructure for Phase 1 deployment.

---

## 18.2 Reliability

### NFR-001 — Reminder Reliability

The system shall reliably create and process scheduled reminder occurrences without silently losing scheduled reminders.

### NFR-002 — Persistent Scheduled Jobs

Reminder jobs shall be persisted so that application restarts or temporary worker failures do not silently discard scheduled work.

### NFR-003 — Duplicate Prevention

The system shall prevent duplicate reminder occurrences and duplicate notification sends caused by retries, concurrent workers, webhook retries, or application restarts.

### NFR-004 — Idempotent Processing

Reminder processing, notification processing, incoming responses, webhook handling, and adherence processing shall be designed to be safely repeatable where applicable.

### NFR-005 — Failure Isolation

A failure in one notification channel or provider shall not cause the entire reminder system to fail.

### NFR-006 — Provider Failure Handling

Temporary provider failures shall be handled through the defined retry and escalation mechanisms without converting provider failure into patient non-adherence.

**Critical Rule:** Provider failure shall never automatically become NOT_TAKEN adherence status.

### NFR-007 — Historical Integrity

Notification, reminder, adherence, and audit history shall not be silently lost or overwritten because of operational failures, configuration changes, or system updates.

---

## 18.3 Performance

### NFR-008 — Normal API Responsiveness

Under normal expected Phase 1 load, common API requests shall return within response times suitable for interactive web use.

### NFR-009 — Background Processing

Time-consuming operations such as notification delivery, retry processing, audio generation, and scheduled reminder execution shall not unnecessarily block interactive API requests.

### NFR-010 — Reminder Execution Timeliness

The reminder engine shall process scheduled jobs as close as reasonably possible to their configured execution time, subject to infrastructure and provider limitations.

### NFR-011 — Dashboard Performance

Dashboard requests shall remain responsive under the expected Phase 1 workload and shall avoid unnecessary retrieval of large historical datasets.

### NFR-012 — Database Efficiency

Database queries shall use appropriate indexes, filtering, pagination, and query design to avoid unnecessary performance degradation under Phase 1 data volumes.

---

## 18.4 Availability

### NFR-013 — Service Availability

The system shall be designed for reliable availability appropriate for a healthcare reminder service.

### NFR-014 — Graceful Recovery

Temporary backend, worker, Redis, database, or provider interruptions shall be recoverable without silently losing reminder or notification work.

### NFR-015 — No Single In-Process Scheduler Dependency

Reminder scheduling shall not depend solely on an in-memory timer inside a web-server process.

### NFR-016 — Deployment Restart Safety

Routine application deployment or service restart shall not cause scheduled reminders to disappear.

---

## 18.5 Data Integrity

### NFR-017 — Transactional Integrity

Related critical database changes shall use appropriate transactional mechanisms where consistency requires multiple changes to succeed together.

### NFR-018 — Referential Integrity

Relationships between patients, medications, schedules, reminders, notifications, adherence records, and audit records shall maintain database integrity.

### NFR-019 — Historical Preservation

Historical reminder, notification, adherence, and audit records shall remain traceable after medication changes, schedule changes, patient transfer, or record archival.

### NFR-020 — UTC Timestamps

System timestamps shall be stored consistently in UTC, while user-facing times shall be converted according to the applicable timezone.

### NFR-021 — In-Flight Contact Changes

When a notification attempt is created, it shall capture the destination/contact information applicable at creation time. Future reminder occurrences shall use the patient's current contact configuration. This prevents later contact changes from silently rewriting historical notification destinations.

### NFR-022 — Phone Number Non-Uniqueness

The system shall NOT assume phone numbers uniquely identify patients. Multiple patients may share the same phone number. Internal Patient ID remains the primary patient identity.

### NFR-023 — Telegram Identity Preservation

Telegram link/unlink/relink operations shall not corrupt patient identity or historical reminder/adherence records. Historical records shall remain preserved when Telegram association changes.

### NFR-024 — Combined Reminder Integrity

When multiple reminder occurrences are combined into one communication (Telegram/SMS/Voice), each reminder occurrence shall remain independently traceable with its own adherence decision. Communication grouping shall not merge adherence records.

---

## 18.6 Scalability

### NFR-025 — Phase 1 Scale Targets

The architecture shall support the Phase 1 scale targets without requiring fundamental architectural redesign.

**Approved Phase 1 Scale Targets:**
- Up to 1,500 patients
- Up to 100 doctors  
- Up to 3 Admins
- Design target: approximately 4,500 reminder occurrences per day
- Capacity target: at least 1,000 reminder occurrences per hour
- At least 100 active concurrent users

**Important:** These are design targets for architecture validation and capacity planning, not predictions of actual usage patterns or guaranteed production traffic levels.

### NFR-026 — Horizontal Worker Scaling

The reminder/notification processing architecture shall allow background workers to be scaled independently when workload increases.

### NFR-027 — Provider-Independent Scaling

Increasing notification volume shall not require changes to core reminder or adherence business logic.

### NFR-028 — Database Growth Planning

The database design shall account for growth in reminder, notification, adherence, and audit records through appropriate indexing, pagination, and retention/archival planning.

---

## 18.7 Security

### NFR-029 — Secure Authentication

Authentication credentials and authentication flows shall follow secure industry practices as defined in Step 4B.

### NFR-030 — Password Protection

Passwords shall never be stored in plaintext and shall use appropriate secure password-hashing mechanisms.

### NFR-031 — Server-Side Authorization

Authorization shall be enforced on the backend and shall not rely solely on frontend restrictions.

### NFR-032 — Role Isolation

Doctor and Admin permissions shall be enforced consistently across API endpoints and relevant data operations.

### NFR-033 — Doctor Patient Isolation

Doctors shall not access patients, medications, reminders, adherence records, or other protected information outside their authorization scope.

### NFR-034 — Deactivated Staff Access

Deactivated or inactive staff accounts shall not continue accessing protected functionality. Enforcement shall be server-side and shall not rely solely on frontend session management.

### NFR-035 — Secret Protection

API keys, provider credentials, communication channel credentials, database credentials, reset tokens, and other secrets shall not be committed to source control or exposed in logs.

### NFR-036 — Secure Transport

Production application and API communication shall use HTTPS/TLS.

### NFR-037 — Input Validation

User-provided and externally supplied data shall be validated and sanitized according to its context.

### NFR-038 — Webhook Security

External provider webhooks shall be authenticated or verified using the provider's supported security mechanisms and processed idempotently per Step 19 requirements.

### NFR-039 — Rate Limiting

Security-sensitive endpoints and externally exposed endpoints shall have appropriate rate limiting or abuse protection.

---

## 18.8 Privacy and Data Minimization

### NFR-040 — Minimum Necessary Data

The system shall collect and process only data necessary for Phase 1 functionality.

### NFR-041 — Sensitive Data Minimization

Sensitive clinical information shall not unnecessarily appear in logs, notification metadata, webhook records, or technical error messages.

### NFR-042 — Notification Privacy

SMS, Telegram, and Voice content shall expose only information necessary to communicate the medication reminder per approved communication requirements.

### NFR-043 — Logging Privacy

Application and infrastructure logs shall avoid unnecessary patient-identifying and clinical information.

### NFR-044 — Audit Privacy

Audit records shall contain sufficient information for accountability without unnecessarily duplicating complete clinical records per Step 15 requirements.

---

## 18.9 Observability and Monitoring

### NFR-045 — Structured Logging

Important application events shall be logged in a structured and searchable format.

### NFR-046 — Correlation

Important multi-step operations shall use correlation/request identifiers so related API, job, notification, and provider events can be traced.

### NFR-047 — Notification Monitoring

The system shall provide sufficient operational information to identify notification failures, repeated provider failures, and abnormal notification behavior.

### NFR-048 — Background Job Monitoring

Failures of critical reminder and notification jobs shall be detectable through appropriate monitoring.

### NFR-049 — Error Monitoring

Unexpected application errors shall be detectable through appropriate error monitoring mechanisms.

### NFR-050 — No Sensitive Secrets in Logs

Logs shall never intentionally contain passwords, reset tokens, API secrets, provider credentials, or equivalent security credentials.

---

## 18.10 Backup and Recovery

### NFR-051 — Database Backup

Production database data shall be backed up using appropriate backup mechanisms provided by the selected infrastructure.

### NFR-052 — Recovery Targets

The deployment shall have documented recovery procedures meeting the following targets:

- **RPO (Recovery Point Objective)**: ≤ 15 minutes
- **RTO (Recovery Time Objective)**: ≤ 1 hour

**Important:** These targets shall be verified against actual deployment infrastructure capabilities before production launch. The implementation must confirm that selected infrastructure (Vercel, Render, managed PostgreSQL, managed Redis) can support these objectives.

### NFR-053 — Backup Verification

Backups shall be periodically verified to ensure they are usable for recovery.

### NFR-054 — Historical Data Recovery

Recovery procedures shall preserve critical reminder, adherence, notification, patient, medication, and audit history to the extent supported by the backup point.

---

## 18.11 Maintainability

### NFR-055 — Modular Architecture

The system shall be organized into maintainable modules with clear separation of responsibilities.

### NFR-056 — Provider Abstraction

Notification providers (Telegram, SMS, Voice) shall remain replaceable without requiring changes to core reminder, escalation, or adherence logic.

**Architectural Requirement:** Core application shall communicate with notification providers through abstraction interfaces that isolate provider-specific implementation details.

### NFR-057 — Configuration Over Hard-Coding

Appropriate operational values such as notification provider configuration, escalation boundaries, and environment-specific settings shall be configurable rather than unnecessarily hard-coded.

### NFR-058 — Environment Separation

Development, testing/staging, and production configurations shall be separated and managed through appropriate secure mechanisms.

### NFR-059 — Database Migration Management

Database schema changes shall use controlled, versioned migrations.

### NFR-060 — Code Quality

Production code shall follow consistent project conventions and shall avoid unnecessary duplication and tightly coupled components where reasonable.

---

## 18.12 Testability

### NFR-061 — Automated Testing

Critical business logic shall have automated tests covering essential functional requirements.

### NFR-062 — Reminder Testing

Reminder scheduling, cancellation, rescheduling, duplicate prevention, and restart/retry behavior shall be testable.

### NFR-063 — Notification Testing

Notification orchestration, retries, escalation, provider failures, and response handling shall be testable without requiring real SMS or Voice calls in development/test environments.

### NFR-064 — Adherence Testing

Adherence rules, active response windows, late responses, duplicate responses, and conflicting events shall be covered by automated tests.

### NFR-065 — Authorization Testing

Role and patient-access restrictions shall be tested at the backend/API level to verify server-side enforcement.

---

## 18.13 Deployment and Operations

### NFR-066 — Reproducible Development Environment

The application shall provide a reproducible containerized development environment using Docker.

This ensures consistent development setup, dependencies, build process, and testing environment across development team members.

### NFR-067 — Reproducible Deployment

The application shall be deployable using documented and repeatable configuration appropriate for the selected infrastructure.

### NFR-068 — Environment Variables and Secrets

Environment-specific secrets and configuration shall be managed through the deployment platform's secure configuration mechanism rather than source code.

### NFR-069 — Docker Configuration

Docker container configuration shall be tested and validated for:
- Reproducible development environment
- Consistent application builds
- Local and integration testing environment
- Portable backend/application build processes

**Deployment Architecture:** Current approved architecture: Frontend (Vercel), Backend (Render), Database (Managed PostgreSQL), Redis (Managed Redis). Docker supports development/build workflows; production components use managed services as specified.

### NFR-070 — Health Monitoring

Backend services and critical background processing shall expose sufficient health information for operational monitoring.

### NFR-071 — Graceful Shutdown

Application workers shall handle shutdown in a way that minimizes interrupted or lost processing.

### NFR-072 — Deployment Safety

Application deployments shall minimize the risk of corrupting or losing scheduled reminder and notification processing.

### NFR-073 — Telegram-First Production

The system shall support initial production deployment with Telegram active and SMS/Voice integrations prepared but inactive. Architecture shall support later activation of SMS/Voice channels without rewriting core reminder logic.

---

## 18.14 Compatibility

### NFR-074 — Modern Web Browsers

The Doctor/Admin web application shall support current commonly used modern browsers.

### NFR-075 — Responsive Interface

The web interface shall remain usable on common desktop, tablet, and mobile screen sizes relevant to Phase 1 users.

### NFR-076 — API Compatibility

Internal API contracts shall be versioned or managed in a way that prevents uncontrolled breaking changes between frontend and backend.

---

## 18.15 Usability

### NFR-077 — Clear Status Representation

The interface shall clearly distinguish adherence states and notification outcomes:
* TAKEN (patient explicitly indicated medication was taken)
* NOT_TAKEN (patient explicitly indicated medication was not taken)
* NO_RESPONSE (response window closed without valid accepted response)
* Notification failure (technical delivery problem)
* Pending/escalation states

### NFR-078 — Actionable Errors

User-facing errors shall explain what went wrong and, where appropriate, what action the user can take.

### NFR-079 — Confirmation of Critical Actions

Actions that can materially affect patient reminders, medications, schedules, access, or configuration shall provide appropriate confirmation.

### NFR-080 — Localization Consistency

Supported user-facing languages (English, Amharic, Afaan Oromoo) shall use the centralized localization mechanism consistently.

### NFR-081 — Accessibility

The web interface shall follow reasonable accessibility practices including:
* Readable text and adequate contrast
* Keyboard-accessible controls where applicable
* Clear and meaningful labels
* Appropriate semantic UI structure
* Distinguishable loading, empty, and error states

---

## 18.16 Configuration and Change Safety

### NFR-082 — Controlled Configuration

Configuration changes shall be validated before becoming active per Step 17 requirements.

### NFR-083 — Configuration Auditability

Significant configuration changes shall be traceable to the user or system actor that made them per Step 15 requirements.

### NFR-084 — Future-Only Configuration Changes

Changes to reminder/escalation configuration shall apply to future reminder occurrences and shall not silently rewrite historical reminder or adherence outcomes.

### NFR-085 — Safe Defaults

The system shall provide validated default configuration sufficient to operate safely when an administrator has not customized optional settings.

---

## 18.17 Operational Transparency

### NFR-086 — Distinguish Business and Technical Failure

The system shall distinguish patient adherence outcomes from technical notification/provider failures.

**Critical Distinction:** Notification delivery status is separate from adherence status. Provider failures do not equal patient non-adherence.

### NFR-087 — Provider Status Mapping

External provider statuses shall be mapped into a consistent internal notification status model.

### NFR-088 — Traceability

A reminder occurrence shall be traceable through its notification attempts, patient responses, adherence outcome, and relevant audit events.

### NFR-089 — No Silent Failure

Critical failures affecting reminder creation, notification processing, adherence processing, or data persistence shall be detectable and shall not silently disappear.

---

## 18.18 Data Retention

### NFR-090 — Retention Policy

Reminder, adherence, notification, and audit history shall be retained per documented/configurable retention policy.

### NFR-091 — No Invented Retention Periods

The system shall not assume specific legal retention durations unless explicitly established by organizational or legal requirements.

Exact retention period shall be specified before production based on applicable requirements.

### NFR-092 — Controlled Deletion

Historical data retention and deletion actions shall be controlled, documented, and auditable. Records shall not be silently deleted through ordinary business operations.

---

## 18.19 Cross-Document Authority

To prevent conflicting requirements, the following document authority applies:

| Quality Area | Primary Requirement Section |
|--------------|----------------------------|
| Authentication security details | Step 4B |
| Notification business logic | Steps 10B-13 |
| Adherence business logic | Step 14B |
| Audit requirements | Step 15 |
| Dashboard performance | Step 16 |
| Settings validation | Step 17 |
| Detailed security controls | Step 19 |
| System-wide quality attributes | Step 18 |
| Performance and capacity targets | Step 18 |
| Recovery objectives | Step 18 |
| Data integrity principles | Step 18 |

**Critical Rule:** Step 18 defines quality attributes and operational constraints. Where NFRs affect functional behavior governed by other documents, those documents remain authoritative for business logic while Step 18 establishes quality expectations.

---

## 18.20 Phase 1 Scope Boundary

These non-functional requirements define the quality and operational characteristics required for Phase 1 production deployment.

Phase 1 does NOT require implementation of:
* Multi-region infrastructure
* Active-active disaster recovery architecture
* Enterprise-scale data warehousing
* Advanced analytics infrastructure
* Kubernetes orchestration
* Complex service meshes
* AI-based monitoring systems
* Advanced real-time WebSocket infrastructure beyond operational needs
* Infrastructure complexity not justified by Phase 1 requirements

The implementation shall remain production-oriented and operationally sound while avoiding unnecessary infrastructure complexity for Phase 1 scale.

---

## 18.21 NFR Acceptance Criteria

Step 18 shall be considered satisfied for Phase 1 when:

1. System supports approved Phase 1 scale targets (1,500 patients, 100 doctors, 3 Admins, 4,500 reminders/day, 1,000 reminders/hour capacity, 100 active users).
2. Reminder jobs are persisted and survive application restarts.
3. Duplicate reminder and notification prevention mechanisms are implemented and tested.
4. Provider failures do not automatically become NOT_TAKEN adherence status.
5. Historical reminder, adherence, notification, and audit records are preserved during system operations.
6. Background processing does not block interactive API requests.
7. Database queries use appropriate indexing and pagination for Phase 1 data volumes.
8. Backup/recovery procedures meet RPO ≤ 15 minutes and RTO ≤ 1 hour targets (verified against actual infrastructure).
9. Docker provides reproducible development environment and build process.
10. Provider abstraction allows notification provider replacement without rewriting core logic.
11. Communication channel enable/disable supports Telegram-first production with prepared-but-inactive SMS/Voice.
12. Phone numbers are not assumed to be globally unique patient identifiers.
13. Telegram link/unlink/relink operations preserve historical records.
14. Combined reminder communications maintain independent adherence decisions per occurrence.
15. Deactivated staff accounts cannot access protected functionality (server-side enforced).
16. Server-side authorization is enforced for all protected resources.
17. Sensitive information is minimized in logs per privacy requirements.
18. Webhook processing is idempotent and authenticated per security requirements.
19. Configuration changes apply to future occurrences without rewriting historical outcomes.
20. System clearly distinguishes adherence states (TAKEN, NOT_TAKEN, NO_RESPONSE) from notification failures.
21. Deployment architecture supports Vercel frontend, Render backend, managed PostgreSQL, managed Redis.
22. Critical business logic has automated test coverage.
23. Operational failures are detectable through monitoring and logging.
24. Retention follows documented/configurable policy without assumed legal periods.
25. Environment-specific secrets are managed through secure deployment mechanisms, not source code.

---

## 18.22 Relationship With Other Requirements

Step 18 shall support, not override, requirements established elsewhere.

Where NFRs affect authentication behavior, Step 4B remains authoritative for authentication requirements.

Where NFRs affect notification behavior, Steps 10B-13 remain authoritative for communication business logic.

Where NFRs affect adherence behavior, Step 14B remains authoritative for adherence rules.

Where NFRs affect audit behavior, Step 15 remains authoritative for audit requirements.

Where NFRs affect security controls, Step 19 remains authoritative for detailed security implementation.

Step 18 defines the quality attributes, operational constraints, and system-wide non-functional expectations necessary to ensure reliable, secure, performant, and maintainable operation of all system functions.
