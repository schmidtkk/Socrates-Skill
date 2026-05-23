# Socrates Skill — the Socratic-Feynman method as a Claude Code plugin

**Language / 语言:** [English](#english) · [简体中文](#简体中文)

---

<a id="english"></a>

## English

A two-phase discipline for understanding the mechanism behind something:

- **Socratic phase** — ask sharp, discriminative questions to find the gaps. Question quality matters more than quantity.
- **Feynman phase** — re-articulate the answers as plain-language prose. If you cannot tell the mechanism as a coherent story, you have not understood it; the gap in articulation points back to the next question.

Three commands across two phases:

- **`/socratic:ask`** — Claude grills *you* one question at a time, with structured candidate answers + free-text fallback. At session end, Claude writes a `mechanism.md` whose header is a **Feynman Synthesis** threading your answers into a 4-paragraph story.
- **`/socratic:think`** — Claude self-grills over a concept or code path, generates a candidate question list, **pauses for you to review/edit the list once**, runs read-only probes, then writes the same `mechanism.md` (§0 Feynman Synthesis at the top + §1-§9 audit below).
- **`/socratic:feynman`** — Teaching-mode synthesis pass over a prior session. Re-derives the mechanism explanation in blog-post voice for a smart reader new to this design, surfaces things it **couldn't explain simply** as interactive gap prompts, and writes a human-readable `conclusion.html`.

It is **not** a Q&A assistant, a code reviewer, or a planning tool. It is a discipline for asking *few sharp questions* and producing *verifiable mechanism explanations that read as stories*.

### Install

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

### What it does

| Command | Mode | Who gets grilled | User touchpoints | Output |
|---|---|---|---|---|
| `/socratic:ask <concept\|project> <topic>` | live dialogue | you | answers every question | live Q&A + `.socrates/<ts>/mechanism.md` at session end |
| `/socratic:think <concept\|project> <topic>` | self-grill + plan review | Claude | **one mandatory review** of the question list before probes run | full `.socrates/<ts>/` artifact tree including candidate JSON, probe evidence, and mechanism model (§0 Feynman Synthesis + §1-§9 audit) |
| `/socratic:feynman [session-path]` | teaching-mode synthesis | — | draft review checkpoint + interactive gap prompts | `.socrates/<ts>/conclusion.html` — blog-style explanation with failure list |

ASK and THINK enforce the same grill/synthesis rules under the hood. FEYNMAN reuses the verification and gap-honesty rules, but does not run the critic gate unless it asks you to start a new `:think` pass.

- **Question budget:** target 3–5 grill questions, absolute cap 7, then forced synthesis
- **Critic gate:** ≥ 8 candidate questions generated, each scored on a two-axis rubric (Paul/Elder type × six quality criteria), only kept questions are asked
- **Probe execution:** read-only operations auto-run (Read/Grep/Glob/WebSearch/WebFetch/git log); side-effect operations enumerated as "pending probes" requiring approval
- **Two-pass synthesis:**
  - **Pass 1** fills 9 audit sections (Surface · Pressure · Core mechanism · Why this form · Why not alternatives · Invariants · Failure modes · Evidence · Remaining uncertainty), each claim tagged `[verified: evidence/NN]` / `[user-asserted]` / `[unverified]` / `[assumption]`.
  - **Pass 2** writes a **§0 Feynman Synthesis** at the top — 4 paragraphs in SCQA shape (Situation → Complication → Question → Answer), every claim followed by a Toulmin warrant ("because…"), verification tags inherited as inline markers (`^v` verified, `^p` user-asserted, `^u` unverified, `^a` assumption), and an explicit anti-alternative clause pointing to §5. Missing sections are not smoothed over — they are named in §0 as gaps and surfaced in §9.

### Use cases

#### 1. Understand a paper's mechanism, not just its result

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
## 0. Feynman Synthesis

**Situation.** Classifier-free guidance is a sampling-time trick in diffusion models
that mixes a class-conditional score with an unconditional one to sharpen samples
toward the condition [§1]^p. ...

**Complication.** The naive view treats the mix as a weighted average, but Ho & Salimans
chose subtraction with weight w-1, not a convex combination [§2]^p. The reason this
matters is that subtraction can over-emphasize the conditional direction in ways an
average never could…

## 3. Core mechanism
Guidance amplifies the conditional direction in score space by treating the
unconditional model as a "background" to subtract. [user-asserted]

## 9. Remaining uncertainty
The Karras 2024 distillation reinterpretation was not probed. If it's correct,
question Q3 should be re-opened with a derivation probe.
```

#### 2. Reverse-engineer an unfamiliar code module

```
/socratic:think project src/auth
```

Project mode runs **fetch-then-think** — Claude first reads README, globs the directory, greps for entry symbols, then generates candidates grounded in what's actually there.

After candidates are drafted you get a **review checkpoint** — Claude prints the kept question list (with Paul-type and decision-impact for each) plus 2–3 dropped candidates with reasons, and asks via AskUserQuestion:

> How do you want to proceed?
> - Proceed with these 5 questions (recommended)
> - Drop some (Claude asks a follow-up for IDs)
> - Add a question (Claude asks a follow-up for the question text)
> - Pivot — regenerate from a different angle (Claude asks a follow-up for the angle)
> - Skip-grill — go straight to synthesis

Once you approve, Claude runs the probes and writes the artifacts at `.socrates/<ts>/`:

```
.socrates/2026-05-21-1830-think-project-src-auth/
  priming/00-readme.md
  priming/01-glob-top.txt
  priming/02-grep-session-init.txt
  candidates.json          # 12 candidates, 5 kept, full critic verdicts
  evidence/00-grep-lock-token.txt
  evidence/01-git-log-session-py.txt
  mechanism.md             # §0 Feynman Synthesis + §1-§9 verified mechanism model
```

Useful when you've inherited a module and need to know *why* it's shaped this way before you change it.

#### 3. Self-check before teaching or writing

```
/socratic:ask concept "reward shaping in PPO"
```

Use this when you think you understand something and want to verify by being grilled. Claude's job is to find the spot you're glossing over. The mandatory follow-up after each answer ("why did you pick that option?") prevents auto-pilot answering.

#### 4. Pre-write mechanism notes for a paper-reading session

```
/socratic:think concept "DPO loss derivation"
```

Concept mode uses **think-then-fetch** — Claude generates candidates from its training knowledge, then verifies each claim via WebFetch on authoritative sources during the probe phase. You get a verifiable mechanism model you can iterate on before reading the paper.

#### 5. Distill a completed session into a shareable teaching document

```
/socratic:feynman
```

Run after `/socratic:think` or `/socratic:ask`. Claude re-derives the mechanism explanation in **blog-post voice** for a reader who knows the field but has never seen this specific design. If no prior session exists, it can run in **cold draft** mode from user-provided context, but the output is explicitly marked ungrilled/unverified. The discipline: every causal step must be explicit, every term glossed or dropped, no "you'd need to look at the code."

Where the explanation breaks down — jargon Claude can't gloss, circular reasoning, concepts it can only point to rather than explain — these surface as **interactive gap prompts** via `AskUserQuestion`. You can supply the missing explanation, ask Claude to dig deeper with a new `:think` pass, or leave the gap acknowledged. The failure list is a research signal, not an embarrassment.

Output is `conclusion.html` in the same `.socrates/<ts>/` directory: a human-readable document with color-coded verification markers and a failure list at the bottom.

The two phases compose: run `:think` to build a verified evidence base, then `:feynman` to distill it into something shareable. Run `:feynman` again after resolving gaps to get a tighter synthesis each time.

### What you'll actually see

Every grill question is an `AskUserQuestion` call with 3–4 candidate answers Claude has drafted + Other. The rules Claude follows when drafting those candidates are codified in `SKILL.md §6`:

- Each option = a *different* mechanism hypothesis (no two options paraphrase the same idea)
- No option marked "correct" or "recommended" (this is not a quiz)
- Other always available for "your truth isn't in my list"
- After a normal answer, Claude *must* ask why you picked that option (open text); control tokens like `skip` / `enough` / `pivot` execute immediately

Flow-control keywords work via the Other channel:

| Type into Other | Effect | Scope |
|---|---|---|
| `skip` | Skip this question, move to next | per-grill-question (ASK mode) |
| `enough` | Stop questioning immediately, synthesize from what you've got | any time |
| `pivot` | Discard remaining candidates, regenerate from new angle | any time |
| Skip-grill option | Abandon all questions, go straight to synthesis | THINK review checkpoint only |

### Architecture

The plugin lives at `plugins/socratic/` and exposes its discipline through `SKILL.md` + a `taste/` directory containing the Paul/Elder taxonomy, quality axes, two-axis critic rubric, and per-topic-mode seed banks. Everything else is convention.

```
plugins/socratic/
├── .claude-plugin/plugin.json
├── commands/{ask,think,feynman}.md
└── skills/socratic/
    ├── SKILL.md                    # source of truth — all design decisions encoded
    └── taste/
        ├── paul-elder-types.md     # 6 question types, bilingual stems
        ├── quality-axes.md         # 6 quality criteria + scoring rubric
        ├── critic-rubric.md        # two-axis scoring + drop rules
        └── seeds/
            ├── concept.md          # stems for concept/math/paper topics
            └── project.md          # stems for engineering/code topics
```

If you want to read the source of truth, start at `plugins/socratic/skills/socratic/SKILL.md`.

### Not for

- General Q&A or coding assistance — use Claude Code's default behavior
- Plan / design review — use the `Plan` agent or `grill-me` skill
- Code review or refactoring — use `code-reviewer-pro` or similar
- Debugging — use `debugger` agent
- Memory or note-taking — use the `ck` skill or Claude Code's native memory

The skill auto-loads only on `/socratic:*` invocations or explicit mentions of "Socratic questioning" / "mechanism grilling". It will not hijack normal conversations.

### Topic-mode argument grammar

```
/socratic:ask   <concept|project>  <topic>
/socratic:think <concept|project>  <topic>
/socratic:feynman [session-path]
```

- `concept` — a concept, definition, formula, paper section. Topic is a quoted string.
  - `concept "diffusion guidance"`
  - `concept "reward shaping"`
- `project` — a file, directory, or `.` for the whole repo. Topic is a path.
  - `project src/auth`
  - `project .`

`paper` and `task` were intentionally dropped from MVP — `paper` overlaps `concept`, `task` overlaps the existing `grill-me` skill.

### Naming and intellectual debts

The name "Socratic-Feynman" was suggested by the user during dogfooding. It reflects two distinct disciplines fused into one workflow:

**Socratic** — Paul/Elder's classical taxonomy of six question types (clarification, assumption, evidence, perspective, consequence, meta-question) shapes the critic. Pólya's "Understand the problem" phase shapes the opening ritual. Question quality > quantity is the core taste rule.

**Feynman** — the §0 synthesis pass follows the Feynman Technique's central claim: if you cannot explain it plainly to a smart non-specialist, you have not understood it. Structurally, §0 borrows Minto's SCQA from consulting writing, Toulmin's warrant from argumentation theory, and the ladder-of-abstraction (Hayakawa, identified by Pinker as Feynman's signature) for the move between abstract claim and concrete example.

A pure Socratic transcript is a list of Q-and-A — scattered beads. A pure Feynman synthesis without preceding Socratic discipline is confident prose that may smooth over the very gaps you should be naming. The two methods together: ask sharp, synthesize honestly.

**Key references:**

- Paul, R. & Elder, L. (2006). *The Thinker's Guide to the Art of Socratic Questioning.* Foundation for Critical Thinking.
- Pólya, G. (1945). *How to Solve It: A New Aspect of Mathematical Method.* Princeton University Press.
- Minto, B. (1978, rev. 2003). *The Pyramid Principle: Logic in Writing, Thinking, and Problem Solving.* Pearson.
- Toulmin, S. (1958). *The Uses of Argument.* Cambridge University Press.
- Hayakawa, S. I. (1939, rev. 1990 with Hayakawa, A. R.). *Language in Thought and Action.* Harcourt Brace.
- Pinker, S. (2014). *The Sense of Style: The Thinking Person's Guide to Writing in the 21st Century.* Viking. (Identifies the ladder of abstraction as a signature of Feynman's prose.)
- Feynman, R. P. (1999). *The Pleasure of Finding Things Out.* Perseus Books. (Source for the "explain it plainly" articulation test.)

### Status

MVP. Dogfooding now. Expect edits to seed banks, candidate-answer rules, critic rubric, and the §0 Feynman Synthesis contract as real friction surfaces. Open an issue (or run `/socratic:think project .` against the skill itself and post the resulting `mechanism.md`) if something feels off.

Public repo; license TBD pending stabilization.

---

<a id="简体中文"></a>

## 简体中文

一个两阶段的训练法，用来理解一件事背后的机理：

- **苏格拉底阶段** —— 用尖锐的、区分性强的问题找出理解的 gap。问题质量比数量更重要。
- **费曼阶段** —— 把答案重新写成朴素散文。如果你没法把机理讲成一个连贯的故事，那就是没真懂；讲不通的地方，正是下一个该问的问题。

三条命令，覆盖两个阶段：

- **`/socratic:ask`** —— Claude 一次问你一个问题，每题给 3–4 个候选答案 + Other 自由输入。会话结束时 Claude 写一份 `mechanism.md`，开头是一段 **§0 费曼综合**，把你的答案串成 4 段故事。
- **`/socratic:think`** —— Claude 对一个概念或代码路径进行自我 grill，先生成候选问题清单，**强制让你 review/编辑一次**，再跑只读 probe，然后写同样格式的 `mechanism.md`（§0 费曼综合在顶 + §1-§9 审计层在下）。
- **`/socratic:feynman`** —— 对已有苏格拉底会话进行**教学模式综合**：以博客风格重新推导机理解释，对"无法简单解释"的地方发起交互式追问，写出人类可读的 `conclusion.html`。

它**不是**一个 Q&A 助手、代码 reviewer 或者计划工具。它是一种纪律：**少问尖锐的问题，产出能验证、能当故事读的机理解释。**

### 安装

```
/plugin marketplace add schmidtkk/Socrates-Skill
/plugin install socratic@socratic
```

装完后重启 Claude Code（插件不会热加载）。然后 `/plugin` 应该能看到 `socratic@socratic` 处于启用状态。

**本地目录安装**（开发场景，不用走 GitHub）：

```
/plugin marketplace add /absolute/path/to/Socrates-Skill
/plugin install socratic@socratic
```

### 它做什么

| 命令 | 模式 | 谁被 grill | 用户介入点 | 产物 |
|---|---|---|---|---|
| `/socratic:ask <concept\|project> <topic>` | 即时对话 | 你 | 每一题都答 | 实时 Q&A + 会话结束时的 `.socrates/<ts>/mechanism.md` |
| `/socratic:think <concept\|project> <topic>` | 自我 grill + 计划 review | Claude | 在跑 probe 前 **强制 review** 一次问题清单 | 完整 `.socrates/<ts>/` 产物树：候选 JSON、probe 证据、机理模型（§0 费曼综合 + §1-§9 审计）|
| `/socratic:feynman [session-path]` | 教学模式综合 | — | 草稿 review + 交互式 gap 追问 | `.socrates/<ts>/conclusion.html` —— 博客风格解释 + failure list |

ASK 和 THINK 强制执行同一套 grill / synthesis 规则。FEYNMAN 复用 verification 和 gap-honesty 规则，但除非它建议你另跑一次 `:think`，否则不跑 critic gate。

- **问题预算：** 目标 3–5 个 grill 问题，绝对上限 7 个，到顶强制综合
- **Critic 闸门：** 必须生成 ≥ 8 个候选问题，每个用双轴评分（Paul/Elder 类型 × 6 个质量维度），只问通过的
- **Probe 执行：** 只读操作自动跑（Read/Grep/Glob/WebSearch/WebFetch/git log）；有副作用的操作仅列出，需要用户授权
- **两遍合成：**
  - **Pass 1** 填 9 段审计层（Surface · Pressure · Core mechanism · Why this form · Why not alternatives · Invariants · Failure modes · Evidence · Remaining uncertainty），每条 claim 打标签 `[verified: evidence/NN]` / `[user-asserted]` / `[unverified]` / `[assumption]`。
  - **Pass 2** 在最上面写一段 **§0 费曼综合** —— 4 段 SCQA 结构（Situation → Complication → Question → Answer），每条 claim 后面跟一个 Toulmin warrant（"因为…"），verification 标签作为行内 marker 继承（`^v` 已验证，`^p` 用户陈述，`^u` 未验证，`^a` 假设），并且至少有一句明示的"不是 X 因为…"反方引用 §5。N/A 段落不会被叙事流光滑掉 —— 必须在 §0 显式点名，再写入 §9。

### 用例

#### 1. 理解一篇论文的机理，而不是只记住结论

```
/socratic:ask concept "classifier-free guidance"
```

Claude 用一次 3 题的 AskUserQuestion 仪式开场，先 pin 住*你*当前的理解水平。然后问 3–5 个尖锐问题，比如：

> Q1/5 (assumption): CFG 公式用 `w-1` 权重减去无条件得分。哪个最接近你当前的心智模型？
>
> - 减法是 score perturbation —— 把采样方向往条件方向推
> - 算上等价于加权平均，只是写法不同
> - 这其实是某种 distillation 的伪装（Karras 2024 的解释）
> - 形式无所谓，只要是单调组合都行
> - Other → 自己写
>
> *(也可以选 Other 输入 `skip` / `enough` / `pivot` 来控制流程。)*

问完 5 题后 Claude 写 `mechanism.md`：

```
## 0. Feynman Synthesis

**Situation.** Classifier-free guidance 是 diffusion 模型采样阶段的一个技巧 ……[§1]^p

**Complication.** 简单解读会把 mix 当成加权平均，但 Ho & Salimans 选了 w-1 的减法，不是凸组合 [§2]^p ……

## 3. Core mechanism
Guidance 通过把无条件模型当成"背景"减掉，放大了 score 空间里的条件方向。[user-asserted]

## 9. Remaining uncertainty
Karras 2024 提出的 distillation 重新解释没被探查过。如果对，Q3 需要用 derivation probe 重开。
```

#### 2. 逆向一个不熟悉的代码模块

```
/socratic:think project src/auth
```

project 模式是 **fetch-then-think** —— Claude 先 Read README、Glob 目录结构、Grep 入口符号，然后基于真实代码生成候选问题。

候选生成完会触发一个 **review checkpoint** —— Claude 把保留的问题清单（每题带 Paul-type 和 decision-impact）+ 2–3 个被 drop 的候选（带 drop 原因）打印出来，再用 AskUserQuestion 问你：

> 你想怎么继续？
> - 就用这 5 个问题（推荐）
> - drop 某几个（Claude 会追问要 drop 哪些 ID）
> - 加一个问题（Claude 会追问问题文本）
> - Pivot —— 换个角度重新生成（Claude 会追问新角度）
> - Skip-grill —— 直接跳过问题阶段去综合

你点 Proceed 后 Claude 跑 probe，把产物写到 `.socrates/<ts>/`：

```
.socrates/2026-05-21-1830-think-project-src-auth/
  priming/00-readme.md
  priming/01-glob-top.txt
  priming/02-grep-session-init.txt
  candidates.json          # 12 个候选，5 个保留，完整 critic 评分
  evidence/00-grep-lock-token.txt
  evidence/01-git-log-session-py.txt
  mechanism.md             # §0 费曼综合 + §1-§9 已验证机理模型
```

适用场景：接手一个模块，动它之前想搞清楚*为什么*是这个形态。

#### 3. 在教别人或写之前自检

```
/socratic:ask concept "reward shaping in PPO"
```

适合"你觉得自己懂了，想让人 grill 一下"的场景。Claude 的工作就是找你在哪一步是在糊弄过去的。每题答完后强制问"为什么选这个？"，防止你 auto-pilot 选项。

#### 4. 读论文前先写机理草稿

```
/socratic:think concept "DPO loss derivation"
```

concept 模式是 **think-then-fetch** —— Claude 先从训练数据生成候选，然后在 probe 阶段用 WebFetch 验证每条 claim。读论文前先有一份可迭代的机理模型。

#### 5. 把已完成会话蒸馏成可分享教学文档

```
/socratic:feynman
```

通常在 `/socratic:think` 或 `/socratic:ask` 后运行。Claude 会用**博客文章口吻**重新推导机理解释，面向“懂这个领域、但没见过这个设计”的读者。如果找不到 prior session，它也可以用用户现场提供的 context 跑 **cold draft**，但输出会明确标为没有经过 grill / evidence 验证。

如果解释里出现无法 gloss 的术语、循环论证、或只能指向源码/论文而不能讲清楚的地方，这些会通过 `AskUserQuestion` 变成交互式 gap prompt。最终输出是同一 `.socrates/<ts>/` 目录里的 `conclusion.html`。

### 你实际会看到什么

每个 grill 问题都是一次 `AskUserQuestion`，附 3–4 个 Claude 起草的候选 + Other。候选答案的起草规则写在 `SKILL.md §6`：

- 每个选项 = 一个*不同的*机理假设（不允许两个选项换皮说同一件事）
- 没有任何选项被标"正确"或"推荐"（这不是答题）
- Other 永远可用，给"你的真相不在我的清单里"的情况留口
- 普通作答后 Claude *必须*问你为什么选这个（开放文本）；`skip` / `enough` / `pivot` 这类控制词会立刻执行

流程控制关键词通过 Other 走：

| Other 里输入 | 效果 | 范围 |
|---|---|---|
| `skip` | 跳过本题，进入下一题 | 单题（ASK 模式）|
| `enough` | 立刻停止问问题，用现有内容综合 | 任何时候 |
| `pivot` | 抛弃剩余候选，换角度重新生成 | 任何时候 |
| Skip-grill 选项 | 抛弃所有问题，直接跳到综合 | 仅 THINK review checkpoint |

### 架构

插件主体在 `plugins/socratic/`，纪律通过 `SKILL.md` + `taste/` 目录暴露：Paul/Elder 分类法、质量维度、双轴 critic 评分、各模式的 seed 库。其它都是约定。

```
plugins/socratic/
├── .claude-plugin/plugin.json
├── commands/{ask,think,feynman}.md
└── skills/socratic/
    ├── SKILL.md                    # 源真理 —— 所有设计决策编码于此
    └── taste/
        ├── paul-elder-types.md     # 6 类问题，中英 stems
        ├── quality-axes.md         # 6 质量维度 + 评分 rubric
        ├── critic-rubric.md        # 双轴打分 + drop 规则
        └── seeds/
            ├── concept.md          # 概念/数学/论文 stems
            └── project.md          # 工程项目 stems
```

想看源真理，从 `plugins/socratic/skills/socratic/SKILL.md` 开始。

### 不适用场景

- 一般 Q&A 或代码协助 —— 用 Claude Code 默认行为
- 计划/设计 review —— 用 `Plan` agent 或 `grill-me` skill
- 代码 review 或重构 —— 用 `code-reviewer-pro` 之类
- 调试 —— 用 `debugger` agent
- 记忆/笔记 —— 用 `ck` skill 或 Claude Code 自带 memory

skill 只在 `/socratic:*` 调用或明确提到 "Socratic questioning" / "mechanism grilling" 时加载。它不会劫持正常对话。

### Topic-mode 参数语法

```
/socratic:ask   <concept|project>  <topic>
/socratic:think <concept|project>  <topic>
/socratic:feynman [session-path]
```

- `concept` —— 概念、定义、公式、论文章节。`topic` 用引号包起来。
  - `concept "diffusion guidance"`
  - `concept "reward shaping"`
- `project` —— 文件、目录，或 `.` 表示整个 repo。`topic` 是路径。
  - `project src/auth`
  - `project .`

`paper` 和 `task` 故意从 MVP 拿掉了 —— `paper` 和 `concept` 重叠，`task` 和现有的 `grill-me` skill 重叠。

### 命名与思想债务

"苏格拉底-费曼"这个名字是用户在 dogfeed 阶段提的。它反映了两种不同纪律融合到一个工作流里：

**苏格拉底** —— Paul/Elder 经典的六类问题分类法（澄清、假设、证据、视角、后果、元提问）塑造了 critic。Pólya 的"理解问题"阶段塑造了开场 ritual。"问得准比问得多重要"是核心品味规则。

**费曼** —— §0 综合阶段遵循费曼技巧的核心断言：**如果你不能用朴素的语言讲给一个聪明的非专家听，你就没真懂**。结构上，§0 借了 Minto 咨询写作的 SCQA、Toulmin 论证理论里的 warrant、还有 Hayakawa 的 ladder of abstraction（Pinker 把它认定为费曼散文的标志），用来在抽象 claim 和具体例子之间上下移动。

纯苏格拉底的产物是一串 Q-A —— 散落的珠子。纯费曼的综合没有前置的苏格拉底纪律，会产出"自信的散文"恰好盖住你本该指认的 gap。两者一起：**问得尖锐，综合得诚实。**

**核心参考文献：**

- Paul, R. & Elder, L. (2006). *The Thinker's Guide to the Art of Socratic Questioning.* Foundation for Critical Thinking.
- Pólya, G. (1945). *How to Solve It: A New Aspect of Mathematical Method.* Princeton University Press.
- Minto, B. (1978, rev. 2003). *The Pyramid Principle: Logic in Writing, Thinking, and Problem Solving.* Pearson.
- Toulmin, S. (1958). *The Uses of Argument.* Cambridge University Press.
- Hayakawa, S. I. (1939, rev. 1990 with Hayakawa, A. R.). *Language in Thought and Action.* Harcourt Brace.
- Pinker, S. (2014). *The Sense of Style: The Thinking Person's Guide to Writing in the 21st Century.* Viking.（把 ladder of abstraction 认定为费曼散文的标志）
- Feynman, R. P. (1999). *The Pleasure of Finding Things Out.* Perseus Books.（"朴素解释"作为理解检验的原始出处）

### 状态

MVP。目前在 dogfeed。seed 库、候选答案规则、critic rubric、§0 费曼综合 contract 都会随真实使用摩擦而调整。有问题的话开 issue（或者对 skill 自己跑一次 `/socratic:think project .`，把产出的 `mechanism.md` 贴上来）。

公开 repo；license 待稳定后定。
