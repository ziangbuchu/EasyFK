# Commit Message 示例库

这里按 type 组织。每条给"反例 → 正例 → 为什么"。写 commit 前拿不准时，找最接近的场景参考。

## feat

### 新组件

反例：`feat: 加 Button 组件`
正例：`feat(ui): 新增 Button 组件以替换各页面散落的原生按钮样式`
为什么：说清楚了要解决"样式散落"这个问题，而不是只报告存在这么个组件。

### 新增属性 / 选项

反例：`feat(button): 加 size 属性`
正例：`feat(ui): Button 支持 size 以适配表单紧凑布局`
为什么：触发需求是表单布局，这是"为什么要 size"。

### 新页面 / 新路由

反例：`feat: 加设置页`
正例：`feat(settings): 新增用户设置页承载原来散落在弹窗里的偏好项`

### 新接口

反例：`feat: 加 /api/orders/bulk 接口`
正例：`feat(api): 订单批量创建接口以降低移动端弱网下的重试次数`

## fix

### 功能错误

反例：`fix: 修 login 问题`
正例：`fix(auth): 登录时 token 刷新竞争导致用户被意外登出`
为什么：说清楚了"什么现象 + 什么原因"，回滚或回顾时能定位。

### UI bug

反例：`fix: 修样式`
正例：`fix(ui): 下拉菜单在 Safari 下超出视口被裁掉`

### 边界情况

反例：`fix: 修空数组报错`
正例：`fix(report): 空数据集渲染图表时崩溃，改为显示占位文案`

### 回归

反例：`fix: rollback`
正例：`fix(search): 恢复中文分词，上一版误删了 jieba 初始化`

## perf

反例：`perf: 优化列表`
正例：`perf(list): 虚拟滚动替代全量渲染，10k 条数据首屏由 2.8s 降到 300ms`
为什么：带数字的性能改动最好给基准，没数字也要说是哪种场景下的性能。

反例：`perf: 缓存`
正例：`perf(api): 用户信息请求加本地缓存，首页重复调用从 7 次降到 1 次`

## refactor

反例：`refactor: 改 UserService`
正例：`refactor(user): 拆 UserService 为 Query / Command 两个服务以分离缓存策略`
为什么：重构必须说清楚"为什么结构要这样改"，否则就是无意义搬代码。

反例：`refactor: 提取函数`
正例：`refactor(form): 把校验逻辑抽到 validators/ 以便跨表单复用`

反例：`refactor: 改名`
正例：`refactor(api): 统一 service 方法名为 get/post/put/delete 前缀`

## docs

反例：`docs: 改 README`
正例：`docs(readme): 增加本地开发 HTTPS 证书配置步骤`

反例：`docs: 加注释`
正例：`docs(auth): 注释说明 token 刷新窗口为什么是 5 分钟`
为什么：加注释的价值在于"为什么 5 分钟"，不是"加了注释"。

反例：`docs: typo`
正例：`docs: 修正贡献指南里 pnpm 命令的拼写`
备注：typo 这类无需动机的 commit，summary 本身就是动作也可以，因为动机是"消除阅读干扰"太显然。

## test

反例：`test: 加测试`
正例：`test(auth): 补充 token 刷新竞争场景的回归用例`

反例：`test: 修测试`
正例：`test(e2e): 修复 CI 上因并发端口冲突导致的 flaky`

## build

反例：`build: 升级依赖`
正例：`build(deps): 升级 vite 到 5.x 以解决本地 HMR 卡死`

反例：`build: 改配置`
正例：`build(vite): 开启 sourcemap 以便线上错误栈能映射到源码`

## ci

反例：`ci: 改 workflow`
正例：`ci(release): 发布流水线改用 pnpm cache 把构建耗时从 8min 降到 3min`

反例：`ci: 加 job`
正例：`ci: 新增 lint-commits job 校验 PR 所有 commit 标题符合约定`

## chore

反例：`chore: 杂项`
正例：`chore: 把脚本从 package.json 迁到 scripts/ 目录以便分类`

反例：`chore: 更新 .gitignore`
正例：`chore: .gitignore 忽略 .idea/ 避免 JetBrains 用户误提交`

## style

反例：`style: 格式化`
正例：`style: 应用 prettier 3.x 新规则，统一箭头函数括号`
注意：这里的 style 是代码格式化，不是 CSS 样式。CSS 改动用 feat/fix/refactor。

## 带 body 的完整示例

### feat + 动机 + 权衡

```
feat(upload): 大文件分片上传支持断点续传

后台统计显示 >50MB 的上传失败率 18%，弱网环境下基本没法完成。
引入分片 + 后端 merge 机制，客户端记录 chunk 状态到 IndexedDB。

权衡：
- 小文件（<5MB）不走分片，避免额外请求开销。
- merge 失败的孤儿 chunk 由后端 cron 每日清理。
```

### fix + 根因 + 复现路径

```
fix(auth): 登录时 token 刷新竞争导致用户被意外登出

复现：同时打开两个标签页，A 刷新 token 成功写入新 token，
B 用旧 token 发请求收到 401 后触发登出逻辑。

改用 BroadcastChannel 同步 token 变更，B 在 401 时先读本地最新 token 重试。
```

### refactor + 方向 + 迁移提示

```
refactor(user): 拆 UserService 为 UserQueryService 和 UserCommandService

读写混在一个 service 里，缓存策略没法针对性调：查询要强缓存，
写入要立即失效。拆开后查询走 5min LRU，写入后精确失效相关 key。

调用方迁移：`userService.find*` → `userQueryService.find*`，
`userService.update/create/delete` → `userCommandService.*`。
保留旧 import 路径做软过渡，下个大版本删除。
```

## Breaking change 完整示例

### API 重命名

```
feat(api)!: 重命名 fetchUser 为 getUser

统一 service 层命名：查询用 get，修改用对应动作词。

BREAKING CHANGE: fetchUser 已移除。调用方改用 getUser，参数与返回值不变。
```

### 配置格式变更

```
feat(config)!: 配置文件从 .env 迁到 config/*.ts

类型化配置在 IDE 里能自动补全，避免环境变量名打错。

BREAKING CHANGE: 不再读取 .env 文件。迁移步骤：
1. 把 .env 内容填到 config/local.ts（模板见 config/example.ts）
2. CI 环境变量改用 config/ci.ts 读取 process.env
3. 删除项目根目录 .env 文件
```

### 删除公开导出

```
refactor(utils)!: 移除已废弃的 deepClone，请用 structuredClone

structuredClone 已是浏览器原生 API，Node 17+ 也内置。

BREAKING CHANGE: `import { deepClone } from '@/utils'` 不再可用。
直接换成 structuredClone，语义一致；如需 class 实例深拷贝，
使用 lodash.clonedeep 替代。
```

## 判不准的场景

### 一个 commit 横跨多个 type

原则：按主要目的选 type，次要改动在 body 里提一句。

```
fix(search): 中文分词在包含数字时漏切词

顺手把分词器初始化从组件 mount 时挪到模块 top-level，
之前每次 mount 都重建实例。
```

这个主要是修 bug，顺手的重构在 body 里说明，不拆成两个 commit。

拆不拆 commit 的边界：两个改动有因果关系（A 是为了让 B 能改）就合并；
没因果就拆开。

### 改了 CSS 算什么

- 新增视觉能力（出现新组件、新状态）：`feat`
- 修 UI bug（错位、颜色不对）：`fix`
- 重构样式结构不改视觉：`refactor`
- 只跑 prettier / stylelint 自动修：`style`

### chore 还是 build

- 动了 package.json 的 dependencies / scripts / 构建相关配置：`build`
- 动了 .gitignore、改脚本路径、移动非构建文件：`chore`

### docs 还是 chore

- 动了面向用户 / 贡献者的文档（README、docs/、CONTRIBUTING）：`docs`
- 动了非文档类的元信息（LICENSE、CODEOWNERS）：`chore`

## 语言选择的边缘情况

- 中文项目偶尔出现英文 commit（比如 revert bot 自动生成）：新 commit 继续用中文，不跟风。
- 英文项目的中文贡献者想写中文：不行，跟项目主语言保持一致。
- scope 永远用英文小写（`feat(用户): ...` 不推荐，虽然技术上能工作）。
- 专有名词、库名、命令名保留原样：`feat(ui): 升级到 Vue 3.4`，不要翻译。
