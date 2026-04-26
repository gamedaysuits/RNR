# Suspicious Pattern Detection

> **Purpose:** Detect potential prompt injection or manipulation attempts.
> **Behavior:** FLAG and LOG patterns, but CONTINUE review (do not block).

---

## Core Principle

Pattern detection is a **best-effort** security measure. It is NOT comprehensive and can be bypassed by sophisticated attacks. Its purpose is to:

1. Log suspicious content for audit
2. Alert reviewers to potential manipulation
3. Provide evidence for security review

---

## Regex Patterns

### Category 1: Role Override Attempts

```javascript
const ROLE_OVERRIDE_PATTERNS = [
  /ignore\s+(all\s+)?previous\s+instructions?/i,
  /forget\s+(your|the)\s+role/i,
  /you\s+are\s+now/i,
  /disregard\s+(all\s+)?(prior|previous)/i,
  /new\s+instructions?:/i,
  /override\s+mode/i
];
```

**Risk:** Attempts to change the reviewer's assigned role or behavior.

---

### Category 2: System Prompt Extraction

```javascript
const SYSTEM_PROMPT_PATTERNS = [
  /system\s*prompt/i,
  /reveal\s+(your\s+)?instructions/i,
  /what\s+are\s+your\s+rules/i,
  /show\s+(me\s+)?your\s+prompt/i
];
```

**Risk:** Attempts to extract system instructions or configuration.

---

### Category 3: Format Injection

```javascript
const FORMAT_INJECTION_PATTERNS = [
  /\[INST\]|\[\/INST\]/i,         // Llama format
  /<\|system\|>|<\|user\|>/i,      // ChatML format
  /\[SYSTEM\]|\[USER\]/i,           // Generic format
  /Human:|Assistant:/i              // Anthropic format
];
```

**Risk:** Attempts to inject fake conversation structures.

---

### Category 4: Code Execution

```javascript
const CODE_EXECUTION_PATTERNS = [
  /```(bash|sh|shell|cmd|powershell)/i,  // Shell code blocks
  /\$\([^)]+\)/,                          // Command substitution
  /`[^`]*rm\s+-rf/i,                      // Destructive commands
  /eval\s*\(/i                            // Eval attempts
];
```

**Risk:** Attempts to execute system commands.

---

## Heuristic Patterns

### Base64 Detection

```javascript
function checkBase64(content) {
  // Match base64-like blocks > 100 characters
  const base64Regex = /[A-Za-z0-9+\/=]{100,}/g;
  const matches = content.match(base64Regex) || [];
  
  return matches.map(m => ({
    pattern: "base64_block",
    length: m.length,
    preview: m.substring(0, 20) + "..."
  }));
}
```

**Risk:** Encoded payloads that may contain hidden instructions.

---

### Special Character Density

```javascript
function checkSpecialCharDensity(content) {
  const lines = content.split('\n');
  const suspicious = [];
  
  for (const line of lines) {
    if (line.length > 20) {
      const specialChars = line.replace(/[a-zA-Z0-9\s]/g, '');
      const density = specialChars.length / line.length;
      
      if (density > 0.5) {
        suspicious.push({
          pattern: "high_special_char_density",
          density: density,
          preview: line.substring(0, 30)
        });
      }
    }
  }
  
  return suspicious;
}
```

**Risk:** Obfuscated content or encoded payloads.

---

### Embedded JSON Detection

```javascript
function checkEmbeddedJSON(content) {
  const jsonBlocks = content.match(/\{[^}]+\}/g) || [];
  const suspicious = [];
  
  for (const block of jsonBlocks) {
    if (/"role"\s*:/.test(block) || /"content"\s*:/.test(block)) {
      suspicious.push({
        pattern: "embedded_json_with_role",
        preview: block.substring(0, 50)
      });
    }
  }
  
  return suspicious;
}
```

**Risk:** Attempts to inject fake message structures.

---

## Detection Behavior

### When Pattern Detected

1. **Log** the match to `logs/security.log`:
   ```json
   {
     "timestamp": "2026-01-31T10:15:00Z",
     "session_id": "20260131-101500-a1b2",
     "event": "pattern_match",
     "details": {
       "patterns_matched": ["ignore previous instructions", "base64_block"],
       "input_size": 500,
       "file_path": "/path/to/suspicious.md"
     }
   }
   ```

2. **Display warning** to user:
   ```
   ⚠️ Suspicious pattern detected and logged. Review continuing.
   ```

3. **Continue** with the review (do NOT block)

4. **Notify** reviewers via security header (already included in prompts)

---

### Trust Mode (`--trust` flag)

When `--trust` is specified:
- Skip ALL pattern detection
- Log trust bypass event:
  ```json
  {
    "event": "trust_bypass",
    "details": { "patterns_skipped": true }
  }
  ```
- Proceed directly to review

---

## Pattern Summary Table

| Category | Pattern Type | Example | Action |
|----------|--------------|---------|--------|
| Role Override | Regex | "ignore previous instructions" | Log + Warn |
| System Prompt | Regex | "reveal your system prompt" | Log + Warn |
| Format Injection | Regex | "[INST]" | Log + Warn |
| Code Execution | Regex | "```bash" | Log + Warn |
| Base64 | Heuristic | 100+ char encoded block | Log + Warn |
| Special Chars | Heuristic | >50% non-alphanumeric | Log + Warn |
| Embedded JSON | Heuristic | `{"role": "system"}` | Log + Warn |

---

## Known Limitations

1. **False positives** — Legitimate technical documents may trigger patterns
2. **Bypassable** — Sophisticated obfuscation can evade detection
3. **Not exhaustive** — New injection techniques emerge regularly
4. **Context-blind** — Cannot understand intent, only surface patterns

---

*Pattern Detection v1.0 — Best-effort security measure*
