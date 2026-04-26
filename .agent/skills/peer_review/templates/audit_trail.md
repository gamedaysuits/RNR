# R&R Audit Trail

**Session:** {{SESSION_ID}}
**Document:** {{DOCUMENT_PATH}}
**Started:** {{START_TIMESTAMP}}
**Ended:** {{END_TIMESTAMP}}

---

## Session Configuration

| Setting | Value |
|---------|-------|
| Max Iterations | {{MAX_ITERATIONS}} |
| Trust Mode | {{true/false}} |
| Word Count | {{WORD_COUNT}} |

---

## Iteration History

| Iter | Technical | Product | Ops | AC Verdict | Issues Fixed | Issues Remaining |
|------|-----------|---------|-----|------------|--------------|------------------|
| 1 | {{PASS/FAIL}} | {{PASS/FAIL}} | {{PASS/FAIL}} | {{ACCEPT/REVISE}} | - | {{N}} |
| 2 | {{PASS/FAIL}} | {{PASS/FAIL}} | {{PASS/FAIL}} | {{ACCEPT/REVISE}} | {{N}} | {{N}} |
| ... | ... | ... | ... | ... | ... | ... |

---

## Final Status: {{ACCEPTED|ESCALATED|ABORTED}}

{{If ACCEPTED:}}
✅ Document approved after {{N}} iteration(s).

{{If ESCALATED:}}
⚠️ Maximum iterations ({{MAX_ITERATIONS}}) reached. Human intervention required.

{{If ABORTED:}}
❌ Session aborted by user.

---

## Remaining Issues (if escalated)

{{List any unresolved issues that triggered escalation}}

1. [{{SEVERITY}}] {{Issue description}} — from {{Reviewer}}
2. [{{SEVERITY}}] {{Issue description}} — from {{Reviewer}}
...

---

## Files Generated

### Input
- `.peer_review/{{SESSION_ID}}/input/original.md`

### Iterations
{{For each iteration:}}
- `.peer_review/{{SESSION_ID}}/iter_{{N}}/document.md`
- `.peer_review/{{SESSION_ID}}/iter_{{N}}/review_technical.md`
- `.peer_review/{{SESSION_ID}}/iter_{{N}}/review_product.md`
- `.peer_review/{{SESSION_ID}}/iter_{{N}}/review_ops.md`
- `.peer_review/{{SESSION_ID}}/iter_{{N}}/synthesis.md`

### Final Output
{{If ACCEPTED:}}
- `.peer_review/{{SESSION_ID}}/final/document.md`

---

## Security Log Summary

| Event Type | Count |
|------------|-------|
| validation_pass | {{N}} |
| validation_fail | {{N}} |
| pattern_match | {{N}} |
| escalation | {{N}} |

{{If any pattern_match events:}}
### Patterns Detected
{{List of patterns that were flagged during review}}

---

*Audit trail generated {{END_TIMESTAMP}}*
