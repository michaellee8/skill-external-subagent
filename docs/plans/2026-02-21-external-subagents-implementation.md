# External Subagents Skill Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a universal agentskills.io skill that teaches coding agents how to invoke other agent CLIs (Claude Code, Codex CLI, Gemini CLI) as external subagents.

**Architecture:** Single `SKILL.md` with 3 CLI-specific reference files under `references/`. The main skill covers the decision flow and common patterns; reference files provide detailed flag documentation per tool.

**Tech Stack:** Markdown (agentskills.io spec), shell commands

---

### Task 1: Create directory structure

**Files:**
- Create: `external-subagents/SKILL.md` (placeholder)
- Create: `external-subagents/references/` (directory)

**Step 1: Create the skill directory and references subdirectory**

```bash
mkdir -p external-subagents/references
```

**Step 2: Create a minimal placeholder SKILL.md to validate structure**

Create `external-subagents/SKILL.md` with just the frontmatter:

```markdown
---
name: external-subagents
description: Use when the user explicitly asks to invoke another coding agent CLI (Claude Code, Codex CLI, Gemini CLI) to perform tasks like code review, brainstorming, or research from a different model's perspective.
---

# External Subagents

(placeholder)
```

**Step 3: Commit**

```bash
git add external-subagents/
git commit -m "chore: scaffold external-subagents skill directory"
```

---

### Task 2: Write the main SKILL.md

**Files:**
- Create: `external-subagents/SKILL.md`

**Step 1: Write the complete SKILL.md**

Replace the placeholder with the full skill content. The file must include these sections in order:

1. **YAML Frontmatter** - `name: external-subagents`, `description` (triggering conditions only, max 1024 chars), `compatibility` noting shell access requirement
2. **Overview** - One-sentence purpose: teaches agents to invoke other agent CLIs as external subagents when explicitly instructed by the user. Core principle: different models provide different perspectives.
3. **When to Use** - Only when user explicitly asks. List example triggers: "get a second opinion from Codex", "have Gemini review this", "run this through Claude". List when NOT to use: never autonomously, never for tasks the current agent can do itself.
4. **Core Flow** - Numbered steps:
   - Step 1: Detect available CLIs (`which claude codex gemini`)
   - Step 2: If user didn't specify, suggest from what's available
   - Step 3: Check if running in sandbox, prepare to escape if needed
   - Step 4: Build the command (prompt + flags + skill/context paths)
   - Step 5: Execute in background if possible, otherwise foreground
   - Step 6: Report the subagent's stdout output to the user
5. **Command Templates** - One template per CLI tool:
   - Claude Code: `claude -p "..." --dangerously-skip-permissions`
   - Codex CLI: `codex exec --full-auto "..."`
   - Gemini CLI: `gemini -p "..." --approval-mode yolo --sandbox`
6. **Sandbox Escape** - Explain that if the parent agent is sandboxed, the subagent invocation must run outside the sandbox. List per-agent mechanisms:
   - Claude Code: `dangerouslyDisableSandbox: true`
   - Codex CLI: run on host, not inside workspace sandbox
   - Gemini CLI: invoke outside Seatbelt/Docker sandbox
   - Note: the subagent itself still runs sandboxed with full permissions
7. **Passing Context & Skills** - How to enrich the prompt:
   - Tell subagent to read skill files at absolute paths in the prompt text
   - Run subagent in same working directory
   - Mention specific files in the prompt rather than expecting discovery
   - Example prompt showing skill file reference
8. **Background Execution** - Run subagent in background if agent supports it (e.g., Claude Code `run_in_background`, Codex `&` + wait). This allows running multiple subagents in parallel or continuing other work.
9. **Reference** - Links to the 3 reference files:
   - `See [Claude Code reference](references/claude-code.md) for detailed flags.`
   - `See [Codex CLI reference](references/codex-cli.md) for detailed flags.`
   - `See [Gemini CLI reference](references/gemini-cli.md) for detailed flags.`

Target: under 500 lines, under 5000 tokens. Keep it concise - detailed flags go in reference files.

**Step 2: Verify frontmatter conforms to agentskills.io spec**

Check:
- `name` is lowercase, letters/numbers/hyphens only, matches directory name
- `description` is under 1024 characters
- No consecutive hyphens in name
- Name doesn't start or end with hyphen

**Step 3: Commit**

```bash
git add external-subagents/SKILL.md
git commit -m "feat: write main SKILL.md for external-subagents"
```

---

### Task 3: Write references/claude-code.md

**Files:**
- Create: `external-subagents/references/claude-code.md`

**Step 1: Write the Claude Code reference file**

This is a quick-reference (~100-150 lines) covering:

1. **Non-interactive invocation** - `claude -p "prompt"` syntax
2. **Key flags table:**
   - `--dangerously-skip-permissions` - skip all permission prompts (sandboxed full access)
   - `--model <name>` - model override (sonnet, opus, haiku, or full model ID)
   - `--output-format <fmt>` - text (default), json, stream-json
   - `--max-turns <n>` - limit agentic turns
   - `--max-budget-usd <n>` - limit spend
   - `--tools <list>` - restrict available tools (e.g., "Read,Grep,Glob" for read-only)
   - `--allowedTools <list>` - auto-approve specific tools
   - `--add-dir <path>` - add working directories
   - `--append-system-prompt "text"` - append to system prompt
   - `--append-system-prompt-file <path>` - append file to system prompt
   - `--no-session-persistence` - don't save session
3. **Stdin piping** - `cat file | claude -p "prompt"`
4. **Environment variables:**
   - `ANTHROPIC_API_KEY` - API key
   - `ANTHROPIC_MODEL` - default model override
5. **Sandbox escape note** - When parent is sandboxed, use `dangerouslyDisableSandbox: true` on the Bash tool to invoke `claude -p`
6. **Example invocations:**
   - Basic code review
   - Read-only analysis with tool restrictions
   - With budget cap

**Step 2: Commit**

```bash
git add external-subagents/references/claude-code.md
git commit -m "feat: add Claude Code CLI reference for external-subagents"
```

---

### Task 4: Write references/codex-cli.md

**Files:**
- Create: `external-subagents/references/codex-cli.md`

**Step 1: Write the Codex CLI reference file**

Quick-reference (~100-150 lines) covering:

1. **Non-interactive invocation** - `codex exec "prompt"` syntax
2. **Key flags table:**
   - `--full-auto` - workspace-write sandbox + on-request approvals
   - `--yolo` / `--dangerously-bypass-approvals-and-sandbox` - bypass all
   - `-m <model>` / `--model <model>` - model override
   - `--json` - newline-delimited JSON output
   - `-o <path>` / `--output-last-message <path>` - write final message to file
   - `-C <path>` / `--cd <path>` - set working directory
   - `-s <mode>` / `--sandbox <mode>` - read-only, workspace-write, danger-full-access
   - `-a <policy>` / `--ask-for-approval <policy>` - untrusted, on-request, never
   - `--skip-git-repo-check` - allow running outside git repos
   - `--ephemeral` - don't persist session
3. **Stdin piping** - `echo "prompt" | codex exec -`
4. **Environment variables:**
   - `CODEX_API_KEY` - API key (exec mode only)
   - `OPENAI_API_KEY` - alternative API key
   - `OPENAI_BASE_URL` - custom endpoint
5. **Sandbox escape note** - Codex exec runs on host by default; if parent is sandboxed, invoke codex outside the sandbox
6. **Example invocations:**
   - Basic code review with full-auto
   - Read-only analysis
   - With JSON output to file

**Step 2: Commit**

```bash
git add external-subagents/references/codex-cli.md
git commit -m "feat: add Codex CLI reference for external-subagents"
```

---

### Task 5: Write references/gemini-cli.md

**Files:**
- Create: `external-subagents/references/gemini-cli.md`

**Step 1: Write the Gemini CLI reference file**

Quick-reference (~100-150 lines) covering:

1. **Non-interactive invocation** - `gemini -p "prompt"` syntax
2. **Key flags table:**
   - `--approval-mode <mode>` - default, auto_edit, yolo
   - `--sandbox` / `-s` - enable sandboxed execution
   - `-m <model>` / `--model <model>` - model override (gemini-2.5-pro, gemini-2.5-flash, or aliases: auto, pro, flash, flash-lite)
   - `--output-format <fmt>` - text (default), json, stream-json
   - `--yolo` / `-y` - auto-approve all (deprecated in favor of --approval-mode yolo)
   - `--all-files` / `-a` - include all files as context
   - `--include-directories <list>` - comma-separated additional directories
   - `-r <id>` / `--resume <id>` - resume session
   - `--extensions <list>` / `-e` - extensions to enable ("none" to disable all)
3. **Stdin piping** - `cat file | gemini -p "prompt"`
4. **File reference syntax** - `@path/to/file` in prompts
5. **Environment variables:**
   - `GEMINI_API_KEY` - API key
   - `GEMINI_MODEL` - default model
   - `GOOGLE_API_KEY` - Google Cloud auth
   - `GEMINI_SANDBOX` - sandbox config
6. **Sandbox escape note** - When parent is sandboxed, invoke gemini outside the sandbox (Docker/Seatbelt)
7. **Example invocations:**
   - Basic code review with yolo + sandbox
   - With file references
   - With specific output format

**Step 2: Commit**

```bash
git add external-subagents/references/gemini-cli.md
git commit -m "feat: add Gemini CLI reference for external-subagents"
```

---

### Task 6: Final review and validation

**Files:**
- Review: all files in `external-subagents/`

**Step 1: Verify directory structure matches design**

```bash
find external-subagents/ -type f | sort
```

Expected output:
```
external-subagents/SKILL.md
external-subagents/references/claude-code.md
external-subagents/references/codex-cli.md
external-subagents/references/gemini-cli.md
```

**Step 2: Verify SKILL.md frontmatter**

Check that:
- `name: external-subagents` matches directory name
- `description` is under 1024 characters
- Name is lowercase, no consecutive hyphens, doesn't start/end with hyphen

**Step 3: Verify SKILL.md is under 500 lines**

```bash
wc -l external-subagents/SKILL.md
```

Expected: under 500

**Step 4: Verify reference files are ~100-150 lines each**

```bash
wc -l external-subagents/references/*.md
```

**Step 5: Verify all cross-references in SKILL.md point to existing files**

Check that the three reference links in SKILL.md resolve to actual files.

**Step 6: Read through SKILL.md end-to-end for completeness**

Verify all 9 sections from Task 2 are present and coherent.

**Step 7: Commit any fixups if needed**

```bash
git add -A external-subagents/
git commit -m "fix: address review findings in external-subagents skill"
```
