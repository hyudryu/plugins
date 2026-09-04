# Set up pstack

In this page you install the plugin, pick which models pstack uses, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

In a ZCode session, set up the plugin (or user-level skills):

```text
/add-plugin pstack
```

ZCode loads the pstack skills.

## Pick your models

Run:

```text
/setup-pstack
```

[`setup-pstack`](../../skills/setup-pstack/SKILL.md) confirms the delegation wiring: ZCode runs one model per session and subagents inherit it automatically, so there's no per-role model file to maintain. It then optionally offers a verification skill.

You only confirm what you care about. There are no per-role overrides to track—the single configured model applies everywhere.

## Accept the verification offer, or don't

At the end of setup, `setup-pstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers when it earns its place.

After setup, start a new chat. The model rule applies to new sessions.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. The first item is always "read the Principles section". The rest are the matched playbook's steps copied in, the Feature playbook for this prompt. If `poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. `poteto-mode` is sticky. It stays on for the conversation until you opt out by saying so.

Next: [Route work through `poteto-mode`](./02-poteto-mode.md).
