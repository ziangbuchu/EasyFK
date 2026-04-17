---
name: commit-message
description: 写 git commit message 时进入——把改动整理成符合本项目规范的一行 type(scope): summary，必要时带 body 解释动机、带 footer 标注 breaking change。常见触发：'帮我 commit 一下'、'写个 commit message'、'帮我写提交说明'、'准备提交了'、'这次改动怎么写 commit'。不包括发 PR / 推远端 / 改 git 配置——那些走别的流程。
---

# commit-message

本项目所有 commit 都按这份规范写。目的不是形式主义，是让一年后回来翻 log 的人（大概率是你自己）能一眼看懂"这次为什么要改"。

## 1. 格式

```
type(scope): summary

可选的 body：解释动机、背景、做了什么取舍

可选的 footer：BREAKING CHANGE、Co-Authored-By、issue 关联
```

**首行**：`type(scope): summary`，整行 ≤ 72 字符（含 type、括号、冒号、空格）。

- 中文项目用中文写 summary 和 body。英文项目用英文。不要混写。
- summary 讲清楚"为什么"，不是"改了什么"。文件名、函数名在 diff 里都看得到，log 里要看的是意图。
- 结尾不加句号 / 不加感叹号。

**body**：可选但强烈建议。首行之后空一行再写。段落间空行。没有长度硬限制但单行建议 ≤ 80。

**footer**：每项一行。可以出现 `BREAKING CHANGE: ...`、`Co-Authored-By: ...`、`Refs: #123` 等。

## 2. type 怎么选

只用这十种，别自己造新的：

| type | 用在什么时候 |
| --- | --- |
| `feat` | 新增用户可感知的能力 |
| `fix` | 修复 bug（用户层面看得到的问题） |
| `chore` | 杂活：依赖升级、配置调整、非功能性清理 |
| `docs` | 只动文档（README、注释、指南） |
| `refactor` | 行为不变的代码重组，不是 feat 也不是 fix |
| `test` | 只动测试代码 |
| `perf` | 以性能为目的的改动 |
| `style` | 只动格式 / 空格 / 分号，不改行为 |
| `build` | 构建系统、打包、发布配置 |
| `ci` | CI/CD 流水线 |

判不准的时候问自己：**用户能不能看出这次改动的效果？** 能 → `feat` 或 `fix`；不能 → 从下面挑。

## 3. 关键原则：写"为什么"，不只是"改了什么"

这是整份规范里最容易跑偏的一条。

**反例**（只写了"改了什么"）：
```
fix(auth): 修改 login 函数
```

**正例**（说清"为什么"）：
```
fix(auth): 登录失败时清除缓存的 session，避免用户卡在僵尸登录态
```

body 也服务同一目标。如果 summary 已经说清了，body 可以省；如果需要解释背景、取舍、为什么不是另一种方案，就写 body：

```
refactor(editor): 把 selection 逻辑从组件挪到 store

原来 selection 状态散在三个组件里，跨组件同步靠 props 传，
增加新交互时经常漏改一处。挪到 store 后所有读写走单一入口，
也方便 undo/redo 接进来。
```

写完首行之后停一下，问自己："三个月后只看这一行，我能明白当时为什么这么改吗？" 不能就补 body。

## 4. Breaking change

破坏性改动必须在 footer 明确标注：

```
feat(api): 重构 user 接口的返回结构

把 profile 字段从顶层挪到 user.profile 下，和 team.profile 对齐。

BREAKING CHANGE: 所有消费 /api/user 的调用方需要把 response.profile 改成 response.user.profile。
```

即使 type 是 `refactor` 或 `chore`，只要对外契约变了，都加 `BREAKING CHANGE:` footer。summary 也可以在 type 后加 `!` 强调（`feat(api)!: ...`），但 footer 的描述不能省。

## 5. Co-author（协作者署名）

用官方格式，每人一行：

```
Co-Authored-By: Name <email@example.com>
```

- 位置：footer 区（body 之后空一行）
- 和 Claude / AI 协作的提交按现有约定写，沿用项目里已有的 Co-Authored-By 行即可
- 多个协作者就写多行，不要挤一行

## 6. scope 怎么填

scope 是可选的，但本项目大多数改动都能找到对应模块，建议填。

- 用目录名或模块名（`auth`、`editor`、`api`、`cli`）
- 小写、单数、不加路径分隔符
- 跨多个模块时，要么挑主要的那个，要么省略 scope。不要写 `feat(auth,api,ui):` 这种

实在找不到合适 scope 就省略：`docs: 补充 README 的安装步骤` 是合法的。

## 7. 写完之前自查

提交之前扫一眼：

- [ ] 首行 ≤ 72 字符
- [ ] type 在上面的十个里
- [ ] summary 说了"为什么"而不只是"改了什么"
- [ ] 中文项目用中文、英文项目用英文，没混
- [ ] 有 breaking change 的话 footer 写了 `BREAKING CHANGE:`
- [ ] 协作者署名格式正确

## 8. 参考

需要看更多实例（各种 type、有 body 的、有 breaking 的、多协作者的），去查 [reference/examples.md](reference/examples.md)。正文里列的是骨架，那份文件是肉。遇到自己写的 commit 和某类场景对不上时去翻。
