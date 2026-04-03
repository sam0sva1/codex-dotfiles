# Porting From Claude

This document records how we sync the shared Codex dotfiles from the current `~/.claude` setup.

## Goal

Keep the Codex workflow skills, role skills, and global guidance aligned with the latest Claude Code source material while preserving only the minimal Codex-specific adaptations.

## Source Of Truth

Use the live local Claude files as the source of truth:

- `~/.claude/CLAUDE.md`
- `~/.claude/skills/*/SKILL.md`
- `~/.claude/agents/*.md`

Do not sync from memory, from old commits, or from copied snippets unless the current Claude files are unavailable.

## What Belongs In Shared Git

Track these in the shared repo:

- `AGENTS.md`
- Custom workflow skills under `skills/`
- Custom role skills under `skills/`
- `agents/openai.yaml` metadata for those custom skills
- Porting notes such as this file

Do not track machine-local or Codex-install-local state:

- `config.toml`
- `rules/`
- `skills/.system/`
- auth, sessions, caches, sqlite files, logs, temporary directories

## File Mapping

### Workflow skills

Claude workflow skills map directly:

- `~/.claude/skills/<name>/SKILL.md` -> `skills/<name>/SKILL.md`

Current direct mappings:

- `api-contract-first`
- `brainstorm`
- `cross-service-changes`
- `execute-plan`
- `go-plan`
- `plan-check`
- `preflight`
- `self-review`
- `task-commit`
- `tdd`

### Agent prompts

Claude agents are represented in Codex as skills:

- `~/.claude/agents/architect.md` -> `skills/architect/SKILL.md`
- `~/.claude/agents/critic.md` -> `skills/critic/SKILL.md`
- `~/.claude/agents/debugger.md` -> `skills/debugger/SKILL.md`
- `~/.claude/agents/explorer.md` -> `skills/explorer/SKILL.md`
- `~/.claude/agents/planner.md` -> `skills/high-level-planner/SKILL.md`

We intentionally use `high-level-planner` as the Codex skill directory, and map `@planner` to it in `AGENTS.md`.

### Global guidance

- `~/.claude/CLAUDE.md` -> `AGENTS.md`

`AGENTS.md` should stay as close as possible to `CLAUDE.md`. The Codex-specific compatibility block at the top is the main expected delta.

## Porting Rules

### 1. Prefer exact sync

Default rule: copy Claude wording exactly.

Bring over:

- new sections
- changed constraints
- updated hard gates
- new red flags
- added examples
- changed process steps
- wording refinements that change meaning or emphasis

Do not paraphrase unless Codex compatibility requires it.

### 2. Preserve Codex-only adaptations

Keep these adaptations in Codex even when Claude says something else:

- Use `update_plan` where Claude-specific plan mode or `TodoWrite` is referenced.
- Replace references to `~/.claude/...` paths with the equivalent `~/.codex/...` path when the text points to a local Codex skill file.
- For agent-derived skills, keep a `name:` field in frontmatter because they live as first-class Codex skills.
- Keep the Codex compatibility preface in `AGENTS.md`.
- Keep the `@planner` -> `$high-level-planner` mapping unless the repo is explicitly changed to add a real `planner` alias.

### 3. Keep slash and agent language when useful

Inside synced content, keep references like `/brainstorm`, `/go-plan`, or `@critic` when they are part of the source wording.

The compatibility mapping lives in `AGENTS.md`, so the content itself should stay close to Claude unless a change is required for Codex execution.

### 4. Preserve skill metadata

For workflow skills copied from Claude, keep frontmatter fields from Claude, including:

- `name`
- `description`
- `allowed-tools`
- `argument-hint`

For agent-derived Codex skills, keep:

- `name`
- `description`

Also preserve or update `agents/openai.yaml` when the skill meaning changes enough that the UI metadata or default prompt should change.

### 5. Do not sync Codex system skills from shared git

`skills/.system/` belongs to the local Codex installation and may change with Codex releases. It must not be treated as a shared mirror of Claude content.

If a change is needed in `.system/`, treat it as a local Codex concern, not as part of the Claude sync workflow.

## Recommended Sync Procedure

1. Read the current source files from `~/.claude`.
2. Diff each mapped pair against the Codex target.
3. Port all meaningful content changes.
4. Re-check every pair with `diff -u`.
5. Confirm the only remaining diffs are intentional Codex adaptations.
6. Update `AGENTS.md` if the compatibility mapping changed.
7. Update this document if a new porting rule or exception was discovered.
8. Commit and push the shared repo changes.

## Verification Checklist

After syncing, verify:

- Every Claude workflow skill has a current Codex counterpart.
- Every Claude agent has a current Codex role skill counterpart.
- No hard gate, red flag, or process section was accidentally dropped.
- `AGENTS.md` still matches `CLAUDE.md` except for the Codex compatibility block and other explicit Codex-only adaptations.
- Remaining diffs are only from:
  - `update_plan` replacing Claude-only planning mechanics
  - local path substitutions from `~/.claude` to `~/.codex`
  - `name:` in agent-derived Codex skills
  - the `@planner` mapping to `$high-level-planner`

## Suggested Diff Commands

Workflow skills:

```bash
for name in api-contract-first brainstorm cross-service-changes execute-plan go-plan plan-check preflight self-review task-commit tdd; do
  diff -u "$HOME/.claude/skills/$name/SKILL.md" "$HOME/.codex/skills/$name/SKILL.md" || true
done
```

Agent-derived skills:

```bash
diff -u "$HOME/.claude/agents/architect.md" "$HOME/.codex/skills/architect/SKILL.md" || true
diff -u "$HOME/.claude/agents/critic.md" "$HOME/.codex/skills/critic/SKILL.md" || true
diff -u "$HOME/.claude/agents/debugger.md" "$HOME/.codex/skills/debugger/SKILL.md" || true
diff -u "$HOME/.claude/agents/explorer.md" "$HOME/.codex/skills/explorer/SKILL.md" || true
diff -u "$HOME/.claude/agents/planner.md" "$HOME/.codex/skills/high-level-planner/SKILL.md" || true
```

Global guidance:

```bash
diff -u "$HOME/.claude/CLAUDE.md" "$HOME/.codex/AGENTS.md" || true
```

## Current Known Exceptions

- `execute-plan` uses `update_plan` instead of Claude's `TodoWrite`.
- `go-plan` refers to Codex planning workflow instead of Claude's built-in `/plan` mode.
- `plan-check`, `self-review`, and `task-commit` use natural language like "if the user provided..." instead of literal `$ARGUMENTS`.
- `planner` is represented in Codex as `high-level-planner`.

If new exceptions appear during future syncs, add them here immediately.
