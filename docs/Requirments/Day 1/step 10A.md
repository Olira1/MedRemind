Excellent. **Step 9 is now locked.** ✅

We move to **Day 1 — Step 10: Notification Requirements**.

This step defines the **general notification orchestration layer**. It will apply to Telegram, SMS, and Voice, but it will **not** define the technical details of each channel. Those belong to Steps 11–13.

# Step 10 — Notification Requirements

## 10.1 What is a Notification?

We established:

```text
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

A **Reminder** answers:

> "The patient should take this medication now."

A **Notification** answers:

> "How are we going to communicate that reminder to the patient?"

For example:

```text
Reminder #123
   │
   ├── Telegram notification
   ├── SMS notification
   └── Voice notification
```

These are separate records/events connected to the same Reminder Occurrence.

---

# 10.2 Notification Orchestrator

I recommend that we create a central **Notification Orchestrator**.

Conceptually:

```text
                    Reminder
                       │
                       ▼
             Notification Orchestrator
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Telegram        SMS         Voice
       Adapter       Adapter      Adapter
          │            │            │
          ▼            ▼            ▼
      Provider      Provider     Provider
```

The orchestrator decides:

* Which channel to use
* In what order
* When to retry
* When to fallback
* When to stop
* Whether a notification has already been processed
* How provider results affect the next action

The individual channel implementations should **not** make these decisions.

---

# 10.3 Multiple channels

We have already agreed that multiple channels are allowed.

But we should now make the meaning precise:

> **Multiple channels do not mean that Telegram, SMS, and Voice are automatically sent simultaneously.**

Instead, a reminder can have an ordered notification policy.

Example:

```text
Priority 1 → Telegram
Priority 2 → SMS
Priority 3 → Voice
```

The system uses the next channel when the defined fallback conditions are met.

---

# 10.4 Default channel order

I recommend the default order:

```text
1. Telegram
2. SMS
3. Voice
```

Why?

### Telegram

* Very low communication cost
* Convenient interaction
* Supports buttons
* Good for patient responses

### SMS

* Works on ordinary mobile phones
* Does not require Internet access
* More broadly available

### Voice

* Doesn't depend on reading text
* Useful for patients who have difficulty using messaging
* Can provide audio in the patient's preferred language
* Potentially more expensive

So Voice becomes the strongest fallback rather than the first option.

---

# 10.5 Is the order fixed?

**No.**

The architecture should support a configurable order.

For example:

Patient A:

```text
Telegram → SMS → Voice
```

Patient B:

```text
SMS → Voice
```

Patient C:

```text
Voice → SMS
```

However, I recommend that Phase 1 have a **system default policy**, while allowing the Doctor/admin configuration we explicitly decide to expose.

We should not unnecessarily complicate the UI by allowing every possible combination unless there is a real requirement.

---

# 10.6 What triggers fallback?

This is the most important part.

A fallback should **not** happen merely because a notification was sent.

We need to distinguish:

### Provider failure

Example:

```text
Telegram
   ↓
Provider/API error
   ↓
FAILED
```

This may immediately trigger retry or fallback depending on the error.

### Delivery failure

Example:

```text
SMS
   ↓
Provider accepts message
   ↓
Later delivery report
   ↓
UNDELIVERED
```

This can trigger fallback.

### No patient response

Example:

```text
Telegram
   ↓
DELIVERED
   ↓
No response
```

This is different.

We should not automatically interpret it as delivery failure.

---

# 10.7 Recommended fallback decision

I recommend a **policy-driven approach**.

For each channel, the system can have:

```text
Channel
  ↓
Attempt
  ↓
Provider result
  ↓
Policy evaluation
  ↓
Retry / Wait / Fallback / Stop
```

For example:

```text
Telegram attempt
      ↓
   FAILED
      ↓
Retry?
 ┌────┴────┐
YES       NO
 ↓         ↓
Retry    SMS
```

If Telegram is successfully delivered:

```text
Telegram
   ↓
DELIVERED
   ↓
Wait for patient response
```

If the configured response/fallback window expires:

```text
No response
   ↓
Fallback policy
   ↓
SMS
```

This means **provider delivery state and patient response timeout can both potentially trigger fallback**, but they are separate triggers.

---

# 10.8 We should distinguish retry from fallback

This is very important.

### Retry

Try the **same channel again**.

```text
SMS attempt #1
      ↓
temporary failure
      ↓
SMS attempt #2
```

### Fallback

Move to the **next channel**.

```text
Telegram
   ↓
failed / timeout according to policy
   ↓
SMS
```

So:

```text
FAILURE
   ↓
Retry same channel?
   ↓
If retry exhausted
   ↓
Fallback channel?
```

This avoids immediately switching channels for a temporary provider problem.

---

# 10.9 Retryable vs non-retryable failures

Not every failure should be retried.

### Example: retryable

```text
Provider temporarily unavailable
Network timeout
Temporary server error
Rate-limit response
```

We may retry.

### Example: non-retryable

```text
Invalid phone number
Patient has no Telegram connection
Invalid provider credentials
Unsupported destination
```

Repeatedly retrying these doesn't help.

Therefore, the notification system needs a classification such as:

```text
RETRYABLE
NON_RETRYABLE
```

The provider adapters can translate provider-specific errors into our internal categories.

---

# 10.10 Retry limits

We should never retry forever.

Example:

```text
Attempt 1
Attempt 2
Attempt 3
       ↓
Maximum attempts reached
       ↓
Fallback / FAILED
```

The maximum number should be configurable.

We will decide the exact values during technical configuration/testing rather than hardcoding them into business requirements.

---

# 10.11 Backoff

Retries should use increasing delays.

Conceptually:

```text
Attempt 1 → immediately
Attempt 2 → after short delay
Attempt 3 → after longer delay
```

This protects both our system and the external provider.

The exact delay strategy will be decided during implementation.

---

# 10.12 Notification response window

We also need a defined period during which the patient can respond.

For example:

```text
08:00
 ↓
Reminder notification
 ↓
Response window
 ↓
10:00
 ↓
No response
```

At that point the system can apply the configured fallback/expiration policy.

### Important

The **response window is not the same as the reminder schedule**.

The schedule says:

> Medication is due at 08:00.

The response window says:

> How long should the system wait for a response before taking the next action?

---

# 10.13 Recommended initial response window

I previously suggested **2 hours** as an initial default.

I still recommend using:

> **2 hours as the default configurable response window.**

But we should make it configurable rather than permanently hardcoding it.

Example:

```text
Default response window = 2 hours
```

Later, the product could support:

```text
30 minutes
1 hour
2 hours
4 hours
```

if the clinical workflow requires it.

---

# 10.14 Important: should every channel wait 2 hours?

**No.**

This is an important refinement.

Suppose Telegram fails because of an obvious provider error:

```text
08:00
Telegram → FAILED
```

We should not necessarily wait until 10:00 before trying SMS.

The fallback policy can distinguish:

```text
Technical failure
     ↓
Retry/fallback sooner
```

from:

```text
Delivered successfully
     ↓
Patient hasn't responded
     ↓
Wait for response window
```

Therefore, the system should have **different timing rules for technical failure and response timeout**.

---

# 10.15 Example complete fallback flow

Let's use:

```text
Telegram → SMS → Voice
```

and response window:

```text
2 hours
```

### Scenario A — Telegram works

```text
08:00
   ↓
Telegram sent
   ↓
Telegram delivered
   ↓
Patient presses TAKEN at 08:05
   ↓
DONE
```

No SMS.

No Voice.

---

### Scenario B — Telegram fails immediately

```text
08:00
   ↓
Telegram attempt
   ↓
Temporary failure
   ↓
Retry
   ↓
Still fails
   ↓
SMS
```

SMS succeeds:

```text
SMS delivered
   ↓
Patient replies "1"
   ↓
TAKEN
```

No Voice.

---

### Scenario C — Telegram delivered but no response

```text
08:00
   ↓
Telegram delivered
   ↓
No patient response
   ↓
Response window expires
   ↓
SMS
```

If SMS receives:

```text
2
```

then:

```text
NOT_TAKEN
```

No Voice.

---

### Scenario D — All channels fail

```text
Telegram
   ↓
FAILED
   ↓
SMS
   ↓
FAILED
   ↓
Voice
   ↓
NO_ANSWER
```

Then the system records:

```text
Notification:
FAILED / NO_RESPONSE

Adherence:
NO_RESPONSE
```

**Not Taken** must not be inferred.

---

# 10.16 Should fallback happen after "no response"?

This needs careful design.

I recommend:

> **Yes, if the notification policy explicitly enables response-based fallback.**

For example:

```text
Telegram → wait for response → SMS → wait → Voice
```

But this should be a policy, not a universal assumption.

Why?

Because some patients may prefer:

```text
Telegram only
```

while another patient may need:

```text
SMS → Voice
```

Therefore the notification policy needs to support:

```text
Fallback trigger:
- technical failure
- delivery failure
- response timeout
```

with appropriate configuration.

---

# 10.17 Multiple channels and patient preferences

The patient has communication preferences.

For example:

```text
Patient:
Preferred language: Afaan Oromoo

Channels:
☑ Telegram
☑ SMS
☑ Voice
```

Another patient might have:

```text
Channels:
☐ Telegram
☑ SMS
☑ Voice
```

The notification orchestrator must respect the channels enabled for that patient.

It should never send through a channel that the patient is not configured/eligible to use.

---

# 10.18 Patient has no Telegram account

Suppose the Doctor enables:

```text
Telegram → SMS → Voice
```

but the patient has never linked Telegram.

The system should not repeatedly attempt Telegram.

Instead:

```text
Telegram
   ↓
Patient not eligible
   ↓
Skip
   ↓
SMS
```

This is different from a temporary Telegram provider failure.

---

# 10.19 Patient has no phone number

If SMS and Voice require a phone number and the patient doesn't have one:

```text
Telegram → SMS → Voice
```

could become:

```text
Telegram
   ↓
Failed
   ↓
SMS unavailable
   ↓
Voice unavailable
```

The system should record why those channels were unavailable.

It should not attempt impossible deliveries.

---

# 10.20 One reminder, multiple notification attempts

A reminder might produce:

```text
Reminder #123
│
├── Telegram attempt #1
├── Telegram attempt #2
├── SMS attempt #1
└── Voice attempt #1
```

All of these belong to:

```text
Reminder #123
```

This is why we need:

```text
Reminder ID
Notification ID
Attempt ID
```

or an equivalent data model.

We will design the exact database structure later.

---

# 10.21 Idempotency

The Notification Orchestrator must be idempotent.

Example:

```text
Worker A:
process Reminder #123

Worker B:
process Reminder #123
```

Both workers must not send duplicate SMS messages.

The system should recognize:

> "This notification action has already been created/processed."

and safely avoid duplication.

---

# 10.22 Concurrency protection

This is related but not identical to idempotency.

Suppose two background workers see:

```text
Reminder #123 = DUE
```

at exactly the same time.

Only one should acquire the right to process the notification action.

Conceptually:

```text
Reminder #123
       ↓
Worker A → LOCK/CLAIM → PROCESS
Worker B → cannot claim → WAIT/EXIT
```

This will be important when we implement background jobs.

---

# 10.23 Provider abstraction

The Notification Orchestrator should never depend directly on a specific paid provider.

Instead:

```text
Notification Orchestrator
        ↓
     SMS Adapter
        ↓
   SMS Provider Interface
        ↓
 ┌───────────────┐
 │ Mock Provider │
 │ Real Provider │
 └───────────────┘
```

Same for Voice.

Telegram also has its own adapter.

This allows us to develop with mock providers and later replace them.

---

# 10.24 Development environment

For development:

```text
Telegram → Real Telegram Bot
SMS      → Mock SMS Provider
Voice    → Mock Voice Provider
```

The mock provider should simulate realistic outcomes:

```text
SUCCESS
FAILED
TIMEOUT
DELIVERED
UNDELIVERED
```

and webhook events where appropriate.

This lets us test the notification engine without paying for every test message/call.

---

# 10.25 Production environment

Production will use:

```text
Telegram → Telegram Bot API
SMS      → Selected production SMS provider
Voice    → Selected production voice provider
```

The Notification Orchestrator should remain unchanged.

Only configuration/provider implementation should change wherever possible.

---

# 10.26 Provider outage

Suppose the SMS provider is down.

The system should not crash the entire reminder engine.

Instead:

```text
SMS Provider
     ↓
Unavailable
     ↓
Notification failure
     ↓
Retry/fallback policy
     ↓
Voice
```

Other patients/reminders should continue processing.

One provider failure must not bring down the entire notification system.

---

# 10.27 Notification audit trail

We need enough information to answer:

> What happened to this reminder?

For example:

```text
Reminder: #123

08:00 Telegram attempted
08:00 Telegram failed
08:01 Telegram retry
08:01 Telegram failed
08:02 SMS attempted
08:02 SMS delivered
08:12 Patient replied "1"
```

This is extremely useful for:

* Doctor investigation
* Admin troubleshooting
* Production monitoring
* Customer support
* Auditing

---

# 10.28 Security

Notification processing must not expose sensitive information unnecessarily.

For example, logs should avoid storing:

* Full patient medical information
* Unnecessary medication details
* Authentication credentials
* Provider secrets
* API tokens

Provider credentials must be stored securely through environment/secret management.

Webhook endpoints must also verify that incoming requests are legitimate.

---

# 10.29 Notification monitoring

For production, we should be able to monitor things like:

```text
Telegram success rate
SMS success rate
Voice success rate
Provider failures
Retry counts
Fallback counts
Webhook failures
Notification latency
```

This doesn't mean we need a huge monitoring dashboard in Phase 1.

But the system architecture should generate enough structured information to support monitoring.

---

# 10.30 Proposed Notification Requirements

These are the requirements I recommend we lock for Step 10:

| ID      | Requirement                                                                                                      |
| ------- | ---------------------------------------------------------------------------------------------------------------- |
| NOT-001 | The system shall treat notification delivery as separate from the underlying reminder occurrence.                |
| NOT-002 | The system shall provide a central notification orchestration layer.                                             |
| NOT-003 | The notification orchestration layer shall determine channel selection, retry, fallback, and stopping behavior.  |
| NOT-004 | Phase 1 shall support Telegram, SMS, and Voice notification channels.                                            |
| NOT-005 | The system shall support multiple notification channels for a reminder.                                          |
| NOT-006 | Multiple enabled channels shall not automatically imply simultaneous delivery.                                   |
| NOT-007 | The system shall support an ordered channel priority/fallback policy.                                            |
| NOT-008 | The default Phase 1 channel order shall be Telegram → SMS → Voice.                                               |
| NOT-009 | The channel order shall be configurable by policy rather than hardcoded into business logic.                     |
| NOT-010 | The system shall distinguish notification retry from channel fallback.                                           |
| NOT-011 | The system shall classify notification failures as retryable or non-retryable where possible.                    |
| NOT-012 | Retryable failures shall be eligible for retry.                                                                  |
| NOT-013 | The system shall limit the maximum number of retry attempts.                                                     |
| NOT-014 | Retry attempts shall use a backoff strategy.                                                                     |
| NOT-015 | The system shall support delivery-status-based fallback where reliable delivery information is available.        |
| NOT-016 | The system shall support response-timeout-based fallback where enabled by notification policy.                   |
| NOT-017 | Technical failures and patient-response timeouts shall be treated as distinct fallback triggers.                 |
| NOT-018 | The system shall use a configurable response window.                                                             |
| NOT-019 | The default response window shall initially be 2 hours.                                                          |
| NOT-020 | The system shall not wait for the full response window when a failure policy requires earlier retry or fallback. |
| NOT-021 | The system shall skip channels for which the patient is not eligible or configured.                              |
| NOT-022 | Multiple notification attempts for one reminder shall remain associated with the same reminder occurrence.       |
| NOT-023 | Notification processing shall be idempotent.                                                                     |
| NOT-024 | Concurrent workers shall not unintentionally process the same notification action more than once.                |
| NOT-025 | Notification provider integrations shall be isolated behind provider/adapter interfaces.                         |
| NOT-026 | The core notification business logic shall not depend on a specific external provider.                           |
| NOT-027 | SMS and Voice providers shall be replaceable without rewriting core notification orchestration logic.            |
| NOT-028 | Development shall support mock SMS and Voice providers.                                                          |
| NOT-029 | Mock providers shall support realistic success and failure scenarios for testing.                                |
| NOT-030 | Production provider credentials shall be managed securely.                                                       |
| NOT-031 | Provider failures shall not cause the entire notification system to fail.                                        |
| NOT-032 | The system shall preserve notification attempt and status history.                                               |
| NOT-033 | The system shall record relevant notification events for auditing and troubleshooting.                           |
| NOT-034 | The system shall support secure provider webhook processing where applicable.                                    |
| NOT-035 | Duplicate webhook events shall not cause duplicate state transitions or actions.                                 |
| NOT-036 | Notification logs shall avoid unnecessary exposure of sensitive patient/clinical information.                    |
| NOT-037 | The notification system shall expose sufficient structured information for production monitoring and alerting.   |
| NOT-038 | A notification delivery failure shall not by itself be interpreted as patient non-adherence.                     |

---

# One important refinement before we lock Step 10

There is one decision I want to make explicit because it affects the architecture.

## Should fallback depend on "patient response"?

I recommend **yes, but only according to the notification policy**.

For example:

### Policy A — Response-based fallback

```text
08:00 Telegram
       ↓
Delivered
       ↓
Wait 2 hours
       ↓
No response
       ↓
SMS
       ↓
Wait
       ↓
Voice
```

### Policy B — Delivery-based fallback

```text
08:00 Telegram
       ↓
Not delivered
       ↓
SMS
```

But:

```text
Telegram → Delivered
```

means we don't immediately send SMS.

### Policy C — Primary channel only

```text
Telegram
   ↓
Delivered
   ↓
No response
   ↓
No fallback
   ↓
NO_RESPONSE
```

This flexibility is useful because **not every patient needs the same escalation strategy**.

---

# My recommendation for Phase 1

I recommend that we support a **default policy**:

```text
Telegram
   ↓
If technical/delivery failure → retry/fallback according to policy
   ↓
If delivered but no patient response within response window
   ↓
SMS
   ↓
If SMS fails or no response according to policy
   ↓
Voice
```

But the **exact retry counts, timing, and fallback conditions should be configurable**, not hardcoded.

That gives us:

**Simple default behavior for Phase 1 + production flexibility later.**

---

## Step 10 status

I recommend that we **do not yet lock Step 10 until you approve the notification policy**, particularly:

1. ✅ Multiple channels are allowed.
2. ✅ Default order = **Telegram → SMS → Voice**.
3. ✅ Channels are sequential/fallback by default, **not simultaneous**.
4. ✅ Technical failure can trigger retry/fallback.
5. ✅ Delivered-but-no-response can trigger fallback after the configured response window.
6. ✅ Default response window = **2 hours**, configurable.
7. ✅ Retry and fallback are separate concepts.
8. ✅ Provider-specific logic stays behind adapters.
9. ✅ Mock providers can be replaced by production providers later.
10. ✅ One reminder can have multiple notification attempts, but they all belong to the same reminder occurrence.

**Once you accept these, Step 10 is locked and we move to Step 11 — Telegram Requirements**, where we will define the Telegram Bot behavior in detail, including patient linking, messages, `Taken/Not Taken` buttons, callbacks, security, and failure handling.
