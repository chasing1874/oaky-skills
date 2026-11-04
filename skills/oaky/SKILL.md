---
name: oaky
description: 把橡果 Oaky 当作个人知识库：收藏与记录、整理标签、回顾关联、基于自己的资料问答和写成稿件；也用于公开资料/社媒研究、沉淀我的方法、管理橡果待办与定时任务、读取已授权应用。用户提到橡果、Oaky 或要用其中内容时使用。
---

# 橡果 Oaky

橡果是用户的个人知识库：用户在这里收藏和记录，整理、回顾并关联这些内容，再基于自己的资料回答问题或写成稿件。本 Skill 让用户在当前 AI 工具中使用这些内容。只操作用户请求涉及的对象；橡果数据与外部宿主的本地文件、记忆、浏览器和任务是不同资源。

## 连接与发现

首次使用或认证失败时读取 [安装与授权](references/setup.md)。已连接时不要重复安装或创建令牌。

1. 使用 `oaky agent capabilities` 确定能力由 `oaky_cli` 还是 `host_native` 承担。
2. 使用 `oaky agent tools` 获取当前业务目录，仅对任务需要的工具运行 `oaky agent tool describe --name <tool>`。
3. 按返回的 inputSchema 构造对象，经 stdin 或请求文件调用 `oaky agent tool invoke --name <tool> --input -`。复杂领域再按需发现单个 Format / operation，不一次加载全部合同。

目录与服务端执行结果优先于本文。工具消失、操作不可用或 schema 变化时刷新相关描述，修正实际请求；不猜旧别名，不尝试内部接口绕过限制。

## 按任务读取

- 记一条文字/链接/图片/语音、查找和读取收藏、标签、回收站、基于自己资料的回顾与问答：[收藏、记录与整理](references/library.md)。
- 把资料和想法写成完整稿件、平台图文，或修改已存稿件：[创作工作流](references/content-creation-kits/workflow.md)。只读取所需 Format 引用；不要把探索性的短回答自动存成作品。
- 用户明确要沉淀或修改自己长期的方法与偏好（写作、学习、工作）：[我的方法与偏好](references/creative-practice-learning/workflow.md)。
- 研究公开网页、平台内容、创作者或评论：[公开资料研究](references/social-data-research/workflow.md)。
- 橡果待办、定时任务、已授权应用：[待办、任务与应用](references/tasks-and-apps.md)。

## 执行与恢复

- 身份由本机 Oaky 授权配置提供。不要把 token 放进 Skill、请求 JSON、命令参数或输出；不要替用户申请管理员权限。
- describe 中的 `idempotency` 决定是否传 `--idempotency-key`。同一逻辑写入因网络中断重试时复用同一个非秘密 key；请求内容或目标变化时换 key。CLI 不会替调用者自动重试写入。
- HTTP/退出码错误和 JSON 中的 `ok:false` 都是失败。先看错误的 `code/path/recovery`，修正后再调用；不能把“请求已提交”当作“任务已完成”。
- 使用返回的真实 ID、revision、Resource Ref 和 opaque continuation。引用图片不会自动把图片送给当前模型；需要检查时使用服务支持的读取/下载路径，再交给宿主自己的图片能力。
- 读取操作不隐式创建创作空间；遇到 `project_required` 时让用户选择/建立空间，不猜 ID。修改作品保留未修改内容与资产引用，并按当前 revision 执行。
- 外部宿主不继承橡果内部会话、附件或云电脑。遇到 `operation_unavailable_for_channel` 时说明可用替代路径；不要伪造内部会话。
- 未经用户请求不发布内容、创建计划、同步全部个人 Skill 或调用付费生成。已明确委托的操作直接按范围执行，不重复索要确认。
