---
name: peer-review
description: Multi-agent blind peer review with iterative refinement
---

# R&R Peer Review Skill

> A multi-agent blind peer review system that iteratively improves documents through
> specialized feedback from Technical, Product, and Operations perspectives.

---

## Command Reference

| Command | Description |
|---------|-------------|
| `/peer-review <doc>` | Start peer review of document |
| `/peer-review --help` | Show help and examples |
| `/peer-review --trust` | Skip pattern checks (rapid iteration) |
| `/peer-review --resume` | Resume interrupted session |
| `/peer-review --max-iter N` | Set max iterations (default: 10) |
| `/peer-review --cleanup [--days N]` | Remove old sessions (default: 30 days) |

---

## Core Protocol

```
Validate → Author → Reviews(3x) → Area Chair → Verdict
                         ↑                        ↓
                         └────── REVISE ──────────┘
```

### Protocol Flow

1. **Validate** — Check document exists, meets size requirements, scan for suspicious patterns
2. **Author** — Acknowledge receipt, prepare V1 for review
3. **Reviews** — Three blind reviewers (Technical, Product, Ops) evaluate independently
4. **Area Chair** — Synthesize reviews, run meta-quality gate, issue verdict
5. **Verdict** — ACCEPT (done) | REVISE (loop back) | ESCALATE (max iterations hit)

---

## Constants

```
MAX_ITERATIONS_DEFAULT = 10
MIN_WORD_COUNT = 50
RETENTION_DAYS_DEFAULT = 30
SESSION_ID_FORMAT = "{YYYYMMDD}-{HHMMSS}-{random4}"
```

---

## Session ID Generation

Format: `{YYYYMMDD}-{HHMMSS}-{random4}`

Example: `20260131-101500-a1b2`

- `YYYYMMDD` — Date in ISO format
- `HHMMSS` — Time in 24-hour format
- `random4` — 4 alphanumeric characters (a-z, 0-9)

Collision probability: 36^4 = 1.6M combinations. Acceptable for <1000 sessions/day.

---

## State Management

### Session Directory Structure

```
.peer_review/{session_id}/
├── state.json              # Session state (iteration, phase, verdicts)
├── state.json.bak          # Backup of previous state
├── input/
│   └── original.md         # Original document copy
├── iter_1/
│   ├── document.md         # V1 (or V{n})
│   ├── review_technical.md
│   ├── review_product.md
│   ├── review_ops.md
│   └── synthesis.md
├── iter_2/
│   └── ...
├── final/
│   └── document.md         # Accepted final version
└── logs/
    └── security.log        # Security event log
```

### State Schema (state.json)

```json
{
  "session_id": "20260131-101500-a1b2",
  "document_path": "/path/to/original.md",
  "created_at": "2026-01-31T10:15:00Z",
  "updated_at": "2026-01-31T10:30:00Z",
  "current_iteration": 1,
  "current_phase": "review",
  "max_iterations": 10,
  "trust_mode": false,
  "versions": {
    "V1": "hash_abc123"
  },
  "iterations": [
    {
      "iteration": 1,
      "technical_verdict": "FAIL",
      "product_verdict": "PASS",
      "ops_verdict": "FAIL",
      "ac_verdict": "REVISE",
      "issues_count": 6
    }
  ]
}
```

### Atomic Write Protocol

To ensure data integrity:

1. Write new content to `state.json.tmp`
2. If `state.json` exists, copy to `state.json.bak`
3. Rename `state.json.tmp` to `state.json` (atomic on POSIX)
4. **Windows fallback:** Copy then delete (not atomic, but functional)

### Recovery Protocol

When resuming a session:

1. Check if `state.json` exists → load and continue
2. If `state.json` missing but `state.json.bak` exists → restore from backup
3. If neither exists → session is lost, notify user

---

## Phase Transitions

### Phase 0: Validate

**Trigger:** User invokes `/peer-review <doc>`

**Actions:**
1. Parse command arguments
2. Load validation rules from `validation/input_checks.md`
3. Validate file exists and is readable
4. Count words (minimum 50)
5. Check encoding (UTF-8)
6. If not `--trust` mode: Scan for suspicious patterns (see `validation/patterns.md`)
7. Log validation result to security.log
8. On failure: Display rejection message (see `validation/rejection_messages.md`)
9. On success: Generate session ID, create session directory, copy input document

### Phase 1: Author Initial

**Trigger:** Validation passed

**Actions:**
1. Load `prompts/_security_header.md` + `prompts/author_initial.md`
2. Present document to Author persona
3. Author acknowledges receipt, reports word count and estimated review time
4. Output: "Document V1 ready for peer review."
5. Update state: `current_phase = "review"`

### Phase 2: Blind Review

**Trigger:** Author completed

**Actions:**
1. For each reviewer (Technical, Product, Ops):
   a. Load `prompts/_security_header.md` + appropriate reviewer prompt
   b. CRITICAL: Include memory firewall — reviewer must NOT see other reviews
   c. Execute review, save to `iter_{n}/review_{type}.md`
   d. Parse verdict (PASS/FAIL) from output
2. Update state with all three verdicts

### Phase 3: Area Chair Synthesis

**Trigger:** All three reviews complete

**Actions:**
1. Load `prompts/_security_header.md` + `prompts/area_chair.md`
2. Load all three reviewer outputs as context (AC is allowed to see all reviews)
3. Perform Meta-Quality Gate:
   - Problem Validity: Is this worth solving?
   - Scope Appropriateness: Right-sized solution?
   - Execution Viability: Can it be built as specified?
4. Consolidate issues by severity (CRITICAL > MAJOR > MINOR > SUGGESTION)
5. Deduplicate overlapping issues
6. Issue verdict: ACCEPT or REVISE with numbered fix list
7. Save to `iter_{n}/synthesis.md`
8. Update state with AC verdict

### Phase 4: Verdict

**Trigger:** Area Chair completed

**Decision Tree:**

```
IF ac_verdict == "ACCEPT":
    → Copy final document to final/document.md
    → Update state: current_phase = "complete"
    → Output: "✅ Document accepted after {n} iteration(s)"
    
ELIF current_iteration >= max_iterations:
    → Load prompts/escalation.md
    → Prepare audit trail
    → Use notify_user to alert human with options:
        [1] Accept with known issues
        [2] Provide manual guidance
        [3] Abort session
    → Update state: current_phase = "escalated"
    
ELSE:
    → Increment current_iteration
    → Load prompts/author_revision.md
    → Pass consolidated fix list to Author
    → Author produces V(n+1)
    → Return to Phase 2 (Blind Review)
```

---

## Output Format

### Progress Display

```
📂 Session: 20260131-101500-a1b2
📊 Document: 247 words (~2 min)

ITER 1: FAIL/FAIL/PASS → REVISE(4)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT

Output: .peer_review/20260131-101500-a1b2/final/document.md
```

### Verdict Codes

- `PASS` — No CRITICAL issues AND fewer than 2 MAJOR issues
- `FAIL` — Any CRITICAL issue OR 2+ MAJOR issues
- `ACCEPT` — All reviewers passed, document approved
- `REVISE` — Issues found, author must revise
- `ESCALATE` — Max iterations exceeded, human intervention required

---

## File References

### Prompts (prepend _security_header.md to each)

| File | Purpose |
|------|---------|
| `prompts/_security_header.md` | Security preamble for all personas |
| `prompts/author_initial.md` | First-time document submission |
| `prompts/author_revision.md` | Post-R&R revision handling |
| `prompts/reviewer_technical.md` | Technical feasibility review |
| `prompts/reviewer_product.md` | Product/UX review |
| `prompts/reviewer_ops.md` | Operations/security review |
| `prompts/area_chair.md` | Synthesis and meta-quality gate |
| `prompts/escalation.md` | Human escalation handling |

### Templates

| File | Purpose |
|------|---------|
| `templates/review_output.md` | Individual reviewer output format |
| `templates/synthesis_output.md` | Area Chair output format |
| `templates/audit_trail.md` | Session history for escalation |
| `templates/help_output.md` | --help command output |

### Validation

| File | Purpose |
|------|---------|
| `validation/input_checks.md` | Validation rules |
| `validation/patterns.md` | Suspicious pattern definitions |
| `validation/rejection_messages.md` | User-friendly error messages |

### Logs

| File | Purpose |
|------|---------|
| `logs/security.log.schema.json` | Security log format definition |

---

## Error Handling

### Graceful Failures

1. **Validation errors** → Display rejection message, do NOT create session
2. **File read errors** → Display error, suggest checking path
3. **Pattern detection** → Log warning, CONTINUE review (don't block)
4. **State corruption** → Attempt recovery from .bak, notify user if impossible
5. **Max iterations** → Escalate to human, provide clear options

### Logging Protocol

All significant events logged to `logs/security.log`:

```json
{
  "timestamp": "2026-01-31T10:15:00Z",
  "session_id": "20260131-101500-a1b2",
  "event": "validation_pass",
  "details": { "input_size": 247, "patterns_matched": [] }
}
```

Event types: `validation_pass`, `validation_fail`, `pattern_match`, `escalation`

---

## Known Limitations

1. **Memory Firewall = best-effort** — LLM context may persist between calls
2. **Pattern matching ≠ comprehensive** — Sophisticated injections may bypass
3. **POSIX atomicity only** — Windows fallback is less safe
4. **Subjective quality** — Meta-assessments involve judgment calls

---

*SKILL.md v1.0 — R&R Peer Review Orchestration*
