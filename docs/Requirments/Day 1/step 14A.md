Absolutely. **Step 13 is now locked.** ✅

And yes, your sequence is correct. We will **not skip or combine** the remaining Day 1 steps.

Our remaining Day 1 sequence is:

1. ~~Step 1 — Phase 1 Scope~~ ✅
2. ~~Step 2 — Actors~~ ✅
3. ~~Step 3 — Core Business Workflow~~ ✅
4. ~~Step 4 — Authentication~~ ✅
5. ~~Step 5 — Doctor/Admin~~ ✅
6. ~~Step 6 — Patient~~ ✅
7. ~~Step 7 — Medication~~ ✅
8. ~~Step 8 — Medication Schedule~~ ✅
9. ~~Step 9 — Reminder~~ ✅
10. ~~Step 10 — Notification~~ ✅
11. ~~Step 11 — Telegram~~ ✅
12. ~~Step 12 — SMS~~ ✅
13. ~~Step 13 — Voice~~ ✅
14. **Step 14 — Adherence** ← **NOW**
15. Step 15 — Audit Log
16. Step 16 — Dashboard
17. Step 17 — Settings
18. Step 18 — Non-functional Requirements
19. Step 19 — Phase 1 Exclusions
20. Step 20 — Requirement Acceptance Criteria
21. Step 21 — Requirements Review

After Step 21, we will have our **complete and reviewed Phase 1 Requirements Specification**. Only then should we move into the next phase of technical design/implementation.

---

# Day 1 — Step 14: Define Adherence Requirements

This is one of the most important parts of the entire system.

The notification system answers:

> **"Did we communicate the reminder?"**

The adherence system answers:

> **"What did the patient report about taking the medication?"**

These must remain separate.

---

## 14.1 What is an adherence record?

An adherence record represents the patient's response to a particular medication reminder occurrence.

For example:

```text
Patient: Abebe
Medication: Amoxicillin
Scheduled time: 08:00

Patient response:
TAKEN
```

The system should be able to trace that response back to:

```text
Patient
   ↓
Medication
   ↓
Schedule
   ↓
Reminder Occurrence
   ↓
Notification
   ↓
Patient Response
   ↓
Adherence Record
```

This traceability is important for clinical monitoring and auditing.

---

# 14.2 Adherence statuses

Based on everything we have agreed so far, I recommend these primary patient-response outcomes:

```text
TAKEN
NOT_TAKEN
NO_RESPONSE
```

### TAKEN

The patient explicitly indicated that they took the medication.

Examples:

* Telegram → **Taken**
* SMS → `1`
* Voice → keypad `1`

### NOT_TAKEN

The patient explicitly indicated that they did not take it.

Examples:

* Telegram → **Not Taken**
* SMS → `2`
* Voice → keypad `2`

### NO_RESPONSE

The patient did not provide a valid response within the defined adherence response window.

This is **not the same as NOT_TAKEN**.

That distinction is extremely important.

---

# 14.3 No response ≠ Not Taken

Consider:

```text
08:00 → Reminder
08:10 → SMS
08:20 → Voice
```

Patient never responds.

We should record:

```text
NO_RESPONSE
```

We should **not** automatically record:

```text
NOT_TAKEN
```

because we do not actually know whether the patient took the medicine.

So:

```text
NO_RESPONSE ≠ NOT_TAKEN
```

This prevents the system from making an unsupported clinical assumption.

---

# 14.4 Adherence is based on explicit patient response

The system should only classify:

```text
TAKEN
```

when there is an explicit valid patient response.

For example:

```text
Telegram → Taken
SMS → 1
Voice → 1
```

Similarly:

```text
NOT_TAKEN
```

requires an explicit valid response.

---

# 14.5 Notification delivery is not adherence

This distinction must be enforced throughout the system.

### Example 1

```text
SMS → DELIVERED
Patient → no response
```

Result:

```text
Notification = DELIVERED
Adherence = NO_RESPONSE
```

### Example 2

```text
Voice → ANSWERED
Patient → hangs up
```

Result:

```text
Call = ANSWERED
Adherence = NO_RESPONSE
```

### Example 3

```text
Telegram → delivered
Patient → presses Taken
```

Result:

```text
Notification = DELIVERED
Adherence = TAKEN
```

---

# 14.6 One adherence decision per reminder occurrence

A medication may be scheduled every day.

For example:

```text
08:00 Amoxicillin
2026-09-14
2026-09-15
2026-09-16
...
```

Each occurrence should have its own adherence record.

So:

```text
Sep 14 → TAKEN
Sep 15 → NOT_TAKEN
Sep 16 → NO_RESPONSE
```

These must remain separate records.

We should never have one generic:

```text
Amoxicillin adherence = TAKEN
```

because adherence changes from occurrence to occurrence.

---

# 14.7 Why the reminder occurrence is important

We need to distinguish:

> Medication schedule

from:

> Individual medication-taking event.

For example:

```text
Schedule:
Amoxicillin
Every day at 08:00
```

creates:

```text
Reminder #1001 → Sep 14, 08:00
Reminder #1002 → Sep 15, 08:00
Reminder #1003 → Sep 16, 08:00
```

Each reminder can produce its own adherence outcome.

---

# 14.8 Patient responds through multiple channels

This is where our earlier multiple-channel design becomes important.

Example:

```text
08:00 → Telegram
08:10 → SMS
08:20 → Voice
```

Suppose:

```text
08:05 → Patient presses Telegram "Taken"
```

Then:

```text
Adherence = TAKEN
```

and the system should stop further escalation.

Therefore:

```text
Telegram → TAKEN
       ↓
Stop SMS
       ↓
Stop Voice
```

---

# 14.9 What if SMS was already sent?

Suppose:

```text
08:00 → Telegram
08:10 → SMS
08:12 → Patient presses Telegram "Taken"
```

The SMS has already happened.

We cannot undo it.

So the system records:

```text
Telegram = DELIVERED
SMS = SENT/DELIVERED
Adherence = TAKEN
```

and prevents **future** escalation.

This is why notification history and adherence history must be separate.

---

# 14.10 What if two channels receive responses?

Example:

```text
08:05 → Telegram → Taken

08:06 → SMS → 1
```

Both indicate the same adherence decision.

The system must not create two separate medication-taking events.

Instead, they should be associated with the same reminder occurrence.

For example:

```text
Reminder #1001
     │
     ├── Telegram response → TAKEN
     │
     └── SMS response → duplicate/secondary confirmation
```

The exact event-history behavior can preserve both incoming responses, but the reminder should have **one final adherence state**.

---

# 14.11 What if the responses conflict?

This is an important edge case.

Example:

```text
08:05 → Telegram → TAKEN

08:07 → SMS → NOT_TAKEN
```

We must not silently overwrite one with the other.

This is a contradictory patient response.

My recommendation:

> Preserve both response events, flag the reminder as having conflicting responses, and apply a defined final-state policy rather than silently choosing one.

For example:

```text
Reminder
   │
   ├── Telegram → TAKEN
   │
   └── SMS → NOT_TAKEN
            ↓
      CONFLICTING_RESPONSE
```

The doctor can see that the patient gave inconsistent responses.

We should **not invent clinical information** to resolve the conflict.

---

# 14.12 First response vs final response

Earlier we discussed:

```text
08:03 → Taken
08:05 → Not Taken
```

I recommend that we distinguish:

### Response events

Every valid response can be preserved as an event.

```text
08:03 → TAKEN
08:05 → NOT_TAKEN
```

### Reminder's adherence state

The system needs a controlled final state.

Rather than silently overwriting the first response, we can represent:

```text
Response history:
TAKEN → NOT_TAKEN

Current state:
CONFLICTING
```

This gives us a much better audit trail.

---

# 14.13 Recommended adherence state model

I recommend:

```text
PENDING
TAKEN
NOT_TAKEN
NO_RESPONSE
CONFLICTING
```

### PENDING

The response window is still open.

Example:

```text
08:00 reminder
08:05
```

No response yet:

```text
PENDING
```

### TAKEN

Valid explicit Taken response.

### NOT_TAKEN

Valid explicit Not Taken response.

### NO_RESPONSE

Response window expired without a valid response.

### CONFLICTING

Multiple valid responses contradict each other and have not been resolved according to the defined policy.

This is safer than forcing every situation into only three values.

---

# 14.14 Should PENDING appear in doctor's adherence history?

I recommend:

* During an active reminder: `PENDING`
* After the response window closes: `NO_RESPONSE`

For example:

```text
08:00 → PENDING
08:15 → PENDING
09:00 → NO_RESPONSE
```

This makes the dashboard useful in real time.

---

# 14.15 Adherence response window

We previously agreed that notification escalation timing and adherence response timing are different.

For example:

```text
Medication due: 08:00

08:00 → Telegram
08:10 → SMS
08:20 → Voice

Adherence response window:
08:00 → 09:00
```

The patient can respond during the defined window.

After the window closes:

```text
No valid response
       ↓
NO_RESPONSE
```

The actual duration should be configurable according to our final requirements.

---

# 14.16 Late responses

Suppose:

```text
08:00 → Medication due
09:00 → Response window closes
09:30 → Patient presses Taken
```

The system should not simply treat it as a normal on-time adherence response.

Instead:

```text
Late response
      ↓
Record response event
      ↓
Mark as late
      ↓
Apply late-response policy
```

This preserves the fact that the patient eventually responded without pretending the response occurred on time.

---

# 14.17 Should a late "Taken" become TAKEN?

My recommendation:

> **Yes, record the patient's explicit statement that they took it, but preserve that it was a late response.**

For example:

```text
Adherence:
TAKEN

Response:
TAKEN
Response time:
09:30

Expected:
08:00

Late:
YES
```

This is much more informative than simply throwing away the response.

However, the dashboard can distinguish:

```text
Taken on time
Taken late
Not taken
No response
```

without incorrectly treating late medication-taking as on-time adherence.

---

# 14.18 Adherence timestamps

Each adherence event should have relevant timestamps.

At minimum:

```text
scheduled_at
response_at
```

Potentially also:

```text
recorded_at
```

For example:

```text
Scheduled:
08:00

Patient responded:
08:07

Backend recorded:
08:07
```

These timestamps help with monitoring and auditing.

---

# 14.19 Source channel

The adherence response should record where it came from.

For example:

```text
response_channel = TELEGRAM
```

or:

```text
response_channel = SMS
```

or:

```text
response_channel = VOICE
```

This allows the doctor/system to understand how the patient responded.

---

# 14.20 Do not confuse response channel with notification channel

For example:

```text
Telegram → reminder sent
SMS → reminder sent
Voice → reminder sent
```

Patient might then respond through:

```text
SMS
```

The adherence record should say:

```text
response_channel = SMS
```

while notification history separately records all notification attempts.

---

# 14.21 Doctor visibility

Doctors should be able to see adherence information for **patients they are authorized to access**.

For example:

```text
Patient
   ↓
Medication
   ↓
Schedule
   ↓
Adherence history
```

A doctor should be able to see something like:

| Date   | Medication  | Scheduled | Result      |
| ------ | ----------- | --------: | ----------- |
| Sep 14 | Amoxicillin |     08:00 | Taken       |
| Sep 14 | Amoxicillin |     20:00 | Not Taken   |
| Sep 15 | Amoxicillin |     08:00 | No Response |

The exact dashboard design will be defined in Step 16.

---

# 14.22 Historical adherence

Adherence records should not disappear simply because:

* the doctor changes,
* the medication is discontinued,
* the schedule changes,
* the patient is reassigned.

This connects directly to your earlier question about transferring a patient from one doctor to another.

The adherence history belongs to the **patient and their medication/reminder history**, not to the current doctor's personal account.

Authorization determines who can see it.

Conceptually:

```text
Patient
   │
   ├── Historical medications
   ├── Historical schedules
   ├── Historical reminders
   └── Historical adherence
```

If Doctor A transfers the patient to Doctor B:

```text
Doctor A
   ↓
Patient
   ↓
History remains
   ↓
Doctor B gains authorized access
```

We do not copy the history into a new patient.

---

# 14.23 Deleted medication

If a doctor stops a medication, historical adherence records should remain.

For example:

```text
Amoxicillin
Jan 1 → Jan 10
```

If discontinued on Jan 10:

```text
Medication = discontinued
```

Historical records remain available.

We should avoid destructive deletion of clinical history.

---

# 14.24 Deleted schedule

Similarly, if a schedule is changed:

```text
08:00 → 1 tablet
```

becomes:

```text
08:00 → 2 tablets
```

the old reminder/adherence history should remain associated with the historical schedule occurrence.

The new schedule applies going forward.

---

# 14.25 Adherence calculation

For the dashboard, we may eventually calculate something like:

```text
Adherence rate = Taken / eligible reminder occurrences
```

But we must be careful about:

* `PENDING`
* `NO_RESPONSE`
* `NOT_TAKEN`
* conflicting responses
* late responses

Therefore, I recommend that the underlying **raw adherence events remain authoritative**, while calculated percentages are derived from them.

We should not store only:

```text
Adherence = 72%
```

because that loses the underlying evidence.

---

# 14.26 No artificial adherence

The system must never infer:

```text
SMS delivered → Taken
Call answered → Taken
Telegram delivered → Taken
```

Only explicit patient interaction should create `TAKEN`.

This is one of the most important clinical integrity requirements.

---

# 14.27 Patient response validation

Before recording a response, the backend should validate:

1. The patient exists.
2. The reminder exists.
3. The notification exists.
4. The response channel is valid.
5. The response belongs to that patient.
6. The reminder is eligible for response.
7. The response hasn't already been processed as the same event.
8. The response is valid for that channel.

Only then should it affect adherence.

---

# 14.28 Idempotency

Example:

```text
Telegram callback:
TAKEN
```

received twice.

We should end up with:

```text
One logical response
```

not:

```text
Two adherence records
```

The same applies to:

* SMS
* Voice DTMF
* webhooks
* background workers

---

# 14.29 Adherence should be immutable enough for auditability

I recommend that we **do not simply overwrite history**.

Instead of:

```text
TAKEN → NOT_TAKEN
```

with the original value lost, preserve the event history.

For example:

```text
Response events:
08:03 → TAKEN
08:05 → NOT_TAKEN

Final state:
CONFLICTING
```

This gives us traceability.

---

# 14.30 Proposed formal Adherence Requirements

| ID      | Requirement                                                                                                                                      |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| ADH-001 | Phase 1 shall record patient medication-adherence responses for individual reminder occurrences.                                                 |
| ADH-002 | Each medication reminder occurrence shall have an independently trackable adherence state.                                                       |
| ADH-003 | The system shall support TAKEN, NOT_TAKEN, and NO_RESPONSE outcomes.                                                                             |
| ADH-004 | The system shall support PENDING for reminder occurrences whose adherence response window remains open.                                          |
| ADH-005 | The system shall support a CONFLICTING state when contradictory valid patient responses are received.                                            |
| ADH-006 | A valid explicit patient response shall be required to classify an occurrence as TAKEN.                                                          |
| ADH-007 | A valid explicit patient response shall be required to classify an occurrence as NOT_TAKEN.                                                      |
| ADH-008 | Failure to receive a response shall not automatically be classified as NOT_TAKEN.                                                                |
| ADH-009 | Notification delivery shall remain separate from medication adherence.                                                                           |
| ADH-010 | A delivered SMS shall not automatically create a TAKEN adherence result.                                                                         |
| ADH-011 | An answered Voice call shall not automatically create a TAKEN adherence result.                                                                  |
| ADH-012 | A delivered Telegram message shall not automatically create a TAKEN adherence result.                                                            |
| ADH-013 | Telegram Taken responses shall be recorded as TAKEN.                                                                                             |
| ADH-014 | Telegram Not Taken responses shall be recorded as NOT_TAKEN.                                                                                     |
| ADH-015 | SMS response `1` shall be recorded as TAKEN.                                                                                                     |
| ADH-016 | SMS response `2` shall be recorded as NOT_TAKEN.                                                                                                 |
| ADH-017 | Voice DTMF `1` shall be recorded as TAKEN.                                                                                                       |
| ADH-018 | Voice DTMF `2` shall be recorded as NOT_TAKEN.                                                                                                   |
| ADH-019 | Unrecognized patient responses shall not automatically modify adherence state.                                                                   |
| ADH-020 | Every valid adherence response shall be associated with the correct reminder occurrence.                                                         |
| ADH-021 | The system shall prevent duplicate processing of the same logical patient response.                                                              |
| ADH-022 | Multiple responses through different channels shall not create multiple independent adherence decisions for the same reminder occurrence.        |
| ADH-023 | Contradictory valid responses shall not be silently overwritten.                                                                                 |
| ADH-024 | The system shall preserve response events required to understand contradictory or changed responses.                                             |
| ADH-025 | The system shall record the response channel for each valid adherence response.                                                                  |
| ADH-026 | The system shall record the scheduled time and response time for adherence events.                                                               |
| ADH-027 | The system shall distinguish on-time responses from late responses.                                                                              |
| ADH-028 | Late explicit patient responses shall be preserved as response events rather than silently discarded.                                            |
| ADH-029 | Late responses shall not be treated as on-time adherence merely because the response was eventually received.                                    |
| ADH-030 | The system shall use a defined adherence response window for each reminder occurrence.                                                           |
| ADH-031 | An occurrence with no valid response when its response window closes shall become NO_RESPONSE.                                                   |
| ADH-032 | PENDING shall be used only while the applicable adherence response window remains open.                                                          |
| ADH-033 | A valid patient response shall stop further notification escalation where applicable.                                                            |
| ADH-034 | Notification attempts that occurred before a patient response shall remain in notification history.                                              |
| ADH-035 | Adherence records shall remain associated with the patient and relevant medication/reminder history.                                             |
| ADH-036 | Transfer of a patient between authorized doctors shall not destroy historical adherence records.                                                 |
| ADH-037 | Discontinuing a medication shall not destroy historical adherence records.                                                                       |
| ADH-038 | Changing or discontinuing a medication schedule shall not destroy historical adherence records.                                                  |
| ADH-039 | Historical adherence data shall remain traceable to the relevant medication, schedule, reminder occurrence, and patient.                         |
| ADH-040 | Doctors shall only access adherence information for patients they are authorized to access.                                                      |
| ADH-041 | Adherence summaries shall be derived from underlying reminder/adherence records rather than replacing the underlying records.                    |
| ADH-042 | The system shall not infer medication adherence solely from notification delivery, call connection, or channel availability.                     |
| ADH-043 | The backend shall validate patient, reminder, notification, channel, ownership, and response eligibility before recording an adherence response. |
| ADH-044 | Adherence processing shall be idempotent.                                                                                                        |
| ADH-045 | Adherence response history shall provide sufficient traceability for audit and clinical monitoring.                                              |
| ADH-046 | The system shall preserve the distinction between patient-reported adherence and system-observed notification status.                            |

---

# One important recommendation before we lock Step 14

There is **one remaining policy decision** I want us to make explicitly.

Suppose:

```text
08:00 → Reminder
08:05 → Patient presses TAKEN
```

Later:

```text
08:20 → Patient presses NOT_TAKEN
```

I recommend our final state becomes:

> **CONFLICTING**

rather than simply changing:

```text
TAKEN → NOT_TAKEN
```

And both responses remain in the history.

This is the safest approach because the system should **not silently decide which contradictory patient statement is clinically correct**.

So the model would be:

```text
                    Reminder
                       │
              ┌────────┴────────┐
              │                 │
          Response 1        Response 2
           08:05              08:20
           TAKEN            NOT_TAKEN
              │                 │
              └────────┬────────┘
                       ↓
                 CONFLICTING
```

A doctor can then see that there was an inconsistent response.

**I recommend we adopt this rule and lock Step 14 with it.**
