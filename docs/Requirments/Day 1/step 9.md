Absolutely. We will now work on **Step 9 only** and will not mix it with Notification, Telegram, SMS, Voice, or Adherence requirements.

# Day 1 — Step 9: Reminder Requirements

## 9.1 Purpose of Step 9

We need to define what a **Reminder** means in our system.

A reminder is not the SMS, Telegram message, or phone call itself.

Instead:

```text
Medication
     ↓
Medication Schedule
     ↓
Reminder Occurrence
     ↓
Notification
     ↓
Patient Response
     ↓
Adherence
```

For example:

> Abebe should take Amoxicillin 500 mg, 1 tablet at 08:00 on September 7.

That specific 08:00 event is a **Reminder Occurrence**.

Later, Step 10 will decide how that occurrence is communicated.

---

# 9.2 Reminder vs Schedule

This distinction is critical.

### Schedule

Defines the rule:

> Take Amoxicillin every day at 08:00.

### Reminder occurrence

Represents one actual instance:

> September 7, 08:00 — Amoxicillin is due.

So:

```text
Schedule
"Every day at 08:00"
       ↓
Generates
       ↓
Sep 7 08:00
Sep 8 08:00
Sep 9 08:00
Sep 10 08:00
...
```

Each of those is an independent reminder occurrence.

---

# 9.3 Reminder must have a unique identity

Every reminder occurrence needs a unique ID.

Example:

```text
Reminder ID: REM-20260907-000123
Patient: Abebe
Medication: Amoxicillin 500 mg
Dose: 1 tablet
Scheduled: Sep 7, 2026 08:00
```

This ID will later allow us to connect:

```text
Reminder
   │
   ├── Telegram notification
   ├── SMS notification
   ├── Voice notification
   │
   └── Patient response
```

This is also important for preventing duplicate processing.

---

# 9.4 When should a reminder be created?

The system should generate reminders from an active medication schedule.

For example:

```text
Schedule:
Every day
08:00
Sep 7 → Sep 14
```

The system generates:

```text
Sep 7 08:00
Sep 8 08:00
Sep 9 08:00
Sep 10 08:00
Sep 11 08:00
Sep 12 08:00
Sep 13 08:00
Sep 14 08:00
```

However, we should **not necessarily generate thousands of future reminders years into the future**.

For a production system, I recommend a controlled generation strategy.

We can generate upcoming occurrences within a defined planning window.

For example:

```text
Schedule
    ↓
Reminder Generator
    ↓
Upcoming reminder occurrences
```

The exact generation window will be a technical decision during database/job architecture.

The requirement should simply state that the system must reliably generate all applicable future occurrences.

---

# 9.5 No retroactive reminders

We already approved this in Step 8.

If today is:

> September 7 at 17:00

and a Doctor creates:

> Start date: September 1
> Time: 08:00

the system should **not** suddenly create:

```text
Sep 1 08:00
Sep 2 08:00
Sep 3 08:00
...
```

as new notifications.

Only future applicable occurrences should be generated.

This rule is now part of Step 9 as well.

---

# 9.6 Reminder lifecycle

A reminder needs a lifecycle.

I recommend the following conceptual states:

```text
SCHEDULED
    ↓
DUE
    ↓
PROCESSING
    ↓
NOTIFICATION_IN_PROGRESS
    ↓
COMPLETED / EXPIRED
```

But we need to be careful here.

**Notification status and adherence status are NOT reminder status.**

For example:

```text
Reminder: DUE

Telegram: DELIVERED

Patient: TAKEN
```

The reminder itself is one thing; the notification and adherence are separate things.

Therefore, I recommend keeping the Reminder status relatively simple.

---

# 9.7 Recommended reminder states

I recommend:

### `SCHEDULED`

The reminder occurrence exists but its scheduled time has not arrived.

Example:

```text
Current time: 07:00
Reminder: 08:00
Status: SCHEDULED
```

### `DUE`

The scheduled time has arrived and the reminder should be processed.

```text
08:00
↓
DUE
```

### `PROCESSING`

The system is currently processing the reminder.

This protects against concurrent workers processing the same occurrence incorrectly.

### `EXPIRED`

The reminder's response/processing window has ended without an effective patient response.

### `COMPLETED`

The reminder has reached its terminal business state.

For example, the patient responded:

> Taken.

or:

> Not Taken.

However, whether the reminder becomes `COMPLETED` immediately after a patient response will be finalized with the Adherence requirements in Step 14.

So we can define the lifecycle now while keeping adherence semantics separate.

---

# 9.8 Reminder expiration

A reminder should not remain active forever.

Example:

```text
08:00
Reminder becomes DUE
       ↓
Notification process
       ↓
Patient has a defined response window
       ↓
No response
       ↓
Reminder EXPIRES
```

After expiration:

```text
No new response should normally change the reminder
```

unless we explicitly define a correction/late-response policy.

We will define that carefully in Step 14.

---

# 9.9 Reminder timing accuracy

The system should process reminders as close as reasonably possible to their scheduled time.

For example:

```text
Scheduled:
08:00:00

Processing:
08:00:05
```

A small infrastructure delay may occur.

We should **not** require unrealistic "exact millisecond" execution.

Instead, the production requirement should be:

> The system shall process reminders within an acceptable configured tolerance around their scheduled time.

The exact tolerance can be determined during technical implementation/testing.

---

# 9.10 Timezone

Step 8 already established:

> Initial timezone: `Africa/Addis_Ababa`

Therefore:

```text
Patient schedule:
08:00 Africa/Addis_Ababa
```

must remain 08:00 for the patient even if the backend server is running in UTC.

The reminder system must use the configured timezone when calculating due times.

---

# 9.11 What if the server is temporarily down?

This is a very important production requirement.

Suppose:

```text
07:55
Server working

08:00
Server goes down

08:10
Server comes back
```

We cannot simply lose the 08:00 reminder.

The system must have **persistent reminder state**.

When the system recovers, it can determine:

> There was an applicable reminder scheduled for 08:00 that has not been processed.

Then it applies the defined recovery/expiration policy.

However, we should **not automatically send an outdated reminder** without considering the response window.

For example, a reminder that becomes due at 08:00 but the server only recovers at 14:00 should not blindly be sent at 14:00 as though it were still an 08:00 reminder.

This behavior will be designed in the technical architecture.

---

# 9.12 Reminder and schedule changes

Suppose the Doctor has:

```text
Schedule:
08:00 daily
```

and the system has:

```text
Sep 8 08:00 → Reminder
Sep 9 08:00 → Reminder
Sep 10 08:00 → Reminder
```

Doctor changes the schedule to:

```text
20:00 daily
```

Historical reminders must remain unchanged.

Future reminders that have not been delivered should follow the new schedule.

So:

```text
Past:
Sep 7 08:00
    ↓
UNCHANGED

Future:
Sep 8 08:00
    ↓
Recalculated

New:
Sep 8 20:00
```

This follows the rule we already approved in Step 8.

---

# 9.13 Reminder and medication status

The reminder engine must respect medication status.

### Medication ACTIVE

```text
Medication ACTIVE
      ↓
Schedule ACTIVE
      ↓
Reminders generated
```

### Medication PAUSED

```text
Medication PAUSED
      ↓
No new reminders
```

### Medication STOPPED

```text
Medication STOPPED
      ↓
No future reminders
```

Historical reminders remain.

---

# 9.14 Reminder and schedule status

Likewise:

```text
Schedule ACTIVE
    ↓
Generate applicable reminders
```

but:

```text
Schedule PAUSED
    ↓
No new reminders
```

and:

```text
Schedule ENDED
    ↓
No future reminders
```

---

# 9.15 Duplicate reminder prevention

This is a critical production requirement.

Imagine two background workers execute simultaneously:

```text
Worker A
   ↓
Generate Sep 7 08:00

Worker B
   ↓
Generate Sep 7 08:00
```

We must not end up with:

```text
❌ Reminder A — Sep 7 08:00
❌ Reminder B — Sep 7 08:00
```

Instead:

```text
✅ One reminder occurrence
```

The database and application architecture should enforce uniqueness.

---

# 9.16 Reminder must be persistent

I do **not** recommend keeping reminders only in application memory.

Bad:

```text
Server memory
   ↓
Reminder
```

If the server restarts:

```text
Reminder disappears ❌
```

Instead:

```text
Database
   ↓
Persistent Reminder
   ↓
Background worker
```

This is particularly important because you plan to deploy on services such as Render/Vercel rather than relying on a continuously running local machine.

---

# 9.17 Reminder processing should be asynchronous

We should not make the Doctor's web request wait for:

```text
Telegram
SMS
Voice
```

For example, when a Doctor creates a schedule:

```text
Doctor
  ↓
API
  ↓
Save schedule
  ↓
Return success
```

The reminder system can independently process future occurrences.

Later:

```text
Background job
   ↓
Reminder becomes due
   ↓
Notification system
```

This separation will be important for reliability.

---

# 9.18 What a reminder should contain

At minimum, each reminder occurrence needs enough information to identify:

* Patient
* Medication
* Dose
* Scheduled date/time
* Timezone
* Source schedule
* Current reminder state

It should also have timestamps such as:

* Created at
* Scheduled for
* Processing time
* Completed/expired time

The exact database structure will be designed later.

---

# 9.19 Reminder should preserve historical truth

Suppose the reminder was:

```text
Sep 7, 08:00
Amoxicillin 500 mg
1 tablet
```

and the Doctor changes the medication on Sep 8.

The Sep 7 reminder should still represent:

> Amoxicillin 500 mg, 1 tablet

It should not dynamically display the new medication information.

Therefore, at the appropriate point in the architecture, we need to preserve the relevant snapshot of what the patient was instructed to take for that occurrence.

This is an important production-data principle.

---

# 9.20 Doctor visibility

A Doctor should be able to see appropriate reminder information for their patients.

For example:

```text
Patient: Abebe

Medication: Amoxicillin 500 mg

Today's reminders:

08:00  ✓ Taken
14:00  ? Pending
20:00  Scheduled
```

The detailed adherence interpretation will be handled in Step 14.

---

# 9.21 Admin visibility

The Admin role should have appropriate system-level visibility according to the authorization model we already approved.

An Admin should be able to investigate reminder processing when necessary, particularly for operational troubleshooting.

But the exact Admin dashboard functionality will be defined as part of the Admin Portal requirements.

---

# 9.22 Reminder requirements

Here is the proposed formal requirement set.

| ID      | Requirement                                                                                                                                                |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| REM-001 | The system shall create a uniquely identifiable reminder occurrence for each applicable medication schedule occurrence.                                    |
| REM-002 | A reminder occurrence shall belong to one medication schedule.                                                                                             |
| REM-003 | A reminder occurrence shall be associated with the relevant patient and medication.                                                                        |
| REM-004 | Each reminder shall have a scheduled date/time.                                                                                                            |
| REM-005 | Reminder scheduling shall use the configured patient/system timezone.                                                                                      |
| REM-006 | Phase 1 shall use `Africa/Addis_Ababa` as the initial timezone.                                                                                            |
| REM-007 | The system shall generate reminder occurrences from active medication schedules.                                                                           |
| REM-008 | The system shall not generate retroactive reminders for occurrences whose scheduled time has already passed when a schedule is newly created or activated. |
| REM-009 | Paused medication schedules shall not generate new reminder occurrences.                                                                                   |
| REM-010 | Ended medication schedules shall not generate future reminder occurrences.                                                                                 |
| REM-011 | Stopped or paused medications shall not generate new applicable reminders.                                                                                 |
| REM-012 | Reminder occurrences shall be persisted so they survive application/server restarts.                                                                       |
| REM-013 | The system shall prevent unintended duplicate reminder occurrences.                                                                                        |
| REM-014 | Each reminder occurrence shall be independently trackable.                                                                                                 |
| REM-015 | Reminder processing shall occur asynchronously from normal Doctor portal requests where appropriate.                                                       |
| REM-016 | The system shall process due reminders within an acceptable configured time tolerance.                                                                     |
| REM-017 | Reminder processing shall distinguish scheduled, due, processing, completed, and expired states as applicable.                                             |
| REM-018 | Reminder expiration shall be based on a defined response/processing window.                                                                                |
| REM-019 | The system shall not treat notification delivery status as the reminder's adherence status.                                                                |
| REM-020 | Changes to a medication schedule shall not rewrite historical reminder occurrences.                                                                        |
| REM-021 | Future applicable reminder occurrences shall reflect valid schedule changes.                                                                               |
| REM-022 | Historical reminder occurrences shall preserve the relevant medication/dose information applicable at the time of the occurrence.                          |
| REM-023 | Reminder generation shall respect medication start and end dates.                                                                                          |
| REM-024 | Reminder generation shall respect schedule start and end dates.                                                                                            |
| REM-025 | The system shall support recovery from temporary processing/service interruptions without silently losing applicable reminder occurrences.                 |
| REM-026 | The system shall avoid sending an outdated reminder after a prolonged service interruption when its configured response window has already expired.        |
| REM-027 | Reminder records shall contain sufficient timestamps and identifiers to support operational investigation and auditing.                                    |
| REM-028 | Reminder access shall respect the established Doctor–patient authorization boundaries.                                                                     |

---

# 9.23 One architectural point I want to lock

There is an important distinction we should preserve throughout the project:

```text
              SCHEDULE
                 │
                 ▼
        REMINDER OCCURRENCE
                 │
                 ▼
          NOTIFICATION(S)
                 │
                 ▼
        PATIENT RESPONSE
                 │
                 ▼
             ADHERENCE
```

We should **never collapse these into one database object or one business concept**.

For example:

> "SMS was delivered"

doesn't mean:

> "Patient took the medicine."

And:

> "Patient didn't answer"

doesn't mean:

> "Patient didn't take the medicine."

This separation will make our production system much more reliable and clinically meaningful.

---

# Step 9 recommendation

I recommend that we **lock Step 9 with the requirements above**, including:

* ✅ Persistent reminder occurrences
* ✅ Unique reminder identity
* ✅ No retroactive reminders
* ✅ Timezone-aware scheduling
* ✅ Schedule/medication status respected
* ✅ Historical reminder preservation
* ✅ Future reminder recalculation after schedule changes
* ✅ Duplicate prevention
* ✅ Recovery from server interruptions
* ✅ Reminder expiration
* ✅ Asynchronous processing
* ✅ Clear separation between Reminder, Notification, and Adherence

And we **do not yet define**:

* Telegram behavior
* SMS behavior
* Voice behavior
* Provider selection
* Provider pricing
* Channel fallback timing
* Patient response interpretation

Those belong to **Steps 10–13**.

If you accept Step 9, we will lock it and move to **Step 10 — Notification Requirements**, where we will specifically resolve the question you raised earlier: **exactly when Telegram → SMS → Voice transitions occur, what triggers each transition, retry vs fallback, and how multiple channels are coordinated.**
