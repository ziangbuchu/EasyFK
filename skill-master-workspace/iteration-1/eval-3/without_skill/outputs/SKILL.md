---
name: commit-message
description: 写 git commit message 时触发——生成或审阅 commit 标题和正文。覆盖格式 `type(scope): summary`、type 取值、中英文选择、首行长度、body 写法、breaking change 标注、co-author 行等规则。适用于手动提交、脚本化提交、PR 标题起草等所有需要落成 commit 的场景。
---

# Commit Message

## 何时用这个 skill

满足任一即触发：

- 用户让你写一条 commit message 或 PR 标题。
- 你准备调 `git commit`，在起草 message。
- 用户贴了一条 commit message 让你审阅或改写。
- 用户问"这个改动该写什么 commit"。

不要在只是讨论 git 操作（rebase、cherry-pick、reset）而不涉及 message 文本时触发。

## 一分钟速查

标准格式：

```
type(scope): summary

body（可选但建议）

BREAKING CHANGE: ... （仅在有破坏性变更时）
Co-Authored-By: Name <email>
```

硬性规则：

1. **type** 必须是以下之一：`feat` `fix` `chore` `docs` `refactor` `test` `perf` `style` `build` `ci`。见下方 [type 选择](#type-选择)。
2. **scope** 可选，用小写，指代改动的模块/目录/功能域，例：`feat(auth): ...`。
3. **summary** 说清楚"为什么"改，不只是"改了什么"。见 [写动机不是写动作](#写动机不是写动作)。
4. **首行 ≤72 字符**，含 type、scope、冒号、summary 全部。
5. **中文项目用中文，英文项目用英文**。跟项目历史 commit 保持一致；不确定时看 `git log` 最近 20 条。
6. **body 可选但建议**。改动超出"一眼能懂"就写 body，用来解释动机、权衡、副作用。
7. **breaking change** 用 footer 标注：`BREAKING CHANGE: 描述破坏了什么、调用方怎么迁移`。同时在 type 后加 `!`，例 `feat(api)!: ...`。
8. **co-author 用官方格式**：`Co-Authored-By: Name <email>`，空行隔开放 footer 区。

## type 选择

按"用户视角影响"而不是"碰了哪类文件"挑：

| type     | 用在什么时候                                         |
|----------|------------------------------------------------------|
| feat     | 新增用户可见的能力                                   |
| fix      | 修一个 bug（用户视角能感知的错误行为）               |
| perf     | 不改行为只改性能                                     |
| refactor | 不改行为、不改性能，只改代码结构                     |
| docs     | 只动文档（README、注释、指南）                       |
| test     | 只动测试代码（新增、修复、重构测试）                 |
| build    | 改构建系统、依赖、打包配置                           |
| ci       | 改 CI/CD 配置（GitHub Actions、流水线等）            |
| chore    | 以上都不是的杂活（更新 .gitignore、改脚本路径等）    |
| style    | 只改代码格式（空格、分号、lint 自动修复），不含样式表 |

常见误区：

- 改了 CSS 样式是 **feat/fix/refactor**，看视觉影响，不是 `style`。`style` 只指代码格式化。
- 加了一个新函数但没暴露给用户，是 `refactor` 不是 `feat`。
- 修 bug 顺手重构了一段，拆成两个 commit；拆不动就按主要目的选 `fix`。

## 写动机不是写动作

summary 要回答"为什么这次改动值得存在"，而不是"我碰了哪几个文件"。

反例 → 正例：

- `fix: 修改 login.ts` → `fix(auth): 登录时 token 刷新竞争导致用户被登出`
- `feat: 加了 Button 组件 size 属性` → `feat(ui): Button 支持 size 以适配表单紧凑布局`
- `refactor: 重构 user service` → `refactor(user): 拆分 UserService 让查询和写入分别走缓存策略`
- `chore: 更新依赖` → `chore(deps): 升级 vite 到 5.x 以解决本地 HMR 卡死`

判断法：把 summary 念出来，如果听者会问"那又怎样？"，就没写到动机。

## body 什么时候写、怎么写

写 body 的信号：

- 改动背后有非显然的原因或权衡。
- 有副作用、回滚注意事项、临时方案。
- 改了多个文件且它们之间的联系不明显。
- 修了个表面上不相关的 bug（"顺手修了 X 因为它挡住了 Y"）。

不写 body 的信号：

- 纯文档 typo。
- 依赖版本号小升。
- 一个自包含的小重构，标题已经说清楚。

body 结构（没有硬格式，但这样写最省读者时间）：

```
<1-3 句交代动机：之前是什么状态、为什么要改>

<改了什么的要点，只写非显然的部分>

<如果有：副作用 / 注意事项 / 后续 TODO>
```

## Breaking change

任一情况算 breaking：

- 改了公开 API 的签名、返回值、行为。
- 改了配置文件格式、环境变量名。
- 删了公开导出的函数 / 组件 / 命令。
- 默认行为反转。

标注方式（两处都要）：

1. type 后加 `!`：`feat(api)!: 重命名 fetchUser 为 getUser`
2. footer 写 `BREAKING CHANGE:` + 空行上文 + 一句说破坏了什么 + 一句说怎么迁移。

例：

```
feat(api)!: 重命名 fetchUser 为 getUser

统一 service 层命名风格，所有 REST 调用以 get/post/put/delete 开头。

BREAKING CHANGE: fetchUser 已移除。调用方改用 getUser，参数签名保持不变。
```

## Co-author

放在 message 末尾 footer 区，跟 body 之间空一行：

```
Co-Authored-By: Claude <noreply@anthropic.com>
```

多个 co-author 就多写几行，每行一个。不要合并、不要改格式（GitHub 要精确匹配才会识别）。

## 审阅别人的 commit message

收到让你改写的 message 时按这个顺序检查：

1. type 选对了吗？（参考 [type 选择](#type-选择)）
2. summary 写了动机还是只写了动作？
3. 首行超 72 字符了吗？
4. 语言跟项目历史一致吗？
5. 破坏性变更标了吗？
6. body 该有没有？有的话讲清楚了吗？

给出改写版本时保留原作者意图，别夹带你自己的解读。

## 更多示例

各 type 的正反例、复杂场景、难判断情况见 [reference/examples.md](reference/examples.md)。
