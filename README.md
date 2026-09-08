# engram

Claude Code 全局自进化记忆插件。灵感来自 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 的有界自进化记忆：**不是记得多，而是记下的都值得**。

## 架构

四层组合，代码与数据严格分离：

| 层 | 实现 | 位置 |
|---|---|---|
| 规则层 | 维护规则由 SessionStart hook 注入（不侵入用户 CLAUDE.md） | `hooks/inject.js` |
| 自动化层 | SessionStart 注入 + Stop 后台巩固 | `hooks/hooks.json` |
| 能力层 | 手动审查/初始化命令 | `skills/memory-review/` |
| 数据层 | 有界记忆文件，独立 git 管理 | `~/.engram/`（默认） |

数据流：会话开始 → 注入规则与记忆 → 干活 → Stop → Haiku 后台读 transcript 提炼 → 查重/合并/淘汰 → 更新有界文件。

## 安装

```bash
# 本地开发模式（推荐先验证）
claude --plugin-dir /path/to/engram

# 通过自托管 marketplace（推送到 GitHub 后）
/plugin marketplace add <owner>/engram
/plugin install engram@engram
```

即插即用：安装后无需任何手动步骤——数据目录（模板、archive/logs、git 仓库）由首次 Stop 巩固自动初始化。`/engram:memory-review` 仅用于手动审查整理已有记忆。

## 配置

| 环境变量 | 默认值 | 说明 |
|---|---|---|
| `ENGRAM_MEMORY_DIR` | `~/.engram` | 数据目录位置（勿放 `~/.claude/` 内，该目录受 Claude Code 敏感路径保护，巩固进程无法写入） |

脚本内可调参数（`hooks/consolidate.js` 顶部）：巩固最小间隔（默认 1800 秒）、transcript 门槛（默认 30000 字节）、巩固模型（默认 Haiku）。

## 记忆文件约定

- `MEMORY.md`：跨项目通用事实/偏好/教训，硬上限 3000 字符，一行一条；
- `USER.md`：用户画像（沟通偏好、技术背景、环境事实），硬上限 1500 字符；
- `archive/`：被淘汰条目的按日归档，防"结构性遗忘"不可恢复；
- 超限时由模型自主浓缩取舍（写入时刻淘汰），这是本插件的核心哲学。

## 多机同步

插件（逻辑）随 git 仓库分发；记忆数据（状态）建议在 `$ENGRAM_MEMORY_DIR` 内设置私有远程仓库，机器间 push/pull 同步。

## 验证

```bash
node --check hooks/inject.js && node --check hooks/consolidate.js   # 语法检查
claude plugin validate .                                            # 插件结构校验
```

端到端：开新会话教它一个偏好 → 结束对话 → 约 1 分钟后查看 `$ENGRAM_MEMORY_DIR/logs/consolidate.log` 与 `MEMORY.md` → 新会话确认注入生效。

## 依赖与限制

- 目标环境为 Claude Code 支持的全部桌面环境（macOS/Linux/Windows）：hook 逻辑为纯 node，经 hooks.json 的 exec-form 调用，不依赖 Git Bash/PowerShell；需 `node` 与 `claude` CLI 在 PATH 中（Windows 实测欢迎反馈）；
- 无语义检索为已知局限，依靠"全局层只存精华 + 条目量少"缓解；
- 仅面向 Claude Code，不为通用 MCP 客户端设计。

## 卸载

`/plugin uninstall` 或移除 `--plugin-dir` 即可，规则随插件消失；数据目录保留，可手动删除。
