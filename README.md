# Prezzly Skills

Agent skills for building HTML presentations compatible with [Prezzly](https://prezzly.io) — including presenter mode, speaker notes, and live slide control.

## Install

```bash
npx skills add tagpilot/prezzly-skills
```

List available skills:

```bash
npx skills add tagpilot/prezzly-skills --list
```

Install a specific skill:

```bash
npx skills add tagpilot/prezzly-skills --skill prezzly-presentations
```

## What's included

| Skill | Description |
|-------|-------------|
| `prezzly-presentations` | Build HTML slide decks with Prezzly runtime, `.slide` convention, `data-notes`, and upload via MCP (`uploadUrl` + curl for assets) |

## Requirements

- Node.js 18+
- An AI agent that supports the [skills CLI](https://github.com/vercel-labs/skills) (Cursor, Claude Code, Codex, etc.)
