# Critic Rubric — Two-Axis Scoring

The critic evaluates each candidate question on TWO axes:
1. **Paul/Elder type** (`paul-elder-types.md`): exactly one of clarification / assumption / evidence / perspective / consequence / meta-question
2. **PRD qualities** (`quality-axes.md`): all six must pass

A question is **kept** only if both axes pass. Otherwise → **drop**.

---

## Step 1: Generate ≥ 8 candidates

Before scoring, generate at least 8 candidate questions. More is fine; aim for 10–15. Quantity here is necessary because the critic's job is to *filter aggressively*.

For each candidate, capture this schema:

```json
{
  "id": "Q-cand-01",
  "question": "<the question text>",
  "paul_type": "clarification | assumption | evidence | perspective | consequence | meta",
  "target_uncertainty": "<what unknown does this reduce?>",
  "why_it_matters": "<one sentence, decision-impact perspective>",
  "expected_answer_type": "definition | evidence | counterexample | constraint | test | derivation",
  "decision_impact": "understanding | plan | risk | verification | memory",
  "probe": "<concrete probe action, or 'user-supplied' if ask mode>",
  "quality_scores": {
    "mechanism_revealing": true,
    "uncertainty_reducing": true,
    "discriminative": true,
    "falsifiable": true,
    "grounded": true,
    "actionable": true
  },
  "keep_or_drop": "keep | drop",
  "drop_reason": "<if drop, which failure>"
}
```

---

## Step 2: Score Paul-type

Assign exactly one Paul-type. If the question is genuinely cross-type, prefer the *higher* type number (later types are more discriminative):
- meta-question (6) > consequence (5) > perspective (4) > evidence (3) > assumption (2) > clarification (1)

If the question doesn't fit any → it's not Socratic. **Drop.**

---

## Step 3: Score quality axes

Score each of the 6 axes as `true`/`false` per `quality-axes.md`. ALL must be `true` to keep. ANY `false` → drop, and record `drop_reason: "fails <axis>"`.

---

## Step 4: Drop rules (additional, applied after step 3)

Even if a question passes both axes, drop it under these conditions:

- **Already answered:** the answer can be inferred from material already in `priming/` or `evidence/` so far. Drop with reason `"already answered in priming"`.
- **User can self-look-up (ask mode only):** the answer is a quick search away and doesn't need user dialogue. Drop with reason `"trivially lookupable"`.
- **Off-topic:** the question targets material outside the declared topic. Drop with reason `"off-topic"`.
- **Cosmetic length:** the question exists only to fill the budget, not because it has high info gain. Drop with reason `"cosmetic"`.
- **Restates an existing kept question:** Drop with reason `"duplicate of <other-id>"`.

---

## Step 5: Diversity check (across kept set)

After dropping individually-failing candidates, examine the *kept set*:

- **Paul-type diversity:** at least 3 different Paul-types should be represented in the kept set (for a 5-question budget). If not, regenerate or downgrade overrepresented questions to make room.
- **Probe diversity:** if `think` mode, no more than 2 questions should rely on the same probe type (e.g., 5 `evidence` questions all needing WebFetch → narrow).
- **Coverage of mechanism layers:** ideally the kept set spans: surface (what), pressure (why-need), form (why-this-form), alternatives (why-not-other), failure (where-it-breaks). Missing layers → regenerate to cover.

---

## Step 6: Rank by decision impact

Among the kept set, order by `decision_impact` priority:

1. `verification` (changes what evidence we'd seek)
2. `plan` (changes next step)
3. `risk` (changes risk model)
4. `understanding` (changes mental model)
5. `memory` (changes what to record)

Ask the highest-impact question first. This way, if the user says `enough` after Q1, we extracted the best signal.

---

## Step 7: Persistence

- **ASK mode:** keep `candidates.json` *in memory only* (do not write). Show user only `(N candidates → kept K)` header.
- **THINK mode:** ALWAYS write the full list (kept + dropped, with verdicts) to `.socrates/<ts>/candidates.json`. This is the audit trail.

---

## Sanity check before sending Q1

Before issuing the first AskUserQuestion call, run a self-check on the kept Q1:

- [ ] Could I answer this with one short sentence? If yes, the question is too narrow. Reformulate.
- [ ] Would *any* answer change my next probe or plan? If no, drop and pick next.
- [ ] Is there a less-grounded version of this question floating in my candidates I prefer for stylistic reasons? If yes, kill that bias — keep the grounded one.
- [ ] Does it have the skip/enough/pivot tail line per `../contracts/interaction.md`? If not, add.

If any check fails, fix before sending.
