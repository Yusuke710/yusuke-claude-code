---
name: codex
description: |
  Use Codex CLI (OpenAI) to consult and review code or wording.
  Triggers: "codex", "consult codex", "ask codex", "code review", "review this"
  Use cases: (1) Wording/message review, (2) Code review, (3) Design consultation, (4) Bug investigation, (5) Difficult problem investigation
---

# Codex

A skill for executing code review and analysis using Codex CLI.

## Command

```bash
codex exec --full-auto --sandbox read-only --cd <project_directory> "<request>"
```

## Parameters

| Parameter | Description |
|-----------|-------------|
| `--full-auto` | Run in fully automatic mode |
| `--sandbox read-only` | Read-only sandbox (for safe analysis) |
| `--cd <dir>` | Target project directory |
| `"<request>"` | Request content |

## Examples

### Code Review
```bash
codex exec --full-auto --sandbox read-only --cd /path/to/project "Review this project's code and point out areas for improvement"
```

### Bug Investigation
```bash
codex exec --full-auto --sandbox read-only --cd /path/to/project "Investigate the cause of errors in the authentication process"
```

## Workflow

1. Receive request from user
2. Identify the target project directory
3. Execute Codex using the command format above
4. Report results to user
