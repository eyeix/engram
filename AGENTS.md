# 验证要求（node + JSON 项目）

本项目无传统单元测试框架，所有变更必须通过以下检查后方可提交：

1. **语法检查**：`node --check hooks/*.js`，禁止带语法错误提交；
2. **插件校验**：`claude plugin validate .`，结构变更后必跑；
3. **JSON 校验**：`node -e "JSON.parse(require('fs').readFileSync('<file>'))"`，涉及 `.claude-plugin/*.json` 与 `hooks/hooks.json` 时必跑；
4. **手动端到端**（涉及 hook 行为变更时）：`claude --plugin-dir .` 加载后，开新会话验证注入输出、触发一次对话结束验证巩固日志。

## 约定

- 代码注释、Git 提交信息使用中文，遵循 Conventional Commits；
- 记忆数据目录（默认 `~/.engram`）不属于本仓库，勿将其内容提交；
- 目标环境 = Claude Code 支持的全部桌面环境（macOS/Linux/Windows）。hook 逻辑用 node 实现并通过 hooks.json 的 exec-form（`command` + `args`）调用，绕开 Windows 无 Git Bash 时的 PowerShell 兜底；hook 内不得调用 bash/PowerShell 或平台专属命令，路径一律 `path.join` 拼接；
- 巩固子进程调用 `claude` CLI 时 prompt 走 stdin（避免引号转义），Windows 需 `shell: true` 解析 .cmd shim。
