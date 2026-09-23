Great. **Step 11 — Telegram Requirements is now locked.** ✅

We now move to **Step 12 — SMS Requirements**.

As agreed, this step will define **only SMS-specific requirements**. The general retry/fallback/orchestration rules remain in Step 10, and Voice will be handled separately in Step 13.

# Step 12 — SMS Requirements

## 12.1 Purpose of SMS

SMS is a notification channel for patients who:

* don't use Telegram,
* don't have Telegram connected,
* cannot receive Telegram,
* or reach SMS through the configured fallback policy.

The architecture is:

```text
Reminder
   ↓
Notification Orchestrator
   ↓
SMS Adapter
   ↓
SMS Provider
   ↓
Mobile Network
   ↓
Patient's Phone
```

The important boundary is:

> **Our application should communicate with an SMS provider, not directly with Ethio Telecom.**

The provider is responsible for connecting our application to the appropriate telecommunications network(s).

This means our application should not contain Ethio Telecom-specific logic.

---

# 12.2 SMS Provider Abstraction

This is especially important because we want to start with free/mock services and later move to production.

We should have:

```text
Notification Orchestrator
          ↓
      SMS Adapter
          ↓
    SMS Provider Interface
          ↓
     ┌───────────────┐
     ↓               ↓
Mock SMS          Real SMS
 Provider          Provider
```

During development:

```text
SMS → Mock Provider
```

Later:

```text
SMS → Production Provider
```

The core notification system should remain unchanged.

---

# 12.3 Why we need an adapter

We should **not** write our business logic like:

```text
if provider == "ProviderA":
    ...
```

everywhere.

Instead:

```text
sendSMS(message)
```

is handled by our SMS interface.

The provider-specific implementation handles:

* API authentication
* provider request format
* provider response format
* delivery status mapping
* provider errors
* webhook processing

This makes replacing the provider much easier.

---

# 12.4 SMS recipient

SMS requires a valid phone number.

The patient's phone number should be stored separately from:

* Patient ID
* Telegram Chat ID
* Doctor information

Conceptually:

```text
Patient
├── Patient ID
├── Phone number
├── Telegram connection
└── Preferred language
```

The phone number should be normalized into a consistent format.

For Ethiopia, we should establish one canonical representation during implementation rather than allowing multiple inconsistent formats.

---

# 12.5 Phone number validation

Before sending SMS, the system should validate the destination number.

For example:

```text
Patient phone
      ↓
Validate
      ↓
Valid? ── No → SMS unavailable
  │
 Yes
  ↓
Send SMS
```

An invalid number should be treated differently from a temporary provider failure.

---

# 12.6 SMS message language

We already established three Phase 1 languages:

* English
* Amharic
* Afaan Oromoo

SMS must respect:

```text
Patient → Preferred language
```

Example:

```text
Patient A
Preferred language = English
       ↓
English SMS
```

and:

```text
Patient B
Preferred language = Afaan Oromoo
       ↓
Afaan Oromoo SMS
```

The same centralized localization/template system used for Telegram should be used where practical.

---

# 12.7 SMS message content

The SMS should be concise because SMS has length limitations.

Example:

```text
Medication Reminder:
Amoxicillin 500 mg.
Please take 1 tablet now.
Reply 1 if Taken, 2 if Not Taken.
```

However, we need to be careful about message length when using Amharic or Afaan Oromoo because character encoding can affect SMS segmentation and cost.

Therefore, the SMS implementation must account for:

* character encoding,
* message length,
* multipart SMS,
* provider limits,
* provider pricing implications.

We don't need to choose the provider yet.

---

# 12.8 Patient response

We have already agreed that the patient should be able to respond to an SMS reminder.

I recommend:

```text
1 = Taken
2 = Not Taken
```

For example:

```text
SMS:
"Medication reminder: Take Amoxicillin 500 mg.
Reply 1 if Taken, 2 if Not Taken."
```

Patient:

```text
1
```

Backend:

```text
SMS response
    ↓
Validate
    ↓
Find notification/reminder
    ↓
TAKEN
```

Patient:

```text
2
```

becomes:

```text
NOT_TAKEN
```

---

# 12.9 Why numeric responses?

Numeric responses are preferable to requiring the patient to type:

> "I have taken my medicine."

because users may type:

* taken
* Took
* yes
* 1
* done
* okay
* etc.

That creates unnecessary natural-language parsing.

With:

```text
1 = Taken
2 = Not Taken
```

the system has a much more deterministic interface.

---

# 12.10 What about other replies?

Suppose the patient replies:

```text
"yes"
```

or:

```text
"done"
```

or:

```text
"3"
```

or:

```text
"please call me"
```

We should **not automatically interpret arbitrary text as adherence** in Phase 1.

Instead:

```text
Unrecognized response
       ↓
Record as unrecognized
       ↓
Do not automatically classify as TAKEN
```

We can optionally send:

> "Please reply 1 for Taken or 2 for Not Taken."

The exact behavior can be finalized during implementation.

---

# 12.11 Associating an incoming SMS with a reminder

This is one of the hardest parts of SMS.

Suppose the patient receives:

```text
08:00
Take Amoxicillin
```

Then replies:

```text
1
```

How does our backend know which reminder the `1` belongs to?

We need a deterministic association strategy.

The system can use information such as:

```text
Sender phone number
        +
Active/pending reminder
        +
Response timing/context
```

The exact algorithm should be defined carefully.

We should **not simply say**:

> "Find the latest reminder for this phone number."

because there could be multiple medication reminders close together.

---

# 12.12 Example of the ambiguity problem

Suppose:

```text
08:00 → Medication A
08:05 → Medication B
```

Patient sends:

```text
1
```

Which medication did they take?

Therefore, the SMS response design must ensure that the backend can reliably associate the response with the correct reminder.

One approach is to include a short response reference in the message, for example:

```text
Reply:
1 ABC = Taken
2 ABC = Not Taken
```

But that makes the patient experience less simple.

Another approach is to allow only one active SMS response context at a time for a patient.

We should evaluate this during the detailed API/data design.

**For now, the requirement is that every valid SMS response must be unambiguously associated with the correct notification/reminder.**

---

# 12.13 Incoming SMS is different from outgoing SMS

There are two directions:

### Outgoing

```text
Our Backend
   ↓
SMS Provider
   ↓
Patient
```

### Incoming

```text
Patient
   ↓
Mobile Network
   ↓
SMS Provider
   ↓
Webhook
   ↓
Our Backend
```

We need both.

---

# 12.14 SMS provider webhook

The provider will typically notify our backend about events such as:

```text
SMS accepted
SMS delivered
SMS failed
SMS undelivered
```

and potentially:

```text
Incoming SMS
```

The exact events depend on the provider.

Our backend should expose a secure webhook endpoint for providers that support webhooks.

---

# 12.15 Webhook idempotency

A provider may send the same webhook more than once.

For example:

```text
SMS_DELIVERED_123

received
received again
received again
```

Our system should not create three delivery events that trigger three fallback actions.

Instead:

```text
Webhook
   ↓
Check event identity
   ↓
Already processed?
 ├── YES → safely ignore
 └── NO  → process
```

---

# 12.16 Delivery status

We should maintain an internal status model independent of the provider.

For example:

```text
QUEUED
SENT
DELIVERED
FAILED
UNDELIVERED
```

The exact statuses will be finalized when we implement the notification data model.

The provider may use completely different terminology.

For example:

```text
Provider:
"accepted"
"delivered"
"expired"
```

Our adapter translates:

```text
Provider status
      ↓
Internal status
```

---

# 12.17 Sending is not delivery

This distinction is critical.

If our provider says:

```text
SMS → ACCEPTED
```

that doesn't necessarily mean:

> The patient's phone received the SMS.

So:

```text
ACCEPTED ≠ DELIVERED
```

The Notification Orchestrator should use the strongest reliable status available.

---

# 12.18 SMS technical failure

Suppose:

```text
08:00
SMS attempt
     ↓
Provider timeout
```

The SMS adapter reports:

```text
TEMPORARY_FAILURE
```

The Notification Orchestrator decides whether to:

```text
Retry SMS
```

or:

```text
Fallback → Voice
```

according to Step 10.

The SMS component itself should **not decide to call the patient**.

---

# 12.19 SMS permanent failure

Example:

```text
Invalid phone number
```

Retrying ten times doesn't fix it.

So:

```text
Invalid number
    ↓
NON_RETRYABLE
    ↓
Notification Orchestrator
    ↓
Next channel according to policy
```

---

# 12.20 SMS rate limiting

Production providers may impose limits.

For example:

```text
100 SMS/minute
```

or another provider-specific limit.

Our adapter should translate rate-limit responses into a retryable condition where appropriate.

The retry scheduler should then apply backoff.

---

# 12.21 Duplicate SMS prevention

We must protect against:

```text
Worker A → SMS
Worker B → SMS
```

for the same notification.

The system should guarantee that the same logical notification action is not unintentionally sent twice.

This follows the idempotency requirement from Step 10.

---

# 12.22 SMS and fallback

Example:

```text
Telegram
   ↓
No response / escalation condition
   ↓
SMS
```

The SMS system doesn't need to know **why** it was selected.

It simply receives:

```text
Send SMS notification for Reminder #123
```

and sends it.

Likewise, if SMS fails:

```text
SMS
 ↓
FAILED
 ↓
Notification Orchestrator
 ↓
Voice
```

Voice selection is outside the SMS module.

---

# 12.23 SMS and patient response stopping escalation

Suppose:

```text
Telegram
   ↓
No response
   ↓
SMS
```

Patient replies:

```text
1
```

Then:

```text
SMS → TAKEN
```

The Notification Orchestrator should stop further escalation:

```text
❌ Voice
```

The response is then passed to the adherence system.

---

# 12.24 Late SMS responses

Suppose:

```text
08:00 Reminder
08:10 SMS
09:00 Reminder expires
09:30 Patient replies "1"
```

The backend must not blindly treat this as a normal current response.

It must check:

```text
Reminder state
Notification state
Response time
```

and then apply the late-response policy.

The exact adherence behavior will be defined in Step 14.

---

# 12.25 SMS security

We need to protect:

### Provider credentials

Never:

```text
GitHub repository
.env committed to Git
Frontend code
```

Instead use secure environment/secret storage.

### Webhooks

Incoming provider requests should be authenticated/verified according to the provider's supported mechanism.

### Logs

Avoid logging unnecessary:

```text
Patient medical information
Full SMS content
Provider secrets
```

where it isn't needed.

---

# 12.26 SMS provider independence

Our system should not assume:

> "The provider is EthiopianSmsProvider."

Instead:

```text
SMS Provider Interface
        │
        ├── MockSmsProvider
        │
        ├── ProviderA
        │
        └── ProviderB
```

This means we can evaluate providers later based on:

* Ethiopia coverage
* Price
* Delivery reliability
* API quality
* Sender ID support
* Incoming SMS support
* Delivery reports
* Webhooks
* Support
* Regulatory requirements

We do **not** need to choose the production provider in Step 12.

---

# 12.27 Development provider

For development, I recommend:

```text
MockSmsProvider
```

The mock should allow us to simulate:

```text
SUCCESS
FAILED
TIMEOUT
DELIVERED
UNDELIVERED
INVALID_NUMBER
RATE_LIMITED
```

and incoming patient replies.

For example:

```text
Mock SMS Dashboard/Test API
       ↓
Simulate "1"
       ↓
Backend receives incoming SMS
       ↓
TAKEN
```

This lets us develop the entire workflow without paying for real SMS.

---

# 12.28 Production provider

When the product is ready:

```text
MockSmsProvider
      ↓
ProductionSmsProvider
```

The Notification Orchestrator should remain unchanged.

Only configuration and provider implementation should change.

This is one of the major architectural goals of our project.

---

# 12.29 Important Ethiopia consideration

Because the product will operate in Ethiopia, we should **not assume that every generic international SMS provider will provide the same capabilities in Ethiopia**.

Before production, we need to verify the chosen provider's:

* Ethiopia coverage
* Sender ID availability
* Two-way SMS support
* Delivery reports
* API/webhook support
* Pricing
* Regulatory/compliance requirements
* Local telecom connectivity

Whether we ultimately work directly with Ethio Telecom or through another provider is a **production provider-selection decision**, not something we should hard-code into the application.

---

# 12.30 Proposed SMS Requirements

| ID      | Requirement                                                                                                            |
| ------- | ---------------------------------------------------------------------------------------------------------------------- |
| SMS-001 | Phase 1 shall support SMS as a patient notification channel.                                                           |
| SMS-002 | SMS shall be accessed through an SMS provider abstraction rather than directly from core business logic.               |
| SMS-003 | The application shall support replaceable SMS provider implementations.                                                |
| SMS-004 | Development shall support a mock SMS provider.                                                                         |
| SMS-005 | The mock SMS provider shall simulate realistic success and failure scenarios.                                          |
| SMS-006 | Production SMS provider selection shall remain independent of the core notification orchestration logic.               |
| SMS-007 | The system shall require a valid patient phone number before attempting SMS delivery.                                  |
| SMS-008 | Patient phone numbers shall be stored separately from Telegram identifiers.                                            |
| SMS-009 | Phone numbers shall be normalized and validated before SMS delivery.                                                   |
| SMS-010 | SMS reminder content shall support English, Amharic, and Afaan Oromoo in Phase 1.                                      |
| SMS-011 | SMS content shall use the patient's configured preferred language.                                                     |
| SMS-012 | SMS templates shall use the application's centralized localization/template mechanism where appropriate.               |
| SMS-013 | SMS content shall account for SMS character encoding and message-length limitations.                                   |
| SMS-014 | Phase 1 shall support patient SMS responses.                                                                           |
| SMS-015 | `1` shall represent Taken.                                                                                             |
| SMS-016 | `2` shall represent Not Taken.                                                                                         |
| SMS-017 | Valid SMS responses shall be processed by the backend and passed to the adherence system.                              |
| SMS-018 | Unrecognized SMS responses shall not automatically be interpreted as Taken or Not Taken.                               |
| SMS-019 | The system shall provide an appropriate handling path for unrecognized patient responses.                              |
| SMS-020 | Each valid incoming SMS response shall be unambiguously associated with the correct notification/reminder.             |
| SMS-021 | Incoming SMS shall be handled separately from outgoing SMS delivery.                                                   |
| SMS-022 | The system shall support provider delivery-status events where the selected provider supports them.                    |
| SMS-023 | Provider-specific delivery statuses shall be translated into the application's internal notification status model.     |
| SMS-024 | SMS accepted/sent status shall not automatically be treated as SMS delivered.                                          |
| SMS-025 | The system shall support secure provider webhooks where applicable, using authentication and validation mechanisms.     |
| SMS-026 | Duplicate provider webhook events shall not create duplicate state transitions or notification actions.                |
| SMS-027 | SMS provider errors shall be classified into appropriate retryable/non-retryable conditions where possible.            |
| SMS-028 | SMS retry behavior shall follow the general Notification Requirements defined in Step 10.                              |
| SMS-029 | SMS fallback behavior shall be controlled by the Notification Orchestrator, not by the SMS provider adapter.           |
| SMS-030 | The SMS component shall not directly initiate Voice fallback.                                                          |
| SMS-031 | Duplicate SMS sends shall be prevented through the system's idempotency and concurrency controls.                      |
| SMS-032 | SMS notification attempts shall remain associated with their originating reminder occurrence.                          |
| SMS-033 | A valid patient response shall stop further notification escalation where applicable.                                  |
| SMS-034 | Late SMS responses and responses through closed channels shall be validated against the reminder and notification state before being accepted, following the late-response and closed-channel policies defined in Step 14B. |
| SMS-035 | SMS provider credentials shall be stored securely and shall not be committed to source control.                        |
| SMS-036 | SMS webhook endpoints shall implement appropriate authentication/verification mechanisms per provider capabilities.     |
| SMS-037 | SMS logs shall avoid unnecessary exposure of sensitive patient or clinical information.                                |
| SMS-038 | The system shall not assume a specific Ethiopian telecom provider in its core SMS implementation.                      |
| SMS-039 | Production SMS provider selection shall verify Ethiopia-specific coverage and required capabilities before deployment. |
| SMS-040 | The system shall support provider replacement without rewriting the core reminder and notification business logic.     |
| SMS-041 | SMS processing shall support the escalation timing defined by the Notification Requirements.                           |
| SMS-042 | SMS delivery failure shall not automatically be interpreted as patient non-adherence.                                  |
| SMS-043 | The system shall maintain one active SMS response context per patient at any given time.                                |
| SMS-044 | The backend shall ensure that incoming SMS responses can be unambiguously associated with the currently active SMS response context for that patient. |
| SMS-045 | Old/closed SMS response contexts shall not be able to modify adherence for closed or expired reminders.                 |
| SMS-046 | Multiple reminder occurrences due at the same time may be combined into one SMS communication.                          |
| SMS-047 | When simultaneous reminders are combined in one SMS, each reminder occurrence shall be addressable using numbered item codes (e.g., "1A", "2A" for Reminder A; "1B", "2B" for Reminder B). |
| SMS-048 | Each reminder occurrence shall maintain its own independent adherence decision regardless of SMS message grouping.      |
| SMS-049 | Communication grouping in SMS shall not merge underlying reminder or adherence records.                                 |
| SMS-050 | Responses received through a closed SMS channel (after escalation has moved to another channel) shall be recorded as response events but shall not modify the reminder's adherence state. |
| SMS-051 | An SMS notification attempt shall capture the patient's phone number at the time the attempt is created.                |
| SMS-052 | Future reminder occurrences shall use the patient's current valid phone number configuration.                           |
| SMS-053 | Changing a patient's phone number after an SMS notification attempt has been created shall not silently rewrite the destination of that already-created attempt. |
| SMS-054 | The same phone number may appear on multiple patient records in Phase 1.                                                |
| SMS-055 | The system shall not enforce global phone number uniqueness across patients.                                            |
| SMS-056 | Patient identity shall be determined by internal Patient ID, not by phone number.                                       |
| SMS-057 | SMS webhook endpoints shall implement provider-supported authenticity verification mechanisms.                          |
| SMS-058 | State-changing SMS webhooks that cannot be authenticated sufficiently shall not be trusted for adherence or state-changing operations. |
| SMS-059 | Initial Phase 1 production deployment may launch with SMS inactive while Telegram is active.                            |
| SMS-060 | SMS shall remain architecturally prepared (provider abstraction exists, orchestrator supports SMS) such that SMS can be activated in production without redesigning core reminder/notification logic. |
| SMS-061 | SMS delivery status shall remain distinct from patient adherence status, consistent with the requirement that delivery does not equal adherence. |

---

# 12.31 The architecture we are locking toward

At this point, our architecture is becoming quite clean:

```text
                         REMINDER
                            │
                            ▼
                NOTIFICATION ORCHESTRATOR
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
        Telegram          SMS            Voice
          Adapter        Adapter         Adapter
             │              │              │
             ▼              ▼              ▼
        Telegram API    SMS Provider   Voice Provider
```

And separately:

```text
Patient Response
      │
      ├── Telegram callback
      ├── SMS incoming message
      └── Voice keypad
               │
               ▼
          Backend
               │
               ▼
          Adherence
```

This separation will allow us to replace:

```text
Mock SMS
   ↓
Real SMS Provider
```

without changing the rest of the system.

---

# One issue we need to resolve before locking Step 12

There is one important **SMS UX decision** that we should not ignore.

We said:

> Patient replies `1` or `2`.

But consider this situation:

```text
08:00 → Amoxicillin reminder
08:05 → Patient receives SMS

08:10 → Another medication reminder
08:15 → Patient replies "1"
```

Our backend needs to know **which reminder the `1` belongs to**.

So before we lock Step 12, I recommend we explicitly decide how we will associate an incoming SMS with its reminder.

### My recommendation

For Phase 1, keep the patient experience simple:

> **The system should maintain a single active SMS response context per patient at a time, and each incoming `1`/`2` response should be associated with that active notification only when the association is unambiguous.**

If there are multiple active SMS reminders that could match the response, the system should **not guess**. It should handle the message as ambiguous and follow a defined safe path.

We can also design a short reminder/reference code if the product requirements eventually require multiple simultaneous SMS response contexts.

I recommend we make this decision **before accepting Step 12**, because it affects the database and API design later.


---

## Cross-Document Consistency Requirements

### Consistency with Step 10B — Notification/Escalation

**SMS-CONSISTENCY-001**

SMS requirements SHALL be consistent with Step 10B notification requirements, specifically:
- NOTIF-INVARIANT-001: Communication grouping preserves independent adherence (SMS-046 through SMS-049)
- NOTIF-INVARIANT-002: Provider failure never becomes NOT_TAKEN (SMS-042)
- NOTIF-INVARIANT-003: Delivery does not equal adherence (SMS-024, SMS-061)
- NOTIF-INVARIANT-004: One active response channel (SMS-043, SMS-044, SMS-045, SMS-050)
- NOTIF-REQ-002: SMS availability requires valid phone number (SMS-007, SMS-009)
- NOTIF-REQ-004: Phone number cardinality (SMS-054, SMS-055, SMS-056)
- NOTIF-REQ-005: In-flight contact change behavior (SMS-051, SMS-052, SMS-053)
- NOTIF-REQ-013: SMS simultaneous reminders (SMS-046 through SMS-049)
- NOTIF-REQ-014: SMS response association (SMS-043, SMS-044, SMS-045)
- NOTIF-REQ-016: Provider abstraction (SMS-002, SMS-003, SMS-040)
- NOTIF-REQ-017: Notification Orchestrator responsibility (SMS-028, SMS-029, SMS-030)
- NOTIF-REQ-018: Production channel activation (SMS-059, SMS-060)

### Consistency with Step 14B — Adherence

**SMS-CONSISTENCY-002**

SMS response handling SHALL be consistent with Step 14B adherence requirements, specifically:
- ADH-PRINCIPLE-001: One reminder, one adherence decision (SMS-048, SMS-049)
- ADH-REQ-002: Closed channels cannot change adherence (SMS-045, SMS-050)
- ADH-REQ-006: SMS valid responses are "1"=TAKEN, "2"=NOT_TAKEN (SMS-015, SMS-016, SMS-017)
- ADH-REQ-008: Unrecognized responses do not change adherence (SMS-018, SMS-019)
- ADH-REQ-011: Delivery ≠ adherence (SMS-024, SMS-042, SMS-061)
- ADH-REQ-012: Provider failure ≠ NOT_TAKEN (SMS-042)
- ADH-REQ-013-015: Response timing and windows (SMS-034)

### Consistency with Step 19 — Security

**SMS-CONSISTENCY-003**

SMS security requirements SHALL be consistent with Step 19 security requirements, specifically:
- Webhook authenticity verification (SMS-036, SMS-057, SMS-058)
- Credential protection (SMS-035)
- Sensitive data minimization in logs (SMS-037)
- Idempotency and replay protection (SMS-026, SMS-031)

---

## Requirement Ownership

**Step 12 owns:**
- SMS-specific channel behavior
- SMS provider abstraction requirements
- Outgoing SMS delivery
- Incoming SMS response handling
- SMS message formatting and encoding constraints
- SMS webhook processing
- SMS-specific localization requirements
- SMS response context management

**Step 12 does NOT own:**
- Reminder generation (Step 9)
- Notification escalation logic (Step 10)
- Adherence decision rules (Step 14)
- Audit logging (Step 15)
- Security implementation details (Step 19)
- Configuration/settings UI (Step 17)
- Patient clinical records (Step 6)

---

## Implementation Notes

- Exact SMS provider API selection is an implementation detail
- SMS message templates will be defined during implementation
- Webhook endpoint URL structure is an implementation detail
- Phone number normalization format (E.164 or other) is an implementation detail subject to provider requirements
- SMS encoding (GSM-7, UCS-2, etc.) selection is provider-specific
- Numbered item code format for simultaneous reminders (e.g., "1A"/"2A" vs other schemes) is an implementation/template detail
- SMS rate limiting handling is provider-specific and shall be abstracted
- SMS provider credentials management follows Step 19 secret management requirements
- Ethiopia-specific provider selection criteria will be evaluated before production launch

---

## Unresolved Human Decisions

**NONE IDENTIFIED** for Step 12.

The one active SMS response context per patient (SMS-043) was already approved.

Simultaneous reminder numbered item code syntax is correctly deferred to implementation/template design.

Late-response policy details are owned by Step 14B.
