# writing-complex-issues

编写复杂 issue 的 agent skill：把大型任务方向拆解成 GitHub issue 树（umbrella/RFC + 原子 children），并把每个 issue thread 补齐到「headless agent 零额外指令、只看 issue 不偏离」的目标完备标准。

> **仅 claude-fable 模型可用。** 方法要求全局求解相互耦合的决策并自行裁决、做多轮对抗自审——其他模型执行会产出臆断契约与不完备的目标快照。

## 安装

```bash
npx skills add mouriya-s-lab/writing-complex-issues
```

按提示选择目标 agent（Claude Code / Codex 等）。也可直接试用不安装：

```bash
npx skills use mouriya-s-lab/writing-complex-issues
```

## 这个 skill 解决什么

普通 issue 写作假设有人在旁边补上下文；headless 执行没有这个人。本 skill 把 issue 定义为**任务目标的完备表达**——业务（为什么做、做成后谁观察到什么）、架构（系统的哪个部件、哪条边界、防什么漂移）、代码锚点（已知事实钉在哪）三层齐备，同时**不预钉实现**：

- **内容三分类**：已知事实必写且自包含；本质自由的细节必不写（写了是虚假约束）；不实验不知道的必须标记为未知并给实验路径。
- **失败模式引导，不设禁令围栏**：每条默认值带理由，偏离默认允许——条件是写明例外的理由。
- **决策不回传**：留白决策点经可判定性测试后全局求解、自行裁决；不可判定的进「未决设计问题」并 gate 依赖它的 children。
- **thread 分层**：body（目标）→ 决策 comment → 架构切片 → 观测义务 → 设计修正，append-only 补齐。

## 结构

```
skills/writing-complex-issues/
  SKILL.md                          # 标准、硬基线、三分类、路由（按阶段必读）、思维方式
  references/
    umbrella-body.md                # 开树：umbrella 十区块（系统定位锚、契约、未决设计问题、分阶段…）
    child-body.md                   # 写 child：九区块 + 四种 child 类型 + parent 归属
    comment-layers.md               # 补齐 thread：决策 / 架构切片四问 / 观测义务 / 设计修正
    decision-closure.md             # 收口留白：三分类详则、可判定性测试、全局求解
    adversarial-review.md           # 审查：换面扫描至干涸、生成性基准迭代法
    mechanics.md                    # 落地：sub-issue 挂接、re-parent、发布前自检
```

SKILL.md 的路由表标明每份 reference 在什么情形下**必读**、何时可不读及为什么——它们是阶段门控的必读件，不是可选参考。

## License

MIT
