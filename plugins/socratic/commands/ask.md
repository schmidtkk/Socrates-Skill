---
description: Grill the user one Socratic question at a time to surface hidden mechanism. Usage: /socratic:ask <concept|project> <topic>
argument-hint: <concept|project> <topic>
---

Run the **socratic skill in ASK mode**.

Raw arguments: `$ARGUMENTS`

## What to do

1. Load the skill body at `skills/socratic/SKILL.md` (relative to this plugin root) and obey it.
2. Parse `$ARGUMENTS` as: `<topic-mode> <topic>`
   - `topic-mode` MUST be one of: `concept` | `project`
   - `topic` is the rest of the line (a quoted string for concept, a path for project)
3. Follow the "ASK mode flow" section of SKILL.md — do not improvise the flow.

## If arguments invalid

If `$ARGUMENTS` is empty, or topic-mode is not `concept`/`project`, respond with exactly:

```
Usage: /socratic:ask <concept|project> <topic>

Examples:
  /socratic:ask concept "diffusion guidance"
  /socratic:ask concept "reward shaping in PPO"
  /socratic:ask project src/auth
  /socratic:ask project .
```

…and stop. Do not start the grill.
