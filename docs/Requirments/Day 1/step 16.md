Great. **Step 15 — Audit Log Requirements is now locked.**

We now move to **Step 16 — Dashboard Requirements**. We will focus only on the dashboard in this step.

# Step 16 — Dashboard Requirements

The dashboard is the main working screen for Doctors and Admins after login.

The key principle I recommend is:

> **The dashboard should help a user quickly understand what needs attention and take the appropriate action.**

It should not become a general-purpose analytics/reporting system in Phase 1.

---

## 16.1 Dashboard users

There should be different dashboard views according to role.

### Doctor dashboard

A Doctor should see information limited to:

* Their authorized patients
* Their patients' medication/reminder activity
* Adherence information within their scope
* Notification problems requiring attention

### Admin dashboard

Admin should have a broader operational view according to their authorization.

For example:

* Total doctors
* Total patients
* Overall reminder activity
* Notification system health
* Failed notifications
* Operational problems
* Important recent activity

The Admin dashboard should **not expose unnecessary clinical information** merely because the user is an Admin.

---

# 16.2 Doctor dashboard — recommended layout

I recommend a relatively simple dashboard.

### A. Summary cards

At the top:

```text
Patients       Today's Reminders       Taken       Attention Needed
   125                 310               248              62
```

These should be calculated from real system data.

The exact metrics should be finalized carefully so that the numbers don't become misleading.

---

### B. Today's reminder status

A Doctor should quickly see today's reminder activity.

For example:

| Status              | Count |
| ------------------- | ----: |
| Pending             |    18 |
| Taken               |   248 |
| Not Taken           |    21 |
| No Response         |    23 |
| Failed Notification |     4 |

This gives the Doctor an immediate picture of today's activity.

---

### C. Patients needing attention

This is one of the most useful parts.

For example:

```text
Patients needing attention

Patient A    3 missed responses
Patient B    2 NOT_TAKEN responses
Patient C    Notification failures
Patient D    No response to today's reminders
```

The Doctor can select a patient and go to the patient's details.

We should **not** call this an AI-generated "risk score" in Phase 1.

It should be based on explicit, understandable rules.

---

### D. Recent reminder activity

Show recent reminder events:

```text
08:15  Patient A — Medication X — TAKEN
08:10  Patient B — Medication Y — NOT TAKEN
08:05  Patient C — Medication Z — NO RESPONSE
08:02  Patient D — Medication X — SMS failed
```

This should link to the relevant patient/reminder details.

---

# 16.3 Patient search

The Doctor dashboard should provide patient search.

Search could use appropriate patient identifiers such as:

* Patient name
* Patient ID
* Phone number, subject to privacy rules

The search results must respect doctor authorization.

A Doctor must never be able to search their way into another Doctor's patients.

---

# 16.4 Filters

Dashboard information should be filterable where useful.

For example:

### Time

* Today
* Yesterday
* Custom date range, if needed

### Reminder/adherence status

* Pending
* Taken
* Not Taken
* No Response

### Notification status

* Sent
* Delivered
* Failed
* etc.

### Patient

* Specific patient

We should avoid adding too many filters in Phase 1 unless they solve a real workflow problem.

---

# 16.5 Adherence presentation

The dashboard should distinguish:

**TAKEN**

**NOT TAKEN**

**NO RESPONSE**

These must not be combined into a generic "success/failure" metric.

For example:

```text
Taken       80%
Not Taken   10%
No Response 10%
```

This is much more meaningful than:

```text
Adherence: 80%
```

because "No Response" is not equivalent to "Not Taken."

---

# 16.6 Notification monitoring

The dashboard should make important notification failures visible.

For example:

```text
Notification issues

SMS failures       3
Voice failures     2
Telegram failures  1
```

Selecting the problem should allow the authorized user to investigate the associated reminder/patient.

Again:

> **Notification failure ≠ patient non-adherence.**

The dashboard must preserve this distinction.

---

# 16.7 Escalation visibility

Because we have Telegram → SMS → Voice escalation, the dashboard should provide enough information to understand where a reminder currently is.

For example:

```text
Patient A
08:00 Telegram → No response
08:10 SMS      → Waiting for response
```

Or:

```text
Patient B
08:00 Telegram → Taken
Escalation     → Stopped
```

This will be especially useful for troubleshooting.

---

# 16.8 Dashboard refresh

The dashboard should show reasonably current information.

For Phase 1, I recommend:

* Normal data refresh when the page loads
* User-triggered refresh
* Automatic refresh/polling where useful for active reminder/notification status

We do **not** necessarily need a complex real-time WebSocket architecture for the entire dashboard in Phase 1.

If later requirements show a strong need for real-time updates, that can be added deliberately.

---

# 16.9 Dashboard actions

The dashboard should be an entry point into actual workflows.

For example:

**Doctor**

* View patient
* Add patient
* View medication
* View reminder history
* Investigate notification failure

**Admin**

* View doctors
* View patients within scope
* View system/notification status
* Review important audit activity
* Manage appropriate settings

The dashboard itself should not contain complicated editing workflows.

---

# 16.10 Empty states

Production-quality dashboards must handle empty states.

Examples:

> "No patients have been assigned yet."

> "No reminders are scheduled for today."

> "No notification problems detected."

> "No adherence responses have been recorded."

These should be clear and useful rather than showing blank tables.

---

# 16.11 Error states

If dashboard data cannot be loaded, the system should clearly tell the user.

For example:

> "We couldn't load today's reminder data. Please try again."

It should not display a misleading `0` just because an API request failed.

This is particularly important for clinical information.

---

# 16.12 Privacy

The dashboard should show only the minimum information necessary for the user's role.

For example, a Doctor should not see:

* Other doctors' unrelated patients
* Unrelated audit records
* Provider credentials
* Internal security secrets
* Unnecessary technical information

Admin access must also follow the defined authorization policy.

---

# 16.13 Phase 1 dashboard boundary

I recommend **not** including these in Phase 1 unless we explicitly add them:

* Advanced analytics
* Predictive adherence scores
* AI-generated clinical recommendations
* Complex charts/reports
* Financial dashboards
* Billing analytics
* Population-health analytics
* Prescription analytics
* Appointment analytics

The Phase 1 dashboard should primarily be an **operational clinical reminder/adherence dashboard**.

---

# Proposed formal Step 16 requirements

## 16. Dashboard Requirements

### Purpose

The system shall provide role-appropriate dashboards that allow authorized users to monitor patients, medication reminders, adherence activity, notification status, and important actions requiring attention.

### DASH-001 — Role-Based Dashboard

The system shall provide dashboard views appropriate to the user's authorized role.

### DASH-002 — Doctor Scope

A Doctor dashboard shall display information only for patients and records the Doctor is authorized to access.

### DASH-003 — Admin Scope

An Admin dashboard shall display information within the Admin's authorized administrative scope.

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

## One decision before we lock Step 16

There is one point I recommend we settle now:

### Should the Doctor dashboard show **"Patients needing attention"**?

My recommendation is **yes**, but it should be based on transparent rules rather than an AI/risk score.

For example:

* Recent `NOT_TAKEN`
* Multiple `NO_RESPONSE`
* Repeated notification failures
* A currently unresolved reminder problem

The Doctor can click the patient and investigate the underlying records.

This would make the dashboard much more useful clinically while staying within Phase 1 scope.

**If you accept Step 16 with this recommendation, I'll lock it and move to Step 17 — Settings Requirements.**
