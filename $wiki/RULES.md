# LLM Wiki 编译规范

> 这是 LLM 编译和维护 $wiki/ 时需要遵守的规范。

## 目录结构

```
$raw/      ← 原始资料（文章、论文、推文等），只增不改
$wiki/     ← 编译产物（概念文章、实践指南），由 LLM 维护
$output/   ← 分析报告、问答产出、临时文件，可归档回 $wiki/
```

## 文章格式规范

### $wiki/ 文章 frontmatter

```yaml
---
title: 文章标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - $raw/来源文件名.md        # 来自原始资料时填写
  - $wiki/上游文章.md         # 来自 wiki 内部衍生时填写
output_origin: $output/文件名.md  # 从 $output/ 归档时填写（可选）
tags:
  - 概念标签
related:
  - [[其他相关文章]]
---
```

### $output/ 文件 frontmatter

```yaml
---
type: output                  # output（分析报告）| query（问答）| report（调研）
date: YYYY-MM-DD
query: "触发本次输出的问题或指令"
sources:                      # 参考了哪些 wiki 文章或 raw 资料
  - $wiki/文章名.md
archived_to: $wiki/归档文章名.md  # 若已归档，填写目标路径；否则留空
---
```

## $output/ 命名规范

```
YYYY-MM-DD_简短描述.md
```

示例：`2026-04-03_Harness小白实践全流程.md`

## 编译流程（$raw → $wiki）

1. **读取原始资料**：扫描 `$raw/` 下的文件
2. **提取核心概念**：识别文章中的关键概念
3. **检查是否已有对应 Wiki 文章**：查询 `$wiki/index.md`
4. **新增或更新 Wiki 文章**：
   - 新概念 → 在 `$wiki/` 下创建新 `.md` 文章
   - 已有概念 → 追加新信息，不覆盖旧内容
5. **添加 backlinks**：在文章末尾维护 `## 相关资料` 章节
6. **更新 index.md**：同步文章列表和统计数字

## 输出归档流程（$output → $wiki）

触发条件：输出内容满足以下任一条件，应归档进 `$wiki/`：

| 判断标准 | 说明 |
|---------|-----|
| **可复用性** | 未来问类似问题时，这份输出能直接作为答案 |
| **知识密度** | 包含从多个来源综合提炼的新洞见，而非单一来源复述 |
| **实践价值** | 包含可直接操作的步骤、模板、对比表格 |
| **概念澄清** | 厘清了 wiki 中已有文章没有覆盖的概念关系或边界 |

**不需要归档的输出**：单次数据查询、调试中间结果、临时比较、对话性回复。

归档步骤：
1. 在 `$output/` 文件的 frontmatter 填写 `archived_to` 字段
2. 在 `$wiki/` 创建归档文章，frontmatter 填写 `output_origin` 字段
3. 更新 `$wiki/index.md` 统计和文章列表

## Wiki 文章结构

```markdown
# 概念名称

## 核心定义
[简洁的一段话定义]

## 关键要点
- 要点 1
- 要点 2

## 详细内容
[展开说明]

## 相关资料
- [[关联文章1]]
- [[关联文章2]]

## 原始来源
- [来源标题]($raw/文件名.md)
```

## 健康检查项目

- [ ] 检查是否有孤立文章（无 backlinks）
- [ ] 检查是否有概念重复但文章分开的情况
- [ ] 检查 sources 字段是否都能找到对应 $raw 文件
- [ ] 检查 $output/ 下有无值得归档但尚未归档的输出（archived_to 为空）
- [ ] 建议 3-5 个新的值得深入研究的主题
