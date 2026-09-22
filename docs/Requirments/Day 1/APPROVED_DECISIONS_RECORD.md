# MedRemind Phase 1 - Approved Decisions Record

**Document Version:** 1.0
**Last Updated:** 2026-09-22
**Purpose:** Track all approved decisions and their incorporation into requirements

---

## Decision Incorporation Status

This document tracks the 47 approved decisions from the requirements freeze preparation prompt and their incorporation status into the Day 1 requirements.

**Legend:**
- ✅ Incorporated
- ⏳ In progress
- ❌ Not yet started

---

## Communication & Notification Decisions

### D1 - SMS Simultaneous Reminders
**Status:** ✅ Incorporated
**Decision:** Multiple reminder occurrences at the same time may be combined into one SMS communication using numbered item codes, but each reminder occurrence retains its own independent adherence decision.
**Target Documents:** Step 12 (SMS), Step 14 (Adherence)
**Key Principle:** Communication grouping must never merge underlying reminder/adherence records.
**Incorporated In:** NOTIF-INVARIANT-001 and NOTIF-REQ-013 in Step 10B, commit 5ae6292

### D2 - Voice Simultaneous Reminders
**Status:** ✅ Incorporated
**Decision:** Multiple reminder occurrences at the same time may be combined into one Voice call. Each medication/reminder must be presented separately with independent DTMF responses.
**Target Documents:** Step 13 (Voice), Step 14 (Adherence)
**Incorporated In:** NOTIF-INVARIANT-001 and NOTIF-REQ-015 in Step 10B, commit 5ae6292

### D3 - Telegram Simultaneous Reminders
**Status:** ✅ Incorporated
**Decision:** Multiple reminder occurrences at the same time may be combined into one Telegram message. Each reminder occurrence remains independent with its own response mechanism (inline buttons).
**Target Documents:** Step 11 (Telegram), Step 14 (Adherence)
**Incorporated In:** NOTIF-INVARIANT-001 and NOTIF-REQ-010 in Step 10B, commit 5ae6292

### Combined Communication Invariant
**Status:** ✅ Incorporated
**Decision:** System may group several simultaneous reminder occurrences into one communication (Telegram/SMS/Voice), but every reminder occurrence remains an independent domain object with its own lifecycle, notification history, response events, and adherence decision.
**Target Documents:** Steps 9, 10, 11, 12, 13, 14
**Critical Rule:** Communication grouping is presentation/delivery optimization only.
**Incorporated In:** NOTIF-INVARIANT-001 in Step 10B, commit 5ae6292

---

## Scale & Performance Decisions

### D4 - Phase 1 Scale Targets
**Status:** ✅ Incorporated
**Decision:** Approved Targets:
- Up to 1,500 patients
- Up to 100 doctors
- Up to 3 Admins
- Approximately 4,500 reminder occurrences/day (design target)
- At least 1,000 reminder occurrences/hour (capacity target)
- At least 100 active users
**Target Documents:** Step 18 (NFRs)
**Note:** These are design targets, not predictions
**Incorporated In:** NFR-021, commit 83b98d7

### D5 - Backup/Recovery Targets
**Status:** ✅ Incorporated
**Approved Targets:**
- RPO ≤ 15 minutes
- RTO ≤ 1 hour
**Target Documents:** Step 18 (NFRs)
**Note:** Must be verified against actual deployment infrastructure capabilities
**Incorporated In:** NFR-047, commit 83b98d7

---

## Provider & Channel Decisions

### D6 - Provider Outage Behavior
**Status:** ✅ Incorporated
**Decision:** If notification provider fails: (1) Retry per policy, (2) Transition to next escalation channel when applicable, (3) Preserve notification attempt/history, (4) Record failure, (5) Never convert PROVIDER_FAILURE into NOT_TAKEN
**Target Documents:** Step 10 (Notification), Steps 11/12/13 (Channels)
**Critical Rule:** Provider failure and adherence are separate concepts
**Incorporated In:** NOTIF-INVARIANT-002 in Step 10B, commit 5ae6292

### D7-D12 - Communication Availability
**Status:** ⏳ In progress
**Decisions:**
- Telegram available only when patient has explicitly linked and backend has valid association
- SMS requires valid configured phone number
- Voice requires valid configured phone number
- System must not attempt delivery through unavailable channels
**Target Documents:** Steps 6 (Patient), 10 (Notification), 11/12/13 (Channels)

### D13 - Phone Number Cardinality
**Status:** ⏳ In progress
**Decision:** Do not assume phone number is globally unique to one patient. Same phone may appear on multiple patient records. Patient identity determined by internal Patient ID.
**Target Documents:** Step 6 (Patient), Step 12 (SMS)
**Rule:** No unique database constraint on patient phone number

### D14 - In-Flight Contact Change Behavior
**Status:** ⏳ In progress
**Decision:** Notification attempt captures destination/contact information when created. Changing patient's phone/Telegram after notification created must not silently rewrite destination of already-created attempt. Future reminders use current valid contact configuration.
**Target Documents:** Steps 6 (Patient), 10 (Notification)

### D15 - Telegram Lifecycle
**Status:** ⏳ In progress
**Decision:** Telegram must support link/unlink/relink. When relinking: old association becomes inactive, old tokens invalidated, new identity becomes active, historical records unchanged.
**Target Documents:** Step 11 (Telegram)
**Rule:** Telegram identity must not be primary patient identity

### D16 - SMS Response Association
**Status:** ⏳ In progress
**Decision:** One active SMS response context per patient (Option A). System must ensure incoming response can be unambiguously associated with currently active SMS response context. Old SMS messages cannot modify closed reminders.
**Target Documents:** Step 12 (SMS), Step 14 (Adherence)

---

## Adherence & Response Window Decisions

### D17 - Adherence Response Window
**Status:** ⏳ In progress
**Decision:** A closed channel can NEVER change adherence. Only the currently eligible/open response channel can produce accepted adherence decision.
**Target Documents:** Step 14 (Adherence), Step 10 (Notification)
**Critical Rule:** Responses through closed channels may be preserved as events but cannot change adherence

### D18 - Main Adherence States
**Status:** ⏳ In progress
**Decision:** Main adherence states: PENDING, TAKEN, NOT_TAKEN, NO_RESPONSE. Do not introduce CONFLICTING as primary adherence state.
**Target Documents:** Step 14 (Adherence)

### D19 - CONFLICTING Behavior
**Status:** ⏳ In progress
**Decision:** CONFLICTING is an event/processing condition, NOT an adherence state. If concurrent response events create conflict: preserve raw events, detect conflict, resolve deterministically, do not expose CONFLICTING as normal adherence outcome.
**Target Documents:** Step 14 (Adherence)

### D20 - Adherence Rules
**Status:** ⏳ In progress
**Valid Responses:**
- Telegram: Taken/Not Taken
- SMS: 1=Taken, 2=Not Taken
- Voice: 1=Taken, 2=Not Taken
**Invalid Responses:** Unrecognized text/button/DTMF must not automatically change adherence. Record as UNRECOGNIZED_RESPONSE.
**No Response:** If response window closes without valid accepted response → NO_RESPONSE (not NOT_TAKEN)
**Target Documents:** Steps 11, 12, 13, 14

### D21 - Delivery ≠ Adherence
**Status:** ⏳ In progress
**Decision:** Telegram delivered ≠ Taken. SMS delivered ≠ Taken. Voice answered ≠ Taken. Only accepted valid patient response changes adherence.
**Target Documents:** Steps 10, 11, 12, 13, 14
**Critical Invariant:** Must be explicit across all documents

### D22 - One Active Response Channel
**Status:** ⏳ In progress
**Decision:** For each reminder occurrence, only one response channel may be active at a time. When escalation moves, previous channel closes, next opens. Responses through closed channel cannot change adherence.
**Target Documents:** Steps 10, 14
**All response events:** May remain in history

---

## Authorization & Access Control Decisions

### D23 - Admin Clinical Authority
**Status:** ⏳ In progress
**Decision:** Doctors manage clinical records for assigned patients including medication information, schedules, reminder configuration within system/Admin boundaries. Admin has broad system/operational authority but does not ordinarily modify clinical treatment records merely because account has administrative privileges.
**Target Documents:** Steps 2, 4, 5, 17, 19
**Rule:** Exceptional administrative clinical workflows must be explicitly authorized, narrowly scoped, audited

### D24 - Reminder/Escalation Configuration Authority
**Status:** ⏳ In progress
**Approved Model:**
- Admin controls: system-wide available channels, system defaults, global escalation policy, safety boundaries, allowable configuration limits
- Doctor can configure: reminder/escalation behavior for assigned patients within Admin-defined boundaries
- Patient-level config can override system default subject to Admin boundaries
- Doctor cannot configure another Doctor's patients
**Target Documents:** Steps 5, 17

### D25 - Configuration Precedence
**Status:** ⏳ In progress
**Decision:** Admin boundaries → Doctor patient configuration → schedule-specific configuration, with patient communication availability as hard constraint
**Target Documents:** Step 17
**Precedence:** Admin defines global allowed, Doctor configures assigned-patient behavior, schedule-specific refines, patient communication availability constrains

### D26 - Fundamental Rules Not Configurable
**Status:** ⏳ In progress
**Non-Configurable System Invariants:**
- 1 = Taken, 2 = Not Taken
- NO_RESPONSE != NOT_TAKEN
- Closed channels cannot change adherence
- Delivery ≠ adherence
- Audit records append-only
- Doctor access limited to authorized patients
- Admin/Doctor role boundaries server-side enforced
**Target Documents:** Steps 14, 15, 17
**Rule:** These are business rules, not ordinary settings

---

## Account & Security Decisions

### D27 - First Admin Bootstrap
**Status:** ✅ Incorporated
**Required Security Properties:**
1. First Admin creation through controlled initial deployment/setup
2. No permanent master password
3. No hidden backdoor
4. Initial credential/setup secret securely generated
5. Initial setup secret single-use or disabled after initialization
6. Process creates first Admin with required security controls
7. Initial Admin creation auditable
8. Bootstrap mechanism cannot arbitrarily create additional Admins
**Target Documents:** Steps 4, 19
**Note:** Define security properties first; implementation mechanism later
**Incorporated In:** AUTH-031, AUTH-032, AUTH-033 in Step 4B section 4.13, commit 0de38e6

### D28 - Admin MFA
**Status:** ✅ Incorporated
**Decision:** Admin MFA is mandatory in Phase 1. Doctor MFA architecturally supported but not mandatory (if already in requirements, preserve per current normative version).
**Target Documents:** Step 4
**Rule:** Do not weaken this requirement
**Incorporated In:** AUTH-003, AUTH-009, AUTH-010, AUTH-011, AUTH-012 in Step 4B, confirmed in Phase 1 summary table, commit 0de38e6

### D29 - Password Recovery Ownership
**Status:** ✅ Incorporated  
**Approved Behavior:**
- Doctor/Admin can request password reset through registered email
- System must not reveal whether email belongs to account
- Reset link/token short-lived
- Reset token single-use
- Reset token securely generated/stored
- Rate limiting applies
- Reset events audited
- Passwords never emailed
- Lost email requires controlled administrative/account recovery
- No name/phone self-service recovery
- Admin losing both password and email uses organization/deployment-owner recovery
- No hidden master password/backdoor
- Password reset may revoke existing sessions
**Ownership:** Step 4 is normative owner
**Target Documents:** Step 4 (primary), Step 17 (UI entry point only)
**Rule:** Avoid duplicating as conflicting requirements in Step 17
**Incorporated In:** AUTH-022 through AUTH-030 in Step 4B sections 4.10-4.12, commit 0de38e6 (already present in original Step 4B)

---

## Webhook & Integration Decisions

### D30 - Webhook Security
**Status:** ⏳ In progress
**Decision:** State-changing webhooks must use provider-supported authenticity verification whenever provider offers verification mechanism. If webhook cannot be authenticated sufficiently, must not be trusted for state-changing operations.
**Additional Requirements:** Webhook processing must be idempotent, validated against expected reminder/patient/channel context, protected against replay/duplicate
**Target Documents:** Steps 10, 11, 12, 13, 19

---

## Production & Deployment Decisions

### D31 - SMS/Voice Production Status
**Status:** ⏳ In progress
**Decision:** Phase 1 includes SMS and Voice architecturally, but initial production launch may be Telegram-only.
**Requirements Must Distinguish:**
- Development: Real Telegram Bot, Mock SMS provider, Mock Voice provider
- Architecture: SMS/Voice provider abstraction exists, orchestrator supports channels, production providers can be activated later without redesigning core logic
- Initial production: Telegram may be active, SMS/Voice may be configured but inactive
**Target Documents:** Steps 10, 12, 13
**Rules:** Do not hard-code production SMS/Voice vendor. Do not claim real SMS/Voice provider required for initial release unless explicitly required.

### D32 - Provider Abstraction
**Status:** ⏳ In progress
**Decision:** Core application must not directly depend on specific SMS or Voice vendor. Use conceptual interfaces (SmsProvider, VoiceProvider, TelegramProvider). Provider-specific status/errors mapped to internal notification model.
**Target Documents:** Steps 10, 12, 13
**Rule:** Exact class/interface names are implementation details

### D33 - Notification Architecture Invariant
**Status:** ⏳ In progress
**Decision:** Notification Orchestrator owns: retry decisions, escalation transitions, provider failure handling, channel eligibility, idempotency/concurrency handling, notification attempt creation. Providers should not contain business escalation logic.
**Target Documents:** Step 10

---

## Audit & History Decisions

### D34 - Audit Requirements
**Status:** ⏳ In progress
**Decision:** Preserve existing audit model. Audit separate from application logs, notification history, adherence records. 
**Important Events:** authentication/security, account activation/deactivation, role/permission changes, patient creation/update/archive/restore, patient assignment/transfer, contact changes, Telegram link/unlink, medication creation/update/discontinuation, schedule creation/update/activation/pause/discontinuation, reminder/notification config changes, significant Admin actions, authorization violations
**Audit Record Contents:** timestamp, actor, action, entity, result, correlation ID, limited structured metadata
**Do Not Store:** passwords, password hashes, reset tokens, OTPs, API keys, provider credentials, unnecessary full medical records, unnecessary full message contents, secret webhook payloads
**Target Documents:** Step 15
**Rule:** Audit records append-only for ordinary users

---

## Dashboard & UI Decisions

### D35 - Dashboard Scope
**Status:** ⏳ In progress
**Decision:** Do not expand dashboard scope. Preserve approved requirements.
**Doctor Dashboard:** patient summary, today's reminders, adherence status, attention-needed patients, notification failures, escalation state, recent activity, search/filter
**Admin Dashboard:** broader operational visibility including doctors, patients, reminder activity, notification health/failures, operational activity
**Excluded:** AI risk scores, predictive adherence, AI clinical recommendations, population analytics, financial analytics, appointment analytics
**Target Documents:** Step 16

---

## NFR & Technical Decisions

### D36 - NFR Documentation
**Status:** ✅ Incorporated
**Decision:** Normalize NFRs replacing vague phrases ("expected Phase 1 load", "acceptable recovery") with approved quantitative targets from D4 (scale) and D5 (RPO/RTO).
**Target Documents:** Step 18
**Rule:** If NFR needs future threshold but none approved, mark as remaining pre-production specification item rather than inventing number
**Incorporated In:** NFR-021 and NFR-047, commit 83b98d7

### D37 - Docker Requirement
**Status:** ✅ Incorporated
**Decision:** Docker must be represented as actual documented Phase 1 requirement/NFR or deployment requirement.
**Requirements Establish:**
- Reproducible containerized development environment
- Consistent application build
- Appropriate container configuration for backend/application services
- Docker-related configuration tested
- Production deployment compatible with selected hosting architecture
**Current Architecture:** Frontend=Vercel, Backend=Render, Database=Managed PostgreSQL, Redis=Managed Redis
**Target Documents:** Step 18 (NFRs), potentially new deployment requirements section
**Rule:** Docker for reproducibility/portable build; don't require every production component in user-managed Docker if architecture doesn't require it
**Incorporated In:** NFR-061 and NFR-064, commit 83b98d7

---

## Traceability & Verification Decisions

### D38 - Traceability Matrix
**Status:** ⏳ In progress
**Decision:** Create or update requirements traceability matrix.
**Minimum Mapping:** Requirement → Acceptance Criterion → Design/Architecture element → Implementation/Test (latter two pending)
**Immediate Mapping:** Requirement ID → Acceptance Criterion ID
**Purpose:** Detect requirements with no acceptance criteria, acceptance criteria with no source requirement, duplicate IDs, orphaned requirements
**Target Documents:** New TRACEABILITY_MATRIX.md document

### D39 - Acceptance Criteria Consistency
**Status:** ⏳ In progress
**Decision:** Review Step 20 carefully. Do not rewrite wholesale. Ensure every important approved decision represented, contradictions removed, criteria agree with normative requirements, obsolete criteria from superseded versions marked/updated, existing numbering preserved where possible.
**Important Examples Must Be Consistent:**
- Simultaneous reminders
- SMS response association
- Voice multi-reminder calls
- Telegram grouped messages
- Closed-channel response behavior
- CONFLICTING as event/processing condition only
- Provider outage
- Webhook verification
- Admin clinical authority
- Configuration precedence
- First Admin bootstrap
- Telegram lifecycle
- Deactivated staff access
- Retention
- Deployment recovery targets
**Target Documents:** Step 20
**Rule:** Do not create new criteria merely to increase count

---

## Special Conditions & Edge Cases

### D40 - Deactivated Staff Accounts
**Status:** ⏳ In progress
**Decision:** Inactive/deactivated staff account cannot access protected functionality. Server-side enforcement required. Don't rely solely on frontend hiding. Existing sessions/tokens rejected or revoked per authentication/session design.
**Target Documents:** Steps 4, 5, 19

### D41 - Retention
**Status:** ⏳ In progress
**Decision:** Do not invent legal retention period. State that reminder/adherence/notification/audit history retained per documented/configurable retention policy. Exact legal/organizational duration specified before production if not already defined. No silent deletion, controlled deletion/archival, traceable retention behavior.
**Target Documents:** Steps 15, 19
**Rule:** Don't pretend specific Ethiopian legal retention period established unless repository explicitly establishes it

---

## Explicit Exclusions

### D42 - Things NOT to Add
**Status:** ⏳ In progress
**Explicitly Excluded (unless already in repository):**
- AI adherence prediction
- AI clinical recommendations
- Advanced analytics
- Financial features
- Appointments
- Population health analytics
- Quiet hours
- Dead-letter queue architecture
- Additional communication channels
- WhatsApp
- Email reminders as patient notification channel
- Additional Admin roles
- Complex organizational tenancy
- Arbitrary phone-number uniqueness
- Arbitrary consent workflow
- Additional clinical workflows
- Hidden recovery accounts
- Master passwords
- Provider-specific business logic
- Real SMS/Voice vendor selection
- AWS infrastructure
- AWS-specific architecture
**Target Documents:** All steps - verification review
**Rule:** If already explicitly in repository, preserve it unless prompt explicitly supersedes

---

## Scope & Boundaries

### D43 - Scope of This Task
**Decision:** Work limited to requirements documentation and requirements normalization/consistency/traceability.
**DO NOT:**
- Implement backend
- Implement frontend
- Create migrations
- Create Dockerfiles (unless repository requires documentation-only placeholder)
- Configure Render/Vercel/Telegram
- Create provider credentials
- Write application code

### D44 - File Editing Strategy
**Decision:** Prefer editing existing requirement documents instead of creating parallel requirements system. If creating new file, must have clear purpose.
**Recommended Additions:**
- Requirements status/index
- Decision/disposition record
- Requirements traceability matrix
- Day 1 freeze checklist
**Rule:** Do not create unnecessary duplicate versions

---

## Final Verification Requirements

### D45 - Day 1 Freeze Readiness
**Decision:** After changes, perform final consistency audit checking all critical areas (see prompt section 45 for complete checklist)

### D46 - Final Report Required
**Decision:** Provide structured report with:
- A. Files changed (path, what, why)
- B. Files added (list and reason)
- C. Files marked superseded (old, replacement, reason)
- D. Requirement decisions incorporated (confirm each decision)
- E. Contradictions resolved (list each with resolution)
- F. Remaining unresolved items (issue, affected doc, why unresolved, blocks freeze?)
- G. Scope changes (confirm no new Phase 1 features added beyond approved decisions)
- H. Verification (duplicate IDs, A/B versions, cross-document contradictions, traceability, NFR numbers, terminology consistency)

### D47 - Critical Final Rule
**Decision:** Do not mark Day 1 as "frozen" simply because files edited. Requirements described as "Ready for final human freeze approval" only if all known contradictions resolved and no blocking unresolved requirement remains. Human project owner makes final freeze decision.

---

## Incorporation Progress

**Overall Status:** ⏳ In progress (0% complete)

**Next Steps:**
1. Update CURRENT/NORMATIVE documents with approved decisions
2. Resolve contradictions between documents
3. Create traceability matrix
4. Generate freeze-readiness checklist
5. Perform final consistency audit
6. Generate comprehensive change report

---

## Change History

| Date | Decision Area | Incorporated By |
|------|---------------|-----------------|
| 2026-09-22 | Decision record created | Requirements Normalization |

