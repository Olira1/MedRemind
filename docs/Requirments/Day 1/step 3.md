Excellent. **Step 2 is approved and locked.**

We now have two separate roles:

* **ADMIN** — broader system-level permissions.
* **DOCTOR** — access only to patients assigned to that doctor.
* **PATIENT** — no web account in Phase 1; interacts through Telegram, SMS, and Voice.
* **SYSTEM** — performs automated reminder/notification operations.
* **External providers** — Telegram, SMS, and Voice services.

We can now move forward.

# Day 1 — Step 3: Core Business Workflows

### Goal

Here we define **exactly how the system is expected to work from beginning to end**.

This is one of the most important requirements steps because later:

* Database design will follow these workflows.
* APIs will support these workflows.
* Reminder jobs will implement these workflows.
* Notifications will implement these workflows.
* Adherence records will come from these workflows.
* Tests will verify these workflows.

We should therefore avoid technical implementation details for now. We are defining **business behavior**, not code.

---

## Workflow 1 — Doctor/Admin authentication

### Doctor

```text
Doctor
  ↓
Open web application
  ↓
Enter credentials
  ↓
Authentication
  ↓
Credentials valid?
  ├── No → Authentication failure
  └── Yes
       ↓
Doctor dashboard
```

After authentication, the Doctor should only see data they are authorized to access.

### Admin

```text
Admin
  ↓
Open web application
  ↓
Enter credentials
  ↓
Authentication
  ↓
Credentials valid?
  ├── No → Authentication failure
  └── Yes
       ↓
Admin area/dashboard
```

The Admin has broader permissions than a Doctor.

---

# Workflow 2 — Doctor creates a patient

```text
Doctor
  ↓
Patient management
  ↓
Create patient
  ↓
Enter required patient information
  ↓
Validate information
  ↓
Save patient
  ↓
Patient becomes assigned to that Doctor
```

After creation, the patient can be used when creating medication and reminder schedules.

The exact patient fields will be defined later in the **Patient Requirements** step.

---

# Workflow 3 — Doctor adds medication

```text
Doctor
  ↓
Select patient
  ↓
Add medication
  ↓
Enter medication information
  ↓
Save
  ↓
Medication associated with patient
```

A patient may have **multiple medications**.

A medication must belong to a specific patient.

---

# Workflow 4 — Doctor creates medication schedule

After adding a medication:

```text
Doctor
  ↓
Select medication
  ↓
Create schedule
  ↓
Define when medication should be taken
  ↓
Define reminder configuration
  ↓
Save schedule
```

The schedule becomes the source from which the system determines when reminders should occur.

A medication may have **one or more scheduled times**, depending on the requirements we establish later.

---

# Workflow 5 — System creates a reminder

When the scheduled time arrives:

```text
Medication Schedule
        ↓
Scheduled time reached
        ↓
Reminder generated
        ↓
Notification job created
        ↓
Configured notification channel(s)
```

The reminder should be associated with the correct:

* Patient
* Medication
* Schedule
* Scheduled occurrence

This association is important because later we need to know **exactly which medication reminder produced a particular notification and adherence response**.

---

# Workflow 6 — Send Telegram reminder

If Telegram is configured:

```text
Reminder
   ↓
Telegram notification job
   ↓
Telegram service
   ↓
Patient receives message
   ↓
Patient selects:
   ├── Taken
   └── Not Taken
```

The system records the notification delivery information and the patient's response separately.

---

# Workflow 7 — Send SMS reminder

If SMS is configured:

```text
Reminder
   ↓
SMS notification job
   ↓
SMS provider
   ↓
Patient receives SMS
   ↓
Patient replies
   ↓
System interprets response
   ↓
Taken / Not Taken / Invalid or unrecognized
```

As agreed earlier, we will define the exact SMS response rules later.

---

# Workflow 8 — Voice reminder

If Voice is configured:

```text
Reminder
   ↓
Voice notification job
   ↓
Voice provider
   ↓
Patient's phone rings
   ↓
Patient answers
   ↓
Automated audio reminder plays
   ↓
Patient presses keypad:
   ├── 1 → Taken
   └── 2 → Not Taken
```

The voice provider reports the call/result information back to our backend.

---

# Workflow 9 — Patient response → adherence

This is an important distinction.

The system receives a patient response:

```text
Taken
   ↓
Adherence Record
   ↓
Patient took medication
```

or:

```text
Not Taken
   ↓
Adherence Record
   ↓
Patient did not take medication
```

The **notification delivery record** and **adherence record** remain separate.

For example:

```text
SMS delivery: DELIVERED
Patient response: NOT TAKEN
```

This means:

> The reminder successfully reached the patient, but the patient reported not taking the medication.

That's very different from:

```text
SMS delivery: FAILED
Patient response: NONE
```

---

# Workflow 10 — Notification failure and retry

A production system must handle failures.

Example:

```text
Reminder
   ↓
Send notification
   ↓
Failed
   ↓
Retry
   ↓
Success?
   ├── Yes → Delivered/Completed
   └── No
        ↓
     Retry again
        ↓
     Retry limit reached
        ↓
     Mark as Failed
```

The system should preserve the history of attempts rather than simply replacing the previous failure.

This will allow the Doctor/Admin to understand what happened.

---

# Workflow 11 — Doctor monitors patient adherence

```text
Doctor
  ↓
Select patient
  ↓
View medication/adherence information
  ↓
See reminder history
  ↓
See Taken / Not Taken responses
  ↓
Evaluate patient's adherence
```

The Doctor should be able to distinguish between:

* Medication taken
* Medication not taken
* No response
* Notification failure
* Other states that we define later

---

# Workflow 12 — Doctor monitors notification delivery

The Doctor/Admin should be able to see whether notifications were successfully processed.

For example:

```text
Reminder
   ↓
Notification
   ↓
Pending
   ↓
Sending
   ↓
Sent
   ↓
Delivered
```

or:

```text
Reminder
   ↓
Notification
   ↓
Failed
   ↓
Retry
   ↓
Failed
```

The exact status model will be defined later in **Notification Requirements**.

---

# Workflow 13 — Audit workflow

Important system/user actions should generate audit records.

For example:

```text
Doctor
  ↓
Changes medication schedule
  ↓
System records audit event
```

The audit record should allow us to determine things such as:

* Who performed the action
* What action occurred
* Which relevant record was affected
* When it happened

We will define exactly which actions require auditing later.

---

# Workflow 14 — Overall end-to-end workflow

Putting everything together:

```text
                    ┌──────────────┐
                    │    DOCTOR    │
                    └──────┬───────┘
                           │
                           ▼
                    Create Patient
                           │
                           ▼
                    Add Medication
                           │
                           ▼
                    Create Schedule
                           │
                           ▼
                ┌─────────────────────┐
                │   REMINDER SYSTEM   │
                └──────────┬──────────┘
                           │
                     Reminder Due
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
         Telegram         SMS          Voice
             │             │             │
             ▼             ▼             ▼
         Patient        Patient       Patient
         responds       responds      presses key
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Patient Response
                           │
                           ▼
                    Adherence Record
                           │
                           ▼
                    Doctor Dashboard
```

Alongside this:

```text
Every important operation
          ↓
      Audit Log
```

and:

```text
Every notification attempt
          ↓
 Notification Delivery Record
```

---

# Critical business rule

There is one principle I want to establish now:

> **A reminder being delivered does not mean the medication was taken.**

The system must separately track:

**Reminder/Notification**

> Was the reminder successfully communicated?

and

**Adherence**

> What did the patient report about taking the medication?

This separation will be fundamental to our database and application design.

---

# Step 3 proposed locked workflows

So, the proposed Phase 1 core workflow set is:

1. Authentication
2. Patient creation/management
3. Medication management
4. Medication scheduling
5. Reminder generation
6. Telegram notification
7. SMS notification
8. Automated voice notification
9. Patient response
10. Adherence recording
11. Notification delivery tracking
12. Failure/retry handling
13. Doctor adherence monitoring
14. Notification monitoring
15. Audit logging

I recommend **locking these workflows as the Phase 1 core business workflows**.

Before I lock Step 3, there is one point I want your approval on:

**Should a single medication reminder be allowed to use multiple notification channels—for example, Telegram + SMS + Voice for the same scheduled medication occurrence—or should the Doctor choose only one channel per reminder?**

My recommendation for a production system is **multiple channels should be allowed**, because if a patient doesn't receive Telegram, SMS or Voice can provide another communication path. The exact rules for when/how multiple channels are triggered can be defined later.
