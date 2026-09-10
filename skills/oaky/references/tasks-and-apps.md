# 待办、定时任务与应用

## 待办

用户明确要求写入橡果待办时，描述 `queryTodos` / `manageTodo`，按当前合同查询、新建、修改或完成真实待办。宿主的临时执行计划不等于用户的持久待办。不要因一次创作自动创建待办。

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
