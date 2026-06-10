# GitHub 落地机制与发布前自检

什么时候读我：树写好了要落到 GitHub（创建、挂接、re-parent、打 label）、以及发布前最后过一遍自检时。

## 创建顺序

umbrella 先建（取得编号）→ 无依赖 children → 有依赖 children（Depends 行引用真实编号）→ 挂接 → 验证。编号一律从创建返回的 URL 解析，不预设连续——并发创建会让假设的编号漂移，comment 里的编号引用用占位符 + 创建后回填。

label 遵守目标 repo 的契约（如目标 repo 自带的 issue 分类体系：`kind:code` / `kind:comment` / `kind:spike` 之类）：目标 repo 有 issue 分类契约时它覆盖本节默认。

## sub-issue 挂接（GraphQL）

```bash
PARENT_ID=$(gh api graphql -F num=<parent> -f query='
  query($num:Int!){repository(owner:"<owner>",name:"<repo>"){
    issue(number:$num){id}}}' --jq '.data.repository.issue.id')

CHILD_ID=$(gh api graphql -F num=<child> -f query='
  query($num:Int!){repository(owner:"<owner>",name:"<repo>"){
    issue(number:$num){id}}}' --jq '.data.repository.issue.id')

gh api graphql -f query='
  mutation($p:ID!,$c:ID!){addSubIssue(input:{issueId:$p,subIssueId:$c}){
    subIssue{number}issue{number}}}' -f p="$PARENT_ID" -f c="$CHILD_ID"
```

`-F` 给类型化整数（number），`-f` 给字符串（ID），搞混报 `Variable $num of type Int! was provided invalid value`。挂完用 `subIssues.totalCount` 复核数量。已知失败形态：

- `Could not resolve to Issue node with the global id of 'PR_...'` —— 想把 PR 当 child。PR 不能做 sub-issue，重画树（在 PR 与 would-be children 间插 issue，或把 children 提为 siblings）。
- `Sub issue may only have one parent` —— child 已有 parent。要么先 `removeSubIssue` 脱钩，要么接受现 parent、这条线用散文引用。
- `Issue may not contain duplicate sub-issues` —— 幂等 no-op，忽略。
- HTTP 504 —— API 抽风，重试同一调用。

跨 repo 挂接（同 org）支持；closed parent + closed child 也能挂。re-parent 用 `removeSubIssue`（同形 mutation）脱钩再挂，**可逆且不动 body**——这是「移出 ≠ 删除」的机制保障。

## 发布前自检

逐条过草稿，对应区块见 umbrella-body.md / child-body.md：

- 标题单一问题，无「和 / + / 、」拼接；无内部脚手架 ID。
- 「背景 / 为什么」读成一段连贯论证；每句动机带可核实引用；引用的 `#N` 真实存在（不确定就 `gh issue view` 验）；操作员方向引用与方向原文逐字一致（含日期锚）。
- SKILL.md「方向解析」四槽已落位：指称对象有认定依据、或已降格为「unknown + audit 承载」；动机有来源、或已显式假设/回传；交付约束（若有）已入契约并转绝对日期。
- umbrella：系统定位区块在契约之前且选定了结构模型；未决设计问题逐项有「为什么不可判定 + 解决方 child + 被 gate 的 children」；关闭验证每行可独立证伪；残留清单每条有 owner。
- child：继承条款是逐字快照不是指针；预期结果是性质表述；验收表过了对抗自检（最省事路径不能在目标未达成时通过）；c 类未知已标记。
- 全文扫一遍「归 PR / 待定 / 再说」字样：每处过了可判定性测试（decision-closure.md）——可判定的已裁决，不可判定的进了未决区块。
- 语言与语气合硬基线（中文正文、固定 token 英文、terse imperative）。
