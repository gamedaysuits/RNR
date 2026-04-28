# R&R Peer Review — End User Guide

> **For:** Users who want to USE the R&R skill (not build it)  
> **Assumes:** Skill is already installed in your agent environment

---

## Quick Start

### Review a Document
```
/peer-review ./my-document.md
```

### Review an Obsidian Vault
```
/peer-review ./my-vault/
```

### Review with a Domain-Specific Roster
```
/peer-review ./business-plan.md --roster business
```

That's it. The system will:
1. Detect your input type (file, vault, codebase, manifest)
2. Load the appropriate reviewer roster
3. Run N blind reviews
4. Synthesize feedback with domain-specific quality checks
5. Either ACCEPT or request revisions

---

## Commands

| Command | What it does |
|---------|--------------|
| `/peer-review <target>` | Start a review (file, directory, or manifest) |
| `/peer-review --help` | Show help |
| `/peer-review --trust` | Skip security pattern checks |
| `/peer-review --resume` | Resume interrupted session |
| `/peer-review --max-iter N` | Limit iterations (default: 10) |
| `/peer-review --type <type>` | Override auto-detected input type |
| `/peer-review --roster <name>` | Select reviewer roster (default: software) |
| `/peer-review --cleanup` | Remove old sessions (>30 days) |

---

## Input Types

The system auto-detects what you're reviewing:

| Type | When | Example |
|------|------|---------|
| **Single File** | Target is a file | `./my-prd.md` |
| **Vault** | Directory of Markdown files | `./prospectus/` |
| **Codebase** | Directory with source code | `./src/` |
| **Manifest** | `.rnr-manifest.json` file | `./.rnr-manifest.json` |

Override with `--type`: `/peer-review ./docs/ --type vault`

---

## Reviewer Rosters

Choose a roster that matches your content:

| Roster | Reviewers | Best For |
|--------|-----------|----------|
| `software` ★ | Technical, Product, Ops | PRDs, RFCs, specs |
| `business` | Financial, Market, Risk | Business plans, pitch decks |
| `code` | Quality, Security, Architecture | Codebases, PRs |
| `academic` | Methodology, Literature, Ethics | Papers, theses |
| `creative` | Craft, Narrative, Audience | Writing, scripts |
| `legal` | Compliance, Precedent, Risk | Contracts, policies |
| `flexi` | User-defined | Anything |

★ = default (used when no `--roster` is specified)

### Custom Rosters

Create your own reviewer panel:

1. Copy `rosters/flexi.yaml` to your project root as `.rnr-reviewers.yaml`
2. Edit the reviewer names, focus areas, and meta-gate checks
3. Run `/peer-review` — your custom roster is auto-detected

---

## Content Requirements

| Requirement | Value |
|-------------|-------|
| **Minimum length** | 50 words |
| **Recommended length** | 200-2000 words per document |
| **Format** | Markdown (.md) preferred |
| **Encoding** | UTF-8 |

For vaults and codebases, the combined content across all included files is evaluated.

---

## What to Expect

### Sample Session — Vault + Business Roster

```
/peer-review ./prospectus/ --roster business

📂 Session: 20260131-110000-c3d4
📊 Input: ./prospectus/ (vault, 12 files, 8200 words)
🎭 Roster: business (Financial Analyst, Market Strategist, Risk Assessor)

--- Iteration 1 ---
🔍 Financial Analyst: FAIL (1 CRITICAL, 1 MAJOR)
🔍 Market Strategist: PASS
🔍 Risk Assessor: FAIL (2 MAJOR)

📋 Area Chair Synthesis:
   Meta-Gate: Market Thesis ✅ | Financial Viability ⚠️ | Execution ✅
   5 issues to address (1 CRITICAL, 3 MAJOR, 1 MINOR)

Verdict: REVISE

--- Revision ---
Addressing issues...

--- Iteration 2 ---
🔍 Financial Analyst: PASS
🔍 Market Strategist: PASS
🔍 Risk Assessor: PASS

📋 Area Chair Synthesis:
   Meta-Gate: All ✅
   0 issues remaining

Verdict: ✅ ACCEPT

📁 Output: .peer_review/20260131-110000-c3d4/final/document.md
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
- **ACCEPT:** All reviews passed. Done!
- **REVISE:** Issues found. Author makes fixes, re-review.
- **ESCALATE:** Max iterations reached. Human decides.

---

## Output Files

```
.peer_review/{session_id}/
├── state.json              # Session tracking
├── input/
│   ├── original.md         # (single-file)
│   └── package/            # (multi-file inputs)
├── iter_1/
│   ├── document.md
│   ├── review_{name}.md    # Per-reviewer output
│   └── synthesis.md
├── final/
│   └── document.md         # ← Your final document
└── logs/
    └── security.log
```

---

## Resuming Interrupted Sessions

```
/peer-review --resume

Found: 20260131-110000-c3d4 (Iteration 2, Phase: review)
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
| ⚠️ Content may exceed context | Very long content. May be truncated by LLM. |
| ⚠️ Roster not found | Specified roster doesn't exist. Check available options. |

---

## FAQ

### Q: How long does a review take?
**A:** Usually 2-5 minutes per iteration. Simple documents: 1 iteration. Complex vaults: 2-3 iterations.

### Q: Can I review code files?
**A:** Yes! Use `--roster code` and `--type codebase` for optimized code review.

### Q: Can I review a whole Obsidian vault?
**A:** Yes, this is a primary use case. Point the tool at your vault directory and it will assemble a review package from all Markdown files.

### Q: What if I disagree with a reviewer?
**A:** Use `--max-iter 1` and choose "Accept with known issues" when escalated.

### Q: Are my documents sent anywhere?
**A:** No. Everything runs locally in your agent session. Documents never leave your machine.

### Q: Can I customize the reviewers?
**A:** Yes! Create a `.rnr-reviewers.yaml` in your project root, or use the `flexi` roster.

### Q: Which roster should I use?
**A:** Match the roster to your content:
- Writing a PRD? → `software`
- Business plan? → `business`
- Reviewing code? → `code`
- Research paper? → `academic`
- Not sure? → `flexi` and customize

---

## Troubleshooting

See [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) for common issues.

---

*End User Guide v2.0*
