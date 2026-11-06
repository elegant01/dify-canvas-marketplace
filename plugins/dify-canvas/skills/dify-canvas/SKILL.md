---
name: dify-canvas
description: 读取、修改、发布并试运行 Dify 工作流画布（草稿）。当用户要求改 Dify 应用/工作流里的节点、提示词、代码、变量、连线坐标，或要求发布草稿、在画布上试跑一轮，使用 `dify_canvas_get`/`dify_canvas_patch`/`dify_canvas_publish`/`dify_run`。不用于调用已发布的对外 API、编写 DSL 文件、或 Dify 服务端运维与数据库变更。
---

# Dify 画布操作

通过 `dify-canvas` MCP 操作用户已登录的 Dify 站点。所有读写都走浏览器会话，工具自带备份与校验。

## 先决条件

- 本机有 `node`/`npx` 可用（`dify-canvas-mcp` 由 npx 拉起）。
- 日常 Chrome 已登录目标 Dify 站点。**凭据就是浏览器登录态**：不要向用户索要账号、密码或 API Key，也不要把它们写进任何文件或参数里。

## 建立上下文（一次会话内只做一次）

1. `dify_instances` 看已配置的实例。
2. 没有实例时 `dify_setup { alias, baseUrl }`，只需要别名和站点地址。
3. 需要连浏览器再排障时用 `dify_browser_status`；它只读状态，不触发授权。

通道按 chrome-pool → 已开调试端口的 Chrome → `autoConnect` 连日常 Chrome → 专用 Chrome 的顺序自动解析。**默认复用用户的日常浏览器，不要先推专用窗口，也不要让用户改配置或重启浏览器来绕开连接问题。**只有前三条全部不可用时，才提议 `dify_browser_launch`，并说明那是独立配置目录、需在其中单独登录一次。

## 定位对象

`dify_apps { name }` 按名称模糊查。之后所有工具的 `app` 参数都直接写**应用名称**，不要向用户要 UUID。

## 读

`dify_canvas_get { app }` 返回节点索引（id/标题/类型/备注/坐标）与草稿 `hash`；`nodeId` 只取单个节点的完整定义。

改之前必须先读：节点 id、当前 `data`、以及 `hash` 都要来自这一次读取的结果，不能凭记忆或上一轮的 hash 拼参数。

## 改（默认不落盘）

`dify_canvas_patch { app, changes, apply }`，`changes` 是数组，每项：

- `id`：节点 id
- `data`：字段级补丁，只写要改的字段
- `replaceData`：需要整块替换 `data` 时才用
- `position`：`{x, y}`，只在用户要挪位置时写

流程固定两步：

1. 不带 `apply`（默认 dry-run）→ 检查返回的 `diff`、`warnings`、`blocking`，把改动摘要讲给用户确认。
2. 用户认可后带 `apply: true` **并传 `expectedHash`**（上一步读到的 `hash`）写回。

`blocking` 非空时工具自己拒写（悬空边、自环、`default_value` 类型、Code 节点参数不一致）——把原话转达给用户并修改动，不要试图跳过或改写数据糊弄自检。hash 不符说明草稿被别人动过：重新 `dify_canvas_get` 再改，不要强行覆盖。

写回会自动落盘备份并刷新页面核对渲染节点数。返回里的 `backup` 路径要在汇报里给出；要回滚就说清用它，不要手工反向 diff。

## 发布（单独授权）

`dify_canvas_publish` 立即对线上生效，属于影响共享状态的操作：**必须在用户明确点名要发布之后才能调用**，改完草稿不等于授权发布。发布带 `expectedHash`，发布后核对返回的发布态 hash。

## 试运行

`dify_run { app, query, inputs }` 在**草稿**上跑一轮，不发布。返回工作流状态、实际执行的节点顺序、失败节点与最终回复。排查某个节点时把 `query` 换成能命中该分支的输入，而不是反复跑同一句。

验收口径要讲清楚：草稿试跑的记录只进 workflow-runs，日志页看不到。需要对外可查的留痕，只能发布后用 `/v1` 接口跑。不要把「日志里没有」当成故障。

## 汇报

先给结论（改了哪个应用的哪些节点、是否已写回、是否已发布），再给 `hash` 变化、`backup` 路径、试跑命中的节点链。警告和自检项如实列出，不替用户判断「应该没事」。

## 禁区

- 不要用通用浏览器自动化去手动拖节点、点画布或改 DOM；画布一律走上面的工具。
- 不要在 `data` 里塞入用户没有提供的模型名、API Key、外部接口或业务规则；缺信息就问，不要发明。
- 不要为了「顺手优化」改动用户没点名的节点或连线。一次任务只交付指定改动。
