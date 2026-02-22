# external-subagents

A skill that teaches AI coding agents (Claude Code, Codex CLI, Gemini CLI) to invoke each other as external subagents for second opinions, code reviews, and cross-model validation.

## Quick Install

Run from your project root. Installs into whichever agent directories (`.claude/`, `.agents/`, `.gemini/`) already exist:

```bash
command -v curl >/dev/null || { echo "curl is required" >&2; exit 1; }; BASE_URL=https://raw.githubusercontent.com/michaellee8/skill-external-subagent/main/external-subagents; for dir in .claude .agents .gemini; do if [ -d "$dir" ]; then mkdir -p "$dir/skills/external-subagents/references" && ok=1 && for file in SKILL.md references/claude-code.md references/codex-cli.md references/gemini-cli.md; do curl -fsSL --retry 3 "$BASE_URL/$file" -o "$dir/skills/external-subagents/$file" || { echo "Failed to download $file" >&2; ok=0; }; done && [ "$ok" = 1 ] && echo "Installed into $dir/skills/external-subagents/"; fi; done
```

## Usage

After installing, ask your agent things like:

- "Get a second opinion from Codex on this auth implementation"
- "Have Gemini review src/api.py for security issues"
- "Ask Claude Code to suggest a better approach for this function"

The agent will detect available CLIs and invoke the requested tool with your task.

## What's Included

- `SKILL.md` — main skill definition with core flow, command templates, and sandbox handling
- `references/claude-code.md` — Claude Code CLI flags and examples
- `references/codex-cli.md` — Codex CLI flags and examples
- `references/gemini-cli.md` — Gemini CLI flags and examples

## How It Works

1. User explicitly asks for an external agent
2. Skill detects which CLIs are installed (`claude`, `codex`, `gemini`)
3. Builds a sandboxed, non-interactive CLI invocation with the task as prompt
4. Executes and reports results back to the user

See [SKILL.md](external-subagents/SKILL.md) for full details.
