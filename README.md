# Agent Skills Discovery via Well-Known URIs

**Status**: Draft  
**Version**: 0.1  
**Date**: 2026-01-17

## Table of Contents

1. [Abstract](#abstract)
2. [Problem](#problem)
3. [Solution](#solution)
4. [URI Structure](#uri-structure)
5. [Skill Directory Contents](#skill-directory-contents)
6. [Progressive Disclosure](#progressive-discovery)
7. [Discovery Index](#discovery-index)
8. [Examples](#examples)
9. [HTTP Considerations](#http-considerations)
10. [Client Implementation](#client-implementation)
11. [Security Considerations](#security-considerations)
12. [Relationship to Existing Specifications](#relationship-to-existing-specifications)
13. [References](#references)

## Abstract

This document defines a mechanism for discovering [Agent Skills](https://agentskills.io/) using the `.well-known` URI path prefix as specified in [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615). Skills are currently scattered across GitHub repositories, documentation sites, in other sources. A well-known URI provides a predictable location for agents and tools to discover skills published by an organization or project.

## Problem

Agent Skills give AI agents domain-specific capabilities through structured instructions, scripts, and resources. Today, discovering skills requires:

- Searching GitHub repositories
- Reading vendor documentation
- Following links shared on social media
- Manual configuration by end users

There is no standard way to answer: "What skills does example.com publish?"

## Solution

Register `skills` as a well-known URI suffix. Organizations can publish skills at:

```
https://example.com/.well-known/skills/
```

This provides a **single, predictable location** where agents and tooling can discover and fetch skills without prior configuration.

## URI Structure

The well-known skills path uses this hierarchy:

```
/.well-known/skills/index.json          # Required: skills index
/.well-known/skills/{skill-name}/       # Skill directory
/.well-known/skills/{skill-name}/SKILL.md
```

The `{skill-name}` segment must conform to the [Agent Skills specification](https://agentskills.io/specification):

- 1-64 characters
- Lowercase alphanumeric and hyphens only (`a-z`, `0-9`, `-`)
- Must not start or end with a hyphen
- Must not contain consecutive hyphens

## Skill Directory Contents

Each skill directory must contain a `SKILL.md` file and may include supporting resources:

```
/.well-known/skills/pdf-processing/
├── SKILL.md           # Required: instructions + metadata
├── scripts/           # Optional: executable code
│   └── extract.py
├── references/        # Optional: documentation
│   └── REFERENCE.md
└── assets/            # Optional: templates, data files
    └── schema.json
```

The `SKILL.md` file must contain YAML frontmatter with `name` and `description` fields, followed by Markdown instructions.

## Progressive Disclosure

Skills use a three-level loading pattern to manage context efficiently:

| Level | What | When Loaded | Token Cost |
|-------|------|-------------|------------|
| 1 | `name` + `description` from index | At startup or when probing | ~100 tokens per skill |
| 2 | Full `SKILL.md` body | When skill is activated | < 5k tokens recommended |
| 3 | Referenced files (scripts, references, assets) | On demand, as needed | Unlimited |

**Level 1: Index metadata.** Agents fetch `index.json` to learn what skills exist and prefetch their files. Only the name and description are loaded into context initially.

**Level 2: Skill instructions.** When a task matches a skill's description, the agent fetches `SKILL.md` and loads its full instructions into context.

**Level 3: Supporting resources.** The `SKILL.md` body references additional files via relative links. Agents fetch these on demand as the task requires - a form-filling task might need `references/FORMS.md`, while a simple extraction task does not.

This pattern means a skill can bundle extensive reference material without paying a context cost upfront. Agents follow links as needed, fetching only what the current task requires.

### Example: Progressive loading

````yaml
---
name: pdf-processing
description: Extract text and tables from PDF files. Use when working with PDFs or document extraction.
---

# PDF Processing

## Quick Start

Use pdfplumber to extract text:

```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Form Filling

For filling PDF forms, see [references/FORMS.md](references/FORMS.md).

## Advanced Table Extraction

For complex tables with merged cells, see [references/TABLES.md](references/TABLES.md) and run `scripts/extract_tables.py`.
````

An agent handling "extract text from this PDF" loads `SKILL.md` and stops there. An agent handling "fill out this tax form" follows the link to `references/FORMS.md`. The table extraction script and reference stay unfetched until needed.

## Discovery Index

Publishers MUST provide an index at `/.well-known/skills/index.json`. The index enumerates all available skills and their files, enabling clients to discover and prefetch skill resources in a single request.

### Index Format

```json
{
  "skills": [
    {
      "name": "wrangler",
      "description": "Deploy and manage Cloudflare Workers projects.",
      "files": [
        "SKILL.md",
        "references/commands.md",
        "references/configuration.md"
      ]
    },
    {
      "name": "code-review",
      "description": "Review code for bugs, security issues, and best practices.",
      "files": [
        "SKILL.md"
      ]
    }
  ]
}
```

The index contains a single `skills` array. Each entry has these fields:

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill identifier. MUST match the directory name under `/.well-known/skills/` and conform to the [Agent Skills naming specification](https://agentskills.io/specification#name-field): 1-64 characters, lowercase alphanumeric and hyphens only, no leading/trailing/consecutive hyphens. |
| `description` | Yes | Brief description of what the skill does and when to use it. Max 1024 characters per the Agent Skills spec. |
| `files` | Yes | Array of all files in the skill directory, enabling clients to prefetch resources. See [Files Array](#files-array) for format requirements. |

Clients derive the skill path from the `name` field directly:

```
/.well-known/skills/{name}/SKILL.md
```

For example, `"name": "wrangler"` maps to `/.well-known/skills/wrangler/SKILL.md`.

### Files Array

The `files` array lists all files in the skill directory. This enables clients to prefetch and locally cache skill resources, eliminating network requests during task execution.

**Requirements:**

- The array MUST be non-empty
- The array MUST include `SKILL.md`
- `SKILL.md` SHOULD be the first entry
- Paths MUST be relative to the skill directory
- Paths MUST use forward slash (`/`) as the separator
- Paths MUST NOT begin with `/` or contain `..` segments
- Paths MUST contain only printable ASCII characters (0x20-0x7E), excluding `\`, `?`, `#`, `[`, `]`, and control characters
- Each path MUST correspond to an actual file served at `/.well-known/skills/{name}/{path}`

**Example paths:**

```
SKILL.md                    # Required
scripts/deploy.sh           # Script in scripts/ directory  
references/API.md           # Reference documentation
assets/config.template.yaml # Asset file
```

**Caching and progressive disclosure.** Clients MAY prefetch all files listed in the `files` array for local caching. However, clients MUST NOT load all files into context simultaneously. The [progressive disclosure model](https://agentskills.io/specification#progressive-disclosure) still applies: load `SKILL.md` first, then fetch supporting resources on demand as the task requires.

## Examples

### Simple skill (SKILL.md only)

A minimal skill contains just the required `SKILL.md`:

````
GET /.well-known/skills/git-workflow/SKILL.md

---
name: git-workflow
description: Follow team Git conventions for branching and commits.
---

# Git Workflow

Create feature branches from `main`:

```bash
git checkout -b feature/my-feature main
```

Commit messages use conventional commits format:

```
feat: add user authentication
fix: resolve null pointer in login
docs: update API reference
```
````

### Complex skill with resources

A skill with scripts and reference documentation:

```
/.well-known/skills/data-pipeline/
├── SKILL.md
├── scripts/
│   ├── validate.py
│   └── transform.py
├── references/
│   ├── SCHEMA.md
│   └── ERROR_CODES.md
└── assets/
    └── config.template.yaml
```

The `SKILL.md` references these files for progressive disclosure:

```yaml
---
name: data-pipeline
description: Build and validate data pipelines. Use when processing datasets or ETL workflows.
---

# Data Pipeline

## Validation

Run `scripts/validate.py` against your dataset before processing.

For schema requirements, see [references/SCHEMA.md](references/SCHEMA.md).
```

## HTTP Considerations

Servers MUST:

- Serve `/.well-known/skills/index.json` with `application/json` content type
- Serve `SKILL.md` files with `text/markdown` or `text/plain` content type
- Support `GET` and `HEAD` methods
- Return `404 Not Found` for skills or files that do not exist

Servers SHOULD:

- Set appropriate `Cache-Control` headers

Clients MUST:

- Handle redirects (3xx responses)
- Respect cache headers

## Client Implementation

Clients discovering skills from a well-known endpoint MUST:

1. **Fetch `index.json`.** Retrieve `/.well-known/skills/index.json` to enumerate available skills and their files.

2. **Prefetch skill files.** Use the `files` array to download all resources for discovered skills. Cache locally to avoid network requests during task execution.

3. **Apply progressive disclosure.** Load only `name` and `description` at discovery time. Load `SKILL.md` when a skill is activated. Load supporting resources (scripts, references, assets) on demand as the task requires.

4. **Resolve relative paths.** Files listed in the `files` array are relative to the skill directory. Resolve against the skill URL:
   - Skill: `/.well-known/skills/wrangler/`
   - File entry: `scripts/deploy.sh`
   - Resolved URL: `/.well-known/skills/wrangler/scripts/deploy.sh`

5. **Cache aggressively.** Skills change infrequently. Respect `Cache-Control` headers and consider caching content for the duration of a session.

6. **Gate script execution.** Skills may include executable scripts. Clients SHOULD prompt for user confirmation before running any code from a skill, or require explicit opt-in via configuration. Consider sandboxing execution environments and restricting filesystem/network access. Never execute scripts from untrusted origins without user approval.

## Security Considerations

The security considerations from [RFC 8615 Section 4](https://datatracker.ietf.org/doc/html/rfc8615#section-4) apply. Additional considerations for skills:

- **Trust**: Skills contain instructions and executable code. Agents should only use skills from trusted origins. See the [Agent Skills security guidance](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#security-considerations).
- **Access control**: Servers should control write access to `/.well-known/skills/` carefully, especially in shared hosting environments.
- **Script execution**: Agents executing scripts from skills should sandbox execution appropriately.
- **External references**: Skills that fetch external resources introduce additional trust boundaries.

## Relationship to Existing Specifications

This document builds on:

- [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615) - Well-Known URIs
- [Agent Skills Specification](https://agentskills.io/specification) - Skill format and structure

## References

- [Agent Skills](https://agentskills.io/) - Open standard for agent skills
- [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615) - Well-Known Uniform Resource Identifiers
- [Claude Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [OpenAI Codex Skills](https://developers.openai.com/codex/skills/)
- [OpenCode Skills](https://opencode.ai/docs/skills/)
