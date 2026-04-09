# 兑吧团队 Cursor 插件

兑吧团队内部 Cursor 插件市场，包含以下插件：

| 插件 | 说明 |
|------|------|
| [百川需求管理](./plugins/baichuan/) | 通过百川 MCP 查询和分析项目迭代需求 |
| [云帆质量管理](./plugins/yunfan/) | 通过云帆 MCP 查询和分析项目质量数据 |

## 目录结构

```
duiba-cursor-plugins/
├── .cursor-plugin/
│   └── marketplace.json          # 市场清单
├── plugins/
│   ├── baichuan/                 # 百川需求管理插件
│   │   ├── .cursor-plugin/
│   │   │   └── plugin.json
│   │   ├── skills/
│   │   │   └── SKILL.md
│   │   ├── mcp.json
│   │   └── README.md
│   └── yunfan/                   # 云帆质量管理插件
│       ├── .cursor-plugin/
│       │   └── plugin.json
│       ├── skills/
│       │   └── SKILL.md
│       ├── mcp.json
│       └── README.md
└── README.md
```

## 使用方式

### 团队市场（推荐）

1. 团队管理员在 Cursor Dashboard → Settings → Plugins 中导入本仓库
2. 团队成员在 Cursor 插件市场中安装所需插件

### 手动安装

在 Cursor Agent 聊天框中输入：

```
/add-plugin <本仓库 GitHub 地址>
```

## 新增插件

在 `plugins/` 目录下新建文件夹，参考现有插件结构创建 `.cursor-plugin/plugin.json`、`skills/`、`mcp.json` 等文件。
