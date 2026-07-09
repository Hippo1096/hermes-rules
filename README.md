# Hermes 规则同步仓库

本仓库存储 Hermes Agent 的跨机器规则导出包，用于同步 `SOUL.md` / `AGENTS.md` 等运行时规则。

## 目录结构

| 路径 | 说明 |
|------|------|
| `INDEX.json` | 规则包索引，记录各机器规则包元数据 |
| `CHANGELOG.md` | 规则同步变更日志 |
| `<machine-id>/` | 各机器导出的规则包目录 |

## 机器目录命名规范

目录名格式：`<hostname>-<uuid8>`，全小写。

示例：
- `hippo-167caaef` — Hippo [167CAAEF]
- `hippo1096-07b1f500` — Hippo1096 [07B1F500]

## 同步流程

1. 导出本机规则到 `<machine-id>/`
2. `git add -A && git commit && git push`
3. `git pull` 拉取其他机器规则
4. AI 审核其他机器规则，有价值的手动纳入本机 `SOUL.md` / `AGENTS.md`

## 注意

- 本仓库只做版本控制，不自动写回本机规则文件
- 纳入规则必须经过 AI 审核 + 用户确认
- 禁止在规则包中包含 API Key、token 等敏感信息
