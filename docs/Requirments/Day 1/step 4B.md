Good. We’ll proceed exactly as agreed.

### Step 19 decision

We will **pause Step 19 approval** and first revise **Step 4 — Authentication Requirements**. The original Step 4 already owns authentication behavior such as login, passwords, sessions, logout, failed-login protection, password reset, and the patient authentication boundary. 

The revision will add the decisions we made later:

* **Admin MFA is mandatory in Phase 1**
* Doctor MFA is architecturally supported but not mandatory in Phase 1
* Password reset uses a **secure, time-limited email reset link**
* No account/email enumeration
* Reset tokens are single-use and securely handled
* Password reset can revoke existing sessions
* Forgotten/lost email requires controlled administrative recovery
* Admin losing both password and email follows an organization/deployment-owner recovery procedure
* MFA recovery must not create a security bypass
* Rate limiting and account protection remain part of authentication
* We will keep broader security implementation controls for **Step 19**, rather than duplicating them here

One item remains intentionally separate: **who is allowed to create Admin and Doctor accounts**. I won't lock that rule into the revised Step 4 until you explicitly confirm it.

## Revised Step 4

# STEP 4 — AUTHENTICATION REQUIREMENTS

## 4.1 Purpose

The authentication system shall provide secure access to the MedReminder web application for authorized staff users while maintaining a clear separation between authentication, authorization, and patient communication channels.

Phase 1 web authentication applies to **Admin** and **Doctor** users.

Patients shall not have web application accounts in Phase 1. Patients interact with the system through supported reminder and response channels such as Telegram, SMS, and automated Voice.

---

## 4.2 Authentication Scope

Authentication requirements cover:

* Login
* Credentials
* Passwords
* Multi-factor authentication
* Sessions
* Logout
* Failed-login protection
* Password recovery
* Account status
* Authentication-related recovery procedures
* Authorization boundary
* Protection against account enumeration
* Authentication-related security controls

Broader system-wide security controls are defined separately in Step 19 — Security Requirements.

---

## 4.3 Staff Web Accounts

### 4.3.1 Admin Accounts

Admin users shall have authenticated web accounts.

Admin authentication shall require:

1. Email address
2. Password
3. Multi-factor authentication (MFA)

**MFA is mandatory for Admin accounts in Phase 1.**

An Admin shall not be considered fully authenticated until all required authentication factors have been successfully validated.

### 4.3.2 Doctor Accounts

Doctor users shall have authenticated web accounts.

Doctor authentication shall require:

1. Email address
2. Password

The authentication architecture shall be designed so that MFA can be enabled for Doctor accounts without redesigning the authentication system.

Doctor MFA is **not mandatory for Phase 1** unless subsequently approved as a requirement.

---

## 4.4 Login

### AUTH-001 — Secure Staff Login

The system shall provide secure login for authorized Admin and Doctor users.

### AUTH-002 — Email and Password Authentication

The primary web login credentials shall be:

* Registered email address
* Password

### AUTH-003 — Admin MFA

Admin users shall complete the required MFA step during authentication.

The MFA mechanism and implementation shall follow secure industry-standard practices.

### AUTH-004 — Generic Invalid-Credential Response

Invalid authentication attempts shall be rejected.

The system shall provide a generic failure response that does not reveal whether:

* The email exists
* The account is active
* The password was incorrect
* The MFA factor was incorrect

### AUTH-005 — Account Enumeration Protection

Authentication and password-recovery flows shall not disclose whether a particular email address belongs to an account.

The system shall use equivalent or appropriately controlled responses for existing and non-existing accounts where disclosure could enable account enumeration.

---

## 4.5 Password Requirements

### AUTH-006 — Password Confidentiality

Passwords shall never be stored or logged in plaintext.

### AUTH-007 — Secure Password Hashing

Passwords shall be stored using a strong, industry-standard password hashing mechanism designed for password storage.

The authentication system shall never require recovery of the original plaintext password.

### AUTH-008 — Password Change

Authorized users shall be able to change their password through a secure authenticated process.

Password changes shall require appropriate authentication and validation.

---

## 4.6 Multi-Factor Authentication

### AUTH-009 — Mandatory Admin MFA

All Admin accounts shall use MFA in Phase 1.

### AUTH-010 — MFA Protection

MFA secrets, recovery mechanisms, and related authentication information shall be protected from unauthorized access.

### AUTH-011 — MFA Recovery

MFA recovery shall use a controlled and secure recovery process.

The recovery process shall not provide an authentication bypass that is weaker than the protection it replaces.

MFA recovery events shall be appropriately auditable.

### AUTH-012 — Doctor MFA Extensibility

The authentication architecture shall support adding MFA to Doctor accounts in the future without requiring a fundamental redesign of the authentication system.

---

## 4.7 Session Management

### AUTH-013 — Authenticated Sessions

Protected web application resources shall require a valid authenticated session.

### AUTH-014 — Role Recognition

The system shall identify the authenticated user's role and apply the appropriate authorization boundary.

Admin and Doctor roles shall remain distinct.

### AUTH-015 — Session Protection

Authenticated sessions shall be protected against unauthorized reuse and session-related attacks.

### AUTH-016 — Logout

Users shall be able to log out of their authenticated session.

Logout shall invalidate the applicable session so that the protected application cannot continue to be accessed through that invalidated session.

### AUTH-017 — Password Reset Session Handling

Following a successful password reset, existing sessions shall be revoked where appropriate so that a previously compromised session cannot remain authorized indefinitely.

---

## 4.8 Account Status

### AUTH-018 — Active Accounts

Only accounts in an appropriate active state shall be permitted to authenticate.

### AUTH-019 — Inactive Accounts

Inactive or deactivated accounts shall not be permitted to authenticate.

Changing an account's status shall not silently delete its historical application data.

---

## 4.9 Failed Login Protection

### AUTH-020 — Failed Authentication Protection

The system shall protect authentication endpoints against repeated failed authentication attempts.

Protection shall include appropriate rate limiting and/or temporary restrictions.

### AUTH-021 — Abuse Resistance

Authentication protection shall reduce the risk of:

* Brute-force password attacks
* Credential-stuffing attempts
* Automated login abuse
* Excessive MFA attempts

Security controls and implementation details are further defined in Step 19.

---

## 4.10 Password Recovery

### AUTH-022 — Secure Password Recovery

Admin and Doctor users shall have a secure password-recovery process.

### AUTH-023 — Email-Based Password Reset

Phase 1 password recovery shall use the user's registered email address to provide a secure password-reset mechanism.

The system shall send a time-limited password-reset link rather than sending the user's existing password.

### AUTH-024 — Password Reset Token

Password-reset tokens shall:

* Be securely generated
* Be difficult to guess
* Be time-limited
* Be single-use
* Become invalid after successful use
* Not expose sensitive information
* Not be stored or logged in a form that unnecessarily exposes the secret

### AUTH-025 — Password Reset Enumeration Protection

A password-reset request shall not reveal whether the supplied email address belongs to an account.

### AUTH-026 — Password Reset Rate Limiting

Password-reset requests shall be rate limited to reduce abuse and automated account-targeting attempts.

### AUTH-027 — Password Reset Completion

After successful password reset:

* The old password shall no longer authenticate
* The reset token shall become invalid
* Existing sessions shall be revoked where required by the security policy
* The event shall be appropriately auditable

---

## 4.11 Forgotten or Lost Email Address

### AUTH-028 — Controlled Account Recovery

If a Doctor or Admin cannot access or remember the registered email address, the system shall not provide unrestricted self-service account discovery based only on easily obtainable personal information.

Account recovery shall use a controlled administrative or organizational recovery process.

### AUTH-029 — No Sensitive Account Disclosure

The recovery process shall not disclose account information to an unauthorized person merely because they know a patient's, doctor's, or administrator's name, phone number, or other basic identifying information.

---

## 4.12 Admin Recovery

### AUTH-030 — Admin Account Recovery

If an Admin loses access to both the password and the registered email account, recovery shall follow a controlled organization/deployment-owner recovery procedure.

The system shall not contain a hidden master password, universal bypass credential, or undocumented authentication backdoor.

Recovery actions shall be appropriately documented and auditable.

---

## 4.13 Patient Authentication Boundary

### AUTH-031 — No Patient Web Login in Phase 1

Patients shall not receive web application login accounts in Phase 1.

Patients shall interact with the system through approved communication and reminder channels.

### AUTH-032 — Patient Identity Separation

Patient communication-channel identifiers such as:

* Telegram identifiers
* Phone numbers
* SMS response contexts
* Voice call identifiers

shall not be treated as substitutes for staff web authentication.

The backend shall validate the identity and authorization context associated with patient responses.

---

## 4.14 Authorization Boundary

Authentication establishes the identity of a user.

Authorization determines what that authenticated user is permitted to access.

### AUTH-033 — Authenticated Access

Protected application resources shall require successful authentication unless explicitly designated as public or system callback endpoints.

### AUTH-034 — Role-Based Authorization

The system shall enforce role-based authorization after authentication.

An authenticated Doctor shall not automatically receive Admin permissions.

An authenticated Admin shall receive only the permissions assigned to the Admin role.

### AUTH-035 — Patient Data Authorization

Authentication shall not by itself grant access to every patient record.

Patient data access shall be restricted according to the authorization rules defined for the user's role and assigned scope.

---

## 4.15 Authentication Event Auditability

Significant authentication and account-security events shall be available for audit purposes, including where applicable:

* Successful login
* Failed login
* Logout
* Password change
* Password reset request
* Successful password reset
* MFA enrollment/change
* MFA failure
* MFA recovery
* Account activation/deactivation
* Account lock/unlock
* Security-related account recovery

Detailed audit requirements are defined in Step 15 — Audit Log Requirements.

---

## 4.16 Authentication Requirements Outside Step 4

The following are intentionally owned by Step 19 rather than duplicated here:

* Secure cookie/session implementation details
* CSRF protection
* Security headers
* Encryption in transit
* Encryption at rest where applicable
* API security
* Webhook security
* Secret management
* Database security
* Notification security
* Sensitive-data logging controls
* Security monitoring
* Dependency/security scanning
* Penetration/security testing
* Incident-response controls

Step 4 defines **what authentication must do**.

Step 19 defines **the broader security controls required to implement and operate it securely**.

---

## 4.17 Phase 1 Authentication Summary

| Area                            | Phase 1 Requirement                    |
| ------------------------------- | -------------------------------------- |
| Admin web login                 | Required                               |
| Doctor web login                | Required                               |
| Patient web login               | Not supported                          |
| Login credential                | Email + password                       |
| Admin MFA                       | Mandatory                              |
| Doctor MFA                      | Architecture supported; not mandatory  |
| Password storage                | Secure password hashing                |
| Password reset                  | Registered email                       |
| Reset link                      | Time-limited and single-use            |
| Account enumeration             | Prevented                              |
| Failed-login protection         | Required                               |
| Session management              | Required                               |
| Logout                          | Required                               |
| Inactive account login          | Not permitted                          |
| Lost email recovery             | Controlled recovery                    |
| Admin total account loss        | Organization/deployment-owner recovery |
| Hidden master password/backdoor | Prohibited                             |
| Role separation                 | Required                               |
| Patient authorization boundary  | Required                               |
| Authentication security events  | Auditable                              |

---

## 4.18 Requirement Ownership

Step 4 owns the **authentication behavior and recovery rules**.

Step 15 owns the **audit requirements**.

Step 19 owns the **system-wide security controls and secure implementation requirements**.

The three sections shall reinforce one another without creating conflicting authentication rules.

### One decision before we lock Step 4

There is just **one account-management rule** I intentionally left out because I don't want to assume your approval:

> **Should Admins be able to create Doctor accounts, while Doctors cannot create Admin accounts, with the first Admin created through a secure initial deployment/setup process?**

If you agree, we'll add that to Step 4, then **lock Step 4** and move back to Step 19 for its cleaned-up security requirements.
