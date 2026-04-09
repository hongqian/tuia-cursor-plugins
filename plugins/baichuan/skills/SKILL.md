---
name: baichuan
description: 通过百川需求 MCP 查询和分析项目迭代需求。在用户提及百川、需求列表、迭代版本、排期进度、开发状态、某人负责的需求、本周/上周任务、需求统计、pending 分布、需求 ID、项目进度跟踪，或需从百川获取需求上下文时使用。即使用户没有明确说"百川"，只要涉及查需求、看排期、了解谁在做什么、迭代进展等项目管理类问题，都应使用此 skill。
metadata:
  author: duiba
  version: "1.0.0"
---

# 百川需求 MCP

通过百川 MCP（server 名 `baichuan`）查询项目迭代需求的完整指南。提供 3 个只读工具，覆盖需求列表查询、单条详情、统计概览，支持按人员、时间、阶段、版本等多维度筛选。

## When to Apply

在以下场景中使用本 skill：
- 查询某人的需求或本周/上周迭代任务
- 了解需求当前阶段、排期进度
- 查看需求统计分布（阶段/优先级/人员）
- 按 ID 或名称查看单条需求详情
- 筛选特定状态或时间范围的需求

## 工具概览

| 工具 | 用途 | 典型场景 | 前缀 |
|------|------|----------|------|
| get_task_list | 分页查询需求列表，支持多维度筛选 | 查本周需求、某人的任务、某状态的需求 | `list-` |
| get_task_detail | 按 ID 或名称查单条需求详情 | 查看某个具体需求的完整信息 | `detail-` |
| get_task_stats | 统计概览（阶段/优先级/人员分布） | 了解整体进度、谁的待办最多 | `stats-` |

## Quick Reference

### 1. get_task_list 参数

所有参数均可选：

- `space` — 空间 ID
- `name` — 需求名称关键词，模糊搜索
- `status` — 阶段 ID，逗号分隔（见阶段映射表）
- `version` — 版本数据库 ID（注意：不是展示名），逗号分隔
- `labels` — 标签，逗号分隔
- `user` — 人员姓名（模糊匹配），匹配 owner/pending/frontend/backend/test/client/bigdata 所有角色
- `createdAfter` / `createdBefore` — 创建时间范围，ISO 日期如 `2026-03-01`
- `finishBefore` — 预计发布截止日期
- `current` — 页码，默认 1
- `pageSize` — 每页条数，默认 50，最大 200
- `withUsers` — 是否返回人员姓名，默认 false

查人相关的需求时记得加 `withUsers: true`，否则人员字段只有 userId。

### 2. get_task_detail 参数

- `id`（int）— 需求 ID，优先级高于 name
- `name`（string）— 需求名称，模糊匹配，多条命中时返回候选列表

至少传一个。返回结构与 get_task_list 中的单条一致。

### 3. get_task_stats 参数

- `space` — 空间 ID，可选
- `version` — 版本 ID，逗号分隔，可选

返回 `total`、`byStage`（各阶段数量）、`byPriority`（P0/P1/P2/紧急）、`pendingByUser`（每人待处理数）。

## 阶段 ID 映射

`status` 参数使用数字 ID，返回数据中 `currentStage` 是中文名称数组：

| status ID | 对应阶段 |
|-----------|----------|
| 10 | 待评审 |
| 40 | 前端开发中 |
| 80 | 后端开发中 |
| 100 | 测试中 |
| 110 | 待发布 |
| 120 | 已上线 |

实际存在更多阶段（新需求、待owner确认、前端待排期、后端待排期、测试待排期、大数据待排期、前端待开发、待测试等），但这些没有对应的 status ID，无法通过 `status` 参数直接筛选——需要先查出数据再在结果中过滤 `currentStage`。

## 返回数据字段

每条需求包含以下字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 需求 ID |
| name | string | 需求名称 |
| desc | string | 简要描述 |
| url | string | 语雀文档链接 |
| priority | string | 优先级：P0 / P1 / P2 / 紧急 |
| currentStage | string[] | 当前阶段，如 `["前端开发中"]` |
| space | string[] | 所属空间 |
| version | string | 迭代版本展示名，格式 `MM.DD~MM.DD` |
| creator | string | 创建人 |
| owner | string \| null | 负责人 |
| pending | string[] | 待处理人 |
| frontend / backend / test / client / bigdata | string[] | 各角色人员 |
| cost | string | 工时（人天） |
| labels | string[] | 标签 |
| judgeDate | string \| null | 评审日期 |
| techPlanDate | string \| null | 技术方案日期 |
| startTestDate | string \| null | 提测日期 |
| finishDate | string \| null | 预计发布日期 |
| createdAt / updatedAt | string | 创建/更新时间 |
| techPlanUrl / testCaseUrl | string[] | 技术方案/测试用例链接 |
| pipelineUrl | string \| null | 流水线链接 |

## 常见查询模式

### list-person-week — 查某人本周需求

用户问"本周需求"时，通常既想看本周迭代新建的，也想看手头还在进行中的往期需求。分两步查：

1. 调用 `get_task_list`，加上相关筛选条件（如 `user`），设 `pageSize: 200`、`withUsers: true`
2. 从返回结果中分两组：
   - **本周迭代需求**：`version` 匹配本周展示名（如 `03.23~03.27`）
   - **跨周进行中需求**：`version` 不是本周，但 `currentStage` 不包含"已上线"
3. 两组分别展示，让用户一目了然

百川的迭代周期为**自然周（周一到周五）**，版本展示名格式为 `MM.DD~MM.DD`（如 `03.23~03.27`）。因为我们不知道版本的数据库 ID，所以不能直接用 `version` 参数筛选，正确做法是查出数据后根据返回的 `version` 字段判断属于哪个迭代。

### list-last-week — 查上周需求

计算上周一和上周五，按 `version` 展示名过滤。如果用户没有特别说明，只展示上周版本的需求即可，不需要包含跨周的。

### list-unfinished — 查未完成需求

与"本周需求"相同，重点展示 `currentStage` 不包含"已上线"的所有条目。`status` 参数只能正向匹配，不能排除，所以需要查出结果后在应用层过滤。

### list-recent — 查最近/近期需求

`createdAfter` 设为两周前。

### list-month — 查某月需求

`createdAfter` = 月初，`createdBefore` = 月末。

### stats-overview — 统计概览

调用 `get_task_stats`，从 `byStage` 中排除"已上线"即可得到未上线需求分布。`pendingByUser` 数据量大时只展示 top N 或用户关心的人。

## 输出格式

查询结果展示给用户时，根据场景选择合适的格式：

**需求列表（3 条以上）用表格：**

| ID | 需求名称 | 优先级 | 阶段 | 版本 | 负责人 |
|----|---------|--------|------|------|--------|
| 8593 | DSP管理模块重构 | P1 | 待评审 | 03.23~03.27 | 张宇聪 |

- 表格列根据用户关心的维度灵活调整，不必每次都展示所有字段
- 语雀链接可以放在需求名称上作为超链接

**单条需求详情用结构化列表：**
展示完整信息，包括各角色人员、时间节点、相关链接等。

**统计数据用摘要 + 重点数据：**
先给出总结性描述，再列出关键数字。如果 pendingByUser 数据量大，只展示 top N 或用户关心的人。

## 注意事项

- MCP 只提供查询能力，不能写入或修改需求
- `version` 参数需要数据库 ID（我们通常不知道），所以按迭代筛选时用时间 + version 展示名过滤的方式
- 查询结果如果为空，友好地告知用户并建议调整筛选条件
