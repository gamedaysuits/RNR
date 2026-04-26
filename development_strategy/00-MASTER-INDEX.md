# R&R Orchestration System — Development Strategy

> **Version:** 1.0  
> **Created:** 2026-01-31  
> **Methodology:** Agent-Forward Development with Antigravity + Claude Open 4.5

---

## 🎯 Mission

Build a production-ready **Multi-Agent Peer Review Skill** for Antigravity in **12-16 hours** of coordinated agent work, with zero bugs and exportable architecture.

> [!NOTE]
> **Gatekeeper Clarification:** The protocol diagram shows "Gatekeeper" as the final step. This is NOT a separate persona—it is the **verdict logic** within the Area Chair prompt. The Area Chair performs the final ACCEPT/REVISE/ESCALATE decision.

---

## 📁 Documentation Map

### Core Strategy Documents

| Document | Purpose |
|----------|---------|
| [01-HUMAN-COORDINATOR-GUIDE.md](./01-HUMAN-COORDINATOR-GUIDE.md) | **START HERE** — Step-by-step instructions for the human developer |
| [02-AGENT-PROMPTS.md](./02-AGENT-PROMPTS.md) | Copy-paste prompts for each development phase |
| [03-FILE-SPECIFICATIONS.md](./03-FILE-SPECIFICATIONS.md) | Detailed specs for all 17 skill files |
| [04-VERIFICATION-PROTOCOL.md](./04-VERIFICATION-PROTOCOL.md) | Testing procedures and acceptance criteria |

---

### Agent-Specific Documentation

| Agent | File | Responsibility |
|-------|------|----------------|
| **Architect** | [agents/ARCHITECT-AGENT.md](./agents/ARCHITECT-AGENT.md) | File structure, state management, session logic |
| **Prompt Author** | [agents/PROMPT-AUTHOR-AGENT.md](./agents/PROMPT-AUTHOR-AGENT.md) | Persona prompts, security header, escalation logic |
| **Validation** | [agents/VALIDATION-AGENT.md](./agents/VALIDATION-AGENT.md) | Input checks, pattern detection, rejection messages |
| **QA** | [agents/QA-AGENT.md](./agents/QA-AGENT.md) | Verification tests, edge cases, acceptance sign-off |

---

### Supplementary Documentation

| Document | Purpose |
|----------|---------|
| [END-USER-GUIDE.md](./END-USER-GUIDE.md) | How to use the skill once built |
| [GITHUB-README.md](./GITHUB-README.md) | Public-facing repository documentation |
| [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) | Common issues and fixes |

---

## 🚀 Quick Start

1. **Read** [01-HUMAN-COORDINATOR-GUIDE.md](./01-HUMAN-COORDINATOR-GUIDE.md) completely
2. **Follow** the sequential prompts in [02-AGENT-PROMPTS.md](./02-AGENT-PROMPTS.md)
3. **Verify** using [04-VERIFICATION-PROTOCOL.md](./04-VERIFICATION-PROTOCOL.md)
4. **Ship** using the GitHub workflow in [GITHUB-README.md](./GITHUB-README.md)

---

## 📊 Development Phases

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  PHASE 1    │───▶│  PHASE 2    │───▶│  PHASE 3    │───▶│  PHASE 4    │
│ Foundation  │    │  Prompts    │    │ Validation  │    │ Integration │
│  (~3 hrs)   │    │  (~4 hrs)   │    │  (~3 hrs)   │    │  (~4 hrs)   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
     │                   │                  │                  │
     ▼                   ▼                  ▼                  ▼
 SKILL.md           8 Prompt Files    3 Validation      Verification
 Directory          Security Header   Files             E2E Testing
 Structure          Templates         Patterns          Documentation
```

---

## 📋 Reference Architecture

```
.agent/skills/peer_review/
├── SKILL.md                    # Orchestration logic
├── prompts/ (8)
│   ├── _security_header.md     # Prepended to all prompts
│   ├── author_initial.md       
│   ├── author_revision.md      
│   ├── reviewer_technical.md   
│   ├── reviewer_product.md     
│   ├── reviewer_ops.md         
│   ├── area_chair.md           
│   └── escalation.md           
├── templates/ (4)
│   ├── review_output.md        
│   ├── synthesis_output.md     
│   ├── audit_trail.md          
│   └── help_output.md          
├── validation/ (3)
│   ├── input_checks.md         
│   ├── patterns.md             
│   └── rejection_messages.md   
└── logs/
    └── security.log.schema.json
```

---

## 🔗 External References

- **Source Plan:** [IMPLEMENTATION_PLAN.md](../IMPLEMENTATION_PLAN.md)
- **KI:** Multi-Agent Peer Review Orchestration (R&R) v1.4
- **KI:** Agent-Forward Development Methodology

---

*This documentation was generated for agent-forward development with Antigravity.*
