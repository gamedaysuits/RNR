# Template Variables Registry

> **Purpose:** Master list of all placeholder variables used across prompts, templates, and roster configurations.

---

## Core Session Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `{{SESSION_ID}}` | Generated | Session identifier (YYYYMMDD-HHMMSS-random4) |
| `{{DOCUMENT_PATH}}` | User input | Path to the target file/directory |
| `{{ITERATION}}` | State | Current iteration number |
| `{{TIMESTAMP}}` | Generated | ISO 8601 timestamp |
| `{{CURRENT_PHASE}}` | State | Current phase (validate, review, synthesis, complete, escalated) |
| `{{CREATED_AT}}` | State | Session creation timestamp |
| `{{MAX_ITERATIONS}}` | Config | Maximum allowed iterations |
| `{{TRUST_MODE}}` | Flag | Whether pattern checks are skipped |

---

## Input Adapter Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `{{INPUT_TARGET}}` | User input | Raw target path/file as specified by user |
| `{{INPUT_TYPE}}` | Adapter | Detected or overridden input type (single-file, vault, codebase, manifest) |
| `{{FILE_COUNT}}` | Adapter | Number of files in the Review Package |
| `{{TOTAL_WORD_COUNT}}` | Adapter | Combined word count of all files |

---

## Roster Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `{{ROSTER_NAME}}` | Roster YAML | Name of the active roster (e.g., "software") |
| `{{REVIEWER_LIST}}` | Roster YAML | Formatted list of reviewer names and roles |
| `{{META_GATE_CHECKS}}` | Roster YAML | Formatted meta-quality gate checks from roster |

---

## Reviewer Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `{{PERSONA_NAME}}` | Roster YAML | Reviewer's short name (e.g., "Technical") |
| `{{PERSONA_TITLE}}` | Roster YAML | Reviewer's full title (e.g., "Senior Technical Architect") |
| `{{PERSONA_ROLE}}` | Roster YAML | Reviewer's role description |
| `{{FOCUS_AREAS}}` | Roster YAML / Prompt | Reviewer's evaluation focus areas (markdown) |
| `{{SEVERITY_DEFINITIONS}}` | Roster YAML / Prompt | Severity level definitions table |
| `{{REVIEW_GUIDELINES}}` | Roster YAML / Prompt | Numbered guideline list |

---

## Verdict Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `{{PASS\|FAIL}}` | Reviewer output | Individual reviewer verdict |
| `{{ACCEPT\|REVISE}}` | Area Chair | Final synthesis verdict |
| `{{SEVERITY}}` | Issue | Issue severity level (CRITICAL, MAJOR, MINOR, SUGGESTION) |
| `{{REMAINING_ISSUES_COUNT}}` | Escalation | Count of unresolved issues at escalation |
| `{{FINAL_VERDICT}}` | State | Session final outcome |

---

## Template-Only Variables

These appear only in output templates and are filled by the agent at runtime:

| Variable | Template | Description |
|----------|----------|-------------|
| `{{REVIEWER}}` | synthesis_output | Name of the reviewer who raised an issue |
| `{{ISSUES_COUNT}}` | synthesis_output | Total consolidated issue count |

---

## Source Resolution Order

When a variable appears in both the roster YAML and the prompt file, the resolution order is:

1. **Roster YAML inline values** (e.g., `focus_areas` in flexi roster) — highest priority
2. **Prompt file defaults** — used if roster doesn't specify
3. **Session state** — runtime values from the active session

---

*Variables Registry v2.0*
