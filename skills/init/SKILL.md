---
name: init
description: Initialize or update repository-level agent guidance. In Codex, create or refresh AGENTS.md while mining CLAUDE.md and other AI context files.
---

# Init Skill

Respond to the user in Russian.

Create or update an `AGENTS.md` file - a "readme for coding agents" that accelerates understanding for future AI sessions in Codex. If the repository already has a `CLAUDE.md`, treat it as source material and preserve reusable wording where possible so the guidance stays portable across Codex, Pi, and Claude Code.

## Workflow

### 1. Check Existing Context

Before anything else, look for existing context files in order of priority:
- `AGENTS.md` - primary target in Codex, update if it exists
- `CLAUDE.md` - if found, use it as source material for `AGENTS.md`
- `.cursorrules`, `.cursor/rules/` - extract relevant conventions
- `.github/copilot-instructions.md` - extract relevant conventions
- `README.md` - extract project-specific details

If `AGENTS.md` already exists, read it first and merge new insights without overwriting human-authored sections. If `CLAUDE.md` exists too, preserve the strongest portable parts so future sync across systems stays cheap.

### 2. Autonomous Discovery

Explore the repository to build a mental model. You decide which files to read.

- **Identify Stack**: Languages, frameworks, build tools, and their versions.
- **Extract Workflows**: Commands for installing dependencies, running tests (including a single test), building, linting, and starting the project.
- **Map Structure**: Source roots, test directories, config files, entry points.
- **Infer Conventions**: Read sample source files to identify coding styles, architectural patterns (e.g., "functional components", "repository pattern"), naming conventions, and error handling approaches.
- **Find Gotchas**: Non-obvious things that would trip up an AI - monorepo quirks, custom build steps, environment requirements, tricky test setup.

### 3. Generate AGENTS.md

Write the file using this structure as a baseline. Adapt sections to the project's actual needs - skip irrelevant sections, add project-specific ones.

```markdown
# AGENTS.md

This file provides guidance to Codex and other AI agents when working with this repository.

## Overview

{What this project is and does, in 1-3 sentences}

## Stack

{Languages, frameworks, key dependencies with versions if important}

## Project Structure

{Only non-obvious structure. Skip if it's a standard layout for the framework.}
- `{dir}/` - {what's in it and why it matters}

## Development

{Commands for common workflows. Only include what's non-obvious or project-specific.}

- Install: `{cmd}`
- Build: `{cmd}`
- Test all: `{cmd}`
- Test single: `{cmd}`
- Lint: `{cmd}`
- Run: `{cmd}`

## Architecture

{High-level patterns that require reading multiple files to understand. Data flow, key abstractions, how modules connect. This is the most valuable section - focus here.}

## Conventions

{Coding standards actually enforced in this project — not generic best practices. Include formatter/linter config, naming patterns, file organization rules.}

## Gotchas

{Things that are surprising or non-obvious. Environment requirements, workarounds, known issues, "don't touch this because..." notes.}
```

If the user explicitly wants Claude Code compatibility too, you may additionally update `CLAUDE.md`, but the default Codex output is `AGENTS.md`.

### 4. Quality Rules

- **No fluff**: Don't include generic advice like "write clean code" or "handle errors properly."
- **No obvious info**: Don't list every file or describe standard framework structure.
- **No fabrication**: Only document what you actually found in the codebase. Never invent "Common Tasks" or "Tips" sections.
- **Be specific**: Instead of "follow the existing style," say "use camelCase for functions, PascalCase for components, 2-space indent."
- **Focus on architecture**: The "Architecture" section is the highest-value content - it captures knowledge that takes the longest to rediscover.
- **Mention other context files**: If `.cursorrules` or similar files exist, note their presence so future sessions know to check them.
- **Preserve portability**: If `CLAUDE.md` contains strong reusable guidance, keep wording close enough that it can be copied between Codex, Pi, and Claude Code without a full rewrite.
