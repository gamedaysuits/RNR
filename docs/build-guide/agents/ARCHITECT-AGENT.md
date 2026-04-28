# Architect Agent — Foundation & Structure

> **Persona:** Senior Systems Architect  
> **Responsibility:** Directory structure, state management, SKILL.md orchestration  
> **Phase:** 1 (Foundation)

---

## Mission

Build the structural foundation of the R&R skill:
- Complete directory structure (17 files)
- SKILL.md orchestration logic
- State management with atomic writes
- Session recovery mechanisms

---

## Deliverables

| File | Lines | Priority |
|------|-------|----------|
| SKILL.md | 200-400 | CRITICAL |
| Directory structure | 17 files | CRITICAL |
| state.json schema | ~50 | HIGH |
| security.log.schema.json | ~20 | HIGH |

---

## SKILL.md Requirements

### YAML Frontmatter
```yaml
---
name: peer-review
description: Multi-agent blind peer review with iterative refinement
---
```

### Command Interface
```markdown
## Commands

- `/peer-review <document>` - Start review
- `/peer-review --help` - Show help
- `/peer-review --trust` - Skip pattern checks
- `/peer-review --resume` - Resume interrupted session
- `/peer-review --max-iter N` - Set max iterations (default: 10)
- `/peer-review --cleanup [--days N]` - Remove old sessions
```

### Phase Flow
```markdown
## Protocol

1. **VALIDATE** - Input checks, pattern scan, session init
2. **AUTHOR** - Prepare document for review
3. **REVIEW** - Execute 3 blind reviewers sequentially
4. **SYNTHESIZE** - Area Chair consolidates
5. **VERDICT** - ACCEPT, REVISE, or ESCALATE
```

### State Management
```markdown
## State Persistence

### Atomic Write Protocol
1. Write to `state.json.tmp`
2. Backup existing: `state.json` → `state.json.bak`
3. Rename: `state.json.tmp` → `state.json`

### Windows Fallback
1. Copy `state.json` to `state.json.bak`
2. Write new content to `state.json.tmp`
3. Delete `state.json`
4. Rename `state.json.tmp` → `state.json`

### Recovery
If `state.json` missing but `.bak` exists → restore from `.bak`
```

---

## State Schema

```json
{
  "session_id": "20260131-101500-a1b2",
  "document_path": "/path/to/doc.md",
  "created_at": "2026-01-31T10:15:00Z",
  "updated_at": "2026-01-31T10:30:00Z",
  "current_iteration": 1,
  "current_phase": "review",
  "max_iterations": 10,
  "trust_mode": false,
  "versions": {
    "V1": { "path": "V1.md", "hash": "abc123" },
    "V2": { "path": "V2.md", "hash": "def456" }
  },
  "iterations": [
    {
      "iteration": 1,
      "reviews": {
        "technical": { "verdict": "FAIL", "issues": 2 },
        "product": { "verdict": "PASS", "issues": 0 },
        "ops": { "verdict": "FAIL", "issues": 3 }
      },
      "ac_verdict": "REVISE",
      "total_issues": 5
    }
  ]
}
```

---

## Session ID Generation

Format: `{YYYYMMDD}-{HHMMSS}-{random4}`

```javascript
function generateSessionId() {
  const now = new Date();
  const date = now.toISOString().slice(0,10).replace(/-/g, '');
  const time = now.toTimeString().slice(0,8).replace(/:/g, '');
  const rand = Math.random().toString(36).substring(2, 6);
  return `${date}-${time}-${rand}`;
}
// Example: "20260131-101500-x7k2"
```

---

## Directory Layout

```
.peer_review/                          # Runtime data
└── {session_id}/
    ├── state.json
    ├── state.json.bak
    ├── V1.md
    ├── V2.md
    ├── iter_1/
    │   ├── review_technical.md
    │   ├── review_product.md
    │   ├── review_ops.md
    │   └── synthesis.md
    ├── iter_2/
    │   └── ...
    ├── final/
    │   └── document.md
    └── logs/
        └── security.log
```

---

## Quality Checklist

Before handing off to next phase:

- [ ] All 17 skill files exist with headers
- [ ] SKILL.md contains complete orchestration logic
- [ ] State schema is fully defined
- [ ] Atomic write protocol documented
- [ ] Recovery mechanism tested
- [ ] Session ID format verified

---

*Architect Agent Spec v1.0*
