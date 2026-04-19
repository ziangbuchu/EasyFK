# EasySDD

> 厌倦了 OpenSpec 的草台、Oh-My-OpenAgent 的过度设计、SuperPowers 的散装，我从 0 写了一套简单轻巧、但**自洽闭环**的 AI 工作流。

**从开发到解决问题再到沉淀知识，是一个闭环；这一次的产出会变成下一次的输入，越用越顺。**

---

## 缘由

我一直在自己开发一套 Harness Agent（[源码](https://github.com/liuzhengdongfortest/MA)）。一开始就是 VibeCoding，我写设计、AI 写代码改 bug，挺顺利的。直到有一天 Codex 反复解决不了一个并不复杂的问题，反复在同一个地方掉链子，我意识到项目得有一套工作流来撑着才能继续推进。

调研了一圈：OpenSpec 太简单，没有复利工程，生成的 spec 抽象到人根本没法读；SuperPowers 没有流程约束，不知道该用哪个；Oh-My-OpenAgent 又太重。没一个用着顺手的，所以自己写了 EasySDD。

---

## 三大块

### Feature 工作流

很常规：`brainstorm → design → implement → acceptance`。头脑风暴、设计、实现、验收，做过 vibe coding 的人一看就懂。需求小的时候可以走 `easysdd-feature-fastforward` 跳过完整 review。

### Issue 工作流

参考了站内大佬的思路：`report → analyze → fix`。汇报 bug、分析根因、定点修复。

### 辅助工作流

这是 EasySDD 最值得一说的部分。

---

## EasySDD 跟其他框架不一样在哪

### Architecture

整个系统的架构文档常驻在 `easysdd/architecture/`。每做完一个 feature，acceptance 阶段会自动把这次的变更合进架构文档。下次再有人（或 AI）想理解这个系统现在长什么样，有一份永远跟得上代码的入口。

### 术语归一

说起来复杂，简单讲就是统一术语，确保人讲的、AI 理解的、AI 讲回来的、人理解的，是同一个东西。这件事用过你就知道值了。

### Compound（重中之重）

这是 EasySDD 的灵魂。专门设计了一组技能来沉淀复利工程——说白了就是积累知识：

- `easysdd-learning`：从项目里学到了什么
- `easysdd-tricks`：可复用的编程技巧
- `easysdd-decisions`：项目里拍板的技术决定
- `easysdd-explore`：一次代码探索的成果

**那归档之后怎么召回？** EasySDD 只在需要它的时候召回。`easysdd-feature-design` 起草设计前会显式去搜 compound 目录，`easysdd-issue-analyze` 分析 bug 时也会显式去查找。

这样整个系统就处于一种自洽状态：feature 和 issue 过程中积累的知识会反哺到下一次设计里，越往后越顺。

### 工作清单（YAML checklist）

design 完成以后，模型会生成一份 yaml 格式的 checklist，而不是 markdown 或 csv。实测起来 yaml 的遵从率明显更高，我也不知道为啥，但能用。

### Onboarding 技能

专门做了一个上船技能，帮你一键搭出 EasySDD 的项目结构。空仓库和已有零散文档的仓库都能接。

### Libdoc 和 Guidedoc 技能

两个我自己写文档时高频用到的：libdoc 维护库的 API 参考，guidedoc 写开发者/用户指南。自用起来相当趁手。

---

## 一些实际效果

- **自动积累知识**：feature acceptance 阶段会自动写 learning，并更新 architecture 文档
- **关于 Tauri 编译问题的踩坑沉淀**：上次踩过的坑，下次 design 时直接被搜到
- **feature 和 issue 显性存档**：要往回翻"当时解决了哪些 issue"时有据可查

---

## 模型适配

我自己在家和公司都在用。Claude 模型和 GLM 4.7 都能跑，但 Claude 可以一口气吃下一个大颗粒 feature，GLM 4.7 只能切小颗粒，每次 design 还得我把关。模型能力差距，没办法。

---

## 设计上的权衡

> 那 worktree、code-review、子代理开发这些技能呢？

这是设计取舍。小到 OpenSpec、大到 Oh-My-OpenAgent，每家都有自己的权衡。我并不想做一个大而全的框架，**够用就行**——能帮我持续推项目、让项目持续受控、让我安心，就 OK。没有复杂的 Subagent 和 hook 来污染你的 Claude 或 Codex。

---

## 怎么用

```bash
npx skills add https://github.com/liuzhengdongfortest/easysdd
```

进项目后对 Claude 说：

> 「在这个项目里初始化 easysdd」

然后开始第一件事：

> 「我要做 X 功能，走 easysdd-feature 流程」
>
> 「这里有个 bug，走 easysdd-issue 流程」

不知道用哪个子技能也没关系，根技能 `easysdd-core` 会自己路由。

---

## 最后

也不要全依赖 EasySDD。框架虽好不能解决所有情况。我自己只在特性复杂度足够高的时候才走完整流程——你既然约束了 AI 按流程办事，token 是会多耗一些的。一般的小 UI 调整我直接截图 vibe 就完事了。一般一个 feature 走完，Claude Code 的 Opus 大概要一个 200k 的上下文。

GitHub：<https://github.com/liuzhengdongfortest/EasySDD>

---

## License

MIT

## 感谢

部分想法来自 [Linux Do 社区](https://linux.do/)。
