# 3-Week Phase 1 Development Plan — Final Master Plan

## Purpose

The goal of this plan is to have a **working, tested, deployable Phase 1 product by the end of Week 3**.

The three weeks are intentionally divided into:

> **Week 1: Decide what to build and design how it will work.**
> **Week 2: Build the approved system.**
> **Week 3: Prove that it works, harden it, and deploy it.**

We should **not spend all three weeks coding**.

Requirements, workflows, domain/database design, API contracts, architecture, security, reliability, and deployment strategy need to be established before production implementation begins.

The project should be treated as a production-oriented system rather than a prototype.

---

# Phase 1 Scope

Our three-week target is:

```text
Doctor/Admin Portal
        │
        ├── Authentication
        ├── Admin Account Management
        ├── Patient Management
        ├── Medication Management
        ├── Medication Scheduling
        ├── Reminder Management
        ├── Adherence Monitoring
        └── Notification Monitoring
                    │
                    ├── Telegram
                    ├── SMS (Mock → provider-ready)
                    └── Voice (Mock → provider-ready)
```

The system has two primary staff roles:

```text
Admin
  │
  ├── System/operational administration
  ├── Doctor account administration
  ├── System-wide configuration boundaries
  └── Operational/security visibility

Doctor
  │
  ├── Assigned patient management
  ├── Medication management
  ├── Schedule management
  ├── Reminder configuration within allowed boundaries
  └── Adherence/notification monitoring
```

Patients do not have web application accounts in Phase 1.

Patients interact with the system through approved communication channels such as:

```text
Telegram
SMS
Voice
```

SMS and Voice should be provider-independent.

We can develop them using mock providers during Phase 1 while keeping provider interfaces ready for production providers later.

The business logic must not depend directly on a specific SMS or Voice provider.

---

# Engineering Principles

The project follows these principles throughout all three weeks.

## 1. Requirements First

We do not implement features simply because they appear useful.

The approved Phase 1 requirements control the implementation.

If a feature is not in the approved requirements, it is not added simply because it seems beneficial.

If a required behavior needs to change, the requirements must be explicitly reviewed and updated before implementation changes.

---

## 2. Design Before Implementation

Important domain, database, API, architecture, security, reliability, and deployment decisions should be made before the corresponding production code is written.

The implementation follows the approved design rather than becoming a place where major architecture decisions are made accidentally.

---

## 3. Review Before Freeze

Important designs receive an independent Claude review before they are frozen for implementation.

Claude is an independent reviewer and challenger.

A review does not automatically change the design.

Findings must be evaluated and resolved deliberately.

---

## 4. Tests Are Part of Implementation

Testing is not something we postpone until the final day.

Codex should implement appropriate tests alongside the relevant backend/frontend work.

Critical workflows should be tested as they are built.

Week 3 performs the final comprehensive testing and hardening pass.

---

## 5. Provider Independence

SMS and Voice must use provider abstractions.

Business logic must not directly call provider SDKs.

The architecture should support replacing the production provider without redesigning the reminder, notification, or adherence logic.

---

## 6. Reliability Is Designed, Not Added at the End

Retry, idempotency, duplicate prevention, queue behavior, failure handling, and recovery behavior must be designed during Week 1 and implemented during Week 2.

Week 3 verifies that those designs actually work under realistic failure scenarios.

---

## 7. Security Is Designed Before Implementation

Authentication, authorization, patient-data protection, secrets, validation, rate limiting, webhook security, and access isolation must be considered during architecture design.

Week 3 performs security verification and hardening.

---

## 8. No Uncontrolled Scope Expansion

A new feature cannot simply be added during implementation.

If something needs to change:

```text
New requirement/change
        ↓
Requirements review
        ↓
Impact assessment
        ↓
Human approval
        ↓
Requirements updated
        ↓
Design updated if necessary
        ↓
Implementation
```

---

## 9. Admin Is a First-Class Phase 1 Role

Admin functionality is part of the approved Phase 1 scope.

Admin requirements shall be reflected consistently in:

* Authentication
* MFA
* Authorization
* Account management
* Settings
* Dashboard
* Audit
* Security
* Database design
* API design
* Frontend implementation
* Testing
* Deployment/release verification

Admin functionality must not be treated as an afterthought added near the end of development.

---

## 10. Dockerized Development and Deployment Readiness

Dockerization is part of the Phase 1 engineering plan.

The project shall have reproducible containerized development support for the application components that benefit from it.

Docker configuration shall cover, as appropriate:

```text
Frontend
Backend
PostgreSQL
Redis
```

The production deployment architecture remains:

```text
Frontend → Vercel
Backend  → Render
Database → Managed PostgreSQL
Redis    → Managed Redis
```

Dockerization does not require every production service to run inside Docker.

The goal is reproducible development, consistent environments, and a portable backend/application build.

---

## 11. Vertical Feature Integration

After the backend foundation is established, frontend and backend development proceed incrementally rather than as two isolated projects.

Each major feature should follow approximately:

```text
Approved requirement
        ↓
Backend implementation
        ↓
Frontend implementation
        ↓
API integration
        ↓
Feature test
        ↓
Continue to next feature
```

Therefore, frontend/backend integration begins during Week 2 as features are implemented.

Day 15 is the **final frontend integration and completion pass**, not the first time the frontend connects to the backend.

---

# AI Responsibilities

The development process uses different AI tools for different responsibilities.

## ChatGPT

Primary system architect / technical lead.

Responsible for:

* Requirements
* Use cases
* Domain model
* Database design
* API contracts
* System architecture
* Provider abstraction
* Security/reliability design
* Implementation planning
* Technical decisions

ChatGPT does not independently change approved requirements.

---

## Claude

Independent reviewer and challenger.

Claude should not automatically redesign the system.

Its primary role is to challenge the proposed design and identify:

* Contradictions
* Missing requirements
* Domain errors
* Database integrity problems
* Concurrency problems
* Security weaknesses
* Reliability problems
* Failure modes
* Unclear behavior
* Implementation risks

Claude reviews are **quality gates**, not additional development days.

---

## Codex

Repository implementation agent.

Codex is responsible for:

* Inspecting the repository
* Implementing approved designs
* Creating migrations
* Writing production code
* Writing tests
* Running tests
* Implementing Docker configuration
* Reviewing its own changes
* Reporting changed files and results

Codex should not independently expand the approved scope or redesign frozen architecture.

If implementation reveals a genuine design problem, Codex should report it rather than silently changing the architecture.

---

## Human Decision-Maker

The user remains the final decision-maker for:

* Requirements
* Scope
* Important architectural decisions
* Requirements changes
* Release approval

---

# Source of Truth

The project should maintain a clear source of truth:

```text
Phase 1 Requirements
        ↓
Approved Design
        ↓
Implementation
        ↓
Tests
        ↓
Deployment
        ↓
Final Verification
```

The repository documentation should contain the approved:

* Requirements
* Architecture
* API contracts
* Database design
* Important architectural decisions
* Relevant implementation/deployment documentation

AI conversations should not be treated as the permanent source of truth.

---

# WEEK 1 — Requirements, Architecture & Foundation

## Goal

By the end of Week 1, we should know **exactly what we are building, how it should behave, and how the system should be structured**.

Week 1 is primarily a requirements and design week.

It removes ambiguity before production implementation begins.

---

# Day 1 — Requirements

Create the complete Phase 1 Requirements Specification.

Define requirements for:

* Product scope
* Actors
* Core workflows
* Authentication
* Admin
* Doctor
* Patients
* Medications
* Medication schedules
* Reminders
* Notifications
* Telegram
* SMS
* Voice
* Adherence
* Audit logs
* Dashboard
* Settings
* Non-functional requirements
* Security
* Privacy
* Phase 1 exclusions
* Acceptance criteria
* Requirement traceability

Each requirement receives an ID, for example:

```text
AUTH-001
ADMIN-001
PAT-001
MED-001
SCHED-001
REM-001
SMS-001
TEL-001
VOICE-001
ADH-001
AUDIT-001
DASH-001
SET-001
SEC-001
NFR-001
```

Admin requirements must explicitly cover the Admin role and its responsibilities.

Also explicitly define **what is NOT included in Phase 1**.

---

## Day 1 Requirements Review

After the requirements specification is created, Claude performs an independent requirements review.

The review should look specifically for:

* Missing requirements
* Contradictory requirements
* Ambiguous terminology
* Unclear acceptance criteria
* Hidden scope
* Missing failure behavior
* Missing Admin behavior
* Missing authentication/security requirements
* Missing reliability requirements

The review produces findings.

The requirements are not considered ready until those findings have been evaluated.

---

# Day 2 — Requirements Review & Freeze

Compare the requirements against:

* `MeRim`
* `Trial`
* Our Phase 1 goals
* The approved scope

Resolve inconsistencies.

Incorporate valid corrections identified during the Day 1 review.

Then establish:

> **Phase 1 Requirements Baseline v1.0**

After this point, no feature is casually added or removed.

Any future change must first be treated as a requirements change.

### Human Approval

The requirements baseline is explicitly approved before moving into detailed design.

---

# Day 3 — User Flows

Define the complete workflows.

For example:

```text
Doctor Login
    ↓
Dashboard
    ↓
Create Patient
    ↓
Create Medication
    ↓
Create Schedule
    ↓
Reminder generated
    ↓
Notification
    ↓
Patient responds
    ↓
Adherence recorded
    ↓
Doctor sees result
```

Also define Admin workflows.

For example:

```text
Admin Login
    ↓
Admin Dashboard
    ↓
Manage Doctors
    ↓
Manage System Configuration
    ↓
Review Operational/Security Information
```

Define failure flows:

```text
Notification fails
       ↓
Retry
       ↓
Still fails
       ↓
Failed status
       ↓
Doctor/Admin sees appropriate failure information
```

Define expected behavior for important success and failure paths.

---

## Claude Review Gate

Claude reviews the user flows and use cases for:

* Missing states
* Missing failure paths
* Inconsistent behavior
* Unclear ownership
* Incorrect domain transitions
* Missing edge cases
* Gaps between requirements and workflows

The review should challenge the flow rather than simply confirm that it looks reasonable.

---

# Day 4 — Domain + Database Design

Define the core entities.

Examples include:

```text
Admin
Doctor
Patient
Medication
MedicationSchedule
Reminder
NotificationDelivery
AdherenceRecord
TelegramConnection
AuditLog
```

Define:

* Relationships
* Primary keys
* Foreign keys
* Indexes
* Unique constraints
* Statuses
* Timestamps
* Ownership relationships
* Authorization boundaries
* Important lifecycle rules

Admin/Doctor ownership and access boundaries must be represented in the domain/database design.

---

## Important Distinction

**Day 4 is database DESIGN.**

We do not spend this day implementing production migrations merely because the schema has been designed.

The approved design becomes the input for Week 2 implementation.

Actual database schema implementation and migrations are created by Codex during Week 2.

---

## Claude Deep-Review Gate

Claude performs a deeper review of the domain and database design.

The review should specifically challenge:

* Incorrect relationships
* Missing constraints
* Incorrect cardinality
* Ownership/isolation problems
* Duplicate records
* Invalid state transitions
* Timestamp problems
* Scheduling-related data issues
* Notification/adherence semantic confusion
* Concurrency risks
* Indexing problems
* Auditability gaps
* Admin/Doctor authorization boundaries

The database design is frozen only after the review findings have been evaluated.

---

# Day 5 — API + Architecture + Security/Reliability + Docker Design

Define the API contract.

For example:

```text
POST   /auth/login

GET    /patients
POST   /patients
GET    /patients/:id
PATCH  /patients/:id

GET    /patients/:id/medications
POST   /patients/:id/medications

POST   /medications/:id/schedules

GET    /reminders
GET    /notifications
GET    /patients/:id/adherence
```

Admin endpoints and permissions are defined according to the approved requirements.

---

## System Architecture

Establish the architecture:

```text
Vercel
   ↓
React / MeRim
   ↓
NestJS API
   ↓
PostgreSQL
   ↓
Redis / BullMQ
   ↓
Notification Providers
```

Also define:

```text
SmsProvider
VoiceProvider
TelegramProvider
```

and their mock implementations.

---

# Security Design

Before implementation, define the security approach for:

* Authentication
* Admin MFA
* Authorization
* Doctor ownership
* Patient-data access
* Password handling
* Session/token behavior
* Input validation
* Rate limiting
* CORS
* Secrets
* Environment variables
* Sensitive logging
* API access
* Webhook security
* Cross-doctor data isolation

---

# Reliability Design

Before implementation, define expected behavior for:

* Reminder generation
* Scheduled jobs
* Queue failures
* Duplicate jobs
* Duplicate notifications
* Idempotency
* Retries
* Exponential backoff
* Provider failures
* Network timeouts
* Failed deliveries
* Persistent delivery status
* Medication schedule changes
* Medication discontinuation
* Recovery after worker/API failure

Important distinction:

> **Week 1 defines how the system should remain reliable. Week 2 implements it. Week 3 proves that it actually behaves that way.**

---

# Docker Design

Dockerization is explicitly planned before implementation.

Define:

* Backend Dockerfile
* Frontend Dockerfile where appropriate
* Development Docker Compose configuration
* PostgreSQL development container
* Redis development container
* Environment-variable handling
* Container networking
* Build/run expectations
* Production container requirements where applicable

The Docker setup must not introduce a separate architecture from the approved application architecture.

---

## Claude Deep-Review Gate

Claude performs the major Week 1 architecture review.

The review covers:

* API design
* System architecture
* Provider abstraction
* Security architecture
* Reliability architecture
* Queue design
* Idempotency
* Failure handling
* Domain boundaries
* Admin/Doctor authorization
* Docker architecture
* Integration risks

Important findings are resolved before the architecture is frozen.

### Human Approval

At the end of Day 5, the approved architecture becomes the implementation baseline.

---

# End of Week 1 Deliverables

We should have:

* Requirements Specification
* Phase 1 scope locked
* Requirements Baseline v1.0
* User flows
* Domain model
* Database design
* API specification
* System architecture
* Admin/Doctor authorization model
* Provider interfaces
* Security design
* Reliability design
* Docker strategy
* Development environment design
* Claude review findings resolved

**No major ambiguity should remain.**

The database design is ready for implementation, but actual migrations/schema implementation happens in Week 2.

---

# WEEK 2 — Core Development

## Goal

By the end of Week 2, the application should be **functionally working end-to-end in development**.

This is the main coding week.

The implementation follows the designs frozen during Week 1.

Codex implements the approved design and writes tests alongside the implementation.

---

# Week 2 Development Model

Frontend and backend are developed **incrementally together**.

We do not treat them as two isolated projects.

The normal feature-development cycle is:

```text
Approved feature requirement
        ↓
Backend implementation
        ↓
Frontend implementation
        ↓
API integration
        ↓
Test
        ↓
Review
        ↓
Next feature
```

This means frontend integration begins during Week 2 rather than being postponed until Day 15.

---

# Day 6 — Backend Foundation + Docker Foundation

Set up NestJS properly.

Implement:

```text
Configuration
Database
Environment variables
Logging
Error handling
Validation
Authentication foundation
```

Connect PostgreSQL.

Set up migrations based on the approved Day 4 database design.

Set up Redis/BullMQ foundation as required by the approved architecture.

Create the initial Docker development environment, including the required application/service containers.

Verify that the development environment can be started reproducibly.

---

# Day 7 — Authentication + Admin + Doctor

Implement:

```text
Admin authentication
Doctor authentication
Admin MFA
Password handling
Sessions/tokens
Protected routes
Account status
Role authorization
Admin account management
Doctor account management
```

Apply the authentication and security architecture defined during Week 1.

Frontend:

```text
Login
MFA flow where applicable
Protected dashboard
Logout
Role-aware navigation
Admin/Doctor access boundaries
```

Admin account creation and Doctor account creation must follow the approved requirements.

---

# Day 8 — Patients

Backend:

```text
Patient CRUD
Patient authorization
Patient ownership/scope rules
```

Frontend:

```text
Patient list
Add patient
Edit patient
Patient detail
Deactivate patient
```

Connect the existing `MeRim` UI to real APIs.

Ensure Doctor ownership rules are enforced according to the approved requirements and architecture.

---

# Day 9 — Medications + Schedules

Implement:

```text
Medication CRUD
MedicationSchedule
Schedule configuration
Schedule lifecycle
```

Doctor should be able to:

```text
Patient
  ↓
Add medication
  ↓
Set dosage
  ↓
Set schedule
  ↓
Configure permitted reminder/escalation behavior
```

Frontend should now use backend data instead of mock data.

Integrate and test the feature as a vertical slice.

---

# Day 10 — Reminder Engine

This is one of the most important days.

Implement:

```text
Medication Schedule
       ↓
Reminder generation
       ↓
Queue
       ↓
Notification processing
```

Set up:

```text
Redis
BullMQ
Scheduled jobs
```

Implement the reliability behavior defined during Week 1:

* Duplicate prevention
* Retry mechanism
* Failed job handling
* Persistent reminder records
* Idempotency
* Appropriate backoff behavior

The implementation should follow the approved reliability design rather than introducing a new design during coding.

---

# Day 11 — Notification Providers

Implement:

## SMS

```text
SmsProvider
     ↓
MockSmsProvider
```

## Voice

```text
VoiceProvider
     ↓
MockVoiceProvider
```

## Telegram

```text
TelegramProvider
     ↓
Telegram Bot
```

The first two do not need real production providers yet if the approved Phase 1 requirements use mocks.

The important requirement is that the application uses provider interfaces rather than coupling business logic to provider-specific SDKs.

---

# Day 12 — Telegram + Patient Response

Implement:

```text
Reminder
   ↓
Telegram
   ↓
Patient receives message
   ↓
Patient responds
   ↓
Backend receives response
   ↓
Adherence updated
```

This gives us the first real communication channel.

Implement and test:

* Secure Telegram linking
* Reminder delivery
* Taken response
* Not Taken response
* Invalid response handling
* Duplicate callback handling
* Response authorization
* Escalation stopping when appropriate

---

# Day 13 — Adherence + Notification Monitoring

Implement:

## Adherence

The implementation must follow the approved adherence model.

Maintain the distinction between:

```text
Notification delivery
```

and:

```text
Medication adherence
```

A successful notification does not automatically mean the medication was taken.

## Notifications

Track appropriate states such as:

```text
Scheduled
Sent
Delivered
Failed
Retrying
```

## Audit

Record important doctor/admin/system actions according to the approved Audit Log requirements.

Integrate the completed backend and frontend behavior as a vertical slice.

---

# Day 14 — Dashboard Integration + Week 2 Completion

Connect the dashboard to real backend data.

Replace:

```text
mock.ts
```

with:

```text
API
```

Doctor dashboard should show appropriate real:

* Patient counts
* Reminder counts
* Adherence information
* Attention-needed information
* Notification failures
* Escalation state

Admin dashboard/operational views should show the approved administrative information without unnecessary clinical-data exposure.

At the end of Day 14:

```text
Backend
   +
Frontend
   +
Database
   +
Queue
   +
Notifications
   +
Adherence
```

should form a working development system.

---

# End of Week 2 Target

We should have a working system like:

```text
Doctor
 ↓
Login
 ↓
Create Patient
 ↓
Add Medication
 ↓
Set Schedule
 ↓
Reminder generated
 ↓
Telegram/SMS/Voice mock
 ↓
Patient response
 ↓
Adherence recorded
 ↓
Doctor sees result
```

And:

```text
Admin
 ↓
Login + MFA
 ↓
Admin functions
 ↓
Operational/system visibility
```

This is our Week 2 milestone.

---

# Week 2 Claude Review Checkpoint

Before moving into final integration and hardening, Claude performs an implementation review.

This is not a redesign exercise.

The purpose is to identify implementation problems such as:

* Requirements not actually implemented
* Domain logic errors
* Database integrity problems
* Authorization gaps
* Admin/Doctor permission problems
* Queue/retry problems
* Duplicate processing
* Notification/adherence confusion
* Test gaps
* Incorrect provider abstraction
* Unexpected scope changes
* Docker/development-environment problems

Findings are categorized into:

```text
Must fix before proceeding
Should fix
Known limitation
```

Critical findings must be resolved before Week 3 release work continues.

---

# WEEK 3 — Integration, Testing & Production Readiness

## Goal

Turn the working development system into a **deployable Phase 1 product**.

This week is not primarily about adding new features.

It is about making what we built reliable, secure, tested, and deployable.

---

# Day 15 — Complete Frontend Integration

Finish connecting every required `MeRim` screen.

This is the **final frontend integration/completion pass**, not the first frontend/backend integration.

Remove development-only local state where appropriate.

Replace:

```text
mock data
```

with:

```text
API data
```

Check:

* Loading states
* Errors
* Empty states
* Pagination where required
* Form validation
* Success messages
* Role-based UI behavior
* Admin/Doctor navigation
* Responsive behavior where applicable

---

# Day 16 — Notification Reliability

Test:

```text
SMS mock
Telegram
Voice mock
```

Test failures:

```text
Provider unavailable
Network timeout
Invalid phone
Telegram unavailable
Queue failure
```

Verify:

* Retry
* Exponential backoff
* Duplicate prevention
* Idempotency
* Failed-message tracking
* Escalation behavior
* Delivery status handling

This day verifies the reliability design created during Week 1 and implemented during Week 2.

If a problem is discovered, fix the implementation rather than casually changing the requirements.

---

# Day 17 — Security

Perform a security pass.

Check:

* Authentication
* Admin MFA
* Authorization
* Password security
* Input validation
* Rate limiting
* CORS
* Secrets
* Environment variables
* API access
* Patient-data exposure
* Logging of sensitive information
* Webhook security
* Doctor-to-patient isolation
* Admin/Doctor permission boundaries

Ensure that a Doctor cannot access another Doctor's patients when restricted by the approved authorization model.

This is the **security verification and hardening stage**.

It does not replace the security architecture designed during Week 1.

---

# Day 18 — Automated Testing

Create and complete:

## Unit Tests

For appropriate core logic such as:

```text
Reminder generation
Schedule calculation
Adherence processing
Retry logic
Notification logic
Authorization logic
```

## Integration Tests

For:

```text
API
Database
Queue
Providers
Webhooks
```

## End-to-End Test

The most important test:

```text
Login
 ↓
Create patient
 ↓
Create medication
 ↓
Create schedule
 ↓
Generate reminder
 ↓
Send notification
 ↓
Patient responds
 ↓
Adherence recorded
 ↓
Dashboard updated
```

Also test the appropriate Admin workflow.

The goal is not merely to have tests.

The goal is to prove the critical Phase 1 workflows work end-to-end.

---

# Day 19 — Deployment

Deploy:

```text
Frontend → Vercel
Backend → Render
Database → Production PostgreSQL
Redis → Production Redis
```

Configure production environment variables securely.

Examples include:

```text
DATABASE_URL
REDIS_URL
JWT_SECRET
TELEGRAM_BOT_TOKEN
SMS_PROVIDER_*
VOICE_PROVIDER_*
```

Never commit these secrets to GitHub.

Build and verify the production backend container.

Confirm that the deployment can run from the approved Dockerized build configuration.

Real SMS and Voice providers can remain mocked if that is still consistent with the approved Phase 1 requirements.

---

# Day 20 — Staging / Production Simulation

Run realistic scenarios.

## Scenario 1

```text
Patient has 3 doses/day
```

Verify all reminders.

## Scenario 2

```text
Telegram succeeds
SMS fails
Voice mock succeeds
```

Verify independent delivery status and escalation behavior.

## Scenario 3

```text
Notification fails
```

Verify retry.

## Scenario 4

```text
Patient confirms medication
```

Verify adherence.

## Scenario 5

```text
Doctor changes medication schedule
```

Verify future reminders are updated correctly.

## Scenario 6

```text
Doctor stops medication
```

Verify future reminders stop.

## Scenario 7

```text
Admin changes an authorized system setting
```

Verify the setting, authorization, audit behavior, and future-effect rules work correctly.

## Docker Verification

Verify that the containerized application can:

* Build successfully
* Start successfully
* Connect to required services
* Use environment-based configuration
* Run the required tests or health checks
* Operate without development-only assumptions

---

# Day 21 — Final Review

This is the **release gate**.

Compare the running product against:

```text
Phase 1 Requirements Baseline v1.0
```

For every requirement:

```text
REQ-001 → ✅
REQ-002 → ✅
REQ-003 → ✅
...
```

Nothing should be marked complete just because the UI exists.

It needs to work end-to-end.

The final verification chain is:

```text
Requirements
      ↓
Design
      ↓
Implementation
      ↓
Tests
      ↓
Deployment
      ↓
Final verification
```

---

## Final Claude Review

Claude performs the final independent review of the release candidate.

The review focuses on:

* Remaining requirement gaps
* Security issues
* Reliability issues
* Data integrity
* Incorrect behavior
* Critical test gaps
* Deployment risks
* Docker/deployment issues
* Unintended scope changes
* Admin/Doctor authorization issues

Critical findings must be addressed before release.

Then the human performs the final release decision against the requirements baseline.

If all required items pass:

# Phase 1 Complete

---

# Three-Week Timeline at a Glance

| Week       | Main Objective                                                   | Result                                                   |
| ---------- | ---------------------------------------------------------------- | -------------------------------------------------------- |
| **Week 1** | Requirements + architecture + security/reliability/Docker design | Know exactly what we are building and how it should work |
| **Week 2** | Core implementation + incremental frontend/backend integration   | Working end-to-end development system                    |
| **Week 3** | Final integration + testing + hardening + deployment             | Deployable Phase 1 product                               |

---

# Critical Milestones

There are four milestones.

## Milestone 1 — End of Day 5

### Architecture Ready

```text
Requirements ✓
User flows ✓
Domain model ✓
Database design ✓
API ✓
Architecture ✓
Security design ✓
Reliability design ✓
Provider abstraction ✓
Docker strategy ✓
Admin/Doctor authorization model ✓
Claude review ✓
```

At this point, the architecture is approved and ready for implementation.

---

## Milestone 2 — End of Day 14

### Functional MVP Ready

```text
Doctor
 ↓
Patient
 ↓
Medication
 ↓
Schedule
 ↓
Reminder
 ↓
Notification
 ↓
Adherence
```

works end-to-end in development.

Admin functionality required for Phase 1 also works in development.

---

## Milestone 3 — End of Day 20

### Production Candidate

```text
Testing ✓
Security ✓
Reliability ✓
Docker verification ✓
Deployment ✓
Production simulation ✓
```

---

## Milestone 4 — Day 21

### Phase 1 Release

Requirements checklist completely satisfied.

Final review completed.

Release decision made against the approved requirements baseline.

---

# Claude Review Gates

Claude reviews are integrated into the existing three-week schedule.

They do **not** create additional days.

| Stage     | Claude Review                                        |
| --------- | ---------------------------------------------------- |
| Day 1     | Requirements review                                  |
| Day 3     | User-flow review                                     |
| Day 4     | Deep domain/database review                          |
| Day 5     | Deep architecture/security/reliability/Docker review |
| Day 13/14 | Implementation checkpoint                            |
| Day 21    | Final release review                                 |

The purpose of Claude is to act as an **independent challenger**, not as a second developer continuously changing the architecture.

---

# Security & Reliability Lifecycle

The project explicitly follows this lifecycle:

```text
WEEK 1
Design
   ↓
Security architecture
Reliability architecture
Docker/deployment architecture
   ↓
Claude review
   ↓
Freeze

WEEK 2
Implement
   ↓
Authentication
Authorization
Queues
Retries
Idempotency
Provider failure handling
Docker environment
Frontend/backend integration
   ↓
Tests

WEEK 3
Verify
   ↓
Security testing
Reliability testing
Failure simulation
Docker verification
   ↓
Hardening
   ↓
Release
```

This prevents the common problem of discovering security, reliability, integration, or deployment requirements only after the system has already been implemented.

---

# Frontend/Backend Development Lifecycle

The intended development model is:

```text
                    ┌──────────────┐
                    │  Requirement │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │    Backend   │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │   Frontend   │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │ Integration  │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │     Test     │
                    └──────┬───────┘
                           ↓
                    Next feature
```

Therefore:

* Backend and frontend are **not completely independent projects**.
* They are integrated continuously during Week 2.
* Backend foundation comes first because the frontend needs stable API contracts.
* Major features are developed as vertical slices.
* Day 15 is the final frontend integration/completion pass.

---

# Docker Lifecycle

Docker is part of the engineering lifecycle from design through release.

```text
Day 5
Docker architecture
      ↓
Day 6
Development containers
      ↓
Week 2
Application implementation/testing
      ↓
Week 3
Container build verification
      ↓
Day 19
Production deployment verification
      ↓
Day 20
Docker production simulation
      ↓
Day 21
Release verification
```

Docker configuration must remain aligned with the approved application architecture.

---

# Admin Lifecycle

Admin is integrated throughout the project.

```text
Day 1
Admin requirements
      ↓
Day 3
Admin workflows
      ↓
Day 4
Admin domain/authorization model
      ↓
Day 5
Admin security architecture
      ↓
Day 7
Admin implementation
      ↓
Day 14
Admin dashboard/operational integration
      ↓
Day 17
Admin security testing
      ↓
Day 18
Admin automated testing
      ↓
Day 20
Admin operational simulation
      ↓
Day 21
Admin acceptance verification
```

Admin is therefore not a late addition to the project.

It is part of the Phase 1 architecture and implementation from the beginning.

---

# Important Warning About the 3-Week Target

**Three weeks is achievable only if we keep the scope disciplined.**

During these three weeks, we explicitly do not add:

* AI diagnosis
* AI treatment recommendations
* Pharmacy management
* Hospital management
* Appointment management
* Billing
* Insurance
* Full patient web portal
* Mobile app
* Complex analytics
* Advanced reporting
* Other features not contained in the approved Phase 1 requirements

Unless one of those is explicitly included in the approved requirements, it remains outside Phase 1.

Real SMS and Voice provider integration does not need to block Phase 1 development if the approved requirements use mock providers.

Provider interfaces should remain production-ready so that real providers can be integrated later without changing the core business logic.

---

# Rule for Scope Changes

The Phase 1 requirements baseline is the controlling document.

Therefore:

```text
New feature requested
        ↓
Check Phase 1 requirements
        ↓
Already required?
   ┌────┴────┐
  YES        NO
   ↓          ↓
Implement   Requirements
            change proposed
                 ↓
          Review impact
                 ↓
          Human approval
                 ↓
          Requirements updated
                 ↓
          Design updated if needed
                 ↓
             Implement
```

We should not silently add features during implementation.

---

# Rule for the Three Weeks

> **Week 1: Decide what to build and design how it should work.**
>
> **Week 2: Build the approved system.**
>
> **Week 3: Prove that it works, harden it, and deploy it.**

The engineering process is therefore:

```text
Requirements
     ↓
Review
     ↓
Freeze
     ↓
Design
     ↓
Review
     ↓
Freeze
     ↓
Codex Implementation
     ↓
Continuous Testing
     ↓
Integration
     ↓
Security/Reliability Verification
     ↓
Deployment
     ↓
Final Requirements Verification
     ↓
Release
```

This document is the **master development plan** for Phase 1.

It does not replace the detailed Requirements Specification.

The Requirements Specification defines **what the product must do**.

This plan defines **when and how we will design, implement, test, and release it**.
