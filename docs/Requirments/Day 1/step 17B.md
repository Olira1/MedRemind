# STEP 17 — SETTINGS REQUIREMENTS

## 17.1 Purpose and Scope

The system shall provide controlled configuration capabilities that allow authorized users to customize operational behavior within defined security boundaries while preserving fundamental business rules and authorization constraints.

**Cross-Document Authority:** Step 4B defines authentication requirements including password recovery behavior. Step 14B defines adherence rules that are non-configurable. Step 15 defines audit requirements for settings changes. Step 19 defines security controls including server-side authorization enforcement. This document defines settings requirements that support operational flexibility within approved boundaries.

Settings shall provide controlled configuration of:
* User and interface preferences  
* Patient communication settings
* Reminder and escalation behavior
* System-wide administrative settings
* Security and account management entry points
* Localization and timezone behavior

**Critical Constraint:** Settings shall NOT provide mechanisms to bypass fundamental business rules, security boundaries, or authorization constraints established in other requirement documents.

---

## 17.2 Settings Categories

### Administrative Settings (Admin Role)
System-wide configuration affecting overall system behavior:
* Available communication channels (Telegram, SMS, Voice enable/disable)
* Global escalation policy boundaries and safety limits
* System default reminder and escalation policies  
* Allowable configuration ranges for Doctor-level settings
* Provider configuration and system operational settings
* Security-sensitive system configuration

### Clinical Settings (Doctor Role) 
Patient-specific clinical configuration within Admin boundaries:
* Reminder and escalation behavior for assigned patients
* Patient-specific escalation timing and sequence
* Patient communication preferences and overrides
* Clinical workflow customization within authorized scope

### Patient Communication Settings
Configuration affecting patient interaction:
* Primary phone number for SMS/Voice
* Telegram account linking status
* Preferred communication language (English, Amharic, Afaan Oromoo) 
* Channel availability and preferences

### User Interface Settings
Application interface customization:
* User interface language
* Display preferences
* Operational dashboard configuration
* Regional/timezone display preferences

### Security and Account Settings
Authentication and security-related configuration:
* Password recovery initiation (UI entry points)
* Account security preferences where applicable
* Security notification preferences

---

## 17.3 Role-Based Settings Authority

### SET-001 — Admin System Boundaries

Admin users shall control system-wide settings including:
* Available communication channels (Telegram, SMS, Voice enable/disable)
* System default escalation policies and global boundaries
* Safety limits and allowable configuration ranges
* Provider configuration and system-wide operational settings
* Security-sensitive system configuration

### SET-002 — Doctor Patient-Level Configuration

Doctor users shall configure reminder and escalation behavior for patients assigned to them, within boundaries established by Admin.

Doctor configuration capabilities shall include:
* Reminder escalation policies for assigned patients
* Escalation timing within Admin-defined safety boundaries
* Communication channel selection using Admin-enabled channels
* Patient-specific reminder customization

Doctor users shall NOT:
* Configure patients not assigned to them  
* Exceed Admin-defined safety or technical boundaries
* Modify system-wide defaults or boundaries
* Access another Doctor's patient configuration

### SET-003 — Patient Communication Authority

Patient communication settings shall be managed through authorized clinical workflows.

Patient communication settings include:
* Primary phone number for SMS/Voice
* Telegram account linking/unlinking
* Preferred communication language
* Channel availability status

**Critical Boundary:** Patient communication settings must not grant clinical modification authority. Settings changes affect communication delivery but do not override clinical treatment decisions.

### SET-004 — Administrative Clinical Authority

Admin settings shall not provide ordinary clinical modification authority over Doctor-managed treatment records.

Admin operational authority does not automatically grant clinical treatment record modification privileges merely because the user has administrative system access.

Exceptional administrative clinical workflows, if any, must be explicitly authorized, narrowly scoped, and auditable.

---

## 17.4 Configuration Precedence

### SET-005 — Settings Hierarchy

The system shall enforce configuration precedence as follows:

1. **Admin boundaries** (outermost constraint - cannot be exceeded)
2. **Doctor patient configuration** (within Admin boundaries)  
3. **Schedule-specific configuration** (refinements within Doctor configuration)
4. **Patient communication availability** (hard constraint - overrides configured preferences when channel unavailable)

### SET-006 — Hard Constraints

Patient communication availability shall act as a hard constraint that overrides configured preferences:
* Unavailable channels cannot be selected regardless of configuration preferences
* Telegram unavailable → cannot send Telegram regardless of escalation policy
* Invalid phone number → cannot send SMS/Voice regardless of configuration
* System shall automatically use available channels according to policy

### SET-007 — Boundary Enforcement

Lower-level configuration shall NOT override higher-level boundaries:
* Doctor configuration cannot exceed Admin-defined safety limits
* Schedule configuration cannot violate Doctor-defined patient policies  
* No configuration level can bypass fundamental business rules

---

## 17.5 User and Interface Settings

### SET-008 — Interface Language

Users shall configure application interface language independently from patient communication language.

Supported interface languages:
* English
* Amharic  
* Afaan Oromoo

### SET-009 — Display Preferences

Users may configure appropriate display preferences for:
* Dashboard layout preferences
* Regional number and date formatting  
* Timezone display preferences (display only - does not affect reminder scheduling)
* Operational interface customization

### SET-010 — Language Independence

Application interface language shall remain independent from:
* Patient communication language settings
* Notification template language
* System operational language

---

## 17.6 Patient Communication Settings

### SET-011 — Phone Number Management

The system shall support patient phone number configuration with the following constraints:
* One primary phone number per patient for SMS/Voice
* Phone numbers are NOT globally unique across patients
* Same phone number may appear on multiple patient records
* Internal Patient ID remains the primary patient identity

### SET-012 — Telegram Lifecycle

The system shall support Telegram account management:
* **Link:** Associate patient with Telegram account using secure linking mechanism
* **Unlink:** Deactivate patient Telegram association  
* **Relink:** Replace existing association with new Telegram account

Telegram lifecycle requirements:
* Secure temporary linking tokens with expiration
* Token invalidation after successful use or relink
* Historical Telegram notification records preserved during account changes
* Telegram identity is NOT primary patient identity

### SET-013 — Communication Language

Patient communication language shall be configured independently from interface language.

Supported patient communication languages:
* English
* Amharic
* Afaan Oromoo

Communication language affects:
* Notification template selection
* Message content language
* Response instruction language

### SET-014 — Channel Availability

The system shall track and enforce patient communication channel availability:
* Telegram: Available when patient has active linking
* SMS: Available when valid phone number configured  
* Voice: Available when valid phone number configured
* Unavailable channels shall not be selected for new notifications

---

## 17.7 Reminder and Escalation Settings

### SET-015 — System-Wide Escalation Boundaries

Admin users shall define system-wide escalation policy boundaries including:
* Available escalation channels (Telegram, SMS, Voice)
* Minimum and maximum escalation intervals (e.g., 1-60 minutes)
* Default escalation sequences and timing
* Safety limits that cannot be exceeded by lower-level configuration
* Channel-specific operational constraints

### SET-016 — Doctor Patient-Level Escalation

Doctor users shall configure escalation behavior for assigned patients within Admin boundaries:
* Patient-specific escalation timing (within Admin limits)
* Channel sequence using Admin-enabled channels
* Patient-appropriate escalation policies
* Escalation customization for individual patient needs

### SET-017 — Escalation Channel Enable/Disable

Admin users shall control system-wide channel availability:
* Telegram channel enable/disable
* SMS channel enable/disable  
* Voice channel enable/disable

**Production Flexibility:** Initial production deployment may operate Telegram-only with SMS/Voice prepared but inactive until production providers are configured and enabled.

### SET-018 — Channel Selection Logic

The system shall select escalation channels according to:
* Admin-enabled channels (system-wide availability)
* Doctor-configured patient policy (within Admin boundaries)
* Patient communication availability (hard constraint)
* Schedule-specific refinements where applicable

Unavailable channels shall be automatically excluded from escalation sequences.

---

## 17.8 System and Administrative Settings

### SET-019 — Provider Configuration

Communication provider credentials and configuration shall be managed through secure deployment mechanisms rather than ordinary application settings.

Provider configuration requirements:
* Telegram bot credentials via secure environment configuration
* SMS provider credentials via secure environment configuration  
* Voice provider credentials via secure environment configuration
* No provider credentials in ordinary user-editable settings
* No credential exposure in UI, logs, or audit records

### SET-020 — Channel Operational Settings

Admin users may configure operational aspects of communication channels:
* Retry policies and limits
* Timeout configurations
* Provider-specific operational parameters
* Channel health monitoring settings

Provider-specific technical configuration remains in secure deployment settings.

### SET-021 — First Admin Bootstrap Cross-Reference

First Admin account creation requirements are defined in Step 4B (sections AUTH-031 through AUTH-033) and Step 19 security requirements.

Settings interface may provide administrative entry points for ongoing Admin account management but shall not redefine bootstrap security requirements.

---

## 17.9 Security and Account Settings

### SET-022 — Password Recovery Entry Points

The Settings interface may provide UI entry points for password recovery actions (e.g., "Forgot Password" links).

**Authoritative Requirements:** Complete password recovery behavior, security properties, and business rules are normatively defined in Step 4B, requirements AUTH-022 through AUTH-030.

Settings interface shall NOT redefine authentication or password recovery security requirements established in Step 4B.

### SET-023 — Account Security Integration

Settings interface shall enforce authentication and authorization requirements established in Step 4B and security controls established in Step 19.

Security-sensitive settings changes shall:
* Require appropriate authentication
* Enforce role-based authorization boundaries
* Maintain server-side security enforcement
* Follow principle of least privilege

### SET-024 — MFA Settings Cross-Reference

Multi-factor authentication requirements and settings are defined in Step 4B (sections AUTH-009 through AUTH-012).

Settings interface may provide MFA management entry points but shall comply with authentication requirements established in Step 4B.

---

## 17.10 Localization

### SET-025 — Language Separation

The system shall maintain clear separation between:
* **Application interface language** (Doctor/Admin UI language)
* **Patient communication language** (notification content language)

These language settings shall be independent and configurable separately.

### SET-026 — Supported Languages

Phase 1 supported languages for both interface and communication:
* English
* Amharic
* Afaan Oromoo

### SET-027 — Centralized Localization

Localization templates and translations shall remain centralized.

Settings shall provide language selection mechanisms but shall not duplicate notification template logic or translation management within the settings module.

---

## 17.11 Timezone

### SET-028 — Timezone Architecture

The system shall support timezone-aware reminder scheduling while maintaining consistent internal time representation:
* Timestamps stored consistently in UTC where appropriate
* Reminder schedules have explicit timezone association
* Settings provide timezone selection for operational display

### SET-029 — Default Timezone

Ethiopia (UTC+3) may serve as the default operational timezone for the target deployment.

The architecture shall NOT permanently assume UTC+3 as a universal timezone constraint to preserve deployment flexibility.

### SET-030 — Display vs Storage

Timezone settings affect display and user interface presentation but do not override the underlying UTC-based timestamp storage architecture established elsewhere in the system.

---

## 17.12 Configuration Validation

### SET-031 — Settings Validation

All settings changes shall be validated before acceptance:
* Escalation intervals within Admin-defined ranges
* Channel selection using available/enabled channels
* Patient assignment boundaries enforced for Doctor settings
* Invalid configurations rejected with appropriate error messages
* Security-sensitive validation per Step 19 requirements

### SET-032 — Boundary Validation

The system shall prevent configuration changes that violate established boundaries:
* Doctor settings cannot exceed Admin safety limits
* Patient communication channel selection limited to available channels
* Invalid timezone, language, or operational values rejected
* Unauthorized patient configuration attempts blocked

### SET-033 — Validation Error Handling

Invalid settings shall be rejected with clear error messages that:
* Explain the validation failure
* Do not expose sensitive internal information
* Guide users toward valid configuration options
* Maintain security boundaries during error reporting

---

## 17.13 Configuration Change Semantics

### SET-034 — Future Application

Settings changes shall apply to future reminder occurrences and operational behavior.

Existing reminder occurrences shall retain the configuration and policy active when they were created.

### SET-035 — Historical Data Preservation

Settings changes shall NOT:
* Rewrite historical reminder, adherence, or notification outcomes
* Silently modify past clinical records
* Alter completed notification delivery history
* Change historical audit records

### SET-036 — Change Propagation

When settings changes affect future behavior:
* Changes take effect for newly created reminders
* Active reminders may complete under previous configuration
* Escalation policies apply based on reminder creation time
* Patient communication changes affect future delivery attempts

---

## 17.14 Auditability

### SET-037 — Settings Change Audit

Significant settings changes shall be auditable including:
* Admin boundary and safety limit changes
* Doctor patient-level configuration changes
* Channel enable/disable actions
* Provider configuration changes (metadata only, no credentials)
* Security-sensitive settings modifications

**Audit Requirements:** Detailed audit requirements are defined in Step 15. Settings changes shall comply with audit requirements established in Step 15.

### SET-038 — Sensitive Information Protection

Audit records for settings changes shall NOT contain:
* Provider credentials or secrets
* Password-related information
* Security tokens or sensitive configuration values
* Patient communication details beyond necessary metadata

---

## 17.15 Non-Configurable System Invariants  

### SET-039 — Fundamental Business Rules

The following system rules shall NOT be configurable through settings interfaces:
* `1 = Taken, 2 = Not Taken` (SMS and Voice response codes)
* `NO_RESPONSE != NOT_TAKEN` (adherence state distinction)
* Closed channels cannot change adherence
* Delivery status ≠ adherence status
* Audit records are append-only for ordinary users
* Doctor access limited to authorized/assigned patients  
* Admin/Doctor role boundaries enforced server-side
* Communication grouping does not merge adherence records

**Rationale:** These are fundamental business logic rules, not ordinary configuration settings.

### SET-040 — Security Invariants

Settings shall NOT provide mechanisms to:
* Bypass authentication requirements
* Override authorization boundaries
* Disable server-side security enforcement
* Expose or modify security secrets
* Grant unauthorized access to patient data
* Circumvent audit requirements

---

## 17.16 Cross-Document Authority

To prevent conflicting requirements:

| Settings Area | Primary Requirement Section |
|---------------|----------------------------|
| Authentication behavior | Step 4B |
| Password recovery business rules | Step 4B (AUTH-022 to AUTH-030) |
| Settings UI entry points | Step 17 |
| Adherence state definitions | Step 14B |
| Non-configurable adherence rules | Step 17 |
| Security controls | Step 19 |
| Settings security enforcement | Step 17 |
| Audit requirements | Step 15 |
| Settings change auditability | Step 17 |
| Notification business logic | Steps 10B-13 |
| Settings affecting notifications | Step 17 |

**Critical Rule:** Settings requirements shall support and enforce boundaries established in other documents. Where settings affect behavior governed by other documents, those documents remain authoritative for business logic while Step 17 defines configuration interfaces and validation.

---

## 17.17 Settings Acceptance Criteria

Step 17 shall be considered satisfied for Phase 1 when:

1. Admin users can configure system-wide channel availability and escalation boundaries.
2. Doctor users can configure reminder/escalation for assigned patients within Admin boundaries.
3. Doctor settings are restricted to assigned patients only (server-side enforced).
4. Configuration precedence is enforced (Admin → Doctor → Schedule → Availability).
5. Patient communication availability acts as a hard constraint on channel selection.
6. Settings changes apply to future occurrences without rewriting historical data.
7. Phone numbers are not treated as globally unique patient identifiers.
8. Telegram linking/unlinking/relinking is supported with secure token mechanisms.
9. Application interface language is independent from patient communication language.
10. Supported languages (English, Amharic, Afaan Oromoo) are available for both interface and communication.
11. Timezone settings support operational display without breaking UTC storage architecture.
12. Provider credentials are managed through secure deployment, not ordinary settings.
13. Settings validation prevents invalid and unauthorized configuration changes.
14. Fundamental business rules (adherence states, response codes, etc.) are not configurable.
15. Settings changes are auditable per Step 15 requirements without exposing secrets.
16. Password recovery UI entry points cross-reference Step 4B authoritative requirements.
17. Settings security enforcement follows Step 19 authorization and validation requirements.
18. Admin clinical authority boundaries are enforced (no ordinary treatment record modification).
19. Settings interface supports First Admin bootstrap cross-reference to Step 4B requirements.
20. Channel enable/disable supports production deployment flexibility (Telegram-only initial launch).

---

## 17.18 Relationship With Other Requirements

Step 17 shall support, not override, requirements established elsewhere.

Where settings affect authentication behavior, Step 4B remains the authoritative source for authentication and password recovery requirements.

Where settings affect adherence behavior, Step 14B remains authoritative for adherence business logic and non-configurable rules.

Where settings involve security controls, Step 19 remains authoritative for security implementation and server-side enforcement.

Where settings changes require auditing, Step 15 remains authoritative for audit requirements and sensitive data protection.

Where settings affect notification behavior, Steps 10B-13 remain authoritative for communication business logic.

Step 17 defines the configuration interface requirements necessary to provide operational flexibility while maintaining security boundaries, authorization constraints, and fundamental business rules across all system functions.
