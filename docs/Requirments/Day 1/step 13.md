Great. **Step 12 — SMS Requirements is now locked.** ✅

We can now move to **Step 13 — Voice Requirements**.

This is an important step because Voice is different from Telegram and SMS: it is a **real-time telephone call**, and the patient interacts with it using the phone keypad.

# Step 13 — Voice Requirements

## 13.1 Purpose of Voice

Voice is the third communication channel in Phase 1.

The basic architecture is:

```text
Reminder
   ↓
Notification Orchestrator
   ↓
Voice Adapter
   ↓
Voice Provider
   ↓
Telephone Network
   ↓
Patient's Phone Rings
   ↓
Automated Audio
   ↓
Patient presses keypad
   ↓
Voice Provider
   ↓
Backend
   ↓
Adherence
```

For example:

> "This is your medication reminder. Please take Amoxicillin 500 mg now. Press 1 if you have taken the medication. Press 2 if you have not taken the medication."

Then:

```text
Patient presses 1
        ↓
TAKEN
```

or:

```text
Patient presses 2
        ↓
NOT_TAKEN
```

---

# 13.2 Why we chose keypad interaction

Earlier we discussed two approaches:

### Option A — Automated audio only

```text
Call
 ↓
Play message
 ↓
End call
```

This is easy to implement, but it gives us **no direct patient response**.

### Option B — Audio + keypad

```text
Call
 ↓
Play reminder
 ↓
"Press 1 if Taken"
"Press 2 if Not Taken"
 ↓
Patient presses key
 ↓
Backend receives response
```

I recommended **Option B**, and we accepted it.

For our patient-adherence system, this is much more useful because the call becomes an actual adherence interaction rather than just an alarm.

---

# 13.3 Voice provider abstraction

Just like SMS:

```text id="l9gd42"
Notification Orchestrator
          ↓
      Voice Adapter
          ↓
   Voice Provider Interface
          ↓
      ┌───────────────┐
      ↓               ↓
 MockVoice        RealVoice
 Provider          Provider
```

During development:

```text
MockVoiceProvider
```

During production:

```text
ProductionVoiceProvider
```

The rest of the application doesn't need to change.

---

# 13.4 Development with Mock Voice Provider

We can develop the entire Voice workflow without making real phone calls.

For example:

```text id="m6l0k4"
Backend
   ↓
Mock Voice Provider
   ↓
Simulated call
   ↓
Simulated:
  ANSWERED
  NO_ANSWER
  BUSY
  FAILED
  PRESS_1
  PRESS_2
```

This allows us to test:

* scheduling,
* retries,
* escalation,
* call states,
* keypad responses,
* adherence,
* failure handling,

without paying for calls.

This is exactly what we want for the first development stage.

---

# 13.5 Production Voice provider

Later:

```text id="x8wz4y"
MockVoiceProvider
       ↓
ProductionVoiceProvider
       ↓
Telephone network
       ↓
Patient
```

The core application remains unchanged.

The provider selection will be made later based on:

* Ethiopia availability
* pricing
* outgoing call support
* DTMF/keypad support
* voice recording/TTS capabilities
* webhooks
* delivery/call status
* reliability
* regulatory requirements

---

# 13.6 Voice call initiation

The Voice provider should receive a request containing the minimum required information.

Conceptually:

```text id="t8tq2w"
Call patient
   ↓
Phone number
   +
Audio/message information
   +
Reminder reference
```

The Voice Adapter handles provider-specific formatting.

The Notification Orchestrator decides **when** the call should happen.

---

# 13.7 Voice should not own reminder scheduling

The Voice provider should not know:

> "This patient needs medicine at 8:00."

The reminder system already knows that.

Instead:

```text id="8j6ah9"
Reminder Scheduler
       ↓
Notification Orchestrator
       ↓
"Send Voice notification now"
       ↓
Voice Adapter
```

This keeps responsibilities separate.

---

# 13.8 Voice message content

The call should provide enough information for the patient to identify the medication.

For example:

> "Medication reminder. It is time to take Amoxicillin 500 milligrams. Please take one tablet."

Then:

> "Press 1 if you have taken your medication. Press 2 if you have not taken it."

The exact wording will be handled through our message/template system.

---

# 13.9 Preferred language

Voice must respect the patient's:

```text
Preferred language
```

Phase 1:

```text
English
Amharic
Afaan Oromoo
```

Example:

```text id="1r7a9g"
Patient
Preferred language = Amharic
        ↓
Amharic Voice message
```

The same principle applies to Telegram and SMS.

---

# 13.10 How audio is generated

There are two possible approaches.

### Option A — Pre-recorded audio

We prepare audio files:

```text
English
Amharic
Afaan Oromoo
```

and play them during the call.

### Option B — Text-to-Speech

Our system generates speech from text.

```text
Reminder text
     ↓
TTS engine
     ↓
Audio
     ↓
Voice provider
```

For Phase 1, I recommend we design the system so that the **Voice Adapter can support both**, but initially use the simplest reliable approach available for development.

We should **not tightly couple our application to one TTS provider**.

---

# 13.11 Dynamic medication information

A medication reminder contains dynamic information:

```text
Medication name
Dose
Instructions
Time
```

Therefore, generating one audio file for every possible reminder is impractical.

For example:

```text
"Take Amoxicillin 500 mg"
"Take Paracetamol 500 mg"
"Take Metformin 850 mg"
```

would require many combinations.

A production-ready design should support dynamic content.

Conceptually:

```text
Patient data
     ↓
Reminder template
     ↓
Localized text
     ↓
TTS / Audio generation
     ↓
Voice call
```

---

# 13.12 Audio generation should not happen unnecessarily

We don't want to generate the same audio repeatedly if it can safely be reused.

For example, if the exact same localized message is used repeatedly:

```text
Same text
   ↓
Existing audio
   ↓
Reuse
```

rather than:

```text
Generate audio
Generate audio
Generate audio
...
```

This can reduce cost and latency.

The exact caching strategy can be determined during implementation.

---

# 13.13 Voice call states

We need an internal call state model independent of the provider.

For example:

```text id="0m0p3k"
QUEUED
INITIATED
RINGING
ANSWERED
NO_ANSWER
BUSY
FAILED
COMPLETED
```

The exact states will be finalized during database/API design.

The important principle is:

> Provider-specific call statuses should be mapped into our own internal status model.

---

# 13.14 Answered does not mean medication taken

This is extremely important.

Suppose:

```text id="xq8hki"
Call → ANSWERED
```

That does **not** mean:

```text
Medication → TAKEN
```

The patient must explicitly press the appropriate keypad button.

Therefore:

```text id="xvq36y"
ANSWERED
    ≠
TAKEN
```

---

# 13.15 Patient presses 1

Example:

```text id="k0s3j4"
Call answered
   ↓
Audio plays
   ↓
Patient presses 1
   ↓
Backend receives DTMF = 1
   ↓
TAKEN
```

The response is then passed to the adherence system.

---

# 13.16 Patient presses 2

```text id="9y8s7f"
Call answered
   ↓
Audio plays
   ↓
Patient presses 2
   ↓
DTMF = 2
   ↓
NOT_TAKEN
```

Again, Voice itself does not determine adherence. It simply communicates the patient's selection.

---

# 13.17 Patient presses another key

Suppose the patient presses:

```text id="44m0ba"
3
```

or:

```text
9
```

The system should **not interpret that as Taken or Not Taken**.

Instead:

```text id="6t4snp"
Invalid DTMF
   ↓
Handle according to Voice interaction policy
```

For example, we can replay the instruction:

> "Please press 1 if Taken or 2 if Not Taken."

We should define a maximum number of retries to prevent the call from continuing indefinitely.

---

# 13.18 Patient does not press anything

Example:

```text id="w6n5p7"
Call answered
   ↓
Audio played
   ↓
No keypad response
```

After a configured timeout:

```text id="n1u1t7"
NO_RESPONSE
```

This is different from:

```text
NO_ANSWER
```

because the patient answered the phone but did not provide a response.

---

# 13.19 Patient doesn't answer

```text id="5r6nq5"
Call initiated
   ↓
Phone rings
   ↓
No answer
```

This should be recorded as something like:

```text
NO_ANSWER
```

The Notification Orchestrator then determines whether to:

* retry the call,
* wait until the next escalation point,
* or finish the notification sequence.

Voice itself should not make that decision.

---

# 13.20 Patient's phone is busy

```text id="k6n2fd"
Call
 ↓
BUSY
```

This is different from:

```text
NO_ANSWER
```

because the provider/network knows the call encountered a busy condition.

The system should preserve that distinction because it can help with:

* retry policy,
* monitoring,
* troubleshooting,
* audit history.

---

# 13.21 Patient hangs up

Possible scenario:

```text id="c1a6h9"
Call answered
 ↓
Audio starts
 ↓
Patient hangs up
```

The system records the call outcome.

It should **not assume TAKEN**.

Unless the patient explicitly pressed the appropriate keypad key:

```text
HANGUP ≠ TAKEN
```

---

# 13.22 Patient presses 1 and then hangs up

This is different:

```text id="k9u3r7"
Call
 ↓
Patient presses 1
 ↓
Backend receives DTMF 1
 ↓
TAKEN
 ↓
Patient hangs up
```

The valid response has already been received.

So the system should not interpret the later hang-up as a contradictory response.

---

# 13.23 Duplicate DTMF events

As with Telegram and SMS, we should assume external providers can produce duplicate events.

For example:

```text id="m0n8vi"
DTMF 1
DTMF 1
```

The same logical response should not produce duplicate adherence transitions.

Idempotency applies here too.

---

# 13.24 Voice webhooks

The provider will generally communicate call events back to our backend.

For example:

```text id="s3h1m6"
Voice Provider
      ↓
Webhook
      ↓
Backend
```

Events may include:

```text
CALL_INITIATED
CALL_ANSWERED
CALL_FAILED
CALL_COMPLETED
DTMF_RECEIVED
```

The exact events depend on the provider.

Our Voice Adapter translates them into our internal event model.

---

# 13.25 Voice webhook security

Incoming provider webhooks must be:

* authenticated/verified,
* validated,
* idempotent,
* logged appropriately,
* protected against unauthorized requests.

We should never trust arbitrary HTTP requests claiming:

> "Patient pressed 1."

The backend must verify that the event belongs to a legitimate call and reminder.

---

# 13.26 Voice and sensitive information

The call contains clinical information.

Therefore:

> We should minimize the amount of sensitive information spoken aloud.

For example, if possible, we should avoid saying unnecessary diagnosis information.

The call should focus on the medication reminder.

Also, our logs should not unnecessarily store complete audio or sensitive call content.

---

# 13.27 Voice recording

For Phase 1, I recommend:

> **Do not record patient calls unless there is a specific requirement for recording.**

Recording introduces additional:

* storage requirements,
* privacy considerations,
* security requirements,
* compliance considerations,
* operational complexity.

We only need the call interaction and outcome for the initial product.

---

# 13.28 Voice fallback

Example:

```text id="f8f5fr"
Telegram
   ↓
SMS
   ↓
Voice
```

If Voice is selected:

```text id="6d3v2j"
Notification Orchestrator
       ↓
Voice Adapter
       ↓
Make call
```

If Voice fails:

```text id="g9h2r7"
Voice FAILED
       ↓
Notification Orchestrator
       ↓
No additional channel
       ↓
Record failure
```

because Voice is currently our final Phase 1 channel.

The exact behavior will be determined by the escalation policy.

---

# 13.29 Voice and escalation timing

We already decided in Step 10:

> Voice timing must be based on the reminder's predefined escalation schedule.

Not:

```text
Telegram
 ↓
wait 2 hours
 ↓
SMS
 ↓
wait 2 hours
 ↓
Voice
```

Instead:

```text
Medication due = 08:00

08:00 → Telegram
08:10 → SMS
08:20 → Voice
```

These are illustrative times.

The actual intervals will be configurable according to the finalized requirements.

---

# 13.30 Voice and adherence response window

Again, these are separate concepts.

Example:

```text
08:00 → Medication due
08:00 → Telegram
08:10 → SMS
08:20 → Voice
08:20 → Patient presses 2
09:00 → Adherence response window closes
```

The Voice call happens at 08:20.

The adherence response window can remain open until the defined deadline.

---

# 13.31 Voice provider abstraction and future replacement

Our architecture should look like:

```text id="s6xqgp"
                VoiceService
                    │
                    ▼
             VoiceProvider
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
 MockVoiceProvider       RealVoiceProvider
```

So when we move from development to production:

```text
MockVoiceProvider
        ↓
Ethiopia-capable production provider
```

we shouldn't have to rewrite:

* reminders,
* scheduling,
* adherence,
* doctors,
* patients,
* notification orchestration.

Only the provider implementation/configuration changes.

---

# 13.32 Proposed Voice Requirements

| ID      | Requirement                                                                                                                                                  |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| VOI-001 | Phase 1 shall support automated Voice calls as a patient notification channel.                                                                               |
| VOI-002 | Voice shall support automated audio reminders.                                                                                                               |
| VOI-003 | Voice shall support patient keypad/DTMF interaction.                                                                                                         |
| VOI-004 | Phase 1 shall use keypad `1` for Taken.                                                                                                                      |
| VOI-005 | Phase 1 shall use keypad `2` for Not Taken.                                                                                                                  |
| VOI-006 | Answering a call shall not automatically be interpreted as Taken.                                                                                            |
| VOI-007 | Hanging up shall not automatically be interpreted as Taken.                                                                                                  |
| VOI-008 | An explicit valid DTMF response shall be required for Taken/Not Taken classification.                                                                        |
| VOI-009 | Invalid keypad responses shall not automatically be classified as adherence responses.                                                                       |
| VOI-010 | The system shall handle cases where the patient does not provide a keypad response.                                                                          |
| VOI-011 | The system shall distinguish NO_ANSWER from an answered call with no response.                                                                               |
| VOI-012 | The system shall distinguish BUSY from NO_ANSWER where the provider supports that distinction.                                                               |
| VOI-013 | The system shall record call outcomes using an internal call-status model.                                                                                   |
| VOI-014 | Provider-specific call statuses shall be mapped to internal application statuses.                                                                            |
| VOI-015 | Voice shall be accessed through a Voice Provider abstraction.                                                                                                |
| VOI-016 | The system shall support a MockVoiceProvider for development/testing.                                                                                        |
| VOI-017 | The mock provider shall simulate call outcomes and DTMF responses.                                                                                           |
| VOI-018 | The production Voice provider shall be replaceable without changing core notification business logic.                                                        |
| VOI-019 | Voice reminders shall support English, Amharic, and Afaan Oromoo in Phase 1.                                                                                 |
| VOI-020 | Voice language shall follow the patient's configured preferred language.                                                                                     |
| VOI-021 | Voice messages shall use the centralized notification/message-template system.                                                                               |
| VOI-022 | The Voice system shall support dynamic medication reminder content.                                                                                          |
| VOI-023 | The Voice system shall be designed to support appropriate audio generation/TTS mechanisms without tightly coupling core business logic to a single provider. |
| VOI-024 | Audio generation/reuse should avoid unnecessary repeated generation where safely possible.                                                                   |
| VOI-025 | The Voice provider shall receive only the information required to initiate and manage the call.                                                              |
| VOI-026 | Voice-specific provider logic shall remain inside the Voice Adapter/provider layer.                                                                          |
| VOI-027 | The Notification Orchestrator shall determine when a Voice notification is initiated.                                                                        |
| VOI-028 | Voice shall follow the escalation timing defined in the Notification Requirements.                                                                           |
| VOI-029 | Voice shall not independently determine whether SMS or another channel should be used.                                                                       |
| VOI-030 | Voice provider failures shall be returned to the Notification Orchestrator for retry/fallback handling.                                                      |
| VOI-031 | Voice webhook events shall be authenticated/verified using provider-supported mechanisms where available.                                                    |
| VOI-032 | Voice webhook events shall be validated against the corresponding call and reminder.                                                                         |
| VOI-033 | Duplicate Voice webhook events shall not cause duplicate state transitions.                                                                                  |
| VOI-034 | Duplicate DTMF events shall not create duplicate adherence responses.                                                                                        |
| VOI-035 | A valid DTMF response shall be passed to the adherence system.                                                                                               |
| VOI-036 | A valid Taken response shall stop further notification escalation where applicable.                                                                          |
| VOI-037 | A valid Not Taken response shall stop further notification escalation where applicable.                                                                      |
| VOI-038 | Voice responses received after reminder expiration or through closed channels shall be handled according to the late-response and closed-channel policies defined in Step 14B. |
| VOI-039 | Voice calls shall not be recorded by default in Phase 1 unless a specific requirement is introduced.                                                         |
| VOI-040 | The system shall minimize unnecessary exposure of sensitive patient/clinical information in Voice messages.                                                  |
| VOI-041 | Voice logs shall avoid unnecessary storage of sensitive clinical information.                                                                                |
| VOI-042 | Voice provider credentials shall be stored securely and never committed to source control.                                                                   |
| VOI-043 | The system shall distinguish call delivery/outcome from patient adherence.                                                                                   |
| VOI-044 | A successful/answered call shall not itself create a TAKEN adherence event.                                                                                  |
| VOI-045 | Voice interaction shall support a configurable response timeout for keypad input.                                                                            |
| VOI-046 | The system shall limit repeated invalid/no-response prompts to prevent excessively long calls.                                                               |
| VOI-047 | Voice notification attempts shall remain associated with their originating reminder occurrence.                                                              |
| VOI-048 | The Voice implementation shall be testable without making real telephone calls.                                                                              |
| VOI-049 | Multiple reminder occurrences due at the same time may be combined into one Voice call.                                                                      |
| VOI-050 | When simultaneous reminders are combined in one Voice call, each medication/reminder shall be presented separately with independent DTMF response collection. |
| VOI-051 | Each reminder occurrence shall maintain its own independent adherence decision regardless of Voice call grouping.                                            |
| VOI-052 | Communication grouping in Voice shall not merge underlying reminder or adherence records.                                                                    |
| VOI-053 | Responses received through a closed Voice channel (after escalation has moved or completed) shall be recorded as response events but shall not modify the reminder's adherence state. |
| VOI-054 | A Voice call attempt shall capture the patient's phone number at the time the attempt is created.                                                            |
| VOI-055 | Future reminder occurrences shall use the patient's current valid phone number configuration.                                                                |
| VOI-056 | Changing a patient's phone number after a Voice call attempt has been created shall not silently rewrite the destination of that already-created attempt.    |
| VOI-057 | The same phone number may appear on multiple patient records in Phase 1.                                                                                     |
| VOI-058 | The system shall not enforce global phone number uniqueness across patients.                                                                                 |
| VOI-059 | Patient identity shall be determined by internal Patient ID, not by phone number.                                                                            |
| VOI-060 | Voice webhook endpoints shall implement provider-supported authenticity verification mechanisms.                                                             |
| VOI-061 | State-changing Voice webhooks (DTMF, call outcome) that cannot be authenticated sufficiently shall not be trusted for adherence or state-changing operations. |
| VOI-062 | Initial Phase 1 production deployment may launch with Voice inactive while Telegram is active.                                                               |
| VOI-063 | Voice shall remain architecturally prepared (provider abstraction exists, orchestrator supports Voice) such that Voice can be activated in production without redesigning core reminder/notification logic. |
| VOI-064 | Voice call failure shall not automatically be interpreted as patient non-adherence.                                                                          |
| VOI-065 | Voice call status shall remain distinct from patient adherence status, consistent with the requirement that call delivery/outcome does not equal adherence.  |

---

# Step 13 Architecture

At this point, our three channels are becoming symmetrical:

```text
                         REMINDER
                            │
                            ▼
                NOTIFICATION ORCHESTRATOR
                    /        |        \
                   /         |         \
                  ▼          ▼          ▼
             Telegram       SMS        Voice
              Adapter      Adapter     Adapter
                  │          │           │
                  ▼          ▼           ▼
              Telegram     SMS         Voice
                API       Provider     Provider
                  │          │           │
                  ▼          ▼           ▼
              Patient     Patient     Patient
                  │          │           │
                  ▼          ▼           ▼
              Response    Response    DTMF
                  \          |           /
                   \         |          /
                    └────────┼─────────┘
                             ▼
                         Adherence
```

This is the architecture I recommend we continue with.

## One particularly important production principle

The **three channels should be treated as interchangeable communication mechanisms**, while the business logic remains centralized.

That means:

```text
Telegram ≠ Reminder logic
SMS ≠ Reminder logic
Voice ≠ Reminder logic
```

Instead:

```text
Reminder + Notification Policy
             ↓
    Notification Orchestrator
             ↓
       Communication
```

This will make the system much easier to maintain and replace providers later.

---

### Step 13 status

I recommend that **Step 13 is now ready for your review/acceptance**.

Once you accept it, we will move to **Step 14 — Adherence Requirements**.

That step is especially important because it will formally define exactly what happens with:

* `TAKEN`
* `NOT_TAKEN`
* `NO_RESPONSE`
* late responses
* duplicate responses
* changing a response
* responses after escalation
* responses after the adherence window
* how adherence history is calculated
* what doctors see in the patient adherence history

After Step 14, we will have completed the requirements for **Reminder → Notification → Telegram → SMS → Voice → Adherence**, and we can move toward the next major phase: **turning the approved requirements into the technical specification/API and database design**.


---

## Cross-Document Consistency Requirements

### Consistency with Step 10B — Notification/Escalation

**VOI-CONSISTENCY-001**

Voice requirements SHALL be consistent with Step 10B notification requirements, specifically:
- NOTIF-INVARIANT-001: Communication grouping preserves independent adherence (VOI-049 through VOI-052)
- NOTIF-INVARIANT-002: Provider failure never becomes NOT_TAKEN (VOI-064)
- NOTIF-INVARIANT-003: Delivery does not equal adherence (VOI-043, VOI-044, VOI-065)
- NOTIF-INVARIANT-004: One active response channel (VOI-053)
- NOTIF-REQ-003: Voice availability requires valid phone number (shared with SMS)
- NOTIF-REQ-004: Phone number cardinality (VOI-057, VOI-058, VOI-059)
- NOTIF-REQ-005: In-flight contact change behavior (VOI-054, VOI-055, VOI-056)
- NOTIF-REQ-015: Voice simultaneous reminders (VOI-049 through VOI-052)
- NOTIF-REQ-016: Provider abstraction (VOI-015, VOI-018, VOI-026)
- NOTIF-REQ-017: Notification Orchestrator responsibility (VOI-027, VOI-028, VOI-029, VOI-030)
- NOTIF-REQ-018: Production channel activation (VOI-062, VOI-063)

### Consistency with Step 12 — SMS

**VOI-CONSISTENCY-002**

Voice SHALL be consistent with SMS where both channels share phone-based delivery:
- Phone number identity (VOI-057, VOI-058, VOI-059 ↔ SMS-054, SMS-055, SMS-056)
- Phone number validation and normalization principles (VOI-054 ↔ SMS-009, SMS-051)
- Provider abstraction (VOI-015 ↔ SMS-002, SMS-003)
- Mock provider development (VOI-016, VOI-017, VOI-048 ↔ SMS-004, SMS-005)
- Production provider independence (VOI-018 ↔ SMS-006, SMS-040)
- Webhook security (VOI-031, VOI-060, VOI-061 ↔ SMS-036, SMS-057, SMS-058)
- Localization (VOI-019, VOI-020, VOI-021 ↔ SMS-010, SMS-011, SMS-012)
- In-flight contact changes (VOI-054-056 ↔ SMS-051-053)
- Production activation state (VOI-062, VOI-063 ↔ SMS-059, SMS-060)

### Consistency with Step 14B — Adherence

**VOI-CONSISTENCY-003**

Voice response handling SHALL be consistent with Step 14B adherence requirements, specifically:
- ADH-PRINCIPLE-001: One reminder, one adherence decision (VOI-051, VOI-052)
- ADH-REQ-002: Closed channels cannot change adherence (VOI-053)
- ADH-REQ-007: Voice valid responses are DTMF "1"=TAKEN, "2"=NOT_TAKEN (VOI-004, VOI-005, VOI-008, VOI-035)
- ADH-REQ-008: Unrecognized responses do not change adherence (VOI-009, VOI-010)
- ADH-REQ-011: Delivery ≠ adherence (VOI-006, VOI-007, VOI-043, VOI-044, VOI-065)
- ADH-REQ-012: Provider failure ≠ NOT_TAKEN (VOI-064)
- ADH-REQ-013-015: Response timing and windows (VOI-038, VOI-045)

### Consistency with Step 19 — Security

**VOI-CONSISTENCY-004**

Voice security requirements SHALL be consistent with Step 19 security requirements, specifically:
- Webhook authenticity verification (VOI-031, VOI-060, VOI-061)
- Credential protection (VOI-042)
- Sensitive data minimization (VOI-040, VOI-041)
- Idempotency and replay protection (VOI-033, VOI-034)

---

## Requirement Ownership

**Step 13 owns:**
- Voice-specific channel behavior
- Automated outbound call requirements
- DTMF/keypad response handling
- Voice provider abstraction requirements
- Call status mapping
- Voice webhook processing
- Audio generation/TTS requirements
- Voice-specific localization requirements
- Call interaction flow

**Step 13 does NOT own:**
- Reminder generation (Step 9)
- Notification escalation logic (Step 10)
- Adherence decision rules (Step 14)
- Audit logging (Step 15)
- Security implementation details (Step 19)
- Configuration/settings UI (Step 17)
- Patient clinical records (Step 6)
- Phone number storage/management (Step 6, shared validation with Step 12)

---

## Implementation Notes

- Exact Voice provider API selection is an implementation detail
- TTS engine/service selection is an implementation detail subject to localization quality requirements
- Audio file formats and encoding are implementation details
- Webhook endpoint URL structure is an implementation detail
- Phone number normalization format (E.164 or other) is an implementation detail subject to provider requirements
- DTMF timeout durations are configurable, not hard-coded
- Maximum invalid-response retry count is configurable
- Call recording is explicitly excluded from Phase 1 unless requirement changes
- Audio caching/reuse strategies are implementation optimizations
- Voice provider credentials management follows Step 19 secret management requirements
- Ethiopia-specific provider selection criteria will be evaluated before production launch
- Simultaneous reminder presentation order and prompting is an implementation/template detail

---

## Unresolved Human Decisions

**NONE IDENTIFIED** for Step 13.

Late-response policy details are owned by Step 14B.

Exact DTMF timeout values are correctly deferred to configuration.

Audio generation/TTS provider selection is correctly deferred to implementation.
