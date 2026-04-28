# Single File Adapter

> **Purpose:** Handle review of a single document file.
> This is the V1-compatible default adapter.

---

## Input

A path to a single file (`.md`, `.txt`, or any text file).

```
/peer-review ./my-document.md
/peer-review ./specs/feature-spec.txt
```

---

## Detection

This adapter is selected when:
- Input is a file path (not a directory)
- Input does not end with `.rnr-manifest.json`

---

## Assembly Process

```
FUNCTION assemble_single_file(file_path):

  1. Validate file exists and is readable
  2. Read file content as UTF-8
  3. Count words
  4. Check encoding (reject null bytes)

  RETURN Review Package:
    input_type: "single-file"
    input_target: file_path
    file_count: 1
    total_word_count: word_count
    files: [
      { path: basename(file_path), role: "primary", word_count: word_count }
    ]
    content: file_content
    metadata:
      warnings: [extension warning if not .md/.txt]
```

---

## Validation Rules

Uses standard validation from `validation/input_checks.md`:

| Check | Action |
|-------|--------|
| File exists | Reject if missing |
| File readable | Reject if permission denied |
| UTF-8 encoding | Reject if invalid |
| Word count ≥ 50 | Reject if too short |
| Extension check | Warn if not `.md`/`.txt` |
| Token estimate | Warn if > 50k tokens |

---

## Content Output

No assembly needed — file content is passed through directly:

```markdown
[entire file content as-is]
```

---

## Backward Compatibility

This adapter produces **identical behavior to V1**. Existing workflows using:

```
/peer-review ./my-doc.md
```

Will continue to work exactly as before with no changes to output format, session structure, or reviewer behavior.

---

*Single File Adapter v2.0*
