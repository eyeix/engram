# 260908 · 新增 /engram:sync 记忆同步命令

## 背景与目标
用户要求新增 sync 命令：通过 git 远程仓库同步本地记忆，同步时检查远程记忆并给出冲突合并建议供用户决定，决定后执行 push/pull；数据目录未 git init 时应提示用户。

## 现状分析
- 数据目录 `$GM` 由首次 Stop 巩固自动 `git init`（consolidate.js `ensureDataDir`），但自动巩固子进程只有 `Read,Edit,Write,Glob` 权限、不执行 git 提交 → 工作区常态有未提交变更，sync 须前置处理；
- 现有唯一手动命令 `/engram:memory-review`（skills/memory-review），为指令式 SKILL，由会话内 Claude 执行；sync 同构实现，新增 `skills/sync/SKILL.md`；
- README「多机同步」章节此前只有建议性描述，无对应命令。

## 方案设计
- 命令名 `sync`，用户入口 `/engram:sync`；指令式 SKILL 而非脚本：记忆条目的语义级合并必须由模型完成，git 操作由模型调 shell 执行；
- 前置检查（按序）：
  1. 未 git 初始化 → 提示用户（通常由首次会话结束的巩固自动初始化）；文件已齐全时可经确认后手动 `git init + add + commit`；目录或模板缺失时优先建议触发一轮巩固，遵守模板单一来源约束，勿在 SKILL 中复制模板；
  2. 未配置 remote → 询问远程地址（建议私有仓库），确认后 `git remote add origin <url>`；
  3. 脏工作区 → 展示变更摘要，经确认后 `git add -A && git commit`；
- 差异判定：`fetch` 后 `rev-list --left-right --count HEAD...origin/$B`：仅落后 → `--ff-only` 拉取；仅领先 → push；分叉 → `merge --no-commit`；
- 分叉合并：自动合并成功则展示摘要后提交推送；有冲突则 `git show` 取两侧干净版本，按 `[YY-MM-DD] 主题` 配对给建议（仅一侧保留/相同去重/同主题以日期较新者为准），合并结果守 3000/1500 上限，用户确认后写入 → add → commit（完成 merge，保留双父历史）→ push；archive/ 同名日文件做条目并集去重；
- 原则：一切覆盖性操作先取得用户确认；淘汰内容移入 archive/；只动 `$GM`，不触碰插件仓库。

## TODO
- [x] 阶段 1：创建任务文档、更新索引
- [x] 阶段 2：新增 skills/sync/SKILL.md
- [x] 阶段 3：更新 README（命令表 + 多机同步章节）
- [x] 阶段 4：bump 版本至 0.2.0（plugin.json + marketplace.json 两处一致）
- [x] 阶段 5：验证（node --check、JSON 校验、claude plugin validate）——全部通过
- [x] 阶段 6：回顾更新文档状态、按 Conventional Commits 提交
