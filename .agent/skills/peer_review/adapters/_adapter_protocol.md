# Adapter Protocol

> **Purpose:** Define the interface contract for all input adapters.
> Every adapter receives raw user input and produces a **Review Package**.

---

## What is an Adapter?

An adapter translates a user's input target (file, directory, manifest) into a normalized **Review Package** — a structured content bundle that reviewers can evaluate regardless of the input's original format.

```
┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
│  User Input  │────▶│  Input Adapter  │────▶│ Review       │
│  (any type)  │     │  (auto-detect)  │     │ Package      │
└──────────────┘     └─────────────────┘     └──────────────┘
```

---

## Review Package Schema

Every adapter must produce output conforming to this structure:

```json
{
  "input_type": "single-file | vault | codebase | manifest",
  "input_target": "/path/to/input",
  "file_count": 1,
  "total_word_count": 500,
  "files": [
    {
      "path": "relative/path/to/file.md",
      "role": "primary | supporting | config | test",
      "word_count": 500
    }
  ],
  "content": "--- assembled markdown content for reviewers ---",
  "metadata": {
    "link_graph": null,
    "file_tree": null,
    "dependencies": null,
    "warnings": []
  }
}
```

### Field Definitions

| Field | Required | Description |
|-------|----------|-------------|
| `input_type` | ✅ | Which adapter produced this package |
| `input_target` | ✅ | Original user input (path, URL, etc.) |
| `file_count` | ✅ | Number of files in the package |
| `total_word_count` | ✅ | Combined word count across all files |
| `files` | ✅ | Array of file metadata |
| `content` | ✅ | The assembled content reviewers will read |
| `metadata.link_graph` | Optional | Cross-reference map (vault adapter) |
| `metadata.file_tree` | Optional | Directory tree string (codebase adapter) |
| `metadata.dependencies` | Optional | Dependency manifest summary (codebase adapter) |
| `metadata.warnings` | ✅ | Non-blocking warnings (large files, unusual extensions, etc.) |

---

## Auto-Detection Logic

When the user provides an input target without `--type`, detect the adapter automatically:

```
FUNCTION detect_adapter(input):

  IF input ends with ".rnr-manifest.json":
    RETURN "manifest"

  ELIF input is a directory:
    files = list_files(input)
    md_count = count files ending in ".md"
    total_count = count all files (excluding hidden, node_modules, .git)

    IF md_count / total_count > 0.6:
      RETURN "vault"
    ELIF contains any of: package.json, pyproject.toml, go.mod, Cargo.toml,
         pubspec.yaml, Gemfile, pom.xml, build.gradle, Makefile, CMakeLists.txt:
      RETURN "codebase"
    ELSE:
      RETURN "vault"   # Default for directories — vault is the primary use case

  ELSE:
    RETURN "single-file"
```

### Override

Users can force a specific adapter with `--type`:

```
/peer-review ./my-project/ --type codebase
/peer-review ./my-docs/ --type vault
```

---

## Content Assembly Rules

### File Boundary Markers

Multi-file packages must use clear boundary markers so reviewers understand file transitions:

```markdown
---
📄 **File: relative/path/to/file.md** (Role: primary)
---

[file content here]

---
📄 **File: relative/path/to/other.md** (Role: supporting)
---

[file content here]
```

### Content Ordering

1. **Primary files first** — The main document(s) that are the subject of review
2. **Supporting files second** — Context, references, appendices
3. **Config/metadata last** — Package manifests, config files (if relevant)

### Size Management

If the assembled content exceeds ~50,000 tokens (estimated as `characters / 4`):

1. Warn the user: "Review package is large (~{N}k tokens). Consider using a manifest to select key files."
2. **Do NOT truncate** — Let the reviewing agent handle context limits
3. Log the warning to the review package metadata

---

## Adapter Implementations

| Adapter | File | Primary Use Case |
|---------|------|-----------------|
| Single File | `single_file.md` | One `.md` or `.txt` file |
| Vault | `vault.md` | Obsidian vaults, wiki-style PRDs, documentation suites |
| Codebase | `codebase.md` | Source code repositories and projects |
| Manifest | `manifest.md` | User-curated multi-file bundles |

---

*Adapter Protocol v2.0 — Input Normalization Layer*
