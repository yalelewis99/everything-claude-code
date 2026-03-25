# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** (`ecc-universal`) — a collection of production-ready agents, skills, hooks, commands, rules, and MCP configurations. The project provides battle-tested workflows for software development using Claude Code and is distributed as an npm package with a CLI (`ecc`).

## Commands

```bash
# Lint (ESLint + markdownlint)
npm run lint

# Full test suite (CI validators + unit tests)
npm test

# Run individual unit test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js

# Code coverage (80% threshold)
npm run coverage

# Validate harness configuration
npm run harness:audit
```

The full `npm test` runs validators first (`scripts/ci/validate-*.js`) then unit tests (`tests/run-all.js`). When adding new agents, commands, rules, skills, or hooks, always run `npm test` — the validators enforce structural correctness.

## Architecture

```
agents/          # Specialized subagents (Markdown + YAML frontmatter)
commands/        # Slash commands (/tdd, /plan, /e2e, /code-review, etc.)
hooks/           # hooks.json — trigger-based automations
rules/           # Always-follow guidelines loaded into Claude's context
skills/          # Curated knowledge modules (each has skills/<name>/SKILL.md)
schemas/         # JSON schemas for validation (agents, hooks, provenance, etc.)
manifests/       # Install manifests (install-profiles.json, install-modules.json)
scripts/
  ci/            # Validation scripts run in npm test
  lib/           # Shared Node.js utilities (package-manager.js, utils.js, etc.)
  hooks/         # Hook runtime scripts
mcp-configs/     # MCP server configurations (e.g. Context7)
tests/           # Unit tests for scripts/lib utilities
```

### Install System

`manifests/install-profiles.json` defines five profiles (`core`, `developer`, `security`, `research`, `full`) composed from modules in `manifests/install-modules.json`. Use `npx ecc <profile>` to install. The `full` profile is the recommended default.

### Cross-Harness Support

ECC ships content subsets for other AI harnesses:
- **Codex:** `.agents/skills/` + `agents/openai.yaml`
- **Cursor:** `.cursor/skills/` + `.cursor/rules/`
- **OpenCode:** `.opencode/` directory

When adding skills, consider whether they should also be added to these subsets.

### Skill Placement

| Type | Location | Shipped |
|------|----------|---------|
| Curated | `skills/<name>/SKILL.md` (repo) | Yes |
| Learned | `~/.claude/skills/learned/` | No |
| Imported | `~/.claude/skills/imported/` | No |
| Evolved | `~/.claude/homunculus/evolved/skills/` | No |

Only curated skills in `skills/` are included in install manifests. Learned/imported skills require a `.provenance.json` sibling. See `docs/SKILL-PLACEMENT-POLICY.md`.

## File Formats

**Agents** (`agents/*.md`):
```yaml
---
name: agent-name
description: When Claude should invoke this agent (be specific)
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet  # haiku | sonnet | opus
---
```

**Skills** (`skills/<name>/SKILL.md`):
```yaml
---
name: skill-name
description: Brief description
origin: ECC
---
```

**Commands** (`commands/*.md`):
```yaml
---
description: Shown in /help
---
```

**Hooks** (`hooks/hooks.json`): JSON with `hooks.PreToolUse`, `hooks.PostToolUse`, `hooks.SessionStart`, `hooks.Stop` arrays, each entry having `matcher`, `hooks`, and `description`.

## Development Notes

- **File naming:** lowercase with hyphens (`python-reviewer.md`, `tdd-workflow.md`)
- **Commit style:** conventional commits (`feat:`, `fix:`, `docs:`, `test:`)
- **Package manager detection:** npm/pnpm/yarn/bun via `CLAUDE_PACKAGE_MANAGER` env var or project config (`scripts/lib/package-manager.js`)
- **Node.js ≥ 18** required
- **MCP integration:** agents that query live docs include `mcp__context7__resolve-library-id` and `mcp__context7__query-docs` in their tools list; see `skills/documentation-lookup/`
