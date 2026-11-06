# dify-canvas-marketplace

CodeBuddy / WorkBuddy 插件市场源（Git 仓库形态），内含一个插件：`dify-canvas`。

## 安装

在 CLI 里添加本仓库为市场源：

```
/plugin marketplace add elegant01/dify-canvas-marketplace
/plugin install dify-canvas@dify-canvas-marketplace
```

或直接用 CLI 一次性安装本地/Git 包：

```
codebuddy plugin add elegant01/dify-canvas-marketplace
```

## 结构

```text
dify-canvas-marketplace/
├── .codebuddy-plugin/marketplace.json
├── README.md
└── plugins/dify-canvas/
    ├── .codebuddy-plugin/plugin.json
    ├── .mcp.json
    ├── README.md
    └── skills/dify-canvas/SKILL.md
```

## 能力来源

MCP server 本体是 npm 包 [`dify-canvas-mcp`](https://www.npmjs.com/package/dify-canvas-mcp)（`npx -y dify-canvas-mcp`，不锁版本）。本仓库不重复实现任何逻辑，只提供插件清单与 Skill 层的操作规范。

同一套 Skill + MCP 声明也已打成 Qoder 插件（`.qoder-plugin/plugin.json` 形态），两个客户端共用同一个 npm server。

## 前置条件

- 本机 `node` / `npx` 在 PATH 上。
- 日常 Chrome 已登录目标 Dify 站点。凭据即浏览器登录态，不落盘任何 cookie 或 token。

## 安全默认

- `dify_canvas_patch` 默认 dry-run，只返回 diff；`apply: true` 才写回，且写回前自动备份并自检。
- `dify_canvas_publish` 立即上线，需要用户单独授权。
- 备份写在 `~/.dify-canvas/backups/`，实例配置在 `~/.dify-canvas/config.json`。

MIT.
