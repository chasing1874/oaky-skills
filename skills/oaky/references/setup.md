# 安装与授权

Oaky Skill 是标准 Agent Skills 目录，通过公开仓库 `https://github.com/chasing1874/oaky-skills` 或该仓库 Release 的 ZIP 分发。Skill 本身不需要 npm；它使用的执行依赖是 `@oaky-ai/cli`（最低版本 0.2.1），命令为 `oaky`，需要 Node.js 22 或以上及网络访问。

## 已安装 Skill 后的连接

正在读取这份已安装的 Skill 时，不要再次运行 Skill 安装器。先运行 `oaky version`；缺少执行依赖时安装 `npm install -g @oaky-ai/cli@0.2.1`，版本低于 0.2.1 时更新依赖。不要把 npm 包安装成功当成宿主已经发现 Skill。

## Skill 安装与维护

Codex 的用户级目录是 `~/.agents/skills/oaky`，Claude Code 是 `~/.claude/skills/oaky`。可以使用宿主自带的 GitHub Skill 安装器，也可将 ZIP 内的完整 `oaky/` 目录放到对应位置。其他宿主按其实际目录约定安装；有标准格式支持不代表支持本 Skill 所需的本地命令、Node.js 和网络访问。

社区工具 `skills` 也能从仓库安装，例如 `npx skills add chasing1874/oaky-skills --skill oaky --global --agent codex`；Claude Code 使用 `--agent claude-code`。这是可选安装工具，不是 Agent Skills 协议的一部分。只安装用户指定的宿主，检查已有目录；如有同名或本地修改，先保留并说明冲突，不直接覆盖。

使用原安装方式升级或卸载 Skill，不要混用目录所有权。`oaky agent install/uninstall` 是 CLI 自带安装器（兼容渠道），只管理它自己安装的目录；通过社区安装器或 ZIP 安装的目录不要用它管理。切换方式时先备份、确认原目录无本地修改，再用原安装器卸载。Skill 升级与执行依赖升级是两个独立步骤。

## 授权

授权来自橡果设置 → 橡果 Skill。用户提供访问令牌后，通过 stdin 交给 `oaky auth login --token-stdin`，配置保存在 `~/.config/oaky/config.json`，权限为 0600。不要在 shell 参数里写 token，不写入仓库或 Skill 文件。自动化环境也支持 `OAKY_API_TOKEN`；`OAKY_API_URL` 只用于用户明确指定的非默认服务。不要读取或改写无关凭证。

运行 `oaky auth status` 确认账号，再运行 `oaky agent capabilities`。认证失败时说明令牌可能失效或账号不匹配，更新授权后重试，不循环提交业务写入。

卸载 Skill 不撤销账号授权。`oaky auth logout` 清除本地授权；要撤销远端访问，在 App 中删除对应令牌。移除执行依赖可用 `npm uninstall -g @oaky-ai/cli`。

旧命名测试版不自动迁移：重新安装 Oaky 并重新授权，确认可用后自行清理旧安装。不要默默读取旧账号配置。
