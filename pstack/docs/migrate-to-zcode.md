# pstack: Cursor/Hermes → ZCode Migration Plan

**Target:** ZCode (the harness this machine already runs). Skills load from a plugin's
`skills/` directory or user-level `~/.agents/skills/<name>/SKILL.md`.

**State of the tree:** a previous Cursor→Hermes pass is partially applied
(see `docs/migrate-cursor-to-hermes.md`). Frontmatter is normalized, agent files are
rewritten as `delegate_task` templates, and README/automations are half-converted.
This plan therefore has two workstreams:

- **A. Hermes → ZCode** — swap Hermes mechanisms (`delegate_task`, `~/.hermes`,
  Hermes cron, Hermes session search, `skill_manage`) for ZCode mechanisms.
- **B. Cursor → neutral** — finish removing residual Cursor-isms the Hermes pass
  skipped (Bugbot, `/loop`, model slugs, `.cursor` paths, guide chapters).

All ZCode formats below were verified against plugins installed on this machine
(`skill-creator`, `document-skills`, `aws-dev-toolkit` caches).

---

## 1. ZCode platform facts (the mapping table)

| Need | ZCode mechanism |
|---|---|
| Plugin manifest | `.zcode-plugin/plugin.json`: `name`, `version`, `description`, `author`, `license`, `keywords`, `skills: "skills"`. Agents are auto-discovered from `agents/*.md` (installed plugins do not declare an agents key). Optional `mcpServers`. |
| Skill file | `SKILL.md` with frontmatter `name` (kebab-case) + `description` only. Everything else (version, author, metadata) is ignored — keep files minimal. |
| Skill invocation | User types `/<name>` → Skill tool; model may also invoke via Skill tool when the description matches. Skills appear as `pstack:<name>` with a bare-name alias. |
| Supporting dirs | `references/`, `playbooks/`, `templates/`, `scripts/` all fine — no Hermes-style subdir whitelist. The skills list shows each SKILL.md's absolute path, so relative references resolve. |
| Named subagents | Plugin agent = `agents/<name>.md` with frontmatter `name`, `description`, optional `tools: [..]`, `color`; body is the system prompt. Invoked via `subagent_type: "pstack:poteto-agent"` (bare `poteto-agent` also resolves). |
| Delegation | `Agent` tool: `subagent_type`, `description`, `prompt`, `run_in_background: true`. **No per-call `model` parameter** — every subagent runs the session model. |
| Read-only subagent | `Explore` agent type (read-only toolset). `general-purpose` always has all tools; there is no per-call MCP stripping, so "read-only" is prompt discipline unless you use Explore. |
| Todo list | `TodoWrite` / `TodoRead` |
| Ask the user | `AskUserQuestion` (structured, multi-choice, optional previews) |
| Schedules / wake-ups | `CronCreate` / `CronUpdate` / `CronList` / `CronDelete` (cron or delayMinutes, persists across restarts) |
| Session history | `ReadSessionContext` — requires a `sess_*` id and a focused query. **No directory-style session search.** This is the one real gap; see §5. |
| MCP tools | Appear directly in the model's context as `mcp__server__tool`. No runtime "list MCP servers" tool — phrase as "the MCP tools available in your context". |
| Skill authoring | The `skill-creator:skill-creator` plugin skill (create/edit/iterate SKILL.md) |
| Plan mode | `EnterPlanMode` / `ExitPlanMode` (exists; pstack doesn't need it) |
| Browser / UI driving | `browser-use` plugin (control-browser), `computer-use` plugin, `android-emulator` plugin |
| Web | `WebSearch`, `WebFetch`, `web_reader` MCP |

---

## 2. Global mechanical changes (apply to every file)

1. **Frontmatter.** Keep only `name` + `description`. Specifically:
   - `skills/poteto-mode/SKILL.md`: `name: Poteto Mode` → `name: poteto-mode`; delete
     `disable-model-invocation`, `mode`, `icon`, `color`, `reminder` (Cursor UI keys).
   - All other skills: delete the `version`, `author`, `license`, and
     `metadata.hermes: {tags, related_skills}` blocks (ZCode ignores them; they're noise).
     If you want to keep related-skills info, move it into the body under a
     `## Related skills` heading.
2. **`delegate_task` → `Agent`.** Every occurrence ("Spawn via `delegate_task`",
   "N `delegate_task` calls in parallel") becomes: "Spawn N `Agent` calls in one step
   with `subagent_type: "general-purpose"`, a one-line `description`, and a
   self-contained `prompt`; set `run_in_background: true` for long workers."
   Files: poteto-mode, swarm, arena, interrogate, how, why, reflect, recall,
   no-comments, automate-me, architect, and playbooks autopilot-full/-stack,
   orchestrate, multi-phase-plan, feature.
3. **"Single Hermes model" phrasing.** Replace with ZCode truth: "the session model —
   the Agent tool takes no model parameter, so every subagent inherits it; never ask
   for a per-worker model." Files: swarm, arena, interrogate, why, reflect, recall,
   setup-pstack, poteto-mode §Subagents, automate-me, benny setup.
4. **Model slugs.** Delete `grok-4.6-fast-xhigh`, `claude-fable-5-thinking-max`,
   `gpt-5.6-sol-max`, `fable / sol / grok / opus 5` wherever they survive
   (poteto-mode §Subagents defaults; guide/01-setup). There is no model picker in ZCode.
5. **`AskQuestion` → `AskUserQuestion`.** Files: poteto-mode (Non-negotiables),
   automate-me, architect (checkpoint step), create-verification-skill.
6. **Paths.** `~/.hermes/skills/...` → `~/.agents/skills/<name>/SKILL.md` (user-level
   skills on this machine) or in-repo `skills/...`; `~/.hermes/config.yaml` → drop
   (no such config in ZCode); `~/.hermes/benny/` → `~/.zcode/benny/`.
   Files: automate-me, setup-pstack, reflect references (×4), setup-benny.
7. **"Hermes skill authoring"** → "the `skill-creator` skill" (installed ZCode plugin).
   Files: automate-me, reflect (synthesizer routing), authoring-a-skill playbook.
8. **"Hermes automations/cron"** → "`CronCreate` automations". Files: benny pack,
   README, poteto-mode (the /loop replacement language).
9. **"Hermes session history / session search"** → see §5 (per-skill redesign).
10. **MCP enumeration.** "Enumerate available Hermes MCP servers … use the
    available-tools map" → "Read the MCP tools available in your context (they are
    prefixed `mcp__`); skip a category with no matching tool and say so."
    Files: why (Step 3), reflect references, interrogate, poteto-mode Autonomy.
11. **Read-only semantics.** "agent mode (readonly strips MCP)" (poteto-mode §Subagents)
    → "use the `Explore` agent type for read-only workers; `general-purpose` always has
    full tools, so read-only contracts are prompt discipline." Comment Sicko and the
    reflect reviewers keep their no-write instruction text as the enforcement.
12. **Plugin manifest.** Replace `.cursor-plugin/plugin.json` with
    `.zcode-plugin/plugin.json`:

    ```json
    {
      "name": "pstack",
      "version": "1.0.0",
      "description": "rigorous agent workflows you can parallelize with confidence.",
      "author": { "name": "Lauren Tan" },
      "license": "MIT",
      "keywords": ["workflow", "principles", "review", "planning"],
      "skills": "skills"
    }
    ```

    Drop Cursor-marketplace keys (`displayName`, `category`, `tags`, `homepage`
    pointing at cursor/plugins). `agents/` is discovered automatically; the
    `"agents": "./agents/"` key can be dropped.
13. **README.** Install section: `/add-plugin pstack` → install as a ZCode plugin
    (drop the repo into a plugin source / `~/.agents/skills` for user-level). Rewrite
    the remaining Hermes paragraphs (lines ~11, 17, 23, 28, 91, 186–190, 230–233,
    243–245, 251) to name ZCode; `/poteto-mode` slash references stay valid because
    ZCode skills are user-invocable as `/<name>`.
14. **Docs.** `docs/guide/01-setup.md`, `02-poteto-mode.md`, `07-overnight.md` (the
    `/loop until done` line), and `docs/guide/README.md` mention Cursor chat, `/loop`,
    and model picking — rewrite for ZCode. Delete or rewrite
    `docs/migrate-cursor-to-hermes.md` once this plan lands (it documents a target
    you're no longer aiming at).
15. **Scripts.** `skills/poteto-mode/scripts/package.json` name
    `@cursor-skill/poteto-mode-tools` → `@pstack/poteto-mode-tools`.
    `watch-pr/github.ts:339–353` special-cases GitHub author `cursor` and
    `CURSOR_AUTOMATION_ID` markers to recognize Cursor background-agent commits —
    generalize to a configurable bot-author / marker list (a ZCode automation can
    stamp its own marker, e.g. `PSTACK_AUTOMATION_ID`). The rest of the script's
    "cursor" hits are GraphQL pagination cursors — leave alone. Scripts run under
    Bun via plain `Bash` in ZCode: no change needed otherwise.

---

## 3. Router + agents (the core wiring)

### `skills/poteto-mode/SKILL.md` (dispatcher — highest priority)

| Line | Cursor/Hermes-specific | Fix for ZCode |
|---|---|---|
| frontmatter | `name: Poteto Mode`, `disable-model-invocation`, `mode: true`, `icon`, `color`, `reminder` | `name: poteto-mode`; drop the rest. There is no sticky-mode flag in ZCode: encode "once entered, keep applying when a playbook matches; opt out on request" as body text, and optionally add one line to the project's AGENTS.md ("for any non-trivial task, invoke the poteto-mode skill first") to approximate the always-on behavior. |
| Non-negotiables ¶1 | "Start every multi-step task with a todolist" | Keep, name the tool: `TodoWrite`. Works as-is otherwise. |
| "About to `AskQuestion`" | Cursor ask tool | `AskUserQuestion` (same fork-classification rule applies unchanged) |
| "create-skill skill (Cursor's built-in…)" | Cursor built-in | the `skill-creator` skill |
| "`deslop` from the `cursor-team-kit` plugin" | external Cursor plugin | absorb into `unslop` (one line in unslop: "also strip commit-message and diff slop") — recommended by the Hermes plan too |
| "`cursor-team-kit` publishes `control-cli` / `control-ui`" | external Cursor plugin | ZCode has native drivers: `browser-use` (control-browser), `computer-use`, `android-emulator`, plus plain `Bash` for CLIs/TUIs. Rewrite the bullet to name those. |
| "not Cursor's built-in babysit skill" | Cursor built-in | drop the comparison; keep "any PR-status request → the Babysit playbook" |
| "Bugbot or the agentic security review commented" | Cursor's Bugbot | generalize to "a review bot (Bugbot, CodeRabbit, GitHub review bots) or the agentic security review commented" — see `bugbot-triage` below |
| `/loop` references ("`/loop until X`") | Cursor command | "a recurring `CronCreate` automation or a background `Agent` re-checking the finish predicate" — rephrase each occurrence |
| §Subagents | `subagent_type: "poteto-agent"`, `Task` calls, `run_in_background: true`, "agent mode (readonly strips MCP)", per-role model defaults, `grok-4.6-*` / `claude-fable-*` / `gpt-5.6-*` slugs, `/setup-pstack` rule overrides | `subagent_type: "pstack:poteto-agent"`; `Agent` calls (parameters above); read-only via Explore or prompt discipline; delete all model-tiering prose — one session model, tiering happens by *which agent type + prompt*, not by model; setup-pstack reference becomes "see the setup-pstack skill" |
| §Playbooks | relative `playbooks/*.md` pointers | fine as-is in ZCode (SKILL.md path is exposed); optionally note "resolve relative to this file" |

### `agents/poteto-agent.md` and `agents/comment-sicko.md`

Currently rewritten as Hermes `delegate_task` prompt templates. ZCode **does** have
named plugin agents (that's how `judge` ships), so convert both to real agent files:

```markdown
---
name: poteto-agent
description: Runs a task in poteto-mode's full style. Spawn it for any delegate inside a poteto-mode playbook step; it reads the poteto-mode skill (including the inline principles index) before doing any work.
---
<keep the existing body — it's already good; replace "delegate_task goal/context"
wording with "you receive the task in the dispatch prompt">
```

- `comment-sicko.md`: add `tools: [Read, Bash, Grep, Glob]` to frontmatter for a
  hard read-only guarantee (ZCode supports per-agent tool lists), keep the
  "must not write application code" contract in the body.
- Invocation updates: `subagent_type: "pstack:poteto-agent"`,
  `subagent_type: "pstack:comment-sicko"` — update in poteto-mode, no-comments,
  and README. Note the "use poteto-agent, not general-purpose, or it skips the
  principles read" warning stays and now maps to ZCode mechanics exactly.
- Model sentence in both files ("You inherit the single Hermes model") → "You run
  the session model; do not request a different model."

---

## 4. Delegation / review skills

| Skill | What's harness-specific today | Fix |
|---|---|---|
| `swarm` | `delegate_task`, "single Hermes model", "no environment selection concept" | `Agent` calls, ZCode model phrasing, delete the Hermes environment sentence entirely (ZCode has none either — no need to mention it) |
| `arena` | `delegate_task`, single-model language, judge step model-family talk | `Agent` calls; candidates + judge all `general-purpose` (or Explore for the read-only judge — nice fit: the judge only reads and scores). Multi-model diversity is unavailable in ZCode; the independence now comes from separate contexts. If you later want true model diversity, it would have to go through an external model CLI or the SparkDeck MCP — out of scope for v1. |
| `interrogate` | "Launch all reviewers in a single step as parallel `delegate_task` calls", Hermes model | `Agent` calls in one message; same fix. Reviewer independence note: separate contexts, identical prompts — that framing is already in the skill; keep. |
| `how` | fans out 2–4 read-only explorers | `Agent` with `subagent_type: "Explore"` per explorer — Explore is a perfect match (read-only, returns conclusions). |
| `why` | "enumerate available Hermes MCP servers … available-tools map", `delegate_task`, single-model | "Read the `mcp__*` tools in your context"; `Agent` calls; keep the per-category investigator fan-out and `references/sources/*` playbooks verbatim (they're portable). Line 66 "cursor location" → "editor cursor position" is prose fluff — drop the word. |
| `reflect` | "Hermes session history", transcript glob layout (`<id>.jsonl`, `subagents/`), `skill_view`/`delegate_task` names, `~/.hermes/skills` paths in all 4 reference files | See §5 — transcript source is the redesign. Reviewer/synthesizer spawning → `Agent` calls; `skill_view` → "Read of any SKILL.md"; `skill_manage` → "edit the SKILL.md file directly (or via the skill-creator skill)". Update the 4 `references/*` files' evidence-scan bullets to match. |
| `no-comments` | `delegate_task` + inline Comment Sicko template | Two options: (a) spawn `subagent_type: "pstack:comment-sicko"` directly (now that Comment Sicko is a real ZCode agent) — simplest; (b) keep template-inlining. Recommend (a). |
| `architect` | `/how`/`/arena` routing (fine), `AskQuestion` checkpoint | `AskUserQuestion`; keep the explicit-opt-in checkpoint semantics |
| `recall` | Hermes session search, `delegate_task`, chat-UUID citations | See §5 |
| `automate-me` | "Hermes skill authoring", `~/.hermes/skills` scan, `AskQuestion`, session-history mining, "Hermes has no disable-model-invocation" rationale | skill-creator; `~/.agents/skills`; `AskUserQuestion`; §5 for mining; keep the description-scoping advice (still correct for ZCode — there is no invocation-disable flag here either) |
| `setup-pstack` | Entire skill is about "the configured Hermes model" (`~/.hermes/config.yaml`, `hermes config`) | Rewrite around ZCode: "ZCode runs one model per session; the Agent tool has no model parameter, so subagents inherit it. Nothing to configure — this skill just verifies the wiring": confirm no stale per-role config files linger, and offer to generate the verification skill. It shrinks to ~⅓ of its current size. |
| `show-me-your-work` | "reads this run's transcript under `agent-transcripts/`" to verify | See §5 — and note this skill becomes *more* important in ZCode (see §5 recommendation c) |
| `create-verification-skill` / `maintain-verification-skill` | writes project-local `.cursor/skills/verify-<app>/` (per Hermes plan §6; verify current text) | target a project-local ZCode skill location (`.zcode/skills/verify-<app>/SKILL.md` in-repo, or `~/.agents/skills/`) — pick one convention and document it; the feature-map/verify/drive content is framework-agnostic. UI-driving steps → browser-use/computer-use tools. |
| `figure-it-out`, `blast-radius`, `teach`, `tdd`, `bro`, `unslop`, `technical-writing`, `typescript-best-practices`, all 21 `principle-*` | only frontmatter + incidental `delegate_task`/slash mentions | mechanical pass only (§2 items 1, 2, 6) |

---

## 5. The one real gap: session history (recall, reflect, eval, session-pickup, pause-safely, worktree-cleanup, automate-me)

Cursor had transcript directories; Hermes had session search. ZCode exposes
`ReadSessionContext` (needs a `sess_*` id you must already know) and nothing else
agent-facing. Options, in recommendation order:

- **(a) Recommended — make pstack's own trail the primary record.** Lean on
  `show-me-your-work`'s `decisions.tsv`: recall/eval/session-pickup read the
  project's committed decision trail first, then `git`/`gh` state, and use
  `ReadSessionContext` only when the user supplies or the trail references a
  session id. This is more robust than any transcript scraping and matches
  ZCode's model.
- **(b) Mine on-disk session files via Bash** (`~/.zcode/...` JSONL). Possible
  today on this machine but undocumented/fragile; if used, isolate the path
  logic in one helper script under `scripts/` so it can be fixed in one place.
- **(c) Degrade gracefully.** Recall falls back to "live state + shared record
  (the why skill)"; reflect grades from the chat context it *does* have plus the
  decision trail; automate-me mines the trail instead of transcripts.

Per-skill rewrites:

- `recall` — replace "Transcripts live in Hermes session history" with the (a)
  hierarchy: decisions trail → git/gh → ReadSessionContext when a session id is
  known. Keep the two-record framing and the why-skill sweep verbatim.
- `reflect` — "Locate the active transcript" step becomes "work from the current
  conversation + the run's decisions.tsv"; the transcript-layout glob block is
  deleted; reviewer references keep their untrusted-data framing.
- `eval` (playbook) — grades chain-following from "files each candidate actually
  read"; with subagents returning reports instead of transcripts, have each
  candidate append its own files-read list to its report (one line to add), and
  grade from that.
- `session-pickup` / `pause-safely` — replace "transcript, cloud-agent URL" with
  "pushed branch + decisions.tsv + (optional) session id"; pause writes the trail
  and a resume brief. Both playbooks' contracts survive intact.
- `worktree-cleanup` — replace the transcript-scan step with (a) or `git`-based
  staleness (branch merged? worktree mtime?); drop the
  `~/Library/Application Support/Cursor` cache cleanup.
- `automate-me` — mining source becomes (a)/(c).

---

## 6. Autonomy & long-run playbooks

- **`/loop` → ZCode scheduling.** Every "`/loop until X`" (poteto-mode, autonomous-run,
  guide/07) becomes: a `CronCreate` automation (recurring cron or delayMinutes) whose
  prompt re-states the finish predicate, or a background `Agent` with periodic
  re-checks. `autonomous-run.md` should name `CronCreate` explicitly — that is the
  ZCode-native way to "keep going while I sleep", and it survives app restarts.
- **babysit** — modes (`drive`/`background`/`threads-only`/`check`) are pure logic:
  portable. Bugbot mentions → "review-bot comments" (§7). Graphite `gt`/`gh` are
  plain CLIs: **keep**.
- **shipping** — Graphite merge-when-ready machinery: keep (CLI-based). No Cursor
  dependency found beyond Bugbot-adjacent language.
- **autopilot-full / autopilot-stack / orchestrate / multi-phase-plan** —
  `delegate_task` → `Agent`; Bugbot → review-bot; everything else (owner loop,
  verification gates, Graphite stack mechanics) is harness-neutral and strong.
- **visual-parity / hillclimb / perf / forensics playbooks** — no harness coupling
  found beyond §2 mechanicals; visual-parity's screenshot steps map to
  browser-use/computer-use tools.

---

## 7. Bugbot (`references/bugbot-triage.md` + all mentions)

Bugbot is Cursor's PR review bot. The triage rules in this reference are excellent
and bot-agnostic — keep the content, rename the framing:

- Rename file to `references/review-bot-triage.md` (update the 6 referencing files:
  poteto-mode, babysit, autopilot-full, autopilot-stack, multi-phase-plan, opening-a-pr).
- Rewrite the intro: "a review bot commented (Cursor Bugbot, CodeRabbit, GitHub
  review bots, or the agentic security review)". Keep every classification rule —
  they were learned from Bugbot passes and apply to any reviewer bot.
- The babysit "Bugbot pass count" stamping becomes "review-pass count".

---

## 8. benny automation pack (`automations/benny/`)

Already half-converted to Hermes. Finish with ZCode primitives:

- "point Hermes at FOR_AGENTS.md" → "point ZCode at FOR_AGENTS.md" (README, setup-benny).
- "two live Hermes automations" / "Hermes automation tooling" → two `CronCreate`
  automations whose prompts name the exact operational files
  (`triage-automation-prompt.md` / `reproduce-automation-prompt.md` templates get a
  one-paragraph update: "register via CronCreate with this prompt; the automation
  runs in its own session, so every prompt must be self-contained" — which the
  templates already are).
- "Hermes skill source" enabling → "install pstack as a plugin / skills source for
  the target repo" phrasing.
- `~/.hermes/benny/configuration.yaml` → `~/.zcode/benny/configuration.yaml`.
- "Prefer Hermes' Slack integration" → "use the Slack MCP tools available in the
  automation's context, or `BENNY_SLACK_BOT_TOKEN` via the Slack Web API" (the
  token fallback already exists and is fine).
- "single configured Hermes model" → ZCode single-model phrasing (§2.3).
- UI-control dependency (was control-ui/control-cli): ZCode-native answer is
  browser-use/computer-use. The control-adapter reference abstracts this — update
  its examples to those tool names.

---

## 9. Execution order

1. Manifest + frontmatter normalization (§2.1, §2.12) — mechanical, unblocks everything.
2. `poteto-mode` + both `agents/` files (§3) — the router every other skill flows through.
3. Delegation skills (§4): swarm, arena, interrogate, how, why, no-comments, architect, reflect-spawning.
4. `setup-pstack`, `automate-me`, `recall` + the §5 session-history redesign.
5. Playbooks pass (§6): delegate_task/loop/Bugbot sweeps across `playbooks/` + `bugbot-triage` rename.
6. benny pack (§8).
7. README, docs/guide, delete/replace `migrate-cursor-to-hermes.md` (§2.13–14).
8. Scripts touch-ups (§2.15).
9. Validation: load the plugin in ZCode, confirm all skills appear in the skills list
   with kebab-case names, run `poteto-mode` end-to-end on a real task (one playbook
   that spawns poteto-agent, one that runs no-comments), and exercise one
   `CronCreate`-based automation dry-run.

## 10. Risks / gotchas specific to ZCode

- **Description scoping is the only trigger control.** No `disable-model-invocation`.
  poteto-mode, swarm, automate-me descriptions must stay tightly scoped ("Use when the
  user says…") or they will auto-fire on generic requests.
- **No sticky mode.** The AGENTS.md one-liner (§3) is the closest approximation; it
  also only applies to sessions that read that AGENTS.md.
- **Subagent ≠ same conversation.** ZCode subagents start fresh; every `Agent` prompt
  must be self-contained (the skills already demand this — keep the emphasis).
- **Read-only is a prompt contract** unless you use Explore or a `tools:`-restricted
  agent file. Comment Sicko gets `tools:`; the why-investigators keep posture-only
  read-only (they need MCP access, which Explore strips… actually Explore retains
  WebFetch/Bash; if an investigator needs MCP tools, use general-purpose + prompt).
- **Session-history features weaken** without the §5 redesign — do not port them
  expecting transcript parity.
- **Model diversity is gone.** Arena/interrogate independence = separate contexts,
  identical prompts. Fine in practice; say so honestly in the skill text instead of
  implying model variety.
