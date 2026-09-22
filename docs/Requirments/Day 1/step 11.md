Great. **Step 10 is now locked.** ✅

We can move to **Step 11 — Telegram Requirements**.

This step will define **only Telegram-specific behavior**. We will not yet define SMS, Voice, or general notification orchestration because those were handled in Steps 10, 12, and 13.

# Step 11 — Telegram Requirements

## 11.1 Purpose of Telegram in our system

Telegram is one of the communication channels through which the patient receives a medication reminder.

The basic flow is:

```text
Medication Schedule
        ↓
Reminder Occurrence
        ↓
Notification Orchestrator
        ↓
Telegram Adapter
        ↓
Telegram Bot
        ↓
Patient's Telegram
        ↓
Patient Response
```

Telegram is particularly useful in Phase 1 because it can support **interactive responses**.

For example:

> 🔔 Medication Reminder
> Please take Amoxicillin 500 mg — 1 tablet.

Then:

**[Taken] [Not Taken]**

The patient's button selection is sent back to our backend.

---

# 11.2 Telegram Bot

Yes, we should use a **Telegram Bot**.

The bot is the application's interface with Telegram.

Conceptually:

```text
Our Backend
     ↓
Telegram Bot API
     ↓
Our Patient Reminder Bot
     ↓
Patient
```

The bot is **not the same thing as our backend**.

The backend contains our business logic.

The Telegram Bot provides communication with Telegram users.

---

# 11.3 One bot for the application

For Phase 1, I recommend:

> **One Telegram Bot for the entire application.**

Not one bot per doctor and not one bot per patient.

For example:

```text
Patient Reminder Bot
        │
 ┌──────┼──────┐
 ↓      ↓      ↓
Patient Patient Patient
 A       B      C
```

Our backend identifies which patient owns each Telegram account.

This keeps management much simpler.

---

# 11.4 Patient must link Telegram to their account

A patient should not automatically be considered connected to Telegram simply because we know their name or phone number.

We need an explicit linking process.

For example:

```text
Doctor creates patient
        ↓
Patient exists in our system
        ↓
Patient chooses Telegram
        ↓
Patient opens our Telegram Bot
        ↓
Bot provides /start
        ↓
Backend links Telegram account
        ↓
Telegram = CONNECTED
```

This gives us a reliable association:

```text
Our Patient ID
       ↕
Telegram Chat ID
```

---

# 11.5 Do not use the patient's name as the Telegram identity

We should **not** identify a Telegram user using:

```text
Patient name
Phone number
Username
```

as the primary identity.

Instead, Telegram provides its own user/chat identifiers.

We store the appropriate Telegram identifier and associate it with our internal patient ID.

Conceptually:

```text
Patient
ID: PAT-123

Telegram
Chat ID: 123456789
```

This association is what the notification system uses.

---

# 11.6 Telegram linking security

This is important.

Imagine someone knows:

> "Abebe is a patient in our system."

They should not be able to connect their own Telegram account to Abebe's medical account.

Therefore, linking should use a secure, short-lived mechanism.

For example:

```text
Doctor creates patient
        ↓
Backend generates secure linking token
        ↓
Patient opens bot
        ↓
Bot receives token
        ↓
Backend validates token
        ↓
Telegram account linked
```

The token should:

* Be difficult to guess
* Expire
* Be usable only according to the intended linking flow
* Not expose medical information

The exact implementation will be decided during API/security design.

---

# 11.7 Telegram should not expose sensitive information during linking

A linking URL or token should not contain things like:

```text
Patient name
Medication
Diagnosis
Doctor name
```

It should contain only what is necessary to securely establish the relationship.

---

# 11.8 Telegram reminder message

The reminder message should contain the information necessary for the patient to understand the reminder.

For example:

> 🔔 Medication Reminder
> Amoxicillin 500 mg
> Take 1 tablet now.

We need to be careful about how much clinical information is exposed.

The message should follow the minimum necessary principle.

---

# 11.9 Patient's preferred language

Step 6 established that the patient has:

```text
Preferred language:
English
Amharic
Afaan Oromoo
```

Telegram reminders must respect this preference.

For example:

```text
Patient A
Preferred language = English
        ↓
English reminder
```

while:

```text
Patient B
Preferred language = Afaan Oromoo
        ↓
Afaan Oromoo reminder
```

The backend should select the appropriate localized message.

---

# 11.10 Do not hard-code translated text everywhere

I recommend that we don't write:

```text
if language == "English":
    ...
if language == "Amharic":
    ...
if language == "Oromo":
    ...
```

throughout the application.

Instead, use a localization/message-template system.

Conceptually:

```text
Reminder Template
       ↓
Language
       ↓
Localized Message
       ↓
Telegram
```

This will make adding another language later much easier.

---

# 11.11 Telegram response buttons

For Phase 1, I recommend using **inline buttons**.

Example:

```text
┌─────────────────────────────┐
│ 🔔 Medication Reminder      │
│                             │
│ Amoxicillin 500 mg          │
│ Take 1 tablet now.          │
│                             │
│ [ ✅ Taken ] [ ❌ Not Taken ]│
└─────────────────────────────┘
```

This is better than asking the patient to type a response.

---

# 11.12 Patient response semantics

We already agreed:

> If the patient replies **Not Taken**, we record `NOT_TAKEN`.

So:

```text
Taken button
     ↓
TAKEN

Not Taken button
     ↓
NOT_TAKEN
```

Telegram itself does not determine adherence.

The backend does.

Telegram simply provides the patient's response.

---

# 11.13 What happens after the patient responds?

Suppose:

```text
08:00
Telegram reminder
```

Patient presses:

> Taken

The Telegram callback goes to our backend:

```text
Telegram
   ↓
Callback
   ↓
Backend
   ↓
Validate
   ↓
Find Reminder
   ↓
Record response
   ↓
Adherence processing
```

Then the Notification Orchestrator should stop further escalation for that reminder.

So:

```text
Telegram → TAKEN
      ↓
Stop SMS
      ↓
Stop Voice
```

assuming those channels haven't already completed an action.

---

# 11.14 What if the patient presses "Not Taken"?

Example:

```text
08:00 Telegram
      ↓
Patient presses NOT TAKEN
```

Backend records:

```text
Adherence = NOT_TAKEN
```

Further escalation should normally stop because the patient has responded.

We will finalize the exact adherence semantics in Step 14.

---

# 11.15 What if the patient presses a button twice?

Example:

```text
08:03 → Taken
08:04 → Taken again
```

We should not create:

```text
❌ Response #1
❌ Response #2
```

as two separate adherence decisions.

The system should safely handle duplicate callbacks.

This is another reason we need idempotency.

---

# 11.16 What if the patient changes their answer?

Example:

```text
08:03 → Taken

08:05 → Not Taken
```

This is more complicated.

We should **not silently overwrite the first response**.

We need a clear policy for response corrections.

My recommendation is:

> The first valid response establishes the reminder's initial adherence decision, while later responses are recorded as subsequent events and handled according to the adherence correction policy.

We will define the exact rule in Step 14.

---

# 11.17 Expired buttons

Suppose the reminder is no longer active:

```text
Reminder
   ↓
EXPIRED
```

but the patient still has the Telegram message with:

**[Taken] [Not Taken]**

They press **Taken**.

The backend must not blindly treat that as a normal current response.

Instead:

```text
Button pressed
     ↓
Backend checks reminder state
     ↓
Expired?
     ↓
Apply late-response policy
```

This is another reason the Telegram bot must never independently determine adherence.

---

# 11.18 Telegram callback security

When the patient presses a button, Telegram sends callback information to our backend.

The backend should verify:

* The callback came through the expected Telegram integration
* The callback corresponds to a valid notification
* The callback belongs to the correct patient/chat
* The reminder is still eligible for response
* The callback has not already been processed

We should never trust the callback data blindly.

---

# 11.19 Do not put sensitive data directly in callback data

We should avoid putting something like:

```text
patient=Abebe
medication=Amoxicillin
dose=500mg
```

inside the Telegram button callback.

Instead, use a short opaque identifier/reference.

For example:

```text
callback:
REM_RESP_xxxxxxxxx
```

Then our backend looks up the actual reminder.

This is safer and easier to control.

---

# 11.20 Telegram webhook

For production, I recommend using Telegram's webhook mechanism rather than repeatedly polling Telegram from our backend.

Conceptually:

```text
Patient
   ↓
Telegram
   ↓
Webhook HTTPS request
   ↓
Our Backend
   ↓
Process event
```

Our backend should expose a secure webhook endpoint.

---

# 11.21 Webhook duplicate events

We must assume external systems can retry webhook delivery.

So we might receive:

```text
Callback Event #ABC
Callback Event #ABC
```

Our system must not process the same event twice.

Therefore:

```text
Telegram Event
      ↓
Event ID / idempotency check
      ↓
Already processed?
   YES → ignore safely
   NO  → process
```

---

# 11.22 Telegram failure scenarios

The system needs to handle:

### Bot unavailable

```text
Telegram API
     ↓
Failure
```

### Network timeout

```text
Backend
   ↓
Timeout
```

### Patient not connected

```text
Patient
   ↓
Telegram = NOT_CONNECTED
```

### Invalid chat

```text
Telegram
   ↓
Chat unavailable
```

### Telegram rate limiting

```text
Telegram
   ↓
Rate limit
```

These should be converted into our internal notification status/error categories rather than leaking Telegram-specific errors into the rest of our application.

---

# 11.23 Telegram provider abstraction

We should still maintain an adapter boundary:

```text
Notification Orchestrator
          ↓
Telegram Adapter
          ↓
Telegram Bot API
```

This means the core notification system doesn't need to know Telegram API details.

For example:

```text
sendTelegramReminder(...)
```

rather than spreading Telegram API calls throughout the application.

---

# 11.24 Development vs production

Telegram is slightly different from SMS and Voice.

For development, we can use a **real Telegram Bot** because Telegram Bot API usage itself does not require us to pay per patient message in the same way a commercial SMS/voice provider does.

So:

```text
Development:
Real Telegram Bot
```

can be used while:

```text
Development:
Mock SMS
Mock Voice
```

are used.

Later:

```text
Production:
Same Telegram Bot architecture
Production SMS Provider
Production Voice Provider
```

---

# 11.25 What if Telegram is unavailable?

This is where Step 10 takes over.

Telegram should report something like:

```text
SUCCESS
FAILED
UNAVAILABLE
```

to the Notification Orchestrator.

It should **not** decide:

> "Now send SMS."

The orchestrator makes that decision.

So:

```text
Telegram Adapter
       ↓
Result
       ↓
Notification Orchestrator
       ↓
Fallback Policy
       ↓
SMS
```

This separation is very important.

---

# 11.26 Telegram and patient phone number

A Telegram account does not necessarily mean that we should use its phone number as our patient's phone number.

The patient's phone number remains a separate patient/contact field.

Therefore:

```text
Patient
├── Phone number
├── Telegram connection
│     └── Telegram Chat ID
└── Preferred language
```

This gives us flexibility.

---

# 11.27 Telegram should not be the only patient identity

Our primary identity remains:

```text
Patient ID
```

Telegram is simply one communication identity associated with that patient.

This means we can later have:

```text
Patient
   │
   ├── Telegram
   ├── SMS
   └── Voice
```

without changing the patient's core identity.

---

# 11.28 Recommended Telegram requirements

Here is the proposed formal specification.

| ID      | Requirement                                                                                                                                                     |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TEL-001 | Phase 1 shall support Telegram as a patient notification channel.                                                                                               |
| TEL-002 | The application shall use a Telegram Bot for patient communication.                                                                                             |
| TEL-003 | Phase 1 shall use one application-level Telegram Bot unless a future requirement necessitates otherwise.                                                        |
| TEL-004 | A patient shall explicitly link their Telegram account to their patient account before receiving Telegram reminders.                                            |
| TEL-005 | Telegram linking shall use a secure, temporary linking mechanism.                                                                                               |
| TEL-006 | Telegram linking tokens shall expire and shall not expose unnecessary patient/clinical information.                                                             |
| TEL-007 | The system shall associate the patient's internal patient ID with the appropriate Telegram chat/user identifier.                                                |
| TEL-008 | Telegram identifiers shall not replace the application's internal patient ID.                                                                                   |
| TEL-009 | The system shall not use patient name or Telegram username as the primary patient identity.                                                                     |
| TEL-010 | Telegram reminders shall be generated from the Reminder/Notification system rather than independently by the bot.                                               |
| TEL-011 | Telegram reminder content shall support English, Amharic, and Afaan Oromoo in Phase 1.                                                                          |
| TEL-012 | Telegram message language shall follow the patient's configured preferred language.                                                                             |
| TEL-013 | Telegram messages shall use a centralized localization/template mechanism.                                                                                      |
| TEL-014 | Phase 1 Telegram reminders shall support interactive Taken and Not Taken responses.                                                                             |
| TEL-015 | Telegram responses shall be processed by the backend and not treated as adherence decisions by Telegram itself.                                                 |
| TEL-016 | A valid Taken response shall be passed to the adherence system as a Taken response.                                                                             |
| TEL-017 | A valid Not Taken response shall be passed to the adherence system as a Not Taken response.                                                                     |
| TEL-018 | A valid patient response shall cause notification escalation to stop where applicable.                                                                          |
| TEL-019 | Duplicate Telegram callback events shall not create duplicate patient responses or duplicate state transitions.                                                 |
| TEL-020 | The system shall validate that a Telegram response belongs to the expected patient and reminder.                                                                |
| TEL-021 | Telegram callback data shall not unnecessarily contain sensitive patient or clinical information.                                                               |
| TEL-022 | Telegram callbacks shall use an opaque/internal reference to identify the relevant notification/reminder.                                                       |
| TEL-023 | The backend shall verify the reminder state and channel eligibility before accepting a Telegram response.                                                       |
| TEL-024 | Responses received for closed/expired reminders or through closed channels shall be handled according to the defined late-response and closed-channel policies defined in Step 14B rather than automatically treated as current responses. |
| TEL-025 | Telegram webhook processing shall be idempotent.                                                                                                                |
| TEL-026 | Telegram webhook events shall be processed securely using appropriate authentication and validation mechanisms.                                                 |
| TEL-027 | Telegram-specific failures shall be translated into the application's internal notification status/error model.                                                 |
| TEL-028 | Telegram failures shall be handled by the Notification Orchestrator according to the general retry/fallback policy.                                             |
| TEL-029 | Telegram integration code shall be isolated behind a Telegram adapter/interface.                                                                                |
| TEL-030 | The core notification business logic shall not directly depend on Telegram API implementation details.                                                          |
| TEL-031 | Telegram notification attempts shall be associated with the originating reminder occurrence.                                                                    |
| TEL-032 | Telegram delivery/processing status shall remain distinct from patient adherence status.                                                                        |
| TEL-033 | Telegram integration shall not prevent SMS or Voice from being used as fallback channels.                                                                       |
| TEL-034 | Development shall be able to use a real Telegram Bot while SMS and Voice use mock providers.                                                                    |
| TEL-035 | Telegram bot credentials shall be stored securely and shall not be committed to source control.                                                                 |
| TEL-036 | Telegram integration logs shall avoid unnecessary exposure of sensitive patient/clinical information.                                                           |
| TEL-037 | Telegram notification processing shall support the notification escalation timing defined in Step 10.                                                           |
| TEL-038 | The system shall support Telegram account unlinking and relinking.                                                                                              |
| TEL-039 | When a patient relinks Telegram: the old Telegram association shall become inactive, old linking tokens shall be invalidated, the new Telegram identity shall become active, and historical notification and adherence records shall remain unchanged. |
| TEL-040 | Multiple reminder occurrences due at the same time may be combined into one Telegram message for presentation efficiency.                                       |
| TEL-041 | When simultaneous reminders are combined in one Telegram message, each reminder occurrence shall retain its own independent inline response buttons.            |
| TEL-042 | Each reminder occurrence shall maintain its own independent adherence decision regardless of Telegram message grouping.                                         |
| TEL-043 | Communication grouping in Telegram shall not merge underlying reminder or adherence records.                                                                    |
| TEL-044 | Responses received through a closed Telegram channel (after escalation has moved to another channel) shall be recorded as response events but shall not modify the reminder's adherence state. |
| TEL-045 | Telegram webhook endpoints shall implement provider-supported authenticity verification mechanisms.                                                             |
| TEL-046 | State-changing Telegram webhooks that cannot be authenticated sufficiently shall not be trusted for adherence or state-changing operations.                     |
| TEL-047 | Telegram delivery status shall remain distinct from patient adherence status, consistent with the requirement that delivery does not equal adherence.           |

---

# One decision I recommend we make before locking Step 11

There is one Telegram-specific UX/security decision I want us to settle.

## How should the patient link their Telegram account?

I recommend this flow:

```text
Doctor creates patient
       ↓
System generates "Connect Telegram" action
       ↓
Patient receives/gets a secure link or code
       ↓
Patient opens Telegram Bot
       ↓
/start + secure token
       ↓
Backend verifies token
       ↓
Telegram connected
       ↓
Patient can receive reminders
```

After linking:

```text
Patient
   ↓
Telegram = CONNECTED
   ↓
Telegram reminders enabled
```

If they disconnect Telegram:

```text
Telegram = DISCONNECTED
   ↓
Telegram skipped
   ↓
Notification Orchestrator
   ↓
SMS / Voice according to policy
```

I recommend this because it avoids trying to match patients to Telegram accounts using names or phone numbers, which is unreliable and potentially unsafe.

---

## Step 11 status

At this point, I recommend we **review/approve the Telegram requirements rather than immediately moving to Step 12**.

The important things we have established are:

**Telegram's job:**

```text
Send reminder
       ↓
Provide Taken / Not Taken interaction
       ↓
Return patient response
```

**Telegram's job is NOT:**

```text
❌ Decide adherence
❌ Decide whether SMS should be sent
❌ Decide whether Voice should be called
❌ Manage medication schedules
❌ Manage reminders
```

Those responsibilities stay in our backend.

Once you accept Step 11, we'll lock it and move to **Step 12 — SMS Requirements**, where we'll define SMS sending, patient replies, incoming SMS handling, provider delivery reports, retries, and the mock/production provider boundary.


---

## Cross-Document Consistency Requirements

### Consistency with Step 10B — Notification/Escalation

**TEL-CONSISTENCY-001**

Telegram requirements SHALL be consistent with Step 10B notification requirements, specifically:
- NOTIF-INVARIANT-001: Communication grouping preserves independent adherence (TEL-040 through TEL-043)
- NOTIF-INVARIANT-003: Delivery does not equal adherence (TEL-032, TEL-047)
- NOTIF-INVARIANT-004: One active response channel (TEL-044)
- NOTIF-REQ-001: Telegram availability rules
- NOTIF-REQ-010: Telegram simultaneous reminders (TEL-040 through TEL-043)
- NOTIF-REQ-011: Telegram lifecycle support (TEL-038, TEL-039)

### Consistency with Step 14B — Adherence

**TEL-CONSISTENCY-002**

Telegram response handling SHALL be consistent with Step 14B adherence requirements, specifically:
- ADH-PRINCIPLE-001: One reminder, one adherence decision (TEL-042, TEL-043)
- ADH-REQ-002: Closed channels cannot change adherence (TEL-044)
- ADH-REQ-005: Telegram valid responses are Taken/Not Taken (TEL-014, TEL-016, TEL-017)
- ADH-REQ-008: Unrecognized responses do not change adherence (cross-reference for Telegram text vs buttons)
- ADH-REQ-011: Delivery ≠ adherence (TEL-032, TEL-047)

### Consistency with Step 19 — Security

**TEL-CONSISTENCY-003**

Telegram security requirements SHALL be consistent with Step 19 security requirements, specifically:
- Webhook authenticity verification (TEL-045, TEL-046)
- Credential protection (TEL-035)
- Sensitive data minimization in logs (TEL-036)
- Callback data security (TEL-021, TEL-022)

---

## Requirement Ownership

**Step 11 owns:**
- Telegram-specific channel behavior
- Telegram linking/unlinking/relinking
- Telegram bot configuration
- Telegram message formatting and localization
- Telegram callback handling
- Telegram webhook processing
- Telegram provider abstraction

**Step 11 does NOT own:**
- Notification escalation logic (Step 10)
- Adherence decision rules (Step 14)
- Reminder generation (Step 9)
- Audit logging (Step 15)
- Security implementation details (Step 19)
- Configuration/settings UI (Step 17)

---

## Implementation Notes

- Exact Telegram Bot API version and library selection are implementation details
- Telegram message formatting templates will be defined during implementation
- Webhook endpoint URL structure is an implementation detail
- Callback payload encoding/structure is an implementation detail subject to security requirements
- Telegram rate limiting handling is provider-specific and shall be abstracted
- Telegram Bot credentials management follows Step 19 secret management requirements
