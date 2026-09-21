Excellent. **Step 1 is now approved and locked.** We will not change its scope casually later. If a scope change becomes necessary, we will explicitly revisit and approve it.

# Day 1 — Step 2: Actors and Roles

### Goal

Now we define **who interacts with the system and what each actor is allowed to do**.

This is important before designing the database, APIs, authentication, authorization, and audit logging.

We should distinguish between:

1. **Human users**
2. **Patients**
3. **The system itself**
4. **External services/providers**

For this step, we should define their responsibilities without yet designing technical implementation.

---

## Proposed actors

### Actor 1 — Doctor

The Doctor is the primary clinical user of the web application.

The Doctor should be able to:

* Log in securely.
* View their dashboard.
* Create and manage patients.
* View patient information.
* Add medications for patients.
* Modify medication information.
* Create and modify medication schedules.
* Enable/disable reminders.
* Configure reminder channels.
* View reminder history.
* View notification delivery status.
* View failed notifications.
* View patient adherence.
* Review whether medication reminders were responded to as **Taken** or **Not Taken**.
* Manage their own account/profile settings, where applicable.
* Perform permitted actions that are recorded in the audit log.

### Important boundary

The Doctor should **not** be able to:

* Change system-level configuration unless explicitly authorized.
* Access another doctor's patients unless we explicitly define such access.
* Modify audit records.
* Directly manipulate notification delivery records to hide failures.
* Delete historical clinical/adherence information in a way that destroys the audit trail.

These boundaries will be refined when we define authorization and audit requirements.

---

# Actor 2 — Admin

The Admin is a system-management user.

The Admin may have broader privileges than a Doctor.

Proposed Admin capabilities:

* Log in securely.
* View system dashboard/administrative information.
* Manage Doctor accounts.
* Activate/deactivate Doctor accounts.
* View system-level notification status.
* View system failures.
* View relevant audit logs.
* Manage system settings where authorized.
* Monitor notification providers.
* Perform administrative maintenance actions.

### Important question

We need to decide whether **Admin and Doctor are two separate roles**, or whether Phase 1 has only one combined role such as:

> `DOCTOR_ADMIN`

My recommendation is **two separate roles**:

```text
ADMIN
DOCTOR
```

because this gives us proper role-based access control from the beginning and avoids redesigning authorization later.

---

# Actor 3 — Patient

The Patient is different from the Doctor/Admin.

In Phase 1, the patient **does not use a web portal**.

The patient interacts with the system through notification channels.

### Telegram

Patient can:

* Receive medication reminder.
* Select **Taken**.
* Select **Not Taken**.

### SMS

Patient can:

* Receive medication reminder.
* Reply according to the defined response rules.
* The system records the response.

### Voice

Patient can:

* Receive automated telephone call.
* Listen to the medication reminder.
* Press the appropriate keypad number:

  * `1` → Taken
  * `2` → Not Taken

The patient does **not** need a system account/password for these interactions unless we later decide otherwise.

---

# Actor 4 — Reminder/Scheduling System

This is not a human user.

The system itself performs automated work.

It should:

* Evaluate medication schedules.
* Create reminders at the appropriate time.
* Queue notification jobs.
* Send notifications through configured channels.
* Track notification attempts.
* Retry failed notifications according to defined rules.
* Prevent duplicate reminder/notification operations.
* Record notification status.
* Process patient responses.
* Create/update adherence records.
* Generate appropriate audit events.

---

# Actor 5 — Telegram Service

The Telegram service is an external system.

It allows our application to:

* Send Telegram reminders.
* Receive patient responses.
* Provide the response back to our backend through the Telegram integration.

Our system should **not make Telegram-specific behavior part of the core medication/reminder business logic**.

Instead, Telegram should be an external notification adapter.

---

# Actor 6 — SMS Provider

The SMS provider is an external service.

It will eventually:

* Accept SMS sending requests.
* Deliver messages to patient phones.
* Provide delivery information where supported.
* Potentially provide incoming SMS responses.
* Send webhooks/callbacks to our backend.

The exact provider is **not part of Step 2**.

---

# Actor 7 — Voice Provider

The Voice provider is another external service.

It will eventually:

* Initiate outbound calls.
* Connect to the patient's telephone number.
* Play the reminder audio.
* Detect keypad input.
* Report call status.
* Send callbacks/webhooks to our backend.

Again, the specific provider will be selected later.

---

# Actor relationship

The overall relationship becomes:

```text
                    ┌──────────────┐
                    │    ADMIN     │
                    └──────┬───────┘
                           │
                           │ manages
                           ▼
                    ┌──────────────┐
                    │    DOCTOR    │
                    └──────┬───────┘
                           │
                     manages│
                           ▼
                    ┌──────────────┐
                    │   PATIENT    │
                    └──────┬───────┘
                           │
                 receives │ responds
                           ▼
             ┌─────────────────────────┐
             │    REMINDER SYSTEM      │
             └──────┬──────┬──────┬────┘
                    │      │      │
                    ▼      ▼      ▼
                Telegram   SMS   Voice
```

And the system records the resulting:

```text
Notification Delivery
        +
Patient Response
        ↓
Adherence Record
```

---

# One important authorization decision

Before we lock Step 2, I recommend we explicitly decide this:

### Doctor data isolation

Should a Doctor be able to see **only their own patients**, while Admin can see/manage all doctors and patients?

My recommendation is:

> **Yes. A Doctor can access only patients assigned to that Doctor. Admin can access system-wide data according to their permissions.**

This is a much safer production model and gives us a clear authorization boundary.

### Proposed Phase 1 roles

```text
ADMIN
  └── System-wide administrative permissions

DOCTOR
  └── Access only to assigned patients and related clinical/reminder data

PATIENT
  └── No web account in Phase 1
      └── Interacts through Telegram / SMS / Voice

SYSTEM
  └── Automated reminder and notification operations

EXTERNAL PROVIDERS
  ├── Telegram
  ├── SMS
  └── Voice
```

**Please confirm these two decisions:**

1. **Two separate roles:** `ADMIN` and `DOCTOR` — rather than one combined role.
2. **Doctor isolation:** a Doctor can access only their assigned patients; Admin has broader access.

If you approve these, **Step 2 will be locked**, and we'll proceed to **Day 1 — Step 3: Core Business Workflows**.


We now have two separate roles:

ADMIN — broader system-level permissions.
DOCTOR — access only to patients assigned to that doctor.
PATIENT — no web account in Phase 1; interacts through Telegram, SMS, and Voice.
SYSTEM — performs automated reminder/notification operations.
External providers — Telegram, SMS, and Voice services.