Great. **Step 7 is approved and locked.** ✅

We now move to **Day 1 — Step 8: Medication Scheduling Requirements**.

This is a particularly important step because the **schedule is what drives the reminder engine**. If we define scheduling poorly now, we could later have problems with recurring reminders, time zones, missed doses, duplicate notifications, and medication changes.

---

# Step 8 — Medication Scheduling Requirements

## 8.1 What is a medication schedule?

A medication is **what** the patient takes.

A schedule defines **when and how often** the patient should take it.

For example:

```text
Medication
Amoxicillin 500 mg
       ↓
Schedule
Take 1 tablet
at 08:00, 14:00, and 20:00
       ↓
Reminder occurrences
08:00
14:00
20:00
```

Each occurrence becomes independently trackable.

---

# 8.2 What should the Doctor configure?

When creating a schedule, I recommend that the Doctor can specify:

### Required

* Start date
* Time(s) medication should be taken
* Dose quantity
* Schedule pattern
* Reminder channels

### Optional

* End date
* Schedule instructions

The exact UI can be designed later.

---

# 8.3 Schedule patterns

I recommend supporting these patterns in Phase 1:

### A. Once daily

Example:

> Take 1 tablet every day at 08:00.

```text
08:00 every day
```

---

### B. Multiple times per day

Example:

> Take 1 tablet at 08:00, 14:00 and 20:00.

```text
08:00
14:00
20:00
```

This is probably one of the most common patterns for our system.

---

### C. Specific days of the week

Example:

> Take every Monday, Wednesday and Friday at 08:00.

```text
Monday    → 08:00
Wednesday → 08:00
Friday    → 08:00
```

This is useful for medications that aren't taken every day.

---

### D. Specific date range

Example:

> Take every day from September 4 to September 11 at 08:00.

```text
Sep 4 ───────── Sep 11
        ↓
      08:00
```

The schedule automatically stops after the end date.

---

### E. Every X hours

Example:

> Take one dose every 8 hours.

```text
08:00
16:00
00:00
08:00
...
```

However, this type needs careful handling because it is different from "three times per day."

For example:

**Every 8 hours** means intervals of approximately 8 hours.

**Three times daily** could mean fixed times such as:

> 08:00 / 14:00 / 20:00

These should not be treated as identical.

---

# 8.4 Do we really need "every X hours"?

I recommend **yes**, but with controlled input.

The Doctor could select:

```text
Schedule type:
[ Every X hours ]

Interval:
[ 8 ] hours
```

The system calculates the occurrences.

However, we should place reasonable validation around it so a Doctor cannot accidentally configure something like:

> Every 0.5 hours

or:

> Every 100 hours

The acceptable range will be defined in the detailed requirements.

---

# 8.5 Specific times vs intervals

This distinction should be explicit.

### Fixed-time schedule

```text
08:00
14:00
20:00
```

The reminder always targets those times.

### Interval schedule

```text
Starting at 08:00
Every 8 hours
```

The system calculates subsequent occurrences.

This distinction is important for the reminder engine.

---

# 8.6 Start date

Every schedule should have a start date.

Example:

```text
Start date: September 4, 2026
```

The system should not generate reminders before the schedule becomes active.

---

# 8.7 End date

An end date should be optional.

Example:

```text
Start: Sep 4
End: Sep 11
```

After the end date:

```text
No new reminder occurrences
```

If no end date exists:

```text
Continue until explicitly stopped/paused
```

---

# 8.8 Schedule status

I recommend:

```text
ACTIVE
PAUSED
ENDED
```

### ACTIVE

Reminder occurrences can be generated.

### PAUSED

No new reminders are generated.

The schedule can later return to ACTIVE.

```text
ACTIVE
  ↓
PAUSED
  ↓
ACTIVE
```

### ENDED

The schedule has finished.

It should not automatically resume.

---

# 8.9 Schedule pause

Suppose:

```text
Medication:
Amoxicillin

Schedule:
08:00 daily

Status:
ACTIVE
```

Doctor pauses it on September 6.

```text
Sep 4 → reminder
Sep 5 → reminder
Sep 6 → PAUSED
Sep 7 → no reminder
Sep 8 → no reminder
```

If the Doctor resumes it on September 10:

```text
Sep 10 onward → new reminders
```

We should **not create reminders retroactively** for September 7–9.

---

# 8.10 Schedule modification

This is one of the most important requirements.

Suppose:

```text
Old schedule:
08:00
```

Doctor changes it to:

```text
08:00
20:00
```

We should not rewrite historical occurrences.

Instead:

```text
Historical occurrences
        ↓
Remain unchanged

New schedule
        ↓
Applies to future occurrences
```

This preserves the truth of what the system previously instructed the patient to do.

---

# 8.11 Already-created future reminders

We need to think carefully about this.

Suppose the system has already generated tomorrow's reminder:

```text
Tomorrow 08:00
```

Then Doctor changes the schedule today.

We don't want the old future reminder to accidentally remain active if it is no longer valid.

Therefore, we need a rule:

> **When a schedule changes, future reminder occurrences that have not yet been delivered should be recalculated according to the new schedule. Historical/completed occurrences must remain unchanged.**

This is an important requirement for the reminder engine.

---

# 8.12 Time zone

Because this product is initially intended for Ethiopia, we should explicitly handle time zones.

The patient's reminder time should be interpreted according to the configured timezone.

For Phase 1, I recommend:

> **Africa/Addis_Ababa**

rather than relying on the server's timezone.

For example, if our backend is deployed on Render and the server uses UTC, we should **not** assume that:

```text
08:00 server time = 08:00 patient time
```

Instead:

```text
Patient schedule
08:00 Africa/Addis_Ababa
        ↓
Reminder engine
        ↓
Correct execution time
```

This becomes especially important when the application is deployed to different platforms.

---

# 8.13 Daylight-saving consideration

Ethiopia does not currently use seasonal daylight-saving clock changes, but we should still store the timezone explicitly rather than hardcoding assumptions.

That makes the system more portable if we later support patients in other countries.

---

# 8.14 Reminder channels at schedule level

We already decided that multiple channels can be used.

So a schedule could have:

```text
Reminder channels:

☑ Telegram
☑ SMS
☑ Voice
```

or:

```text
☑ Telegram
☑ SMS
☐ Voice
```

The exact fallback strategy will be defined in the Notification Requirements.

---

# 8.15 Multiple schedules for one medication

A medication can have multiple schedules if the clinical instructions require it.

For example:

```text
Medication:
Amoxicillin 500 mg

Schedule A:
08:00 daily

Schedule B:
14:00 daily

Schedule C:
20:00 daily
```

However, from a product-design perspective, I recommend we consider **one schedule with multiple times** for this simple case:

```text
08:00
14:00
20:00
```

rather than creating three separate schedules.

This makes the system easier for Doctors to manage and reduces unnecessary complexity.

Multiple schedules can still be supported when they have genuinely different patterns.

---

# 8.16 Example: complex schedule

For example:

> Take 1 tablet Monday, Wednesday and Friday at 08:00 for four weeks.

The system should represent:

```text
Start: Sep 4
End: Oct 2

Days:
Monday
Wednesday
Friday

Time:
08:00
```

Generated occurrences:

```text
Mon → 08:00
Wed → 08:00
Fri → 08:00
Mon → 08:00
...
```

---

# 8.17 Schedule validation

The system should validate schedules before saving them.

Examples of invalid configurations:

```text
❌ No start date
❌ No medication
❌ No time
❌ End date before start date
❌ Invalid interval
❌ No selected days for weekly schedule
```

The backend must perform validation even if the frontend already validates it.

---

# 8.18 Duplicate prevention

The schedule system must not accidentally create duplicate reminder occurrences.

For example:

```text
08:00
08:00
```

for the same medication occurrence should not result in two identical reminders unless the Doctor intentionally configured two distinct reminder events.

This will later connect to **idempotency and unique constraints in the database**.

---

# 8.19 Missed reminder vs schedule

One important distinction:

A schedule determines:

> **When the medication should be taken.**

It does not itself determine whether the patient took it.

For example:

```text
Schedule:
08:00

Reminder:
Delivered at 08:00

Patient:
No response
```

The schedule remains valid.

The patient response/adherence system determines what happened with that particular occurrence.

---

# 8.20 Proposed Scheduling Requirements

| ID      | Requirement                                                                                                     |
| ------- | --------------------------------------------------------------------------------------------------------------- |
| SCH-001 | System shall allow an authorized Doctor to create a medication schedule.                                        |
| SCH-002 | A schedule shall belong to a specific medication.                                                               |
| SCH-003 | A schedule shall have a start date.                                                                             |
| SCH-004 | A schedule may have an optional end date.                                                                       |
| SCH-005 | System shall support fixed daily medication times.                                                              |
| SCH-006 | System shall support multiple medication times within a day.                                                    |
| SCH-007 | System shall support selected days of the week.                                                                 |
| SCH-008 | System shall support date-range-based schedules.                                                                |
| SCH-009 | System shall support interval-based schedules such as every X hours.                                            |
| SCH-010 | System shall distinguish fixed-time schedules from interval-based schedules.                                    |
| SCH-011 | System shall support `ACTIVE`, `PAUSED`, and `ENDED` schedule states.                                           |
| SCH-012 | Paused schedules shall not generate new reminder occurrences.                                                   |
| SCH-013 | Ended schedules shall not generate future reminder occurrences.                                                 |
| SCH-014 | Resuming a paused schedule shall not retroactively generate missed reminder occurrences.                        |
| SCH-015 | Schedule changes shall not rewrite historical reminder occurrences.                                             |
| SCH-016 | Future undelivered reminder occurrences shall be recalculated when their underlying schedule changes.           |
| SCH-017 | System shall use an explicit timezone for schedule interpretation.                                              |
| SCH-018 | Phase 1 shall use `Africa/Addis_Ababa` as the initial patient/system timezone.                                  |
| SCH-019 | System shall validate schedule configurations before saving them.                                               |
| SCH-020 | System shall prevent unintended duplicate reminder occurrences.                                                 |
| SCH-021 | A medication may have multiple schedules where required.                                                        |
| SCH-022 | A schedule may support multiple notification channels.                                                          |
| SCH-023 | Schedule generation shall determine when reminders are due but shall not itself determine medication adherence. |
| SCH-024 | Doctor access to schedules shall respect patient ownership/authorization.                                       |

---

# One important production decision

There is one scheduling behavior I recommend we explicitly lock.

## What happens if a Doctor creates a schedule in the past?

For example, today is:

> September 4, 2026 at 5:00 PM

Doctor creates:

> Start date: September 1
> Time: 08:00

Should the system create missed reminders for:

```text
Sep 1 08:00
Sep 2 08:00
Sep 3 08:00
Sep 4 08:00
```

?

### My recommendation: No.

The system should **not retroactively generate reminders for times that have already passed**.

Instead:

```text
Schedule created:
Sep 4 at 17:00

First applicable future occurrence:
Sep 5 at 08:00
```

This prevents a newly created schedule from suddenly generating a large number of old notification jobs.

It also keeps the meaning of reminders clear:

> A reminder is an instruction for an upcoming medication occurrence, not a mechanism for reconstructing historical reminders.

Historical adherence can be entered separately if we later decide that feature is necessary.

---

## Step 8 decision

I recommend we lock the scheduling behavior with:

> **Schedules created or activated after their intended start date do not generate retroactive reminders for already-passed occurrences. Only future applicable occurrences are generated.**

If you accept this recommendation, **Step 8 will be locked**, and we'll proceed to **Step 9 — Reminder & Notification Requirements**, where we'll finally define in detail how Telegram, SMS, and Voice work, including retries, fallback, patient responses, delivery status, and provider abstraction.
