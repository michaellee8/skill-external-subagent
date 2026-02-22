# README.md Design

## Overview

Populate the project README.md with a developer-concise overview, quick install one-liner, usage examples, file structure, and how-it-works summary.

## Quick Install Command

```bash
command -v curl >/dev/null || { echo "curl is required" >&2; exit 1; }; BASE_URL=https://raw.githubusercontent.com/michaellee8/skill-external-subagent/main/external-subagents; for dir in .claude .agents .gemini; do if [ -d "$dir" ]; then mkdir -p "$dir/skills/external-subagents/references" && ok=1 && for file in SKILL.md references/claude-code.md references/codex-cli.md references/gemini-cli.md; do curl -fsSL --retry 3 "$BASE_URL/$file" -o "$dir/skills/external-subagents/$file" || { echo "Failed to download $file" >&2; ok=0; }; done && [ "$ok" = 1 ] && echo "Installed into $dir/skills/external-subagents/"; fi; done
```

Reviewed by Codex (GPT-5.2) and Gemini for bash/curl best practices. Key features:
- `command -v curl` check upfront
- `curl -fsSL --retry 3` with proper error handling
- `ok` flag to suppress misleading success messages on partial failure
- `if/then/fi` for clearer flow
- Mirrors source directory structure into target

## README Sections

1. **Title + description** — one-line summary
2. **Quick Install** — the one-liner with brief note
3. **Usage** — 3 example prompts
4. **What's included** — 4 files listed
5. **How it works** — 4-step summary with link to SKILL.md

## Design Decisions

- Developer-concise tone (no badges, no beginner explanations)
- Single one-liner handles all three agent directories
- No separate per-tool install commands
- Link to SKILL.md for full details rather than duplicating content
