# Concept Mode — Seed Questions

Seed bank for `/socratic:ask concept` and `/socratic:think concept`. These are **stems** — adapt them to the specific topic. Never paste verbatim.

The bank is organized by Paul/Elder type to support the diversity check (`critic-rubric.md` step 5).

---

## Ritual seeds (for opening self-check AskUserQuestion in ASK mode)

These probe the user's current cognitive state. Use 2–3 of these per session.

### Understanding level

```
你对 "<topic>" 的当前理解程度是？
- 听过名字，没用过
- 用过 / 见过实现，但说不清为什么这么设计
- 知道公式或定义，但有些边界条件不确定
- 自认为搞懂了，想验证机理
```

### Gap focus

```
你最想搞清楚的是哪一类？
- 为什么是这个数学/逻辑形式（推导 / 直觉）
- 边界 / 反例 / failure mode
- 和相邻方法的区别
- 工程实现里的坑
```

### Material seen

```
你已经接触过哪些材料？
- 原论文 / 原始定义来源
- 二手解读（博客 / 视频 / 教科书）
- 代码实现
- 都还没
```

---

## Grill seeds, by Paul-type

### Type 1 — Clarification (Pólya: "understand the problem")

- 你说的 `<key term>` 是什么意思？里面的 `<sub-symbol>` 指什么？
- 这个定义里 `<condition X>` 是必要的，还是只是证明方便？
- 能换种说法吗？比如不用术语 `<jargon>`。
- 这个概念和邻近概念 `<adjacent>` 的边界在哪？
- 能给一个最小的例子吗？最退化的非平凡例子是什么？

### Type 2 — Assumption

- 这个定义里默认了什么？比如默认了 `<some property>` 总是成立。
- 如果去掉条件 `<X>`，会坏在哪里？
- 这个公式是不是假设了 `<distribution / regularity / smoothness>`？
- 哪些前提是数学上必要的，哪些只是历史习惯？

### Type 3 — Evidence

- 原论文里是怎么证成这一点的？是 derivation、empirical 还是 just stipulation？
- 有没有反例能证明 naive 版本不够？
- 如果存在一个 `<X>` 不满足这个性质，会发生什么？这种 `<X>` 在实际里见过吗？
- 哪个实验 / ablation 最能证伪这个 mechanism 解释？

### Type 4 — Perspective / Alternatives

- 为什么不是 `<obvious alternative>`？比如加法不是减法？
- `<this method>` 和 `<sibling method>` 解决的是同一个 pressure 吗？
- 反对这个 mechanism 解释的人会怎么说？
- 谁会觉得这个设计不必要 / 过度工程？为什么？

### Type 5 — Consequence

- 如果这个 mechanism 解释是对的，下游模型 / 算法应该有什么 specific 性质？这条性质能直接测吗？
- 如果 `<parameter>` 趋向极限会发生什么？哪些限定情形能用解析解 sanity-check？
- 如果这条 invariant 真的成立，是不是 `<some claim>` 也必须成立？
- 这个解释能预测一个没被原作者提出的现象吗？

### Type 6 — Meta-question

- 我们为什么在意这个机理？回答之后我们的下一步会变成什么？
- 这个问题本身假设了什么？也许我们问错了问题？
- 如果只能挑一个问题问，机理理解收益最大的是哪个？
- 这个机理理解会被未来的论文 / 实验改写的概率有多大？

---

## Anti-patterns (drop these)

- "请总结一下 `<topic>`" — not Socratic, not discriminative.
- "你怎么看 `<topic>`？" — too open, no probe attached.
- "`<topic>` 的优点是什么？" — leads to platitudes, not mechanism.
- "`<topic>` 重要吗？" — yes/no, no info gain.
