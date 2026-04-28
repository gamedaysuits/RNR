# Manifest Adapter

> **Purpose:** Handle user-curated multi-file review packages via `.rnr-manifest.json`.
> Maximum user control — define exactly which files to include, their roles, and review context.

---

## Input

A path to a `.rnr-manifest.json` file.

```
/peer-review ./plan.rnr-manifest.json
/peer-review ./review-config.rnr-manifest.json
```

---

## Detection

This adapter is selected when:
- Input path ends with `.rnr-manifest.json`

---

## Manifest Schema

```json
{
  "title": "Q2 Business Plan Review",
  "description": "Review the business plan narrative against the financial model",
  "context": "Optional free-text context for reviewers about what to focus on",
  "roster": "business",
  "files": [
    {
      "path": "./business-plan.md",
      "role": "primary",
      "label": "Main narrative document"
    },
    {
      "path": "./financial-model.md",
      "role": "primary",
      "label": "Revenue projections and unit economics"
    },
    {
      "path": "./market-research.md",
      "role": "supporting",
      "label": "TAM/SAM/SOM analysis"
    },
    {
      "path": "./appendix/competitor-matrix.md",
      "role": "supporting",
      "label": "Competitive landscape reference"
    }
  ]
}
```

### Schema Fields

| Field | Required | Description |
|-------|----------|-------------|
| `title` | Optional | Human-readable title for the review session |
| `description` | Optional | Brief description of what's being reviewed |
| `context` | Optional | Free-text guidance for reviewers (prepended to review package) |
| `roster` | Optional | Override default roster (equivalent to `--roster` flag) |
| `files` | ✅ | Array of file objects to include |
| `files[].path` | ✅ | Relative path from manifest location to the file |
| `files[].role` | Optional | `primary` (default), `supporting`, `config`, `reference` |
| `files[].label` | Optional | Human-readable description of the file's purpose |

---

## Assembly Process

```
FUNCTION assemble_manifest(manifest_path):

  1. PARSE manifest JSON
     - Validate against schema
     - Resolve all file paths relative to manifest location

  2. VALIDATE files
     - Check each file exists and is readable
     - Reject if ANY required file is missing (list all missing files)
     - Warn for non-.md files

  3. READ AND ORDER content
     - Primary files first (in manifest order)
     - Supporting files second (in manifest order)
     - Config/reference files last

  4. ASSEMBLE REVIEW PACKAGE
     - Include manifest title, description, context as header
     - Concatenate files with boundary markers and labels

  RETURN Review Package:
    input_type: "manifest"
    input_target: manifest_path
    file_count: N
    total_word_count: sum of all file word counts
    files: [array from manifest with added word counts]
    content: assembled content
    metadata:
      manifest_title: title from manifest
      roster_override: roster from manifest (if specified)
      warnings: [missing optional files, size warnings]
```

---

## Output Format

```markdown
# Review Package: {{title}}

{{description}}

> **Reviewer Context:** {{context}}

**Files:** {{N}} documents
**Total Words:** {{word_count}}

---

## File Manifest

| # | File | Role | Label | Words |
|---|------|------|-------|-------|
| 1 | business-plan.md | primary | Main narrative document | 2400 |
| 2 | financial-model.md | primary | Revenue projections and unit economics | 1800 |
| 3 | market-research.md | supporting | TAM/SAM/SOM analysis | 900 |
| 4 | competitor-matrix.md | supporting | Competitive landscape reference | 600 |

---

## Documents

---
📄 **File: business-plan.md** (Role: primary | "Main narrative document" | 2400 words)
---

[file content]

---
📄 **File: financial-model.md** (Role: primary | "Revenue projections and unit economics" | 1800 words)
---

[file content]

...
```

---

## Manifest-Specific Validation

| Check | Action |
|-------|--------|
| Manifest is valid JSON | Reject with parse error |
| `files` array exists and is non-empty | Reject ("Manifest has no files") |
| All file paths resolve | Reject (list ALL missing files, not just first) |
| All files readable | Reject with specific file errors |
| Combined word count ≥ 50 | Reject |
| Total token estimate > 50k | Warn |

---

## Use Cases

### Business Plan + Financial Model
```json
{
  "title": "Series A Pitch Review",
  "roster": "business",
  "files": [
    { "path": "./pitch-deck-notes.md", "role": "primary", "label": "Pitch narrative" },
    { "path": "./financial-projections.md", "role": "primary", "label": "5-year model" },
    { "path": "./market-analysis.md", "role": "supporting" }
  ]
}
```

### Selective Codebase Review
```json
{
  "title": "Auth Module Review",
  "roster": "code",
  "context": "Focus on the authentication refactor in src/auth/",
  "files": [
    { "path": "./src/auth/middleware.ts", "role": "primary" },
    { "path": "./src/auth/providers.ts", "role": "primary" },
    { "path": "./src/auth/types.ts", "role": "supporting" },
    { "path": "./tests/auth.test.ts", "role": "supporting", "label": "Test coverage" }
  ]
}
```

### Cross-Domain Documentation
```json
{
  "title": "Product Launch Readiness",
  "roster": "software",
  "files": [
    { "path": "./prd.md", "role": "primary" },
    { "path": "./architecture.md", "role": "primary" },
    { "path": "./runbook.md", "role": "supporting" },
    { "path": "./changelog.md", "role": "reference" }
  ]
}
```

---

*Manifest Adapter v2.0 — User-Curated Review Packages*
