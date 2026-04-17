# bug-hunter — 新 description 提案

## 推荐方案（只用 description 一个字段）

```yaml
description: 用户手里已经有一个"出问题的现象"——报错日志、stack trace、异常表现、复现步骤——想找出根因并定位到具体出错代码时进入。典型触发："这个报错怎么回事"、"帮我看下这个 stack trace"、"XX 功能坏了 / 不对 / 报错了"、"这 bug 从哪来的"、"排查一下这个异常"。不用于写新代码时的顺手挑错或 code review——那些场景有别的 skill 管；只在有明确故障信号、需要定位根因时用。
```

## 备选方案（description + when_to_use 拆分，如果觉得单条太长）

```yaml
description: 从报错日志、stack trace、或异常症状里找 bug 根因并定位到具体代码行。
when_to_use: 用户手里已经有一个故障现象需要排查时用。典型触发："这个报错怎么回事"、"帮我看下这个 stack trace"、"XX 功能坏了 / 不对"、"这 bug 从哪来的"、"排查下这个异常"。不用于写新代码时的顺手挑错或 code review，那些场景不进这个 skill。
```

> 推荐用单条方案。触发语和边界写在一起，Claude 读到时上下文连贯；拆成两字段容易让边界那句话的权重被稀释。
