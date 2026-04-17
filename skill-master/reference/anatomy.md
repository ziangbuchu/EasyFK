# SKILL.md 结构速查

SKILL.md 的字段、语法、lifecycle 细节。需要时查这份，不用整篇记下来。

## 1. Frontmatter 全字段

所有字段都在 `---` 之间的 YAML 里。除 `description` 外都是可选的。

| 字段 | 类型 | 作用 | 注意 |
| --- | --- | --- | --- |
| `name` | string | slash 命令名和显示名 | 小写字母、数字、连字符；≤64 字符；不写就用目录名 |
| `description` | string | Claude 判断要不要触发的依据 | 必写。和 `when_to_use` 合起来最多 1536 字符，超了截断 |
| `when_to_use` | string | description 装不下的补充触发语 | 附加在 description 后面，共享 1536 字符上限 |
| `argument-hint` | string | autocomplete 时显示参数格式 | 比如 `[issue-number]` 或 `[filename] [format]` |
| `disable-model-invocation` | bool | `true` = 只能用户手动 `/` 触发 | 有副作用的 skill（deploy、commit、send-message）默认开 |
| `user-invocable` | bool | `false` = 从 `/` 菜单隐藏，只 Claude 能用 | 背景知识类（"legacy system context"）用这个 |
| `allowed-tools` | string / list | 这个 skill 活跃时，这些工具不问权限 | 空格分隔或 YAML list。其他工具仍走常规权限 |
| `model` | string | 切到指定模型 | 贵或者慢的 skill 可以换模型 |
| `effort` | string | `low` / `medium` / `high` / `xhigh` / `max` | 取决于模型支不支持 |
| `context` | string | `fork` = 在独立子 agent 里跑 | 必须配 `agent` 字段；skill 本身要是"有明确任务"的才有意义 |
| `agent` | string | `context: fork` 时用哪种子 agent | `Explore` / `Plan` / `general-purpose` / 自定义 |
| `hooks` | object | 绑到这个 skill 生命周期的 hooks | 见 https://code.claude.com/docs/en/hooks |
| `paths` | string / list | 只在编辑匹配路径时自动触发 | glob 格式，逗号分隔或 YAML list |
| `shell` | string | `bash`（默认）/ `powershell` | Windows 场景用 PowerShell 要显式开 |

### 两组互动关系

**`disable-model-invocation` + `user-invocable` 的组合**：

| 配置 | 用户能 `/` 触发 | Claude 自动触发 | description 是否常驻上下文 |
| --- | --- | --- | --- |
| 默认 | ✅ | ✅ | ✅ |
| `disable-model-invocation: true` | ✅ | ❌ | ❌（触发才加载） |
| `user-invocable: false` | ❌ | ✅ | ✅ |

**`context: fork` 的代价**：子 agent 没有主会话历史，只看 SKILL.md + CLAUDE.md。适合"给我一个独立任务，你去做完告诉我结果"类 skill，不适合"基于我们现在的讨论做点什么"类 skill。

## 2. 字符替换

在 SKILL.md 正文任何地方都能用，Claude 看到之前就被替换掉。

| 占位符 | 含义 |
| --- | --- |
| `$ARGUMENTS` | 用户触发时跟在 skill 名后的所有内容。没出现这个占位符时会自动追加 `ARGUMENTS: <value>` 到末尾 |
| `$ARGUMENTS[0]` / `$0` | 第一个参数。shell 风格分词，多词用引号包 |
| `$ARGUMENTS[N]` / `$N` | 第 N+1 个参数 |
| `${CLAUDE_SESSION_ID}` | 当前会话 ID |
| `${CLAUDE_SKILL_DIR}` | 当前 skill 的目录。跨目录引用 skill 自带的脚本/文件时用 |

**例子**：`/migrate-component SearchBar React Vue` → `$0=SearchBar`，`$1=React`，`$2=Vue`。

## 3. 预处理语法（`!` 命令）

SKILL.md 被送给 Claude **之前**执行 shell，把输出塞进 prompt。

**行内形式**：
```markdown
PR 差异：!`gh pr diff`
当前分支：!`git branch --show-current`
```

**多行形式**（代码块开头写 `!`）：
````markdown
```!
node --version
git status --short
```
````

注意：
- 这是预处理，不是 Claude 的工具调用。Claude 只看到最终替换后的文本
- 配合 `allowed-tools: Bash(gh *)` 能跳过权限弹窗
- 管理员可以在 settings 里 `"disableSkillShellExecution": true` 全局关掉

## 4. 内容生命周期

一个 skill 被触发时，整份 SKILL.md 作为**一条消息**进入会话，留到结束。Claude Code 不会在后续轮次重读文件。含义：

- SKILL.md 里写的是"整段任务期间都成立的原则"，不是"这一步做完就忘的步骤"
- 改了 SKILL.md 当前会话不会刷新，下次会话才生效
- 会话被 auto-compact 时，最近调用的每个 skill 会保留前 5000 token 再拼到摘要后面；所有保留 skill 共享 25000 token 预算，老的可能被丢
- 觉得 skill "失效了" 通常不是内容丢了，是 Claude 选了别的路径。改法：加强 description、把指令写成原则式、或者用 hooks 硬约束

## 5. 安放位置和优先级

| 层级 | 路径 | 适用范围 |
| --- | --- | --- |
| Enterprise | 管理员配置 | 组织全员 |
| 用户级 | `~/.claude/skills/<name>/` | 这台机器所有项目 |
| 项目级 | `<project>/.claude/skills/<name>/` | 这个仓库 |
| Plugin | `<plugin>/skills/<name>/` | 装了这个插件的地方 |

同名冲突：enterprise > 用户 > 项目。Plugin 用 `<plugin>:<name>` 命名空间，不会冲突。`.claude/commands/<name>.md` 和 `.claude/skills/<name>/SKILL.md` 同名时，skill 优先。

monorepo 下，`.claude/skills/` 支持嵌套——如果你在 `packages/frontend/` 下工作，Claude 会同时加载 `packages/frontend/.claude/skills/` 里的 skill。

## 6. Live change detection

Claude Code 监听 skill 目录。新建、修改、删除一个 skill 的文件**在当前会话内就生效**（前提是目录在会话开始时已经存在）。新建一个之前不存在的顶层 skill 目录需要重启 Claude Code 才能识别。

## 7. 描述为什么会被截断

descriptions 全体被塞进上下文让 Claude 能"看见"所有可用 skill。如果 skill 多到字符预算（默认 8000 字符，或动态的上下文 1%，两者取大），descriptions 会被短截，关键词可能丢。

两个调整方式：
- 设环境变量 `SLASH_COMMAND_TOOL_CHAR_BUDGET` 提高预算
- 从源头压：每条 description + when_to_use 合起来 1536 字符截断，把最关键触发语放前面

## 8. 工具子集

`allowed-tools` 用空格或 YAML list 都行：

```yaml
allowed-tools: Read Grep Bash(git *)
```

或

```yaml
allowed-tools:
  - Read
  - Grep
  - Bash(git *)
```

`Bash(git *)` 表示任何 `git` 开头的命令自动通过。精确匹配写完整命令。**不会**限制 Claude 的工具能力——只是跳过权限弹窗。要阻止某些工具，在 `/permissions` 里加 deny 规则。
