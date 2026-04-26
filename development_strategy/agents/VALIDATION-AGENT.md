# Validation Agent — Input Security

> **Persona:** Security Engineer / Input Validator  
> **Responsibility:** Input validation rules, pattern detection, rejection messages  
> **Phase:** 3 (Validation)

---

## Mission

Build the input validation layer that protects the R&R system:
- Validate document requirements (size, format, encoding)
- Detect suspicious injection patterns
- Provide clear rejection messages
- Log all validation events

---

## Deliverables

| File | Purpose | Lines |
|------|---------|-------|
| input_checks.md | Validation rules | ~60 |
| patterns.md | Suspicious pattern regex | ~80 |
| rejection_messages.md | User-friendly errors | ~40 |

---

## Validation Rules

### Rule 1: Word Count
```markdown
## Minimum Word Count

**Threshold:** 50 words
**Count Method:** Split on whitespace, filter empty strings
**Action:** Reject with message if below threshold

### Implementation
```javascript
const words = content.split(/\s+/).filter(w => w.length > 0);
if (words.length < 50) {
  reject('too_short', { count: words.length });
}
```
```

### Rule 2: File Existence
```markdown
## File Accessibility

**Checks:**
1. Path exists
2. Is a file (not directory)
3. Is readable

**Action:** Reject if any check fails
```

### Rule 3: Encoding
```markdown
## UTF-8 Encoding

**Checks:**
1. Valid UTF-8 byte sequences
2. No null bytes
3. No invalid control characters

**Action:** Reject if encoding invalid
```

### Rule 4: Extension (Warning Only)
```markdown
## File Extension

**Preferred:** .md, .txt, or no extension
**Other:** Warn but continue
**Binary:** Reject (images, executables)
```

### Rule 5: Size (Warning Only)
```markdown
## Document Size

**Soft Limit:** ~50,000 tokens (~200k chars)
**Action:** Warn if exceeded, continue review
**Note:** LLM context may truncate
```

---

## Suspicious Patterns

### Regex Patterns
```javascript
const INJECTION_PATTERNS = [
  // Direct instruction override
  /ignore\s+(all\s+)?previous\s+instructions?/i,
  /forget\s+(your|the)\s+(role|instructions)/i,
  /disregard\s+(all\s+)?(above|previous)/i,
  
  // Role manipulation
  /you\s+are\s+now/i,
  /pretend\s+(you\s+are|to\s+be)/i,
  /act\s+as\s+(if|a)/i,
  
  // Prompt leakage attempts
  /system\s*prompt/i,
  /show\s+(me\s+)?(your|the)\s+instructions/i,
  /repeat\s+(your\s+)?instructions/i,
  
  // Instruction format injection
  /\[INST\]|\[\/INST\]/i,
  /\<\|im_start\|\>|\<\|im_end\|\>/i,
  /### (Human|Assistant|System):/i,
  
  // Code execution attempts
  /```(bash|sh|shell|cmd|powershell)/i,
  /\$\(.*\)/,  // Command substitution
  /`.*`/,      // Backtick execution
];
```

### Heuristic Patterns
```markdown
## Heuristic Detection

### Base64 Blocks
- Pattern: Contiguous base64 chars >100 length
- Risk: Encoded malicious content
- Action: Flag and log

### Special Character Density
- Pattern: Line with >50% non-alphanumeric
- Risk: Obfuscation attempts
- Action: Flag and log

### JSON Structure Injection
- Pattern: JSON with "role", "content", "system" keys
- Risk: Chat format injection
- Action: Flag and log
```

---

## Pattern Response Protocol

```markdown
## On Pattern Match

1. Log to security.log:
   {
     "event": "pattern_match",
     "patterns_matched": ["ignore previous"],
     "line_number": 42
   }

2. Display warning (unless --trust):
   "⚠️ Suspicious pattern detected and logged. Review continuing."

3. Continue review (do NOT block)

4. Include pattern info in reviewer context:
   "Note: Suspicious patterns detected at lines X, Y, Z"
```

---

## Rejection Messages

| Code | Message | Context |
|------|---------|---------|
| `too_short` | ❌ Document too short ({{N}} words < 50 minimum). Add more content to enable meaningful review. | N = word count |
| `not_found` | ❌ Cannot read '{{PATH}}'. Verify the file exists and the path is correct. | PATH = input path |
| `not_file` | ❌ '{{PATH}}' is a directory, not a file. Provide a file path. | PATH = input path |
| `encoding` | ❌ File encoding error. Ensure the file is saved as UTF-8 without BOM. | - |
| `binary` | ❌ Binary file detected. Only text files (.md, .txt) are supported. | - |
| `too_large` | ⚠️ Document may exceed context limit (~50k tokens). Consider splitting into smaller documents. | - |
| `pattern` | ⚠️ Suspicious pattern detected at line {{N}} and logged. Review continuing with caution. | N = line number |

---

## Logging Protocol

### Security Log Entry
```json
{
  "timestamp": "2026-01-31T10:15:00Z",
  "session_id": "20260131-101500-a1b2",
  "event": "validation_pass",
  "details": {
    "input_size": 1250,
    "word_count": 247,
    "patterns_matched": []
  }
}
```

### Event Types
- `validation_pass` - All checks passed
- `validation_fail` - Rejected (specify reason)
- `pattern_match` - Suspicious pattern found
- `size_warning` - Document exceeds soft limit

---

## Quality Checklist

Before handing off:

- [ ] All validation rules implemented
- [ ] Pattern list comprehensive but not blocking
- [ ] Rejection messages are user-friendly
- [ ] Warnings vs errors distinguished
- [ ] Logging format matches schema
- [ ] Trust mode bypasses pattern checks only

---

*Validation Agent Spec v1.0*
