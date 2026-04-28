# Area Chair Synthesis — Iteration {{ITERATION}}

**Session:** {{SESSION_ID}}
**Roster:** {{ROSTER_NAME}}

## Meta-Quality Gate

| Check | Status | Notes |
|-------|--------|-------|
{{For each meta_gate check: | check.name | PASS/CONCERN | brief explanation |}}

## Review Summary

| Reviewer | Verdict | Critical | Major | Minor |
|----------|---------|----------|-------|-------|
{{For each reviewer: | reviewer.name | PASS/FAIL | N | N | N |}}

## Consolidated Fix List

{{If REVISE: numbered list of unique issues, deduplicated across reviewers}}

1. [{{SEVERITY}}] {{Issue description}} — from {{Reviewer}}
2. [{{SEVERITY}}] {{Issue description}} — from {{Reviewer}}
...

## Verdict: {{ACCEPT|REVISE}}

{{If ACCEPT: "Document approved. No outstanding issues."}}
{{If REVISE: "Address the above issues and resubmit for re-review."}}
