# Project Mode — Seed Questions

Seed bank for `/socratic:ask project` and `/socratic:think project`. These are **stems** — adapt them to the specific path/module. Never paste verbatim.

Drawn from Staff+ engineer architecture review patterns + Paul/Elder taxonomy.

---

## Ritual seeds (for opening self-check AskUserQuestion in ASK mode)

### Understanding level

```
你对 "<path>" 的当前理解是？
- 知道大概目录布局，没读过实现
- 读过入口文件，没追到核心逻辑
- 读过核心逻辑，但说不清各模块为什么这么切分
- 改过这块代码，想验证我的心智模型
```

### Gap focus

```
你最想搞清楚的是？
- 模块边界 — 为什么这样切分
- 不变量 — 这块代码在保护什么
- failure mode — 在什么情况下会坏
- 历史包袱 — 哪些是必要约束、哪些是 legacy
```

### Files seen

```
你已经看过哪些文件？
- README / CLAUDE.md / 设计文档
- 入口 (main / __init__ / index)
- 核心 module 的实现
- 测试文件
- 都还没 (let Claude prime)
```

---

## Grill seeds, by Paul-type

### Type 1 — Clarification

- 这里的 `<key concept>`（比如 "session"、"context"、"token"）在这个项目里到底指什么？和你理解的 generic 含义有什么区别？
- 这个模块的边界在哪？哪些文件属于它，哪些只是 import 它？
- `<class / function>` 名字暗示 X，但实际行为是 Y。哪个是真相？
- 这个项目里 `<term1>` 和 `<term2>` 是同义词还是不同概念？

### Type 2 — Assumption

- 这块代码默认了什么不变量？比如默认了 `<some state>` 在调用前一定已经被初始化。
- 这个函数的 caller 都被强制满足某个 precondition 吗？怎么强制的？
- 这个 lock / mutex 假设了什么并发模型？多线程 / 多进程 / async？
- 这个错误处理代码假设了什么样的失败模式？真实失败模式真的是这种吗？

### Type 3 — Evidence

- 这个文件最近 5 个 commit 改了什么？设计意图能从 commit message 看出来吗？
- 测试文件里测了哪些行为？哪些行为没测？没测的部分是 invariant 还是 happy path？
- 这个 module 的真实调用方有几个？grep 一下：是 1 个、2 个、还是 50 个？数字本身是机理信号。
- 这个分支有 issue / PR 关联吗？设计动机藏在那里吗？

### Type 4 — Perspective / Alternatives

- 为什么是 `<this pattern>` 而不是 `<obvious alternative>`？比如为什么用 sentinel file 而不是直接共享内存？
- 这块代码合并 / 拆分 / 重排序之后会破坏什么？
- 如果用另一种语言 / 框架重写，哪些 design 会变、哪些是 fundamental？
- 这个 abstraction 是为了什么 client 设计的？如果没有这个 client，还需要这个 abstraction 吗？

### Type 5 — Consequence

- 如果这个模块 panic / crash 了，blast radius 是什么？谁会注意到、谁不会？
- 这个 invariant 如果被破坏，会立即可见还是会沉默地腐蚀状态？
- 这个 cache / queue 如果 backed up 30 分钟，下游会怎样？metrics 会先看到什么？
- 如果用户量增加 10×，第一个 bottleneck 会在哪？为什么？

### Type 6 — Meta-question

- 我们为什么要还原这块代码的机理？是要改它、扩展它、调试它、还是只是要理解它？
- 这块代码的机理是否被 `<higher abstraction>` 完全抽象掉了？也许我们该问那一层。
- 真实的不确定性来自代码本身，还是来自我们不知道某个外部依赖的行为？
- 如果作者还在，他/她会觉得我们问对了问题吗？

---

## Staff+ engineer patterns (always usable for project mode)

These are universal project-mode questions that map to Type 5 (consequence) / Type 4 (perspective):

- **Failure-first:** "What happens when `<dependency>` returns partial data?" / "What's the blast radius if `<queue>` backs up?"
- **Guarantees:** "Which of read-after-write / at-least-once / strict ordering does this actually need?"
- **Operability:** "How would you trace a request through this end-to-end?" / "What metrics indicate imminent saturation?"
- **Rollback:** "What's the rollback strategy if this deploy goes bad?"
- **Innovation cost:** "What specific problem does `<new tech>` solve that the existing tooling cannot?"

---

## Anti-patterns (drop these)

- "解释一下这个项目" — not Socratic, no probe, no specific gap.
- "这个项目用了什么技术栈？" — lookupable; drop.
- "这块代码写得好吗？" — taste-laden, not mechanism-revealing.
- "如果重写会怎么写？" — speculative, not grounded in current code.
