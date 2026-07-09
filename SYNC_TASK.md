<!-- versions: 0.19.0@2026-07-09T18:40:00, 0.18.0@2026-07-09T16:00:00, 0.17.0@2026-07-02T09:09:21 -->

# Hermes 配置同步任务通道

> 其他机器 pull 后，sync 技能自动检查并执行。

## 同步架构（v6 — 去掉绝对路径指针同步）

本机 Hippo [167CAAEF] 已落地以下架构，其他机器按本任务执行：

```
同步 = 配置同步 + 规则同步 + 记忆同步（三种逐步执行）

通用原则：
- 不同机器 ID 的专属技能都会进入同步包，这是为了保持同步简洁、一次覆盖全部机器。
- 各机器运行时只使用与本机 SOUL.md 声明的机器标识相匹配的技能/配置，其余作为参考或跳过。
- 本机真实标识以 SOUL.md 为准，不依赖记忆、技能名或仓库中的历史记录。

① 配置同步（最简）
   hermes-config 仓库同步目录内容：
   skills/ scripts/ plugins/ memories/ hindsight/
   config.yaml 每机器独立，不纳入同步
   旧版绝对路径指针文件 hindsight / hindsight.json / memories / plugins / skills / scripts 已停止同步，
   改为每机器本地维护，仓库内仅保留目录内容。

② 规则同步（导出审核，非粗暴覆盖）
   SOUL.md + AGENTS.md → 导出 → rule-exports/<机器ID>/
   → git push → pull → AI 审核 → 纳入本机

③ 记忆同步（已有流程）
   Hindsight → 导出包 → agent-memory-hub → push/pull → 导入审核
   导出模板含临时记忆（第9章），来自 MEMORY.md
```

---

## 跨机器路径修复任务（070903）

**触发条件**：另一台电脑 pull 后，出现运行时目录为空、或浏览器/CDP配置指向 `C:/Users/Administrator/...` / `C:/Users/14302/...` / `C:/Users/stair/...` 等非本机路径。

**执行人**：sync 技能 / Hermes 自动或半自动执行。

**修复顺序**：
1. 读取本机 `SOUL.md` 中的机器标识与路径约定。
2. 检查运行时目录 `skills/ scripts/ plugins/ memories/ hindsight/` 是否为空或指向错误路径。
3. 若内容缺失，优先从 `hermes-config/.junction-backup/` 恢复对应目录到本机运行时路径。
4. 对运行时文本文件做本机路径替换：
   - `C:/Users/14302/` → 本机实际用户目录
   - `C:/Users/stair/` → 本机实际用户目录
   - 机器标识以 `SOUL.md` 为准
5. 重新执行 `python scripts/deploy.py` 或 `python scripts/git-smart-sync.py --force` 做一次同步写回。
6. 验证：`python scripts/verify-hindsight.py`，并检查技能/插件目录非空。

**禁止**：
- 不要继续 rebase/merge junction/symlink 仓库，优先 reset 或只改写文本指针。
- 不要把本机 `SOUL.md` 声明的 Edge/CDP 配置被其他机器的 Chrome 配置覆盖。

---

## 任务

### 070904
- from: Hippo1096 [07B1F500]
- to: all
- created: 2026-07-09
- status: pending
- completed_by:

同步技能规范已更新，其他机器 pull 后请注意以下变更：

1. 技能/插件同步改为最简模式：中转站已链接到实际运行时目录，不再需要运行时→中转站的清空覆盖步骤
2. 三类同步各自独立：技能/插件同步、规则同步、记忆同步，不要混用
3. 规则同步保持 AI 审核纳入机制，不自动覆盖本机 SOUL.md
4. 任务单读取放在 Git 同步之后、释放锁之前

如有疑问，以 hermes-config-sync 技能文档和本任务单为准。

当前无其他待处理任务。

## 归档说明

- 070901：机器专属技能命名规范，已完成
- 070902：v5 同步架构迁移，已完成；本机已完成 junction/symlink 化、rule-exports 导出、sync 技能恢复与 Hermes 重启加载
- 070903：停止同步 6 个绝对路径指针文件，改为目录内容同步；更新 SYNC_TASK.md 与 .gitignore；修复 SOUL.md symlink 为 regular file；推送 origin/main 并验证 clean
