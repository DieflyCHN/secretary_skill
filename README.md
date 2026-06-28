# secretary.skill

本项目是一个本地优先的个人事务参谋 / 秘书系统原型。

它不是一个全自动 Agent，也不是一个试图读取一切、理解一切、执行一切的“全能助手”。它的目标更窄，也更现实：

- 用户主动 push 必要事项；
- 系统保存原始输入；
- 本地 LLM 按协议解析为结构化 note；
- embedding 负责语义检索；
- LLM 基于真实 note 生成日报、项目简报、风险提示；
- 所有事实状态必须可追溯；
- 系统不得擅自删除、完成、归档或执行外部动作。

当前版本是一个最小可运行原型，用于验证：

1. protocol 能否约束 LLM 行为；
2. push 能否稳定生成结构化 note；
3. Ollama embedding 能否召回相关事项；
4. report 能否基于真实 note 生成基本简报；
5. 系统能否避免把没有发生的事情编进报告。


# 1. 项目动机

AI 时代的个人助手不一定要从“全自动执行”开始。

更合理的落地形态可能是：

- 低自治；
- 高可靠；
- 可追溯；
- 本地优先；
- 手动 push；
- 只处理必要信息；
- 以记录、提醒、汇报、轻分析为核心。

现实中的秘书也不需要知道首长知道的一切，更不应该默认读取所有私人资料。用户只需要把必要事项 push 给秘书，秘书负责：

- 记录；
- 整理；
- 分类；
- 提醒；
- 汇报；
- 给出有限的下一步建议。

因此本项目的核心不是“让 AI 替我做一切”，而是构建一个可信的个人事务态势系统。


# 2. 基本架构构想

项目分为三个 Surface。

## Surface0: Human / Input Surface

人通过终端发送指令。

输入端可以分版本：

- V1: CLI 直接输入；
- V2: ChatBot，例如 Telegram / Matrix；
- V3: 专用 App / WebUI / PWA。

输入语言也可以分版本：

- V0: 完全机器友好的格式化语言，例如纯 JSON/YAML；
- V1: 人类可写、机器友好、规则固定的轻格式语言；
- V2: 偏自然语言，由 LLM 解析。

当前推荐采用 V1：

    TYPE: CONTENT ; key=value ; key=value

例如：

    todo: 下周三前完成动物学报告初稿 ; p=high ; project=zoology
    idea: secretary.skill 应采用事件日志作为事实源 ; project=secretary
    wait: 等待课程平台发布动物学报告的正式提交格式 ; p=medium ; project=zoology
    risk: 动物学报告如果不先确定变量、控制组和实验组，后面容易写散 ; p=high ; project=zoology
    note: SSH 远程工作 profile 应该偏向高信息密度和低 GUI 依赖 ; project=linux

这种格式的原则是：

- 不写复杂 JSON；
- 不追求完全自然语言；
- 使用固定死规则；
- 无特殊例外；
- 人类输入成本低；
- LLM 解析成本低。


## Surface1: Secretary / Desk Surface

LLM 扮演“秘书”。

它收到用户 push 后：

1. 读取 protocol；
2. 解析 raw input；
3. 生成结构化 note；
4. 保存到写字桌；
5. 必要时生成报告。

这里要区分两个概念：

- Secretary: LLM 处理器；
- Desk: 当前工作区 / 写字桌。

LLM 不是事实来源。事实来源是保存下来的 raw note 和事件记录。

写字桌包含当前仍然有效的信息，例如：

- active todo；
- waiting item；
- risk；
- stale item；
- 尚未结算的 idea；
- 当前项目上下文。

写字桌不是长期档案，而是当前事务态势。


## Surface2: Cabinet / Archive Surface

档案柜负责长期归档。

完成事项、过期事项、已结算事项可以进入档案柜。

但在当前设想中：

- LLM 可以建议归档；
- 用户或确定性规则确认归档；
- LLM 不应擅自归档；
- 原始输入不得删除。

未来可以从已完成事项中生成：

- 周报；
- 项目复盘；
- 长期知识沉淀；
- 归档索引。


# 3. 核心原则

## 3.1 Raw input 永远保留

用户输入的原始文本必须保留。

LLM 可以添加解释，但不能覆盖原文。

错误设计：

    只保存 LLM 解析后的字段，不保存用户原话。

正确设计：

    raw: "todo: 下周三前完成动物学报告初稿 ; p=high ; project=zoology"

    type: "todo"
    title: "完成动物学报告初稿"
    priority: "high"
    project: "zoology"

即使 LLM 解析错了，也能回到 raw 重新解析。


## 3.2 LLM 不拥有事实

LLM 可以：

- 解析；
- 总结；
- 分类；
- 建议；
- 汇报。

LLM 不可以：

- 删除原始记录；
- 擅自标记完成；
- 擅自归档；
- 擅自执行外部动作；
- 把 protocol 示例当成真实事项；
- 把未检索到的信息编进报告。


## 3.3 记录优先于智能

本项目的 Level0 不是“弱功能”，而是基础功能。

最重要的是：

- 不漏记；
- 不乱改；
- 可检索；
- 可汇报；
- 可追溯。


## 3.4 建议优先于执行

系统初期不追求自动执行。

能力分级：

- Level0: 记录与提醒；
- Level1: 记录 + 优先级 + 风险 + 下一步建议；
- Level2: 生成草稿、准备操作，但需要用户确认；
- Level3: 授权代理，执行有限外部动作。

当前原型处于 Level0 到 Level1 之间。


# 4. 当前实现状态

当前已经实现一个最小可运行原型。

目录结构：

    ~/Applications/secretary_skill/
    ├── protocol/
    │   └── v0.1.md
    ├── data/
    │   ├── notes.jsonl
    │   ├── chunks.jsonl
    │   └── embeddings.npy
    └── scripts/
        └── sec.py

当前命令：

    ./scripts/sec.py index
    ./scripts/sec.py push 'todo: ... ; p=high ; project=...'
    ./scripts/sec.py search "查询内容"
    ./scripts/sec.py report "报告请求"

当前依赖：

- Python venv；
- requests；
- numpy；
- Ollama；
- 一个 embedding 模型；
- 一个本地 chat/instruct 模型。

当前测试中使用：

- embedding 模型: embeddinggemma；
- chat 模型: qwen2.5:7b-instruct。


# 5. 当前工作流程

## 5.1 index

命令：

    ./scripts/sec.py index

作用：

1. 读取 protocol/v0.1.md；
2. 按 section 切块；
3. 读取 data/notes.jsonl；
4. 将 protocol chunks 与 note chunks 一起送入 Ollama embedding；
5. 生成：
   - data/chunks.jsonl
   - data/embeddings.npy

意义：

    protocol 和 notes 都进入语义检索系统。


## 5.2 push

命令示例：

    ./scripts/sec.py push 'todo: 下周三前完成动物学报告初稿 ; p=high ; project=zoology'

流程：

1. 接收 raw input；
2. 使用 embedding 检索相关 protocol；
3. 将 raw input + protocol context 交给 LLM；
4. LLM 输出结构化 JSON；
5. 保存到 notes.jsonl；
6. 重建 embedding index。

生成 note 示例：

    {
      "id": "note_20260426_220948_33cef8",
      "created_at": "2026-04-26T22:09:48+02:00",
      "raw": "todo: 下周三前完成动物学报告初稿 ; p=high ; project=zoology",
      "type": "todo",
      "title": "完成动物学报告初稿",
      "content": "下周三前完成动物学报告初稿",
      "priority": "high",
      "project": "zoology",
      "status": "active",
      "tags": [],
      "deadline_text": "下周三前",
      "deadline": null,
      "uncertainty": []
    }


## 5.3 search

命令示例：

    ./scripts/sec.py search "动物学报告"

流程：

1. 将 query 转成 embedding；
2. 与 chunks.jsonl / embeddings.npy 中的所有 chunk 比相似度；
3. 返回最接近的 protocol 或 note。

测试结果表明：

- "动物学报告" 能召回 zoology 相关 todo / risk / wait；
- "secretary 写字桌 active waiting risk" 能召回 protocol Desk 和 secretary 相关 idea；
- "SSH 远程工作" 能召回 linux / ssh profile 相关 note 和 todo。

这说明 embedding 已经有实际作用。


## 5.4 report

命令示例：

    ./scripts/sec.py report "生成今日日报，列出高优先级事项、风险和下一步动作"

当前逻辑：

1. 用 query 做 embedding 检索；
2. 只检索 note，不再把 protocol 作为事实输入；
3. 将检索到的 notes 交给 LLM；
4. LLM 生成报告。

当前已经修复一个重要问题：

    report 不再把 protocol 示例当成真实事项。

之前曾出现过：

- protocol 里的示例 wait: 等教授回复推荐信；
- protocol 里的示例 risk: 动物学报告如果不先定结构；

被 LLM 当成真实事项写进日报。

修复方式：

    report 阶段只使用 kind="note" 的检索结果作为事实来源。


# 6. 当前 protocol v0.1 设计

当前 protocol 规定：

## Role

系统是本地 secretary skill。

它不拥有事实，只负责记录、解析、存储、检索和汇报。

## Core rule

原始用户输入必须保留。

系统可以添加结构化解释，但不得覆盖或删除原始输入。

## Note types

每条 user push 必须分类为：

- todo: 可执行任务；
- note: 中性记录或事实；
- idea: 未完成构想、设计思路、未来可能工作；
- wait: 等待他人、外部事件或后续条件；
- risk: 潜在问题；
- done: 用户表示某事已经完成；
- update: 用户补充或修改已有 note；
- ask: 用户要求检索、总结或报告。

## Input protocol v1

推荐格式：

    TYPE: CONTENT ; key=value ; key=value

## Priority

- critical: 立即处理；
- high: 重要且时间敏感；
- medium: 相关但不紧急；
- low: 背景项或未来可能性。

## Status

- active: 仍然有效；
- done: 用户确认已完成；
- archived: 进入长期档案；
- stale: 长期未触碰；
- waiting: 被外部条件阻塞。

## Desk

写字桌是当前工作区。

它包含：

- active；
- waiting；
- stale；
- risk-related notes。

## Cabinet

档案柜是长期归档区。

v0.1 中，秘书可以建议归档，但不应静默归档。

## Reporting

日报应包括：

1. urgent / high-priority active tasks；
2. 7 天内 deadline；
3. waiting items；
4. stale items；
5. risks；
6. suggested next actions。

周报应包括：

1. completed items；
2. unresolved carry-over items；
3. important ideas；
4. project-level progress；
5. risks and bottlenecks。


# 7. 当前测试样例

已经 push 的样例包括：

    todo: 下周三前完成动物学报告初稿 ; p=high ; project=zoology

    idea: secretary.skill 应采用事件日志作为事实源 ; project=secretary

    todo: 明天整理动物学报告的文章结构和小标题 ; p=high ; project=zoology

    todo: 本周内阅读 Ouyang 2024 soil warming 论文并摘出实验设计要点 ; p=medium ; project=zoology

    risk: 动物学报告如果不先确定变量、控制组和实验组，后面容易写散 ; p=high ; project=zoology

    wait: 等待课程平台发布动物学报告的正式提交格式 ; p=medium ; project=zoology

    idea: secretary.skill 的写字桌应该只显示 active、waiting、risk 三类事项 ; project=secretary

    todo: 给 secretary.skill 增加 done 命令，完成事项不能删除，只能改变状态 ; p=medium ; project=secretary

    note: SSH 远程工作 profile 应该偏向高信息密度和低 GUI 依赖 ; project=linux

    todo: 给 bash ssh profile 增加常用 alias 和 tmux 启动提示 ; p=low ; project=linux


# 8. 当前表现评价

## 已经成功的部分

### 8.1 基本链路跑通

当前已经实现：

    protocol → embedding
    push → parse → note
    note → embedding
    search → semantic recall
    report → note-based summary

这说明项目的核心技术路线成立。


### 8.2 push 解析基本可用

LLM 能把轻格式输入解析成：

- type；
- title；
- content；
- priority；
- project；
- status；
- deadline_text；
- uncertainty。

大多数样例解析正确。


### 8.3 embedding 检索有效

搜索 "动物学报告" 能召回：

- zoology risk；
- zoology wait；
- zoology todo；
- 相关 protocol。

搜索 "SSH 远程工作" 能召回：

- linux SSH note；
- bash ssh profile todo。

这说明 embedding 足以支撑当前的语义召回。


### 8.4 report 事实边界初步可信

修复后，report 不再把 protocol 示例当事实。

例如没有 done 事项时，它能说：

    当前没有找到任何标记为已完成的任务。

没有 archived 事项时，它能说：

    当前没有归档的事项。


# 9. 当前暴露的问题 / bug

## 9.1 report 不能只靠 embedding

当前日报仍依赖 embedding 检索 note。

问题：

- embedding 可能漏掉某些应该出现的 note；
- 例如已有 wait 事项，但日报可能说“暂无等待事项”；
- 这是因为 wait note 没有被当前 query 召回。

结论：

    embedding 适合 search，不适合作为状态报告的唯一依据。

日报、周报、完成列表、归档列表这类状态报告应由代码硬过滤。


## 9.2 project 边界不能完全靠语义检索

搜索 "secretary 写字桌 active waiting risk" 时，可能召回 zoology risk。

这不是模型错误，因为 query 中有 risk，而 zoology 那条也是 risk。

但对于项目报告来说，这不可接受。

结论：

    project report 必须支持硬过滤 project == secretary / zoology / linux。


## 9.3 done / archived 还没有真正的状态流转

当前没有实现：

    sec done NOTE_ID
    sec archive NOTE_ID
    sec update NOTE_ID
    sec snooze NOTE_ID

因此现在所有事项主要还是 active / waiting。

done 和 archived 目前只是 protocol 中的概念，还不是系统能力。


## 9.4 缺少真正的 Desk / Cabinet 视图

当前还没有：

- desk/todo.md；
- desk/waiting.md；
- desk/risks.md；
- cabinet/weekly archive；
- reports/daily；
- reports/weekly。

当前只有 notes.jsonl 与 embedding index。


## 9.5 日期解析还不可靠

当前 "下周三前" 通常只进入：

    deadline_text: "下周三前"
    deadline: null

后续需要确定性 date resolver。

尤其应考虑：

- 当前时区 Europe/Berlin；
- 今天；
- 明天；
- 下周三；
- 本周内；
- 月底前；
- 具体日期；
- due=YYYY-MM-DD。

不应完全依赖 LLM 解析日期。


## 9.6 next actions 有时过于泛

例如：

    确保按时完成动物学报告初稿，以免影响后续工作进度。

这类建议正确但无操作性。

后续 prompt 应要求：

    next actions must be concrete, physical, and executable.

例如：

- 列出文章结构；
- 明确变量、实验组、控制组；
- 摘出 Ouyang 论文中的实验设计要点；
- 写初稿第一版。


## 9.7 null 字段有时不规范

曾出现：

    "priority": "null"

而不是：

    "priority": null

后续应强化 JSON schema 或输出后做清洗。


## 9.8 uncertainty 字段曾误收确定字段

曾出现：

    "uncertainty": ["p=high"]

这不合理，因为 p=high 是明确字段。

需要在 prompt 中说明：

    p=high, project=..., due=... are explicit fields.
    Do not put them into uncertainty.


# 10. 下一步优化方向

## 10.1 增加确定性 list 命令

应实现：

    sec list
    sec list --status active
    sec list --status waiting
    sec list --status done
    sec list --status archived
    sec list --type risk
    sec list --project zoology
    sec list --priority high

这些不应该靠 LLM。

它们应该直接读取 notes.jsonl 并硬过滤。


## 10.2 增加 today 命令

应实现：

    sec today

它不应该依赖 embedding 检索。

它应直接读取所有 notes，然后选出：

- priority=critical/high；
- status=waiting；
- type=risk；
- deadline within 7 days；
- stale active items；
- active todo。

再交给 LLM 生成日报。

原则：

    代码负责选事实；
    LLM 负责写报告。


## 10.3 增加 project report

应实现：

    sec project zoology
    sec project secretary
    sec project linux

逻辑：

1. 代码筛选 project；
2. 按 type/status/priority 分组；
3. LLM 生成项目简报。

这样可以避免不同 project 串线。


## 10.4 增加 done 命令

命令：

    sec done NOTE_ID

原则：

- 不删除原 note；
- 不覆盖 raw；
- 追加状态变化事件，或生成新状态；
- done 必须由用户确认。

MVP 简化方案：

- 读取 notes.jsonl；
- 生成新的 notes_state.json；
- 或重写 note status 为 done，同时保留 raw 与 updated_at。

更理想方案：

    event sourcing

例如：

    {"event": "created", "note_id": "...", "raw": "..."}
    {"event": "status_changed", "note_id": "...", "from": "active", "to": "done"}

长期应采用事件日志。


## 10.5 增加 archive / settle

命令：

    sec settle suggest
    sec archive NOTE_ID

原则：

- LLM 可以建议归档；
- 用户确认后归档；
- v0.1 不允许静默归档；
- 完成区不等于档案柜。

状态区别：

- done: 用户确认完成；
- archived: 已进入长期档案；
- reported: 已进入日报/周报/复盘。


## 10.6 加入 date resolver

可以先支持简单规则：

- due=YYYY-MM-DD 直接解析；
- 明天；
- 后天；
- 本周内；
- 下周一到下周日；
- 月底前。

时区固定：

    Europe/Berlin

原始 deadline_text 仍然保留。


## 10.7 增加 SQLite 或事件日志

当前 notes.jsonl 可用，但后续更稳的设计是：

    events.jsonl 是事实源
    SQLite 是查询缓存
    notes/*.md 是人类可读视图
    embeddings.npy 是语义索引

原则：

    events.jsonl 不删不改。
    notes/ 可以重建。
    desk/ 可以重建。
    reports/ 可以重建。


# 11. 技术路线

## 当前技术路线

当前采用：

- Python CLI；
- Ollama chat model；
- Ollama embedding model；
- JSONL note storage；
- NumPy 向量相似度；
- protocol markdown；
- RAG-style retrieval；
- LLM report generation。

优点：

- 简单；
- 本地；
- 可控；
- 没有复杂服务；
- 适合 8GB VRAM；
- 易于 debug。

缺点：

- 没有状态机；
- 没有硬过滤；
- 没有 date resolver；
- 没有双向消息；
- 没有长期归档；
- report 仍可能受 embedding 召回影响。


## 后续技术路径 A: CLI-first

优先保持 CLI。

原因：

- 稳定；
- 适合 SSH；
- 适合服务器模式；
- 易于组合 systemd timer；
- 容易 debug。

目标命令：

    sec push
    sec search
    sec list
    sec today
    sec project
    sec done
    sec archive
    sec settle
    sec report


## 后续技术路径 B: systemd timer

用于秘书-人 push。

例如：

- 早上生成日报；
- 晚上生成复盘；
- 每周生成周报；
- SSH login 时显示 brief。

可行形式：

    systemd user timer
    cron
    shell login hook
    notify-send
    Telegram message


## 后续技术路径 C: ChatBot

用于手机端双向通信。

候选：

- Telegram Bot；
- Matrix Bot；
- ntfy；
- Email；
- HTTP webhook。

双向消息格式应保持命令化：

    /push todo: ...
    /done note_...
    /today
    /project zoology
    /snooze note_... 2d

不应一开始做完全自然语言 ChatBot。


## 后续技术路径 D: Obsidian 视图

Obsidian 适合作为展示层，不适合作为唯一事实源。

建议：

    JSONL / events 是事实源
    Obsidian markdown 是 view

可生成：

    Secretary/Today.md
    Secretary/Open Tasks.md
    Secretary/Waiting.md
    Secretary/Risks.md
    Secretary/Projects/zoology.md
    Secretary/Projects/secretary.md


## 后续技术路径 E: 向量库

当前 NumPy 足够。

后续数据量变大后可考虑：

- sqlite-vec；
- Chroma；
- FAISS；
- LanceDB。

但当前阶段不急。


# 12. 设计边界

## 12.1 Search 与 Report 的分工

search 适合语义召回：

    我之前提过动物学报告吗？
    SSH 远程工作相关事项有哪些？
    secretary 写字桌设计有哪些想法？

report 分两类：

### 语义报告

可以用 embedding。

例如：

    生成 secretary 项目设计简报。

### 状态报告

不能只靠 embedding，必须硬过滤。

例如：

    今天有哪些高优先级事项？
    有哪些 waiting？
    有哪些 done？
    有哪些 archived？
    zoology 项目有哪些风险？

状态报告必须由代码读取 notes.jsonl 或事件日志。


## 12.2 Protocol 与 Fact 的分工

protocol 是规则，不是事实。

notes 是事实。

错误：

    从 protocol 示例中提取任务写进日报。

正确：

    protocol 只约束行为；
    report 只根据 notes 写事实。


## 12.3 LLM 与 Code 的分工

LLM 负责：

- 解析；
- 总结；
- 分类；
- 成文；
- 建议；
- 发现可能风险。

代码负责：

- 保存；
- 状态；
- 过滤；
- 排序；
- 权限；
- 时间；
- 文件；
- 执行；
- 审计。

核心原则：

    Code owns state.
    LLM writes interpretation.


# 13. 项目当前定位

当前版本可以称为：

    secretary.skill v0.1-rag-prototype

它不是正式秘书系统，而是一个验证原型。

当前已经验证：

1. protocol 可以写入 embedding index；
2. 用户输入可以结构化为 note；
3. note 可以通过 embedding 召回；
4. LLM 可以基于 note 生成报告；
5. 修复后可以避免 protocol 示例污染事实报告。

当前尚未完成：

1. 确定性状态层；
2. done/archive 状态流转；
3. project/status/priority 硬过滤；
4. 真正的 Desk/Cabinet 文件视图；
5. 日期解析；
6. 定时推送；
7. 双向消息通道；
8. 权限系统；
9. 周报归档。


# 14. 优先级路线图

## P0: 可信状态查询

实现：

    sec list
    sec list --status waiting
    sec list --status done
    sec list --type risk
    sec list --project zoology

目标：

    不靠 LLM，不靠 embedding，直接过滤 notes。


## P1: today

实现：

    sec today

目标：

    确定性选取当前写字桌事项，然后 LLM 生成日报。


## P2: done

实现：

    sec done NOTE_ID

目标：

    事项能从 active 进入 done，但 raw 永远保留。


## P3: project report

实现：

    sec project PROJECT_NAME

目标：

    项目报告不串线。


## P4: archive / settle

实现：

    sec settle suggest
    sec archive NOTE_ID

目标：

    写字桌和档案柜分离。


## P5: date resolver

实现：

    due=YYYY-MM-DD
    明天
    下周三
    本周内
    月底前

目标：

    日报能识别临近截止事项。


## P6: 双向消息

实现：

    Telegram / Matrix / ntfy / HTTP webhook

目标：

    人-秘书 push 与秘书-人 push 闭环。


# 15. 当前项目的一句话总结

secretary.skill 是一个本地优先的个人事务参谋系统。它通过 CLI/ChatBot/App 接收用户主动 push 的事项，将每条输入保存为不可丢失的 note，由本地 LLM 按 protocol 解析为轻结构化数据，并通过 embedding 支持语义检索与报告生成。系统的目标不是自动替用户做事，而是维护一个可信、可追溯、可汇报的个人事务态势图。

当前实现已经跑通最小 RAG 原型，但仍需加入确定性状态层，才能从“能搜索和总结的便签系统”进化为真正的“写字桌 / 档案柜”秘书系统。
