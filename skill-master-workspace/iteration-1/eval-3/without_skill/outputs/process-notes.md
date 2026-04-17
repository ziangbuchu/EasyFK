# 抽 commit-message skill 的决策笔记

## 这事该不该做成 skill

做。理由三条：

1. **重复触发**：commit message 是每天写的东西，不是一次性规约。每次 Claude 都要翻 CLAUDE.md 才能找到规则，浪费 context。
2. **触发边界清晰**：只在"正在起草 commit message"时需要，其他场景（读代码、写功能）完全不需要加载这 200 行规则。留在 CLAUDE.md 等于每个任务都付这份成本。
3. **内容够厚**：200 行已经超过 CLAUDE.md 里单个话题的合理体量，拖慢通读速度。

反方向也考虑过——有些团队把 commit 规约留在 CLAUDE.md 是因为 "每次都得提醒"。但 skill 的 description 写对了就会自动触发，不会漏。所以不成立。

## 要不要 disable-model-invocation

不要。理由：

- skill 价值就在自动触发。用户不会每次 commit 前都想着"先 /commit-message"。
- 触发成本很低——Claude 识别到"要写 commit"就加载，识别不到就不加载。这正是 skill 机制的正常用法。
- `disable-model-invocation` 适合的场景：skill 有副作用（调外部服务、花钱）、或者只想做成纯手动工具。这里都不沾。

## 拆分策略

SKILL.md 放**每次写 commit 都要用上**的内容：

- 格式骨架
- type 取值表
- 动机优先的核心原则
- body 什么时候写
- breaking change 标注法
- co-author 格式
- 审阅 checklist

`reference/examples.md` 放**遇到才查**的内容：

- 每个 type 的反例→正例对（几十条）
- 复杂场景的完整带 body 示例
- 边缘情况判断（跨 type、CSS 算什么、chore vs build）

分界线的判断标准：如果这条信息"每次写 commit 都要默读一遍才能写对"，留在 SKILL.md；如果是"只有遇到类似场景才翻出来对照"，去 reference。

examples 单独放是因为：

- 它量大（几十条示例本身就占版面）。
- 它是查阅式的，Claude 在具体 commit 场景下才需要某一两条，而不是全读。
- 放在 SKILL.md 会让主规则被示例淹没。

## description 字段怎么写的

写了三句：**什么时候触发 + 覆盖什么 + 适用哪些场景**。

- 第一句给触发信号（写 git commit、PR 标题）。
- 第二句列覆盖的规则维度，让 Claude 知道这里有它要的答案。
- 第三句扩展边界（手动、脚本化、PR 标题），避免因为措辞差异漏触发。

避免了两种常见坑：
- 太短只写 "Write commit messages"——触发不精准。
- 太长塞细节——description 不是内容区，塞了也是重复。

## CLAUDE.md 那边怎么处理

没在交付物里处理，但推荐用户保留一行指针：

```
## Git commit

写 commit message 时参考 commit-message skill。
```

这样新人通读 CLAUDE.md 能知道有这套规范存在，不会以为项目没规约。但实际的 200 行细则完全搬走。

## 没做的事

- 没做 commit message linter / 校验脚本。skill 是给 Claude 用的，不是 CI 工具。如果用户想要 commit-msg hook，那是另一个任务。
- 没做 PR 描述模板。PR 描述比 commit message 规则更多，值得单独一个 skill，不和 commit 混。
- 没做 release notes 生成。那是 commit 规约的**下游消费**，也不属于这里。
