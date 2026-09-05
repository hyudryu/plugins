---
name: setup-pstack
description: "Use when setting up pstack, for 'set up pstack', or checking that pstack's delegation wiring is healthy in DeepSeek Harness. Confirms the skills load and subagents spawn, then optionally offers a verification skill."
---

# Setup pstack

dsh runs the session model, and `subagent` children inherit it by default, so
there is no per-role model to configure. This skill verifies the wiring, then
optionally offers a verification skill.

## Steps

### 1. Confirm the skills load

Check that pstack's skills resolve in this environment: the `skill` tool's
catalog lists `poteto-mode` (and the rest), resolved from the project's
`.dsh/skills/` or the user's `~/.agents/skills/`. If the catalog is missing
them, say where they were expected and how to fix the install (copy the
`skills/` directories into one of those roots and restart `dsh`).

### 2. Smoke-check delegation

Spawn one trivial `subagent` call with a read-only prompt ("report the output
of `git log --oneline -1` and nothing else"). Confirm it returns a result on
the session model. If it fails, report the error — do not reconfigure
anything; delegation is platform-managed.

### 3. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a
`verify-*` skill, or an existing harness). If not, offer once: "want a
project-local verification skill, so agents can drive the app the way a user
does and prove changes work? I can generate one with create-verification-skill."
On yes, invoke `create-verification-skill` (it writes to
`.dsh/skills/verify-<app>/`). On no, move on without pushing.

## Confirm

Tell the user that pstack runs on the session model, that `subagent` children
inherit it by default, and that no per-role model configuration is needed.
Re-running this skill re-verifies the same setup.
