# Prezzly Skills

Agent skills for building HTML presentations compatible with [Prezzly](https://prezzly.ai) - including presenter mode, speaker notes, and live slide control.

Docs: [Install the Prezzly skill](https://prezzly.ai/docs/skills/install/) (also [Polish](https://prezzly.ai/pl/docs/skills/install/)). Connect MCP first: [Connect an MCP client](https://prezzly.ai/docs/mcp/connect/).

## Install

Requires Node.js 18+ and an agent that supports the [skills CLI](https://github.com/vercel-labs/skills) (Cursor, Claude Code, Codex, and others).

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

Scope to one agent or to your user profile when the CLI supports it:

```bash
npx skills add tagpilot/prezzly-skills --agent cursor
npx skills add tagpilot/prezzly-skills -g
```

After install, `SKILL.md` lands in the tree your agent already reads:

| Agent | Typical path |
|-------|--------------|
| Cursor | `.cursor/skills/prezzly-presentations/SKILL.md` or `.agents/skills/prezzly-presentations/SKILL.md` |
| Claude Code | `.claude/skills/prezzly-presentations/SKILL.md` |
| Codex and others | `.agents/skills/prezzly-presentations/SKILL.md` |

Manual install: copy `skills/prezzly-presentations/SKILL.md` into that path and keep the folder name.

## What's included

| Skill | Description |
|-------|-------------|
| `prezzly-presentations` | Build HTML slide decks with Prezzly runtime, `.slide` convention, `data-notes`, and upload via MCP (`uploadUrl` + curl for assets) |

The skill does not replace a live MCP connection. Paste `https://mcp.prezzly.ai` into the client, then ask the agent to publish the deck.
