---
name: peer-review
description: Multi-agent blind peer review with iterative refinement
version: "2.0"
---

# R&R Peer Review Skill

> A general-purpose multi-agent blind peer review system that iteratively improves
> documents, vaults, codebases, and multi-file projects through configurable
> reviewer panels and domain-specific feedback.

---

## Command Reference

| Command | Description |
|---------|-------------|
| `/peer-review <target>` | Start peer review of target (file, directory, manifest) |
| `/peer-review --help` | Show help and examples |
| `/peer-review --trust` | Skip pattern checks (rapid iteration) |
| `/peer-review --resume` | Resume interrupted session |
| `/peer-review --max-iter N` | Set max iterations (default: 10) |
| `/peer-review --type <type>` | Override auto-detected input type |
| `/peer-review --roster <name>` | Select reviewer roster (default: software) |
| `/peer-review --cleanup [--days N]` | Remove old sessions (default: 30 days) |

### Input Types (`--type`)

| Type | Description |
|------|-------------|
| `single-file` | One document (`.md`, `.txt`) |
| `vault` | Directory of Markdown files (Obsidian vaults, wiki PRDs) |
| `codebase` | Source code project directory |
| `manifest` | User-curated `.rnr-manifest.json` file |

### Reviewer Rosters (`--roster`)

| Roster | Reviewers | Best For |
|--------|-----------|----------|
| `software` (default) | Technical, Product, Ops | PRDs, RFCs, specs |
| `business` | Financial, Market, Risk | Business plans, pitch decks |
| `code` | Quality, Security, Architecture | Codebases, modules |
| `academic` | Methodology, Literature, Ethics | Papers, proposals |
| `creative` | Craft, Narrative, Audience | Writing, scripts |
| `legal` | Compliance, Precedent, Risk | Contracts, policies |
| `flexi` | User-defined (template) | Anything |
| Custom | `.rnr-reviewers.yaml` in project root | Your domain |

---

## Core Protocol

```
Validate → Adapt → Author → Reviews(N) → Area Chair → Verdict
                                ↑                        ↓
                                └────── REVISE ──────────┘
```

### Protocol Flow

1. **Validate** — Parse arguments, detect input type, load roster
2. **Adapt** — Run input adapter to assemble Review Package
3. **Author** — Acknowledge receipt, prepare V1 for review
4. **Reviews** — N blind reviewers (from roster) evaluate independently
5. **Area Chair** — Synthesize reviews, run domain-specific meta-quality gate, issue verdict
6. **Verdict** — ACCEPT (done) | REVISE (loop back) | ESCALATE (max iterations hit)

---

## Constants

```
MAX_ITERATIONS_DEFAULT = 10
MIN_WORD_COUNT = 50
RETENTION_DAYS_DEFAULT = 30
SESSION_ID_FORMAT = "{YYYYMMDD}-{HHMMSS}-{random4}"
DEFAULT_ROSTER = "software"
DEFAULT_ADAPTER = "single-file"
MAX_TOKENS_SOFT = 50000
MIN_REVIEWERS = 2
MAX_REVIEWERS = 5
```

---

## Session ID Generation

Format: `{YYYYMMDD}-{HHMMSS}-{random4}`

Example: `20260131-101500-a1b2`

---

## State Management

### Session Directory Structure

```
.peer_review/{session_id}/
├── state.json              # Session state
├── state.json.bak          # Backup of previous state
├── input/
│   ├── original.md         # Original document copy (single-file)
│   └── package/            # Review package (multi-file inputs)
│       ├── _manifest.json  # Assembled package metadata
│       └── *.md            # Copied source files
├── iter_1/
│   ├── document.md         # V1 (or V{n})
│   ├── review_{name_1}.md  # Reviewer 1 output (name from roster)
│   ├── review_{name_2}.md  # Reviewer 2 output
│   ├── review_{name_3}.md  # Reviewer 3 output
│   └── synthesis.md        # Area Chair synthesis
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
  "input_target": "/path/to/input",
  "input_type": "vault",
  "roster": "software",
  "created_at": "2026-01-31T10:15:00Z",
  "updated_at": "2026-01-31T10:30:00Z",
  "current_iteration": 1,
  "current_phase": "review",
  "max_iterations": 10,
  "trust_mode": false,
  "file_count": 8,
  "total_word_count": 4500,
  "reviewers": ["Technical", "Product", "Operations"],
  "versions": {
    "V1": "hash_abc123"
  },
  "iterations": [
    {
      "iteration": 1,
      "verdicts": {
        "Technical": "FAIL",
        "Product": "PASS",
        "Operations": "FAIL"
      },
      "ac_verdict": "REVISE",
      "issues_count": 6
    }
  ]
}
```

### Atomic Write Protocol

1. Write new content to `state.json.tmp`
2. If `state.json` exists, copy to `state.json.bak`
3. Rename `state.json.tmp` to `state.json` (atomic on POSIX)
4. **Windows fallback:** Copy then delete (not atomic, but functional)

### Recovery Protocol

1. Check if `state.json` exists → load and continue
2. If missing but `state.json.bak` exists → restore from backup
3. If neither exists → session is lost, notify user

---

## Phase Transitions

### Phase 0: Validate & Adapt

**Trigger:** User invokes `/peer-review <target>`

**Actions:**
1. Parse command arguments (target, flags)
2. **Resolve roster:** `--roster` flag → manifest `roster` field → `.rnr-reviewers.yaml` → default (`software`)
3. Load roster YAML and validate (2-5 reviewers)
4. **Detect input type:** `--type` flag → auto-detection (see `adapters/_adapter_protocol.md`)
5. **Run input adapter:** Assemble Review Package (see adapter docs)
6. Validate Review Package: word count ≥ 50, encoding checks
7. If not `--trust` mode: Scan for suspicious patterns (see `validation/patterns.md`)
8. Log validation result
9. On failure: Display rejection message
10. On success: Generate session ID, create session directory, store input

### Phase 1: Author Initial

**Trigger:** Validation and adaptation passed

**Actions:**
1. Load `prompts/_security_header.md` + `prompts/author_initial.md`
2. Present Review Package to Author persona
3. Author acknowledges receipt, reports word count, file count, estimated review time
4. Output: "Document V1 ready for peer review."
5. Update state: `current_phase = "review"`

### Phase 2: Blind Review

**Trigger:** Author completed

**Actions:**
1. Load active roster from state
2. For each reviewer in roster:
   a. Load `prompts/_security_header.md` + reviewer's prompt file (from roster YAML)
   b. CRITICAL: Include memory firewall — reviewer must NOT see other reviews
   c. Execute review, save to `iter_{n}/review_{reviewer_name}.md`
   d. Parse verdict (PASS/FAIL) from output
3. Update state with all verdicts

### Phase 3: Area Chair Synthesis

**Trigger:** All reviews complete

**Actions:**
1. Load `prompts/_security_header.md` + `prompts/area_chair.md`
2. Inject `{{REVIEWER_LIST}}` from roster (names and roles)
3. Inject `{{META_GATE_CHECKS}}` from roster's `meta_gate` field
4. Load all reviewer outputs as context
5. Perform Meta-Quality Gate (domain-specific checks from roster)
6. Consolidate issues by severity
7. Deduplicate overlapping issues
8. Issue verdict: ACCEPT or REVISE with numbered fix list
9. Save to `iter_{n}/synthesis.md`
10. Update state

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
    → Notify user with options:
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
📊 Input: ./prospectus/ (vault, 8 files, 4500 words)
🎭 Roster: software (Technical, Product, Operations)

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

### Adapters

| File | Purpose |
|------|---------|
| `adapters/_adapter_protocol.md` | Adapter interface contract and auto-detection logic |
| `adapters/single_file.md` | Single document adapter (V1 compatible) |
| `adapters/vault.md` | Obsidian vault / wiki PRD adapter (primary) |
| `adapters/codebase.md` | Source code project adapter |
| `adapters/manifest.md` | User-curated `.rnr-manifest.json` adapter |

### Rosters

| File | Purpose |
|------|---------|
| `rosters/_roster_protocol.md` | Roster configuration schema and resolution |
| `rosters/software.yaml` | Default: Technical, Product, Ops |
| `rosters/business.yaml` | Financial, Market, Risk |
| `rosters/code.yaml` | Quality, Security, Architecture |
| `rosters/academic.yaml` | Methodology, Literature, Ethics |
| `rosters/creative.yaml` | Craft, Narrative, Audience |
| `rosters/legal.yaml` | Compliance, Precedent, Legal Risk |
| `rosters/flexi.yaml` | User-defined template |

### Prompts (prepend _security_header.md to each)

| File | Purpose |
|------|---------|
| `prompts/_security_header.md` | Security preamble for all personas |
| `prompts/author_initial.md` | First-time document submission |
| `prompts/author_revision.md` | Post-R&R revision handling |
| `prompts/reviewer_template.md` | Parameterized base for custom reviewers |
| `prompts/reviewer_technical.md` | Technical feasibility (software) |
| `prompts/reviewer_product.md` | Product/UX (software) |
| `prompts/reviewer_ops.md` | Operations/security (software) |
| `prompts/reviewer_financial_analyst.md` | Financial viability (business) |
| `prompts/reviewer_market_strategist.md` | Market opportunity (business) |
| `prompts/reviewer_risk_assessor.md` | Risk identification (business) |
| `prompts/reviewer_code_quality.md` | Code quality (code) |
| `prompts/reviewer_security_auditor.md` | Security audit (code) |
| `prompts/reviewer_architecture.md` | Architecture (code) |
| `prompts/reviewer_methodology.md` | Methodology (academic) |
| `prompts/reviewer_literature.md` | Literature (academic) |
| `prompts/reviewer_ethics.md` | Ethics (academic) |
| `prompts/reviewer_craft.md` | Writing craft (creative) |
| `prompts/reviewer_narrative.md` | Narrative structure (creative) |
| `prompts/reviewer_audience.md` | Audience fit (creative) |
| `prompts/reviewer_compliance.md` | Compliance (legal) |
| `prompts/reviewer_precedent.md` | Legal precedent (legal) |
| `prompts/reviewer_legal_risk.md` | Liability/risk (legal) |
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

### Other

| File | Purpose |
|------|---------|
| `VARIABLES.md` | Master registry of template placeholders |
| `logs/security.log.schema.json` | Security log format definition |

---

## Error Handling

### Graceful Failures

1. **Validation errors** → Display rejection message, do NOT create session
2. **File/directory not found** → Display error, suggest checking path
3. **Roster not found** → Display available rosters, suggest `--roster` flag
4. **Adapter failure** → Display error with input type, suggest `--type` override
5. **Pattern detection** → Log warning, CONTINUE review (don't block)
6. **State corruption** → Attempt recovery from .bak, notify user if impossible
7. **Max iterations** → Escalate to human, provide clear options

### Logging Protocol

All significant events logged to `logs/security.log`:

```json
{
  "timestamp": "2026-01-31T10:15:00Z",
  "session_id": "20260131-101500-a1b2",
  "event": "validation_pass",
  "details": {
    "input_type": "vault",
    "input_target": "./prospectus/",
    "file_count": 8,
    "total_word_count": 4500,
    "roster": "software",
    "patterns_matched": []
  }
}
```

Event types: `validation_pass`, `validation_fail`, `pattern_match`, `trust_bypass`, `adapter_error`, `roster_load`, `escalation`

---

## Custom Roster (`.rnr-reviewers.yaml`)

Place a `.rnr-reviewers.yaml` file in your project root to define a custom reviewer panel. See `rosters/_roster_protocol.md` for the full schema and `rosters/flexi.yaml` for a template to copy and customize.

Quick start:
1. Copy `rosters/flexi.yaml` to your project root as `.rnr-reviewers.yaml`
2. Edit reviewer names, focus areas, and meta-gate checks
3. Run `/peer-review` — it will auto-detect the custom roster

---

## Known Limitations

1. **Memory Firewall = best-effort** — LLM context may persist between calls
2. **Pattern matching ≠ comprehensive** — Sophisticated injections may bypass
3. **POSIX atomicity only** — Windows fallback is less safe
4. **Subjective quality** — Meta-assessments involve judgment calls
5. **Context limits** — Large vaults/codebases may exceed model context windows
6. **Single-agent execution** — Reviews run sequentially in one agent context, not parallel across separate agents

---

*SKILL.md v2.0 — R&R Peer Review Orchestration*
