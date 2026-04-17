# 过程说明

1. 只读了 `D:\playground\easysdd\.claude\skills\skill-master\SKILL.md` 的前 30 行，拿到当前 description 原文。按用户要求没看同目录其他 reference 文件，也没把 SKILL.md 当 skill 触发、只作为文本素材读。
2. 把 description 拆成三句看它承担的信息：
   - 第一句：总纲（"做 skill 相关工作时进入"）。
   - 第二句：四个具体场景 + 自我优化补充。
   - 第三句：用户原话例句清单。
3. 对照发现第三句的例句和第二句的场景是一一对应的同义重复，是主要冗余。
4. 列出必须保留的触发点：写新 skill、沉淀流程、调优、修 description 不触发/乱触发、自我优化。核查每一个在新版里都有对应表达。
5. 合并同义表述、删掉"从零"等冗余修饰词，把三句压成一句、字数接近减半。
6. 写 proposal.md，明确标出提案版本、保留项、删除项和理由。
7. 没有修改 SKILL.md 本身，只在指定的 outputs 目录下写了这两个文件。
