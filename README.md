# Oaky Skill

在 Codex、Claude Code 中把橡果当作你的个人知识库：收藏与记录、整理、回顾关联、基于自己的资料问答并写成稿件，也能做公开资料研究、管理待办和定时任务。

这是标准 [Agent Skills](https://agentskills.io/specification) 目录，不是 npm 包。`skills/oaky/SKILL.md` 是入口，`references/` 提供按需加载的工作流。版本：**0.2.2**。

## 安装 Skill

把下面这句话发给你的 AI 工具：

> 请安装 https://github.com/chasing1874/oaky-skills/tree/v0.2.2/skills/oaky 中的 Oaky Skill，只安装到当前宿主。保留已有同名 Skill 的本地修改。然后读取 references/setup.md，检查执行依赖，并指导我从橡果设置里的「橡果 Skill」获取授权。不要在安装期间创建内容、定时任务或运行付费生成。

也可以使用社区 [skills 安装器](https://github.com/vercel-labs/skills)，选择当前宿主的一条命令：

```bash
# Codex
npx skills add chasing1874/oaky-skills --skill oaky --global --agent codex
# Claude Code
npx skills add chasing1874/oaky-skills --skill oaky --global --agent claude-code
```

`npx skills` 是可选的社区安装工具，不是标准要求。安装前检查已有同名 Skill，不直接覆盖本地修改。

不用 Node.js 安装 Skill 也可以：下载 [Release ZIP](https://github.com/chasing1874/oaky-skills/releases/download/v0.2.2/oaky-skill-0.2.2.zip)，把完整的 `oaky/` 目录放到：

| 宿主 | 用户技能目录 |
| --- | --- |
| Codex | `~/.agents/skills/oaky/` |
| Claude Code | `~/.claude/skills/oaky/` |

安装器或 ZIP 只负责 Skill 文件。宿主未发现时刷新技能或重启会话。其他宿主需同时支持本地命令、Node.js 和网络访问；不承诺 Claude 网页版等受限沙箱自动兼容。

## 连接橡果

安装后让 AI 按 `references/setup.md` 检查执行依赖：Node.js 22+ 和 `@oaky-ai/cli@0.2.1`。这个 npm 包提供 `oaky` 命令，用于鉴权与访问服务。Skill 格式本身不依赖 npm，Oaky 的业务执行依赖这个工具。

从橡果设置 → 橡果 Skill 复制安装与授权说明给你信任的 AI。令牌通过 stdin 配置，不要提交到 Skill、仓库、命令参数或日志。完成后用 `oaky auth status` 与 `oaky agent capabilities` 验证连接。

Skill、执行依赖、橡果 API 分别更新。升级/移除 Skill 使用原安装方式，不混用社区安装器、ZIP 与 `oaky agent install` 的目录管理。`oaky agent install` 是 CLI 自带安装器（兼容渠道），只维护它自己安装的副本。

## 能力与边界

包含收藏与记录、整理、回顾与问答、写成稿件、我的方法与偏好、公开资料研究、待办、定时任务与已授权应用数据的工作流。实际权限和工具合同以服务端动态目录为准；定时任务在橡果任务会话执行，不在当前 Codex/Claude 对话中执行。

本仓库仅发布技能文件，不包含个人授权、业务数据或橡果服务源码。标准格式与目录安装已经验证；真实账号业务和宿主模型执行仍需各环境验收。
