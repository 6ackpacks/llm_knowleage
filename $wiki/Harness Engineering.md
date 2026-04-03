---
title: Harness Engineering
created: 2026-04-03
updated: 2026-04-03
sources:
  - $raw/别再卷模型了，2026 年 Agent 的胜负手在 Harness！给你的Agent 搭操作系统吧.md
  - $raw/工程技术：在智能体优先的世界中利用 Codex.md
  - $raw/Harness Engineering 在讨论什么：三个 Scaling 维度的统一框架.md
tags:
  - Harness Engineering
  - Agent
  - LLM
  - AI工程
  - 架构
related:
  - [[Agent Scaling 三个维度]]
  - [[LLM知识库工作流]]
---

# Harness Engineering

## 核心定义

**Harness（框架/套具）** 是包裹在 LLM 模型外层、让 Agent 能可靠、持续、大规模工作的运行时基础设施——类比于操作系统之于 CPU。没有 Harness，模型再聪明也只是聊天框；有了 Harness，才能让 Agent 连续工作 6+ 小时、并行运行数百个实例、产出百万行生产级代码。

> Phil Schmid（HuggingFace）比喻：模型 = CPU，上下文窗口 = 内存，Agent = 应用程序，**Harness = 操作系统**。

## 三层工程演进

| 层次 | 解决的问题 | 核心技术 |
|------|-----------|---------|
| **Prompt Engineering** | 说什么 | 单次交互指令 |
| **Context Engineering** | 知道什么 | 上下文准备、历史记录、工具描述 |
| **Harness Engineering** | 怎么持续、稳定、大规模干活 | 评估闭环 + 架构约束 + 记忆治理 |

三层叠加，缺任何一层都会出问题。

## 三根支柱

### 支柱 1：评估闭环（Eval Loop）

**核心原则：Agent 不能自己给自己打分。**

Anthropic 称之为"评估驱动开发"（类比 TDD：先定义"做得好"的标准，再让 Agent 工作，由独立评估器打分）。

- 评估器独立于 Generator，用 Playwright 操作真实应用验证产出
- 评估器不只检查 Agent，也在检查"评估本身"是否合理
- 关键数据：CORE-Bench 上 Opus 4.5 初始得分 42%，修复评分 bug + 放宽 scaffold 限制后跳至 **95%**
  → 很多时候不是模型不行，是 Harness 有问题
- Anthropic 用此方案：6 小时、花 200 美元，Agent 完成一个完整游戏

### 支柱 2：架构约束（Architecture Constraints）

**核心原则：约束比指令有效。约束越多，反而越可靠。**

OpenAI Codex 团队实践（5 个月 100 万行代码零手写）：

- **严格分层架构**：Types → Config → Repo → Service → Runtime → UI，每层只能单向依赖
- **机械执行**：通过自定义 linter 和 CI 强制架构不变量，违规直接拒绝，lint 错误信息本身写成 Agent 可理解的修复指引
- linter 本身也由 Codex 生成——Agent 给自己写规矩、自己遵守
- **Repo 即记录系统**：`AGENTS.md` 只做目录（约 100 行），深层知识放 `docs/` 结构化目录；"Codex 看不到的等于不存在"

数据对比：
- LangChain 实验：只改 Harness（不换模型），Terminal Bench 2.0 通过率 52.8% → **66.5%**
- Vercel：删掉 80% 的 Agent 工具，步骤更少、速度更快、效果更好

### 支柱 3：记忆治理（Memory Governance）

**核心问题：多 Agent 共享知识库时，一个 Agent 的幻觉会污染所有 Agent。**

PrismerCloud 的"进化引擎"方案：
- 信号（Signal）→ 基因（Gene）→ 技能涌现（Skill Emergence）
- 只有经过验证确实有效的知识才提升为"基因"
- 3 行 prompt + 记忆系统 ≈ 200 行精心编写的专家 prompt，且前者持续进化

### 补充：熵对抗（Entropy Management）

Agent 系统运行久了会自然腐化（文档过期、架构被绕过、知识库堆积过时信息）。

OpenAI 的做法：
- 将"黄金原则"编码进 repo（如：优先用共享工具包而非手写辅助函数）
- 定期运行后台 Codex 任务扫描偏差，开重构 PR，大多数 1 分钟内审阅并自动合并
- 功能类似垃圾回收：持续还小额债务，而非积累后一次性解决

> OpenAI 原则：**"当 Agent 遇到困难时，把它当作信号：找出缺什么，反馈到代码库，始终由 Codex 自己写修复。"**

## 四条共识（各家收敛点）

1. **人类的核心工作转向设计工作环境**，而非写代码
2. **知识必须版本化、存在于 repo 中**（Google Docs/Slack 里的知识对 Agent 不存在）
3. **约束比指令有效**（约束是确定性的，指令是模糊的）
4. **完美主义是吞吐量的敌人**（纠错比等待便宜；最小阻塞合并策略）

## OpenAI Codex 案例关键数据

- 时间：5 个月（2025-08 至 2026-02）
- 规模：~100 万行代码，~1,500 个 PR，3 名工程师（后扩至 7 名）
- 吞吐量：平均每人每天 3.5 个 PR（团队扩大后吞吐量仍增加）
- 人工：零行手写代码，零直接 code review（全部 agent-to-agent review）
- Agent 单次运行：常超过 6 小时（通常在工程师睡眠时执行）

## 设计原则：Build to Delete

> "竞争优势不再是 prompt，而是你的 Harness 捕获的轨迹。每次 Agent 的成功和失败，都是训练下一代的数据。"
> — Phil Schmid（HuggingFace）

- 今天写的"聪明逻辑"，明天模型升级可能就不需要了
- 架构必须模块化，随时准备撕掉重来
- Manus 6 个月重构了 5 次 Harness；LangChain 1 年内重新架构 3 次

## 主要开源项目

| 项目 | 特点 | 适用场景 |
|------|------|---------|
| LangChain DeepAgents | 规划+文件操作+子Agent+上下文压缩，115k stars | 通用起点，最低门槛 |
| DeerFlow 2.0（字节）| SuperAgent Harness，沙盒+持久记忆+子Agent编排 | 开箱即用多Agent OS |
| OpenHands | SWE-bench 77.6%，模型无关，MIT协议 | 代码Agent + 评估 |
| SWE-agent（Princeton/Stanford）| 评估驱动极致，NeurIPS 2024 | 科学衡量Agent能力 |
| Goose（Block）| 通用on-machine Agent，Apache 2.0 | 非代码任务 |
| PrismerCloud | 记忆治理，进化引擎 | 多Agent共享知识库 |
| Cognee | 知识图谱驱动记忆，6行代码接入 | 语义连接型记忆 |

## 相关资料

- [[Agent Scaling 三个维度]]
- [[LLM知识库工作流]]

## 原始来源

- [别再卷模型了，2026 年 Agent 的胜负手在 Harness！]($raw/%E5%88%AB%E5%86%8D%E5%8D%B7%E6%A8%A1%E5%9E%8B%E4%BA%86%EF%BC%8C2026%20%E5%B9%B4%20Agent%20%E7%9A%84%E8%83%9C%E8%B4%9F%E6%89%8B%E5%9C%A8%20Harness%EF%BC%81%E7%BB%99%E4%BD%A0%E7%9A%84Agent%20%E6%90%AD%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%90%A7.md)
- [工程技术：在智能体优先的世界中利用 Codex（Ryan Lopopolo, OpenAI, 2026-02-11）]($raw/%E5%B7%A5%E7%A8%8B%E6%8A%80%E6%9C%AF%EF%BC%9A%E5%9C%A8%E6%99%BA%E8%83%BD%E4%BD%93%E4%BC%98%E5%85%88%E7%9A%84%E4%B8%96%E7%95%8C%E4%B8%AD%E5%88%A9%E7%94%A8%20Codex.md)
- [Harness Engineering 在讨论什么：三个 Scaling 维度的统一框架（2026-03-30）]($raw/Harness%20Engineering%20%E5%9C%A8%E8%AE%A8%E8%AE%BA%E4%BB%80%E4%B9%88%EF%BC%9A%E4%B8%89%E4%B8%AA%20Scaling%20%E7%BB%B4%E5%BA%A6%E7%9A%84%E7%BB%9F%E4%B8%80%E6%A1%86%E6%9E%B6.md)
