---
title: Agent Scaling 三个维度
created: 2026-04-03
updated: 2026-04-03
sources:
  - $raw/Harness Engineering 在讨论什么：三个 Scaling 维度的统一框架.md
tags:
  - Harness Engineering
  - Agent
  - Scaling
  - 架构
  - Anthropic
  - OpenAI
  - Cursor
related:
  - [[Harness Engineering]]
---

# Agent Scaling 三个维度

## 核心定义

Harness Engineering 的本质是让 AI 构建软件变得 **scalable**，而 scalability 有三个独立维度。OpenAI、Cursor、Anthropic 在 2026 年 Q1 先后发布的实践报告，各自解了其中一个维度——尽管都使用"harness engineering"这个词，但讨论的工程问题截然不同。

## 三个维度

### 维度 1：时间 Scalability（Anthropic）
**问题：让一个 Agent 连续可靠地跑几小时**

长时间运行引发两类问题：
- **方向漂移**：上下文窗口填满后，模型一致性衰减，偏离原定方向、遗忘早期约束
- **自评失真**：Agent 发现自己产出的缺陷，但随后说服自己这些缺陷可以接受

**解法：三角色架构**
- **Planner**：把一句话需求扩展成完整 product spec，只做高层设计，不进入实现
- **Generator**：按 spec 实现功能
- **Evaluator**：持 sprint contract，用 Playwright 操作真实应用验证，与 Generator 无共享内部状态（独立性是能纠偏的前提）

**Harness 组件生命周期**：每个组件都是对当前模型能力边界的假设，有不同的过期速度。
- Sonnet 4.5 → Opus 4.5 → Opus 4.6 三代模型演化：context reset 先被淘汰，sprint 分解随后淘汰，evaluator 仍有价值
- 正确做法：逐一移除旧组件并测试质量是否真的下降，而非持续叠加

**案例数据**：数字音频工作站，运行约 4 小时，成本 $124，generator 第一轮连续跑 2h7m。对比基线：单 agent 跑 20 分钟花 $9，核心功能无法正常使用。

---

### 维度 2：空间 Scalability（Cursor）
**问题：让几百个 Agent 并行工作，投入 10 倍计算获得 10 倍吞吐量**

**任务**：从零构建 Rust 浏览器引擎，数百 agent 并行运行一周，生成超过 100 万行代码。

**四次架构迭代**：

| 尝试 | 方案 | 失败原因 |
|------|------|---------|
| 第一次 | 全等地位 agent + 共享状态文件 | 持锁太久、行为回避风险；20个agent退化到1-3个吞吐量 |
| 第二次 | Planner/Executor/Worker/Judge 四角色 | 被最慢 Worker 瓶颈 |
| 第三次 | 合并 Planner 和 Executor | 角色过载：随机休眠、停止生成任务 |
| **最终方案** | **递归 Planner-Worker 架构** | ✓ 实现线性扩展 |

**最终方案关键设计**：
- 根 Planner 持有整个项目范围，范围过大时生成子 Planner（递归）
- Worker 在独立 repo 副本上工作，完成后写 handoff（做了什么/发现什么/有何担忧）提交给请求方 Planner
- Worker 之间互不感知，信息严格向上流动
- 移除集中式 Integrator（它会成为瓶颈），接受稳定错误率，让错误被其他 agent 自然修复

**关键发现**：
- 峰值吞吐量约 **1000 commits/hour**
- repo 从 monolith 重构为多个独立 crate 后，编译等待大幅缩短，吞吐量成倍提升
  → 为 Agent 优化的 repo 结构与为人类优化的可能不同

---

### 维度 3：交互 Scalability（OpenAI + Symphony）
**问题：当 Agent 产出速度远超人类注意力时，人怎么用最少介入 steer 大量 Agent**

**Symphony**（2026-03 开源，Elixir/BEAM 构建）：
- 把 Linear 等项目管理工具变成 Agent 的 job scheduler
- Ticket 移到 Todo 状态 → 自动创建独立工作空间（fresh git clone + 隔离 agent session）→ 派 Codex 执行 → 产出 Proof of Work（CI 结果 + walkthrough，有时含录屏）→ 开 PR
- BEAM supervision tree 处理重启和 backoff；可管理数百个并发 implementation run
- 策略通过 repo 内 `WORKFLOW.md` 配置（YAML frontmatter + Liquid 模板化 prompt），随代码版本控制

**人类介入从"写 prompt 并触发"简化为"写 ticket 并移动状态"**：
- 上游：写 ticket + 维护 harness（文档/测试/架构约束）
- 下游：review Proof of Work 和 PR
- 中间：完全自主执行

**OpenAI 对人类注意力的三层解法**：
1. 让 Agent 自我验证（Chrome DevTools Protocol + 独立可观测性栈）
2. 机械化约束取代人工 review（自定义 linter + 修复指引）
3. 自动化熵管理（黄金原则 + 后台 Agent 扫描 + 自动重构 PR）

## 三个维度的依赖关系

```
交互 Scalability
    ↑ 依赖
时间 Scalability + 空间 Scalability
```

- **空间会放大时间的问题**：单个 agent 的方向漂移局限在一个 PR；数百个 agent 同时漂移，错误以并行度倍数积累
- **交互依赖另两者成熟度**：若每个 run 都需要人中途干预，ticket 驱动退化为手动批处理
- **模型适配是跨维度难题**：不同角色需匹配不同模型，且随模型迭代持续变化（如 GPT-5.2 在长自主运行中优于 Opus 4.5）

## 框架使用方法

遇到"harness engineering"讨论时，先问：**它在解决哪个维度的 Scaling？**

- 时间维度 → 关注 evaluator 独立性、sprint 分解、context reset
- 空间维度 → 关注 worker 隔离、递归规划、错误率策略
- 交互维度 → 关注人类介入界面、自动触发机制、Proof of Work 设计

把三个维度混在一起讨论，必然造成混乱。

## 局限性

这三个维度的 Scaling 主要服务于**头部需求**（极复杂系统、大型基础设施）。对更多普通开发者，AI 对软件的更深远影响可能在另一方向：让软件本身更简单、更一次性、更贴合具体用户需求（Generative Kernel 方向）。

## 相关资料

- [[Harness Engineering]]

## 原始来源

- [Harness Engineering 在讨论什么：三个 Scaling 维度的统一框架（2026-03-30）]($raw/Harness%20Engineering%20%E5%9C%A8%E8%AE%A8%E8%AE%BA%E4%BB%80%E4%B9%88%EF%BC%9A%E4%B8%89%E4%B8%AA%20Scaling%20%E7%BB%B4%E5%BA%A6%E7%9A%84%E7%BB%9F%E4%B8%80%E6%A1%86%E6%9E%B6.md)
