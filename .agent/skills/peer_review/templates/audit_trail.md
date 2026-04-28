# Audit Trail — Session {{SESSION_ID}}

**Document:** {{DOCUMENT_PATH}}
**Input Type:** {{INPUT_TYPE}}
**Roster:** {{ROSTER_NAME}}
**Started:** {{CREATED_AT}}
**Status:** {{CURRENT_PHASE}}

---

## Session Configuration

| Setting | Value |
|---------|-------|
| Input Target | {{INPUT_TARGET}} |
| Input Type | {{INPUT_TYPE}} |
| Roster | {{ROSTER_NAME}} |
| Max Iterations | {{MAX_ITERATIONS}} |
| Trust Mode | {{TRUST_MODE}} |
| File Count | {{FILE_COUNT}} |

---

## Iteration History

| Iter | {{For each reviewer: reviewer.name}} | AC Verdict | Issues |
|------|{{For each reviewer: ----------|}}|------------|--------|
{{For each iteration: | N | reviewer verdicts... | ACCEPT/REVISE | N |}}

---

## Issue Evolution

### Iteration 1
{{Consolidated fix list from iteration 1}}

### Iteration 2
{{Consolidated fix list from iteration 2 — which issues resolved, which persisted}}

...

---

## Final State

**Total Iterations:** {{CURRENT_ITERATION}}
**Final Verdict:** {{FINAL_VERDICT}}
**Unresolved Issues:** {{REMAINING_ISSUES_COUNT}}

{{If escalated: list remaining unresolved issues}}

---

## Document Versions

| Version | Iteration | Hash |
|---------|-----------|------|
{{For each version: | VN | N | hash |}}

---

*Audit Trail Template v2.0*
