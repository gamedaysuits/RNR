# R&R Peer Review — End User Guide

> **For:** Users who want to USE the R&R skill (not build it)  
> **Assumes:** Skill is already installed in your Antigravity workspace

---

## Quick Start

```
/peer-review ./my-document.md
```

That's it. The system will:
1. Validate your document
2. Run 3 blind reviews (Technical, Product, Operations)
3. Synthesize feedback
4. Either ACCEPT or request revisions

---

## Commands

| Command | What it does |
|---------|--------------|
| `/peer-review <doc>` | Start a review |
| `/peer-review --help` | Show help |
| `/peer-review --trust` | Skip security pattern checks |
| `/peer-review --resume` | Resume interrupted session |
| `/peer-review --max-iter N` | Limit iterations (default: 10) |
| `/peer-review --cleanup` | Remove old sessions (>30 days) |

---

## Document Requirements

| Requirement | Value |
|-------------|-------|
| **Minimum length** | 50 words |
| **Recommended length** | 200-2000 words |
| **Format** | Markdown (.md) preferred |
| **Encoding** | UTF-8 |

---

## What to Expect

### Sample Session

```
/peer-review ./feature-prd.md

📂 Session: 20260131-101500-a1b2
📊 Document: 247 words (~3 min estimated)

Starting peer review...

--- Iteration 1 ---
🔍 Technical Review: FAIL (2 MAJOR issues)
🔍 Product Review: PASS
🔍 Operations Review: FAIL (1 CRITICAL issue)

📋 Area Chair Synthesis:
   3 issues to address (1 CRITICAL, 2 MAJOR)

Verdict: REVISE

--- Revision ---
Addressing issues...

--- Iteration 2 ---
🔍 Technical Review: PASS
🔍 Product Review: PASS
🔍 Operations Review: PASS

📋 Area Chair Synthesis:
   0 issues remaining

Verdict: ✅ ACCEPT

📁 Output: .peer_review/20260131-101500-a1b2/final/document.md
```

---

## Understanding Verdicts

### Reviewer Verdicts
- **PASS:** No critical issues, fewer than 2 major issues
- **FAIL:** At least 1 critical OR 2+ major issues

### Issue Severity
| Level | Meaning |
|-------|---------|
| CRITICAL | Must fix. Blocks acceptance. |
| MAJOR | Should fix. Significant issue. |
| MINOR | Nice to fix. Small improvement. |
| SUGGESTION | Optional. Food for thought. |

### Final Verdicts
- **ACCEPT:** Document passed all reviews. Done!
- **REVISE:** Issues found. Author makes fixes, re-review.
- **ESCALATE:** Max iterations reached. Human decides.

---

## Output Files

After a session, find your files at:

```
.peer_review/{session_id}/
├── state.json          # Session tracking
├── V1.md               # Original version
├── V2.md               # Revised version (if any)
├── iter_1/
│   ├── review_technical.md
│   ├── review_product.md
│   ├── review_ops.md
│   └── synthesis.md
├── final/
│   └── document.md     # ← Your final document
└── logs/
    └── security.log
```

---

## Resuming Interrupted Sessions

If your session is interrupted:

```
/peer-review --resume

Found: 20260131-101500-a1b2 (Iteration 2, Phase: review)
[1] Resume
[2] Abandon

> 1

Resuming session...
```

---

## Warnings You May See

| Warning | Meaning |
|---------|---------|
| ⚠️ Suspicious pattern detected | Document contains text that looks like prompt injection. Logged but not blocked. |
| ⚠️ Document may exceed context | Very long document. May be truncated by LLM. |

---

## FAQ

### Q: How long does a review take?
**A:** Usually 2-5 minutes per iteration. Simple documents: 1 iteration. Complex documents: 2-3 iterations.

### Q: Can I review code files?
**A:** The system is designed for prose documents (PRDs, RFCs, plans). Code files may work but aren't optimized.

### Q: What if I disagree with a reviewer?
**A:** You can use `--max-iter 1` and choose "Accept with known issues" when escalated.

### Q: Are my documents sent anywhere?
**A:** No. Everything runs locally in your Antigravity session. Documents never leave your machine.

### Q: Can I customize the reviewers?
**A:** Yes, edit the prompt files in `.agent/skills/peer_review/prompts/`.

---

## Troubleshooting

See [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) for common issues.

---

*End User Guide v1.0*
