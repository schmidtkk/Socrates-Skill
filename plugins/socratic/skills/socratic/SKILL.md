---
name: socratic
description: Socratic-Feynman method for mechanism exploration. Loads when the user invokes /socratic:ask, /socratic:think, or /socratic:feynman, or explicitly asks for "Socratic questioning", "Feynman synthesis", "mechanism grilling", "help me reveal the mechanism behind X", or "grill me on <topic> through mechanism questions". Do NOT auto-load for general Q&A, debugging, or plan reviews — those are handled by grill-me / Plan / other skills.
---

# Socratic Skill — the Socratic-Feynman method

A two-phase discipline for understanding the mechanism behind something:

- **Socratic phase (pre-synthesis):** ask sharp, discriminative questions to find the gaps in understanding. Question quality matters more than quantity.
- **Feynman phase (post-grill):** re-articulate the answers as a plain-language story. If you cannot tell the story coherently, you have not understood — and the gap in articulation becomes the next question.

The plugin exposes three slash commands across two workflows:

| Command | Socratic phase | Feynman phase | User involvement |
|---|---|---|---|
| `/socratic:ask <concept\|project> <topic>` | Claude grills the user one question at a time | Claude synthesizes §0 from the user's answers + transcript | answers every grill question |
| `/socratic:think <concept\|project> <topic>` | Claude grills itself, user reviews the question list once | Claude synthesizes §0 from §1-§9 it filled itself | one review checkpoint on the question list |
| `/socratic:feynman [session-path]` | — (post-questioning) | Claude re-derives §0 in teaching voice; failure list surfaces gaps interactively | one draft review checkpoint + gap prompts |

ASK and THINK produce a `mechanism.md` whose header is a **§0 Feynman Synthesis** (4 SCQA paragraphs threading the mechanism into a story), followed by §1-§9 as the structured audit. FEYNMAN consumes a prior session when available and writes `conclusion.html`; in cold-start mode it writes only a clearly downgraded teaching draft.

It is **not** a generic chatbot, code reviewer, or planner. It enforces taste: fewer questions, sharper questions, evidence-grounded synthesis.

---

## 0. Identity statement (do not deviate)

The skill is **Socratic-Feynman**, not Socratic-pure or Feynman-alone. Two design choices need stating:

1. **Why not Socratic-pure:** Claude provides **candidate answers** alongside each grill question (as AskUserQuestion options). This trades some classical Socratic "tutor only asks" purity for usability — mitigated by the mandatory follow-up ("why did you pick that?") and the Other-channel (see §6).

2. **Why add Feynman:** A pure Socratic transcript is a list of Q-and-A — *scattered beads*, not a *threaded chain*. The Feynman synthesis (see §8 §0) re-articulates the answers as plain prose, structured by SCQA and made traceable by explicit Toulmin warrants. If the prose cannot be made to flow, the failure points to a gap that returns to §9 Remaining uncertainty.

**Phase boundary (CRITICAL):** the contract flips at synthesis time.
- *Pre-synthesis (Socratic phase):* questions are open, alternatives kept alive, no teaching, no candidate-answer is marked "correct".
- *Post-grill (Feynman phase):* §0 is a plain-language explanation. Teaching IS the discipline here. The boundary is "once the question budget is exhausted".

The synthesis MUST preserve what the Socratic phase produced — verification tags, anti-alternatives, ungrilled gaps. See §8 for the explicit Pass 2 contract that prevents Feynman from silently smoothing over Socratic gaps.

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

- **Target budget:** 3-5 grill questions per session.
- **Extension budget:** you may ask Q6 or Q7 only if an early answer opens a *new* high-value mechanism gap that changes the synthesis.
- **Absolute cap:** Q7. At Q7 you MUST stop questioning and enter Synthesis, even if uncertainty remains. Unanswered branches go into the *Remaining uncertainty* section.

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

Every ASK-mode grill question MUST include this line in the question text (after the actual question, on its own line):

> You can also pick **Other** and type `skip` (skip this question), `enough` (synthesize now), or `pivot` (regenerate questions from new angle).

When the user's Other response contains those tokens, handle the control token before the mandatory "why" follow-up:

| Token | Action |
|---|---|
| `skip` | Mark this question as `skipped`, move to next kept candidate. |
| `enough` | Discard remaining candidates, jump to Synthesis (§8). Unasked questions become *Remaining uncertainty* entries. |
| `pivot` | Discard remaining candidates, re-run candidate generation (§2.2) using accumulated answers as new context, then continue. Pivot consumes 1 question of budget. |

For control-token responses, do NOT ask "why did you pick that?" and do NOT append a normal `{question, answer, why}` transcript item. Append a control event instead, e.g. `{question_id, control: "skip|enough|pivot", raw_other: "..."}`.

THINK review checkpoints (§7.4) and FEYNMAN checkpoints (§9) do NOT use the generic `skip/enough/pivot` tail. They use their own checkpoint-specific options.

**Note on `/socratic:feynman`:** The FEYNMAN mode (§9) is exempt from §2.1 question budget and §2.2 critic gate — it runs no grill questions. It DOES inherit §2.3 probe rules (any probes during session detection must be read-only) and uses AskUserQuestion as its universal interaction surface for all checkpoints and gap prompts.

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
    mechanism.md             # §0 Feynman Synthesis + §1-§9 audit (per §8). Always written at session end.
    conclusion.html          # FEYNMAN mode: teaching-voice synthesis, human-readable. Written by /socratic:feynman (§9).
```

The slug derivation: lowercase the topic, replace non-alphanumerics with `-`, truncate to 40 chars.

FEYNMAN warm path writes `conclusion.html` into the resolved prior session directory. FEYNMAN cold path creates `.socrates/<YYYY-MM-DD-HHMM>-feynman-cold-<topic-slug>/` and writes only `conclusion.html`; it must not fabricate a `mechanism.md` audit.

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

1. If the user used **Other**, first inspect it for `skip`, `enough`, or `pivot` (§2.4). Control tokens take priority over the mandatory follow-up.
2. For a normal selectable answer or non-control Other answer, ask the mandatory follow-up in your chat response: *why did you pick that option?* This restores Socratic openness.
3. After the user supplies the follow-up, append `{question, answer, why, other_used}` to `transcript.md`.
4. Check budget — after Q3 you may synthesize if the mechanism is already clear; after Q5 synthesize unless a new high-value gap appeared; Q6-Q7 are extension only, and Q7 is the absolute cap.
5. Re-run a *fast* critic on remaining kept candidates given new information. Drop any that are now answered or stale. If too few remain, regenerate.
6. Pick next highest-impact question.

### 5.4 Synthesis at session end (two passes)

When budget hit, or user said `enough`, or no kept candidates remain, run the synthesis. This is **two passes**, not one.

#### 5.4.1 Pass 1 — Fill §1-§9 from the transcript [resolves C5]

Read these inputs:
- `transcript.md` — every question + user's selected option + their free-text "why"
- `self-check.json` — the ritual answers (user's stated understanding level, gap focus, materials seen)
- `evidence/*` — anything probed during the session (rare in ASK mode but possible)

Assign each claim the user articulated to its §1-§9 section. Use this mapping as a guide:

| Section | What goes here from a typical transcript |
|---|---|
| §1 Surface | The naive description the user gave (often from ritual `understanding_level`). |
| §2 Pressure | What problem/constraint the user identified as motivating the design. Often surfaces in Q1-Q2 answers. |
| §3 Core mechanism | The mechanism the user converged on (or the strongest candidate after follow-ups). |
| §4 Why this form | Reasons the user gave for the specific shape (often surfaces in Q3-Q4). |
| §5 Why not alternatives | Alternatives the user explicitly considered and rejected (often through Q4 "perspective" questions). N/A if not probed. |
| §6 Invariants | Properties the user said the mechanism preserves. N/A if not probed. |
| §7 Failure modes | Edge cases / failure conditions surfaced in Q5 "consequence" questions. N/A if not probed. |
| §8 Evidence | Citations from `evidence/` if any. In ASK mode, the user's stated material in `self-check.json.files_seen` is also a weak form of evidence. |
| §9 Remaining uncertainty | Questions skipped or partially answered; sections marked N/A above; any contradictions Claude noticed in the transcript. |

**Verification tagging is mandatory.** For every non-trivial claim in §1-§7, decide its tag:

- `[verified: evidence/NN]` — only if a probe in `evidence/` directly supports it.
- `[user-asserted]` — the user said it, no probe verified. Default for most ASK-mode claims.
- `[assumption]` — neither the user nor a probe established it; Claude inferred it to make the synthesis cohere. Treat with explicit caution.

In ASK mode the dominant tag is `[user-asserted]`. That's expected — but it must be honored by Pass 2.

#### 5.4.2 Pass 2 — Write §0 Feynman Synthesis

Re-read your own §1-§9 from Pass 1. Then write §0 at the top of `mechanism.md` per the full contract in §8 (Synthesis template). Key §0 requirements specific to ASK mode:

- **Authorship-aware framing:** use "the model you traced converges on…" / "your account so far is that…" — not "X is true". The user owns their answers.
- All other §8 §0 requirements (SCQA shape, Toulmin warrants, tag inheritance, anti-alternative reference, gap honesty, ladder oscillation) apply identically.

#### 5.4.3 Finish

Write the full `mechanism.md` (§0 followed by §1-§9). Present a 5-line summary in chat with the file path so the user can read.

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

### Phase boundary — when teaching IS allowed [resolves C6]

The "no teaching" rules above apply **only during the Socratic phase** (candidate-answer drafting, before the question budget is exhausted). Once synthesis begins (§5.4 / §7.6), the contract flips: §0 Feynman Synthesis MUST teach — write plain-language prose that a smart non-specialist can follow. The two phases are layered, not contradictory:

| Phase | Behavior | Lives in |
|---|---|---|
| Pre-synthesis (Socratic) | Ask, don't teach. Keep alternatives open. | §5.2-5.3 (ASK) / §7.3-7.5 (THINK) |
| Post-grill (Feynman) | Explain plainly. Thread answers into one story. | §5.4.2 (ASK) / §7.6.2 (THINK), per §8 §0 contract |

If you find yourself wanting to teach during a candidate-answer draft, you are crossing the boundary — defer that articulation to §0.

### After the answer

- ALWAYS issue the mandatory follow-up (§5.3 step 1): "为什么选这个 / 哪里不对 / 你的理解里漏了什么？"
- If user picked **Other**, that's a positive signal: your candidate set was insufficient. Note this in `transcript.md` as `"other_used": true` for that question. After the run, this should inform improvements to `taste/seeds/`.

---

## 7. THINK mode flow

### 7.1 Startup — mode-split

The mode-split only affects **where candidates come from** (training data vs. real material). Probe execution always happens at §7.5, regardless of mode — *after* the review checkpoint, *before* synthesis. The diagram is the same for both modes; only step 1 differs.

For `concept` mode → **think-then-fetch**:
1. Skip priming. Generate candidates immediately based on Claude's training data + the topic string. (No `priming/` directory is written.)
2. Continue to §7.2 (virtual self-check) → §7.3 (critic) → §7.4 (review checkpoint) → §7.5 (probes — for concept, these are typically WebSearch + WebFetch against authoritative sources, with their outputs landing in `.socrates/<ts>/evidence/`) → §7.6 (synthesis).

For `project` mode → **fetch-then-think**:
1. **Priming first** (write to `.socrates/<ts>/priming/`):
   - Read `README.md`, `pyproject.toml`/`package.json`, and any `CLAUDE.md` if present. **Note:** if `CLAUDE.md` is gitignored (common in active development), the resulting `mechanism.md` may silently depend on untracked local context — record this in `priming/00-sources.md` as `CLAUDE.md included (local-only, not in git)` so §0 / §9 can disclose it later.
   - Glob top-level structure (`ls -la` equivalent via Bash, max 100 entries).
   - If `<topic>` is a path, Read its main files (`*.py`, `*.ts`, `*.js`, `*.go`, `*.md` up to 5 files).
   - Grep for likely entry symbols (`main`, `__init__`, `default export`, route registrations).
2. THEN generate candidates, grounded in the priming material. Continue through §7.2 → §7.3 → §7.4 → §7.5 (probes are Read/Grep/Glob/read-only Bash, with possible WebFetch for upstream docs) → §7.6.

If the user picks **Skip-grill** at the §7.4 checkpoint, §7.5 is skipped entirely. Synthesis then runs from whatever exists in `priming/` (for project) or from candidates.json question text + training data alone (for concept, since concept has no priming). The Skip-grill path is the lowest-rigor path — §0 must explicitly say so.

### 7.2 Virtual self-check

Claude fills out a self-check JSON on its own behalf — what *Claude* currently believes about the topic, what's uncertain, what evidence exists. Same schema as §5.1's `self-check.json` but `"mode": "think"` and `"virtual": true`.

### 7.3 Candidate generation + critic

Same critic gate as §2.2. Write the full candidate list with verdicts to `candidates.json`:

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
    "drop_reason": "...",
    "final_status": "kept | dropped_by_critic | dropped_by_user | added_by_user"
  }
]
```

`keep_or_drop` is the critic's preliminary verdict (present-tense, values `keep` / `drop`). `final_status` is the canonical post-user-review state (past-participle, values `kept` / `dropped_by_critic` / `dropped_by_user` / `added_by_user`). They are NOT synonymous string-wise — `final_status: "keep"` is invalid. At Pass-1 critic time, set both:

| Critic verdict | `keep_or_drop` | `final_status` (initial) |
|---|---|---|
| Question survives both axes | `"keep"` | `"kept"` |
| Question fails any axis | `"drop"` | `"dropped_by_critic"` |

After the §7.4 review checkpoint, only `final_status` is mutated (the user can transition `kept` → `dropped_by_user`, or `added_by_user` entries can appear). `keep_or_drop` stays frozen as the critic's original judgment, useful for post-mortem analysis. Downstream code MUST filter on `final_status`, never on `keep_or_drop`.

### 7.4 **Question list review checkpoint (MANDATORY user touchpoint)**

THINK mode is NOT fully silent. After the critic finishes, the user reviews the kept question list before any probes run. This is the one place where Claude's question selection meets human judgment in THINK mode.

**Steps:**

1. **Print the kept list to chat** as a compact markdown table:

   ```
   Claude wants to pursue these N questions:

   | ID  | Paul-type    | Impact         | Question                                |
   |-----|--------------|----------------|------------------------------------------|
   | Q1  | assumption   | understanding  | Why is …?                                |
   | Q2  | perspective  | verification   | Why not the obvious alternative …?        |
   | …   | …            | …              | …                                        |

   (Generated K candidates, kept N. Dropped Q-cand-3 [fails discriminative], Q-cand-7 [duplicate of Q2].)
   ```

   Also include 2–3 representative *dropped* candidates with their drop reasons. This lets the user spot a question Claude wrongly killed.

2. **Issue an AskUserQuestion** asking the user what to do with the list. Exactly one question, with options:

   - **Proceed with the N questions as listed** *(recommended if nothing looks off)*
   - **Drop some questions** — follow-up will ask for IDs.
   - **Add a question** — follow-up will ask for the question text.
   - **Pivot — regenerate from a different angle** — follow-up will ask for the angle.
   - **Skip-grill — go straight to synthesis** — abandon all questions and synthesize from priming + training knowledge alone (lowest rigor).

   Do NOT include the generic `skip / enough / pivot` tail from §2.4 here. This is a checkpoint, not a single grill question.

3. **Apply the user's decision:**

   | User choice | Action |
   |---|---|
   | Proceed | Continue to §7.5 with the kept set unchanged. |
   | Drop some questions | Ask one follow-up AskUserQuestion for the IDs to drop. For each named ID, set `final_status: "dropped_by_user"`. Continue to §7.5. |
   | Add a question | Ask one follow-up AskUserQuestion for the question text. Append `{id: "Q-user-1", question: Y, final_status: "added_by_user", quality_scores: null, probe: null}` to `candidates.json`. Run a *fast* critic pass on Y only — if it fails any quality axis, flag in chat and proceed only after the user confirms. Then immediately derive a concrete probe for Y (subject to §2.3 read-only rule) and set `probe` to it before continuing. |
   | Pivot | Ask one follow-up AskUserQuestion for the angle. Discard current `candidates.json`, regenerate ≥ 8 fresh candidates using that angle, run critic, GOTO step 1. Pivot is allowed AT MOST ONCE per session; a second pivot request becomes a hard Proceed. |
   | Skip-grill | Mark all `kept` entries as `final_status: "dropped_by_user"`. Skip §7.5 entirely (no probes). Jump to §7.6 Synthesis. In synthesis, every §1-§9 claim that can't be grounded in priming will be tagged `[unverified]` (THINK mode) — including concept-mode sessions which have no priming at all, in which case nearly everything will be `[unverified]` and §0 must say so. |

4. **Update `candidates.json`** on disk to reflect the final statuses.

### 7.5 Probe execution

For each entry with `final_status: "kept"` or `"added_by_user"`:

1. Run the probe (subject to §2.3 read-only rule).
2. Write probe output to `.socrates/<ts>/evidence/NN-<short-name>.{txt,json,md}`.
3. Tag the question with the evidence file reference.

### 7.6 Synthesis (two passes)

Like ASK mode, THINK synthesis is two passes.

#### 7.6.1 Pass 1 — Fill §1-§9 from candidates + evidence

Read these inputs:
- `candidates.json` (after review checkpoint, with `final_status` set per entry)
- `evidence/*` (probe outputs from §7.5)
- `priming/*` (only for `project` mode; READMEs, glob output, key file reads)
- `self-check.json` (Claude's virtual self-check from §7.2)

For each `candidates.json` entry with `final_status: kept` or `added_by_user` that has a probe output in `evidence/`, derive the corresponding §1-§9 claim. The question's `decision_impact` and `paul_type` hint at which section it belongs in (e.g., `paul_type: assumption` → typically §2 Pressure or §4 Why this form; `paul_type: consequence` → typically §7 Failure modes; `paul_type: perspective` → §5 Why not alternatives).

**Verification tagging is mandatory.** For every claim in §1-§7:
- `[verified: evidence/NN]` — a probe in `evidence/` directly supports it.
- `[unverified]` — derived from candidate question text or self-check, but no probe ran (e.g., user picked `Skip` at the review checkpoint).
- `[assumption]` — neither probe nor priming established it; Claude inferred to make synthesis cohere. Treat with explicit caution in Pass 2.

Sections without any verifiable claim get `N/A: <one-sentence reason>` — the absence itself is signal (e.g., `N/A: question Q-cand-7 about alternatives was dropped by user at review checkpoint`).

#### 7.6.2 Pass 2 — Write §0 Feynman Synthesis

Re-read your own §1-§9 from Pass 1. Then write §0 at the top of `mechanism.md` per the full contract in §8.

THINK mode does NOT need authorship-aware framing (Claude wrote both §1-§9 and §0). Use direct assertion: "The mechanism is X^v, because Y^v [§3, §4]". But the other constraints from §8 §0 apply identically — SCQA shape, Toulmin warrants, tag inheritance, anti-alternative reference, gap honesty, ladder oscillation, plain language.

#### 7.6.3 Finish

Write the full `mechanism.md` (§0 followed by §1-§9). Print a 5-line chat summary including the file path.

---

## 8. Synthesis template (`mechanism.md`)

`mechanism.md` has **10 sections**: §0 (Feynman Synthesis) at the top, followed by §1-§9 (the structured audit). §0 is produced by **Pass 2** of synthesis (after §1-§9 are filled by Pass 1 — see §5.4 / §7.6 for mode-specific pass-1 mechanics).

§1-§9 are **all required**. If a section has no content because evidence is insufficient, write `N/A: <one-sentence reason — that reason is itself a signal>`.

Tag every non-trivial claim in §1-§7 with `[verified: evidence/NN]`, `[user-asserted]` (ASK mode), `[unverified]` (THINK mode), or `[assumption]`. §8 and §9 are inherently meta and don't need tags.

### 8.0 The §0 Feynman Synthesis contract (Pass 2)

§0 sits at the very top of `mechanism.md`. It is a **plain-language explanation** that threads §1-§9 into a single causal story. Its job is to make a smart non-specialist able to follow the mechanism — *and* to expose any gap that resists threading.

Why it exists: a checklist of 9 sections is *scattered beads*, not a *logical chain*. §0 is the chain.

#### Format

Exactly **4 paragraphs**, one per SCQA element, each 60–120 words (total 240–480 words):

1. **Situation** — paint the surface (draws from §1). Plain language, concrete enough that the reader sees the thing.
2. **Complication** — the pressure that surface doesn't explain (draws from §2). Why the naive view is insufficient.
3. **Question** — the specific mechanism question that emerges. Often phrased as "so why X?" or "how does Y reconcile with Z?".
4. **Answer** — the mechanism + its justification + the alternatives it beats (draws from §3-§7). End with the strongest evidence or, if unverified, the assumption made.

#### MUST

- **SCQA shape** — exactly 4 paragraphs, in S/C/Q/A order. Mark them with the labels (e.g., `**Situation.** …`).
- **Toulmin warrants** — every claim followed by "because" / "this works because" / "the reason is" linking grounds to claim. Warrants make the logical chain *visible*; without them §0 is just prose.
- **Verification-tag inheritance (C2):** every claim in §0 carries the tag of its underlying §1-§9 section. Use **four** inline markers (do not collapse — `^p` and `^u` look similar but mean different things, and the difference is load-bearing for C4):
  - `^v` after a claim drawn from `[verified: evidence/NN]` source (probe verified it)
  - `^p` after a claim drawn from `[user-asserted]` source (ASK mode only — the user said it; ownership is theirs, not Claude's)
  - `^u` after a claim drawn from `[unverified]` source (THINK mode — Claude couldn't probe it, e.g., Skip-grill path or budget exhausted before probing)
  - `^a` after a claim drawn from `[assumption]` source (Claude inferred it to make synthesis cohere)
- **Section cross-references** — bracket-cite the underlying section after each claim (e.g., `[§3]`, `[§5]`). Reader can drill down for detail.
- **Anti-alternative reference (C1):** at least one explicit `"but not X, because Y"` or `"rather than Z, which would fail because…"` clause, pointing to §5. If §5 is `N/A`, say so: `"alternatives were not investigated [§5 N/A]"`.
- **Gap honesty (C3):** any §1-§9 section marked `N/A` MUST be acknowledged in §0 (e.g., `"we did not investigate failure modes [§7 N/A], so the story below treats the mechanism as universally robust — an unverified claim"`). No silent gap-filling.
- **Ladder of abstraction (C8):** every paragraph contains at least one concrete reference — to `evidence/NN` if verified, or to an example tagged `(invented for illustration; no evidence basis)` if not.
- **Plain language** — no jargon without immediate gloss. Assume the reader knows the field but not this specific design.

#### MUST NOT

- Be a bullet list. Continuous prose only.
- Restate §1-§9 verbatim. §0 *threads*; §1-§9 enumerates.
- Exceed 4 paragraphs or 480 words.
- Drop verification markers (`^v` / `^p` / `^u` / `^a`). Their absence is a confidence-leak.
- Smooth over `N/A` sections with confident prose. Name the gap.

#### Constraint precedence (when something has to yield)

If §1-§9's actual content makes it impossible to satisfy every MUST simultaneously, yield in the order below — never sacrifice the higher item to save the lower:

1. **Gap honesty (C3)** — never sacrifice. If a section is `N/A`, you must name it. No exceptions.
2. **Verification-tag inheritance (C2)** — never sacrifice. Every claim keeps its `^v/^p/^u/^a` marker even if it makes a sentence clunky.
3. **Authorship-aware framing (C4, ASK mode only)** — never sacrifice in ASK mode. Better to write awkward "your account converges on…" prose than smooth "X is true" prose.
4. **Anti-alternative reference (C1)** — must include at least one. If §5 is `N/A`, the alternative reference becomes a *meta* gap acknowledgment ("alternatives were not investigated [§5 N/A]").
5. **SCQA paragraph labels** — drop the bold `**Situation.**` labels before merging paragraphs together. Spirit > letter.
6. **Per-paragraph word count (60–120)** — flexible by ±20% in either direction.
7. **Total word cap (480)** — flexible by +10% if and only if 1-6 above are all preserved.

If you find yourself yielding past level 4, that's a signal §1-§9 has bigger problems than §0 can compensate for — append to §9 Remaining uncertainty and consider this a near-failed synthesis.

#### ASK mode specifics (C4)

In ASK mode, §0 uses **authorship-aware framing**: "the model you traced converges on…" / "your account so far is that…" / "by the answers you gave, the mechanism appears to be…". Never assert the user's answers as truth. The user's words are theirs; Claude's job in §0 is to *re-articulate* them clearly, not to *endorse* them.

#### Failure handling

If §1-§9 cannot be threaded into 4 coherent paragraphs (contradictory claims, too many gaps, missing pressure), do not force it. Instead:

1. Write §0 with whatever does thread.
2. In §9 Remaining uncertainty, add a new entry: `"§0 could not bridge §X → §Y because <reason>; this is a real conceptual gap, not a writing failure"`.
3. The unbridgeable gap is itself a finding — surface it, don't hide it.

### 8.1 Template

```markdown
# Mechanism Model — <topic>

**Generated:** <YYYY-MM-DD HH:MM>
**Mode:** ask | think
**Topic mode:** concept | project
**Session:** `.socrates/<ts>/`

## 0. Feynman Synthesis

**Situation.** <60–120 words: the surface view, in plain language, with one concrete example. [§1] >

**Complication.** <60–120 words: the pressure / problem the surface doesn't explain, with warrant. [§2]^v or ^p (ASK mode) >

**Question.** <60–120 words: the mechanism question that emerges. Why this and not the obvious thing? [§5]^u or [§5 N/A] >

**Answer.** <60–120 words: the mechanism, with at least one Toulmin warrant linking it to the pressure, an anti-alternative clause, and an evidence/assumption tag. [§3]^v, [§4]^v, "but not X^u, which would fail because Y [§5]". >

---

## 1. Surface
<naive description, with verification tags on claims>

## 2. Pressure
<problem/constraint that forced the design, tagged>

## 3. Core mechanism
<the actual mechanism, tagged>

## 4. Why this form
<why this specific shape, tagged>

## 5. Why not alternatives
<1–3 alternatives + why they fail, tagged; or N/A if not probed>

## 6. Invariants / properties
<what it preserves, tagged; or N/A>

## 7. Failure modes
<conditions under which it breaks, tagged; or N/A>

## 8. Evidence
<per-claim citation, references to evidence/NN files or external URLs>

## 9. Remaining uncertainty
<unresolved questions; sections marked N/A above; contradictions noticed during Pass 2>
```

---

## 9. FEYNMAN mode flow

`/socratic:feynman` is a **teaching-mode synthesis pass** decoupled from the questioning phase. In warm mode it consumes a prior `:ask` or `:think` session and writes a blog-style `conclusion.html`. In cold mode it can write a lower-confidence teaching draft from user-provided context only, but it must clearly mark the output as ungrilled and unverified.

### 9.1 Session detection

On invocation, resolve the source in this order:

1. **Argument**: if `[session-path]` is provided, use that `.socrates/<ts>/` directory.
2. **Conversation context**: if no argument, check whether a `:think` or `:ask` session was completed in the current conversation (the model knows what it wrote). Use it if found.
3. **Disk auto-detect**: if no live session in context, find the latest `.socrates/<ts>/` directory on disk.
4. **Multiple sessions found** → AskUserQuestion listing the candidates, ask the user which to use.
5. **No session found** → AskUserQuestion: provide a session path / run cold from user context / cancel and run `/socratic:think` first.

Never fail silently. Every ambiguity surfaces to the user via AskUserQuestion.

### 9.2 Warm vs cold source contract

**Warm path** (prior session exists):
- `mechanism.md` is REQUIRED. If absent, ask the user for a different session path or offer cold mode.
- `candidates.json`, `evidence/*`, `priming/*`, `self-check.json`, and `transcript.md` are OPTIONAL. Read them if present; if missing, record the absence in the failure list or evidence section instead of inventing content.
- Verification markers inherit from `mechanism.md` where possible. Claims from missing optional artifacts are `^u` if stated but unverified, or `^a` if inferred.

**Cold path** (no prior session):
- AskUserQuestion for the topic and the minimum context the user wants taught.
- Create `.socrates/<YYYY-MM-DD-HHMM>-feynman-cold-<topic-slug>/`.
- Write only `conclusion.html`; do NOT fabricate `mechanism.md`, `candidates.json`, or an evidence audit.
- Put a visible `cold draft` notice near the top of the HTML. All substantive claims default to `^u` or `^a`; recommend running `/socratic:think` to build evidence.

### 9.3 Teaching-mode synthesis contract

Synthesize **as if writing a technical blog post** for this audience:

> A smart reader who knows the field but has never seen this specific design or project before. They are reading to learn, without access to source code or papers. They will read this once, linearly.

This is the **curse-of-knowledge test**: every claim must be followable by someone who does not already know it. The audience is a smart peer, not a child.

**MUST:**
- Every causal step explicit: "because…" / "which means…" / "so that…"
- Every new term glossed in the same sentence, or dropped
- Concrete example per paragraph (ladder of abstraction)
- Preserve the causal spine: surface situation → pressure/complication → mechanism question → answer. This spine may be implicit in the article structure; do not force four fixed sections.
- Toulmin warrants on every claim; verification-tag inheritance (`^v` / `^p` / `^u` / `^a`)

**Three failure modes — MUST NOT produce smooth prose; MUST produce failure-list items:**

| Code | Failure | What it reveals |
|------|---------|-----------------|
| [F1] | Jargon used without gloss | Writer knows the term; reader doesn't — gap in articulation |
| [F2] | Circular reasoning ("X because X") | Writer understands X implicitly; couldn't reconstruct it from first principles |
| [F3] | "Reader must consult source to understand this" | Writer can't explain it — only point to it |

**Project mode — code as evidence:** In `project` mode, code snippets MAY be quoted verbatim as evidence blocks. But every code block MUST be followed by a plain-language interpretation: what this code does, and why it is shaped this way. Quoting code ≠ explaining code.

### 9.4 Interactive failure list

During synthesis, when the model encounters [F1], [F2], or [F3]:

1. Mark it inline: `[F1: "<term>" not glossed]` / `[F2: circular — "<phrase>"]` / `[F3: cannot explain without source]`. Do NOT write smooth prose over it.
2. After the full draft is complete, handle failures based on count:

   - **≤ 2 failures**: resolve inline, one at a time. One AskUserQuestion per failure:
     - Options: dig deeper (propose a new :think probe on this sub-topic) / user supplies the explanation / drop this claim / leave as an acknowledged gap in conclusion.html
   - **≥ 3 failures**: collect all first, organize by type, then issue ONE AskUserQuestion:

     ```
     Conceptual gaps (F1/F2): [list]
     Evidence gaps (F3): [list]
     ```

     Options: tackle all / user picks which to resolve / user supplies bulk answers / leave all as acknowledged gaps

3. Fold any user-supplied explanations back into the synthesis draft before writing the output file.

These failure moments are high-value research signals — they expose the frontier of current understanding. Do NOT hide them. Surface them to the user as collaborative moments.

### 9.5 Draft review checkpoint

After synthesis is drafted (including failure resolution):

Issue ONE AskUserQuestion showing the article draft or a compact excerpt plus outline. Options:
- Approve — write `conclusion.html`
- Revise this section — Other: describe the change
- Add context — Other: supply additional information to fold in

**CRITICAL:** Use AskUserQuestion. Do not stop the conversation. Keep the dialogue alive between draft and final output.

### 9.6 Output: `conclusion.html`

Write `conclusion.html` to the resolved `.socrates/<ts>/` session directory.

Structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Feynman Synthesis — {topic}</title>
  <!-- embedded CSS: readable body font, monospace for code, color-coded verification markers -->
</head>
<body>
  <header>
    <h1>Feynman Synthesis — {topic}</h1>
    <p class="meta">Session: {session-path or cold draft directory} · Generated: {date} · Source: {warm|cold}</p>
    <!-- cold path only: visible warning that this draft has no prior grill/evidence base -->
  </header>

  <article>
    <!-- Blog-style teaching document. Use headings that fit the topic; do not force SCQA section labels. -->
  </article>

  <section id="failure-list">
    <h2>What could not be explained simply</h2>
    <!-- F1/F2/F3 items with user resolution notes, or a note that none remain -->
  </section>

  <section id="evidence">
    <h2>Evidence</h2>
    <!-- citations to evidence/ files; code blocks followed by plain-language interpretation -->
  </section>

  <footer>
    <p>Verification: {count ^v} verified · {count ^p} user-asserted · {count ^u} unverified · {count ^a} assumption</p>
  </footer>
</body>
</html>
```

Verification markers render as color-coded superscripts: `^v` green · `^p` blue · `^u` orange · `^a` red.

After writing, print a 3-line chat summary: file path · verification score (% of claims that are `^v`) · count of open failure-list items (0 is ideal).

---

## 10. Out of scope (do NOT do these things)

- Do not attempt `paper` or `task` topic modes — they are not in MVP scope. Reply with the usage hint instead.
- Do not write outside `.socrates/<ts>/`. Do not modify the host project's code.
- Do not maintain a persistent question-pattern memory or mechanism-cache. Memory is handled by Claude Code's native memory system and the `ck` skill, not here.
- Do not install Claude Code hooks. This skill is prompt-disciplined, not hook-enforced.
- Do not ask the user open-text grill questions in `ask` mode. ALL grill questions use AskUserQuestion with 3–4 candidate answers + Other. Open text is only the *mandatory follow-up* after each answer.
- Do not let the candidate-answer list become a multiple-choice quiz. Re-read §6 if tempted.

---

## 11. References inside this plugin

- `taste/paul-elder-types.md` — six Paul/Elder question categories with stems
- `taste/quality-axes.md` — PRD's 6 quality criteria, rubric
- `taste/critic-rubric.md` — combined two-axis scoring, drop rules
- `taste/seeds/concept.md` — question stems for concept/paper topics
- `taste/seeds/project.md` — question stems for engineering/code topics

---

## 12. Source

This skill originated from local `prd.md` working notes, which are not bundled with the distributable plugin. The current bundled contract deliberately differs from the original PRD in these ways:

- Renamed `/mec` → `/socratic:ask` + `/socratic:think` (clearer namespace).
- Dropped `paper` and `task` topic modes (paper folded into concept; task overlaps existing `grill-me` skill).
- Memory layer (PRD §6) removed from MVP — covered by file system + Claude Code memory + ck.
- Added: AskUserQuestion as primary interaction channel; candidate-answer drafting contract (§6); two-axis taste taxonomy from Paul/Elder + PRD qualities; mode-split THINK startup.
- Added `/socratic:feynman` as a teaching-mode synthesis command with warm and cold paths.
