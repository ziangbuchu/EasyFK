---
name: change-summary
description: 每次写完或改完代码、即将结束一轮编辑工作时触发，用中文给用户一份结构化小结——列出改动过的文件、新增或调整的功能、以及是否引入了 breaking change。适用于任何包含 Edit/Write/MultiEdit/NotebookEdit 等代码修改动作的回合结尾。Trigger at the end of any turn that modified source code: produce a Chinese-language change summary covering files touched, features added or changed, and breaking-change warnings.
---

# change-summary

## 何时触发

只要这一轮对话里你（Claude）执行过任何改动代码的动作——`Edit`、`Write`、`MultiEdit`、`NotebookEdit`、或者通过 Bash 直接改文件——并且即将结束本轮回复，就在最终回复的末尾追加一份中文小结。

不触发的情况：
- 纯问答、纯阅读、纯搜索，没有落盘任何修改
- 只改了配置文件且用户明确只问"改一行"这种极小范围的操作（此时小结会比改动还长，反而噪音）
- 用户明确说"别总结了 / 直接做就行 / 不用写摘要"

## 产出格式

用户期望的是**一份紧凑的中文小结**，直接贴在最终回复末尾。格式固定为三段：

```
---
## 本轮改动小结

**改动文件**
- `path/to/file1.ts` — 一句话说明这个文件里改了什么
- `path/to/file2.vue` — 同上

**功能变化**
- 新增：<用一句话说清加了什么能力>
- 修改：<原来怎样 → 现在怎样>
- 删除：<移除了什么>

**Breaking change**
- 无  ← 如果没有，就写"无"
- 或：列出每一项 breaking change，格式为「影响面 → 谁会受影响 → 迁移办法」
```

## 怎么收集信息

按以下顺序收集，越靠前越优先：

1. **回忆本轮动作**。你自己在这一轮里调用过 `Edit` / `Write` / `MultiEdit` 的文件路径最准，直接列出来。
2. **如果不确定是否漏了什么**，在写小结前跑一次 `git status` 和 `git diff --stat` 兜底，交叉验证文件清单。不要跑 `git diff` 全文——那太长。
3. **判断功能变化**：从 diff 的语义出发，不要照抄代码行。问自己"一个不看代码的同事，读了这句能不能知道发生了什么？"
4. **判断 breaking change**：看有没有动到这些东西——导出的 API 签名、配置字段名、CLI 参数、数据库 schema、对外事件格式、默认行为。只要动了且不向后兼容，就是 breaking。内部重构不是 breaking。

## 写作准则

- 中文、短句、去 AI 味。不要"我为您完成了…""希望能帮到您"这类套话。
- 文件路径用反引号包裹。
- 一条改动一行，不要展开成段落。
- "功能变化"里如果这一轮只是修 bug 或重构，就写「修复：<bug 描述>」或「重构：<范围>，行为不变」，不要硬凑"新增"。
- "Breaking change"宁可多写一条也别漏。拿不准就写出来让用户判断。

## 反例

不要写成这样：

> 本次修改中，我对项目进行了多方面的改进和优化，包括但不限于代码结构的调整、性能的提升以及相关文档的更新，具体变动如下……

要写成这样：

> **改动文件**
> - `src/api/user.ts` — 把 `getUser` 拆成 `getUserById` 和 `getUserByEmail`
>
> **功能变化**
> - 修改：`getUser(idOrEmail)` 按参数类型分派 → 现在是两个独立函数，调用方必须选一个
>
> **Breaking change**
> - `getUser` 已移除 → 所有调用方需要改名 → 搜 `getUser(` 全局替换

## 和 git 的关系

这个小结描述的是**本轮对话造成的改动**，不是 git 仓库此刻的全部未提交差异。如果用户在对话开始前就有未提交的改动，不要把那些算进小结里。判断边界的方法：只列出你自己这一轮调用工具改过的文件。
