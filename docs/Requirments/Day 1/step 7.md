Absolutely. **Step 6 is locked.** ✅

Now we move to **Day 1 — Step 7: Medication Requirements**.

The purpose of this step is to define exactly what a Doctor can do with medications and what information the system needs in order to generate reliable reminders.

We will **not design the database yet**. First we define the business requirements; database/API design comes later.

# Step 7 — Medication Requirements

## 7.1 Medication's role in the system

The basic relationship is:

```text
Doctor
   ↓
Patient
   ↓
Medication
   ↓
Medication Schedule
   ↓
Reminder
   ↓
Notification
   ↓
Patient Response
   ↓
Adherence
```

A patient can have multiple medications.

A medication can have one or more schedules.

---

# 7.2 Creating a medication

An authorized Doctor should be able to add a medication to one of their patients.

Example:

```text
Patient: Abebe

Add Medication

Medication name: Amoxicillin
Strength: 500 mg
Form: Tablet
Instructions: Take after food
Start date: September 4
End date: September 11
```

After saving:

```text
Patient
  └── Amoxicillin 500 mg
```

The medication belongs to that patient.

A Doctor cannot add a medication to another Doctor's patient.

---

# 7.3 Medication information

I recommend that Phase 1 support the following information.

### Required

* Medication name
* Strength/dose
* Dosage unit
* Medication form
* Start date
* Medication status

### Optional

* End date
* Instructions/notes

For example:

| Field           | Example         |
| --------------- | --------------- |
| Medication name | Amoxicillin     |
| Strength        | 500             |
| Unit            | mg              |
| Form            | Tablet          |
| Instructions    | Take after food |
| Start date      | Sep 4, 2026     |
| End date        | Sep 11, 2026    |
| Status          | Active          |

---

# 7.4 Why separate strength and unit?

We should avoid storing:

> `500mg`

as one uncontrolled text field if we can structure it.

Instead:

```text
Strength = 500
Unit = mg
```

This gives us more reliable data.

Similarly, dosage instructions can be structured where appropriate.

---

# 7.5 Medication form

The system should support common medication forms.

For Phase 1, we can start with:

* Tablet
* Capsule
* Syrup/Liquid
* Injection
* Other

We should not unnecessarily create a huge medical-form list now.

If a medication doesn't fit the predefined choices, `Other` can be used.

---

# 7.6 Dose quantity

We need to distinguish:

### Medication strength

> Amoxicillin **500 mg**

from:

### Dose quantity

> Take **2 tablets**

So a reminder might say:

> Take 2 tablets of Amoxicillin 500 mg.

This distinction is important.

---

# 7.7 Medication instructions

The Doctor may provide additional instructions.

Examples:

> Take after food.

> Take with water.

> Do not take on an empty stomach.

These instructions can be included in the reminder where appropriate.

However, we should be careful about putting unnecessary clinical information into SMS/Telegram/voice messages.

The detailed notification content will be defined later.

---

# 7.8 Medication start and end

A medication can have a treatment period.

Example:

```text
Start: September 4
End: September 11
```

The system should generate applicable reminders only within the active treatment period.

Conceptually:

```text
Before Sep 4
❌ No medication reminders

Sep 4 – Sep 11
✅ Medication reminders

After Sep 11
❌ No new medication reminders
```

---

# 7.9 What if there is no end date?

Some medications may not have a predetermined end date.

For example:

> A medication is intended to continue indefinitely.

Therefore, I recommend allowing:

```text
End date = optional
```

If no end date exists:

```text
Start date
     ↓
Continue until explicitly stopped/paused
```

---

# 7.10 Medication status

I recommend these basic statuses:

```text
ACTIVE
PAUSED
STOPPED
```

### ACTIVE

Medication is currently part of the patient's treatment plan.

Applicable reminders can be generated.

### PAUSED

Medication is temporarily suspended.

No new reminders should be generated while paused.

Historical information remains.

### STOPPED

Medication has been discontinued.

No future reminders should be generated.

Historical information remains.

---

# 7.11 Pause vs Stop

This distinction is useful.

Example:

### Pause

A patient temporarily stops taking a medication because of a temporary clinical decision.

```text
ACTIVE
  ↓
PAUSED
  ↓
ACTIVE
```

The medication can later resume.

### Stop

The medication is permanently discontinued.

```text
ACTIVE
  ↓
STOPPED
```

It should not automatically resume.

---

# 7.12 Medication history

Just like patient history, we should preserve medication history.

For example:

```text
Amoxicillin
   ↓
Created
   ↓
Active
   ↓
Paused
   ↓
Active
   ↓
Stopped
```

We should be able to determine what happened and when.

We should not simply overwrite the record so that historical state changes disappear.

---

# 7.13 Editing medication

A Doctor should be able to modify permitted medication information.

For example:

```text
500 mg
   ↓
Doctor changes
   ↓
250 mg
```

But because medication changes can affect reminders, we need to be careful.

A change should not silently corrupt existing reminder/adherence history.

For example:

```text
Sep 4, 08:00
Amoxicillin 500 mg
Taken
```

Later the Doctor changes the medication to 250 mg.

We must **not rewrite the September 4 historical reminder** to say 250 mg.

The historical event should remain historically accurate.

This is another reason our future data model needs proper historical records.

---

# 7.14 Medication deletion

I recommend **no ordinary hard-delete operation** for medications.

Instead:

```text
ACTIVE
  ↓
STOPPED
```

Historical reminders and adherence records remain.

This is especially important for a production clinical system.

---

# 7.15 Medication schedules

The medication itself doesn't tell us **when** the patient takes it.

The schedule does.

For example:

```text
Medication:
Amoxicillin 500 mg

Schedule:
08:00
14:00
20:00
```

Or:

```text
Medication:
Paracetamol 500 mg

Schedule:
Every 8 hours
```

We will define the exact supported scheduling models in the next step.

For now:

> Medication and medication schedule are separate concepts.

---

# 7.16 Medication and reminder relationship

A medication can produce multiple reminder occurrences.

For example:

```text
Amoxicillin
    │
    ▼
Schedule
08:00 / 14:00 / 20:00
    │
    ▼
Reminder occurrences
    │
    ├── Sep 4 08:00
    ├── Sep 4 14:00
    ├── Sep 4 20:00
    ├── Sep 5 08:00
    └── ...
```

Each occurrence needs to be independently trackable.

That means if the patient takes the 08:00 dose but misses the 14:00 dose, the system must record those separately.

---

# 7.17 Multiple medications

A patient can have multiple active medications simultaneously.

Example:

```text
Abebe
│
├── Amoxicillin
│     └── 08:00 / 14:00 / 20:00
│
├── Paracetamol
│     └── 09:00 / 21:00
│
└── Vitamin D
      └── 08:00
```

The system must keep each medication and its schedules separate.

---

# 7.18 Medication conflict checking

I **do not recommend** adding automatic drug-interaction or clinical decision-support functionality to Phase 1.

For example, we should not attempt:

> "Amoxicillin and medication X may interact."

That would require a reliable clinical drug database and validated clinical logic.

It is outside our approved Phase 1 scope.

The system's job in Phase 1 is primarily:

> **Manage medication schedules and reminders, not make medical decisions.**

---

# 7.19 Proposed Medication Requirements

| ID      | Requirement                                                                                                |
| ------- | ---------------------------------------------------------------------------------------------------------- |
| MED-001 | System shall allow an authorized Doctor to create a medication for an assigned patient.                    |
| MED-002 | Medication shall belong to a specific patient.                                                             |
| MED-003 | System shall support medication name.                                                                      |
| MED-004 | System shall support medication strength and unit.                                                         |
| MED-005 | System shall support medication form.                                                                      |
| MED-006 | System shall support dose quantity.                                                                        |
| MED-007 | System shall support medication instructions/notes.                                                        |
| MED-008 | System shall support medication start date.                                                                |
| MED-009 | System shall support an optional medication end date.                                                      |
| MED-010 | System shall support `ACTIVE`, `PAUSED`, and `STOPPED` medication states.                                  |
| MED-011 | Paused medications shall not generate new medication reminders while paused.                               |
| MED-012 | Stopped medications shall not generate future medication reminders.                                        |
| MED-013 | A medication without an end date shall remain applicable until explicitly paused or stopped.               |
| MED-014 | System shall allow an authorized Doctor to update permitted medication information.                        |
| MED-015 | Medication changes shall not rewrite historical reminder, notification, or adherence records.              |
| MED-016 | Ordinary medication management shall not physically delete historical medication-related records.          |
| MED-017 | A patient may have multiple medications.                                                                   |
| MED-018 | A medication may have one or more schedules.                                                               |
| MED-019 | Each medication reminder occurrence shall be independently trackable.                                      |
| MED-020 | System shall not provide automatic drug-interaction or clinical decision-support functionality in Phase 1. |
| MED-021 | Medication access shall respect Doctor-to-patient authorization boundaries.                                |

---

# One important decision before we lock Step 7

There is one area where I don't want to make the wrong assumption:

## Should we support dose quantity?

For example:

> **Amoxicillin — 500 mg — Take 2 tablets**

versus simply:

> **Amoxicillin — 500 mg — Take at 08:00**

I strongly recommend **yes**.

The system should distinguish:

```text
Medication strength:
500 mg

Dose quantity:
2 tablets

Schedule:
08:00
```

Then the reminder can accurately communicate:

> "It is time to take 2 tablets of Amoxicillin 500 mg."

This is much more useful than only knowing that the medication is due.

---

## Another scheduling-related point

We should **not yet define "every X hours", "twice daily", "specific days", etc.** as medication requirements.

Those belong to **Step 8 — Medication Scheduling Requirements**, where we'll define exactly what scheduling patterns the Doctor can configure.

That step will be particularly important because the reminder engine depends on it.

### Step 7 proposed status

I recommend locking Step 7 with:

* ✅ Structured medication information
* ✅ Strength + unit
* ✅ Dose quantity
* ✅ Medication form
* ✅ Instructions
* ✅ Start/end dates
* ✅ Active/Paused/Stopped
* ✅ Historical preservation
* ✅ Multiple medications per patient
* ✅ Multiple schedules per medication
* ❌ Drug-interaction/clinical decision support

**If you approve this, Step 7 will be locked and we'll proceed to Step 8 — Medication Scheduling Requirements.**
