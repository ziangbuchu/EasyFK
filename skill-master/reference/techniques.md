# skill 写作技巧

SKILL.md 里给不了那么细的例子和模式，都在这里。碰到具体问题时翻相应小节。

## 1. description 调优

### 1.1 拆解一个好 description

基础结构 = `做什么` + `触发条件` + `触发语样例`（可选的`边界说明`）。

**反例**（典型问题在后面）：

```
description: Helps review code
```

问题：
- 动词模糊。"Helps" 可以指任何帮助
- 没说触发条件
- 没说什么时候不该触发
- 英文写给中文用户，触发命中率降一大截

**改进版**：

```
description: 用户想 review 某段代码或 PR 时进入——读 diff、识别问题、给修改建议。常见触发："帮我 review 这个 PR"、"看看我这次改动"、"扫一眼有没有问题"。只做代码层面的 review，不做架构评审（架构用另一个 skill）。
```

每一部分的作用：
- `用户想 review 某段代码或 PR 时进入` = 触发条件（用户意图）
- `读 diff、识别问题、给修改建议` = 做什么（能力范围）
- `常见触发：...` = 原话样例，命中率最高的是这些
- `只做代码层面，不做架构` = 边界，防止和相邻 skill 冲突

### 1.2 description 的"推一点"原则

Claude Code 的默认倾向是少触发 skill。所以 description 要写得**有点意图向的**——明确"这类场景就该用我"，而不是"我能帮做这个"。

| 被动描述（容易漏触发） | 意图向描述（触发率稳） |
| --- | --- |
| "Can help with commit messages" | "用户要提 commit 时进入，生成符合仓库风格的 commit message" |
| "Supports PDF extraction" | "用户给了 PDF 要提取内容、总结、查数据时进入" |
| "Dashboard builder" | "用户想做 dashboard 或数据可视化时进入——即使没明说 'dashboard' 也适用" |

### 1.3 防止过度触发

如果一个 description 容易在**不该触发的场景**里被点到，常见情况：

- 关键词太泛：`"code"`、`"data"`、`"file"` 这种词容易和无关场景撞
- 场景重叠：和另一个 skill 的触发条件接近

修补方式：

```
description: 做 X 的时候用——...（主描述）。注意：做 Y 时不要用这个，Y 用 <other-skill>。
```

显式点名相邻 skill 并排除，命中准确度会明显上升。

### 1.4 用 when_to_use 补触发语

如果 description 本身想保持精简，又希望覆盖更多触发说法，用 `when_to_use`：

```yaml
---
name: api-docgen
description: 用户要为某个 API 生成或更新文档时进入——扫代码提取签名、拼 Markdown、更新 OpenAPI spec。
when_to_use: "常见触发：'给这个接口写文档'、'更新 OpenAPI'、'把这个 handler 的参数说明补上'、'api 文档需要 refresh 一下'。"
---
```

两段拼起来不超过 1536 字符就行。

## 2. 正文组织：progressive disclosure 实战

### 2.1 什么时候拆 reference/

信号：

- SKILL.md 逼近 500 行
- 某段内容只有 1/5 左右的调用会用到
- 有长表格、长字段列表、长 API 清单——是"字典式"查阅材料，不是叙述式指令

反面信号（不要拆）：

- 每次调用都要用到——拆了只是增加一次读文件的绕路
- 几十行就能讲完——拆了反而让 SKILL.md 变成一堆指路标

**留指路句的写法**（让 Claude 知道有这个文件、什么时候查）：

```markdown
## 遇到的情况
...（正文主干）

## 细节参考
- 完整字段列表：[reference/fields.md]——需要查某个字段含义时读
- 历史 API 兼容矩阵：[reference/compat.md]——涉及版本迁移时读
```

别写 `See reference/xxx.md for more details`——这种话 Claude 会忽略。直接说"什么时候读"。

### 2.2 什么时候加 scripts/

标志：

- 任务里有**确定性的重复计算**（扫目录、做数据转换、渲染固定格式）
- 每次让 Claude 重写的脚本都差不多
- 输出格式高度结构化（HTML、JSON、特定格式 CSV）

写脚本的好处：
- 确定性——每次输出一样
- 省 token——Claude 不用每次生成几百行代码
- 容易迭代——改一次所有调用都生效

调用写法：

```yaml
---
name: codebase-visualizer
allowed-tools: Bash(python *)
---

运行：

\`\`\`bash
python ${CLAUDE_SKILL_DIR}/scripts/visualize.py .
\`\`\`
```

脚本用 `${CLAUDE_SKILL_DIR}` 引用，避免 Claude 当前目录跑飞。

### 2.3 什么时候加 assets/

- 模板文件（Markdown 骨架、JSON schema、配置样板）
- 示例输入/输出（Claude 可以当 few-shot 参考）
- 二进制素材（图标、字体）

在 SKILL.md 里点明怎么用。

## 3. 写作风格

### 3.1 MUST/NEVER 的节制

看到自己写全大写的硬性词，停一下：

- 能不能改成"为什么这样做"的解释？
- 这个规则是真的"跳过会错"，还是"跳过结果会不一样"？

只在**跳过会真正出错**（数据丢失、权限泄露、流程崩）时保留 MUST。其他情况改成解释式语言：

```markdown
# 不好
NEVER commit without running the tests first.

# 好
提 commit 之前先跑测试——否则 CI 会在别人分支 merge 时暴露问题，返工成本高。
```

### 3.2 命令式而不是被动式

```markdown
# 不好
Tests should be run before commits are made.

# 好
提 commit 之前跑测试。
```

Claude 读命令式比被动式顺——指令清楚，也省字。

### 3.3 原则 > 规则

skill 是要被跑成千上万次的。围着 3 个具体例子列规则，换到第 4 个例子就不适用。把规则往上抽一层：

```markdown
# 太具体（换个例子就失效）
遇到 timeout 错误，等 5 秒再重试，最多 3 次。
遇到 429 错误，按 Retry-After header 等。
遇到 500 错误，exponential backoff。

# 原则式（通用）
区分临时错误和永久错误——临时错误值得重试，永久错误立刻失败。
重试要听 server 的（Retry-After header），没有就 exponential backoff。
重试次数要给上限，避免级联放大。
```

### 3.4 告诉 Claude 为什么

skill 的力量来自"让 Claude 理解"，不是"让 Claude 记住规则"。每写一条指令问自己：

- 为什么这样做？
- 不这样会怎样？

把答案写进指令。这样遇到没预料过的边界情况，Claude 能根据底层原因推理，不用你把所有情况都列出来。

## 4. 子 agent 和 context fork

### 4.1 什么时候用 `context: fork`

适合：
- 任务**独立**，不需要主会话历史（研究、扫码、做报告）
- 想要**干净**的上下文，避免把主会话搞乱
- 任务可能**长跑**，fork 出去跑完返回结果

不适合：
- 需要基于当前对话的 skill（"在我们刚讨论的那个方案里做 X"）——fork 过去子 agent 不知道你们讨论了什么
- 只是给 Claude 一些参考资料的 skill——那种 skill 主上下文用就行，没必要 fork

### 4.2 两种 skill × subagent 模式

| 方向 | 系统 prompt | 任务内容 | 额外载入 |
| --- | --- | --- | --- |
| Skill with `context: fork` | agent 类型的（`Explore`/`Plan` 等） | SKILL.md 正文作为 prompt | CLAUDE.md |
| Subagent with `skills:` 字段 | 子 agent 自己的 markdown body | 主 agent 委派的消息 | 预加载的 skills + CLAUDE.md |

`context: fork` 适合"我想把这段流程作为**任务**派出去"。`skills:` 字段适合"我想做个**角色**，它带着这些参考资料"。

### 4.3 fork skill 模板

```yaml
---
name: deep-research
description: 用户想对某个主题做深度调研时进入——跨仓库找证据、读代码、汇总发现
context: fork
agent: Explore
---

调研 "$ARGUMENTS"：

1. 用 Glob / Grep 找相关文件
2. 读关键文件，理解代码路径
3. 总结发现，引用具体文件和行号
```

fork 过去的子 agent 看到的就是这段正文（替换 `$ARGUMENTS` 后）。注意：
- 这个 skill **必须有明确任务**——如果正文只是"这里有一些 API 约定..."没有动词，子 agent 会困惑，啥也不做就回来
- `agent` 字段决定环境（模型、工具、权限）

## 5. 常见坑

### 5.1 一个 skill 想做好几件事

表现：description 写起来吭哧吭哧、触发得不准、正文像三本不同的手册拼起来。

修：拆成多个 skill。如果确实有"入口 → 分发"的需求，做一个轻的入口 skill，在正文里告诉 Claude "要做 X 用 /skill-a，做 Y 用 /skill-b"。

### 5.2 把 CLAUDE.md 该做的事放进 skill

CLAUDE.md = 每次会话自动加载的事实和约定。skill = 按需触发的 playbook。如果内容是"这个项目的编码规范"、"我们用 pnpm 不是 npm"，那是 CLAUDE.md 的事，不是 skill。skill 应该是**任务导向**的。

### 5.3 为了"完整"塞一堆不用的 frontmatter 字段

每个字段都是理解成本。只写你真正需要的。尤其 `context: fork`、`paths`、`model`——不清楚要不要用就别用。

### 5.4 用 skill 装文档

skill 是让 Claude **做事**的。如果内容是"Vue 的 Composition API 怎么用"，那是技术文档，不是 skill。可以考虑放进 [CLAUDE.md](https://code.claude.com/docs/en/memory) 或独立文档系统。

### 5.5 忘了 description 是外界唯一入口

你知道 skill 是干嘛的——因为你写的它。但 Claude 和其他用户只看 description。每次改完 description 读一遍，假装你从没见过这个 skill，问"我能看出它什么时候该用吗"。看不出就继续改。

## 6. 排错速查

| 症状 | 大概率原因 | 先试什么 |
| --- | --- | --- |
| 该触发没触发 | description 不包含用户实际说法 | 加用户原话到 description |
| 不该触发却触发 | description 太泛 / 关键词冲突 | 加边界说明、排除相邻 skill |
| 触发了行为不对 | 正文指令不清 / 有歧义 / 缺原则 | 先简化正文，移除冗余；然后看是不是某条指令没解释为什么 |
| 第 N 轮后 "失效了" | Skill 加载一次就不重读；Claude 选了别路径 | 把正文改成"原则式"；或考虑 hooks 硬约束 |
| 跑得慢/贵 | 正文过长 / Claude 在写本该是脚本的代码 | 拆 reference/，或加 scripts/ |
| description 被截断 | 有太多 skill，或这条 description 太长 | 往前顶关键词；或设 `SLASH_COMMAND_TOOL_CHAR_BUDGET` |

## 7. 最小自测清单

写完 / 改完 skill，跑这四步：

1. **读 description**，假装你就是 Claude——会拉进来吗？
2. **跑 1 个典型触发场景**——行为和预期一致吗？
3. **跑 1 个邻近但不该触发的场景**——不会误触发吧？
4. **跑 1 个该触发但说法不同的场景**（用户换种说法）——还能命中吗？

4 步都通过，交付。有一步不过，回到相应章节找对应修补模式。
