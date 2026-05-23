# FEYNMAN Mode Flow

Command: `/socratic:feynman [session-path]`

FEYNMAN writes a teaching-mode `conclusion.html`. It does not run a new Socratic question phase.

Read before running:

- `contracts/interaction.md`
- `contracts/artifacts.md`
- `synthesis/conclusion-html.md`

## Session Detection

Resolve the source in this order:

1. Argument path, if supplied.
2. Completed `:ask` or `:think` session in current conversation.
3. Latest `.socrates/<ts>/` directory on disk.
4. If multiple candidates exist, ask the user which to use.
5. If none exists, ask whether to provide a path, run cold from user context, or cancel and run `/socratic:think` first.

Never fail silently.

## Warm Path

Use when a prior session exists.

- `mechanism.md` is required. If absent, ask for a different session or offer cold mode.
- `candidates.json`, `evidence/*`, `priming/*`, `self-check.json`, and `transcript.md` are optional.
- Read optional artifacts when present; record missing optional artifacts as evidence gaps instead of inventing them.
- Inherit verification markers from `mechanism.md` where possible.

Write `conclusion.html` into the resolved prior session directory.

## Cold Path

Use only when no prior session exists and the user chooses cold mode.

1. Ask for topic and minimum context.
2. Create `.socrates/<YYYY-MM-DD-HHMM>-feynman-cold-<topic-slug>/`.
3. Write only `conclusion.html`.
4. Add a visible `cold draft` notice near the top.
5. Mark substantive claims `^u` or `^a`.
6. Recommend `/socratic:think` to build a verified mechanism model.

Do not fabricate `mechanism.md`, `candidates.json`, or evidence.

## Failure Handling

While drafting, do not smooth over:

| Code | Failure |
|---|---|
| `F1` | Jargon used without gloss. |
| `F2` | Circular reasoning. |
| `F3` | Reader must consult source to understand. |

If <= 2 failures, resolve them one at a time with AskUserQuestion. If >= 3, group them and issue one AskUserQuestion. Options should include: dig deeper with a new `:think` probe, user supplies explanation, drop claim, or leave acknowledged.

After failure handling, show the article draft or compact excerpt plus outline in one AskUserQuestion. On approval, write `conclusion.html`.
