# 过程笔记

## (a) 这是 skill 问题还是 CLAUDE.md 问题？

明确是 skill。判断逻辑：

- **CLAUDE.md 永远在上下文里**。放 200 行 commit 规范，代价是用户每次对话（改代码、问问题、debug）都在为这段规则付 token，哪怕这一轮根本不涉及提交。这是典型的"用永久预算买了偶发需求"。
- **写 commit message 是一个有明确触发条件的离散任务**——用户说"帮我 commit"、"写个提交信息"时才需要。这完全符合 skill 的 "description 常驻 + 正文按需加载" 模型。
- **规则还会继续长**。examples 几十条、breaking change、co-author 这些都是低频但要精确查的细节——正是 `reference/` 的典型用例。塞在 CLAUDE.md 里只会让它继续膨胀。

所以方向正确：抽成 skill，CLAUDE.md 里留一句话指路就行（类似 "commit 规范见 commit-message skill"），或者干脆不留，靠 skill 的 description 自动触发。

## (b) disable-model-invocation：auto 还是 manual？

**选 auto（不加这个字段）**。推理：

- `disable-model-invocation: true` 的适用场景是**有副作用且不可逆**的操作——push、发布、部署、真正执行 git commit 写进历史。skill-master 的 SKILL.md §2.1 明确说"有副作用默认 manual"。
- 但这个 skill 做的事是**起草 commit message 文本**，不是执行 `git commit`。起草本身无副作用——用户看完文字才决定是否提交，执行提交是下游另一步。
- 更重要的是体验：用户说"帮我 commit 这次改动"时，期望 Claude 自动按项目规范写。如果要用户先记住 `/commit-message` 才能触发，就本末倒置了——规范的价值恰恰在于"每次都按它来"，不该靠用户记命令。
- 如果未来这个 skill 扩展成**自动执行 git commit**（而不只是起草文本），那时候再加 `disable-model-invocation` 或者拆成两个 skill（起草 auto、执行 manual）。当前范围没这个问题。

结论：frontmatter 里只保留 `name` 和 `description`，其他字段不加——skill-master §5 "常见陷阱"也提醒不要为了"全面"堆字段。

## (c) SKILL.md 和 reference/ 怎么切

遵循 skill-master §1 "20% 规则"——某段内容只有 20% 调用会用到，就进 reference。

**SKILL.md 里放**（每次触发都会用到的核心）：
- 格式骨架 `type(scope): summary`
- 10 个 type 的速查表
- "写为什么不只是什么" 这个关键原则 + 一个对比例子
- 72 字符 / 语言 / 结尾不加标点这些硬约束
- breaking change footer 的格式
- co-author 格式
- scope 怎么填
- 提交前自查清单

这些是"写任何一次 commit 都可能要核对"的东西。总长控制在 ~140 行，远低于 500 行上限、也低于 300 行默认目标。

**reference/examples.md 里放**（按需查阅）：
- 每种 type 的 2-3 条具体例子
- 带 body 的完整形态
- 带 breaking change 的完整形态
- 带 co-author（结对、和 AI、多协作者）的完整形态
- 综合性反例

理由：examples 是"用户写具体某类 commit 时翻一条对着抄"的材料。单次触发平均只会看 1-2 条，没必要全塞进正文。SKILL.md 里已经给了关键原则和少量核心例子，够 80% 情况直接用。

**没有加 scripts/ 和 assets/**。commit message 是文本生成任务，没有确定性的计算逻辑可以外包给脚本；也不需要模板文件——格式简单到正文描述就能精确表达。如果将来加一个"从 diff 里自动猜 type 和 scope"的功能，那才值得出一个 script。

## (d) 跳过的澄清问题和假设

严格按 skill-master §2.1，应该先问用户 5 个问题确认意图。这次任务要求直接出成品，所以做了几个合理假设：

1. **没要到用户 CLAUDE.md 里的原 200 行**。只有用户给的规则摘要。后果是 examples 里的具体场景是我基于通用 Conventional Commits 实践重构的，可能和用户原文件里的 examples 不完全重合。交付时建议用户核对一遍，尤其是 scope 命名风格（`auth` 还是 `authentication`？`api` 还是 `http`？）要用项目里实际在用的那一套。
2. **安装层级默认项目级**（`.claude/skills/commit-message/`）。commit 规范是"这个项目"的约定，不是通用工具。如果用户维护多个风格一致的项目，再考虑挪到用户级。
3. **skill 名用 `commit-message`**。没和用户确认；备选是 `commit` 或 `git-commit`。`commit-message` 更精确地表达了"起草 message" 的边界，也和 "真执行 commit" 区分开。
4. **description 里没排除所有可能混淆的邻近 skill**（比如和发 PR、写 release note 的区分）。只排除了"发 PR / 推远端 / 改 git 配置"最常见的三类。用户如果有别的 skill 和这个容易混再补充。
5. **中英文混写的判断**。用户说"中文项目用中文"，我补了一句"英文项目用英文"，并明确"不要混写"。这是合理延伸，但如果用户项目里就是允许中英混写（比如 summary 中文、code identifier 英文），得调整。

如果这个 skill 是真的要装进项目，建议用户跑几次真实 commit 场景看触发和产出，按 skill-master §2.6 和 §3 的路子迭代 description 和正文。第一版不用追求完美。
