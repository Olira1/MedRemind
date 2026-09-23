Yes — these are exactly the right questions to resolve **before we lock Step 14**. The key is to separate **channel response handling** from the **single reminder-level adherence decision**.

## 1. Telegram + SMS responses: what is the ideal design?

Your example is very good:

> **08:00** → Telegram
> **08:10** → SMS
> **08:12** → Patient presses Telegram “Taken”

You are essentially asking:

> Should Telegram remain capable of accepting a response after its escalation window has ended?

### My recommendation: use a **channel response window**

I would recommend that we **close a channel's response capability when the system moves to the next channel**.

So:

| Time  | Event                            | Telegram                    | SMS        |
| ----- | -------------------------------- | --------------------------- | ---------- |
| 08:00 | Telegram reminder sent           | **OPEN**                    | Not active |
| 08:10 | Telegram window closes; SMS sent | **CLOSED**                  | **OPEN**   |
| 08:12 | Patient tries Telegram "Taken"   | **Rejected as late/closed** | OPEN       |
| 08:15 | Patient responds by SMS "1"      | CLOSED                      | **TAKEN**  |
| 08:20 | Voice stage                      | Not needed                  | Not needed |

This is cleaner than allowing all previous channels to remain open.

### Why I recommend this

It gives us a very clear rule:

> **At any moment, there is one active response channel for the reminder.**

That eliminates most cross-channel conflicts.

For example, without closing Telegram:

> 08:12 Telegram → Taken
> 08:14 SMS → Not Taken

Now we have a genuine conflict.

The system has to decide:

* Which response wins?
* The first one?
* The latest one?
* The highest-priority channel?
* Should the doctor manually resolve it?

None of those is ideal for a medication-adherence system.

With channel windows:

> 08:00–08:10 → Telegram accepts response
> 08:10–08:20 → SMS accepts response
> 08:20 onward → Voice accepts response

Only the **active channel** can create the current adherence decision.

---

## But should the Telegram response at 08:12 disappear?

**No.**

This is an important distinction.

The system should record that:

> Patient attempted to respond through Telegram at 08:12, but Telegram's response window had already closed.

That should be stored as a **response event**, but it should **not change the reminder's adherence state**.

For example:

```text
Reminder: 08:00 medication

08:00 Telegram sent
08:10 Telegram response window closed
08:10 SMS sent
08:12 Telegram "Taken" received
      → rejected as late/closed channel
08:15 SMS "1" received
      → adherence = TAKEN
```

The doctor's page could show something like:

**Adherence:** TAKEN
**Recorded via:** SMS
**Response time:** 08:15

And, if the doctor opens the reminder's detailed history:

**Notification history**

* 08:00 Telegram — Sent
* 08:10 Telegram — Response window closed
* 08:10 SMS — Sent
* 08:12 Telegram — Late response received / not accepted
* 08:15 SMS — Taken response accepted

So we don't lose information, but we also don't let old channels interfere with the current decision.

---

# Should both Telegram and SMS responses appear on the doctor's page?

**Yes, but not as two separate adherence decisions.**

This is the important UI distinction.

The doctor should see **one adherence result per medication reminder**, for example:

> **8:00 AM Medication**
> **Status: TAKEN**
> Response channel: SMS
> Response time: 08:15

Then the doctor can expand **Response/Notification History**:

> Telegram — Sent 08:00
> Telegram — Window closed 08:10
> Telegram — Late response received 08:12 — Not accepted
> SMS — Sent 08:10
> SMS — Taken received 08:15 — Accepted

So:

**One reminder → one adherence outcome**

but

**One reminder → many notification/response events**

That's the model I recommend.

---

# What about conflicts?

With channel windows, we dramatically reduce conflicts.

### Normal case

```text
08:00 Telegram → patient responds Taken
```

Result:

```text
TAKEN
```

SMS/Voice escalation stops.

---

### Patient doesn't respond

```text
08:00 Telegram → no response
08:10 Telegram closes
08:10 SMS → no response
08:20 SMS closes
08:20 Voice
```

Eventually:

```text
NO_RESPONSE
```

assuming the overall adherence response window closes without a valid response.

---

### Late response

```text
08:00 Telegram
08:10 Telegram closes
08:12 Telegram Taken
```

We record:

```text
Late Telegram response = Taken
Accepted for? NO
```

It should not change the active reminder decision if SMS is already the active channel.

But the event remains available for audit/history.

---

### What if the patient responds on the active SMS channel?

```text
08:10 SMS
08:15 SMS → 1
```

Then:

```text
TAKEN
```

and future escalation stops.

---

## One important refinement

I would **not** define the channel window merely as "10 minutes" in the requirements yet.

Instead define:

> Each notification channel has a configurable response window. When the window expires, that channel can no longer produce an accepted adherence response for that reminder.

Then later we can configure:

```text
Telegram response window = 10 minutes
SMS response window      = 10 minutes
Voice response window    = call duration / configured timeout
```

The actual values should be configurable rather than hard-coded.

---

# The overall model I recommend

I think this is the cleanest Phase 1 architecture:

```text
                 REMINDER
                    │
                    ▼
          Notification Orchestrator
                    │
              ┌─────┴─────┐
              ▼           ▼
          Telegram       Window
          08:00          10 min
              │
        ┌─────┴─────┐
        │           │
     Response     No response
        │           │
     TAKEN/       08:10
    NOT_TAKEN       │
                    ▼
                   SMS
                 10 min
                    │
              ┌─────┴─────┐
              │           │
           Response     No response
              │           │
              ▼          08:20
          adherence        │
                           ▼
                         Voice
```

And underneath that:

```text
              ONE REMINDER
                   │
                   ▼
          ONE ADHERENCE STATE
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
     TAKEN     NOT_TAKEN   NO_RESPONSE
```

While separately:

```text
ONE REMINDER
     │
     ├── Telegram events
     ├── SMS events
     ├── Voice events
     └── Response events
```

This separation is extremely important.

---

# 2. What if the patient sends random text?

Excellent question.

We should **not** interpret arbitrary text as adherence.

For SMS, for example:

```text
1 → TAKEN
2 → NOT_TAKEN
```

Anything else:

```text
"hello"
"yes"
"I took it"
"not today"
"okay"
"please call me"
"123"
```

should be classified as:

> **UNRECOGNIZED_RESPONSE**

and **must not change adherence**.

For example:

```text
08:10 SMS reminder sent

08:12 Patient: "yes"

→ UNRECOGNIZED_RESPONSE
→ adherence remains PENDING
→ system continues according to escalation policy
```

We should **not automatically interpret `"yes"` as TAKEN**.

That's especially important for a clinical system.

---

## Should the patient receive a reply?

I recommend yes, but keep it simple.

For example:

> "We couldn't understand your response. Please reply 1 for Taken or 2 for Not Taken."

And importantly:

**This is not a new reminder and does not reset the escalation clock.**

So if the SMS window ends at 08:20, sending `"hello"` at 08:18 doesn't give the patient another 10 minutes.

---

# What about Telegram?

Same principle.

Telegram has structured buttons:

**[ Taken ] [ Not Taken ]**

So normally the patient won't send arbitrary text.

But they might type:

> "I took it"

That should also be:

```text
UNRECOGNIZED_RESPONSE
```

rather than automatically converting it to `TAKEN`.

We can send:

> "Please use the Taken or Not Taken button to respond to this reminder."

---

# One more important distinction

I recommend that we **do not make `NO_RESPONSE` a patient response**.

It is a **system-generated final adherence state**.

So:

### Patient-generated

```text
TAKEN
NOT_TAKEN
```

### System-generated

```text
NO_RESPONSE
```

And:

```text
UNRECOGNIZED_RESPONSE
LATE_RESPONSE
CLOSED_CHANNEL_RESPONSE
```

are **events/metadata**, not adherence outcomes.

This gives us a much cleaner data model.

---

# My recommended decision for Step 14

I recommend we change the earlier Step 14 design slightly:

### **A. One active response channel at a time**

When escalation moves from Telegram → SMS → Voice, the previous channel's response window closes.

### **B. Closed channels cannot change adherence**

Responses arriving through a closed channel are recorded as late/closed-channel events but don't modify the reminder's adherence state.

### **C. One reminder = one adherence decision**

Telegram, SMS, and Voice do not create separate adherence records.

### **D. Preserve all response events**

The doctor's detailed history can show what happened on every channel.

### **E. Only recognized responses affect adherence**

* Telegram Taken → `TAKEN`
* Telegram Not Taken → `NOT_TAKEN`
* SMS `1` → `TAKEN`
* SMS `2` → `NOT_TAKEN`
* Voice `1` → `TAKEN`
* Voice `2` → `NOT_TAKEN`

### **F. Random/unrecognized responses don't affect adherence**

They are stored as `UNRECOGNIZED_RESPONSE`.

### **G. NO_RESPONSE is system-generated**

It occurs only when the overall response window closes without a valid accepted response.

### **H. Conflicts become exceptional rather than normal**

Because only the active channel can change adherence, the common Telegram-vs-SMS conflict disappears.

There can still be a rare race condition—for example, two valid events arrive almost simultaneously around a channel transition. We should handle that deterministically on the backend using the reminder/channel state and transaction/idempotency rules, rather than asking the doctor to guess.

**I strongly recommend this model for a production clinical system.**

If you agree with this decision, I'll revise **Step 14 — Adherence Requirements** accordingly and lock it. Then we will move separately to **Step 15 — Audit Log Requirements**, exactly as you requested.


---

## Approved Adherence Requirements for Phase 1

### Core Adherence Principles

**ADH-PRINCIPLE-001 — One Reminder, One Adherence Decision**

Each reminder occurrence SHALL have exactly one adherence decision.

Multiple notification channels (Telegram, SMS, Voice) SHALL NOT create separate adherence records for the same reminder occurrence.

The adherence decision is a property of the reminder, not of the individual notification channels.

**ADH-PRINCIPLE-002 — Main Adherence States**

The primary adherence outcomes SHALL be:
- **PENDING**: Awaiting patient response
- **TAKEN**: Patient explicitly indicated medication was taken
- **NOT_TAKEN**: Patient explicitly indicated medication was not taken
- **NO_RESPONSE**: Response window closed without valid accepted response

These state names follow the repository's terminology and may be adjusted during implementation to match existing conventions, but the semantic distinctions SHALL be preserved.

**ADH-PRINCIPLE-003 — CONFLICTING is Not an Adherence State**

`CONFLICTING` SHALL be treated as an event or processing condition, NOT as a primary adherence state.

If concurrent response events create a conflict:
- The system SHALL preserve raw response events
- The system SHALL detect the conflict
- The system SHALL resolve deterministically using defined rules
- The system SHALL NOT expose `CONFLICTING` as a normal adherence outcome to doctors

**ADH-PRINCIPLE-004 — NO_RESPONSE is Distinct from NOT_TAKEN**

`NO_RESPONSE` (the patient did not respond) SHALL be clinically and analytically distinguishable from `NOT_TAKEN` (the patient explicitly indicated they did not take the medication).

These SHALL NOT be merged or treated as equivalent.

### Channel Response Window Model

**ADH-REQ-001 — One Active Response Channel**

For each reminder occurrence, only one response channel SHALL be active at any given time.

When escalation moves from one channel to another (e.g., Telegram → SMS → Voice), the previous channel's response window SHALL close.

**ADH-REQ-002 — Closed Channels Cannot Change Adherence**

Responses received through a closed channel SHALL be recorded as response events for audit/history purposes.

Responses received through a closed channel SHALL NOT modify the reminder's adherence state.

This is a critical safety invariant.

**ADH-REQ-003 — Channel Window Closure**

When the system transitions from one notification channel to the next during escalation, the previous channel's ability to accept adherence-changing responses SHALL be closed.

Example: If Telegram window closes at T+10min and SMS begins, a Telegram response arriving at T+12min SHALL be recorded but SHALL NOT change adherence if SMS is the active channel.

**ADH-REQ-004 — Response Event Preservation**

ALL response events SHALL be preserved in the notification/response history, including:
- Accepted responses (changed adherence)
- Late responses (closed channel)
- Unrecognized responses
- Duplicate responses

The doctor SHALL be able to view complete response history for audit and clinical review purposes.

### Valid Response Recognition

**ADH-REQ-005 — Telegram Valid Responses**

Valid Telegram responses SHALL be:
- **Taken** button/callback → `TAKEN`
- **Not Taken** button/callback → `NOT_TAKEN`

**ADH-REQ-006 — SMS Valid Responses**

Valid SMS responses SHALL be:
- **"1"** → `TAKEN`
- **"2"** → `NOT_TAKEN`

**ADH-REQ-007 — Voice Valid Responses**

Valid Voice responses SHALL be:
- **DTMF "1"** → `TAKEN`
- **DTMF "2"** → `NOT_TAKEN`

**ADH-REQ-008 — Unrecognized Responses Do Not Change Adherence**

Unrecognized or invalid patient responses SHALL NOT automatically be interpreted as `TAKEN` or `NOT_TAKEN`.

Examples of unrecognized responses:
- SMS: "yes", "okay", "I took it", "hello", "3", arbitrary text
- Telegram: Typed text instead of button press
- Voice: DTMF other than 1 or 2, no DTMF

Unrecognized responses SHALL be classified as `UNRECOGNIZED_RESPONSE` events and SHALL NOT change the reminder's adherence state.

**ADH-REQ-009 — Unrecognized Response Handling**

When an unrecognized response is received, the system MAY:
- Send a clarification message (e.g., "Please reply 1 for Taken or 2 for Not Taken")
- Record the event for audit purposes
- Continue escalation according to policy (if response window is still open)

Sending a clarification message SHALL NOT reset the escalation clock or extend the response window.

**ADH-REQ-010 — NO_RESPONSE Generation**

`NO_RESPONSE` SHALL be system-generated when the overall adherence response window closes without the system having received and accepted a valid patient response.

`NO_RESPONSE` is NOT a patient-provided response; it is a system-determined final state.

### Adherence vs Delivery Separation

**ADH-REQ-011 — Delivery Does Not Equal Adherence (Cross-Channel)**

The system SHALL maintain clear separation across all channels:
- Telegram message delivered ≠ medication `TAKEN`
- SMS message delivered ≠ medication `TAKEN`
- Voice call answered ≠ medication `TAKEN`

Only an explicit, valid, accepted patient response SHALL change adherence from `PENDING` to `TAKEN` or `NOT_TAKEN`.

**ADH-REQ-012 — Provider Failure Does Not Equal NOT_TAKEN**

Technical notification failures (Telegram unavailable, SMS provider down, Voice call failed) SHALL NOT automatically be converted to patient `NOT_TAKEN` adherence.

Provider failure and patient adherence are separate concepts.

The system SHALL distinguish technical failure from patient non-adherence.

### Response Timing and Windows

**ADH-REQ-013 — Adherence Response Window**

Each reminder occurrence SHALL have an adherence response window defining how long the patient may respond.

The adherence response window is separate from:
- Notification escalation timing (when channels are attempted)
- Individual channel response windows (when each channel can accept responses)

**ADH-REQ-014 — Late Response Policy**

Responses received after the adherence response window closes SHALL be validated against the reminder and notification state before processing.

The system SHALL apply a defined late-response policy rather than blindly accepting late responses as current adherence.

**ADH-REQ-015 — First Valid Response Establishes Initial Adherence**

When a valid response is received through the currently active channel, it SHALL establish the reminder's adherence decision.

Further escalation SHALL stop where applicable.

Subsequent responses within the same window SHALL be handled according to response-correction policy.

### Conflict Handling

**ADH-REQ-016 — Deterministic Conflict Resolution**

If two valid response events arrive nearly simultaneously around a channel transition (rare race condition), the system SHALL resolve the conflict deterministically using:
- Reminder/channel state
- Transaction/idempotency rules
- Timestamp precedence
- Defined business rules

The system SHALL NOT require doctors to manually resolve normal concurrent-response scenarios.

**ADH-REQ-017 — Raw Event Preservation During Conflicts**

When conflicting response events occur, the system SHALL:
- Preserve all raw response events
- Detect the conflict
- Resolve according to defined rules
- Record the resolution decision
- Make all events available for audit

### Response vs Adherence Event Classification

**ADH-REQ-018 — Patient-Generated Responses**

Patient-generated responses that can establish adherence:
- `TAKEN` (via Telegram Taken, SMS "1", Voice DTMF "1")
- `NOT_TAKEN` (via Telegram Not Taken, SMS "2", Voice DTMF "2")

**ADH-REQ-019 — System-Generated Adherence States**

System-generated adherence determinations:
- `NO_RESPONSE` (response window closed without valid accepted response)

**ADH-REQ-020 — Response Event Metadata**

Response event classifications that are NOT primary adherence outcomes:
- `UNRECOGNIZED_RESPONSE` (invalid/unrecognized patient input)
- `LATE_RESPONSE` (response received after window closed)
- `CLOSED_CHANNEL_RESPONSE` (response received through closed channel)
- `DUPLICATE_RESPONSE` (redundant response event)

These are metadata/event types, not primary adherence states.

### Doctor Visibility and UI

**ADH-REQ-021 — One Adherence Result Per Reminder**

The doctor's primary view SHALL show one adherence result per reminder occurrence.

Example display:
```
8:00 AM — Amoxicillin 500mg
Status: TAKEN
Channel: SMS
Response Time: 08:15
```

**ADH-REQ-022 — Detailed Response History Available**

The doctor SHALL be able to access detailed notification and response history showing:
- All notification attempts (Telegram, SMS, Voice)
- All response events (accepted, rejected, late, unrecognized)
- Channel transitions
- Timestamps
- Event types

This enables clinical review and system troubleshooting.

### Cross-Document Consistency

**ADH-REQ-023 — Adherence Rules Consistent with Notification**

Adherence requirements SHALL be consistent with:
- Step 10: Notification orchestration and escalation (NOTIF-INVARIANT-001 through NOTIF-REQ-018)
- Step 11: Telegram response handling (TEL-014 through TEL-024)
- Step 12: SMS response handling (SMS-014 through SMS-021, SMS-033, SMS-034)
- Step 13: Voice response handling (to be defined)

**ADH-REQ-024 — Audit Integration**

All adherence state changes and response events SHALL be available for audit logging as defined in Step 15 — Audit Log Requirements.

Audit records SHALL support:
- Adherence timeline reconstruction
- Response event verification
- Clinical record completeness
- System behavior verification

---

## Requirement Ownership and Cross-References

These adherence requirements depend on and reinforce:
- **Step 9**: Reminder occurrence generation
- **Step 10**: Notification orchestration, one active channel rule, delivery ≠ adherence
- **Step 11**: Telegram response mechanism
- **Step 12**: SMS response mechanism and one active SMS context
- **Step 13**: Voice response mechanism
- **Step 15**: Audit logging of adherence events
- **Step 16**: Dashboard display of adherence outcomes
- **Step 17**: Configuration of response windows (separate from escalation timing)

---

## Implementation Notes

- Exact state names (PENDING, TAKEN, NOT_TAKEN, NO_RESPONSE) may be adapted during implementation to match existing repository conventions
- The semantic distinctions between states MUST be preserved regardless of naming
- Response window durations are configurable, not hard-coded
- Conflict resolution algorithms are implementation details, but MUST be deterministic
- All channel response mechanisms MUST follow the one-active-channel model
