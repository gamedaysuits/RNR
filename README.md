# R&R Peer Review Skill

> General-purpose multi-agent blind peer review with iterative refinement

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Version](https://img.shields.io/badge/version-2.0-blue)

---

## What is this?

R&R (Revise & Resubmit) is an AI skill that provides **rigorous, automated peer review** for documents, vaults, codebases, and multi-file projects. It simulates an academic-style review board with configurable, domain-specific reviewer panels who evaluate your work independently.

### Features

- **🔒 Blind Review:** N reviewers evaluate in isolation — no cross-contamination
- **🎯 Configurable Rosters:** 7 preset reviewer panels (software, business, code, academic, creative, legal, flexi)
- **📂 Multi-Target:** Review single files, Obsidian vaults, codebases, or curated manifests
- **🔄 Iterative Refinement:** Automatic revision cycles until acceptance
- **🏛️ Domain-Aware Quality Gate:** Area Chair adapts meta-checks to your content domain
- **🛡️ Security Hardened:** Pattern detection for prompt injection attempts
- **💾 State Persistence:** Resume interrupted sessions, atomic writes with backup
- **🔧 Fully Customizable:** Drop `.rnr-reviewers.yaml` in your project root for custom panels

---

## Installation

1. Copy the `peer_review` folder to your agent skills directory:

```bash
cp -r peer_review .agent/skills/
```

2. Restart your IDE or reload skills.

3. Verify installation:

```
/peer-review --help
```

### Compatibility

Works with any AI agent environment that supports skill files:
- Antigravity
- Claude Code
- Any IDE with agent skill support

---

## Quick Start

### Review a Document
```
/peer-review ./my-prd.md
```

### Review an Obsidian Vault
```
/peer-review ./prospectus/
```

### Review with a Specific Roster
```
/peer-review ./business-plan.md --roster business
```

### Review a Codebase
```
/peer-review ./src/ --type codebase --roster code
```

### Example Output

```
📂 Session: 20260131-101500-a1b2
📊 Input: ./prospectus/ (vault, 12 files, 8200 words)
🎭 Roster: business (Financial Analyst, Market Strategist, Risk Assessor)

ITER 1: FAIL/PASS/FAIL → REVISE(5)
ITER 2: PASS/PASS/PASS → ✅ ACCEPT

Output: .peer_review/.../final/document.md
```

---

## Commands

| Command | Description |
|---------|-------------|
| `/peer-review <target>` | Review a file, directory, or manifest |
| `/peer-review --help` | Show help |
| `/peer-review --trust` | Skip pattern checks |
| `/peer-review --resume` | Resume interrupted session |
| `/peer-review --max-iter N` | Limit iterations (default: 10) |
| `/peer-review --type <type>` | Override input type detection |
| `/peer-review --roster <name>` | Select reviewer roster |
| `/peer-review --cleanup` | Remove old sessions |

---

## Input Types

| Type | Auto-Detected When | Example |
|------|-------------------|---------|
| `single-file` | Target is a file | `./my-prd.md` |
| `vault` | Directory of Markdown files | `./prospectus/` |
| `codebase` | Directory with source code | `./src/` |
| `manifest` | File is `.rnr-manifest.json` | `./.rnr-manifest.json` |

---

## Reviewer Rosters

| Roster | Reviewers | Best For |
|--------|-----------|----------|
| `software` ★ | Technical, Product, Ops | PRDs, RFCs, specs |
| `business` | Financial, Market, Risk | Business plans, pitch decks |
| `code` | Quality, Security, Architecture | Codebases, modules |
| `academic` | Methodology, Literature, Ethics | Research papers |
| `creative` | Craft, Narrative, Audience | Writing, scripts |
| `legal` | Compliance, Precedent, Risk | Contracts, policies |
| `flexi` | User-defined | Anything |

★ = default

### Custom Rosters

Create a `.rnr-reviewers.yaml` in your project root:

1. Copy `rosters/flexi.yaml` as your starting template
2. Customize reviewer names, focus areas, and meta-gate checks
3. Run `/peer-review` — your custom roster is auto-detected

---

## How It Works

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ Validate │─▶│  Adapt   │─▶│ Review   │─▶│Synthesis │─▶│ Verdict  │
│          │  │          │  │ (N×)     │  │          │  │          │
└──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
                                 ▲                            │
                                 └──────── REVISE ────────────┘
```

1. **Validate:** Parse arguments, detect input type, load roster
2. **Adapt:** Run input adapter to assemble Review Package
3. **Review:** N blind reviewers evaluate independently
4. **Synthesis:** Area Chair consolidates with domain-specific quality checks
5. **Verdict:** Accept, revise, or escalate to human

---

## File Structure

```
.agent/skills/peer_review/
├── SKILL.md                     # Main orchestration
├── VARIABLES.md                 # Template variable registry
├── adapters/                    # Input type handlers
│   ├── _adapter_protocol.md
│   ├── single_file.md
│   ├── vault.md
│   ├── codebase.md
│   └── manifest.md
├── rosters/                     # Reviewer configurations
│   ├── _roster_protocol.md
│   ├── software.yaml (default)
│   ├── business.yaml
│   ├── code.yaml
│   ├── academic.yaml
│   ├── creative.yaml
│   ├── legal.yaml
│   └── flexi.yaml
├── prompts/                     # Persona definitions (22 prompts)
│   ├── _security_header.md
│   ├── author_initial.md
│   ├── author_revision.md
│   ├── area_chair.md
│   ├── escalation.md
│   ├── reviewer_template.md
│   ├── reviewer_technical.md
│   ├── reviewer_product.md
│   ├── reviewer_ops.md
│   └── ... (15 domain-specific reviewers)
├── templates/                   # Output formats
├── validation/                  # Input validation
└── logs/                        # Security logging
```

---

## Configuration

### Roster Resolution Order
```
--roster flag → manifest roster field → .rnr-reviewers.yaml → default (software)
```

### Max Iterations
Default: 10. Override with `--max-iter N`.

### Trust Mode
Skip pattern detection with `--trust`. Use for rapid iteration on known-safe content.

### Session Retention
Default: 30 days. Customize with `--cleanup --days N`.

---

## Security

- Pattern detection for common prompt injection phrases
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

Built for AI-native development environments.

*R&R Peer Review Skill v2.0*
