# Vault Adapter

> **Purpose:** Handle review of Obsidian vaults, documentation wikis, and multi-file Markdown suites.
> This is the **primary adapter** — R&R's most common use case.

---

## Input

A path to a directory containing Markdown files, typically an Obsidian vault or documentation wiki.

```
/peer-review ./prospectus/
/peer-review ./my-vault/ --type vault
/peer-review ./docs/wiki/
```

---

## Detection

This adapter is selected when:
- Input is a directory path
- More than 60% of non-hidden files are `.md` files
- OR: auto-detection defaults to `vault` for ambiguous directories (vault is the primary use case)
- OR: user specifies `--type vault`

---

## Assembly Process

```
FUNCTION assemble_vault(directory_path):

  1. DISCOVER files
     - Recursively scan directory for all files
     - Exclude hidden files/dirs (starting with .)
     - Exclude common noise: node_modules/, .git/, __pycache__/, .obsidian/
     - Include: .md files (primary), .txt files (secondary)
     - Sort files alphabetically by path

  2. BUILD LINK GRAPH
     - Scan each .md file for internal links:
       - Wiki-style: [[page-name]] or [[page-name|display text]]
       - Markdown-style: [text](./relative/path.md) or [text](relative/path.md)
       - Heading anchors: [[page-name#heading]] or [text](./path.md#heading)
     - Build adjacency map: { source_file → [target_files] }
     - Identify orphan files (no incoming or outgoing links)
     - Identify hub files (most connections — likely index/overview pages)

  3. DETERMINE FILE ROLES
     - Hub files (≥5 outgoing links): role = "index"
     - Files named README.md, index.md, 00-*, overview*: role = "index"
     - Files linked by index/hub files: role = "primary"
     - Orphan files: role = "supporting"
     - All others: role = "primary"

  4. ORDER CONTENT
     - Index files first (sorted by link count, descending)
     - Primary files second (sorted alphabetically)
     - Supporting/orphan files last (sorted alphabetically)

  5. ASSEMBLE REVIEW PACKAGE
     - Prepend vault structure summary (file tree + link graph)
     - Concatenate files with boundary markers
     - Include cross-reference summary for reviewers

  RETURN Review Package:
    input_type: "vault"
    input_target: directory_path
    file_count: N
    total_word_count: sum of all file word counts
    files: [array of file metadata with roles]
    content: assembled content (see Output Format below)
    metadata:
      link_graph: adjacency map
      warnings: [orphan warnings, broken link warnings, size warnings]
```

---

## Output Format

The assembled content follows this structure:

```markdown
# Vault Review Package

**Source:** {{directory_path}}
**Files:** {{N}} documents
**Total Words:** {{word_count}}

---

## Vault Structure

### File Map
{{indented file tree showing directory structure}}

### Cross-Reference Graph

| File | Outgoing Links | Incoming Links | Role |
|------|---------------|----------------|------|
| overview.md | 8 | 0 | index |
| architecture.md | 3 | 5 | primary |
| api-spec.md | 1 | 4 | primary |
| glossary.md | 0 | 2 | supporting |

### Link Health

- **Broken links:** {{list of [[target]] references that don't resolve to a file}}
- **Orphan files:** {{files with no incoming or outgoing links}}

---

## Documents

---
📄 **File: overview.md** (Role: index | 450 words)
---

[file content]

---
📄 **File: architecture.md** (Role: primary | 1200 words)
---

[file content]

---
📄 **File: api-spec.md** (Role: primary | 800 words)
---

[file content]

---
📄 **File: glossary.md** (Role: supporting | 200 words)
---

[file content]
```

---

## Vault-Specific Validation

In addition to standard validation rules:

| Check | Threshold | Action |
|-------|-----------|--------|
| Directory exists | - | Reject |
| Contains .md files | ≥ 1 | Reject ("No Markdown files found") |
| Combined word count | ≥ 50 | Reject |
| Broken internal links | Any | Warn (list them — reviewers should flag these) |
| Orphan files | Any | Warn (may indicate incomplete documentation) |
| Total token estimate | > 50k | Warn ("Large vault. Consider using a manifest to select key files.") |
| File count | > 50 | Warn ("Large vault with {{N}} files. Review may be lengthy.") |

---

## Obsidian-Specific Handling

### Excluded Paths
```
.obsidian/          # Obsidian config
.trash/             # Obsidian trash
_templates/         # Template files
_attachments/       # Binary attachments (images, PDFs)
```

### Wiki Link Resolution

Obsidian uses shortest-path matching for `[[links]]`. The adapter resolves links using:

1. **Exact filename match** (case-insensitive): `[[my-page]]` → `my-page.md`
2. **Path suffix match**: `[[folder/page]]` → `any/path/folder/page.md`
3. **Unresolved**: Mark as broken link in the link graph

### Frontmatter

If `.md` files contain YAML frontmatter (`---` delimited), preserve it in the assembled content. Reviewers may find metadata (tags, aliases, status) useful for context.

---

## Reviewer Guidance

When the vault adapter assembles the review package, it prepends this note to reviewers:

```markdown
> **📚 Vault Review Mode**
>
> You are reviewing a multi-file documentation vault. Pay attention to:
> - **Cross-reference integrity** — Do internal links resolve correctly?
> - **Coverage completeness** — Are there gaps in the documentation?
> - **Consistency** — Is terminology, tone, and structure consistent across files?
> - **Information architecture** — Is the vault well-organized and navigable?
> - **Redundancy** — Is content unnecessarily duplicated across files?
```

---

## Example: Obsidian Vault PRD

Given a vault at `./prospectus/` containing:

```
prospectus/
├── product-vision.md        ← links to 6 other files
├── architecture.md          ← links to data-pipeline, api-spec
├── data-pipeline.md         ← links to architecture
├── api-spec.md              ← standalone
├── pricing-model.md         ← links to product-vision
├── development-roadmap.md   ← links to architecture, pricing
└── glossary.md              ← orphan (no links in or out)
```

The adapter would:
1. Identify `product-vision.md` as the index (most outgoing links)
2. Map the full link graph
3. Flag `glossary.md` as an orphan
4. Assemble content with `product-vision.md` first, then linked files, then `glossary.md` last
5. Warn about the orphan in the metadata

---

*Vault Adapter v2.0 — Primary Input Adapter for R&R*
