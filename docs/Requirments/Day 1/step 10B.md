Yes. **I agree with you**, and I would change the Step 10 design accordingly.

The previous idea of:

> "Send Telegram at 08:00 → wait 2 hours → if no response, send SMS at 10:00"

is **not a good default design for a medication reminder system**.

For medication adherence, the **medication due time is the anchor**, not the patient's response timeout.

## The better approach: fixed escalation timing

Suppose the medication is due at **08:00**.

We can define a notification escalation window around that time:

```text
08:00
  │
  ├── Telegram
  │
  │   if Telegram fails → SMS quickly
  │
  │   if Telegram succeeds but patient doesn't respond
  │       ↓
  │   wait a SHORT configured period
  │       ↓
 08:15
  │
  ├── SMS
  │
  │   if still no response
  │       ↓
 08:30
  │
  └── Voice
```

The exact times **08:15 / 08:30 are examples only**. We should define the actual values as requirements before implementation.

The important principle is:

> **A patient should not receive the next reminder hours after the medication was due simply because they didn't respond.**

---

# I recommend separating three different clocks

This is the key architectural improvement.

### 1. Medication due time

This comes from the schedule.

```text
Medication due:
08:00
```

This is fixed.

---

### 2. Notification escalation timing

This determines when we try another channel.

For example:

```text
08:00 → Telegram
08:10 → SMS
08:20 → Voice
```

These times are calculated relative to the **08:00 reminder**, not relative to an arbitrary 2-hour response window.

---

### 3. Adherence response window

This answers a different question:

> "How long do we allow the patient to respond before we classify the reminder as having no response?"

For example:

```text
08:00 → medication due
08:00–09:00 → response period
09:00 → NO_RESPONSE if nothing received
```

This is an **adherence/business rule**, and we will properly define it in **Step 14**.

So we should **not use the 2-hour response window as the channel escalation timer**.

---

# Example: what I think our system should do

Suppose:

```text
Medication: Amoxicillin
Due: 08:00

Channel policy:
Telegram → SMS → Voice
```

We could configure:

```text
08:00 → Telegram
08:10 → SMS if escalation condition is met
08:20 → Voice if escalation condition is met
```

Again, these intervals are only examples.

### Patient responds at 08:03

```text
08:00 Telegram
08:03 Patient → TAKEN
```

Then:

```text
❌ No SMS
❌ No Voice
```

Done.

---

### Telegram fails immediately

```text
08:00 Telegram
      ↓
Technical failure
      ↓
Retry / fallback according to policy
      ↓
SMS
```

We don't wait until 10:00.

---

### Telegram succeeds but patient doesn't respond

Suppose the escalation policy says:

```text
08:00 Telegram
08:10 SMS
08:20 Voice
```

Then:

```text
08:00 → Telegram
08:10 → no response → SMS
08:20 → no response → Voice
```

This is much more appropriate for a medication reminder.

---

# But there is one more important question

I don't recommend automatically saying:

> "No response to Telegram = send SMS."

because the system needs to know **what kind of event occurred**.

For example:

### Case 1 — Telegram failed

```text
Telegram → FAILED
```

We can move to SMS according to the technical-failure policy.

### Case 2 — Telegram delivered

```text
Telegram → DELIVERED
Patient → no response
```

Now the system follows the **scheduled escalation policy**.

### Case 3 — Patient responded

```text
Telegram → DELIVERED
Patient → TAKEN
```

Stop all further channels.

So the orchestrator essentially evaluates:

```text
Reminder
   ↓
Current time
   ↓
Notification state
   ↓
Patient response?
   ↓
Provider status?
   ↓
Escalation policy
   ↓
Next action
```

---

# I would also change our terminology

Previously we used:

> "response window"

for too many things.

I recommend we use three explicit terms:

| Term                          | Meaning                                                  |
| ----------------------------- | -------------------------------------------------------- |
| **Due Time**                  | When medication should be taken                          |
| **Escalation Schedule**       | When the system moves between notification channels      |
| **Adherence Response Window** | How long the system accepts a response for that reminder |

This will prevent confusion when we implement the system.

---

# Example of a production policy

A possible Phase 1 policy could eventually look like:

```text
Reminder
Due: 08:00

Primary:
08:00 → Telegram

Fallback:
If Telegram technically fails:
    retry according to retry policy

If Telegram remains unavailable:
    SMS at configured escalation time

If no patient response:
    Voice at configured escalation time

Stop escalation immediately if:
    TAKEN
    or NOT_TAKEN
```

Then separately:

```text
Adherence
Response accepted until:
09:00

If no response by 09:00:
    NO_RESPONSE
```

Notice the difference:

**Voice might happen at 08:20**, while **NO_RESPONSE might not be determined until 09:00**.

That is perfectly reasonable.

---

# One thing I would NOT do

I would not hard-code:

```text
Telegram = 10 minutes
SMS = 10 minutes
Voice = 10 minutes
```

into the application.

Instead, make the policy configurable:

```text
Channel 1: Telegram
Offset: 0 minutes

Channel 2: SMS
Offset: configurable

Channel 3: Voice
Offset: configurable
```

Then later, based on real-world testing and clinical/business requirements, we can change the timing **without rewriting the notification engine**.

---

# My revised recommendation for Step 10

I would therefore modify the requirements we discussed earlier.

### Remove this idea:

> ❌ "Wait up to 2 hours before moving to the next channel."

### Replace it with:

> ✅ **Channel escalation shall follow a predefined/configurable escalation schedule relative to the reminder's due time.**

And:

> ✅ **Patient response timeout shall be treated separately from notification escalation timing.**

And:

> ✅ **A patient response shall immediately stop further notification escalation for that reminder, where appropriate.**

And:

> ✅ **Technical/provider failures may cause earlier retry or fallback according to the failure policy rather than waiting for the scheduled escalation time.**

And:

> ✅ **The system shall not send a late fallback notification merely because a long adherence response window has expired.**

---

## One more thing: this affects Step 14

This decision makes our later **Adherence Requirements** much cleaner.

For example:

```text
08:00  Medication due
08:00  Telegram
08:10  SMS
08:20  Voice
09:00  Adherence response window closes
```

If the patient presses **Taken at 08:25**, that's a patient response after the Voice notification but before the adherence deadline.

If they respond at **09:30**, we'd need to decide whether that is a late response and how it is recorded.

That belongs in Step 14.

### So I recommend we change Step 10 before locking it:

**Due time ≠ escalation timing ≠ adherence response window.**

That is the production-oriented design I would use.

And I agree with your concern: **waiting two hours to send the next channel is not appropriate as the default medication-reminder escalation strategy.**


---

## Approved Notification Requirements for Phase 1

### Critical Notification Invariants

**NOTIF-INVARIANT-001 — Simultaneous Reminder Communication Grouping**

The system MAY group multiple reminder occurrences that are due at the same time into a single communication for presentation efficiency (one Telegram message, one SMS, one Voice call).

However, each reminder occurrence SHALL remain an independent domain object with:
- Its own reminder ID
- Its own notification history
- Its own response events
- Its own adherence decision

Communication grouping is a presentation/delivery optimization only. The system SHALL NOT merge underlying reminder or adherence records.

**NOTIF-INVARIANT-002 — Provider Failure Handling**

If a notification provider fails to deliver a reminder, the system SHALL:
1. Retry per the defined retry policy
2. Transition to the next escalation channel when applicable per policy
3. Preserve the notification attempt and history
4. Record the failure appropriately
5. **NEVER** automatically convert a `PROVIDER_FAILURE` into patient `NOT_TAKEN` adherence

Provider failure and patient adherence are separate concepts.

**NOTIF-INVARIANT-003 — Delivery Does Not Equal Adherence**

The system SHALL maintain a clear distinction across all notification channels:
- Telegram message delivered ≠ medication TAKEN
- SMS delivered ≠ medication TAKEN  
- Voice call answered ≠ medication TAKEN

Only an accepted valid patient response SHALL change adherence status.

**NOTIF-INVARIANT-004 — One Active Response Channel**

For each reminder occurrence, only one response channel SHALL be active at any given time.

When escalation moves from one channel to another (e.g., Telegram → SMS), the previous channel's response window SHALL close, and the new channel's response window SHALL open.

Responses received through a closed channel SHALL be recorded as events but SHALL NOT modify the reminder's adherence state.

### Communication Availability Requirements

**NOTIF-REQ-001 — Telegram Availability**

Telegram SHALL be available as a notification channel only when:
- The patient has explicitly linked their Telegram account to their patient record, AND
- The backend has a valid Telegram association for that patient

The system SHALL NOT attempt Telegram delivery when these conditions are not met.

**NOTIF-REQ-002 — SMS Availability**

SMS SHALL be available as a notification channel only when:
- The patient has a valid configured phone number

The system SHALL NOT attempt SMS delivery without a valid phone number.

**NOTIF-REQ-003 — Voice Availability**

Voice SHALL be available as a notification channel only when:
- The patient has a valid configured phone number

The system SHALL NOT attempt Voice delivery without a valid phone number.

**NOTIF-REQ-004 — Phone Number Cardinality**

The system SHALL NOT assume that phone numbers are globally unique to one patient.

The same phone number MAY appear on multiple patient records.

Patient identity SHALL be determined by the internal Patient ID, not by phone number.

Database constraints SHALL NOT enforce phone number uniqueness across patients.

**NOTIF-REQ-005 — In-Flight Contact Change Behavior**

When a notification attempt is created, it SHALL capture the destination/contact information (phone number, Telegram chat ID) at that time.

Changing a patient's phone number or Telegram association after a notification attempt has been created SHALL NOT silently rewrite the destination of that already-created attempt.

Future reminder occurrences SHALL use the current valid contact configuration.

### Escalation and Timing Requirements

**NOTIF-REQ-006 — Escalation Timing Anchor**

Channel escalation timing SHALL be calculated relative to the medication due time, not relative to patient response timeouts.

**NOTIF-REQ-007 — Fixed Escalation Schedule**

The system SHALL support configurable escalation schedules that define when each channel is attempted relative to the reminder due time.

Example: Telegram at T+0min, SMS at T+10min, Voice at T+20min (where T is medication due time).

**NOTIF-REQ-008 — Response Window Separate from Escalation**

The adherence response window (how long a patient can respond) SHALL be treated separately from notification escalation timing.

A patient MAY be able to respond after escalation has moved to another channel, subject to the adherence response window rules defined in Step 14 - Adherence Requirements.

**NOTIF-REQ-009 — Immediate Escalation on Technical Failure**

If a notification channel experiences a technical/provider failure, the system MAY move to the next escalation channel immediately or after retry, according to the failure policy, rather than waiting for the scheduled escalation time.

### Telegram-Specific Notification Requirements

**NOTIF-REQ-010 — Telegram Simultaneous Reminders**

Multiple reminder occurrences due at the same time MAY be combined into one Telegram message.

Each reminder SHALL be presented with its own independent response mechanism (inline buttons).

Each reminder SHALL maintain its own adherence decision.

**NOTIF-REQ-011 — Telegram Lifecycle Support**

The system SHALL support Telegram link/unlink/relink operations.

When a patient relinks Telegram:
- The old Telegram association SHALL become inactive
- Old tokens SHALL be invalidated
- The new Telegram identity SHALL become active
- Historical notification and adherence records SHALL remain unchanged

**NOTIF-REQ-012 — Telegram Not Primary Identity**

Telegram identity SHALL NOT serve as the primary patient identity.

The system SHALL use internal Patient ID as the primary identifier.

### SMS-Specific Notification Requirements

**NOTIF-REQ-013 — SMS Simultaneous Reminders**

Multiple reminder occurrences due at the same time MAY be combined into one SMS message using numbered item codes.

Each reminder occurrence SHALL retain its own independent adherence decision.

**NOTIF-REQ-014 — SMS Response Association**

Phase 1 SHALL support one active SMS response context per patient.

The system SHALL ensure that an incoming SMS response can be unambiguously associated with the currently active SMS response context.

Old SMS messages SHALL NOT be able to modify closed/expired reminders.

### Voice-Specific Notification Requirements

**NOTIF-REQ-015 — Voice Simultaneous Reminders**

Multiple reminder occurrences due at the same time MAY be combined into one Voice call.

Each medication/reminder SHALL be presented separately during the call with independent DTMF response collection.

Each reminder SHALL maintain its own independent adherence decision.

### Provider and Architecture Requirements

**NOTIF-REQ-016 — Provider Abstraction**

The core application SHALL NOT directly depend on specific SMS or Voice vendor implementations.

The system SHALL use provider abstractions (conceptual interfaces such as SmsProvider, VoiceProvider, TelegramProvider).

Provider-specific statuses and errors SHALL be mapped to the internal notification model.

**NOTIF-REQ-017 — Notification Orchestrator Responsibility**

The Notification Orchestrator SHALL own:
- Retry decisions
- Escalation transitions
- Provider failure handling
- Channel eligibility determination
- Idempotency and concurrency handling
- Notification attempt creation

Notification providers SHALL NOT contain business escalation logic.

**NOTIF-REQ-018 — Production Channel Activation**

The system SHALL support an initial production launch that is Telegram-only, with SMS and Voice remaining inactive.

SMS and Voice SHALL be architecturally prepared (provider abstraction exists, orchestrator supports channels) such that production providers can be activated later without redesigning core reminder/notification logic.

---

## Requirement Ownership and Cross-References

These notification requirements work in conjunction with:
- **Step 9**: Reminder occurrence generation
- **Step 11**: Telegram-specific implementation details
- **Step 12**: SMS-specific implementation details
- **Step 13**: Voice-specific implementation details
- **Step 14**: Adherence response rules and state management
- **Step 15**: Audit logging of notification events
- **Step 17**: Configuration of escalation policies and timing
- **Step 19**: Security requirements for webhooks and provider credentials
