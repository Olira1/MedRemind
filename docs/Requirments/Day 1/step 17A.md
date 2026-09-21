Excellent. **Step 16 — Dashboard Requirements is now locked**, including the **“Patients Needing Attention”** section based on transparent rules rather than AI-generated risk scores.

We now move to **Step 17 — Settings Requirements** only.

# Step 17 — Settings Requirements

Settings are important because the system needs configurable behavior without requiring developers to change code every time an operational value changes.

However, there is a danger here:

> If we put too many things into Settings, the system becomes complicated and users can accidentally change critical clinical behavior.

So I recommend that Phase 1 settings be divided into:

1. **User settings**
2. **Notification/reminder settings**
3. **System/admin settings**
4. **Security settings**

And we should strictly control who can change each setting.

---

## 17.1 User settings

A Doctor should be able to manage their own non-clinical preferences.

For example:

* Display name/profile information where permitted
* Preferred interface language
* Time zone, if required
* Notification preferences for the Doctor's own system alerts
* Password/security settings

The Doctor's interface language should support the Phase 1 languages where applicable:

* English
* Amharic
* Afaan Oromoo

Important distinction:

> A Doctor's interface language is separate from a patient's reminder language.

A Doctor may use English while a patient receives reminders in Afaan Oromoo.

---

# 17.2 Patient communication settings

Patient-specific communication settings should be managed from the patient's record rather than being hidden inside global system settings.

For example:

* Patient preferred language
* Phone number
* Telegram connection status
* Available notification channels
* Patient communication preferences

The patient's preferred language controls the language used for reminder communications.

Example:

```text id="q6p5u1"
Doctor UI language: English

Patient:
Preferred language: Afaan Oromoo

Reminder:
→ Afaan Oromoo
```

---

# 17.3 Reminder and escalation settings

This is one of the most important areas.

We have already established the escalation concept:

```text id="v9ip0r"
Telegram
   ↓
SMS
   ↓
Voice
```

The exact timing should be configurable.

For example:

```text id="kq6vzx"
Telegram starts: 08:00
Telegram response window: 10 min

SMS starts: 08:10
SMS response window: 10 min

Voice starts: 08:20
```

These values should **not be hard-coded into the application**.

But there is an important security/clinical consideration:

### Who can change them?

I recommend:

**Admin only**, not Doctor.

A Doctor should configure a patient's medication schedule, but should not be able to globally change the system's escalation behavior.

---

# 17.4 Channel enable/disable

Admin should be able to configure whether a notification channel is available.

For example:

```text
Telegram: ENABLED
SMS:      ENABLED
Voice:    ENABLED
```

If SMS is temporarily unavailable because no production provider is configured, the system should be able to disable that channel without changing the core reminder logic.

The orchestrator then follows the configured available escalation sequence.

---

# 17.5 Provider settings

Production providers will eventually be configured.

Examples:

* SMS provider
* Voice provider
* Telegram Bot configuration

But provider credentials are highly sensitive.

Therefore:

> Provider credentials should **not** be exposed as normal readable settings to Doctors.

Even Admin UI access should be carefully restricted.

Credentials should preferably be supplied through secure deployment/environment configuration rather than casually entered and displayed in the application.

For example:

```text
SMS_API_KEY
VOICE_API_KEY
TELEGRAM_BOT_TOKEN
```

These belong in secure environment/secret management, not ordinary database settings.

---

# 17.6 Localization settings

The system should have centralized localization.

Supported Phase 1 languages:

* English
* Amharic
* Afaan Oromoo

Settings should control:

* Available system languages
* Default language
* Reminder message templates
* Notification templates

But message templates should be centrally managed rather than allowing every Doctor to independently rewrite system-critical messages.

This prevents inconsistent communication.

---

# 17.7 Notification templates

We should support templates for:

### Telegram

Example concept:

> Medication reminder + medication information + response buttons

### SMS

Example concept:

> Medication reminder + `1 = Taken`, `2 = Not Taken`

### Voice

Example concept:

> Automated spoken reminder + `1 = Taken`, `2 = Not Taken`

The template system should support localized versions.

A Doctor should not need to manually construct these messages every time a reminder is sent.

---

# 17.8 Security settings

Security-related settings should include appropriate controls such as:

* Password change
* Session/security management where supported
* Account activation/deactivation
* Role/permission management for authorized Admins
* Authentication policy configuration where appropriate

However, we should avoid exposing dangerous low-level security configuration to ordinary users.

---

# 17.9 System timezone

Because reminder scheduling is time-sensitive, timezone handling must be explicit.

I recommend:

* Store timestamps internally in UTC.
* Patient reminder schedules use an explicit timezone.
* Display times according to the appropriate user's/patient's timezone where applicable.

For the initial Ethiopia-focused system, we can establish an appropriate default timezone, but the architecture should not assume UTC+3 forever.

This matters because:

> A reminder scheduled for 08:00 must mean 08:00 in the patient's intended timezone.

---

# 17.10 Settings change history

Important settings changes should generate audit events.

For example:

```text
Admin
→ changed SMS response window
→ 10 minutes → 15 minutes
→ 12:30
```

That should appear in the audit log.

This is especially important for:

* Escalation timing
* Channel enable/disable
* Security settings
* Roles/permissions
* Notification configuration
* Localization/template configuration

---

# 17.11 Settings validation

Settings must have validation.

For example:

A user should not be able to configure:

```text
SMS response window = -10 minutes
```

or:

```text
Voice starts before Telegram
```

or:

```text
No valid notification channel exists
```

The system should reject invalid configurations before they become active.

---

# 17.12 Settings versioning / effective time

This is particularly important for reminder systems.

Suppose:

> Admin changes SMS escalation from 10 minutes → 15 minutes.

What happens to reminders already scheduled?

We should **not silently rewrite historical reminder behavior**.

I recommend:

> Configuration changes apply to new/future reminder occurrences unless an explicit rule says otherwise.

Existing reminder occurrences should retain the configuration/policy under which they were created, where necessary for traceability.

This prevents historical records from becoming confusing.

Example:

```text
Monday reminder:
SMS window = 10 minutes

Tuesday onward:
SMS window = 15 minutes
```

The Monday reminder should not suddenly appear as though it had a 15-minute window.

This is an important production-quality requirement.

---

# 17.13 Doctor vs Admin settings

A simple permission model:

| Setting                      | Doctor |                          Admin |
| ---------------------------- | -----: | -----------------------------: |
| Own interface language       |      ✅ |                              ✅ |
| Own password                 |      ✅ |                              ✅ |
| Patient language             |      ✅ |                 ✅ within scope |
| Patient contact channels     |      ✅ |                 ✅ within scope |
| Patient medication schedule  |      ✅ |                 ✅ within scope |
| Global escalation timing     |      ❌ |                              ✅ |
| Global channel configuration |      ❌ |                              ✅ |
| Provider configuration       |      ❌ |                     Restricted |
| System templates             |      ❌ |                     Restricted |
| Roles/permissions            |      ❌ | ✅ according to admin authority |
| Security policy              |      ❌ |                     Restricted |
| System settings              |      ❌ |           ✅ according to scope |

The exact Admin permission model can be refined later.

---

# 17.14 What should NOT be configurable?

Some system rules should remain fixed in application logic because allowing users to change them would create unsafe ambiguity.

For example:

* `1 = Taken`
* `2 = Not Taken`
* `NO_RESPONSE` is not `NOT_TAKEN`
* Closed channels cannot change adherence
* Audit logs are append-only
* Doctors cannot access other doctors' patients
* Notification delivery does not equal adherence

These are **business rules**, not ordinary settings.

This is an important distinction:

> **Settings configure behavior; they should not allow users to rewrite the fundamental safety/business rules.**

---

# Proposed formal Step 17 requirements

## 17. Settings Requirements

### Purpose

The system shall provide controlled configuration capabilities for user preferences, patient communication preferences, reminder/notification behavior, localization, security, and administrative operations while preventing unauthorized or unsafe modification of core business rules.

### SET-001 — Settings Access

The system shall provide settings functionality appropriate to the user's authorized role.

### SET-002 — Role-Based Settings

Settings shall be protected by role-based authorization.

### SET-003 — Doctor Personal Settings

Doctors shall be able to manage permitted personal preferences such as interface language and their own security settings.

### SET-004 — Admin Settings

Authorized Admin users shall be able to manage permitted system and administrative settings within their scope.

### SET-005 — Patient Communication Settings

Authorized users shall be able to manage permitted patient communication preferences from the patient record.

### SET-006 — Patient Preferred Language

The system shall allow an authorized user to configure a patient's preferred communication language.

### SET-007 — Supported Languages

Phase 1 shall support English, Amharic, and Afaan Oromoo for applicable user-facing and patient communication content.

### SET-008 — Language Separation

Patient communication language shall be independent of the Doctor's or Admin's interface language.

### SET-009 — Notification Channel Configuration

Authorized Admin users shall be able to configure the availability of supported notification channels.

### SET-010 — Channel Ordering

Authorized Admin users shall be able to configure the permitted notification escalation sequence.

### SET-011 — Escalation Timing

Authorized Admin users shall be able to configure notification escalation timing and response-window values within valid system limits.

### SET-012 — Active Response Window

Settings shall support configuration of the response window associated with each supported notification channel.

### SET-013 — Channel Validation

The system shall validate that notification-channel configuration is logically valid before activation.

### SET-014 — Invalid Configuration Prevention

The system shall reject configuration values that are outside permitted ranges or create invalid reminder/escalation behavior.

### SET-015 — Notification Templates

The system shall support centrally managed notification templates for supported channels.

### SET-016 — Localized Templates

Notification templates shall support the Phase 1 supported languages.

### SET-017 — Template Consistency

Critical system notification content shall be centrally controlled to maintain consistent meaning across channels and languages.

### SET-018 — Provider Abstraction

Notification settings shall remain independent of any specific SMS or Voice provider.

### SET-019 — Provider Credentials

Provider credentials and secrets shall not be exposed through ordinary user settings.

### SET-020 — Secret Management

Production provider credentials shall be managed through appropriate secure secret/configuration mechanisms.

### SET-021 — Telegram Configuration

Telegram Bot configuration shall be restricted to authorized system configuration mechanisms.

### SET-022 — SMS Configuration

SMS provider configuration shall be restricted to authorized system configuration mechanisms.

### SET-023 — Voice Configuration

Voice provider configuration shall be restricted to authorized system configuration mechanisms.

### SET-024 — Security Settings

The system shall provide authorized security-related settings appropriate to the user's role.

### SET-025 — Role Management

Only authorized Admin users shall be permitted to change user roles or permissions.

### SET-026 — Timezone

The system shall support explicit timezone handling for reminder scheduling and time-sensitive operations.

### SET-027 — UTC Storage

System timestamps shall continue to be stored in UTC regardless of displayed timezone.

### SET-028 — Patient Schedule Timezone

Reminder schedules shall use an explicit timezone appropriate to the patient or scheduling context.

### SET-029 — Settings Audit

Significant settings changes shall generate audit events.

### SET-030 — Configuration Traceability

The system shall maintain sufficient information to determine which relevant configuration governed a reminder occurrence when required for historical traceability.

### SET-031 — Historical Integrity

Changes to settings shall not silently rewrite historical reminder, notification, or adherence records.

### SET-032 — Future Configuration

Unless explicitly defined otherwise, configuration changes shall apply to future reminder occurrences rather than retroactively changing completed historical occurrences.

### SET-033 — Configuration Validation

Settings shall be validated before they become active.

### SET-034 — Configuration Feedback

The system shall clearly communicate invalid configuration values to the authorized user.

### SET-035 — Core Business Rules

Users shall not be permitted to modify fundamental safety, authorization, adherence, or audit rules through ordinary settings.

### SET-036 — Adherence Rule Protection

Settings shall not permit users to redefine the meaning of TAKEN, NOT_TAKEN, or NO_RESPONSE.

### SET-037 — Channel Window Protection

Settings shall not permit a closed notification channel to modify adherence contrary to the defined response-window policy.

### SET-038 — Authorization Protection

Settings shall not permit a Doctor to expand their access beyond their authorized patient and data scope.

### SET-039 — Audit Protection

Settings shall not permit ordinary users to disable, modify, or delete required audit functionality.

### SET-040 — Provider Replacement

The system shall allow supported notification providers to be changed without requiring changes to core reminder and adherence business logic.

### SET-041 — Configuration Reliability

Critical configuration changes shall be persisted reliably.

### SET-042 — Configuration Availability

The reminder and notification system shall use only valid active configuration when scheduling or processing future reminder occurrences.

### SET-043 — Default Configuration

The system shall provide safe valid defaults for required settings where defaults are appropriate.

### SET-044 — Configuration Scope

Settings shall have an explicit scope, such as user, patient, organization, or system, and the system shall enforce that scope.

### SET-045 — Permission Enforcement

Settings APIs and interfaces shall enforce authorization independently of frontend visibility.

### SET-046 — Sensitive Data Minimization

Settings interfaces shall not unnecessarily expose sensitive patient information, credentials, tokens, or internal security data.

## The key decision before we lock Step 17

There is one design point I particularly recommend:

### **Global escalation settings should be Admin-controlled, not Doctor-controlled.**

A Doctor can control the **patient's medication schedule**, but should not be able to change the system-wide rule that says:

> Telegram → 10 min → SMS → 10 min → Voice.

This protects consistency and prevents one Doctor from accidentally creating unsafe or inconsistent notification behavior.

The Doctor can still see the applicable escalation configuration when it is relevant to a patient's reminder.

**If you accept Step 17 with this recommendation, I'll lock it and move to Step 18 — Non-Functional Requirements.**
