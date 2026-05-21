# Socrates Skill — the Socratic-Feynman method as a Claude Code plugin

A two-phase discipline for understanding the mechanism behind something:

- **Socratic phase** — ask sharp, discriminative questions to find the gaps. Question quality matters more than quantity.
- **Feynman phase** — re-articulate the answers as plain-language prose. If you cannot tell the mechanism as a coherent story, you have not understood it; the gap in articulation points back to the next question.

Two commands, both running both phases:

- **`/socratic:ask`** — Claude grills *you* one question at a time, with structured candidate answers + free-text fallback. At session end, Claude writes a `mechanism.md` whose header is a **Feynman Synthesis** threading your answers into a 4-paragraph story.
- **`/socratic:think`** — Claude self-grills over a concept or code path, generates a candidate question list, **pauses for you to review/edit the list once**, runs read-only probes, then writes the same `mechanism.md` (Feynman Synthesis at the top + 9-section audit below).

It is **not** a Q&A assistant, a code reviewer, or a planning tool. It is a discipline for asking *few sharp questions* and producing *verifiable mechanism explanations that read as stories*.

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

| Command | Mode | Who gets grilled | User touchpoints | Output |
|---|---|---|---|---|
| `/socratic:ask <concept\|project> <topic>` | live dialogue | you | answers every question | live Q&A + `.socrates/<ts>/mechanism.md` at session end |
| `/socratic:think <concept\|project> <topic>` | self-grill + plan review | Claude | **one mandatory review** of the question list before probes run | full `.socrates/<ts>/` artifact tree including candidate JSON, probe evidence, and 9-section mechanism model |

Both commands enforce the same taste rules under the hood:

- **Question budget:** 3–5 hard cap, 7 soft cap, then forced synthesis
- **Critic gate:** ≥ 8 candidate questions generated, each scored on a two-axis rubric (Paul/Elder type × six quality criteria), only kept questions are asked
- **Probe execution:** read-only operations auto-run (Read/Grep/Glob/WebSearch/WebFetch/git log); side-effect operations enumerated as "pending probes" requiring approval
- **Two-pass synthesis:**
  - **Pass 1** fills 9 audit sections (Surface · Pressure · Core mechanism · Why this form · Why not alternatives · Invariants · Failure modes · Evidence · Remaining uncertainty), each claim tagged `[verified: evidence/NN]` / `[user-asserted]` / `[unverified]` / `[assumption]`.
  - **Pass 2** writes a **§0 Feynman Synthesis** at the top — 4 paragraphs in SCQA shape (Situation → Complication → Question → Answer), every claim followed by a Toulmin warrant ("because…"), verification tags inherited as inline markers (`^v ^u ^a`), and an explicit anti-alternative clause pointing to §5. Missing sections are not smoothed over — they are named in §0 as gaps and surfaced in §9.

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

Project mode runs **fetch-then-think** — Claude first reads README, globs the directory, greps for entry symbols, then generates candidates grounded in what's actually there.

After candidates are drafted you get a **review checkpoint** — Claude prints the kept question list (with Paul-type and decision-impact for each) plus 2–3 dropped candidates with reasons, and asks via AskUserQuestion:

> How do you want to proceed?
> - Proceed with these 5 questions (recommended)
> - Drop some (Other → type IDs to drop, e.g., `drop Q2, Q5`)
> - Add a question (Other → type your question)
> - Pivot — regenerate from a different angle (Other → optionally describe the angle)
> - Skip the question phase, synthesize from priming only

Once you approve, Claude runs the probes and writes the artifacts at `.socrates/<ts>/`:

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

## Naming and intellectual debts

The name "Socratic-Feynman" was suggested by the user during dogfooding. It reflects two distinct disciplines fused into one workflow:

- **Socratic** — Paul/Elder's classical taxonomy of six question types (clarification, assumption, evidence, perspective, consequence, meta-question) shapes the critic. Pólya's "Understand the problem" phase shapes the opening ritual. Question quality > quantity is the core taste rule.
- **Feynman** — the §0 synthesis pass follows the Feynman Technique's central claim: if you cannot explain it plainly to a smart non-specialist, you have not understood it. Structurally, §0 borrows Minto's SCQA from consulting writing, Toulmin's warrant from argumentation theory, and the ladder-of-abstraction (Hayakawa, identified by Pinker as Feynman's signature) for the move between abstract claim and concrete example.

A pure Socratic transcript is a list of Q-and-A — scattered beads. A pure Feynman synthesis without preceding Socratic discipline is confident prose that may smooth over the very gaps you should be naming. The two methods together: ask sharp, synthesize honestly.

---

## Status

MVP. Dogfooding now. Expect edits to seed banks, candidate-answer rules, critic rubric, and the §0 Feynman Synthesis contract as real friction surfaces. Open an issue (or run `/socratic:think project .` against the skill itself and post the resulting `mechanism.md`) if something feels off.

Public repo; license TBD pending stabilization.
