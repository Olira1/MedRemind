# STEP 16 — DASHBOARD REQUIREMENTS

## 16.1 Purpose

The system shall provide role-appropriate dashboards that allow authorized users to monitor patients, medication reminders, adherence activity, notification status, and important actions requiring attention.

**Cross-Document Authority:** Step 4B defines authentication and role-based authorization that dashboard access controls enforce. Step 14B defines adherence states that dashboard displays. Step 19 defines security controls including server-side authorization enforcement. This document defines dashboard requirements to provide operational visibility within appropriate role boundaries.

The dashboard serves as the main working interface for clinical reminder and adherence monitoring while maintaining strict authorization boundaries and role separation.

---

## 16.2 Role-Based Dashboard Design

### Doctor Dashboard Scope
Doctors shall see information limited to:
* Their authorized/assigned patients only
* Patient medication and reminder activity within their scope  
* Adherence information for their patients
* Notification problems requiring their attention
* Relevant operational activity within their authorization

### Admin Dashboard Scope  
Admins shall have broader operational visibility appropriate to their authorized administrative role:
* System-wide operational information
* Doctor and patient operational summaries
* Overall reminder and notification system health
* Administrative operational problems
* Important administrative activity

**Critical Boundary:** Admin dashboard shall minimize unnecessary clinical data exposure and shall not grant clinical modification authority merely because the user has administrative privileges.

---

# 16.2 Doctor Dashboard Components

### Summary Information
The Doctor dashboard shall provide summary cards showing key metrics such as:
- Total assigned patients
- Today's reminder activity
- Adherence status counts  
- Items requiring attention

### Reminder Status Overview
Today's reminder activity organized by status:
- Pending reminders
- Taken responses
- Not Taken responses  
- No Response (timed out)
- Notification failures

### Patients Needing Attention
Identification of patients requiring follow-up based on explicit rules such as:
- Recent NOT_TAKEN responses
- Multiple NO_RESPONSE occurrences
- Repeated notification delivery failures
- Unresolved reminder problems

### Recent Activity
Display of recent reminder and adherence events with appropriate navigation to detailed records.

---

# 16.3 Dashboard Operational Requirements

### Search and Navigation
- Patient search within authorized scope
- Search results must enforce Doctor patient isolation
- Navigation to patient, medication, and reminder details where authorized

### Filtering and Time Periods
- Support for current-day and historical views
- Filtering by adherence states and notification status
- Appropriate date range selection for operational needs

### Data States and Error Handling  
- Clear distinction between loading, empty, and error states
- No misleading zero values when data fails to load
- Appropriate error messages without exposing sensitive technical details

### Information Refresh
- Page-load data refresh
- User-initiated refresh capabilities
- Appropriate automatic updates for operational information where beneficial

---

# 16.3 Formal Dashboard Requirements

### DASH-001 — Role-Based Dashboard

The system shall provide dashboard views appropriate to the user's authorized role.

### DASH-002 — Doctor Scope

A Doctor dashboard shall display information only for patients and records the Doctor is authorized to access.

### DASH-003 — Admin Scope

An Admin dashboard shall display information within the Admin's authorized administrative scope and shall not provide clinical modification authority merely because the user has administrative privileges.

### DASH-004 — Patient Summary

The Doctor dashboard shall provide a summary of the Doctor's authorized patient population.

### DASH-005 — Reminder Summary

The dashboard shall provide a summary of relevant reminder activity for the selected period.

### DASH-006 — Adherence Summary

The dashboard shall provide adherence information using distinct states including TAKEN, NOT_TAKEN, and NO_RESPONSE.

### DASH-007 — Adherence Distinction

The dashboard shall not treat NO_RESPONSE as equivalent to NOT_TAKEN.

### DASH-008 — Notification Summary

The dashboard shall provide visibility into relevant notification activity and failures.

### DASH-009 — Notification-Adherence Separation

Notification delivery or failure status shall remain distinguishable from adherence status.

### DASH-010 — Attention Items

The Doctor dashboard shall identify patients or reminders requiring attention based on explicit system data and defined rules.

### DASH-011 — No Unexplained Risk Scores

The Phase 1 dashboard shall not present AI-generated or unexplained patient adherence-risk scores unless explicitly added to the requirements.

### DASH-012 — Recent Activity

The dashboard shall provide relevant recent reminder, adherence, and notification activity.

### DASH-013 — Patient Search

Authorized Doctors shall be able to search for patients within their authorized scope.

### DASH-014 — Authorization Enforcement

Patient search and dashboard queries shall enforce the same authorization boundaries as the underlying patient records.

### DASH-015 — Filters

The dashboard shall provide appropriate filtering for relevant time periods and reminder/adherence or notification states where required by the workflow.

### DASH-016 — Time Period

The dashboard shall support a current-day view of reminder and adherence activity.

### DASH-017 — Historical Filtering

The dashboard shall support appropriate historical filtering where required for reviewing reminder and adherence activity.

### DASH-018 — Reminder Status

The dashboard shall show the current relevant status of reminder occurrences.

### DASH-019 — Escalation Visibility

The dashboard shall provide sufficient information to understand the current notification/escalation state of a reminder when relevant.

### DASH-020 — Response Channel

Where a valid adherence response exists, the dashboard shall identify the channel through which the accepted response was received.

### DASH-021 — Notification History

Users shall be able to navigate from relevant dashboard information to the associated notification history where authorized.

### DASH-022 — Reminder Traceability

Users shall be able to navigate from relevant dashboard information to the associated reminder and patient record where authorized.

### DASH-023 — Patient Navigation

Authorized users shall be able to navigate from dashboard patient information to the corresponding patient record.

### DASH-024 — Medication Navigation

Where relevant, authorized users shall be able to navigate from reminder information to the associated medication and schedule information.

### DASH-025 — Refresh

Dashboard data shall support page-load refresh and appropriate user-initiated refresh.

### DASH-026 — Current Information

The dashboard shall provide reasonably current information for reminder and notification activity.

### DASH-027 — Automatic Updates

Where useful, the dashboard may automatically refresh relevant operational information without requiring a full application reload.

### DASH-028 — Data Loading Failure

The dashboard shall clearly indicate when required data cannot be loaded.

### DASH-029 — No False Zero

The dashboard shall not display misleading zero values when underlying data failed to load.

### DASH-030 — Empty States

The dashboard shall provide clear empty-state messages when no relevant records or activity exist.

### DASH-031 — Error States

Dashboard errors shall be communicated clearly without exposing sensitive technical information.

### DASH-032 — Doctor Actions

The Doctor dashboard shall provide appropriate navigation to patient, medication, reminder, adherence, and notification workflows.

### DASH-033 — Admin Actions

The Admin dashboard shall provide appropriate navigation to authorized administrative and operational workflows.

### DASH-034 — Role Isolation

Dashboard queries and displayed information shall enforce role-based access control.

### DASH-035 — Doctor Isolation

A Doctor shall not be able to access another Doctor's patients or unrelated dashboard data through dashboard queries, filters, identifiers, or navigation.

### DASH-036 — Privacy Minimization

The dashboard shall display only information necessary for the user's authorized workflow.

### DASH-037 — Operational Focus

The Phase 1 dashboard shall focus on operational patient reminder, adherence, and notification monitoring.

### DASH-038 — No Advanced Analytics

Advanced analytics, predictive adherence scoring, AI-generated clinical recommendations, financial analytics, and other excluded reporting functionality shall not be part of the Phase 1 dashboard unless explicitly approved.

### DASH-039 — Traceability

Dashboard information shall remain traceable to the underlying patient, medication, schedule, reminder, notification, and adherence records where applicable.

### DASH-040 — Consistent State

Dashboard status values shall use the same authoritative state definitions used by the underlying reminder, notification, and adherence systems.

### DASH-041 — No Clinical Inference

The dashboard shall not infer clinical conditions, diagnoses, or treatment recommendations from reminder or adherence data.

### DASH-042 — Performance

Dashboard queries shall be designed to provide acceptable response times for the expected Phase 1 data volume without compromising authorization or data accuracy.

---

# 16.4 Cross-Document Authority

To prevent conflicting requirements:

| Dashboard Area | Primary Requirement Section |
|----------------|----------------------------|
| Authentication and role verification | Step 4B |
| Dashboard role-based access | Step 16 |
| Adherence state definitions | Step 14B |  
| Dashboard adherence display | Step 16 |
| Security and authorization controls | Step 19 |
| Dashboard server-side enforcement | Step 16 |
| Audit of dashboard actions | Step 15 |

**Critical Rule:** Dashboard requirements shall enforce authorization boundaries established in other documents. Where dashboard displays information governed by other documents, those documents remain authoritative for the business logic while Step 16 defines display and access requirements.

---

# 16.5 Dashboard Acceptance Criteria

Step 16 shall be considered satisfied for Phase 1 when:

1. Role-specific dashboards exist for Doctor and Admin users.
2. Doctor dashboard displays information only for patients within the Doctor's authorized scope.
3. Admin dashboard displays information within authorized administrative scope without granting clinical modification authority.
4. Adherence states are displayed using the correct TAKEN, NOT_TAKEN, NO_RESPONSE, and PENDING states.
5. NO_RESPONSE is distinguished from NOT_TAKEN in dashboard displays.
6. Notification delivery failures are not displayed as patient non-adherence.
7. Patients or reminders needing attention can be identified using explicit, defined rules.
8. Loading states are distinguishable from zero/no-data states.
9. Error states do not produce misleading dashboard values or expose sensitive information.
10. Privacy and minimum-necessary information principles are preserved.
11. Dashboard does not include prohibited analytics, predictive scoring, or AI functionality.
12. Dashboard behavior is consistent with approved reminder, notification, and adherence models.
13. Dashboard access is enforced server-side with appropriate authorization boundaries.
14. Doctor cannot access another Doctor's patients through dashboard queries or navigation.
15. Significant dashboard-related administrative actions remain compatible with audit requirements.
16. Dashboard queries provide acceptable performance for expected Phase 1 data volumes.
17. Dashboard information remains traceable to underlying patient, medication, and reminder records.
18. Empty and error states provide clear, useful messages to users.
19. Dashboard refresh capabilities work without compromising authorization or performance.
20. Navigation from dashboard to detailed records respects the same authorization boundaries.

---

# 16.6 Relationship With Other Requirements

Step 16 shall support, not override, requirements established elsewhere.

Where dashboard displays adherence information, Step 14B remains the authoritative source for adherence states and business logic.

Where dashboard requires authentication and authorization, Step 4B and Step 19 remain authoritative for security controls.

Where dashboard actions may require auditing, Step 15 remains authoritative for audit requirements.

Where dashboard involves notification information, Steps 10B-13 remain authoritative for notification business logic.

Step 16 defines the dashboard interface requirements necessary to provide appropriate operational visibility while maintaining authorization boundaries and role separation across all system functions.
