# R&R Peer Review — Help

A general-purpose multi-agent blind peer review system for iterative improvement
of documents, vaults, codebases, and multi-file projects.

---

## Commands

| Command | Description |
|---------|-------------|
| `/peer-review <target>` | Start peer review of a target (file, dir, manifest) |
| `/peer-review --help` | Show this help message |
| `/peer-review --trust` | Skip pattern checks (rapid iteration mode) |
| `/peer-review --resume` | Resume an interrupted session |
| `/peer-review --max-iter N` | Set maximum iterations (default: 10) |
| `/peer-review --type <type>` | Override auto-detected input type |
| `/peer-review --roster <name>` | Select reviewer roster (default: software) |
| `/peer-review --cleanup` | Remove sessions older than 30 days |
| `/peer-review --cleanup --days N` | Remove sessions older than N days |

---

## Input Types

| Type | Description | Example |
|------|-------------|---------|
| `single-file` | One document | `./my-prd.md` |
| `vault` | Directory of Markdown files | `./prospectus/` |
| `codebase` | Source code project | `./src/` |
| `manifest` | Curated file list | `./.rnr-manifest.json` |

Input type is auto-detected but can be overridden with `--type`.

---

## Reviewer Rosters

| Roster | Reviewers | Best For |
|--------|-----------|----------|
| `software` ★ | Technical, Product, Ops | PRDs, RFCs, specs |
| `business` | Financial, Market, Risk | Business plans |
| `code` | Quality, Security, Architecture | Codebases |
| `academic` | Methodology, Literature, Ethics | Papers |
| `creative` | Craft, Narrative, Audience | Writing |
| `legal` | Compliance, Precedent, Risk | Contracts |
| `flexi` | User-defined | Anything |

★ = default roster

### Custom Roster

Place a `.rnr-reviewers.yaml` in your project root to define custom reviewers.
Copy `rosters/flexi.yaml` as a starting template.

---

## Example Sessions

### Single File
```
/peer-review ./my-prd.md

📂 Session: 20260131-101500-a1b2
📊 Input: ./my-prd.md (single-file, 247 words)
🎭 Roster: software (Technical, Product, Operations)

ITER 1: FAIL/FAIL/PASS → REVISE(4)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT
```

### Vault with Business Roster
```
/peer-review ./prospectus/ --roster business

📂 Session: 20260131-110000-c3d4
📊 Input: ./prospectus/ (vault, 12 files, 8200 words)
🎭 Roster: business (Financial Analyst, Market Strategist, Risk Assessor)

ITER 1: FAIL/PASS/FAIL → REVISE(5)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT
```

### Codebase
```
/peer-review ./src/ --type codebase --roster code

📂 Session: 20260131-120000-e5f6
📊 Input: ./src/ (codebase, 24 files, 6100 words)
🎭 Roster: code (Code Quality, Security Auditor, Architecture)

ITER 1: PASS/FAIL/PASS → REVISE(2)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT
```

---

## How It Works

1. **Submit** — You provide a target for review
2. **Adapt** — System detects input type and assembles a Review Package
3. **Validate** — Checks content meets requirements
4. **Review** — N blind reviewers (from your roster) evaluate independently
5. **Synthesize** — Area Chair consolidates feedback with domain-specific quality checks
6. **Decide** — ACCEPT (done), REVISE (iterate), or ESCALATE (max iterations)

---

## Document Requirements

| Requirement | Value |
|-------------|-------|
| Minimum length | 50 words |
| Maximum length | ~50,000 tokens (soft limit) |
| Recommended | 200-2,000 words per document |
| Format | Markdown preferred |
| Encoding | UTF-8 |

---

## Verdict Meanings

| Verdict | Meaning |
|---------|---------|
| PASS | Reviewer approves (no CRITICAL, <2 MAJOR) |
| FAIL | Reviewer rejects (any CRITICAL or 2+ MAJOR) |
| ACCEPT | All reviewers pass, document approved |
| REVISE | Issues found, author must address them |
| ESCALATE | Max iterations reached, human decides |

---

## Issue Severity

| Level | Definition |
|-------|------------|
| **CRITICAL** | Fundamental flaw, blocks approval |
| **MAJOR** | Significant issue, must be addressed |
| **MINOR** | Small issue, should be fixed |
| **SUGGESTION** | Nice-to-have, optional |

---

## Session Files

```
.peer_review/{session_id}/
├── state.json              # Session state
├── input/                  # Original input
│   ├── original.md         # (single-file)
│   └── package/            # (multi-file inputs)
├── iter_N/                 # Iteration files
│   ├── document.md
│   ├── review_{name}.md    # Per-reviewer output
│   └── synthesis.md        # Area Chair synthesis
└── final/document.md       # Approved version
```

---

## Tips

- **Vault reviews** — Obsidian vaults and wiki PRDs work great
- **Use `--roster`** — Match reviewers to your content domain
- **Custom rosters** — Drop `.rnr-reviewers.yaml` in your project root
- **Use `--trust`** — Skip security checks during rapid iteration
- **Use `--resume`** — Continue if your session is interrupted
- **Read reviews** — Each reviewer provides specific, actionable feedback

---

*R&R Peer Review v2.0*
