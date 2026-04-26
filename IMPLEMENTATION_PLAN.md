# R&R Orchestration System - Implementation Plan V14

> **Version:** 1.3  
> **Effort:** 12-16 hours (1 developer)

---

## Executive Summary

A **multi-agent peer review skill** for Antigravity:

1. **Blind peer review** via sequential persona adoption
2. **Quality enforcement** via 3 reviewers + Area Chair synthesis
3. **Guaranteed termination** via max iterations + human escalation
4. **File-based state** with atomic writes + backup

---

## Quick Reference

```bash
/peer-review <doc>              # Start review
/peer-review --help             # Show help  
/peer-review --trust            # Skip pattern checks
/peer-review --resume           # Continue session
/peer-review --max-iter N       # Max iterations (default: 10)
/peer-review --cleanup [--days N]  # Remove old sessions
```

---

## Document Sizing

| Constraint | Value | Notes |
|------------|-------|-------|
| Minimum | 50 words | Below this triggers validation error |
| Maximum | None (soft) | LLM context limits apply (~50k tokens) |
| Recommended | 200-2000 words | Optimal for quality reviews |

---

## Core Protocol

```
Validate → Author → Reviews(3x) → Area Chair → Gatekeeper
                         ↑                          ↓
                         └────── REVISE ────────────┘
```

---

## File Structure (17 files)

```
.agent/skills/peer_review/
├── SKILL.md                    # Orchestration
├── prompts/ (8)
│   ├── _security_header.md     # Prepended to all prompts
│   ├── author_initial.md       
│   ├── author_revision.md      
│   ├── reviewer_technical.md   
│   ├── reviewer_product.md     
│   ├── reviewer_ops.md         
│   ├── area_chair.md           
│   └── escalation.md           
├── templates/ (4)
│   ├── review_output.md        
│   ├── synthesis_output.md     
│   ├── audit_trail.md          
│   └── help_output.md          
├── validation/ (3)
│   ├── input_checks.md         
│   ├── patterns.md             
│   └── rejection_messages.md   
└── logs/
    └── security.log.schema.json
```

**Security header:** Prepended verbatim to each prompt file during assembly.

---

## Log Schema (security.log)

```json
{
  "timestamp": "ISO-8601",
  "session_id": "string",
  "event": "validation_pass|validation_fail|pattern_match|escalation",
  "details": { "patterns_matched": [], "input_size": 0 }
}
```

Events logged: validation results, pattern matches, escalations, trust bypasses.

---

## Session ID

Format: `{YYYYMMDD}-{HHMMSS}-{random4}`

Collision probability: 36^4 = 1.6M combinations. Acceptable for <1000 sessions/day.

---

## Configuration

| Setting | Default | Override |
|---------|---------|----------|
| Max iterations | 10 | `--max-iter N` |
| Retention | 30 days | `--cleanup --days N` |
| Trust mode | Off | `--trust` |

---

## Sample Sessions

### ✅ Happy Path
```
/peer-review ./feature-prd.md

📂 Session created (47 words, ~3 min)

ITER 1: FAIL/FAIL/FAIL → REVISE(6)
ITER 2: PASS/FAIL/PASS → REVISE(1)  
ITER 3: PASS/PASS/PASS → ✅ ACCEPT

Output: .peer_review/.../final/document.md
```

### ❌ Validation Error
```
/peer-review ./too-short.md

❌ Document too short (23 words < 50 minimum)
```

### ⚠️ Escalation
```
/peer-review ./complex.md --max-iter 3

ITER 3: PASS/FAIL/PASS → MAX ITERATIONS

Remaining: 2 issues
[1] Accept anyway  [2] Manual guidance  [3] Abort
> _
```

### ⏸️ Resume
```
/peer-review --resume

Found: 20260131-010000-a1b2 (Iter 2)
[1] Resume  [2] Abandon
> _
```

---

## State & Recovery

- Atomic write: `tmp` → rename → backup `.bak`
- Windows: copy+delete fallback
- Recovery: restore from `.bak` if primary missing

---

## Known Limitations

1. **Memory Firewall = best-effort** — LLM context persists
2. **Pattern matching ≠ comprehensive** — Can be bypassed
3. **POSIX atomicity only** — Windows fallback less safe
4. **Subjective quality** — Meta-assessments are judgment calls

---

## Verification Checklist

| Test | Purpose |
|------|---------|
| Happy path | E2E flow |
| Blind isolation | No leakage |
| Validation | Bad input rejected |
| Pattern detection | Injection flagged |
| Trust mode | Bypass works |
| Resume | Recovery works |
| Escalation | Human notified |
| Backup | `.bak` restore |

---

*V14 | 2026-01-31*
