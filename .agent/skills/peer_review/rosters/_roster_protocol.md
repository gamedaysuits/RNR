# Roster Protocol

> **Purpose:** Define how reviewer rosters are configured, loaded, and applied.
> Rosters determine which reviewer personas evaluate a document and what meta-quality gate the Area Chair uses.

---

## What is a Roster?

A roster is a **configuration file** that defines:
1. Which reviewer personas to use (and how many)
2. Each reviewer's focus areas and evaluation criteria
3. The Area Chair's meta-quality gate checks for this domain

---

## Roster File Format (YAML)

```yaml
# Roster metadata
name: "software"
display_name: "Software Product Review"
description: "Reviews from Technical, Product, and Operations perspectives"
version: "2.0"

# Meta-quality gate checks — used by the Area Chair
meta_gate:
  - name: "Problem Validity"
    question: "Is this problem worth solving? Is the motivation clear and justified?"
  - name: "Scope Appropriateness"
    question: "Is the solution right-sized? Not too ambitious, not too limited?"
  - name: "Execution Viability"
    question: "Can this realistically be built as specified with available resources?"

# Reviewer definitions
reviewers:
  - name: "Technical"
    prompt: "prompts/reviewer_technical.md"
    title: "Senior Technical Architect"
    role: "Technical Feasibility Assessment"
    focus_summary: "Architecture, feasibility, complexity, dependencies, performance"

  - name: "Product"
    prompt: "prompts/reviewer_product.md"
    title: "Product Manager / UX Lead"
    role: "Product & User Experience Assessment"
    focus_summary: "User problems, flows, edge cases, scope, market fit"

  - name: "Operations"
    prompt: "prompts/reviewer_ops.md"
    title: "DevOps / Security Engineer"
    role: "Operations, Security & Feasibility Assessment"
    focus_summary: "Security, timeline, resources, operational complexity, deployment"
```

---

## Resolution Order

When a review session starts, the roster is resolved as follows:

```
1. Check --roster flag          → e.g., --roster business
2. Check manifest roster field  → e.g., "roster": "business" in .rnr-manifest.json
3. Check project root           → .rnr-reviewers.yaml (custom roster)
4. Default                      → "software"
```

### Custom Roster Location

Users can place a `.rnr-reviewers.yaml` file in their **project root** (the directory containing the reviewed content). This file follows the same schema as shipped presets and takes precedence over the default roster.

---

## Shipped Presets

| Preset | File | Reviewers | Best For |
|--------|------|-----------|----------|
| `software` | `software.yaml` | Technical, Product, Ops | PRDs, RFCs, technical specs |
| `business` | `business.yaml` | Financial Analyst, Market Strategist, Risk Assessor | Business plans, pitch decks, prospectuses |
| `code` | `code.yaml` | Code Quality, Security Auditor, Architecture | Codebases, PRs, modules |
| `academic` | `academic.yaml` | Methodology, Literature, Ethics | Research papers, proposals, theses |
| `creative` | `creative.yaml` | Craft, Narrative, Audience | Writing, scripts, articles, portfolios |
| `legal` | `legal.yaml` | Compliance, Precedent, Risk | Contracts, policies, terms of service |
| `flexi` | `flexi.yaml` | User-defined (template) | Anything — maximum flexibility |

---

## Roster Constraints

- **Minimum reviewers:** 2 (at least two perspectives needed for meaningful synthesis)
- **Maximum reviewers:** 5 (more than 5 creates diminishing returns and excess token cost)
- **Recommended:** 3 reviewers (balanced coverage vs. cost)

---

## Interaction with Area Chair

The Area Chair prompt is parameterized to read from the active roster:

1. **Reviewer List** — Area Chair receives names and roles of all reviewers from the roster
2. **Meta-Gate Checks** — Area Chair performs the quality gate checks defined in the roster's `meta_gate` field
3. **Synthesis Table** — Output table columns are dynamically generated from reviewer names

---

## Creating Custom Rosters

### Quick Start

1. Copy any preset YAML to your project root as `.rnr-reviewers.yaml`
2. Edit the reviewer names, prompts, and focus areas
3. Run `/peer-review` — it will auto-detect the custom roster

### Using the Flexi Preset

The `flexi` roster ships with the parameterized `reviewer_template.md` prompt. Define your reviewers inline:

```yaml
name: "flexi"
display_name: "Custom Flex Review"
description: "User-defined reviewer panel"

meta_gate:
  - name: "Core Thesis"
    question: "Is the central argument sound and well-supported?"
  - name: "Completeness"
    question: "Are all necessary aspects covered?"
  - name: "Actionability"
    question: "Can the reader act on this document as-is?"

reviewers:
  - name: "Domain Expert"
    prompt: "prompts/reviewer_template.md"
    title: "Subject Matter Expert"
    role: "Domain-Specific Accuracy Assessment"
    focus_summary: "Factual accuracy, domain conventions, expert-level scrutiny"
    focus_areas: |
      ### 1. Factual Accuracy
      - Are claims supported by evidence?
      - Are domain-specific terms used correctly?
      - Are there factual errors or misleading statements?

      ### 2. Completeness
      - Are key topics adequately covered?
      - Are there significant omissions?

      ### 3. Currency
      - Is the information up to date?
      - Are references to tools, standards, or practices current?
    guidelines: |
      1. **Verify claims** — Check factual statements against your domain knowledge
      2. **Flag jargon** — Note where terminology may confuse the target audience
      3. **Be independent** — Your review must stand alone

  - name: "Clarity Reviewer"
    prompt: "prompts/reviewer_template.md"
    title: "Communications Specialist"
    role: "Clarity & Readability Assessment"
    focus_summary: "Structure, readability, audience alignment, plain language"
    focus_areas: |
      ### 1. Structure
      - Is the document logically organized?
      - Is the flow of information natural?

      ### 2. Readability
      - Is the language appropriate for the target audience?
      - Are sentences clear and concise?

      ### 3. Actionability
      - Can the reader understand what to do next?
      - Are instructions unambiguous?
    guidelines: |
      1. **Read as the audience** — Would the target reader understand this?
      2. **Flag complexity** — Identify unnecessarily complex passages
      3. **Be independent** — Your review must stand alone
```

---

*Roster Protocol v2.0 — Reviewer Configuration System*
