# dify-canvas-marketplace

Dify 工作流画布插件市场源（CodeBuddy / WorkBuddy 形态），内含插件 `dify-canvas`。

能力本体是 npm 包 [`dify-canvas-mcp`](https://www.npmjs.com/package/dify-canvas-mcp)：读节点、改节点、发布草稿、草稿试跑，全部通过你本机已登录的 Chrome 完成，不存任何 cookie 或 token。

## 1. 任何 MCP 客户端（最省事，不需要插件体系）

只要客户端支持 stdio MCP，加这一段即可 —— Claude Desktop、Cline、Windsurf、Trae、Codex、DSH 等都适用：

```json
{
  "mcpServers": {
    "dify-canvas": {
      "command": "npx",
      "args": ["-y", "dify-canvas-mcp"]
    }
  }
}
```

装完第一句配置实例：

```
dify_setup { alias: "prod", baseUrl: "https://<你的 dify 站点>" }
```

之后按应用名称操作，不需要 UUID。

## 2. CodeBuddy / WorkBuddy

```
/plugin marketplace add elegant01/dify-canvas-marketplace
/plugin install dify-canvas@dify-canvas-marketplace
```

或命令行一次到位：

```
codebuddy plugin marketplace add elegant01/dify-canvas-marketplace
codebuddy plugin install dify-canvas@dify-canvas-marketplace
```

装完执行 `/reload-plugins`，或新开会话。

## 3. Qoder

同一套 Skill + MCP 声明另有 Qoder 插件形态（`.qoder-plugin/plugin.json`），已按 Qoder 的「我的发布」流程提交审核，显示名「Dify 画布编排师」；源码不在本仓库，审核通过后从 Qoder 能力市场安装。

## 提供的工具

| 工具 | 作用 |
| --- | --- |
| `dify_instances` / `dify_setup` | 配置 Dify 实例（别名 + 站点地址） |
| `dify_browser_status` / `dify_browser_launch` | 查看浏览器通道 / 兜底启动专用 Chrome |
| `dify_apps` | 列应用，之后按名称引用 |
| `dify_canvas_get` | 读节点索引与草稿 hash，或读单个节点完整定义 |
| `dify_canvas_patch` | 改节点，默认 dry-run，`apply:true` 才写回 |
| `dify_canvas_publish` | 发布草稿并核对发布态 hash |
| `dify_run` | 在草稿上试运行一轮，返回实际执行的节点链 |

## 前置条件

- 本机 `node` / `npx` 在 PATH 上。
- 日常 Chrome 已登录目标 Dify 站点。凭据即浏览器登录态。
- 连接通道按 chrome-pool → 已开调试端口的 Chrome → autoConnect → 专用 Chrome 自动解析，正常情况零配置、零重启。

## 安全默认

- `dify_canvas_patch` 默认 dry-run，只返回 diff；`apply: true` 才写回，写回前自动备份并自检（悬空边、自环、`default_value` 类型、Code 节点参数一致性）。
- `dify_canvas_publish` 立即上线，需要用户单独授权。
- 备份在 `~/.dify-canvas/backups/`，实例配置在 `~/.dify-canvas/config.json`。

## 已知边界

- 草稿试跑记录只写 workflow-runs，Dify 日志页看不到；要对外留痕需发布后走 `/v1`。
- 只能改草稿里已存在的节点内容；画布级新增/删除节点与连线不在写入范围内。

## 仓库结构

```text
dify-canvas-marketplace/
├── .codebuddy-plugin/marketplace.json
├── LICENSE
├── README.md
└── plugins/dify-canvas/
    ├── .codebuddy-plugin/plugin.json
    ├── .mcp.json
    ├── README.md
    └── skills/dify-canvas/SKILL.md
```

本仓库不实现任何逻辑，只提供清单与 Skill 层规范；改能力请去 `dify-canvas-mcp`。

MIT.
