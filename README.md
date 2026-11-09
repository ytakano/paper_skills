# paper_skills

Claude Code skills for writing and checking research papers.

| Skill | Purpose |
| --- | --- |
| [`ai-manuscript-review`](skills/ai-manuscript-review/SKILL.md) | Reviews manuscripts drafted or edited with generative AI for recurring problems: terms used before definition, inconsistent figure series and colors, overused dashes and semicolons, unnecessary negation or contrast, overly long paragraphs, excessive lists, bold or italic emphasis used for highlighting, and inconsistencies between newly edited parts and earlier parts. Not a full peer review. |
| [`reference-audit`](skills/reference-audit/SKILL.md) | Verifies that each reference exists and that its bibliographic metadata is correct, then produces a Markdown report with links for human checking. |

## Installation

Link or copy each skill directory into a Claude Code skills directory.

```sh
# For all projects (personal skills)
mkdir -p ~/.claude/skills
ln -s "$PWD/skills/ai-manuscript-review" ~/.claude/skills/
ln -s "$PWD/skills/reference-audit" ~/.claude/skills/

# Or for a single project
mkdir -p /path/to/project/.claude/skills
ln -s "$PWD/skills/"* /path/to/project/.claude/skills/
```

Invoke a skill explicitly with `/ai-manuscript-review` or `/reference-audit`, or let Claude select it from the request.
