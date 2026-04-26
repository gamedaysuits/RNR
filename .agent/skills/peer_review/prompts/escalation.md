<!-- Prepend _security_header.md before use -->

# Escalation Handler — Human Intervention Required

This prompt is triggered when the maximum number of iterations has been reached without achieving ACCEPT verdict.

---

## Context

- **Session ID:** {{SESSION_ID}}
- **Document:** {{DOCUMENT_PATH}}
- **Iterations Completed:** {{MAX_ITERATIONS}}
- **Final Verdict:** REVISE (unresolved issues remain)

---

## Your Responsibilities

### 1. Summarize Iteration History

Provide a concise summary of what was attempted:

- How many iterations occurred
- What issues were raised
- What was addressed
- Why cycle didn't converge

### 2. List Remaining Issues

Clearly enumerate the unresolved issues that prevented acceptance:

- Severity and description
- Source reviewer
- Why it remains unresolved (if known)

### 3. Present Human Options

The human coordinator must decide next steps:

| Option | Action |
|--------|--------|
| **[1] Accept with Known Issues** | Mark session as accepted despite remaining issues. Issues become documented technical debt. |
| **[2] Provide Manual Guidance** | Human provides specific direction on how to resolve remaining issues. Session continues. |
| **[3] Abort Session** | Terminate the review process. Document is not approved. |

### 4. Prepare Audit Trail

Generate complete session history for human review using `templates/audit_trail.md`.

---

## Output Format

```markdown
# ⚠️ Escalation — Human Intervention Required

**Session:** {{SESSION_ID}}
**Document:** {{DOCUMENT_PATH}}
**Iterations:** {{MAX_ITERATIONS}} (maximum reached)

---

## Summary

After {{MAX_ITERATIONS}} iteration(s), the review cycle has not converged on ACCEPT.

### Iteration History

| Iter | Technical | Product | Ops | Verdict | Issues |
|------|-----------|---------|-----|---------|--------|
| 1 | {{V}} | {{V}} | {{V}} | {{V}} | {{N}} |
| 2 | {{V}} | {{V}} | {{V}} | {{V}} | {{N}} |
...

---

## Remaining Unresolved Issues

{{Numbered list of issues that blocked acceptance}}

1. [{{SEVERITY}}] {{Issue description}} — from {{Reviewer}}
2. [{{SEVERITY}}] {{Issue description}} — from {{Reviewer}}
...

---

## Human Decision Required

Please choose an option:

### [1] Accept with Known Issues
Accept the document in its current state. Remaining issues will be documented as known limitations/technical debt.

### [2] Provide Manual Guidance
Provide specific guidance on how to resolve the remaining issues. The review cycle will continue with your direction.

### [3] Abort Session
Terminate this review session. The document will not be approved.

---

**Awaiting your decision...**
```

---

## Tool Usage

When this escalation is triggered, use the `notify_user` tool to ensure the human sees this message and can respond with their choice.

---

*Escalation Prompt v1.0*
