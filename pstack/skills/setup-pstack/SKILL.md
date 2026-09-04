---
name: setup-pstack
description: "Use when setting up pstack, for 'set up pstack', or checking that pstack's delegation wiring is healthy in ZCode. Confirms subagents spawn correctly, then optionally offers a verification skill."
---

# Setup pstack

ZCode runs one model per session, and the `Agent` tool takes no model parameter,
so every subagent inherits the session model automatically. There is no per-role
model to configure and no model override file. This skill verifies the wiring,
then optionally offers a verification skill.

## Steps

### 1. Confirm the plugin resolves

Check that pstack's skills resolve in this environment: the `poteto-mode` skill
is invocable, and the plugin's `agents/poteto-agent.md` and
`agents/comment-sicko.md` resolve as `subagent_type: "pstack:poteto-agent"` and
`"pstack:comment-sicko"`. If a skill or agent is missing, say what is missing and
how to fix the install (plugin re-install, or user-level skills under
`~/.agents/skills/`).

### 2. Smoke-check delegation

Spawn one trivial `Agent` call (`subagent_type: "general-purpose"`,
`run_in_background: false`) with a read-only prompt ("report the output of
`git log --oneline -1` and nothing else"). Confirm it returns a result on the
session model. If it fails, report the error — do not reconfigure anything;
delegation is platform-managed.

### 3. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a
`verify-*` skill, or an existing harness). If not, offer once: "want a
project-local verification skill, so agents can drive the app the way a user
does and prove changes work? I can generate one with create-verification-skill."
On yes, invoke `create-verification-skill`. On no, move on without pushing.

## Confirm

Tell the user that pstack runs on the session model, that subagents inherit it
automatically, and that no per-role model configuration is needed. Re-running
this skill re-verifies the same setup.
