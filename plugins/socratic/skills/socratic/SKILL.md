---
name: socratic
description: Socratic-inspired mechanism exploration. Loads when the user invokes /socratic:ask or /socratic:think, or explicitly asks for "Socratic questioning", "mechanism grilling", "help me reveal the mechanism behind X", or "grill me on <topic> through mechanism questions". Do NOT auto-load for general Q&A, debugging, or plan reviews — those are handled by grill-me / Plan / other skills.
---

# Socratic Skill

This skill exposes two slash commands:

| Command | Mode | Audience | Output |
|---|---|---|---|
| `/socratic:ask <concept\|project> <topic>` | dialogue | the **user** | live grill, then a synthesis at session end |
| `/socratic:think <concept\|project> <topic>` | silent self-grill | **Claude** | `.socrates/<ts>/mechanism.md` + supporting artifacts |

It is **not** a generic chatbot. It enforces taste: fewer questions, sharper questions, evidence-grounded synthesis.

---

## 0. Identity statement (do not deviate)

This skill is **Socratic-inspired**, not Socratic-pure. It deviates from classical Socratic dialogue on one point: Claude provides **candidate answers** alongside each question (as AskUserQuestion options). This trades some Socratic "tutor only asks" purity for usability — and is mitigated by mandatory follow-up ("why did you pick that one?") and Other-channel handling (see §6).

Everything else follows Socratic discipline: probe assumptions, demand evidence, surface alternatives, examine consequences.

---

## 1. Taste (apply to every question generated)

A question is worth asking ONLY if it is all six:

1. **Mechanism-revealing** — exposes *why* something is designed/defined that way
2. **Uncertainty-reducing** — narrows a known unknown
3. **Discriminative** — distinguishes two possible explanations or designs
4. **Falsifiable** — can be tested by evidence, counterexample, derivation, or probe
5. **Grounded** — tied to material (code, paper, formula, doc, experiment)
6. **Actionable** — the answer changes a next step (understanding, plan, verification, or risk)

If a candidate question fails any of the six → **drop it**. See `taste/quality-axes.md` for the rubric.

Each question also has a **type** (Paul/Elder taxonomy) from `taste/paul-elder-types.md`:
clarification · assumption · evidence · perspective · consequence · meta-question.

The critic checks **both axes** before keeping a question (see `taste/critic-rubric.md`).

---

## 2. Universal contracts (apply to BOTH `ask` and `think` modes)

### 2.1 Question budget

- **Hard cap:** 5 grill questions per session.
- **Soft cap:** 7 (you may add up to 2 follow-ups if an early answer opens a *new* high-value mechanism gap).
- **At Q7 you MUST stop questioning** and enter Synthesis, even if uncertainty remains. Unanswered branches go into the *Remaining uncertainty* section.

The Self-Check ritual (§5.1) does NOT count against the budget — it is structured AskUserQuestion, not a grill question.

### 2.2 Critic gate

Before asking ANY question, you MUST have generated at least 8 candidate questions and run them through the critic in `taste/critic-rubric.md`. The kept questions are those that pass both axes (Paul-type valid + all 6 quality criteria pass).

In `ask` mode: the candidate list is *internal* — show the user only the kept questions, with a small header `(N candidates → kept K)`.
In `think` mode: write the full candidate list with verdicts to `.socrates/<ts>/candidates.json` (see §7 schema).

### 2.3 Probe rules

Probes verify hypotheses against external evidence. Probe execution rules:

| Probe type | Auto-run? |
|---|---|
| Read / Grep / Glob | YES (no prompt) |
| WebSearch / WebFetch | YES |
| Read-only Bash (`git log`, `git show`, `git diff`, `ls`, `cat`, `wc`) | YES |
| Read-only MCP (zread, gitnexus query, context7) | YES |
| Bash with side effects (`mv`, `rm`, `git checkout`, `git commit`, script execution) | NO — enumerate in `mechanism.md` as "pending probes, needs user approval" |
| Network writes (POST/PUT/DELETE outside WebFetch GET) | NO — enumerate |
| Code edits | NO — enumerate |

Probe outputs go to `.socrates/<ts>/evidence/NN-<short-name>.{txt,json,md}` numbered sequentially.

### 2.4 Skip / enough / pivot keywords

Every AskUserQuestion you send during a grill session MUST include this line in the question text (after the actual question, on its own line):

> You can also pick **Other** and type `skip` (skip this question), `enough` (synthesize now), or `pivot` (regenerate questions from new angle).

When the user's Other response contains those tokens, behave as follows:

| Token | Action |
|---|---|
| `skip` | Mark this question as `skipped`, move to next kept candidate. |
| `enough` | Discard remaining candidates, jump to Synthesis (§8). Unasked questions become *Remaining uncertainty* entries. |
| `pivot` | Discard remaining candidates, re-run candidate generation (§2.2) using accumulated answers as new context, then continue. Pivot consumes 1 question of budget. |

---

## 3. Topic modes

| Mode | What `<topic>` is | Example |
|---|---|---|
| `concept` | A concept, definition, formula, paper section. Quoted string. | `concept "diffusion guidance"` · `concept "reward shaping in PPO"` |
| `project` | A path: file, directory, or `.` for the whole repo. | `project src/auth` · `project .` |

If the user passes a topic-mode other than these two, the slash command short-circuits with the usage example. Do not try to be helpful with `paper` or `task` — those are explicitly out of MVP scope (see §11).

---

## 4. The `.socrates/<ts>/` artifact tree

Every `/socratic:think` run, and every `/socratic:ask` run that completes Synthesis, writes to:

```
.socrates/
  <YYYY-MM-DD-HHMM>-<mode>-<topic-mode>-<topic-slug>/
    self-check.json          # ASK mode: user's ritual answers. THINK mode: Claude's virtual self-check.
    candidates.json          # full candidate list with critic verdicts (THINK always; ASK only when --persist)
    priming/                 # THINK + project mode: README, key file reads, glob outputs
      00-readme.md
      01-glob-top.txt
      …
    evidence/                # probe outputs, numbered
      00-grep-foo.txt
      01-webfetch-hosalimans-2022.md
      …
    transcript.md            # ASK mode: the actual Q&A trace
    mechanism.md             # 9-section synthesis (§8). Always written at session end.
```

The slug derivation: lowercase the topic, replace non-alphanumerics with `-`, truncate to 40 chars.

Don't write anywhere else. Don't write outside `.socrates/`. The `.gitignore` already excludes it.

---

## 5. ASK mode flow

### 5.1 Opening ritual (does NOT count against budget)

Issue ONE AskUserQuestion call with 2–3 questions probing the user's current cognitive state. Use the seed bank in `taste/seeds/<topic-mode>.md` for ritual question stems.

Constraints on ritual questions:
- Each MUST have 3–4 short selectable options + Other.
- Options MUST be discriminative (no two options describe the same state).
- The first option labeled "Other" is added automatically by AskUserQuestion — do not list it explicitly.
- Cover at minimum: (a) the user's current understanding level, (b) what kind of mechanism gap they care about.
- For `project` mode add a third question: (c) which files/areas the user has already looked at.

Persist user's answers to `.socrates/<ts>/self-check.json`:

```json
{
  "mode": "ask",
  "topic_mode": "concept|project",
  "topic": "<original input>",
  "ritual": {
    "understanding_level": "<user answer>",
    "gap_focus": "<user answer>",
    "files_seen": "<user answer, project only>",
    "other_notes": "<any Other text>"
  }
}
```

### 5.2 Candidate generation → critic → first question

1. Generate **at least 12** candidate questions tied to the topic AND the ritual answers.
2. Run each through `taste/critic-rubric.md`. Drop questions that fail the rubric.
3. From kept questions, pick the **single highest-impact** one (by `decision_impact` and Paul-type diversity).
4. Send it via AskUserQuestion. Question format:
   - The question text itself (one sentence)
   - 3–4 candidate answers Claude has drafted (see §6 for rules)
   - "Other" (auto-added)
   - The skip/enough/pivot tail line (§2.4)
5. Add `(N candidates → kept K)` as a small header above the question in chat (not inside the AskUserQuestion call).

### 5.3 Per-turn loop (Q2 through Q5/Q7)

After each user answer:

1. **Mandatory follow-up** in your chat response: ask the user *why* they picked that option (open text). This restores Socratic openness.
2. Append `{question, answer, why}` to `transcript.md`.
3. Check budget — if Q5 reached and no new high-value gap discovered, go to Synthesis. If a new gap appeared, allowed to add Q6 (and at most Q7).
4. Re-run a *fast* critic on remaining kept candidates given new information. Drop any that are now answered or stale. If too few remain, regenerate.
5. Pick next highest-impact question.

### 5.4 Synthesis at session end

When budget hit, or user said `enough`, or no kept candidates remain, run §8 (Synthesis). Write `mechanism.md`. Present a 5-line summary in chat with a link to the file.

---

## 6. Candidate-answer contract (CRITICAL — this is where ASK mode lives or dies)

When drafting the 3–4 selectable answers for each grill question, you MUST follow these rules:

### MUST
- **Discriminative**: each option represents a *different* mechanism hypothesis. Two options describing the same idea in different words → fail.
- **Plausible**: each option must be defensible by *some* reasonable interpretation. No straw-man options.
- **Coverage breadth**: span the realistic answer space. If you suspect the correct answer is in the space, at least one option should be in that region.
- **Asymmetric specificity**: options should be roughly equally detailed. Don't make one option suspiciously longer than the others — that telegraphs "this is the right one".

### MUST NOT
- Provide a single obviously-correct option (turns grill into a multiple-choice quiz, kills exploration).
- Tag an option as "(correct)" or "(recommended)". This is not a quiz; you do not know the answer.
- Provide fewer than 3 or more than 4 options (Other excluded). Fewer kills coverage, more overloads cognition.
- Use options as a vehicle to teach. The user's job is to choose / write Other / explain why. Your job is to draft and listen.

### After the answer

- ALWAYS issue the mandatory follow-up (§5.3 step 1): "为什么选这个 / 哪里不对 / 你的理解里漏了什么？"
- If user picked **Other**, that's a positive signal: your candidate set was insufficient. Note this in `transcript.md` as `"other_used": true` for that question. After the run, this should inform improvements to `taste/seeds/`.

---

## 7. THINK mode flow

### 7.1 Startup — mode-split

For `concept` mode → **think-then-fetch**:
1. Skip priming. Generate candidates immediately based on Claude's training data + the topic string.
2. During Synthesis (§8), for each `Evidence` claim, run WebSearch + WebFetch to authoritative sources to verify.

For `project` mode → **fetch-then-think**:
1. **Priming first** (write to `.socrates/<ts>/priming/`):
   - Read `README.md`, `CLAUDE.md`, `pyproject.toml`/`package.json` if present.
   - Glob top-level structure (`ls -la` equivalent via Bash, max 100 entries).
   - If `<topic>` is a path, Read its main files (`*.py`, `*.ts`, `*.js`, `*.go`, `*.md` up to 5 files).
   - Grep for likely entry symbols (`main`, `__init__`, `default export`, route registrations).
2. THEN generate candidates, grounded in the priming material.

### 7.2 Virtual self-check

Claude fills out a self-check JSON on its own behalf — what *Claude* currently believes about the topic, what's uncertain, what evidence exists. Same schema as §5.1's `self-check.json` but `"mode": "think"` and `"virtual": true`.

### 7.3 Candidate generation + critic + verification

Same critic gate as §2.2, but write the full candidate list with verdicts to `candidates.json`:

```json
[
  {
    "id": "Q1",
    "question": "...",
    "paul_type": "evidence | assumption | clarification | perspective | consequence | meta",
    "target_uncertainty": "...",
    "why_it_matters": "...",
    "expected_answer_type": "definition | evidence | counterexample | constraint | test | derivation",
    "decision_impact": "understanding | plan | risk | verification | memory",
    "probe": "<concrete probe action>",
    "quality_scores": {
      "mechanism_revealing": true,
      "uncertainty_reducing": true,
      "discriminative": true,
      "falsifiable": true,
      "grounded": true,
      "actionable": true
    },
    "keep_or_drop": "keep | drop",
    "drop_reason": "..."
  }
]
```

For each kept question, run its probe (subject to §2.3 read-only rule). Probe outputs → `evidence/`.

### 7.4 Synthesis

Run §8 producing `mechanism.md`. Each claim in the 9 sections MUST be tagged `[verified: evidence/NN]` or `[unverified]` or `[assumption]`.

---

## 8. Synthesis template (`mechanism.md`)

9 sections, **all required**. If a section has no content because evidence is insufficient, write `N/A: <one-sentence reason why this section couldn't be filled — that reason is itself a signal>`.

```markdown
# Mechanism Model — <topic>

**Generated:** <YYYY-MM-DD HH:MM>
**Mode:** ask | think
**Topic mode:** concept | project
**Session:** `.socrates/<ts>/`

## 1. Surface
<What does this thing look like on the outside? The naive description.>

## 2. Pressure
<What problem/constraint forced this design? What pressure is it responding to?>

## 3. Core mechanism
<What is the actual mechanism doing the work? The one-sentence "aha".>

## 4. Why this form
<Why is *this specific* form (definition / equation / architecture / sequence of steps) the right shape for the pressure?>

## 5. Why not alternatives
<What 1–3 alternative designs exist? Why do they fail under the constraints?>

## 6. Invariants / properties
<What does this mechanism guarantee or preserve?>

## 7. Failure modes
<Under what conditions does it break? What counterexamples or edge cases?>

## 8. Evidence
<Per-claim citation. Reference `evidence/NN` files or external URLs.>

## 9. Remaining uncertainty
<What is still unclear? What probes would resolve it? What questions went unanswered in the budget?>
```

Tag every non-trivial claim in sections 1-7 with `[verified: evidence/NN]`, `[unverified]`, or `[assumption]`. Sections 8 and 9 are inherently meta and don't need tags.

---

## 9. Out of scope (do NOT do these things)

- Do not attempt `paper` or `task` topic modes — they are not in MVP scope. Reply with the usage hint instead.
- Do not write outside `.socrates/<ts>/`. Do not modify the host project's code.
- Do not maintain a persistent question-pattern memory or mechanism-cache. Memory is handled by Claude Code's native memory system and the `ck` skill, not here.
- Do not install Claude Code hooks. This skill is prompt-disciplined, not hook-enforced.
- Do not ask the user open-text grill questions in `ask` mode. ALL grill questions use AskUserQuestion with 3–4 candidate answers + Other. Open text is only the *mandatory follow-up* after each answer.
- Do not let the candidate-answer list become a multiple-choice quiz. Re-read §6 if tempted.

---

## 10. References inside this plugin

- `taste/paul-elder-types.md` — six Paul/Elder question categories with stems
- `taste/quality-axes.md` — PRD's 6 quality criteria, rubric
- `taste/critic-rubric.md` — combined two-axis scoring, drop rules
- `taste/seeds/concept.md` — question stems for concept/paper topics
- `taste/seeds/project.md` — question stems for engineering/code topics

---

## 11. Source

This skill is a faithful implementation of the design crystallized in `prd.md` (in the plugin root) with the following deliberate deviations from the PRD:

- Renamed `/mec` → `/socratic:ask` + `/socratic:think` (clearer namespace).
- Dropped `paper` and `task` topic modes (paper folded into concept; task overlaps existing `grill-me` skill).
- Memory layer (PRD §6) removed from MVP — covered by file system + Claude Code memory + ck.
- Added: AskUserQuestion as primary interaction channel; candidate-answer drafting contract (§6); two-axis taste taxonomy from Paul/Elder + PRD qualities; mode-split THINK startup.
