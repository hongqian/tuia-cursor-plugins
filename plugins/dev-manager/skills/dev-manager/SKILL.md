---
name: dev-manager
description: MUST use this skill for development implementation tasks when the project is identified as based on 聚推后台项目模板（React + TypeScript + Vite + Ant Design + ahooks + Tailwind） in README; do not enable for daily conversation.
---

# 后台项目开发 Skill 模板（聚推后台项目模板）

## 1) 需求前置与实施流程

### 1.0 适用判定（强制）

- 一旦判定当前项目确为【聚推后台项目模板】，必须启用本 Skill 的全部流程，不可跳过。
- 若无法确认是否为该模板项目，先向用户确认后再继续。

### 1.1 明确需求

- 必须先检查项目根目录是否存在 `frontend-technical-solution.md`。
- 若不存在，必须先基于用户需求创建该文件，再进入编码阶段。
- 新建方案可包含：需求目标、需求理解、页面与路由设计、接口与数据结构、状态管理、组件拆分、交互与异常处理、实施步骤、风险与待确认项等。具体视需求而定。

### 1.2 方案审核与落地

- 新产出的 `frontend-technical-solution.md` 必须先经过用户审核。
- 在用户明确“通过/确认”前，仅允许继续讨论与修订方案，不进入业务实现。
- 通过审核后的方案文件必须保留在项目根目录，不得删除或挪动到其他目录。

### 1.3 开发阶段

- 若项目根目录缺少 `frontend-technical-solution.md`，必须先基于用户需求创建该文件，再进入后续开发。
- 该文件需先经用户审核通过，后续开发诉求必须严格按此已批准文档执行。
- 用户后续诉求必须严格对齐 `frontend-technical-solution.md` 执行。
- 若用户诉求与方案冲突，先更新或完善方案并获用户确认，再实施代码改动。
- 每次开发前需先核对本次任务对应的方案章节，避免偏离。
- 开发实现时必须同步参考本 Skill 的「2) 代码规范」执行，确保目录、路由、API、UI、自动导入与质量门禁一致。
- 开发完成后必须逐项执行并勾选「3) 执行检查清单」。

### 1.4 需求变更与 Skill 反哺（强制）

- 若用户后续诉求属于“对既有产出内容的修改”，必须先输出三点分析：
  1) 本次修改意图是什么；
  2) 首次产出为何未覆盖该要求；
  3) 如何避免下次遗漏（可执行约束）。
- 同时必须在项目根目录维护 `skills-opt.md`，记录上述 Skill 优化点与改进规则。
- `skills-opt.md` 建议按“日期 / 变更背景 / 漏项原因 / 新增约束 / 后续检查项”结构持续追加。

## 2) 代码规范

### 2.1 目录与文件结构约定

```text
src/
├── apis/           # 接口与请求封装（公共接口可放 apis/common，需先确认）
├── components/     # 业务组件（公共组件可放 components/common，需先确认）
├── constants/      # 常量/枚举
├── hooks/          # 自定义 hooks
├── layouts/        # 布局壳（PageLayout）
├── pages/          # 页面（文件即路由）
├── stores/         # 全局状态（Context + Provider）
├── utils/          # 工具与 request 实例
├── App.tsx
├── main.tsx
└── index.css
```

放置建议：

- **新页面**：`src/pages/<module>/<page>.tsx`
- **业务接口**：`src/apis/<module>/index.ts`
- **业务类型**：`src/apis/<module>/type.ts`
- **页面私有组件**：页面目录内 `components/`
- **通用工具/hook**：`src/utils` / `src/hooks`

文件结构维护原则：

- **单页优先**：原则上简单页面的逻辑保留在页面文件内，不强制拆分。
- **复杂度阈值**：当页面逻辑复杂且页面代码超过 `300` 行时，必须拆分并将可复用/可读性收益明显的部分抽离到业务组件。
- **公共层新增需确认**：若需要新增公共内容（如公共接口、公共组件、自定义 hooks、全局状态、utils），必须先说明新增原因、复用范围和落位方案，并在用户确认后再创建。
- **默认策略**：未获用户确认前，不在 `src/apis/common`、`src/components/common`、`src/hooks`、`src/stores`、`src/utils` 下新增内容。

### 2.2 路由、布局与菜单约定

- 页面路由通常来自 `src/pages` 的文件路由。
- 菜单结构支持两种常见模式：
  - **三层目录模式**：顶部 1 层主菜单 + 左侧展示后 2 层（示例项目默认模式）。
  - **双层目录模式**：取消顶部主菜单，仅保留左侧 2 层结构（适用于模块较少场景）。
- 菜单 key 必须与路由层级保持一致，便于：
  - 顶部菜单高亮
  - 侧栏 active/open keys 计算
  - 权限过滤（`authList`）

菜单示例（关键点：key 与路径一致）：

三层目录模式（顶部 1 层 + 左侧 2 层）：

```tsx
export const originalMenuList: MenuItem[] = [
  {
    key: '/design',
    label: '设计示例',
    children: [
      {
        key: '/design/table',
        label: '表格',
        children: [
          { key: '/design/table/search', label: '搜索项样式' },
          { key: '/design/table/pagination', label: '分页表格' },
        ],
      },
    ],
  },
]
```

双层目录模式（无顶层菜单，仅左侧 2 层）：

```tsx
export const originalMenuList: MenuItem[] = [
  {
    key: '/table',
    label: '表格',
    children: [
      { key: '/table/search', label: '搜索项样式' },
      { key: '/table/pagination', label: '分页表格' },
    ],
  },
]
```

### 2.3 代码示例

以下内容均为示例代码，需尽量维持跟示例代码一致的结构/风格。

#### 2.3.1 分页请求/响应类型示例

具体参数视系统可变。若后端提供的接口文档与当前定义的分页类型参数不同，则向用户发出警告，暂停开发。
```ts
export type TAPaginationReq<T> = T & {
  currentPage: number
  pageSize: number
  orderBy?: string
  order?: 'asc' | 'desc'
}

export interface TAPaginationRes<T = unknown> {
  totalCount: number
  list: T[]
}
```

#### 2.3.2 `requestTableDataWrap` 适配示例

分页表格推荐使用 `useAntdTable`，`requestTableDataWrap`包装函数用于适配接口分页参数与表格分页参数，完成属性转换。如项目中已有的`requestTableDataWrap`的转换逻辑与现有需求不一致，则向用户发出警告，暂停开发。

```ts
export const requestTableDataWrap = <TParams, TResult>(fn) => async (pagingData, formParams) => {
  const { current, pageSize, sorter } = pagingData
  const res = await fn({
    ...formParams,
    currentPage: current,
    pageSize,
    orderBy: sorter?.field,
    order: sorter?.order ? (sorter.order === 'ascend' ? 'asc' : 'desc') : void 0,
  })

  return { total: res.totalCount, list: res.list }
}
```

#### 2.3.3 搜索表单（固定 label 宽度）示例

列表页需要多条件筛选，推荐使用固定 label 宽度写法；如果项目已有同类封装且交互一致，优先复用现有实现。

```css
.ant-form.fixed-label .ant-form-item-label {
  width: var(--labelWidth);
}
```

```tsx
<Form layout='inline' className='fixed-label [--labelWidth:6rem]' onFinish={onSubmit}>
  <Row gutter={[0, 12]} className='w-full'>
    <Form.Item name='xxx' label='标签'>
      <Select className='min-w-[140px]' allowClear showSearch />
    </Form.Item>
    <Space className='ml-auto'>
      <Button onClick={onReset}>重置</Button>
      <Button type='primary' htmlType='submit'>搜索</Button>
    </Space>
  </Row>
</Form>
```

#### 2.3.4 分页表格（`useAntdTable + Table`）示例

标准后台列表页需要服务端分页、快速跳页、每页条数切换时，推荐采用 `useAntdTable + Table` 组合；如项目既有分页封装满足需求，优先复用并保持参数口径一致。

```tsx
const [form] = Form.useForm()
const { tableProps, search } = useAntdTable(getTableData, { defaultPageSize: 10, form })

<SearchForm form={form} onSubmit={search.submit} onReset={search.reset} />
<Table
  rowKey='id'
  {...tableProps}
  pagination={{
    ...tableProps.pagination,
    showTotal: (total) => `共 ${total} 条`,
    pageSizeOptions: [10, 20, 50],
    showSizeChanger: true,
    showQuickJumper: true,
  }}
/>
```

#### 2.3.5 常量与枚举定义示例

业务域需要维护稳定枚举值、展示文案映射与 `Select` 选项时，推荐使用 `as const + Map + enumToOptions` 组合，避免页面内重复硬编码。

```ts
// src/constants/enums/onOff.ts
export const OnOff = {
  OFF: 0,
  ON: 1,
} as const

export type OnOff = EnumValues<typeof OnOff>

export const EnableMap = {
  [OnOff.OFF]: '禁用',
  [OnOff.ON]: '启用',
}

export const EnableOptions = enumToOptions(EnableMap, OnOff)

// 页面中推荐用法（避免手写 options）
<Select options={EnableOptions} />
```

### 2.4 自动导入约定（按项目配置校准）

若 `vite.config.ts` 中已配置 AutoImport 目录，则**无需显式导入**这些目录下的导出内容（默认直接使用）。仅在命名冲突、语义不清或项目有额外约束时，才手动 `import`。

常见 AutoImport 覆盖目录：

- `src/apis/**`
- `src/components/common/**`
- `src/constants/**`
- `src/hooks/**`
- `src/utils/**`
- `src/stores/**`

以及 `react`、`react-router-dom`、部分 `antd` 组件与 `axios`。

### 2.5 代码质量门禁

- ESLint：遵循项目配置（常见 `@tuia/eslint-config`）。
- TypeScript：保持 strict，避免未使用变量/参数。
- 命名：
  - 变量/函数：`camelCase`
  - 组件/类型：`PascalCase`
  - 常量：`UPPER_SNAKE_CASE`（按需）
- 提交规范：`feat|feature|bugfix|refactor|revert|style|chore`

常量与枚举定义规范：

- **目录约定**：业务常量/枚举放 `src/constants/enums/<domain>.ts`。
- **定义方式**：优先使用 `as const` 对象替代 TypeScript `enum`，并派生值类型（如 `type Xxx = EnumValues<typeof Xxx>`）。
- **命名约定**：
  - 枚举对象：`PascalCase`（如 `OnOff`、`Role`）
  - 枚举键：`UPPER_SNAKE_CASE`（如 `ON`、`OFF`）
  - 映射对象：`<EnumName>Map`（如 `OnOffMap`、`RoleMap`）
  - 枚举选项列表: `<EnumName>Options`（如 `OnOffOptions`、`RoleOptions`）
- **展示映射**：涉及 UI 展示时同步维护 Map，并优先通过 `enumToOptions` 生成 `Select` 选项，避免页面内重复硬编码。
- **边界要求**：同一语义的枚举仅维护一份定义；若需新增公共枚举，遵循“公共层新增需确认”规则，先说明再创建。
- 示例参考 `2.3.5 常量与枚举定义示例`。

完成修改后的 lint 建议：

- 优先使用 IDE 保存时 lint（如已开启）或仅对变更文件执行 lint，避免每次全量扫描。
- 提交前可依赖项目已配置的 `lint-staged`（仅检查暂存文件）。
- 仅在以下场景执行全量 lint（`pnpm lint`）：大范围重构、升级依赖、批量改动基础能力。

```bash
# 仅检查并修复已暂存文件（推荐）
npx lint-staged

# 全量检查（按需）
pnpm lint
```

## 3) 执行检查清单

- [ ] 新增文件路径符合目录约定
- [ ] 菜单 key 与路由路径一致
- [ ] 接口类型定义完整，分页参数映射正确
- [ ] 搜索区/表格遵循模板 UI 写法
- [ ] lint 通过，未引入多余变量与无效类型
- [ ] 提交信息 type 合规

## 4) 例外处理

- 当前项目若已有不同约定（目录、请求层、表单/表格封装），以**项目现状优先**。
- 小改动或 bugfix 不强制重构为模板写法，但新增功能尽量向模板统一。
