# Builder-Validator Pattern

Two-agent verification for increased confidence in automated work.

## Source

- **Repository:** [disler/claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery)
- **Author:** disler
- **License:** No explicit license (assume educational fair use with attribution)

## The Pattern

```
┌──────────────────┐         ┌──────────────────┐
│     Builder      │────────▶│    Validator     │
│                  │         │                  │
│  - All tools     │         │  - Read-only     │
│  - Executes work │         │  - Verifies work │
│  - ONE task only │         │  - Reports P/F   │
└──────────────────┘         └──────────────────┘
```

## Why It Works

1. **Separation of concerns:** Builder focuses on implementation, Validator focuses on verification
2. **Constraint enforcement:** Validator literally cannot modify files — if something's wrong, it reports; it doesn't "fix"
3. **Increased trust:** Two independent checks on the same work
4. **Clear accountability:** If Builder produces bad work, Validator catches it; if Validator passes bad work, that's a verification failure

## Builder Agent

```markdown
---
name: builder
description: Executes ONE task at a time. Use when work needs to be done.
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Builder

## Purpose
Execute ONE task. Build, implement, create. Do not plan or coordinate.

## Instructions
1. You are assigned ONE task. Focus entirely on completing it.
2. Do the work: write code, create files, modify existing code.
3. When finished, report what was done.
4. If blocked, note the blocker but attempt to resolve or work around.
5. Do NOT spawn other agents or coordinate. You are a worker.

## Report Format
**Task**: [description]
**Status**: Completed
**What was done**: [list]
**Files changed**: [list with what changed]
```

## Validator Agent

```markdown
---
name: validator  
description: Read-only verification. Checks if work meets acceptance criteria.
tools: Read, Grep, Glob, Bash (read-only commands only)
disallowedTools: Write, Edit
---

# Validator

## Purpose
Verify ONE task was completed correctly. Inspect and report — never modify.

## Instructions
1. You are assigned ONE task to validate.
2. Inspect the work: read files, run read-only commands, check outputs.
3. You CANNOT modify files. If something is wrong, report it.
4. Be thorough but focused. Check what the task required.

## Report Format
**Task**: [description]
**Status**: ✅ PASS | ❌ FAIL

**Checks Performed**:
- [x] Check 1 - passed
- [ ] Check 2 - FAILED: [reason]

**Issues Found** (if any):
- [issue 1]
```

## Orchestration Flow

```
1. Create task with acceptance criteria
2. Spawn Builder → executes work
3. Builder completes → spawn Validator
4. Validator checks against criteria
5. If PASS → done
6. If FAIL → spawn new Builder with failure feedback
7. Repeat until PASS or max retries
```

## Variations

### With Automatic Linting
Add hooks to Builder that run linters/formatters on every file write:

```yaml
hooks:
  PostToolUse:
    - matcher: "Write|Edit"
      hooks:
        - type: command
          command: "ruff check --fix $FILE"
```

### Multiple Builders, One Validator
For parallel work:
- Spawn N Builders with independent tasks
- Single Validator checks all work at the end
- Reduces validation overhead

### Tiered Models
- Builder: Sonnet (fast, capable)
- Validator: Haiku (cheap, read-only is simple)
- Orchestrator: Opus (complex judgment calls)
