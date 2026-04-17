# commit message 实例集

SKILL.md 里讲的是原则，这里是具体例子。写 commit 时遇到和某类场景对不上，就翻这里找最近的一条。

所有例子都遵循 `type(scope): summary` 格式，首行 ≤ 72 字符，body 说动机，footer 放 breaking change 和 co-author。

---

## feat —— 新能力

**最短形态**（summary 就够说清）：
```
feat(auth): 支持用邮箱验证码登录
```

**带 body 讲动机**：
```
feat(editor): 新增 Markdown 粘贴时自动转富文本

产品侧统计有 40% 的用户从文档工具粘贴内容进来，原来进来是
纯文本，用户需要重新排版。接 markdown-it 做一次转换解决。
```

**首行带强调标记，配合 breaking change**：
```
feat(api)!: /users 接口改为分页返回

BREAKING CHANGE: /users 不再返回整个数组，调用方需要处理
{items, nextCursor} 结构。默认 pageSize=50。
```

---

## fix —— 修 bug

**修可观测的问题**：
```
fix(upload): 上传大文件时偶发进度条卡在 99%

原来在 onProgress 里用 === total 判完成，遇到分片重传会比
total 多几个字节，进度算出来 >100 被 clamp 回 99，但完成
事件已经错过。改为用 >= 判断。
```

**修边界情况**：
```
fix(date): 跨时区用户生日晚一天显示

原来用 new Date(ISOString) 后取 getDate()，在 UTC-8 的用户
手里会把零点的生日算成前一天。统一用 UTC 方法读取。
```

**反例**（别这么写）：
```
fix: 修复 bug           ← 没说什么 bug、为什么
fix(auth): 改 login.ts  ← 写的是文件名不是原因
fix: 修改了一些问题     ← 等于没写
```

---

## chore —— 杂活

```
chore(deps): 升级 vite 到 5.4，修 dev server 启动慢的问题
```
```
chore: 把 .env.example 里过期的 API key 占位符清掉
```
```
chore(scripts): 统一 package.json 的 script 命名风格
```

chore 的 summary 也要说"为什么"——`chore: 升级依赖`就太糊，加一句原因。

---

## docs —— 只动文档

```
docs(readme): 补充 Windows 下的 node-gyp 安装步骤

多个 issue 反映 Windows 用户跟着 README 装会卡在 native
模块编译，补一段 prerequisite。
```
```
docs(api): 给 useSession 加使用示例和常见坑
```

---

## refactor —— 不改行为的重组

```
refactor(editor): 把 selection 逻辑从组件挪到 store

原来 selection 状态散在三个组件里，跨组件同步靠 props 传，
增加新交互时经常漏改一处。挪到 store 后所有读写走单一入口，
也方便 undo/redo 接进来。

没有行为变化，相关 e2e 用例全绿。
```
```
refactor: 把 utils/ 里的单文件大杂烩拆成按领域分的子目录
```

注意：refactor 的 body 应该明确说"没有行为变化"或"测试覆盖已确认等价"，让 reviewer 放心。

---

## test —— 只动测试

```
test(auth): 补充 session 过期场景的集成测试

之前只覆盖了 happy path，过期 / 刷新失败 / 并发刷新三种
路径没有测试。补齐后覆盖率从 62% → 89%。
```
```
test: 把 flaky 的 e2e 用例加 retry，根因另开 issue 跟进
```

---

## perf —— 以性能为目的

```
perf(list): 首屏列表从一次性渲染改为虚拟滚动

10k 条数据场景下首屏 TTI 从 4.2s 降到 480ms。实现用
@tanstack/virtual，没引入新依赖（已在 tree 里）。
```

perf 的 body 最好带数据——多少数据量、改前改后的耗时。没数据的 perf 容易变成空口优化。

---

## style —— 只动格式

```
style: 跑一次 prettier 统一引号风格
```
```
style(editor): 按 eslint 新规则把 let 改成 const
```

style 不应该夹带任何逻辑改动。如果一次提交既改了格式又改了逻辑，拆成两个 commit。

---

## build —— 构建 / 打包 / 发布

```
build: 切换到 pnpm，workspace 协议替换 file:

原来用 npm + file: 链接本地包，monorepo 里经常出现幽灵
依赖和锁文件漂移。pnpm workspace 强制显式依赖关系。
```
```
build(release): 把 dist 产物从 cjs 改为 esm + cjs 双发布
```

---

## ci —— CI/CD

```
ci: 把 e2e 从 PR check 挪到 merge queue 跑

单次 PR 跑 e2e 要 18 分钟是 PR 阻塞主因。挪到 merge queue
后 PR 上只跑 unit + lint，3 分钟内出结果。
```
```
ci(github): 给 release workflow 加上 provenance attestation
```

---

## 带 Breaking Change 的完整形态

```
feat(config)!: 重命名 config.theme → config.appearance.theme

原来的 config.theme 只管颜色，随着 appearance 下要加
density、fontSize，单一字段不够用。统一到 appearance
命名空间下。

BREAKING CHANGE: 所有读 config.theme 的代码需要改为
config.appearance.theme。本次 release 同时提供迁移脚本
scripts/migrate-config.ts，升级时会自动改用户的 config
文件。
```

关键点：
- summary 里的 `!` 和 footer 的 `BREAKING CHANGE:` 都要有
- footer 里说清楚调用方该怎么改，不只是"这里变了"
- 有迁移路径（脚本、codemod、文档）一并在 body 里指出来

---

## 带 Co-Author 的完整形态

**和同事结对**：
```
feat(search): 支持按时间范围过滤

和 @wangwu 结对做的，他负责后端 query builder，我接前端
UI 和状态。

Co-Authored-By: Wang Wu <wangwu@example.com>
```

**和 Claude / AI 协作**：
```
refactor(store): 把 useUserStore 拆成 auth / profile 两块

原来一个 store 挂了 20+ 个字段，读一个字段全量订阅导致
无谓重渲染。按职责拆分。

Co-Authored-By: Claude <noreply@anthropic.com>
```

**多协作者**：
```
feat(editor): 实现协同光标显示

Co-Authored-By: Zhang San <zhangsan@example.com>
Co-Authored-By: Li Si <lisi@example.com>
Co-Authored-By: Claude <noreply@anthropic.com>
```

注意：
- Co-Authored-By 在 footer 区，body 之后空一行
- 每人独占一行，不要用逗号挤在一起
- 邮箱格式必须是 `<email>`，不是圆括号或别的包法

---

## 几条综合性的反例

**一次 commit 塞了 3 件事**：
```
feat: 加登录 + 修了个 bug + 顺手 prettier
```
拆成三个 commit。atomic commit 是 log 能读的前提。

**type 用歪了**：
```
feat(auth): 修复 login 里的空指针     ← 这是 fix
refactor: 新增了导出 csv 的功能       ← 这是 feat
chore: 重写了整个 upload 模块         ← 不是杂活，是 refactor 或 feat
```

**summary 写成了 changelog 条目**：
```
feat(auth): Added login with email and password support for new users, also fixed...
```
首行要精炼。长内容放 body。英文项目用英文没问题，但不要一句话塞全部。

**体外话 / 闲聊**：
```
fix: 试一下这样能不能行
chore: 不知道为什么，反正就是改了下
```
这种 commit 进了历史就洗不掉。想不清楚就别急着 commit。
