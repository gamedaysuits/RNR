# Rejection Messages

> **Purpose:** User-friendly error messages for validation failures.
> **Principle:** Be clear, be helpful, provide actionable guidance.

---

## Error Messages by Code

### `too_short`

**Condition:** Document has fewer than 50 words.

**Message:**
```
❌ Document too short ({{N}} words < 50 minimum)

Your document needs at least 50 words for a meaningful review.
Currently the document has {{N}} words.

Suggestions:
• Add more detail to your requirements
• Expand on the problem statement
• Include technical specifications
```

**Variables:**
- `{{N}}` — Actual word count

---

### `not_found`

**Condition:** File doesn't exist or can't be read.

**Message:**
```
❌ Cannot read '{{PATH}}'

The file could not be found or opened.

Please check:
• The file path is correct
• The file exists at that location
• You have permission to read the file

Tip: Use an absolute path or path relative to current directory.
```

**Variables:**
- `{{PATH}}` — Provided file path

---

### `encoding`

**Condition:** File is not valid UTF-8.

**Message:**
```
❌ Encoding error

The file contains invalid characters or is not UTF-8 encoded.

To fix:
• Re-save the file as UTF-8 in your text editor
• Check for binary or corrupted content
• Remove any special characters that can't be displayed

Most text editors: File → Save As → Encoding: UTF-8
```

---

### `too_large` (Warning Only)

**Condition:** Estimated tokens exceed 50,000.

**Message:**
```
⚠️ Document may exceed context limit

Your document is approximately {{N}} tokens (~{{W}} words).
This may exceed the review context limit (~50k tokens).

The review will continue, but you may experience:
• Truncated reviews
• Missed issues in later sections
• Less thorough analysis

Consider:
• Splitting into multiple smaller documents
• Reviewing sections separately
• Reducing verbosity
```

**Variables:**
- `{{N}}` — Estimated tokens
- `{{W}}` — Word count

---

### `pattern_detected` (Warning Only)

**Condition:** Suspicious pattern found in document.

**Message:**
```
⚠️ Suspicious pattern detected

A potentially problematic pattern was found in your document:
• {{PATTERN_DESCRIPTION}}

This has been logged for security review.
The review will continue normally.

Note: If this was intentional (e.g., documenting security practices),
you can use --trust to skip pattern checks.
```

**Variables:**
- `{{PATTERN_DESCRIPTION}}` — Brief description of what was found

---

### `bad_extension` (Warning Only)

**Condition:** File has non-standard extension.

**Message:**
```
⚠️ Non-standard file extension

The file '{{FILENAME}}' has extension '{{EXT}}'.
Markdown (.md) or plain text (.txt) is recommended for best results.

The review will continue normally.
```

**Variables:**
- `{{FILENAME}}` — File name
- `{{EXT}}` — File extension

---

## Message Guidelines

### Tone
- **Friendly but professional** — Not too casual, not robotic
- **Helpful** — Always provide actionable next steps
- **Clear** — No jargon or ambiguous language

### Structure
1. **Status indicator** — ❌ for errors, ⚠️ for warnings
2. **Brief problem statement** — One line summary
3. **Details** — Specific information about the issue
4. **Solutions** — Concrete steps to fix
5. **Tips** — Additional helpful context (optional)

### Symbols
- ❌ — Error (blocks review)
- ⚠️ — Warning (review continues)
- • — List item
- → — Process step or menu path

---

## Quick Reference

| Code | Symbol | Blocks Review | User Action |
|------|--------|---------------|-------------|
| `too_short` | ❌ | Yes | Add more content |
| `not_found` | ❌ | Yes | Check file path |
| `encoding` | ❌ | Yes | Re-save as UTF-8 |
| `too_large` | ⚠️ | No | Consider splitting |
| `pattern_detected` | ⚠️ | No | Review content or use --trust |
| `bad_extension` | ⚠️ | No | No action required |

---

*Rejection Messages v1.0*
