# 待办、定时任务与应用

## 待办

用户明确要求写入橡果待办时，按 `oaky agent tools` 目录中 `queryTodos` / `manageTodo` 的 actions 签名查询、新建、修改或完成真实待办。宿主的临时执行计划不等于用户的持久待办。不要因一次创作自动创建待办。

## 定时任务

先区分用户要在橡果运行，还是当前宿主运行。明确指定橡果时使用 `queryScheduledTasks` / `manageScheduledTask`；其他场景按上下文选择，不重复创建两份计划。

外部创建的橡果任务由服务端建立独立任务会话，结果到橡果任务会话查看；它不会在 Codex/Claude Code 当前对话中自动唤醒或续写。创建前说明这一交付位置。
先查已有计划防止重复，确认任务内容、时间与时区；相对日期按用户所在时区理解。创建时遵守 describe 的 schedule 字段和幂等策略，不传或猜 sourceConversationId。

例如，用户已明确要求每天 18 点（上海时区）在橡果整理收藏时，核对当前 schema 后可使用：

```json
{"action":"create","title":"每日整理收藏","prompt":"整理今天收藏的内容，给出简要总结。","schedule":{"kind":"recurring","cron":"0 18 * * *","timezone":"Asia/Shanghai"},"resultDelivery":"task_thread"}
```

写入重试复用原 key。创建后读回真实任务 ID、状态和下次运行时间；不能仅凭提交成功声称计划已执行。

## 已授权应用

`queryApplicationConnections` 发现支持和已连接的应用，`invokeApplication` 按当前服务端合同执行。先读取与任务相关的已安装 Skill/协议和可用性。没有授权时引导用户在橡果完成连接；不要索要应用原始密钥或把橡果 token 当第三方 token。

外部应用的能力由当前授权和服务端支持决定，不能承诺任意网站都可操作。发消息、发布或其他外部写入必须有用户相应授权。

## 微信公众号草稿

用户明确要求把文章同步到自己的公众号草稿箱时，先用 `oaky wechat accounts` 确认已连接且只连接了一个公众号，再运行 `oaky wechat draft sync --title … --markdown 文章.md --cover 封面.png --idempotency-key …`。它只建 / 更新草稿，不发表，也不群发；完成后提醒用户到公众号后台草稿箱检查后自行发表。正文只接受 Markdown，图片必须是本地文件，外部图片 URL 要先下载到本地。草稿已存在时，先问用户，再用同一个 key 加 `--confirm-overwrite`（覆盖草稿）或 `--create-new`（另建一份）重跑。具体参数见 [创作工作流](content-creation-kits/workflow.md)。未连接时引导用户在橡果「设置 → 应用连接 → 微信公众号」完成授权，不要索要公众号密钥。
