# dify-canvas

通过本机已登录的 Chrome 读取、修改、发布并试运行 Dify 工作流画布草稿。

## 组成

| 组件 | 内容 |
| --- | --- |
| Skill | `skills/dify-canvas/SKILL.md` —— 操作流程、安全闸门、验收口径 |
| MCP | `.mcp.json` 声明 `dify-canvas`（`npx -y dify-canvas-mcp`），由 `plugin.json` 的 `mcpServers` 指向 |

## 提供的工具

| 工具 | 作用 |
| --- | --- |
| `dify_instances` / `dify_setup` | 配置 Dify 实例（别名 + 站点地址） |
| `dify_browser_status` / `dify_browser_launch` | 查看浏览器通道 / 兜底启动专用 Chrome |
| `dify_apps` | 列应用，之后按名称引用，不需要 UUID |
| `dify_canvas_get` | 读节点索引与草稿 hash，或读单个节点完整定义 |
| `dify_canvas_patch` | 改节点，默认 dry-run，`apply:true` 才写回 |
| `dify_canvas_publish` | 发布草稿并核对发布态 hash |
| `dify_run` | 在草稿上试运行一轮，返回实际执行的节点链 |

## 首次使用

```
dify_setup { alias: "prod", baseUrl: "https://<你的 dify 站点>" }
```

连接通道按 chrome-pool → 已开调试端口的 Chrome → autoConnect → 专用 Chrome 顺序自动解析，正常情况零配置、零重启。

## 前置条件

- 本机 `node` / `npx` 可用。
- 日常 Chrome 已登录目标 Dify 站点；凭据即浏览器登录态，插件不存储任何凭证。

## 已知边界

- 草稿试跑记录只写 workflow-runs，Dify 日志页看不到；需要对外留痕要发布后走 `/v1`。
- 只能改草稿里已存在的节点内容；画布级新增/删除节点与连线不在这个 MCP 的写入范围内。
- 需要 Chrome 允许远程调试；首次 autoConnect 会在浏览器里问一次授权，之后复用。

MIT.
