# R&R Peer Review — Help

A multi-agent blind peer review system for iterative document improvement.

---

## Commands

| Command | Description |
|---------|-------------|
| `/peer-review <doc>` | Start peer review of a document |
| `/peer-review --help` | Show this help message |
| `/peer-review --trust` | Skip pattern checks (rapid iteration mode) |
| `/peer-review --resume` | Resume an interrupted session |
| `/peer-review --max-iter N` | Set maximum iterations (default: 10) |
| `/peer-review --cleanup` | Remove sessions older than 30 days |
| `/peer-review --cleanup --days N` | Remove sessions older than N days |

---

## Example Session

```
/peer-review ./my-prd.md

📂 Session: 20260131-101500-a1b2
📊 Document: 247 words (~2 min)

ITER 1: FAIL/FAIL/PASS → REVISE(4)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT

Output: .peer_review/20260131-101500-a1b2/final/document.md
```

---

## How It Works

1. **Submit** — You provide a document for review
2. **Validate** — System checks document meets requirements
3. **Review** — Three blind reviewers evaluate independently:
   - Technical Architect (feasibility, architecture)
   - Product Manager (user experience, scope)
   - DevOps/Security (operations, security, timeline)
4. **Synthesize** — Area Chair consolidates feedback
5. **Decide** — ACCEPT (done) or REVISE (iterate)

---

## Document Requirements

| Requirement | Value |
|-------------|-------|
| Minimum length | 50 words |
| Maximum length | ~50,000 tokens (soft limit) |
| Recommended | 200-2,000 words |
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

Your review session creates files in `.peer_review/{session_id}/`:

```
.peer_review/{session_id}/
├── state.json              # Session state
├── input/original.md       # Your original document
├── iter_1/                 # Iteration 1 files
│   ├── document.md
│   ├── review_technical.md
│   ├── review_product.md
│   ├── review_ops.md
│   └── synthesis.md
└── final/document.md       # Approved final version
```

---

## Tips

- **Write clear PRDs** — Well-structured documents get faster approvals
- **Use --trust** — Skip security checks during rapid iteration
- **Use --resume** — Continue if your session is interrupted
- **Read reviews** — Each reviewer provides specific, actionable feedback
- **Iterate quickly** — Address all issues in each revision

---

## More Information

See `END-USER-GUIDE.md` for detailed usage instructions.
See `TROUBLESHOOTING.md` for common issues and solutions.

---

*R&R Peer Review v1.0*
