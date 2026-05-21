# Socrates Skill

A Claude Code plugin that turns Claude into a Socratic interviewer when you want to **understand the mechanism behind something**, not just learn its surface. Two commands:

- **`/socratic:ask`** — Claude grills *you* one question at a time, with structured candidate answers + free-text fallback, to surface the mechanism you don't yet see.
- **`/socratic:think`** — Claude self-grills *silently* over a concept or code path, runs read-only probes to verify, and writes a 9-section `mechanism.md` you can read or commit.

It is **not** a Q&A assistant, a code reviewer, or a planning tool. It is a discipline for asking *few sharp questions* and producing *verifiable mechanism explanations*.

---

## Install

```
/plugin marketplace add schmidtkk/Socrates-Skill
/plugin install socratic@socratic
```

Restart Claude Code after install (plugins are not hot-reloaded). Then `/plugin` should list `socratic@socratic` as enabled.

**Local-directory install** (for development, no GitHub round-trip):

```
/plugin marketplace add /absolute/path/to/Socrates-Skill
/plugin install socratic@socratic
```

---

## What it does

| Command | Mode | Who gets grilled | Output |
|---|---|---|---|
| `/socratic:ask <concept\|project> <topic>` | live dialogue | you | live Q&A + `.socrates/<ts>/mechanism.md` at session end |
| `/socratic:think <concept\|project> <topic>` | silent self-grill | Claude | full `.socrates/<ts>/` artifact tree including candidate JSON, probe evidence, and 9-section mechanism model |

Both commands enforce the same taste rules under the hood:

- **Question budget:** 3–5 hard cap, 7 soft cap, then forced synthesis
- **Critic gate:** ≥ 8 candidate questions generated, each scored on a two-axis rubric (Paul/Elder type × six PRD quality criteria), only kept questions are asked
- **Probe execution:** read-only operations auto-run (Read/Grep/Glob/WebSearch/WebFetch/git log); side-effect operations enumerated as "pending probes" requiring approval
- **Synthesis:** 9-section template (Surface · Pressure · Core mechanism · Why this form · Why not alternatives · Invariants · Failure modes · Evidence · Remaining uncertainty), missing sections labeled `N/A: <reason>` (the absence itself is signal)

---

## Use cases

### 1. Understand a paper's mechanism, not just its result

```
/socratic:ask concept "classifier-free guidance"
```

Claude opens with a 3-question AskUserQuestion ritual to pin down *your* current understanding level. Then asks you 3–5 sharp questions like:

> Q1/5 (assumption): The CFG formula subtracts the unconditional score with weight `w-1`. Which is closer to your current mental model?
>
> - Subtracting is a score perturbation — push the sampling direction toward conditional
> - It's algebraically equivalent to weighted averaging, just rewritten
> - It's a distillation step in disguise (Karras 2024 interpretation)
> - The form is incidental; any monotonic combination works
> - Other → type your own
>
> *(You can also select Other and type `skip` / `enough` / `pivot` to control flow.)*

After 5 questions Claude writes `mechanism.md`:

```
## 3. Core mechanism
Guidance amplifies the conditional direction in score space by treating the
unconditional model as a "background" to subtract. [verified: evidence/02-hosalimans-2022.md]

## 5. Why not alternatives
Weighted averaging preserves the unconditional contribution proportionally,
which empirically blurs class boundaries — confirmed in Ho & Salimans Fig 4.
[verified: evidence/02-hosalimans-2022.md]

## 9. Remaining uncertainty
The Karras 2024 distillation reinterpretation was not probed. If it's correct,
question Q3 should be re-opened with a derivation probe.
```

### 2. Reverse-engineer an unfamiliar code module

```
/socratic:think project src/auth
```

Project mode runs **fetch-then-think** — Claude first reads README, globs the directory, greps for entry symbols, then generates candidates grounded in what's actually there. Output at `.socrates/<ts>/`:

```
.socrates/2026-05-21-1830-think-project-src-auth/
  priming/00-readme.md
  priming/01-glob-top.txt
  priming/02-grep-session-init.txt
  candidates.json          # 12 candidates, 5 kept, full critic verdicts
  evidence/00-grep-lock-token.txt
  evidence/01-git-log-session-py.txt
  mechanism.md             # 9-section verified mechanism model
```

Useful when you've inherited a module and need to know *why* it's shaped this way before you change it.

### 3. Self-check before teaching or writing

```
/socratic:ask concept "reward shaping in PPO"
```

Use this when you think you understand something and want to verify by being grilled. Claude's job is to find the spot you're glossing over. The mandatory follow-up after each answer ("why did you pick that option?") prevents auto-pilot answering.

### 4. Pre-write mechanism notes for a paper-reading session

```
/socratic:think concept "DPO loss derivation"
```

Concept mode uses **think-then-fetch** — Claude generates candidates from its training knowledge, then verifies each claim via WebFetch on authoritative sources during the probe phase. You get a verifiable mechanism model you can iterate on before reading the paper.

---

## What you'll actually see

Every grill question is an `AskUserQuestion` call with 3–4 candidate answers Claude has drafted + Other. The rules Claude follows when drafting those candidates are codified in `SKILL.md §6`:

- Each option = a *different* mechanism hypothesis (no two options paraphrase the same idea)
- No option marked "correct" or "recommended" (this is not a quiz)
- Other always available for "your truth isn't in my list"
- After your answer, Claude *must* ask why you picked that option (open text) — the Socratic openness lives here

Flow-control keywords work via the Other channel:

| Type into Other | Effect |
|---|---|
| `skip` | Skip this question, move to next |
| `enough` | Stop questioning immediately, synthesize from what you've got |
| `pivot` | Discard remaining candidates, regenerate from new angle |

---

## Architecture (one sentence)

The plugin lives at `plugins/socratic/` and exposes its discipline through `SKILL.md` + a `taste/` directory containing the Paul/Elder taxonomy, PRD quality axes, two-axis critic rubric, and per-topic-mode seed banks. Everything else is convention.

```
plugins/socratic/
├── .claude-plugin/plugin.json
├── commands/{ask,think}.md
└── skills/socratic/
    ├── SKILL.md                    # 11-section spec, source of truth
    └── taste/
        ├── paul-elder-types.md     # 6 question types, bilingual stems
        ├── quality-axes.md         # 6 quality criteria + scoring rubric
        ├── critic-rubric.md        # two-axis scoring + drop rules
        └── seeds/
            ├── concept.md          # stems for concept/math/paper topics
            └── project.md          # stems for engineering/code topics
```

If you want to read the source of truth, start at `plugins/socratic/skills/socratic/SKILL.md`.

---

## Not for

- General Q&A or coding assistance — use Claude Code's default behavior
- Plan / design review — use the `Plan` agent or `grill-me` skill
- Code review or refactoring — use `code-reviewer-pro` or similar
- Debugging — use `debugger` agent
- Memory or note-taking — use the `ck` skill or Claude Code's native memory

The skill auto-loads only on `/socratic:*` invocations or explicit mentions of "Socratic questioning" / "mechanism grilling". It will not hijack normal conversations.

---

## Topic-mode argument grammar

```
/socratic:ask   <concept|project>  <topic>
/socratic:think <concept|project>  <topic>
```

- `concept` — a concept, definition, formula, paper section. Topic is a quoted string.
  - `concept "diffusion guidance"`
  - `concept "reward shaping"`
- `project` — a file, directory, or `.` for the whole repo. Topic is a path.
  - `project src/auth`
  - `project .`

`paper` and `task` were intentionally dropped from MVP — `paper` overlaps `concept`, `task` overlaps the existing `grill-me` skill.

---

## Status

MVP. Dogfooding now. Expect edits to seed banks, candidate-answer rules, and the critic rubric as real friction surfaces. Open an issue (or run `/socratic:think project .` against the skill itself and post the resulting `mechanism.md`) if something feels off.

Repo currently private; license TBD pending stabilization.
