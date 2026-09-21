Excellent. **Step 4 is approved and locked.** ✅

We now have the authentication foundation:

* `ADMIN` and `DOCTOR` are separate roles.
* Patients have no web account in Phase 1.
* Admin accounts are controlled by Admin-level processes.
* Admin can create/manage Doctor accounts.
* Doctors cannot create Admin accounts.
* Authentication and authorization will be enforced by the backend, not just the UI.

Now we move to:

# Day 1 — Step 5: Doctor/Admin Requirements

## 5.1 Purpose of the Doctor/Admin Portal

The portal is the primary interface through which healthcare staff manage the medication reminder system.

The portal should allow authorized users to:

```text
Manage users
      ↓
Manage patients
      ↓
Manage medications
      ↓
Configure schedules
      ↓
Configure reminders
      ↓
Monitor notifications
      ↓
Monitor adherence
      ↓
Review relevant history/audit information
```

But **Admin and Doctor should not have identical capabilities**.

---

# 5.2 Doctor capabilities

A Doctor should be able to access their own workspace.

### Doctor dashboard

The Doctor can see information related to their assigned patients, such as:

* Number of active patients
* Today's medication reminders
* Today's completed/taken doses
* Missed/not-taken doses
* Notification failures
* Adherence information

The exact dashboard metrics will be defined later.

---

## 5.3 Patient management

A Doctor can:

### Create patient

```text
Doctor
 ↓
Add Patient
 ↓
Enter required information
 ↓
Save
 ↓
Patient assigned to Doctor
```

### View patient

Doctor can view information for their assigned patients.

### Update patient

Doctor can update permitted patient information.

### Deactivate patient

A Doctor can deactivate a patient according to the rules we will define later.

Important:

> Deactivation should not destroy historical medication, reminder, notification, adherence, or audit information.

We should preserve history.

---

# 5.4 Medication management

For an assigned patient, the Doctor can:

* Add medication
* View medication
* Edit medication
* Pause medication
* Stop medication

For example:

```text
Patient: Abebe

Medications
├── Amoxicillin 500 mg
│      └── Active
│
├── Paracetamol 500 mg
│      └── Active
│
└── Vitamin D
       └── Paused
```

The exact medication fields will be specified later.

---

# 5.5 Medication schedule management

The Doctor should be able to configure when a medication needs to be taken.

For example:

```text
Amoxicillin 500 mg

08:00
14:00
20:00
```

The Doctor can:

* Create a schedule
* Edit a schedule
* Pause a schedule
* End a schedule

The system then uses the schedule to generate reminders.

---

# 5.6 Reminder configuration

The Doctor should be able to configure reminder behavior for a medication/schedule.

For example:

```text
Medication
    ↓
Schedule
    ↓
Reminder configuration
    ├── Telegram
    ├── SMS
    └── Voice
```

Multiple channels are allowed, as we approved in Step 3.

Later we will define whether the Doctor chooses:

```text
Telegram + SMS
```

or:

```text
Telegram + SMS + Voice
```

and how fallback/priority works.

---

# 5.7 Notification monitoring

The Doctor should be able to see notification information for their patients.

For example:

```text
Medication Reminder
08:00 AM

Telegram → Delivered
SMS      → Delivered
Voice    → Not attempted
```

Or:

```text
Telegram → Failed
SMS      → Delivered
```

This is important because:

> **Notification failure should not automatically be interpreted as medication non-adherence.**

The Doctor needs to distinguish the two.

---

# 5.8 Adherence monitoring

The Doctor should be able to see patient medication adherence.

For example:

```text
Patient: Abebe

Today

08:00 → Taken
14:00 → Not Taken
20:00 → No response
```

Later we will define exactly how these states are calculated and what time windows apply.

---

# 5.9 Reminder history

The Doctor should be able to inspect historical reminders.

For example:

```text
Date       Medication      Time     Status
------------------------------------------------
Sep 4      Amoxicillin     08:00    Taken
Sep 4      Amoxicillin     14:00    Not Taken
Sep 3      Amoxicillin     20:00    Taken
```

This historical information is important for clinical monitoring.

---

# 5.10 Patient search and filtering

The Doctor should be able to find their patients efficiently.

Potential capabilities:

* Search by patient name
* Search by phone number
* Filter by active/inactive
* Filter by adherence status

We will finalize the exact filters when defining Patient Requirements and Dashboard Requirements.

---

# 5.11 Doctor account management

A Doctor should be able to manage permitted parts of their own account.

For example:

* View profile
* Update permitted profile information
* Change password
* Log out

A Doctor **cannot**:

* Create an Admin
* Modify another Doctor's account
* Access another Doctor's patients
* Change system-level provider configuration
* Modify audit history

---

# 5.12 Admin capabilities

The Admin has broader system-level responsibilities.

### Doctor management

Admin can:

```text
Create Doctor
View Doctor
Update Doctor
Activate Doctor
Deactivate Doctor
```

For example:

```text
ADMIN
  ↓
Create Doctor
  ↓
Doctor receives account
  ↓
Doctor logs in
```

---

# 5.13 Admin patient access

Because Admin is a system-level role, the proposed model allows Admin to access patient information across the system **according to their administrative permissions**.

This does **not** mean Admin should automatically have unlimited clinical privileges.

We should separate:

> **System administration**

from:

> **Clinical decision-making**

An Admin should not automatically be able to alter clinical records simply because they have administrative privileges.

We'll define these boundaries carefully in the authorization requirements.

---

# 5.14 Admin system monitoring

Admin should be able to monitor system-level information such as:

* Notification failures
* Provider failures
* Queue/job failures
* System errors
* General system activity
* Audit events

This is particularly important once the system is running in production.

---

# 5.15 Admin settings

Admin may manage system-level configuration where explicitly authorized.

Examples could include:

```text
Notification configuration
Provider configuration
System defaults
```

However, **provider credentials and secrets should not be casually editable through the normal UI**.

For example:

```text
SMS_API_KEY
VOICE_API_KEY
TELEGRAM_BOT_TOKEN
DATABASE_URL
```

should normally be managed through secure deployment/environment configuration rather than displayed in the Admin portal.

---

# 5.16 Data isolation

This is a critical requirement.

### Doctor

```text
Doctor A
   ↓
Patient A1
Patient A2
Patient A3
```

Doctor A can access those patients.

But:

```text
Doctor A
   ❌
Patient B1
Patient B2
```

if those belong to Doctor B.

### Admin

```text
Admin
 ↓
System
 ├── Doctor A
 │     ├── Patient A1
 │     └── Patient A2
 │
 └── Doctor B
       ├── Patient B1
       └── Patient B2
```

The backend must enforce this isolation.

---

# 5.17 Deactivation rules

We need to distinguish between **deactivation** and **deletion**.

I recommend:

> We do not physically delete important clinical/reminder history through normal portal operations.

For example, if a patient is deactivated:

```text
Patient
   ↓
INACTIVE
```

Historical data remains available according to authorization.

Similarly:

```text
Medication
   ↓
STOPPED
```

doesn't mean historical reminders disappear.

This is important for auditability and data integrity.

---

# 5.18 Proposed Doctor/Admin requirements

Here is our initial requirements set:

| ID        | Requirement                                                                                                               |
| --------- | ------------------------------------------------------------------------------------------------------------------------- |
| ADMIN-001 | System shall provide a Doctor portal.                                                                                     |
| ADMIN-002 | System shall provide an Admin portal.                                                                                     |
| ADMIN-003 | Doctor shall access only patients assigned to that Doctor.                                                                |
| ADMIN-004 | Admin shall have system-level administrative access according to permissions.                                             |
| ADMIN-005 | Doctor shall be able to create patients.                                                                                  |
| ADMIN-006 | Doctor shall be able to view assigned patients.                                                                           |
| ADMIN-007 | Doctor shall be able to update assigned patients.                                                                         |
| ADMIN-008 | Doctor shall be able to deactivate assigned patients.                                                                     |
| ADMIN-009 | Doctor shall be able to manage medications for assigned patients.                                                         |
| ADMIN-010 | Doctor shall be able to manage medication schedules.                                                                      |
| ADMIN-011 | Doctor shall be able to configure reminder channels.                                                                      |
| ADMIN-012 | Doctor shall be able to view reminder history.                                                                            |
| ADMIN-013 | Doctor shall be able to view notification delivery information.                                                           |
| ADMIN-014 | Doctor shall be able to view adherence information for assigned patients.                                                 |
| ADMIN-015 | Doctor shall be able to manage permitted account information.                                                             |
| ADMIN-016 | Admin shall be able to create Doctor accounts.                                                                            |
| ADMIN-017 | Admin shall be able to activate/deactivate Doctor accounts.                                                               |
| ADMIN-018 | Admin shall be able to view system-level notification and operational information.                                        |
| ADMIN-019 | Admin shall be able to view relevant audit information.                                                                   |
| ADMIN-020 | Important historical clinical/reminder/adherence records shall not be destroyed through ordinary deactivation operations. |
| ADMIN-021 | Backend authorization shall enforce Doctor data isolation.                                                                |
| ADMIN-022 | Frontend visibility alone shall not be considered sufficient authorization.                                               |

These are still **draft requirements** until you approve them.

---

# One important issue before we lock Step 5

There is one architectural/business question I want to surface now rather than discovering it during database design.

### What happens when an Admin deactivates a Doctor?

Suppose:

```text
Doctor A
   ↓
Patient A
   ↓
Medication
   ↓
Daily reminders
```

Then Admin deactivates Doctor A.

Should the patient's medication reminders:

### Option 1 — Stop immediately

```text
Doctor deactivated
       ↓
Patient reminders stop
```

### Option 2 — Continue

```text
Doctor deactivated
       ↓
Existing patient schedules continue
       ↓
Reminders continue
```

### Option 3 — Transfer patients

```text
Doctor A deactivated
       ↓
Admin assigns patients
       ↓
Doctor B
       ↓
Reminders continue
```

For a real clinical product, **I recommend Option 3 as the long-term model**, with the ability to transfer patients to another active Doctor.

But I don't want to silently introduce a patient-transfer feature into our requirements if you don't want it in Phase 1.

So for now, please decide:

> **When an Admin deactivates a Doctor, should Phase 1 support transferring that Doctor's patients to another Doctor?**

My recommendation: **Yes, include basic patient reassignment/transfer in Phase 1**, because otherwise deactivating a Doctor can leave active patients without a responsible Doctor.

Once you decide this, we'll finalize and lock **Step 5**, then move to **Step 6 — Patient Requirements**.
