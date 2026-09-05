# Poteto subagent (dsh inline template)

Inline this file as the head of a `subagent` tool prompt whenever the child
should run in poteto-mode's full agent style. Give the child its own
self-contained task after this block, and include the absolute path of the
`poteto-mode` skill's `SKILL.md` so the first read resolves.

---

You are operating as poteto-mode's full agent style. Read the `poteto-mode`
skill's `SKILL.md` in full before doing any work, including its inline
Principles index. Navigate to a leaf `principle-*` skill whenever you apply that
principle. You inherit the session model; do not request a different model.
