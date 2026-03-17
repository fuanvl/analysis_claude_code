# Learn Claude Code -- 从 0 到 1 构建 nano Claude Code-like agent

[English](./README.md) | [中文](./README-zh.md) | [日本語](./README-ja.md)

## 模型就是 Agent

在讨论代码之前，先把一件事说清楚。

**Agent 是模型。不是框架。不是流程图。不是提示词链。**

"Agent"这个词被绑架了。不知从什么时候起，一整个行业开始相信：把提示词节点、if-else 分支、LLM API 调用用有向无环图串起来，就算是在"构建 Agent"了。不是的。他们做出来的东西是鲁布·戈德堡机械 -- 一个过度工程化的、脆弱的、硬编码规则流水线，LLM 在里面只是一个被美化了的文本补全节点。那不是 Agent。那是一个有着宏大幻想的 shell 脚本。

**直说了吧：提示词流 "Agent" 是不做模型的码农的意淫。** 他们试图通过堆叠过程式逻辑来暴力模拟智能 -- 庞大的 if-else 树、节点图、链式提示词瀑布流 -- 然后祈祷足够多的胶水代码能涌现出自主行为。不会的。你不可能通过工程手段编码出 agency。Agency 是学出来的，不是编出来的。那些系统从诞生之日起就已经死了：脆弱、不可扩展、根本不具备泛化能力。它们是 GOFAI（Good Old-Fashioned AI，经典符号 AI）的现代翻版 -- 几十年前就被学界抛弃的符号规则系统，现在刷了一层 LLM 的漆又登场了。

### Agent 到底是什么

远在 LLM 存在之前，AI 学界就有精确的定义：**Agent 是一个能感知环境、做出决策、采取行动以达成目标的模型。** 重点在 *模型* -- 一个学习到的函数，不是一个脚本化的过程。

历史已经写好了证据：

- **2013 -- DeepMind DQN 玩 Atari。** 一个神经网络，只接收原始像素和游戏分数，学会了 7 款 Atari 2600 游戏 -- 超越了所有先前算法，在其中 3 款上击败了人类专家。到 2015 年，同一架构扩展到 [49 款游戏，达到职业人类测试员水平](https://www.nature.com/articles/nature14236)，论文发表在 *Nature*。没有游戏专属规则。没有决策树。只有一个模型，从经验中学习。那个模型就是 agent。

- **2019 -- OpenAI Five 征服 Dota 2。** 五个神经网络，在 10 个月内与自己对战了 [45,000 年的 Dota 2](https://openai.com/index/openai-five-defeats-dota-2-world-champions/)，在旧金山直播赛上 2-0 击败了 **OG** -- TI8 世界冠军。随后的公开竞技场中，AI 在 42,729 场比赛中胜率 99.4%。没有脚本化的策略。没有元编程的团队协调逻辑。模型完全通过自我对弈学会了团队协作、战术和实时适应。

- **2019 -- DeepMind AlphaStar 制霸星际争霸 II。** AlphaStar 在闭门赛中 [10-1 击败职业选手](https://deepmind.google/blog/alphastar-mastering-the-real-time-strategy-game-starcraft-ii/)，随后在欧洲服务器上达到[宗师段位](https://www.nature.com/articles/d41586-019-03298-6) -- 90,000 名玩家中的前 0.15%。一个信息不完全、实时决策、组合动作空间远超国际象棋和围棋的游戏。Agent 是什么？是模型。训练出来的。不是编出来的。

- **2019 -- 腾讯绝悟统治王者荣耀。** 腾讯 AI Lab 的"绝悟"于 2019 年 8 月 2 日世冠杯半决赛上[以 5v5 击败 KPL 职业选手](https://www.jiemian.com/article/3371171.html)。在 1v1 模式下，职业选手 [15 场只赢 1 场，最多坚持不到 8 分钟](https://developer.aliyun.com/article/851058)。训练强度：一天等于人类 440 年。到 2021 年，绝悟在全英雄池 BO5 上全面超越 KPL 职业选手水准。没有手工编写的英雄克制表。没有脚本化的阵容编排。一个从零开始通过自我对弈学习整个游戏的模型。

每一个里程碑都共享同一个架构：**一个训练好的模型，放进一个环境，给予感知和行动的能力。** "Agent"从来都不是外面那层壳。Agent 永远是模型本身。

### "开发 Agent"的两层含义

当一个人说"我在开发 Agent"时，他只可能是两个意思之一：

1. **训练模型。** 通过强化学习、微调、RLHF 或其他基于梯度的方法调整权重。这是 DeepMind、OpenAI、腾讯 AI Lab、Anthropic 在做的事。这是最本义的 Agent 开发 -- 你在从根本上塑造 Agent 的能力。

2. **构建 Harness（工作环境）。** 编写代码来给模型提供一个可操作的环境 -- 工具（文件读写、Shell、网络）、知识库（产品文档、领域资料）、观测通道（git diff、错误日志、浏览器状态）、行动接口（CLI、API 调用）。这就是本仓库所教的。它有价值、有必要，是真正的工程。但它不是在"构建 Agent"。它是在**构建 Agent 栖居的世界。**

模型做决策。Harness 执行。模型做推理。Harness 提供上下文。模型是飞行员。Harness 是驾驶舱。

这个仓库教你造驾驶舱。好的驾驶舱很重要 -- 它决定了 Agent 能看到什么、能做什么。但永远不要把驾驶舱和飞行员搞混。

```
                    THE AGENT PATTERN
                    =================

    User --> messages[] --> LLM --> response
                                      |
                            stop_reason == "tool_use"?
                           /                          \
                         yes                           no
                          |                             |
                    execute tools                    return text
                    append results
                    loop back -----------------> messages[]


    这是最小循环。每个 AI 编程 Agent 都需要这个循环。
    模型决定何时调用工具、何时停止。
    代码只是执行模型的要求。
```

**12 个递进式课程, 从简单循环到隔离化的自治执行。**
**每个课程添加一个机制。每个机制有一句格言。**

> **s01** &nbsp; *"One loop & Bash is all you need"* &mdash; 一个工具 + 一个循环 = 一个智能体
>
> **s02** &nbsp; *"加一个工具, 只加一个 handler"* &mdash; 循环不用动, 新工具注册进 dispatch map 就行
>
> **s03** &nbsp; *"没有计划的 agent 走哪算哪"* &mdash; 先列步骤再动手, 完成率翻倍
>
> **s04** &nbsp; *"大任务拆小, 每个小任务干净的上下文"* &mdash; 子智能体用独立 messages[], 不污染主对话
>
> **s05** &nbsp; *"用到什么知识, 临时加载什么知识"* &mdash; 通过 tool_result 注入, 不塞 system prompt
>
> **s06** &nbsp; *"上下文总会满, 要有办法腾地方"* &mdash; 三层压缩策略, 换来无限会话
>
> **s07** &nbsp; *"大目标要拆成小任务, 排好序, 记在磁盘上"* &mdash; 文件持久化的任务图, 为多 agent 协作打基础
>
> **s08** &nbsp; *"慢操作丢后台, agent 继续想下一步"* &mdash; 后台线程跑命令, 完成后注入通知
>
> **s09** &nbsp; *"任务太大一个人干不完, 要能分给队友"* &mdash; 持久化队友 + 异步邮箱
>
> **s10** &nbsp; *"队友之间要有统一的沟通规矩"* &mdash; 一个 request-response 模式驱动所有协商
>
> **s11** &nbsp; *"队友自己看看板, 有活就认领"* &mdash; 不需要领导逐个分配, 自组织
>
> **s12** &nbsp; *"各干各的目录, 互不干扰"* &mdash; 任务管目标, worktree 管目录, 按 ID 绑定

---

## 核心模式

```python
def agent_loop(messages):
    while True:
        response = client.messages.create(
            model=MODEL, system=SYSTEM,
            messages=messages, tools=TOOLS,
        )
        messages.append({"role": "assistant",
                         "content": response.content})

        if response.stop_reason != "tool_use":
            return

        results = []
        for block in response.content:
            if block.type == "tool_use":
                output = TOOL_HANDLERS[block.name](**block.input)
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": output,
                })
        messages.append({"role": "user", "content": results})
```

每个课程在这个循环之上叠加一个机制 -- 循环本身始终不变。

## 范围说明 (重要)

本仓库是一个 0->1 的学习型项目，用于从零构建 nano Claude Code-like agent。
为保证学习路径清晰，仓库有意简化或省略了部分生产机制：

- 完整事件 / Hook 总线 (例如 PreToolUse、SessionStart/End、ConfigChange)。  
  s12 仅提供教学用途的最小 append-only 生命周期事件流。
- 基于规则的权限治理与信任流程
- 会话生命周期控制 (resume/fork) 与更完整的 worktree 生命周期控制
- 完整 MCP 运行时细节 (transport/OAuth/资源订阅/轮询)

仓库中的团队 JSONL 邮箱协议是教学实现，不是对任何特定生产内部实现的声明。

## 快速开始

```sh
git clone https://github.com/shareAI-lab/learn-claude-code
cd learn-claude-code
pip install -r requirements.txt
cp .env.example .env   # 编辑 .env 填入你的 ANTHROPIC_API_KEY

python agents/s01_agent_loop.py       # 从这里开始
python agents/s12_worktree_task_isolation.py  # 完整递进终点
python agents/s_full.py               # 总纲: 全部机制合一
```

### Web 平台

交互式可视化、分步动画、源码查看器, 以及每个课程的文档。

```sh
cd web && npm install && npm run dev   # http://localhost:3000
```

## 学习路径

```
第一阶段: 循环                       第二阶段: 规划与知识
==================                   ==============================
s01  Agent 循环              [1]     s03  TodoWrite               [5]
     while + stop_reason                  TodoManager + nag 提醒
     |                                    |
     +-> s02  Tool Use            [4]     s04  子智能体             [5]
              dispatch map: name->handler     每个子智能体独立 messages[]
                                              |
                                         s05  Skills               [5]
                                              SKILL.md 通过 tool_result 注入
                                              |
                                         s06  Context Compact      [5]
                                              三层上下文压缩

第三阶段: 持久化                     第四阶段: 团队
==================                   =====================
s07  任务系统                [8]     s09  智能体团队             [9]
     文件持久化 CRUD + 依赖图             队友 + JSONL 邮箱
     |                                    |
s08  后台任务                [6]     s10  团队协议               [12]
     守护线程 + 通知队列                  关机 + 计划审批 FSM
                                          |
                                     s11  自治智能体             [14]
                                          空闲轮询 + 自动认领
                                     |
                                     s12  Worktree 隔离          [16]
                                          任务协调 + 按需隔离执行通道

                                     [N] = 工具数量
```

## 项目结构

```
learn-claude-code/
|
|-- agents/                        # Python 参考实现 (s01-s12 + s_full 总纲)
|-- docs/{en,zh,ja}/               # 心智模型优先的文档 (3 种语言)
|-- web/                           # 交互式学习平台 (Next.js)
|-- skills/                        # s05 的 Skill 文件
+-- .github/workflows/ci.yml      # CI: 类型检查 + 构建
```

## 文档

心智模型优先: 问题、方案、ASCII 图、最小化代码。
[English](./docs/en/) | [中文](./docs/zh/) | [日本語](./docs/ja/)

| 课程 | 主题 | 格言 |
|------|------|------|
| [s01](./docs/zh/s01-the-agent-loop.md) | Agent 循环 | *One loop & Bash is all you need* |
| [s02](./docs/zh/s02-tool-use.md) | Tool Use | *加一个工具, 只加一个 handler* |
| [s03](./docs/zh/s03-todo-write.md) | TodoWrite | *没有计划的 agent 走哪算哪* |
| [s04](./docs/zh/s04-subagent.md) | 子智能体 | *大任务拆小, 每个小任务干净的上下文* |
| [s05](./docs/zh/s05-skill-loading.md) | Skills | *用到什么知识, 临时加载什么知识* |
| [s06](./docs/zh/s06-context-compact.md) | Context Compact | *上下文总会满, 要有办法腾地方* |
| [s07](./docs/zh/s07-task-system.md) | 任务系统 | *大目标要拆成小任务, 排好序, 记在磁盘上* |
| [s08](./docs/zh/s08-background-tasks.md) | 后台任务 | *慢操作丢后台, agent 继续想下一步* |
| [s09](./docs/zh/s09-agent-teams.md) | 智能体团队 | *任务太大一个人干不完, 要能分给队友* |
| [s10](./docs/zh/s10-team-protocols.md) | 团队协议 | *队友之间要有统一的沟通规矩* |
| [s11](./docs/zh/s11-autonomous-agents.md) | 自治智能体 | *队友自己看看板, 有活就认领* |
| [s12](./docs/zh/s12-worktree-task-isolation.md) | Worktree + 任务隔离 | *各干各的目录, 互不干扰* |

## 学完之后 -- 从理解到落地

12 个课程走完, 你已经从内到外理解了 agent 的工作原理。两种方式把知识变成产品:

### Kode Agent CLI -- 开源 Coding Agent CLI

> `npm i -g @shareai-lab/kode`

支持 Skill & LSP, 适配 Windows, 可接 GLM / MiniMax / DeepSeek 等开放模型。装完即用。

GitHub: **[shareAI-lab/Kode-cli](https://github.com/shareAI-lab/Kode-cli)**

### Kode Agent SDK -- 把 Agent 能力嵌入你的应用

官方 Claude Code Agent SDK 底层与完整 CLI 进程通信 -- 每个并发用户 = 一个终端进程。Kode SDK 是独立库, 无 per-user 进程开销, 可嵌入后端、浏览器插件、嵌入式设备等任意运行时。

GitHub: **[shareAI-lab/Kode-agent-sdk](https://github.com/shareAI-lab/Kode-agent-sdk)**

---

## 姊妹教程: 从*被动临时会话*到*主动常驻助手*

本仓库教的 agent 属于 **用完即走** 型 -- 开终端、给任务、做完关掉, 下次重开是全新会话。Claude Code 就是这种模式。

但 [OpenClaw](https://github.com/openclaw/openclaw) (小龙虾) 证明了另一种可能: 在同样的 agent core 之上, 加两个机制就能让 agent 从"踹一下动一下"变成"自己隔 30 秒醒一次找活干":

- **心跳 (Heartbeat)** -- 每 30 秒系统给 agent 发一条消息, 让它检查有没有事可做。没事就继续睡, 有事立刻行动。
- **定时任务 (Cron)** -- agent 可以给自己安排未来要做的事, 到点自动执行。

再加上 IM 多通道路由 (WhatsApp/Telegram/Slack/Discord 等 13+ 平台)、不清空的上下文记忆、Soul 人格系统, agent 就从一个临时工具变成了始终在线的个人 AI 助手。

**[claw0](https://github.com/shareAI-lab/claw0)** 是我们的姊妹教学仓库, 从零拆解这些机制:

```
claw agent = agent core + heartbeat + cron + IM chat + memory + soul
```

```
learn-claude-code                   claw0
(agent 运行时内核:                   (主动式常驻 AI 助手:
 循环、工具、规划、                    心跳、定时任务、IM 通道、
 团队、worktree 隔离)                  记忆、Soul 人格)
```

## 许可证

MIT

---

**模型就是 Agent。代码是 Harness。搞清楚你在造哪个。**
