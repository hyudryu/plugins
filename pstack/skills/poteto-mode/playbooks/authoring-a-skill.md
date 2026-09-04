### Authoring or modifying a skill

**You own the skill's voice.** Agent-facing prose has a higher bar than human prose; unhelpful sentences become instructions.

1. Author the SKILL.md per the ZCode conventions (the `skill-creator` skill is the peer reference). Frontmatter has `name` in lowercase-hyphen form and a `description` as one YAML scalar starting with "Use when"; the description is the skill's only trigger, so scope it tightly. ZCode ignores other frontmatter keys — keep the file minimal. Support the body with sibling subdirectories as needed: `references/` for lookups, `templates/` for reusable scaffolds, `scripts/` for tooling.
2. Validate the skill: the frontmatter parses between the first two `---` lines, `name` is lowercase-hyphen, the description's trigger sits in the opening sentence, every referenced file and support directory exists, and cross-skill links resolve.
3. Test cases if structural; skip if subjective.
4. Run **Opening a PR**.

When in doubt, delete; prose earns its keep by changing a decision. Tell it to do the thing and skip the reason. Explain only when the rule is confusing without one. Match tone to scope. Point at structural sources (types, READMEs, config); hardcoded details go stale (the **encode-lessons-in-structure** principle skill). Delegate to other skills by path; don't restate. A workflow you keep hitting but isn't captured → propose a new skill.

**Reply:** summary of the skill, key design decisions, validation notes.
