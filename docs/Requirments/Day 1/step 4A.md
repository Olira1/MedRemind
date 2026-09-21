
---

# Day 1 — Step 4: Authentication Requirements

Now we move to **Step 4**.

The purpose of this step is to define exactly how **Doctors and Admins access the system**.

Remember our actors:

```text
ADMIN
DOCTOR
PATIENT
```

Only Admin and Doctor have web portal accounts in Phase 1.

The patient does **not** have a web login.

---

# 4.1 Authentication scope

We need to define:

* How users log in
* What credentials they use
* How passwords are handled
* How sessions work
* Logout
* Failed login attempts
* Password reset
* Account status
* Authorization boundary
* Security requirements

Authentication answers:

> **"Who are you?"**

Authorization, which we'll define later, answers:

> **"What are you allowed to do?"**

These are different concepts.

---

# 4.2 Login

### Proposed requirement

A Doctor/Admin should be able to log into the web portal using:

```text
Email
+
Password
```

Flow:

```text
User
 ↓
Login page
 ↓
Enter email + password
 ↓
Backend verifies credentials
 ↓
Valid?
 ├── NO → Login rejected
 └── YES
       ↓
Authenticated session
       ↓
Dashboard
```

---

# 4.3 Invalid credentials

If the email/password combination is incorrect:

```text
Login attempt
     ↓
Invalid credentials
     ↓
Reject authentication
     ↓
Show generic error
```

We should **not reveal whether the email exists**.

For example, instead of:

> "This email does not exist."

we should use something like:

> "Invalid email or password."

This prevents unnecessary account enumeration.

---

# 4.4 Password storage

Passwords must **never be stored as plain text**.

Instead:

```text
User password
      ↓
Secure password hashing
      ↓
Database
```

When logging in:

```text
Entered password
      ↓
Compare with stored password hash
      ↓
Match?
```

We will choose the specific password-hashing technology during the technical design stage.

For now the requirement is:

> **Passwords must be stored using a strong, industry-standard password hashing algorithm and never stored in plaintext.**

---

# 4.5 Session management

After successful login, the user needs an authenticated session.

Conceptually:

```text
Login successful
      ↓
Authenticated session
      ↓
User accesses protected APIs
```

The session must:

* expire according to a defined policy
* be invalidated on logout
* not expose credentials unnecessarily
* be protected against common session attacks

We'll decide whether to use secure cookies, token-based authentication, or another mechanism during the architecture/API stage.

**We should not lock the implementation technology here.**

---

# 4.6 Protected portal

Unauthenticated users must not be able to access protected application resources.

For example:

```text
GET /patients
```

without authentication:

```text
       ↓
401 Unauthorized
```

while an authenticated Doctor:

```text
       ↓
Authorized?
       ↓
Access permitted
```

But authentication alone isn't enough.

A Doctor must also be authorized to access **only their own patients**.

That belongs to authorization requirements.

---

# 4.7 Admin vs Doctor after login

After authentication, the system identifies the user's role.

```text
Login
  ↓
Authenticated user
  ↓
Role
 ├── ADMIN
 │     ↓
 │   Admin portal
 │
 └── DOCTOR
       ↓
     Doctor portal
```

The UI can therefore show different functionality according to role.

More importantly, the **backend must enforce these permissions**.

We must never rely only on hiding buttons in the frontend.

For example, even if a Doctor doesn't see:

> "Delete Doctor"

the backend must still reject a request from a Doctor attempting that operation.

---

# 4.8 Account status

We should have account status.

For example:

```text
ACTIVE
INACTIVE
```

If an Admin deactivates a Doctor:

```text
Doctor account
     ↓
INACTIVE
     ↓
Login rejected
```

The exact behavior for existing sessions will be defined later.

---

# 4.9 Logout

A Doctor/Admin should be able to log out.

```text
Authenticated user
       ↓
Logout
       ↓
Session invalidated
       ↓
Protected resources no longer accessible
```

---

# 4.10 Failed login protection

Because this is a production system, we should protect authentication from brute-force attacks.

For example:

```text
Repeated failed login attempts
             ↓
Rate limiting / temporary restriction
             ↓
Prevent automated guessing
```

We don't need to decide the exact number of attempts yet.

That will be part of our security requirements.

---

# 4.11 Password reset

I recommend that Phase 1 includes password recovery.

A Doctor/Admin who forgets their password should be able to initiate:

```text
Forgot password
      ↓
Enter email
      ↓
Password-reset process
      ↓
Verify reset request
      ↓
Create new password
```

However, we need to decide **how the reset verification is delivered**.

Potential options:

* Email
* SMS
* Other secure mechanism

Since we're already designing SMS infrastructure, it might be tempting to use SMS, but we should choose based on security, cost, and the actual authentication requirements rather than assuming.

---

# 4.12 Patient authentication

For Phase 1:

> **Patients do not have a web application account.**

Therefore:

```text
Patient
  ❌ Web login
  ❌ Password
  ❌ Doctor portal access
```

Instead:

```text
Patient
  ↓
Telegram / SMS / Voice
```

The patient's response must still be associated with the correct patient and reminder.

This will be handled through the notification/channel identity mechanisms we define later.

---

# 4.13 Authentication requirements — draft

I propose these initial requirement IDs:

| ID       | Requirement                                                                    |
| -------- | ------------------------------------------------------------------------------ |
| AUTH-001 | System shall provide secure login for Admin and Doctor users.                  |
| AUTH-002 | Login shall use email and password.                                            |
| AUTH-003 | Invalid credentials shall be rejected.                                         |
| AUTH-004 | System shall not reveal whether a specific account exists during failed login. |
| AUTH-005 | Passwords shall never be stored in plaintext.                                  |
| AUTH-006 | Passwords shall use strong industry-standard hashing.                          |
| AUTH-007 | Protected resources shall require authentication.                              |
| AUTH-008 | System shall identify the authenticated user's role.                           |
| AUTH-009 | System shall support separate Admin and Doctor roles.                          |
| AUTH-010 | Users shall be able to log out.                                                |
| AUTH-011 | Inactive accounts shall not be allowed to authenticate.                        |
| AUTH-012 | Authentication shall have protection against repeated failed login attempts.   |
| AUTH-013 | System shall provide a secure password-reset mechanism.                        |
| AUTH-014 | Patients shall not require web portal accounts in Phase 1.                     |

These are **draft requirements**, not yet locked.

---

# One decision before we lock Step 4

There is one thing I recommend we decide now:

### Should Admin accounts be created only by another Admin?

My recommendation:

```text
First Admin
   ↓
Created during secure system setup

Admin
   ↓
Can create Doctor accounts

Doctor
   ❌ Cannot create Admin accounts
```

This prevents normal Doctors from escalating their privileges.

For example:

```text
ADMIN
 ├── Create Doctor
 ├── Activate Doctor
 └── Deactivate Doctor

DOCTOR
 └── Cannot create ADMIN
```

I recommend this model for our production architecture.

**If you approve this recommendation, Step 4 will have its authentication model established, and then we'll move to Step 5: Doctor/Admin Requirements.**
