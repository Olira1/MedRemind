# Understanding A/B Requirement Document Versions

## Purpose

This guide explains how to interpret the A/B versions of requirement documents in the Day 1 Phase 1 requirements.

---

## Two Different A/B Patterns

There are **TWO different patterns** for A/B documents in this repository:

### Pattern 1: Replacement (B Replaces A)
Some A/B pairs represent **revisions** where the B version **completely replaces** the A version.

### Pattern 2: Continuation (Both A and B Are Valid)
One A/B pair represents a **conversation continuation** where both documents are valid and should be used together.

---

## Pattern 1: Replacement Pattern

### How It Works

In replacement pattern:
- **A version** = Original requirement that was later revised
- **B version** = New improved requirement that **replaces** the A version
- **Status:** A = SUPERSEDED (historical only), B = CURRENT/NORMATIVE (use this)

### What You Should Do

- ✅ **Use ONLY the B version** for implementation
- ❌ **Ignore the A version** (kept only for historical reference)
- The B version contains all the approved requirements

### Replacement Pattern Examples

#### Step 4: Authentication Requirements

**Files:**
- `step 4A.md` = SUPERSEDED ❌
- `step 4B.md` = CURRENT/NORMATIVE ✅

**What Changed:**
- Step 4A: Original authentication requirements
- Step 4B: Added mandatory Admin MFA, improved password recovery, First Admin bootstrap security

**Which to Use:** Use ONLY Step 4B

---

#### Step 10: Notification & Escalation Requirements

**Files:**
- `step 10A.md` = SUPERSEDED ❌
- `step 10B.md` = CURRENT/NORMATIVE ✅

**What Changed:**
- Step 10A: Had 2-hour escalation waits
- Step 10B: Fixed escalation timing (relative to medication due time, not 2-hour waits)

**Which to Use:** Use ONLY Step 10B

---

#### Step 14: Adherence Requirements

**Files:**
- `step 14A.md` = SUPERSEDED ❌
- `step 14B.md` = CURRENT/NORMATIVE ✅

**What Changed:**
- Step 14A: Earlier adherence model
- Step 14B: Improved channel window model (closed channels cannot change adherence)

**Which to Use:** Use ONLY Step 14B

---

#### Step 17: Settings Requirements

**Files:**
- `step 17A.md` = SUPERSEDED ❌
- `step 17B.md` = CURRENT/NORMATIVE ✅

**What Changed:**
- Step 17A: Original settings requirements
- Step 17B: Added comprehensive configuration authority, precedence rules, non-configurable invariants

**Which to Use:** Use ONLY Step 17B

---

## Pattern 2: Continuation Pattern

### How It Works

In continuation pattern:
- **First document** = Original requirement discussion
- **Second document** = Continuation of the same conversation with additional decisions
- **Status:** BOTH = APPROVED (both are valid)

### What You Should Do

- ✅ **Use BOTH documents together**
- ✅ They complement each other, don't contradict
- ✅ The second document adds to (not replaces) the first

### Continuation Pattern Example

#### Step 6: Patient Requirements

**Files:**
- `step 6.md` = APPROVED ✅
- `step 6B.md` = APPROVED ✅

**Important Note:** There is NO "step 6A.md" file. The first file is just "step 6.md"

**What Each Contains:**

**Step 6.md (Original):**
- Patient creation and management
- Basic patient information fields
- Patient assignment to doctors
- Patient archival and restoration
- Core patient lifecycle

**Step 6B.md (Continuation):**
- Patient gender field (added)
- Patient preferred language (English, Amharic, Afaan Oromoo)
- Patient transfer between doctors
- Additional patient-related decisions

**Which to Use:** Use BOTH Step 6 AND Step 6B together

**Why Both Are Valid:**
- They represent one continuous conversation
- Step 6B adds approved features to Step 6
- No contradictions between them
- Together they form the complete patient requirements

---

## Quick Reference Table

| Step | Files | Pattern | Status | What to Use |
|------|-------|---------|--------|-------------|
| **4** | 4A, 4B | Replacement | 4A=SUPERSEDED, 4B=CURRENT | ✅ Use ONLY 4B |
| **6** | 6, 6B | Continuation | BOTH APPROVED | ✅ Use BOTH 6 and 6B |
| **10** | 10A, 10B | Replacement | 10A=SUPERSEDED, 10B=CURRENT | ✅ Use ONLY 10B |
| **14** | 14A, 14B | Replacement | 14A=SUPERSEDED, 14B=CURRENT | ✅ Use ONLY 14B |
| **17** | 17A, 17B | Replacement | 17A=SUPERSEDED, 17B=CURRENT | ✅ Use ONLY 17B |

---

## Why This Distinction Matters

### For Implementation

**Replacement Pattern:**
- Using the A version would mean implementing outdated, superseded requirements
- Could result in building the wrong features (e.g., 2-hour escalation waits)
- Could miss critical requirements (e.g., Admin MFA)

**Continuation Pattern:**
- Using only Step 6 would miss gender, language, and transfer features
- Using only Step 6B would miss core patient management features
- You need both for complete patient requirements

### For Requirements Review

When reviewing requirements, you must understand:
- Which documents are authoritative
- Which documents are superseded
- Which documents work together

---

## Common Questions

### Q: Why keep superseded documents?

**A:** Historical reference and traceability. They show:
- What requirements were originally considered
- Why changes were made
- Evolution of the requirements thinking

### Q: How do I know if a B version replaces or continues?

**A:** Check `REQUIREMENTS_STATUS.md`:
- If it says "SUPERSEDED" → Replacement pattern (ignore the A version)
- If it says "APPROVED as conversation flow" → Continuation pattern (use both)

### Q: What if there's a contradiction between Step 6 and Step 6B?

**A:** There shouldn't be any contradictions. Step 6B only adds to Step 6. If you find a contradiction, report it as a requirements issue.

### Q: Can I implement from Step 4A or 10A?

**A:** NO. Those are superseded. Always use the CURRENT/NORMATIVE versions (4B, 10B, 14B, 17B).

---

## Visual Summary

### Replacement Pattern (Use B Only)

```
Step 4A (SUPERSEDED)    →  ❌ DO NOT USE
        ↓
Step 4B (CURRENT)       →  ✅ USE THIS
```

### Continuation Pattern (Use Both)

```
Step 6 (APPROVED)       →  ✅ USE THIS
        ↓
      adds to (not replaces)
        ↓
Step 6B (APPROVED)      →  ✅ AND THIS
```

---

## Implementation Checklist

Before implementing any feature, verify:

- [ ] I am using Step 4B (not 4A) for authentication
- [ ] I am using Step 10B (not 10A) for notifications
- [ ] I am using Step 14B (not 14A) for adherence
- [ ] I am using Step 17B (not 17A) for settings
- [ ] I am using BOTH Step 6 and Step 6B for patient features
- [ ] I have checked REQUIREMENTS_STATUS.md to confirm document authority
- [ ] I understand which documents are CURRENT/NORMATIVE vs SUPERSEDED

---

## Where to Find Authoritative Status

**Official Source:** `docs/Requirments/Day 1/REQUIREMENTS_STATUS.md`

This document maintains the definitive record of:
- Which documents are CURRENT/NORMATIVE
- Which documents are SUPERSEDED
- Which documents are APPROVED
- How A/B versions relate to each other

**Always consult REQUIREMENTS_STATUS.md if you're unsure about document authority.**

---

## Summary

**Golden Rules:**

1. **For Steps 4, 10, 14, 17:** Use ONLY the B version, ignore the A version
2. **For Step 6:** Use BOTH Step 6 and Step 6B together
3. **When in doubt:** Check REQUIREMENTS_STATUS.md
4. **Never assume:** Always verify document status before implementing

**The wrong document = the wrong implementation = wasted work**

Make sure you're building from the right requirements!

---

**Last Updated:** 2026-09-22  
**Maintained By:** Requirements Governance
