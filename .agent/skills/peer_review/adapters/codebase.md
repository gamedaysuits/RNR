# Codebase Adapter

> **Purpose:** Handle review of source code repositories and projects.
> Focuses on architecture, structure, and documentation quality — not line-by-line code review.

---

## Input

A path to a directory containing source code.

```
/peer-review ./my-project/ --type codebase
/peer-review ./src/
```

---

## Detection

This adapter is selected when:
- Input is a directory path
- Directory contains a recognized project manifest:
  `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pubspec.yaml`,
  `Gemfile`, `pom.xml`, `build.gradle`, `Makefile`, `CMakeLists.txt`,
  `requirements.txt`, `setup.py`, `composer.json`

---

## Assembly Process

```
FUNCTION assemble_codebase(directory_path):

  1. GENERATE FILE TREE
     - Recursive listing with indentation
     - Exclude: .git/, node_modules/, __pycache__/, .venv/, build/, dist/,
       .next/, .nuxt/, coverage/, .DS_Store, *.pyc, *.lock
     - Show file sizes for context
     - Cap tree depth at 4 levels (summarize deeper paths)

  2. IDENTIFY KEY FILES (in priority order)
     a. README.md / README.txt / README
     b. Entry points: main.*, index.*, app.*, src/main.*, src/index.*
     c. Config: package.json, tsconfig.json, pyproject.toml, etc.
     d. Documentation: docs/*.md, CONTRIBUTING.md, ARCHITECTURE.md
     e. Test entry: test/*, tests/*, spec/*, __tests__/*

  3. EXTRACT DEPENDENCY SUMMARY
     - Parse project manifest for dependencies
     - List direct dependencies with versions
     - Note dev vs production dependencies
     - Flag outdated or deprecated packages (if detectable)

  4. ASSEMBLE REVIEW PACKAGE
     - File tree first
     - Dependency summary
     - Key files with full content
     - Summary of remaining files (names only, not content)

  RETURN Review Package:
    input_type: "codebase"
    input_target: directory_path
    file_count: N (key files included in full)
    total_word_count: sum of included file word counts
    files: [array of key file metadata]
    content: assembled content
    metadata:
      file_tree: full tree string
      dependencies: parsed dependency list
      warnings: [size warnings, missing README, etc.]
```

---

## Output Format

```markdown
# Codebase Review Package

**Source:** {{directory_path}}
**Key Files Included:** {{N}}
**Total Files in Project:** {{M}}

---

## Project Structure

```
{{indented file tree}}
```

## Dependencies

| Package | Version | Type |
|---------|---------|------|
| react | ^18.2.0 | production |
| typescript | ^5.0.0 | dev |
| ... | ... | ... |

---

## Key Files

---
📄 **File: README.md** (Role: documentation | 350 words)
---

[file content]

---
📄 **File: src/index.ts** (Role: entry-point | 120 words)
---

[file content]

---
📄 **File: package.json** (Role: config)
---

[file content]

---

## Additional Files (not included in full)

{{list of remaining files by directory, with brief descriptions if detectable}}
```

---

## Codebase-Specific Validation

| Check | Threshold | Action |
|-------|-----------|--------|
| Directory exists | - | Reject |
| Contains recognizable files | ≥ 1 non-config file | Reject ("Empty or unrecognizable project") |
| Has README | - | Warn ("No README found — reviewers will have limited project context") |
| Total key file tokens | > 50k | Warn ("Large codebase. Consider using a manifest to select key files.") |

---

## Reviewer Guidance

```markdown
> **💻 Codebase Review Mode**
>
> You are reviewing a source code project. Focus on:
> - **Architecture** — Is the project structure logical and maintainable?
> - **Documentation** — Is the README comprehensive? Are key decisions documented?
> - **Dependencies** — Are choices reasonable? Any risks or bloat?
> - **Entry points** — Is the code organization clear from the top level?
> - **Test coverage** — Is there evidence of a testing strategy?
> - **Configuration** — Are configs sensible and well-documented?
```

---

## Scope Note

This adapter is designed for **architectural and structural review**, not line-by-line code review. For detailed code review, use the `code` roster with a manifest adapter that targets specific files.

---

*Codebase Adapter v2.0*
