# Changelog

All notable changes to the R&R Peer Review Skill will be documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [2.0.0] — 2026-04-27

### Added
- **Input Adapters** — Multi-target support: single-file, vault (Obsidian/wiki), codebase, manifest
- **Configurable Rosters** — 7 reviewer presets: software (default), business, code, academic, creative, legal, flexi
- **Parameterized Reviewer Template** — Base template for building custom reviewers
- **15 new reviewer prompts** — Domain-specific reviewers for each roster preset
- **Roster-aware Area Chair** — Meta-quality gate adapts to selected roster domain
- **Auto-detection** — Automatically selects adapter based on input type (file, directory, manifest)
- **`--type` flag** — Override auto-detected input type
- **`--roster` flag** — Select reviewer roster (default: software)
- **`.rnr-reviewers.yaml`** — Custom roster support in project root
- **`.rnr-manifest.json`** — User-defined multi-file review packages
- **VARIABLES.md** — Master registry of all template placeholders
- **LICENSE** — MIT license
- **CHANGELOG.md** — This file

### Changed
- **SKILL.md** — Rewritten for V2 orchestration (adapters, rosters, new commands)
- **Area Chair prompt** — Parameterized reviewer list and meta-gate checks
- **Synthesis template** — Dynamic reviewer names from roster
- **Audit trail template** — Dynamic iteration history columns
- **Input validation** — Adapter-aware, supports directories and manifests
- **Help output** — New flags documented
- **State schema** — `document_path` → `input_target`, added `input_type` and `roster` fields
- Relocated `development_strategy/` → `docs/build-guide/`

### Unchanged
- Core R&R loop: Validate → Author → Reviews → Area Chair → Verdict
- Memory firewall in all reviewer prompts
- Security header and pattern detection
- Atomic write protocol for state management
- Escalation flow and human intervention options

---

## [1.0.0] — 2026-04-27

### Added
- Initial R&R Peer Review Skill
- Multi-agent blind peer review with iterative refinement
- 3 reviewer personas: Technical, Product, Operations
- Area Chair synthesis with meta-quality gate
- Security header with injection defense
- Suspicious pattern detection (regex + heuristics)
- Session state management with atomic writes and backup recovery
- Escalation flow with 3 human intervention options
- Complete development strategy documentation
- Verification protocol (9-test suite)
- Test documents (sample PRD, injection test)
