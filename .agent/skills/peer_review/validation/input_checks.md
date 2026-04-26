# Input Validation Rules

> **Purpose:** Define validation checks performed before starting a review session.

---

## Validation Sequence

All checks run in order. Validation fails on first error.

1. File Existence Check
2. File Readability Check
3. Encoding Check
4. Word Count Check
5. Extension Check (warning only)
6. Size Check (warning only)

---

## Rule 1: File Existence

**Check:** File exists at the specified path

**Implementation:**
```
IF NOT file_exists(path):
    FAIL with "not_found" error
```

**Error Code:** `not_found`

---

## Rule 2: File Readability

**Check:** File can be opened and read

**Implementation:**
```
TRY:
    content = read_file(path)
CATCH error:
    FAIL with "not_found" error (permission or read failure)
```

**Error Code:** `not_found`

---

## Rule 3: Encoding Check

**Check:** File is valid UTF-8 encoded text

**Implementation:**
```
TRY:
    decode_utf8(content)
CATCH error:
    FAIL with "encoding" error
    
IF contains_null_bytes(content):
    FAIL with "encoding" error
```

**Error Code:** `encoding`

---

## Rule 4: Word Count Check

**Check:** Document has at least 50 words

**Implementation:**
```javascript
const words = content.split(/\s+/).filter(w => w.length > 0);
const wordCount = words.length;

if (wordCount < 50) {
    FAIL with "too_short" error
    include actual word count in message
}
```

**Error Code:** `too_short`

**Threshold:** `MIN_WORD_COUNT = 50`

---

## Rule 5: Extension Check (Warning Only)

**Check:** File has recommended extension

**Preferred Extensions:**
- `.md` (Markdown)
- `.txt` (Plain text)
- No extension

**Implementation:**
```
extension = get_file_extension(path)

IF extension NOT IN ['.md', '.txt', '']:
    WARN: "Non-standard file extension. Markdown recommended."
    CONTINUE (do not fail)
```

**Behavior:** Warning only, does not block review

---

## Rule 6: Size Check (Warning Only)

**Check:** Document is within recommended token limits

**Implementation:**
```javascript
// Rough token estimate: 1 token ≈ 4 characters
const estimatedTokens = content.length / 4;

if (estimatedTokens > 50000) {
    WARN: "Document may exceed context limit (~50k tokens). Consider splitting."
    CONTINUE (do not fail)
}
```

**Threshold:** `MAX_TOKENS_SOFT = 50000`

**Behavior:** Warning only, does not block review

---

## Logging Requirements

All validation results must be logged to `logs/security.log`:

### On Success
```json
{
  "timestamp": "2026-01-31T10:15:00Z",
  "session_id": "20260131-101500-a1b2",
  "event": "validation_pass",
  "details": {
    "input_size": 247,
    "patterns_matched": [],
    "file_path": "/path/to/document.md"
  }
}
```

### On Failure
```json
{
  "timestamp": "2026-01-31T10:15:00Z",
  "session_id": "pending",
  "event": "validation_fail",
  "details": {
    "validation_error": "too_short",
    "input_size": 23,
    "file_path": "/path/to/document.md"
  }
}
```

---

## Summary Table

| Rule | Check | Threshold | Action on Fail |
|------|-------|-----------|----------------|
| 1 | File exists | - | Reject |
| 2 | File readable | - | Reject |
| 3 | UTF-8 encoded | - | Reject |
| 4 | Word count | ≥50 words | Reject |
| 5 | Extension | .md, .txt | Warn only |
| 6 | Token count | <50k tokens | Warn only |

---

*Input Validation Rules v1.0*
