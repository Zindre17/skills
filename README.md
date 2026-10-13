# skills

Personal Claude Code skills.

| Skill | What it does |
| --- | --- |
| `/refine` | Interviews you about a plan, design or decision before it gets built, in rounds of `AskUserQuestion` forms, writing glossary entries, ADRs and a transcript as decisions land. |
| `/linus` | Implements a feature three times (gut, rethink, final) and pauses so you can review the final design before it is coded. |
| `/linus-auto` | Same as `/linus`, but unattended: no pause before the final attempt. |

`/linus` and `/linus-auto` only run when you invoke them. `/refine` can also trigger on phrases like "grill me" or "stress-test this".

## Install

This repository is a Claude Code plugin and its own marketplace. In a terminal session:

```
/plugin install myskills --marketplace Zindre17/skills
```

Answer `y` to add the marketplace, then pick a scope. Installed this way, the skills are namespaced as `myskills:refine`, `myskills:linus` and `myskills:linus-auto`.

## Dependencies

`/refine` uses the `domain-modeling` skill from [Matt Pocock's skills](https://github.com/mattpocock/skills). The plugin declares `mattpocock-skills@claude-plugins-official` as a dependency, so installing it also installs that plugin, provided the `claude-plugins-official` marketplace is added. If the skill is still missing, `/refine` stops at the start and asks you to install it.

## Development

Run a session against the working copy, so edits apply without a reinstall:

```bash
claude --plugin-dir ~/code/myskills
```

An installed copy only updates when `version` in `.claude-plugin/plugin.json` is bumped.
