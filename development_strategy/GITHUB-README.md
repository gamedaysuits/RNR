# R&R Peer Review Skill

> Multi-agent blind peer review for Antigravity

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## What is this?

R&R (Revise & Resubmit) is an Antigravity skill that provides **rigorous, automated peer review** for your documents. It simulates an academic-style review board with three specialized reviewers who evaluate your document independently.

### Features

- **🔒 Blind Review:** Three reviewers (Technical, Product, Operations) evaluate in isolation
- **🎯 Quality Gate:** Area Chair synthesizes feedback with meta-quality checks
- **🔄 Iterative Refinement:** Automatic revision cycles until acceptance
- **🛡️ Security Hardened:** Pattern detection for prompt injection attempts
- **💾 State Persistence:** Resume interrupted sessions, atomic writes with backup

---

## Installation

1. Copy the `peer_review` folder to your Antigravity skills directory:

```bash
cp -r peer_review .agent/skills/
```

2. Restart Antigravity or reload skills.

3. Verify installation:

```
/peer-review --help
```

---

## Quick Start

```
/peer-review ./my-document.md
```

### Example Output

```
📂 Session: 20260131-101500-a1b2

ITER 1: FAIL/FAIL/PASS → REVISE(4)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT

Output: .peer_review/.../final/document.md
```

---

## Commands

| Command | Description |
|---------|-------------|
| `/peer-review <doc>` | Review a document |
| `/peer-review --help` | Show help |
| `/peer-review --trust` | Skip pattern checks |
| `/peer-review --resume` | Resume session |
| `/peer-review --max-iter N` | Limit iterations |
| `/peer-review --cleanup` | Remove old sessions |

---

## How It Works

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Validate │───▶│ Review   │───▶│ Synthesis│───▶│ Verdict  │
│          │    │ (3x)     │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                     ▲                               │
                     └───────── REVISE ──────────────┘
```

1. **Validate:** Check document meets requirements (50+ words, UTF-8)
2. **Review:** Three blind reviewers evaluate independently
3. **Synthesis:** Area Chair consolidates feedback
4. **Verdict:** Accept, revise, or escalate to human

---

## Document Requirements

- **Minimum:** 50 words
- **Recommended:** 200-2000 words
- **Format:** Markdown preferred
- **Encoding:** UTF-8

---

## File Structure

```
.agent/skills/peer_review/
├── SKILL.md                    # Main orchestration
├── prompts/                    # Persona definitions
├── templates/                  # Output formats
├── validation/                 # Input checks
└── logs/                       # Security logging
```

---

## Configuration

### Max Iterations
Default: 10. Override with `--max-iter N`.

### Trust Mode
Skip pattern detection with `--trust`. Use for rapid iteration on known-safe documents.

### Session Retention
Default: 30 days. Customize with `--cleanup --days N`.

---

## Security

The skill includes protection against prompt injection:

- Pattern detection for common attack phrases
- Memory firewall between reviewers
- Security logging for audit
- Trust mode bypass for power users

---

## License

MIT License. See [LICENSE](./LICENSE).

---

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

---

## Credits

Built with Antigravity + Claude Open 4.5

*R&R Peer Review Skill v1.0*
