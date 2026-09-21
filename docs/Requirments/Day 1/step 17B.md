Yes — I agree with your reasoning on the first point, with an important distinction.

### 1. Reminder and escalation settings

I would **change the requirement** so that doctors can configure reminder/escalation behavior **for patients assigned to them**, while Admin controls the system-wide boundaries.

The best model is:

* **Admin**

  * Defines available channels: Telegram, SMS, Voice.
  * Defines the allowed/default escalation policies and safety limits.
  * Controls provider configuration and system-wide settings.
* **Doctor**

  * Can configure the reminder/escalation policy **for their own assigned patients**.
  * Can decide, for example, whether a particular patient's medication should escalate from Telegram → SMS after 5 minutes rather than 10.
  * Can configure the appropriate escalation sequence within the channels enabled by Admin.
  * Cannot modify another doctor's patients.
* **Patient-level configuration overrides the system default**, but must remain within Admin-defined safety/technical boundaries.

I think this is actually **more clinically appropriate** than making all escalation timing Admin-only. The doctor knows the patient's treatment context and the consequences of a delayed dose.

One important safeguard: the doctor shouldn't be able to configure completely unrestricted values. For example, Admin could define that an escalation interval must be between **1 and 60 minutes**, while the doctor chooses the appropriate value within that range.

Also, every doctor-level change should be audited.

So I recommend changing the Step 17 requirement from:

> "Global escalation timing/channel ordering: Admin only"

to:

> **"System-wide escalation policy and safety boundaries are Admin-controlled. Doctors may configure reminder and escalation settings for patients assigned to them, within the boundaries established by the Admin."**

I would **approve this change and incorporate it into Step 17**.

---

## 2. Password recovery

This needs to be explicitly defined in Phase 1 because otherwise we would have a serious operational problem.

I recommend **email-based password recovery**, with different handling for Doctor and Admin.

### Doctor

If a doctor forgets their password:

1. Click **Forgot Password**.
2. Enter their registered email.
3. System sends a time-limited password-reset link.
4. Doctor opens the link.
5. Creates a new password.
6. Reset token becomes invalid immediately after use.
7. The event is recorded in the audit log.

The system should **not reveal whether an email exists**. For example, the response should be something like:

> "If an account exists for this email, a password reset link has been sent."

That prevents account enumeration.

### What if the doctor forgets their email?

This is slightly different.

I would **not allow the doctor to recover an account simply by providing their name or phone number**, because that creates an account-takeover risk.

Instead, Phase 1 should provide a controlled recovery path, such as:

**"Forgot email / Can't access account?" → Contact administrator / account administrator.**

The Admin can verify the doctor's identity through the organization's established process and help identify or update the account email.

If we later add verified phone-based recovery, that can become another recovery method, but I wouldn't make it a Phase 1 dependency unless we explicitly want to implement and secure it.

### Admin

Admin password recovery should use the same secure email-reset mechanism **if the admin still has access to the registered email**.

But there is an important difference:

**What happens if the only Admin loses both their password and access to their email?**

We should not design a system where the answer is simply "click forgot password."

For Phase 1, I recommend an **administrative recovery procedure** outside the normal application flow, controlled by the deployment/organization owner. This can be documented as an operational recovery procedure rather than a normal user feature.

We should also avoid creating a hidden "master password" or backdoor.

### Security requirements I recommend

Password recovery should include:

* Short-lived, single-use reset tokens.
* Reset tokens stored securely (preferably hashed rather than plaintext).
* Token invalidated immediately after successful reset.
* Rate limiting on reset requests.
* No account-existence disclosure.
* Audit logging of password-reset requests and successful resets.
* Existing sessions should be considered for revocation after a password reset.
* Strong password requirements.
* Never send passwords by email.
* Never store plaintext passwords.
* Password reset credentials must never appear in application logs.

### One more important distinction

**Password recovery ≠ email recovery.**

We should explicitly define these as two different processes:

| Situation                           | Phase 1 solution                                 |
| ----------------------------------- | ------------------------------------------------ |
| Doctor forgot password              | Email reset link                                 |
| Doctor forgot email                 | Controlled Admin/account recovery                |
| Doctor lost access to email         | Controlled administrative recovery               |
| Admin forgot password               | Email reset link                                 |
| Admin lost email access             | Organization/deployment-owner recovery procedure |
| User asks system to reveal password | Never                                            |
| Reset token reused/expired          | Reject                                           |

I recommend adding these requirements to **Step 17 — Settings**, and also cross-referencing them from **Authentication/Security requirements**.

If you agree with both changes, I'll treat them as the approved direction and we can **lock Step 17 and proceed to Step 18 — Non-Functional Requirements**.
