---
name: yunfan
description: 通过云帆质量管理 MCP 查询和分析项目质量数据。在用户提及云帆、问题单、BUG、线上故障、线上答疑、质量统计、某人处理的问题、阻塞级 BUG、待处理问题、某需求下的问题、问题操作记录，或需从云帆获取质量数据上下文时使用。即使用户没有明确说"云帆"，只要涉及查 BUG、看问题单、了解质量状况、线上问题跟踪等质量管理类问题，都应使用此 skill。
metadata:
  author: duiba
  version: "1.0.0"
---

# 云帆质量管理 MCP

通过云帆 MCP（server 名 `yunfan`）查询项目质量数据的完整指南。提供 5 个只读工具，覆盖问题单列表查询、单条详情（含操作记录）、问题单统计、线上答疑列表、线上答疑统计，支持按人员、时间、状态、类型、严重程度等多维度筛选。

## When to Apply

在以下场景中使用本 skill：
- 查询某人待处理的 BUG 或问题单
- 了解某个需求下有哪些问题
- 查看阻塞级/重大问题的分布
- 统计某空间的质量状况
- 查看线上答疑问题及处理进展
- 查询问题单的操作记录（状态变更历史）
- 了解谁的待处理问题最多

## 工具概览

| 工具 | 用途 | 典型场景 |
|------|------|----------|
| get_question_list | 分页查询问题单列表，支持多维度筛选 | 查某人的 BUG、某需求下的问题、阻塞级问题 |
| get_question_detail | 按 ID 查单条问题单详情（含操作记录） | 查看某个具体问题的完整信息和处理历史 |
| get_question_stats | 问题单统计概览（状态/类型/严重程度/处理人分布） | 了解整体质量状况、谁的待办最多 |
| get_online_list | 分页查询线上答疑列表，支持多维度筛选 | 查线上故障、高优先级未解决问题 |
| get_online_stats | 线上答疑统计概览（状态/类型/优先级/处理人分布） | 了解线上问题整体状况 |

## Quick Reference

### 1. get_question_list 参数

所有参数均可选：

- `spaceId` — 空间 ID，不传则查询所有空间
- `taskId` — 关联需求 ID，查询某个需求下的所有问题
- `name` — 问题名称关键词，模糊搜索
- `state` — 问题状态，逗号分隔（见状态映射表）
- `type` — 问题类型，逗号分隔（见类型映射表）
- `level` — 严重程度，逗号分隔（见严重程度映射表）
- `handler` — 处理人姓名，模糊匹配
- `reporter` — 报告人姓名，模糊匹配
- `createdAfter` / `createdBefore` — 创建时间范围，ISO 日期如 `2026-03-01`
- `current` — 页码，默认 1
- `pageSize` — 每页条数，默认 50，最大 200

### 2. get_question_detail 参数

- `id`（int，必填）— 问题单 ID

返回结构与 get_question_list 中的单条一致，额外包含 `records`（操作记录数组）。

### 3. get_question_stats 参数

- `spaceId` — 空间 ID，可选
- `taskId` — 需求 ID，可选

返回 `total`、`byState`（各状态数量）、`byType`（各类型数量）、`byLevel`（各严重程度数量）、`byHandler`（各处理人**待处理**问题数量）。

### 4. get_online_list 参数

所有参数均可选：

- `spaceId` — 空间 ID，主空间或共享空间均可匹配
- `title` — 问题标题关键词，模糊搜索
- `state` — 问题状态（1=待解决, 2=已解决）
- `type` — 问题类型（1=人员操作问题, 2=技术问题, 3=数据问题, 4=待优化）
- `priority` — 优先级（1=低, 2=中, 3=高）
- `handler` — 处理人姓名，模糊匹配
- `createdAfter` / `createdBefore` — 创建时间范围
- `current` / `pageSize` — 分页

### 5. get_online_stats 参数

- `spaceId` — 空间 ID，可选

返回 `total`、`byState`、`byType`、`byPriority`、`byHandler`（各处理人**待解决**问题数量）。

## 枚举映射表

### 问题单状态（state）

| 参数值 | 含义 |
|--------|------|
| 1 | 待处理 |
| 2 | 已解决 |
| 3 | 已关闭 |
| 4 | 被驳回 |
| 5 | 确认驳回 |

返回数据中 `state` 字段已翻译为中文名称，无需二次转换。

### 问题单类型（type）

| 参数值 | 含义 |
|--------|------|
| 1 | BUG |
| 2 | 优化 |
| 3 | 产品问题 |
| 4 | 线上故障 |

### 问题单严重程度（level）

| 参数值 | 含义 |
|--------|------|
| 0 | 轻微 |
| 1 | 一般 |
| 2 | 重大 |
| 3 | 阻塞 |

### 线上答疑状态（state）

| 参数值 | 含义 |
|--------|------|
| 1 | 待解决 |
| 2 | 已解决 |

### 线上答疑类型（type）

| 参数值 | 含义 |
|--------|------|
| 1 | 人员操作问题 |
| 2 | 技术问题 |
| 3 | 数据问题 |
| 4 | 待优化 |

### 线上答疑优先级（priority）

| 参数值 | 含义 |
|--------|------|
| 1 | 低 |
| 2 | 中 |
| 3 | 高 |

## 返回数据字段

### 问题单字段

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 问题单 ID |
| name | string | 问题概要/名称 |
| type | string | 问题类型（已翻译为中文） |
| state | string | 问题状态（已翻译为中文） |
| level | string | 严重程度（已翻译为中文） |
| handler | string \| null | 处理人姓名 |
| reporter | string \| null | 报告人姓名 |
| spaceId | string | 所属空间 ID |
| spaceText | string | 所属空间名称 |
| taskId | string | 关联需求 ID |
| taskText | string | 关联需求名称 |
| taskUrl | string | 关联需求链接 |
| description | string | 富文本描述 |
| note | string \| null | 备注 |
| reason | string \| null | 原因 |
| expectedAt | string | 截止时间 |
| finishAt | string \| null | 完结时间 |
| createdAt / updatedAt | string | 创建/更新时间 |

**get_question_detail 额外字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| records | array | 操作记录列表 |
| records[].operator | string | 操作人姓名 |
| records[].operationType | string | 操作类型 |
| records[].operationNote | string \| null | 操作备注 |
| records[].currentState | string | 操作后状态（已翻译） |
| records[].beforeState | string | 操作前状态（已翻译） |
| records[].createdAt | string | 操作时间 |

### 线上答疑字段

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 问题 ID |
| title | string | 问题标题 |
| state | string | 状态（已翻译为中文） |
| type | string | 类型（已翻译为中文） |
| priority | string | 优先级（已翻译为中文） |
| spaceId | string | 所属空间 ID |
| proposer | string \| null | 提出人姓名 |
| principal | string \| null | 值班负责人姓名 |
| handler | string[] | 处理人姓名数组 |
| description | string | 问题描述 |
| reason | string | 出现原因 |
| reply | string | 改进措施 |
| createdAt / updatedAt | string | 创建/更新时间 |

## 常见查询模式

### query-person-bugs — 查某人的待处理问题

```
get_question_list({ handler: "张三", state: "1", pageSize: 100 })
```

返回后按 `level` 排序展示（阻塞 > 重大 > 一般 > 轻微）。

### query-task-bugs — 查某需求下的所有问题

```
get_question_list({ taskId: "需求ID", pageSize: 200 })
```

按 `state` 分组展示：待处理 / 已解决 / 已关闭。

### query-blocking — 查阻塞级问题

```
get_question_list({ level: "3", state: "1", pageSize: 100 })
```

阻塞级（level=3）问题通常需要立即关注，展示时突出 `handler` 和 `expectedAt`。

### query-online-urgent — 查高优先级线上问题

```
get_online_list({ priority: "3", state: "1", pageSize: 100 })
```

### query-space-stats — 查某空间质量概览

分两步：
1. `get_question_stats({ spaceId: "空间ID" })` — 获取问题单分布
2. `get_online_stats({ spaceId: "空间ID" })` — 获取线上答疑分布

合并展示，让用户了解整体质量状况。

### query-recent-bugs — 查最近新增的问题

```
get_question_list({ createdAfter: "两周前日期", state: "1", pageSize: 100 })
```

### query-question-history — 查某问题的处理历史

```
get_question_detail({ id: 问题ID })
```

从 `records` 字段中展示状态变更时间线。

## 输出格式

查询结果展示给用户时，根据场景选择合适的格式：

**问题单列表（3 条以上）用表格：**

| ID | 问题名称 | 类型 | 严重程度 | 状态 | 处理人 | 截止日期 |
|----|---------|------|---------|------|--------|---------|
| 123 | 登录页面白屏 | BUG | 阻塞 | 待处理 | 张三 | 2026-04-10 |

- 表格列根据用户关心的维度灵活调整
- 按严重程度降序排列（阻塞 > 重大 > 一般 > 轻微）

**单条问题详情用结构化列表：**
展示完整信息，操作记录以时间线形式展示。

**统计数据用摘要 + 重点数字：**
先给出总结性描述，再列出关键数字。`byHandler` 数据量大时只展示 top N 或用户关心的人。

## 注意事项

- MCP 只提供查询能力，不能写入或修改问题单
- 所有枚举字段（状态、类型、严重程度、优先级）在返回数据中已翻译为中文，展示时直接使用
- 查询结果如果为空，友好地告知用户并建议调整筛选条件
- `handler` / `reporter` 参数支持姓名模糊匹配，无需传 ID
- 问题单的 `handler` 字段存储的是单个处理人；线上答疑的 `handler` 是数组（可多人）
