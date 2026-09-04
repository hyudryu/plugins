---
name: reflect
description: "Use when the user says reflect, to spawn three parallel review subagents over the current conversation, surface learnings, and route each to a concrete edit on an existing skill."
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

## When to invoke

- The user said "reflect" or "/reflect".
- A complex task (5+ tool calls) just landed cleanly and the recipe is worth keeping.
- The agent hit dead ends, found the working path, and the path generalizes.
- The user corrected the agent's approach mid-task.
- A non-trivial workflow emerged that isn't captured anywhere.

Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## Process

### 1. Build the session record

There is no transcript file to glob in ZCode, so assemble the evidence before fanning out:

1. The current conversation itself is the primary record. Write a tight digest of the session: the user's goals, the approaches tried, dead ends, corrections, tools and skills invoked, commands and flags that mattered, and what shipped.
2. If this run kept a decision trail (the **show-me-your-work** skill), read the `decisions.tsv` and fold it in. It is evidence of what actually happened, not what was summarized.
3. If the user names a prior session (`sess_*` id), pull focused context from it with `ReadSessionContext`. Never pull from unrelated project sessions; that crosses workspace boundaries and reads private chats from unrelated work.

Pass the digest (plus trail extracts) to the reviewers in place of a transcript path. Every template below accepts either.

### 2. Spawn three reviewers in parallel

One message, three `Agent` calls (`subagent_type: "general-purpose"`), all on the session model (never specify per-reviewer models). Reviewers may need MCP tools for context lookups (tickets, chat threads, observability traces the session referenced); `general-purpose` keeps them available. The prompt forbids file writes; the parent applies edits.

| Lens | Prompt template |
|---|---|
| Judgment | `references/judgment-reviewer.md` |
| Tooling | `references/tooling-reviewer.md` |
| Divergent | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the digest where marked. Reviewers return findings in their `Agent` result.

### 3. Synthesize

One more `Agent` call (`general-purpose`), on the session model. The synthesizer's quality check includes spot-verifying citations, which can require MCP tools; `general-purpose` keeps them available. Use `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. The synthesizer returns a structured Accepted / Rejected / Backlog list.

### 4. Structural enforcement check

Sanity-check the synthesizer's Accepted list. For any item that would be enforced more reliably by a lint rule, script, metadata flag, or runtime check, move it from Accepted to Backlog. The synthesizer already applies this criterion; this is a final pass before edits land. See the **encode-lessons-in-structure** principle skill.

### 5. Apply

Before applying any Accepted edit, present the synthesizer's full Accepted/Rejected/Backlog output to the user and wait for explicit approval. The user picks which subset to apply and may redirect routings. Skill changes affect every future agent in the org; do not auto-apply.

Backlog items file to whatever devex / backlog tracker your team uses automatically. Those are tracker submissions, not skill edits. Only the Accepted list waits for approval.

For each approved Accepted item, follow the Routing field exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): parent does directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): apply via the `skill-creator` skill and run its draft / test / iterate loop.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): run a description-optimization pass on that skill's `description` (the `skill-creator` skill covers this).
- `new skill: <kebab-name>`: create the new skill with the `skill-creator` skill and do not invent the shape ad hoc.

ZCode reads only `name` and `description` from a SKILL.md's frontmatter; keep names kebab-case and descriptions scoped to when the skill should fire. Check every touched skill still parses (frontmatter between the first two `---` lines) before declaring done.

### 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
