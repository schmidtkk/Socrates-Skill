# Mechanism Synthesis

ASK and THINK write `mechanism.md`.

## Structure

`mechanism.md` has 10 sections:

```markdown
# Mechanism Model — <topic>

**Generated:** <YYYY-MM-DD HH:MM>
**Mode:** ask | think
**Topic mode:** concept | project
**Session:** `.socrates/<ts>/`

## 0. Feynman Synthesis

**Situation.** ...

**Complication.** ...

**Question.** ...

**Answer.** ...

---

## 1. Surface
## 2. Pressure
## 3. Core mechanism
## 4. Why this form
## 5. Why not alternatives
## 6. Invariants / properties
## 7. Failure modes
## 8. Evidence
## 9. Remaining uncertainty
```

Sections 1-9 are required. If a section has no content, write `N/A: <one-sentence reason>`.

## Pass 1: Sections 1-9

Map claims into:

| Section | Content |
|---|---|
| 1 Surface | Naive description. |
| 2 Pressure | Problem or constraint motivating the mechanism. |
| 3 Core mechanism | The mechanism itself. |
| 4 Why this form | Why this specific shape. |
| 5 Why not alternatives | Alternatives considered and rejected. |
| 6 Invariants / properties | What the mechanism preserves. |
| 7 Failure modes | Where it breaks. |
| 8 Evidence | Evidence files and source references. |
| 9 Remaining uncertainty | Skips, N/A sections, contradictions, unresolved gaps. |

Use verification tags from `contracts/artifacts.md`.

## Pass 2: Section 0

Section 0 is a plain-language explanation that threads sections 1-9 into one causal story.

Format: exactly 4 paragraphs, 60-120 words each:

1. **Situation.** Surface view from section 1.
2. **Complication.** Pressure from section 2.
3. **Question.** Mechanism question that emerges, including why the obvious alternative matters.
4. **Answer.** Mechanism, warrant, anti-alternative, evidence or assumption.

Requirements:

- SCQA order and labels.
- Every claim has a warrant: "because", "this works because", or "the reason is".
- Every claim inherits `^v`, `^p`, `^u`, or `^a`.
- Every claim cites the underlying section, e.g. `[§3]`.
- Include at least one anti-alternative clause pointing to section 5; if section 5 is N/A, say alternatives were not investigated.
- Acknowledge every N/A section in section 0.
- Every paragraph contains at least one concrete reference: evidence if verified, or an invented example explicitly marked as such if not.
- Plain language; gloss or drop jargon.

Constraint precedence:

1. Gap honesty.
2. Verification marker inheritance.
3. ASK authorship-aware framing.
4. Anti-alternative reference.
5. SCQA labels.
6. Paragraph word count.
7. Total word cap.

ASK mode must use authorship-aware framing: "your account so far..." / "the model you traced..." rather than "X is true".

If sections 1-9 cannot be threaded coherently, write the partial synthesis and add the unbridgeable gap to section 9.
