# 260908 · engram 插件骨架搭建

## 背景

用户希望让 Claude Code 拥有类似 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 的全局自我进化记忆。经调研（Hermes 机制分析 + 现成记忆框架横评 + Claude Code 官方 plugin 机制），决定不用现成框架（claude-mem / mcp-memory-service 等），自建轻量插件 **engram**，实现"Hermes 核心机制的 Claude Code 原生等价物"。

调研要点：
- Hermes 核心 = 有界记忆（MEMORY.md 硬上限，超限时模型自主取舍）+ 主动触发 + 写入时巩固（矛盾在写入时刻解决）+ Skill 自动蒸馏；
- 现成框架全部走"存得多压缩得好"路线，无有界压缩哲学，且托管型与用户隐私偏好冲突；
- Claude Code 官方 plugin 机制（`.claude-plugin/plugin.json` + `hooks/hooks.json` + `skills/`）可完整承载该方案，`--plugin-dir` 本地开发、marketplace 分发。

## 设计决策

| 决策 | 理由 |
|---|---|
| 四层架构：规则（hook 注入）+ 自动化（hooks）+ 能力（skill）+ 数据（独立 git） | 完整对应 Claude Code 扩展模型；装卸干净 |
| 维护规则经 inject.js 注入而非写 CLAUDE.md | 插件可逆性：卸载即消失，不改用户全局配置 |
| Stop hook 触发 `claude -p` + Haiku 后台巩固 | 复刻 Hermes "经验提取→巩固→复用"闭环；Haiku 成本极低 |
| `ENGRAM_CHILD` 环境变量防递归 | 子 Claude 的 Stop hook 会再次触发本脚本，必须硬防线 |
| 30 分钟去抖 + 30KB transcript 门槛 | Stop 每轮响应都触发，防止过度巩固与成本浪费 |
| Skill 蒸馏写 `~/.claude/skills/`（用户目录） | 插件目录会被版本更新覆盖，运行时数据不进插件 |
| 数据目录默认 `~/.engram`，`ENGRAM_MEMORY_DIR` 可覆盖 | `~/.claude/` 受内建敏感路径保护，巩固进程写不进去 |
| hook 逻辑用 node 实现，hooks.json exec-form 调用 | 目标 = Claude Code 全部桌面环境；Windows 无 Git Bash 时 hooks 走 PowerShell，bash 路线必挂 |
| 即插即用：首次巩固自动初始化数据目录 | 安装即可用；`/engram:memory-review` 专职审查整理 |

## 完成状态

骨架搭建、`claude plugin validate` 结构校验、`--plugin-dir` 本地加载与 SessionStart 注入验证、端到端巩固闭环验证（教偏好 → Stop 巩固 → 查重落盘 → 新会话注入生效）均已完成，含后续的 node 全平台重构与即插即用改造。已推送 GitHub（[eyeix/engram](https://github.com/eyeix/engram)，公开），marketplace 安装路径验证通过：`claude plugin marketplace add eyeix/engram` → `claude plugin install engram@engram` → 新会话自动注入生效。Windows 环境为文档依据 + 代码保证，未实测。

后续方向：（可选）多机同步数据目录 git remote。

## 关键约束

跨任务长期约束（权限适配、敏感目录禁区、模板单一来源、node 实现要求等）见 `../../index.md` 全局重要记忆。
