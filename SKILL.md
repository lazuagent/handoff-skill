---
name: handoff-skill
description: Keep resumable task handoffs during long agent work.
version: 0.1.0
author: Marcus, Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [handoff, continuity, checkpoints, model-switching]
    related_skills: []
---

# Handoff Skill

Keep a durable, concise handoff file during long work so another model or session can continue without restarting. This skill records executed facts and next actions; it does not claim that unexecuted plans are complete.

## When to Use

Use when the user says any equivalent of:

- “Use the handoff-skill.”
- “Keep this ready for a model switch.”
- “My tokens may run out.”
- “Make sure another model can continue.”

Do not use for a tiny one-response request unless the user explicitly asks.

## Handoff Location

1. Prefer an existing repository-level `HANDOFF.md`.
2. Otherwise create `HANDOFF.md` in the active project root.
3. If there is no project, use `handoffs/<session-id>.md` under the active Hermes profile home.
4. Never place secrets, passwords, access tokens, private keys, or full credential-bearing commands in the handoff.

One active project may use one `HANDOFF.md`. Separate unrelated projects must not overwrite each other.

## Required Handoff Format

Maintain these headings:

```markdown
# Task Handoff

## Objective
## User Requirements
## Current Status
## Completed and Verified
## Files Changed
## Commands and Results
## Decisions and Assumptions
## Blockers and Risks
## Exact Next Step
## Continuation Instructions
## Last Checkpoint
```

Under `Last Checkpoint`, record an ISO-8601 timestamp, current model when known, current working directory, and whether work is active, blocked, or complete.

## Procedure

1. At activation, locate an existing handoff with `search_files`. Read it with `read_file` before changing it. Completion criterion: previous state is either loaded or confirmed absent.
2. Create or refresh the file immediately with `write_file` for a new file or `patch` for an existing file. Record the current objective, requirements, known state, and exact next action. Completion criterion: a new model can identify what to do next without reading chat history.
3. Continue the actual task. After every material milestone, decision, failure, successful test, deployment, or changed file, update the handoff before starting the next risky or lengthy phase. Completion criterion: every material result is represented accurately.
4. If work lasts longer than three minutes, treat three minutes as the maximum checkpoint interval while the agent is actively able to call tools. Do not interrupt a non-interruptible provider or tool call merely to update the file; checkpoint immediately after control returns. Completion criterion: no tool-controlled work interval exceeds three minutes without either a checkpoint or a documented non-interruptible operation.
5. Before model switching, context exhaustion, intentional pause, risky long-running work, or ending an incomplete turn, update `Exact Next Step` and `Continuation Instructions` first. Completion criterion: the successor has one precise first action, including relevant path or command.
6. On resumption, read the handoff before acting, verify its claims against current files, processes, git state, or external state where relevant, then continue from `Exact Next Step`. Do not repeat completed work unless verification shows it is missing or broken.
7. When the task is finished, mark status `complete`, record verification results, and preserve the file unless the user explicitly asks to remove it. Never delete it automatically.

## Continuation Instructions Template

Write operational instructions, not a narrative:

- Read this entire file before acting.
- Verify the current state; do not trust stale process IDs or deployment claims.
- Continue from `Exact Next Step`.
- Do not redo entries under `Completed and Verified` unless verification fails.
- Preserve the user's constraints and decisions.
- Update this handoff after the next material result.

## Checkpoint Quality Rules

- Record facts from tool output, not guesses.
- Distinguish `planned`, `in progress`, `blocked`, and `verified`.
- Include exact relative file paths and commands where useful.
- Summarize command output; include the decisive error or result, not large logs.
- Record all modified files, including uncommitted ones.
- Note background processes only after checking that they are still running.
- Keep the file concise enough for a replacement model to read quickly.
- Never write private chain-of-thought. Record decisions, evidence, constraints, and next actions only.

## Pitfalls

- A skill is agent behavior, not an independent timer. It cannot write while the model is blocked inside a long API/tool call or after the Hermes process stops.
- Loading this skill in one chat does not automatically activate it in every other chat. The user must invoke it there, or install a separate profile-level hook/plugin.
- Skills are profile-local. Install this skill in every Hermes profile that should recognize it.
- A handoff is not proof that external state still exists. Verify deployments, processes, branches, and remote writes on resumption.

## Verification

Before claiming continuity is protected, verify:

- The handoff file exists at the stated path.
- It contains every required heading.
- `Current Status`, `Exact Next Step`, and `Last Checkpoint` are non-empty.
- Every modified file and material verification result is included.
- The continuation instructions tell a replacement model to read, verify, and continue rather than restart.
