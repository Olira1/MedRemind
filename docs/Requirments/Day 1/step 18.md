Good. **Step 18 is Non-Functional Requirements (NFRs).**

This section is important because it defines **how well the system must work**, not just what features it has. For a healthcare reminder system, reliability, security, performance, recoverability, and maintainability are especially important.

I’ll keep this focused on **Phase 1** and avoid adding unnecessary enterprise-level requirements.

### Step 18 — Proposed Non-Functional Requirements

# 18. Non-Functional Requirements

## 18.1 Purpose

The system shall satisfy non-functional requirements covering reliability, performance, availability, scalability, maintainability, observability, recoverability, compatibility, usability, and operational readiness.

These requirements apply to the Doctor/Admin web application, backend services, reminder engine, notification orchestration, Telegram integration, SMS integration, Voice integration, database, and supporting infrastructure.

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

Temporary provider failures shall be handled through the defined retry and escalation mechanisms.

### NFR-007 — Historical Integrity

Notification, reminder, adherence, and audit history shall not be silently lost or overwritten because of operational failures.

---

## 18.3 Performance

### NFR-008 — Normal API Responsiveness

Under normal expected Phase 1 load, common API requests should normally return within an acceptable response time suitable for interactive web use.

### NFR-009 — Background Processing

Time-consuming operations such as notification delivery, retry processing, audio generation, and scheduled reminder execution shall not unnecessarily block interactive API requests.

### NFR-010 — Reminder Execution Timeliness

The reminder engine shall process scheduled jobs as close as reasonably possible to their configured execution time, subject to infrastructure and provider limitations.

### NFR-011 — Dashboard Performance

Dashboard requests shall remain responsive under the expected Phase 1 workload and shall avoid unnecessary retrieval of large historical datasets.

### NFR-012 — Database Efficiency

Database queries shall use appropriate indexes, filtering, pagination, and query design to avoid unnecessary performance degradation.

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

---

## 18.6 Scalability

### NFR-021 — Phase 1 Scale Targets

The architecture shall support the Phase 1 scale targets without requiring a fundamental architectural redesign.

**Approved Phase 1 Scale Targets:**
- Up to 1,500 patients
- Up to 100 doctors  
- Up to 3 Admins
- Design target: approximately 4,500 reminder occurrences per day
- Capacity target: at least 1,000 reminder occurrences per hour
- At least 100 active concurrent users

These are design targets for architecture validation, not predictions of actual usage.

### NFR-022 — Horizontal Worker Scaling

The reminder/notification processing architecture should allow background workers to be scaled independently when workload increases.

### NFR-023 — Provider-Independent Scaling

Increasing notification volume shall not require changes to core reminder or adherence business logic.

### NFR-024 — Database Growth

The database design shall account for growth in reminder, notification, adherence, and audit records through appropriate indexing, pagination, and retention/archival planning.

---

## 18.7 Security

### NFR-025 — Secure Authentication

Authentication credentials and authentication flows shall follow secure industry practices.

### NFR-026 — Password Protection

Passwords shall never be stored in plaintext and shall use an appropriate secure password-hashing mechanism.

### NFR-027 — Authorization Enforcement

Authorization shall be enforced on the backend and shall not rely solely on frontend restrictions.

### NFR-028 — Role Isolation

Doctor and Admin permissions shall be enforced consistently across API endpoints and relevant data operations.

### NFR-029 — Doctor Patient Isolation

Doctors shall not access patients, medications, reminders, adherence records, or other protected information outside their authorization scope.

### NFR-030 — Secret Protection

API keys, provider credentials, Telegram credentials, database credentials, reset tokens, and other secrets shall not be committed to source control.

### NFR-031 — Secure Transport

Production application and API communication shall use HTTPS/TLS.

### NFR-032 — Input Validation

User-provided and externally supplied data shall be validated and sanitized according to its context.

### NFR-033 — Webhook Security

External provider webhooks shall be authenticated or verified using the provider's supported security mechanisms and processed idempotently.

### NFR-034 — Rate Limiting

Security-sensitive endpoints and externally exposed endpoints shall have appropriate rate limiting or abuse protection.

---

## 18.8 Privacy and Data Minimization

### NFR-035 — Minimum Necessary Data

The system shall collect and process only data necessary for Phase 1 functionality.

### NFR-036 — Sensitive Data Minimization

Sensitive clinical information shall not unnecessarily appear in logs, notification metadata, webhook records, or technical error messages.

### NFR-037 — Notification Privacy

SMS, Telegram, and Voice content shall expose only the information necessary to communicate the medication reminder.

### NFR-038 — Logging Privacy

Application and infrastructure logs shall avoid unnecessary patient-identifying and clinical information.

### NFR-039 — Audit Privacy

Audit records shall contain sufficient information for accountability without unnecessarily duplicating complete clinical records.

---

## 18.9 Observability and Monitoring

### NFR-040 — Structured Logging

Important application events shall be logged in a structured and searchable format.

### NFR-041 — Correlation

Important multi-step operations should use correlation/request identifiers so related API, job, notification, and provider events can be traced.

### NFR-042 — Notification Monitoring

The system shall provide sufficient operational information to identify notification failures, repeated provider failures, and abnormal notification behavior.

### NFR-043 — Background Job Monitoring

Failures of critical reminder and notification jobs shall be detectable.

### NFR-044 — Error Monitoring

Unexpected application errors shall be detectable through appropriate error monitoring.

### NFR-045 — No Sensitive Secrets in Logs

Logs shall never intentionally contain passwords, reset tokens, API secrets, provider credentials, or equivalent security credentials.

---

## 18.10 Backup and Recovery

### NFR-046 — Database Backup

Production database data shall be backed up using an appropriate backup mechanism provided by the selected infrastructure.

### NFR-047 — Recovery Targets

The deployment shall have documented recovery procedures meeting the following targets:

- **RPO (Recovery Point Objective)**: ≤ 15 minutes
- **RTO (Recovery Time Objective)**: ≤ 1 hour

These targets shall be verified against actual deployment infrastructure capabilities before production launch.

### NFR-048 — Backup Verification

Backups should be periodically verified to ensure they are usable for recovery.

### NFR-049 — Historical Data Recovery

Recovery procedures shall preserve critical reminder, adherence, notification, patient, medication, and audit history to the extent supported by the backup point.

---

## 18.11 Maintainability

### NFR-050 — Modular Architecture

The system shall be organized into maintainable modules with clear separation of responsibilities.

### NFR-051 — Provider Abstraction

Notification providers shall remain replaceable without requiring changes to core reminder, escalation, or adherence logic.

### NFR-052 — Configuration Over Hard-Coding

Appropriate operational values such as notification provider configuration, escalation boundaries, and environment-specific settings shall be configurable rather than unnecessarily hard-coded.

### NFR-053 — Environment Separation

Development, testing/staging, and production configurations shall be separated.

### NFR-054 — Database Migration Management

Database schema changes shall use controlled, versioned migrations.

### NFR-055 — Code Quality

Production code shall follow consistent project conventions and shall avoid unnecessary duplication and tightly coupled components.

---

## 18.12 Testability

### NFR-056 — Automated Testing

Critical business logic shall have automated tests.

### NFR-057 — Reminder Testing

Reminder scheduling, cancellation, rescheduling, duplicate prevention, and restart/retry behavior shall be testable.

### NFR-058 — Notification Testing

Notification orchestration, retries, escalation, provider failures, and response handling shall be testable without requiring real SMS or Voice calls.

### NFR-059 — Adherence Testing

Adherence rules, active response windows, late responses, duplicate responses, and conflicting events shall be covered by automated tests.

### NFR-060 — Authorization Testing

Role and patient-access restrictions shall be tested at the backend/API level.

---

## 18.13 Deployment and Operations

### NFR-061 — Reproducible Development Environment

The application shall provide a reproducible containerized development environment using Docker.

This ensures consistent development setup, dependencies, and build process across development team members.

### NFR-062 — Reproducible Deployment

The application shall be deployable using documented and repeatable configuration.

### NFR-063 — Environment Variables and Secrets

Environment-specific secrets and configuration shall be managed through the deployment platform's secure configuration mechanism rather than source code.

### NFR-064 — Docker Configuration

Docker container configuration shall be appropriate for backend/application services and tested for compatibility with the selected hosting architecture.

Current approved architecture: Frontend (Vercel), Backend (Render), Database (Managed PostgreSQL), Redis (Managed Redis).

### NFR-065 — Health Monitoring

Backend services and critical background processing shall expose sufficient health information for operational monitoring.

### NFR-066 — Graceful Shutdown

Application workers shall handle shutdown in a way that minimizes interrupted or lost processing.

### NFR-067 — Deployment Safety

Application deployments shall minimize the risk of corrupting or losing scheduled reminder and notification processing.

---

## 18.14 Compatibility

### NFR-068 — Modern Web Browsers

The Doctor/Admin web application shall support current commonly used modern browsers.

### NFR-069 — Responsive Interface

The web interface should remain usable on common desktop, tablet, and mobile screen sizes relevant to Phase 1 users.

### NFR-070 — API Compatibility

Internal API contracts shall be versioned or managed in a way that prevents uncontrolled breaking changes between frontend and backend.

---

## 18.15 Usability

### NFR-071 — Clear Status Representation

The interface shall clearly distinguish:

* TAKEN
* NOT_TAKEN
* NO_RESPONSE
* notification failure
* pending/escalation states

### NFR-072 — Actionable Errors

User-facing errors shall explain what went wrong and, where appropriate, what action the user can take.

### NFR-073 — Confirmation of Critical Actions

Actions that can materially affect patient reminders, medications, schedules, access, or configuration should provide appropriate confirmation.

### NFR-074 — Localization Consistency

Supported user-facing languages shall use the centralized localization mechanism consistently.

### NFR-075 — Accessibility

The web interface should follow reasonable accessibility practices, including readable text, keyboard-accessible controls, clear labels, and appropriate semantic UI structure.

---

## 18.16 Configuration and Change Safety

### NFR-076 — Controlled Configuration

Configuration changes shall be validated before becoming active.

### NFR-077 — Configuration Auditability

Significant configuration changes shall be traceable to the user or system actor that made them.

### NFR-078 — Future-Only Configuration Changes

Changes to reminder/escalation configuration shall not silently rewrite historical reminder or adherence outcomes.

### NFR-079 — Safe Defaults

The system shall provide validated default configuration sufficient to operate safely when an administrator has not customized optional settings.

---

## 18.17 Operational Transparency

### NFR-080 — Distinguish Business and Technical Failure

The system shall distinguish patient adherence outcomes from technical notification/provider failures.

### NFR-081 — Provider Status Mapping

External provider statuses shall be mapped into a consistent internal status model.

### NFR-082 — Traceability

A reminder occurrence shall be traceable through its notification attempts, patient responses, adherence outcome, and relevant audit events.

### NFR-083 — No Silent Failure

Critical failures affecting reminder creation, notification processing, adherence processing, or data persistence shall be detectable and shall not silently disappear.

---

## 18.18 Phase 1 Scope Boundary

These non-functional requirements define the quality and operational characteristics required for Phase 1.

They do not require Phase 1 to implement:

* multi-region infrastructure,
* active-active disaster recovery,
* enterprise-scale data warehousing,
* advanced analytics infrastructure,
* Kubernetes,
* complex service meshes,
* AI-based monitoring,
* real-time WebSocket infrastructure,
* or other infrastructure whose complexity is not justified by Phase 1 requirements.

The implementation shall remain production-oriented while avoiding unnecessary infrastructure complexity.

This gives us a **production-quality baseline without overengineering Phase 1**.

Please review Step 18 carefully. In particular, I want your decision on **NFR-047/048 (recovery and backup), NFR-067 (responsive UI), and NFR-073 (accessibility)**. If you approve Step 18, we'll lock it and move to **Step 19 — Security Requirements**.
