# 🎯 Perks

Personal collection of agentic skills for Claude Code, Antigravity, Cursor, Codex CLI, and other AI coding assistants.

## Skills

| Skill | Description |
|-------|-------------|
| **deep-plan** | Structured clarification before decisions. Ensures agent asks questions instead of making autonomous decisions when multiple valid approaches exist. |

## Installation

```bash
# Using npx
npx skills add idrevnii/perks

# Using bunx
bunx skills add idrevnii/perks

# Install specific skill only
npx skills add idrevnii/perks --skill "deep-plan"
```

## Compatibility

These skills work with any AI coding assistant that supports the universal `SKILL.md` format:

- 🟣 Claude Code
- 🔵 Antigravity
- 🟠 Cursor
- ⚪ Codex CLI
- 🩵 GitHub Copilot

## Usage

Skills can activate automatically based on context, but **explicit invocation is recommended** for best results:

```
@deep-plan let's discuss the architecture
```

The agent may also auto-trigger `deep-plan` when:
- You're in planning mode
- You ask to "discuss" or "plan" something
- Agent faces decisions without clear context

## License

MIT
