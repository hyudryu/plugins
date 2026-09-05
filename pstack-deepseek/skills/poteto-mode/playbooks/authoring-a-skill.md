### Authoring or modifying a skill

**You own the skill's voice.** Agent-facing prose has a higher bar than human prose; unhelpful sentences become instructions.

1. Author the SKILL.md per dsh's skill format. Frontmatter has `name` matching `^[a-z0-9]+(?:-[a-z0-9]+)*$` and a `description` as one YAML scalar starting with "Use when". Two invocation keys are read verbatim: `disable-model-invocation` (keep the skill out of the model-facing catalog) and `user-invocable`. Heavy, opinionated skills (`*-mode`, `swarm`) set `disable-model-invocation: true`. The catalog shows only `name` and `description`, so the description carries the trigger. Support the body with sibling subdirectories as needed: `references/` for lookups, `templates/` for reusable scaffolds, `scripts/` for tooling.
2. Validate the skill: the frontmatter parses between the first two `---` lines, `name` matches the kebab-case pattern, the description's trigger sits in the opening sentence, every referenced file and support directory exists, and cross-skill links resolve. dsh discovers only top-level `<name>/SKILL.md` bundles (no nested recursive scan), so never nest a skill inside another skill's directory.
3. Test cases if structural; skip if subjective.
4. Run **Opening a PR**.

When in doubt, delete; prose earns its keep by changing a decision. Tell it to do the thing and skip the reason. Explain only when the rule is confusing without one. Match tone to scope. Point at structural sources (types, READMEs, config); hardcoded details go stale (the **encode-lessons-in-structure** principle skill). Delegate to other skills by path; don't restate. A workflow you keep hitting but isn't captured → propose a new skill.

**Reply:** summary of the skill, key design decisions, validation notes.
