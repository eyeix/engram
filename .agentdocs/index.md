# engram · 文档索引

## 项目文档
`../README.md` - 项目说明（架构、安装、配置、验证），面向使用者

## 治理文档
`../AGENTS.md` - 验证要求与提交约定，任何变更前必读

## 已完成任务
`workflow/done/260908-engram-plugin.md` - 插件骨架搭建与首版验证

## 全局重要记忆
- **设计哲学**：有界压缩（bounded memory）。硬上限迫使模型在写入时刻取舍，"记下的都值得"，与"只进不出"的数字囤积路线相反。
- **代码与数据分离**：插件仓库只含逻辑（hooks/skills/manifest）；记忆数据在 `~/.engram/`（可用 `ENGRAM_MEMORY_DIR` 覆盖），自带 git 历史，多机靠私有远程同步。勿放 `~/.claude/` 内——该目录受 Claude Code 内建敏感路径保护，后台巩固进程（acceptEdits）无法写入。
- **规则不进 CLAUDE.md**：维护规则由 `hooks/inject.js` 在 SessionStart 时输出注入，装卸插件即装卸规则，不侵入用户配置。
- **防递归与去抖是硬约束**：`ENGRAM_CHILD` 环境变量防止子 Claude 的 Stop hook 无限递归；30 分钟去抖 + 30KB transcript 门槛防过度巩固。修改 consolidate.js 时不得移除这两道防线。
- **hook 逻辑必须用 node 实现**：目标环境 = Claude Code 支持的全部桌面环境。Windows 无 Git Bash 时 hooks 默认走 PowerShell，bash 脚本必挂；hooks.json 用 exec-form（`command: "node"` + `args`）调用，hook 内不得调用 bash/PowerShell 或平台专属命令。
- **巩固子进程的权限适配**：调用 `claude -p` 巩固时必须带 `--settings '{"env":{"ENABLE_TOOL_SEARCH":"false"}}'`（否则内置编辑工具延迟加载不可见）；权限模式用 acceptEdits，禁用 bypassPermissions；prompt 走 stdin 避免引号转义。
- **即插即用，模板单一来源**：数据目录初始化（模板/archive/logs/git/.gitignore）由 consolidate.js 的 `ensureDataDir` 在首次巩固时自动完成，勿在其他位置（SKILL.md 等）复制模板造成双源；`.gitignore`（忽略 `.last_run`、`logs/`）须在 git init 前落盘且独立于 git 分支判断存在（存量目录升级时补写），运行时状态文件不得入库；`/engram:memory-review` 只负责审查整理。
- **发布流程**：改代码 → bump 版本（`plugin.json` 与 `marketplace.json` 两处须一致）→ push → `claude plugin marketplace update engram` 刷新缓存 → `claude plugin update engram@engram` 升级本机安装（重启会话生效）；`claude plugin install` 对已装插件是幂等的，不会升级。
