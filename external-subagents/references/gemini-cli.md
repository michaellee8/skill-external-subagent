# Gemini CLI Reference

Non-interactive usage reference for Google's Gemini CLI when invoking it as an external subagent.

## Non-interactive Invocation

```bash
gemini -p "Your prompt here."
```

The `-p` flag runs the prompt non-interactively and prints the result to stdout.
Gemini operates on the current working directory by default.

## Key Flags

| Flag | Description |
|------|-------------|
| `-p "prompt"` | Non-interactive mode. Run prompt and exit. |
| `--approval-mode <mode>` | Execution approval policy. Values: `default` (ask for approval), `auto_edit` (auto-approve edits, ask for commands), `yolo` (auto-approve everything). |
| `--sandbox` / `-s` | Enable sandboxed execution. Runs commands in an isolated environment (Docker on Linux, Seatbelt on macOS). |
| `--model <model>` / `-m <model>` | Override the model. Accepts full names (`gemini-2.5-pro`, `gemini-2.5-flash`) or aliases (`auto`, `pro`, `flash`, `flash-lite`). |
| `--output-format <fmt>` | Output format. Values: `text` (default), `json`, `stream-json`. |
| `--yolo` / `-y` | Auto-approve all actions. Shorthand for `--approval-mode yolo`. |
| `--all-files` / `-a` | Include all files in the working directory as context. |
| `--include-directories <list>` | Comma-separated list of additional directories to include as context. |
| `--resume <id>` / `-r <id>` | Resume a previous session by its ID. |
| `--extensions <list>` / `-e <list>` | Extensions to enable. Pass `"none"` to disable all extensions. |

### Approval Modes

- **`default`** -- Asks for user approval before executing commands or making edits. Not suitable for non-interactive use.
- **`auto_edit`** -- Auto-approves file edits but still asks before running shell commands.
- **`yolo`** -- Auto-approves all actions (edits and commands). Required for fully non-interactive subagent use.

### Recommended Flags for Subagent Use

```bash
gemini -p "prompt" --approval-mode yolo --sandbox
```

This combination gives full autonomy (`yolo`) while keeping execution sandboxed (`--sandbox`).

## Stdin Piping

Pipe content into Gemini via stdin combined with `-p`:

```bash
cat src/main.py | gemini -p "Review this code for bugs."
```

```bash
git diff HEAD~1 | gemini -p "Summarize these changes."
```

Stdin content is available to the model alongside the prompt.

## File Reference Syntax

Use `@path/to/file` inside prompts to reference specific files:

```bash
gemini -p "Review @src/auth.py and @src/middleware.py for security issues." \
  --approval-mode yolo --sandbox
```

The `@` prefix tells Gemini to read and include the file contents as context.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `GEMINI_API_KEY` | API key for authenticating with the Gemini API. |
| `GEMINI_MODEL` | Default model to use (overridden by `-m` flag). |
| `GOOGLE_API_KEY` | Google Cloud API key for authentication. |
| `GEMINI_SANDBOX` | Sandbox configuration. Controls default sandbox behavior. |

## Sandbox Escape Note

When the parent agent is running inside a sandbox, the `gemini` binary or its
sandbox runtime (Docker/Seatbelt) may not be accessible from within the sandbox.
Use the parent agent's sandbox-escape mechanism to invoke the `gemini` command on
the host. The subagent itself still runs sandboxed via its own `--sandbox` flag.

**Example for Claude Code as parent agent:**

Set `dangerouslyDisableSandbox: true` on the Bash tool call that runs the
`gemini -p` command. This escapes the parent sandbox for the invocation only --
Gemini's own `--sandbox` flag keeps the subagent's execution isolated.

## Example Invocations

### Basic Code Review (yolo + sandbox)

```bash
gemini -p "Review the code in this repository for potential bugs, \
security issues, and performance problems. Provide actionable suggestions." \
  --approval-mode yolo --sandbox
```

### With File References

```bash
gemini -p "Compare @src/old_parser.py and @src/new_parser.py. \
Identify behavioral differences and potential regressions." \
  --approval-mode yolo --sandbox
```

### With Specific Output Format

```bash
gemini -p "List all TODO comments in this project with file paths and line numbers." \
  --approval-mode yolo --sandbox \
  --output-format json
```

### With Model Override and Additional Directories

```bash
gemini -p "Analyze the API contracts between these services." \
  --approval-mode yolo --sandbox \
  -m gemini-2.5-pro \
  --include-directories ../service-a,../service-b
```

### Piped Input with All Files

```bash
git log --oneline -20 | gemini -p "Summarize the recent changes in this project." \
  --approval-mode yolo --sandbox --all-files
```
