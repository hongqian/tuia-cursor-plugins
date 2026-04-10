---
name: yuque-mcp
description: >-
  通过语雀 MCP（server 名 user-yuque）操作语雀知识库、文档、团队和目录。在用户提及语雀、知识库、文档管理、写文档、发布文档、搜索文档、目录结构、团队成员、文档统计，或需要读取/创建/更新/删除语雀上的内容时使用。当用户提供 duiba.yuque.com 或 *.yuque.com 域名的链接时也应使用此 skill。即使用户没有明确说"语雀"，只要涉及查文档、写知识库、管理团队文档、查看文档历史版本、知识库统计等文档协作类问题，都应使用此 skill。
---

# 语雀 MCP 使用指南

MCP server 名称：`user-yuque`，所有工具通过 `CallMcpTool` 调用，server 参数固定为 `user-yuque`。

## 语雀链接解析

当用户提供语雀 URL 时，按以下规则解析并调用对应工具：

| URL 格式 | 解析方式 | 对应工具 |
|-----------|----------|----------|
| `https://duiba.yuque.com/{group_login}` | 团队首页 → `group_login` | `get_repos`(owner=group_login, owner_type="group") |
| `https://duiba.yuque.com/{group_login}/{book_slug}` | 知识库页 → namespace=`group_login/book_slug` | `get_repo_detail` / `get_docs` / `get_repo_toc` |
| `https://duiba.yuque.com/{group_login}/{book_slug}/{doc_slug}` | 文档页 → namespace=`group_login/book_slug`, doc_id=`doc_slug` | `get_doc_detail` |

**域名判断规则**：
- `duiba.yuque.com` — 兑吧企业空间，调用时使用 team `duiba`（如已配置）
- `www.yuque.com` — 语雀公共空间，使用默认团队
- 其他 `*.yuque.com` — 对应企业空间，根据子域名匹配已配置的 team

**示例**：用户发送 `https://duiba.yuque.com/vw1kkl/yegutw/geqlyr5qhtgq0izt`，解析为：
- team: `duiba`
- namespace: `vw1kkl/yegutw`
- doc_id: `geqlyr5qhtgq0izt`
- → 调用 `get_doc_detail(team="duiba", namespace="vw1kkl/yegutw", doc_id="geqlyr5qhtgq0izt")`

## 核心概念

| 概念 | 说明 | 标识方式 |
|------|------|----------|
| 团队 (team) | 多团队配置下的团队别名 | 字符串，可选参数，不填用默认团队 |
| 知识库 (repo) | 文档的容器，类似文件夹 | `namespace` = `group_login/book_slug` 或 ID |
| 文档 (doc) | 知识库中的具体文档 | `doc_id`（数字 ID）或 `slug`（路径标识） |
| 目录 (toc) | 知识库内的文档组织结构 | `uuid` 标识节点 |

## 常用工作流

### 1. 查找并阅读文档

```
步骤 1: search → 搜索文档关键词，获取 namespace 和 doc slug
步骤 2: get_doc_detail → 用 namespace + doc_id/slug 获取完整内容
```

### 2. 在知识库中创建文档

```
步骤 1: get_repos → 获取目标知识库的 namespace
步骤 2: create_doc → 在知识库中创建文档（默认 markdown 格式）
步骤 3: get_repo_toc → 查看当前目录结构
步骤 4: update_repo_toc → 将新文档插入目录合适位置
```

### 3. 更新已有文档

```
步骤 1: get_doc_detail → 获取文档当前内容和 doc_id（数字）
步骤 2: update_doc → 用 doc_id（必须是数字）更新文档
```

### 4. 浏览知识库结构

```
步骤 1: get_repos → 列出团队/用户下的所有知识库
步骤 2: get_repo_toc → 获取知识库目录树
步骤 3: get_docs → 获取知识库文档列表
```

## 工具速查

### 用户与团队

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| `hello` | 心跳检测 Token 有效性 | team? |
| `get_user` | 获取当前用户信息 | team? |
| `list_teams` | 列出所有已配置团队 | 无 |
| `get_user_groups` | 获取用户所属团队 | id(用户login/ID), role?, offset? |
| `get_group_members` | 获取团队成员列表 | login(团队login/ID), role?, offset? |
| `update_group_member` | 变更成员角色 | login, user_id, role(0管理员/1成员/2只读) |
| `delete_group_member` | 删除团队成员 | login, user_id |

### 搜索

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| `search` | 搜索文档或知识库 | q(关键词), type("doc"/"repo"), scope?, page?, creator? |

### 知识库

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| `get_repos` | 获取知识库列表 | owner(login), owner_type("group"/"user"), offset? |
| `create_repo` | 创建知识库 | group_login, name, slug, description?, public?(0私密/1公开) |
| `get_repo_detail` | 获取知识库详情 | namespace(ID 或 group_login/book_slug) |
| `update_repo` | 更新知识库 | namespace, name?, slug?, description?, public? |
| `delete_repo` | 删除知识库 | namespace |

### 文档

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| `get_docs` | 获取文档列表 | namespace, offset? |
| `create_doc` | 创建文档 | namespace, title, slug?, body?, format?("markdown"/"lake"/"html"), public?, status?(0草稿/1发布) |
| `get_doc_detail` | 获取文档详情含正文 | namespace, doc_id(ID或slug) |
| `update_doc` | 更新文档 | namespace, doc_id(**必须是数字**), title?, body?, format?, public?, status? |
| `delete_doc` | 删除文档 | namespace, doc_id(**必须是数字**) |
| `get_doc_versions` | 获取文档历史版本 | doc_id(数字) |
| `get_doc_version_detail` | 获取版本详情 | version_id(数字) |

### 目录

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| `get_repo_toc` | 获取知识库目录结构 | namespace |
| `update_repo_toc` | 更新目录节点 | namespace, action, 详见下方 |

**update_repo_toc 的 action 参数：**
- `appendNode` — 尾部插入节点
- `prependNode` — 头部插入节点
- `editNode` — 编辑节点
- `removeNode` — 删除节点

额外参数：`action_mode`("sibling"/"child"), `target_uuid`, `node_uuid`, `doc_ids`(数字数组), `type`("DOC"/"LINK"/"TITLE"), `title`, `url`, `open_window`, `visible`

### 统计

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| `get_statistic_all` | 团队汇总统计 | group_login |
| `get_statistic_by_members` | 成员维度统计 | group_login |
| `get_statistic_by_books` | 知识库维度统计 | group_login |
| `get_statistic_by_docs` | 文档维度统计 | group_login |

## 注意事项

1. **namespace 格式**：`group_login/book_slug`（如 `vw1kkl/yegutw`），也可以用数字 ID
2. **doc_id 类型**：`get_doc_detail` 可用 slug 或数字 ID；`update_doc` 和 `delete_doc` 必须用数字 ID
3. **分页**：列表接口支持 `offset` 参数进行分页
4. **文档格式**：创建/更新文档时 `format` 默认 `markdown`，也支持 `lake`（语雀富文本）和 `html`
5. **文档状态**：`status` 为 0 表示草稿，1 表示已发布
6. **创建文档后记得更新目录**：`create_doc` 只创建文档，不会自动加入目录树，需要调用 `update_repo_toc` 将文档挂到目录中
7. **危险操作确认**：执行 `delete_repo`、`delete_doc`、`delete_group_member` 前应向用户确认

## 调用示例

搜索文档：

```json
{
  "server": "user-yuque",
  "toolName": "search",
  "arguments": { "q": "API设计规范", "type": "doc" }
}
```

获取文档详情：

```json
{
  "server": "user-yuque",
  "toolName": "get_doc_detail",
  "arguments": { "namespace": "vw1kkl/yegutw", "doc_id": "geqlyr5qhtgq0izt" }
}
```

创建文档并发布：

```json
{
  "server": "user-yuque",
  "toolName": "create_doc",
  "arguments": {
    "namespace": "vw1kkl/yegutw",
    "title": "新文档标题",
    "body": "# 标题\n\n文档内容...",
    "format": "markdown",
    "status": 1
  }
}
```
