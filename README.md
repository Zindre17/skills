# skills

Personal Claude Code skills.

| Skill | What it does |
| --- | --- |
| `/refine` | Interviews you about a plan, design or decision before it gets built, in rounds of `AskUserQuestion` forms, writing glossary entries, ADRs and a transcript as decisions land. |
| `/linus` | Implements a feature three times (gut, rethink, final) and pauses so you can review the final design before it is coded. |
| `/linus-auto` | Same as `/linus`, but unattended: no pause before the final attempt. |

`/linus` and `/linus-auto` only run when you invoke them. `/refine` can also trigger on phrases like "grill me" or "stress-test this".

## Install

Symlink each skill into your personal skills directory:

```bash
git clone git@github.com:Zindre17/skills.git ~/code/myskills
for s in ~/code/myskills/skills/*/; do ln -s "${s%/}" ~/.claude/skills/; done
```

## Dependencies

`/refine` uses the `domain-modeling` skill from [Matt Pocock's skills](https://github.com/mattpocock/skills) plugin, and stops at the start if it is missing. Install it with:

```
/plugin install mattpocock-skills@claude-plugins-official
```
