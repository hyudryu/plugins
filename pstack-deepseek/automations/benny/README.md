# benny

benny gives you two dsh automations for slack issue reports. one triages each report. the other reproduces confirmed bugs and may prepare a small draft fix.

the files in this directory are dormant setup and automation sources. they do not appear as loadable skills.

## set it up

1. point dsh at [`FOR_AGENTS.md`](./FOR_AGENTS.md) and name the target repository.
2. let setup merge this whole directory into the target at `<target-repo>/automations/benny/`. it must preserve destination-only files and review conflicts instead of overwriting local edits.
3. let setup make the pstack skills visible to the target repository's dsh (project `.dsh/skills/`) for shared dependencies. keep unrelated skill sources and config as they are.
4. keep user-owned configuration outside the copied pack, for example in `<target-repo>/benny/`. adapt [`configuration.example.yaml`](./templates/configuration.example.yaml) and [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md).
5. commit the pstack skill-source configuration, `automations/benny/`, and any secret-free configuration before enabling either automation.
6. review each new automation draft or update existing automations in their editors. then send a harmless test report and verify every source-channel post stays in the original thread.
