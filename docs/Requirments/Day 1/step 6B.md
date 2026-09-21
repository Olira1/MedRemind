Yes. I agree with both points.

### 1. Update to Step 6 — Patient Information

We will add:

* **Gender**
* **Preferred language**

For Phase 1, the supported preferred languages will be:

* English
* Amharic
* Afaan Oromoo

So the patient information will conceptually include:

| Category      | Information                                          |
| ------------- | ---------------------------------------------------- |
| Identity      | First name, Last name, Date of birth, Gender         |
| Contact       | Phone number                                         |
| Communication | Preferred language, notification channels            |
| Telegram      | Telegram account/link status                         |
| System        | Status, assigned Doctor, created/updated information |

The **Preferred language** is particularly important because the same medication reminder may need to be generated differently:

> English → English reminder
> Amharic → Amharic reminder
> Afaan Oromoo → Afaan Oromoo reminder

We'll make the notification architecture support additional languages later without redesigning the whole system.

I will update the Step 6 requirements accordingly, including:

> **PAT-023 — System shall allow the patient's preferred language to be specified as English, Amharic, or Afaan Oromoo.**

---

# 2. What happens to the patient's history after Doctor transfer?

This is a very important question, and the answer should be:

> **Nothing is deleted, moved away, or rewritten. The patient's complete history stays with the patient.**

Only the **current ownership/assignment** changes.

For example:

### Before transfer

```text
Doctor A
   │
   ▼
Patient X
   │
   ├── Medication
   ├── Medication schedules
   ├── Reminders
   ├── Notification history
   ├── Adherence history
   └── Audit history
```

Suppose Patient X has six months of history.

Then Admin transfers Patient X to Doctor B.

### After transfer

```text
Doctor B
   │
   ▼
Patient X
   │
   ├── Medication
   ├── Medication schedules
   ├── Reminders
   ├── Notification history
   ├── Adherence history
   └── Audit history
```

**All six months of history are still there.**

The system does **not** change historical records from:

> "Doctor A managed this patient"

to:

> "Doctor B managed this patient."

That would be historically incorrect.

---

## Think of it as "ownership history"

We should record the patient's Doctor assignments over time.

For example:

| Period         | Responsible Doctor |
| -------------- | ------------------ |
| Jan 1 – Jun 30 | Doctor A           |
| Jul 1 – onward | Doctor B           |

So if someone later looks at an adherence record from **June 15**, the system can determine that Doctor A was responsible for the patient at that time.

An adherence record from **July 10** belongs to the period when Doctor B was responsible.

This is much better for a production clinical system.

---

# Example

Imagine:

**Patient:** Abebe

### January–June

Doctor A manages Abebe.

During this period:

```text
100 reminders
82 Taken
10 Not Taken
8 No Response
```

Then on July 1:

```text
Admin
  ↓
Transfers Abebe
  ↓
Doctor A → Doctor B
```

We **do not reset** Abebe's adherence history.

Doctor B can see the appropriate historical information according to their permissions.

For example:

```text
Patient: Abebe

Historical adherence

Jan–Jun
Managed by: Doctor A
Taken: 82
Not Taken: 10
No Response: 8

Jul–present
Managed by: Doctor B
Taken: ...
Not Taken: ...
No Response: ...
```

This gives Doctor B useful clinical context instead of presenting the patient as if they were newly created.

---

# What about Doctor A's access?

This is also important.

After the transfer:

```text
Doctor A
   ❌ Current patient access
```

and:

```text
Doctor B
   ✅ Current patient access
```

But the historical records **remain unchanged**.

Doctor A should not automatically retain ongoing access to the patient's private data simply because they used to manage the patient.

Whether Doctor A can see any **limited historical information after transfer** is an authorization/privacy rule we'll define later.

For our initial model, I recommend:

> **Once transferred, the previous Doctor loses normal access to the patient. The new Doctor gains access to the patient's existing history.**

The Admin can retain appropriate system-level access.

---

# The key principle

I want us to establish this as a core data rule:

> **Changing patient ownership must never rewrite the patient's historical records.**

We should treat these as separate concepts:

```text
Patient
   │
   ├── Current Doctor Assignment
   │
   └── Assignment History
```

while independently preserving:

```text
Patient
   │
   ├── Medication History
   ├── Schedule History
   ├── Reminder History
   ├── Notification History
   ├── Adherence History
   └── Audit History
```

This design will be very important when we later create the database schema.

### Step 6 status

With your additions and approvals, Step 6 is now **locked** with:

* ✅ Gender
* ✅ Preferred language
* ✅ English
* ✅ Amharic
* ✅ Afaan Oromoo
* ✅ Patient transfer
* ✅ Existing medication schedules continue after transfer
* ✅ Historical data remains intact
* ✅ Assignment history is preserved
* ✅ Previous Doctor loses normal current access after transfer
* ✅ New Doctor gets access to existing patient history

**Next: Day 1 — Step 7: Medication Requirements.**
