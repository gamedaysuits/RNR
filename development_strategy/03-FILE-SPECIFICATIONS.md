# File Specifications — All 17 Skill Files

> **Reference:** Complete specifications for every file in the R&R skill.  
> **Use:** Agents refer to this for exact requirements; Human uses for verification.

---

## Directory Structure Overview

```
.agent/skills/peer_review/           # 17 files total
├── SKILL.md                         # 1. Orchestration
├── prompts/                         # 8 files
│   ├── _security_header.md          # 2. Security preamble
│   ├── author_initial.md            # 3. Initial submission
│   ├── author_revision.md           # 4. Revision handling
│   ├── reviewer_technical.md        # 5. Technical review
│   ├── reviewer_product.md          # 6. Product review
│   ├── reviewer_ops.md              # 7. Operations review
│   ├── area_chair.md                # 8. Synthesis
│   └── escalation.md                # 9. Human escalation
├── templates/                       # 4 files
│   ├── review_output.md             # 10. Review format
│   ├── synthesis_output.md          # 11. AC output format
│   ├── audit_trail.md               # 12. Session history
│   └── help_output.md               # 13. --help output
├── validation/                      # 3 files
│   ├── input_checks.md              # 14. Validation rules
│   ├── patterns.md                  # 15. Suspicious patterns
│   └── rejection_messages.md        # 16. Error messages
└── logs/
    └── security.log.schema.json     # 17. Log format
```

---

## 1. SKILL.md — Main Orchestration

| Property | Value |
|----------|-------|
| **Purpose** | Entry point for `/peer-review` command |
| **Size** | ~200-400 lines |
| **Format** | Markdown with YAML frontmatter |

### Required Sections

```yaml
---
name: peer-review
description: Multi-agent blind peer review with iterative refinement
---
```

1. **Command Parser** — Handle all flags and arguments
2. **Phase Controller** — Manage Validate → Author → Review → AC → Verdict
3. **State Manager** — Atomic writes, backups, recovery
4. **Iteration Loop** — Enforce max iterations, track progress
5. **File References** — Point to prompts/, templates/, validation/
6. **Error Handling** — Graceful failures with user notification

### Key Constants

```markdown
MAX_ITERATIONS_DEFAULT = 10
MIN_WORD_COUNT = 50
RETENTION_DAYS_DEFAULT = 30
SESSION_ID_FORMAT = "{YYYYMMDD}-{HHMMSS}-{random4}"
```

---

## 2. prompts/_security_header.md — Security Preamble

| Property | Value |
|----------|-------|
| **Purpose** | Prepended to ALL persona prompts |
| **Size** | ≤50 lines |
| **Usage** | `cat _security_header.md {prompt}.md` |

### Required Content

- Role boundary definition (REVIEWER, not assistant)
- Prohibited actions list
- Injection pattern awareness
- Logging instruction for suspicious content
- Output constraint reminder

---

## 3. prompts/author_initial.md — Initial Submission

| Property | Value |
|----------|-------|
| **Purpose** | Handle first-time document submission |
| **Persona** | Document Author |
| **Inputs** | `{{DOCUMENT_PATH}}`, `{{SESSION_ID}}` |
| **Outputs** | Confirmation message, V1 status |

### Responsibilities

1. Acknowledge receipt
2. Count words, estimate review time
3. Check basic formatting
4. Output "V1 ready for peer review"

---

## 4. prompts/author_revision.md — Revision Handler

| Property | Value |
|----------|-------|
| **Purpose** | Handle post-R&R revisions |
| **Persona** | Document Author |
| **Inputs** | `{{DOCUMENT_PATH}}`, `{{FIX_LIST}}`, `{{ITERATION}}` |
| **Outputs** | V(n+1), change summary |

### Responsibilities

1. Read consolidated fix list from AC
2. Address ONLY specified issues
3. Create versioned output
4. Report changes made

---

## 5. prompts/reviewer_technical.md — Technical Review

| Property | Value |
|----------|-------|
| **Purpose** | Technical feasibility assessment |
| **Persona** | Senior Technical Architect |
| **Critical** | Must include memory firewall |

### Focus Areas

- Technical feasibility
- Architectural integrity
- Complexity accuracy
- Dependency risks
- Performance implications

### Output Requirements

- Use `templates/review_output.md` format
- Severity: CRITICAL / MAJOR / MINOR / SUGGESTION
- Verdict: PASS (no CRITICAL, <2 MAJOR) or FAIL

---

## 6. prompts/reviewer_product.md — Product Review

| Property | Value |
|----------|-------|
| **Purpose** | User/product assessment |
| **Persona** | Product Manager / UX Lead |
| **Critical** | Must include memory firewall |

### Focus Areas

- User problem validity
- User flow completeness
- Edge case handling
- Scope appropriateness
- Market fit

---

## 7. prompts/reviewer_ops.md — Operations Review

| Property | Value |
|----------|-------|
| **Purpose** | Operations/security assessment |
| **Persona** | DevOps / Security Engineer |
| **Critical** | Must include memory firewall |

### Focus Areas

- Security vulnerabilities
- Timeline realism
- Resource requirements
- Operational complexity
- Deployment risks

---

## 8. prompts/area_chair.md — Synthesis

| Property | Value |
|----------|-------|
| **Purpose** | Consolidate reviews, issue verdict |
| **Persona** | Review Board Chair |
| **Inputs** | All 3 reviewer outputs |

### Responsibilities

1. Read all reviews (dedup allowed)
2. Consolidate overlapping issues
3. Prioritize by severity
4. Perform Meta-Quality Gate:
   - Problem Validity
   - Scope Appropriateness
   - Execution Viability
5. Issue: ACCEPT or REVISE with numbered list

### Output

Use `templates/synthesis_output.md` format

---

## 9. prompts/escalation.md — Human Escalation

| Property | Value |
|----------|-------|
| **Purpose** | Handle max iterations exceeded |
| **Trigger** | `iteration > max_iterations` |
| **Action** | Use `notify_user` tool |

### Output Content

1. Iteration history summary
2. Remaining unresolved issues
3. Human options: Accept / Guide / Abort
4. Complete audit trail

---

## 10. templates/review_output.md — Review Format

```markdown
# {{REVIEWER_TYPE}} Review — Iteration {{ITERATION}}

**Document:** {{DOCUMENT_PATH}}
**Reviewed:** {{TIMESTAMP}}

## Verdict: {{PASS|FAIL}}

## Issues Found

### CRITICAL
- Issue description

### MAJOR
- Issue description

### MINOR
- Issue description

### SUGGESTIONS
- Suggestion description

## Summary
One to two sentence assessment.
```

---

## 11. templates/synthesis_output.md — AC Output

```markdown
# Area Chair Synthesis — Iteration {{ITERATION}}

**Session:** {{SESSION_ID}}

## Meta-Quality Gate
| Check | Status | Notes |
|-------|--------|-------|
| Problem Validity | PASS/CONCERN | ... |
| Scope Appropriateness | PASS/CONCERN | ... |
| Execution Viability | PASS/CONCERN | ... |

## Review Summary
| Reviewer | Verdict | Critical | Major | Minor |
|----------|---------|----------|-------|-------|
| Technical | ... | ... | ... | ... |
| Product | ... | ... | ... | ... |
| Operations | ... | ... | ... | ... |

## Consolidated Fix List
1. [CRITICAL] Issue — from Reviewer
2. [MAJOR] Issue — from Reviewer
...

## Verdict: {{ACCEPT|REVISE}}
```

---

## 12. templates/audit_trail.md — Session History

```markdown
# R&R Audit Trail

**Session:** {{SESSION_ID}}
**Document:** {{DOCUMENT_PATH}}
**Started:** {{START_TIME}}
**Ended:** {{END_TIME}}

## Iteration History
| Iter | Technical | Product | Ops | AC Verdict | Issues |
|------|-----------|---------|-----|------------|--------|
| 1 | FAIL | PASS | FAIL | REVISE | 6 |
| 2 | PASS | PASS | PASS | ACCEPT | 0 |

## Final Status: {{ACCEPTED|ESCALATED|ABORTED}}

## Remaining Issues (if escalated)
- ...

## Files Generated
- .peer_review/{{SESSION}}/V1.md
- .peer_review/{{SESSION}}/V2.md
- .peer_review/{{SESSION}}/final/document.md
```

---

## 13. templates/help_output.md — Help Text

```markdown
# R&R Peer Review — Help

## Commands
| Command | Description |
|---------|-------------|
| `/peer-review <doc>` | Start peer review |
| `/peer-review --help` | Show this help |
| `/peer-review --trust` | Skip pattern checks |
| `/peer-review --resume` | Resume session |
| `/peer-review --max-iter N` | Set max iterations |
| `/peer-review --cleanup` | Remove old sessions |

## Example
```
/peer-review ./my-prd.md

📂 Session: 20260131-101500-a1b2
📊 Document: 247 words (~2 min)

ITER 1: FAIL/FAIL/PASS → REVISE(4)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT
```

## Document Requirements
- Minimum: 50 words
- Recommended: 200-2000 words
- Format: Markdown preferred
```

---

## 14. validation/input_checks.md — Validation Rules

### Rules

| Check | Criteria | Action |
|-------|----------|--------|
| Word count | ≥50 words | Reject if below |
| File exists | Path readable | Reject if not |
| Encoding | Valid UTF-8 | Reject if invalid |
| Extension | .md, .txt preferred | Warn if other |
| Size | <50k tokens | Warn if exceeded |

### Implementation Notes

- Word count: `content.split(/\s+/).filter(w => w).length`
- Log all validation results to security.log

---

## 15. validation/patterns.md — Suspicious Patterns

### Regex Patterns

```javascript
const PATTERNS = [
  /ignore\s+(all\s+)?previous\s+instructions?/i,
  /forget\s+(your|the)\s+role/i,
  /you\s+are\s+now/i,
  /system\s*prompt/i,
  /\[INST\]|\[\/INST\]/i,
  /```(bash|sh|shell|cmd)/i
];
```

### Heuristic Patterns

- Base64 blocks > 100 chars
- Lines with > 50% special characters
- JSON with "role" or "content" keys

### Action

- Log match to security.log
- Continue review with warning
- Do NOT block unless in non-trust mode

---

## 16. validation/rejection_messages.md — Error Messages

| Error Code | Message |
|------------|---------|
| `too_short` | ❌ Document too short ({{N}} words < 50 minimum) |
| `not_found` | ❌ Cannot read '{{PATH}}'. Check file exists. |
| `encoding` | ❌ Encoding error. Ensure UTF-8 encoding. |
| `too_large` | ⚠️ Document may exceed context limit. Consider splitting. |
| `pattern` | ⚠️ Suspicious pattern detected and logged. |

---

## 17. logs/security.log.schema.json — Log Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["timestamp", "session_id", "event"],
  "properties": {
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "session_id": {
      "type": "string"
    },
    "event": {
      "type": "string",
      "enum": ["validation_pass", "validation_fail", "pattern_match", "escalation"]
    },
    "details": {
      "type": "object",
      "properties": {
        "patterns_matched": {
          "type": "array",
          "items": { "type": "string" }
        },
        "input_size": {
          "type": "integer"
        }
      }
    }
  }
}
```

---

*File Specifications v1.0 — Reference document for agents and verification*
