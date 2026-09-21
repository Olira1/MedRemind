Excellent. **Step 5 is approved and locked.** ✅

We have also decided that if a Doctor is deactivated, the system should support **reassigning that Doctor's patients to another active Doctor**, so active patients are not left without clinical ownership.

Now we move to:

# Day 1 — Step 6: Patient Requirements

### Goal

Define exactly what the system must support for a patient, including:

* Patient information
* Patient lifecycle
* Doctor assignment
* Communication channels
* Telegram connection
* Phone numbers
* Medication relationship
* Reminder relationship
* Patient responses
* Deactivation
* Reassignment
* Privacy/data boundaries

We are still defining **requirements**, not database fields or APIs yet.

---

# 6.1 Patient's role in Phase 1

The patient is a recipient and responder, not a web-portal user.

The Phase 1 model is:

```text
                    DOCTOR
                      │
                      │ manages
                      ▼
                   PATIENT
                      │
             ┌────────┼────────┐
             │        │        │
             ▼        ▼        ▼
          Telegram    SMS     Voice
             │        │        │
             └────────┼────────┘
                      ▼
                Patient response
                      │
                      ▼
                  Adherence
```

Therefore, a patient does **not** need:

* Web login
* Web dashboard
* Password
* Mobile app

in Phase 1.

---

# 6.2 Patient creation

A Doctor should be able to create a patient.

Basic workflow:

```text
Doctor
  ↓
Patients
  ↓
Add Patient
  ↓
Enter required information
  ↓
Validate
  ↓
Save
  ↓
Patient assigned to Doctor
```

The system should not allow an incomplete/invalid patient record to be created.

The exact validation rules will be defined later.

---

# 6.3 Patient information

We need to decide which information is actually required.

I recommend separating fields into:

### Identity information

```text
First name
Last name
Date of birth
```

### Contact information

```text
Phone number
```

### Optional contact information

Potentially:

```text
Email
```

### Communication information

```text
Preferred notification channels
Telegram connection status
```

### System information

```text
Status
Assigned Doctor
Created date
Updated date
```

However, we should **not automatically collect information simply because it might be useful someday**.

For a clinical product, collecting unnecessary patient data creates additional privacy and security responsibilities.

So our requirements should follow:

> **Collect only the patient information required for Phase 1 functionality.**

---

# 6.4 Phone number

A phone number is particularly important because SMS and Voice depend on it.

The system should:

* Validate the phone number.
* Store it in a consistent format.
* Associate it with the correct patient.
* Use it for authorized SMS/Voice notifications.

We will later decide the exact accepted format, including Ethiopian numbers.

---

# 6.5 Telegram connection

Because Telegram is one of our real Phase 1 notification channels, we need a way to associate a Telegram account with the correct patient.

The conceptual flow could be:

```text
Doctor creates patient
       ↓
Patient receives/gets Telegram connection instructions
       ↓
Patient opens Telegram bot
       ↓
Patient completes linking process
       ↓
Telegram identity associated with patient
       ↓
Telegram reminders become available
```

This is important because we must **not simply trust a Telegram username entered manually by the Doctor**.

We need a secure association between:

```text
Patient
    ↕
Telegram identity
```

The exact linking mechanism will be specified later under Telegram Requirements.

---

# 6.6 Patient communication channels

A patient can have one or more enabled notification channels.

For example:

```text
Patient: Abebe

Notification channels:

☑ Telegram
☑ SMS
☑ Voice
```

Or:

```text
Patient: Hana

Notification channels:

☑ SMS
☐ Telegram
☑ Voice
```

The Doctor should be able to configure which channels are available for the patient's reminders.

But we need to distinguish between:

### Channel configured

and

### Channel actually usable

For example:

```text
Telegram configured
        ↓
Telegram account linked?
   ├── YES → usable
   └── NO → unavailable
```

Similarly:

```text
SMS enabled
   ↓
Valid phone number?
   ├── YES → usable
   └── NO → unavailable
```

The system should not silently attempt to use a channel that has no valid destination.

---

# 6.7 Multiple channels

We already approved that a reminder may use multiple channels.

Therefore, patient-level configuration might allow:

```text
Telegram + SMS + Voice
```

But the exact notification strategy—such as:

```text
Telegram → SMS → Voice
```

will be defined later in Notification Requirements.

At this stage, we only establish that a patient may have multiple available channels.

---

# 6.8 Patient medication relationship

A patient may have multiple medications.

For example:

```text
Patient: Abebe
│
├── Medication A
│     └── Schedule
│
├── Medication B
│     └── Schedule
│
└── Medication C
      └── Schedule
```

A medication must belong to a patient.

A Doctor should only be able to manage medications for patients they are authorized to access.

---

# 6.9 Patient status

I recommend two basic patient states:

```text
ACTIVE
INACTIVE
```

### Active

The patient can have active medication schedules and receive reminders.

### Inactive

The patient is no longer actively managed through the system.

When a patient becomes inactive, we need to define what happens to future reminders.

My recommendation is:

> **Future reminders should no longer be sent to an inactive patient, but historical reminders, notification records, adherence records, and audit records should remain preserved.**

For example:

```text
Patient ACTIVE
     ↓
Medication schedule active
     ↓
Reminders generated
```

Then:

```text
Patient INACTIVE
     ↓
Future reminder generation stops
     ↓
Historical data remains
```

This is safer than deleting the patient or historical records.

---

# 6.10 Reactivation

I recommend allowing an inactive patient to be reactivated.

For example:

```text
INACTIVE
   ↓
Doctor/Admin reactivates
   ↓
ACTIVE
```

However, reactivation should **not automatically recreate old missed reminders**.

For example, if the patient was inactive from September 1–5 and reactivated September 6:

```text
September 1–5
❌ No reminders should be retroactively generated

September 6 onward
✅ New applicable reminders
```

This avoids creating a large number of meaningless historical reminder jobs.

---

# 6.11 Doctor assignment

Each active patient should have a responsible Doctor.

Example:

```text
Doctor A
   ├── Patient 1
   ├── Patient 2
   └── Patient 3

Doctor B
   ├── Patient 4
   └── Patient 5
```

Doctor A cannot access Patient 4.

The backend must enforce this.

---

# 6.12 Patient reassignment

We approved this in Step 5.

If Doctor A leaves/deactivates:

```text
Doctor A
   ↓
Deactivated
   ↓
Patients need reassignment
```

Admin can:

```text
Patient 1 → Doctor B
Patient 2 → Doctor B
Patient 3 → Doctor C
```

After reassignment:

```text
Doctor B
   ↓
Patient 1
```

Doctor A should no longer have access to Patient 1's active data after the reassignment, according to the authorization rules we will define.

Historical records remain preserved.

---

# 6.13 Patient response

Patients can respond to medication reminders.

### Telegram

```text
Reminder
   ↓
[ Taken ]
[ Not Taken ]
```

### SMS

```text
SMS reminder
   ↓
Patient replies
   ↓
System interprets response
```

### Voice

```text
Phone call
   ↓
Audio reminder
   ↓
Press 1 → Taken
Press 2 → Not Taken
```

The resulting response becomes an adherence event.

---

# 6.14 No response

A patient may simply not respond.

This should **not automatically be treated as "Not Taken"**.

For example:

```text
Reminder delivered
      ↓
No patient response
      ↓
NO RESPONSE
```

This is different from:

```text
Patient explicitly selected
      ↓
NOT TAKEN
```

This distinction is clinically and analytically important.

Later, we will define when a reminder becomes **Missed** because no response occurred.

---

# 6.15 Patient data privacy

Because this is a clinical system, we need strong data-minimization requirements.

A notification should contain only the information necessary for the reminder.

For example, we should avoid sending unnecessary sensitive medical information through a channel if the reminder can work without it.

Similarly:

* Logs should not unnecessarily contain patient-sensitive information.
* Audit records should be carefully designed.
* Doctors should only access authorized patients.
* Admin access should be controlled according to role/permissions.

We'll expand this under Security and Privacy Requirements.

---

# 6.16 Patient deletion

I recommend **not providing ordinary hard deletion of patients** in Phase 1.

Instead:

```text
ACTIVE
  ↓
INACTIVE
```

This preserves historical information needed for:

* adherence history
* reminder history
* notification history
* auditability

If later we need a formal data-retention/deletion process, that should be designed separately rather than adding a simple "Delete Patient" button.

---

# 6.17 Proposed Patient Requirements

Here is the initial requirements set:

| ID      | Requirement                                                                                                    |
| ------- | -------------------------------------------------------------------------------------------------------------- |
| PAT-001 | System shall allow an authorized Doctor to create a patient.                                                   |
| PAT-002 | System shall allow an authorized Doctor to view assigned patients.                                             |
| PAT-003 | System shall allow an authorized Doctor to update assigned patient information.                                |
| PAT-004 | System shall allow an authorized Doctor to deactivate an assigned patient.                                     |
| PAT-005 | System shall support patient reactivation.                                                                     |
| PAT-006 | Each active patient shall have an assigned responsible Doctor.                                                 |
| PAT-007 | Admin shall be able to reassign patients between authorized Doctors.                                           |
| PAT-008 | Doctor access shall be restricted to patients assigned to that Doctor.                                         |
| PAT-009 | Patient records shall support the contact information required for configured notification channels.           |
| PAT-010 | Patient phone numbers shall be validated before being used for SMS or Voice.                                   |
| PAT-011 | System shall support Telegram account association with a patient.                                              |
| PAT-012 | A patient may have multiple available notification channels.                                                   |
| PAT-013 | System shall determine whether a configured notification channel is actually usable.                           |
| PAT-014 | Active patients may receive applicable medication reminders.                                                   |
| PAT-015 | Future reminders shall not be sent after a patient becomes inactive.                                           |
| PAT-016 | Historical patient-related records shall be preserved when a patient is deactivated.                           |
| PAT-017 | Reactivating a patient shall not retroactively generate old reminders.                                         |
| PAT-018 | Patient responses shall be associated with the appropriate medication reminder.                                |
| PAT-019 | Explicit "Not Taken" responses shall be distinguishable from no response.                                      |
| PAT-020 | Patients shall not require web portal accounts in Phase 1.                                                     |
| PAT-021 | Ordinary patient management shall not physically delete historical clinical/reminder records.                  |
| PAT-022 | Patient information exposed through notifications and logs shall be limited according to privacy requirements. |

---

# One issue we should resolve before locking Step 6

There is one important question about **patient ownership**.

Suppose:

```text
Doctor A
   ↓
Patient X
   ↓
Medication
   ↓
Reminders
```

Admin transfers Patient X to Doctor B.

What should happen to the **existing medication schedules and reminders**?

### My recommendation

The patient's existing active medication plans **continue** after reassignment.

Only the responsible Doctor changes:

```text
Before:

Doctor A
   ↓
Patient X
   ↓
Medication/Schedules
   ↓
Reminders


After:

Doctor B
   ↓
Patient X
   ↓
Same active Medication/Schedules
   ↓
Reminders continue
```

The transfer itself should be recorded in the audit log.

This avoids accidentally interrupting a patient's medication reminders simply because their responsible Doctor changed.

The historical data also remains intact.

### Why I recommend this

Imagine Doctor A leaves the clinic on Monday and Admin assigns all patients to Doctor B.

It would be dangerous if:

> **Doctor reassignment automatically stopped medication reminders.**

The patient's treatment/reminder plan should not depend on the identity of the Doctor unless the Admin explicitly changes or stops that plan.

So I recommend:

> **Doctor reassignment changes ownership/access, not the patient's active medication schedule.**

---

## Step 6 decision

Please confirm whether you accept this recommendation:

**When a patient is reassigned from Doctor A to Doctor B, their active medication schedules and future reminders continue unchanged unless the newly responsible Doctor/Admin explicitly changes, pauses, or stops them.**

If you approve this, I'll lock **Step 6** and move to **Step 7 — Medication Requirements**.
