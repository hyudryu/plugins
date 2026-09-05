# DeepSeek Harness port notes

This folder is pstack optimized for **DeepSeek Harness (`dsh`)**, adapted from
the ZCode edition. The engineering substance (principles, playbooks, rubrics)
is identical; only the harness vocabulary and a few mechanisms differ. This
doc is the mapping a future maintainer needs.

## Install

dsh discovers top-level `<name>/SKILL.md` bundles — no nested recursive scan —
at these roots, highest priority first:

1. `<projectRoot>/.dsh/skills/` (project scope)
2. `<projectRoot>/.agents/skills/`
3. `Config.customSkillDirs`
4. `<dshHome>/skills`
5. `~/.agents/skills/` (user scope)

Copy this folder's `skills/*` directories into `.dsh/skills/` (repo) or
`~/.agents/skills/` (personal) and restart `dsh`. Verify with `dsh`'s skill
catalog (the model-facing `skill` tool) or `/setup-pstack`.

`agents/*.md` are **not** registered anywhere — dsh has no plugin-agent
registry. They are prompt templates a parent inlines as the head of a
`subagent` call (poteto-mode and no-comments instruct exactly that).

## Vocabulary mapping applied (ZCode → dsh)

| ZCode edition | dsh edition |
|---|---|
| `Agent` tool, `subagent_type` | `subagent` tool (per-provider); children inherit the session model |
| Read-only workers via `Explore` agent type | prompt-discipline read-only; prefer a deployment-configured restricted `subagent` instance (tool allow/deny) when one exists |
| Named plugin agents `pstack:poteto-agent` / `pstack:comment-sicko` | inline templates from `agents/` |
| `AskUserQuestion` | `ask_user_question` |
| `TodoWrite` | `todo_write` |
| `CronCreate` automations (durable, cron) | `schedule_create` / `schedule_list` / `schedule_delete` — session-local reminders, five-minute minimum, delivered only while the session is open; long-running automation (benny) uses a dedicated long-lived session or an external scheduler |
| `ReadSessionContext` (`sess_*` ids) | no equivalent — session-history skills lean fully on the decision trail (`decisions.tsv`) plus `git`/`gh`, and on a session the user reopens |
| `WebFetch` / `WebSearch` | `web_fetch` / `web_search` |
| `browser-use` / `computer-use` skills | dsh `browser` tool (Playwright) for web UIs; `bash` for CLIs/TUIs |
| ZCode plugin manifest `.zcode-plugin/plugin.json` | none — skill bundles are the distribution |

## dsh-specific optimizations

- `disable-model-invocation: true` on `poteto-mode`, `automate-me`, and `swarm`
  (dsh reads this frontmatter key natively; the ZCode edition had to rely on
  description scoping alone). `user-invocable` stays default-true, so
  `/poteto-mode` still works.
- `poteto-mode` gained a trigger: bulk mechanical work (sweeps, mass edits,
  repetitive checks) prefers `run_code` — dsh's PTC mode scripts tool
  dispatches once instead of emitting dozens of model-turned calls. This is
  the native shape of the build-the-lever principle.
- Generated project verification skills target `<projectRoot>/.dsh/skills/verify-<app>/`,
  dsh's highest-priority project root.
- Skill names already match dsh's kebab-case pattern; skill bundles may carry
  `references/`, `playbooks/`, `scripts/` subdirs freely (resources below a
  bundle are not catalog entries).

## Known gaps vs the ZCode edition

- No durable cross-restart scheduling: overnight "wake me when CI lands" runs
  need the session to stay open, or an external scheduler starting dsh runs.
- No model-facing session search: `recall` / `reflect` / `eval` grade and
  rebuild purely from decision trails, git, and reopened sessions.
- No per-child model selection discipline: dsh *can* select child models
  (`list_subagent_models`), but pstack deliberately keeps one model and gets
  independence from separate contexts with identical prompts.
