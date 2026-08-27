---
name: didacli
description: 滴答清单（DIDA / dida365）任务与效率管理插件。当用户提到滴答清单、DIDA、dida365、待办、任务管理，或要求创建/查询/更新/完成任务、管理清单/标签/习惯/专注记录/倒数日时加载本技能。本插件通过 dida-cli（@suibiji/dida-cli）命令行操作滴答清单，面向 AI Agent：优先使用 --json 获取结构化输出、非交互式执行、谨慎处理删除等破坏性命令。
---

# 滴答清单插件 — AI Agent 使用指南

滴答清单插件通过 `dida-cli`（封装滴答清单 Open API v1 的命令行工具，终端命令 `dida`，别名 `dida-cli`）管理任务、清单、标签、习惯、专注记录与倒数日。本技能指导 AI Agent 正确安装、认证并调用它。

## 前置条件与安装

- 需要 **Node.js >= 20**（`node --version` 先确认）。
- 推荐直接从 npm 安装：

```bash
npm install -g @suibiji/dida-cli
```

- 或从源码安装：`npm install && npm run build && npm link`。
- 安装后验证：`dida --version` 或 `dida --help`。

## 登录认证（先做）

**每次使用前先检查登录状态**：`dida auth status`。未登录时：

```bash
dida auth login            # OAuth 浏览器登录（推荐，本机有浏览器时）
dida auth token <token>    # 直接写入 access token（无浏览器 / 远程环境用此方式）
dida auth logout           # 清除本地 token
```

- **Agent 注意**：`dida auth login` 会打开浏览器并等待人工授权，属于交互式阻塞操作——在无头环境下不要自动执行；应引导用户改用 `dida auth token <token>`。
- **获取 Token**：网页版滴答清单 →「头像」→「设置」→「账户与安全」→「API 口令」中创建并复制。
- Token 是敏感凭据：不要写入文件、日志或聊天记录，不要泄露。

## Agent 调用纪律

1. **机器可读输出优先**：任何查询类命令都加 `--json`，拿到原始 API JSON（camelCase 字段）再解析，不要解析人类可读表格。
2. **非交互执行**：所有命令都是一次性执行；不要把需要人工的 OAuth 流程当作自动步骤。
3. **ID 依赖**：大多数命令需要 `projectId` / `taskId` 等 ObjectId。不知道时先 `dida project list --json` 或 `dida task search` 查到再操作，不要臆造 ID。
4. **破坏性命令需用户确认**：`task delete`、`project delete`、`project group delete`、`auth logout` 执行前必须向用户确认。
5. **日期时间格式**：ISO 8601 带时区，如 `2026-03-10T09:00:00+0000`（注意无冒号的时区偏移）。
6. **优先级映射**：`0` 无，`1` 低，`3` 中，`5` 高（不是 1–5 连续）。
7. **任务状态**：`0` 未完成，`-1` 已放弃，`2` 已完成。

## 常用命令速查

### 任务（task）

```bash
dida task get <projectId> <taskId>
dida task create --title "买牛奶" --project <projectId>
dida task create --title "开会" --project <projectId> --priority 5 --due-date "2026-03-10T09:00:00+0000"
dida task create --title "复盘" --project <projectId> --tags 工作,紧急
dida task update <taskId> --id <taskId> --project <projectId> --title "新标题"
dida task update <taskId> --id <taskId> --project <projectId> --parent-id <parentTaskId>
dida task update <taskId> --id <taskId> --project <projectId> --parent-id null   # 取消父子关联
dida task update <taskId> --id <taskId> --project <projectId> --estimated-duration 1500 --estimated-pomo 5
dida task complete <projectId> <taskId>
dida task delete <projectId> <taskId>                      # 破坏性，先确认
dida task move --from <sourceProjectId> --to <destProjectId> --task <taskId>
dida task completed --projects <projectId> --start-date "2026-03-01T00:00:00+0000" --end-date "2026-03-09T23:59:59+0000"
dida task filter --projects <projectId> --priority 3,5 --status 0
dida task search "季度报告" --projects <projectId> --tags 工作 --status 0
dida task search --due-from "2026-07-01T00:00:00+0000" --due-to "2026-07-31T23:59:59+0000"

# 任务评论
dida task comment list <projectId> <taskId>
dida task comment add <projectId> <taskId> --title "已处理"
dida task comment delete <projectId> <taskId> <commentId>
```

### 清单（project）

```bash
dida project list
dida project get <projectId>
dida project data <projectId>        # 清单 + 未完成任务 + 分组
dida project create --name "工作" --color "#F18181" --view-mode list --kind TASK
dida project update <projectId> --name "新名字" --color "#4AB8A9"
dida project delete <projectId>      # 破坏性，先确认

# 清单分组（文件夹）
dida project group list
dida project group create --name "工作"
dida project group update <groupId> --name "个人"
dida project group delete <groupId>  # 破坏性，先确认

# 看板列
dida project column list <projectId>
dida project column create <projectId> --name "进行中"
dida project column update <projectId> <columnId> --name "已完成"
```

### 标签（tag）

```bash
dida tag list
dida tag create --name urgent --label urgent
```

### 习惯（habit）

```bash
dida habit get <habitId>
dida habit list
dida habit create --name "每天喝水" --repeat "RRULE:FREQ=DAILY;INTERVAL=1" --goal 8 --unit 杯
dida habit update <habitId> --name "每天喝水 2L" --goal 2000 --unit ml
dida habit checkin <habitId> --stamp 20260424 --value 1 --goal 1
dida habit checkins --habits <habitId> --from 20260401 --to 20260430   # 日期格式 yyyyMMdd
```

### 专注（focus）

```bash
dida focus get <focusId> --type pomodoro
dida focus list --from "2026-04-01T00:00:00+0800" --to "2026-04-07T23:59:59+0800" --type pomodoro   # 最大 30 天
dida focus create --type pomodoro --task-id <taskId> --start-time "2026-04-07T09:00:00+0800" --end-time "2026-04-07T09:25:00+0800" --duration 1500
dida focus delete <focusId> --type pomodoro
```

### 倒数日（countdown）

```bash
dida countdown list
```

## 关键字段与格式

### Task JSON 常用字段（`--json` 输出 / create/update 对应）

| 字段 | 含义 | CLI 长选项 |
| ---- | ---- | ---------- |
| `title` / `content` / `desc` | 标题 / 正文 / 说明 | `--title` `--content` `--desc` |
| `startDate` / `dueDate` | 起止时间；不同表示区间 | `--start-date` `--due-date` |
| `timeZone` | 时区 | `--time-zone` |
| `isAllDay` | 全天 | `--all-day` |
| `priority` | 0 无 / 1 低 / 3 中 / 5 高 | `--priority` |
| `reminders` | 提醒规则（见下） | `--reminders` |
| `repeatFlag` | 重复规则 RRULE/ERULE（见下） | `--repeat` |
| `items` | 检查事项（status 0/1） | `--items` |
| `tags` | 标签 | `--tags`（逗号分隔） |
| `parentId` | 父任务 | `--parent-id`（`null`/`none` 取消） |
| `focusSummaries.estimatedDuration` | 预计专注秒数 | `--estimated-duration` |
| `focusSummaries.estimatedPomo` | 预计番茄数（≤60） | `--estimated-pomo` |
| `kind` | `TASK` / `NOTE` / `CHECKLIST` | — |

### reminders 格式

`TRIGGER(;RELATED=START|END)?:(-)?P[nY][nM][nW][nD][T[nH][nM][nS]]`

| 字符串 | 含义 |
| ------ | ---- |
| `TRIGGER:-PT60M` | 参考时间点前 60 分钟 |
| `TRIGGER:-P1DT2H` | 前 1 天 2 小时 |
| `TRIGGER;RELATED=END:-PT15M` | 结束时间前 15 分钟 |
| `TRIGGER:PT0S` | 准时 |

### repeatFlag 格式

- `RRULE` 标准重复 / `ERULE` 高级重复，**不要混用**。
- 示例：`RRULE:FREQ=DAILY`、`RRULE:FREQ=WEEKLY;BYDAY=MO,WE`、`ERULE:NAME=CUSTOM;BYDATE=20260325,20260330`。

### Project JSON 常用字段

`name`(`--name`)、`color`(`--color`)、`viewMode`(`--view-mode`：`list`/`kanban`/`timeline`)、`kind`(`--kind`：`TASK`/`NOTE`)；`closed`、`groupId`、`permission` 只读。

## 典型工作流

1. **创建任务**：`dida auth status` → `dida project list --json` 找到目标清单 ID → `dida task create --title "..." --project <id> --json`。
2. **今日待办**：`dida task search --due-from "..." --due-to "..." --json` 或按清单 `dida project data <projectId> --json`。
3. **完成任务**：先 search/filter 定位 `(projectId, taskId)` → `dida task complete <projectId> <taskId>`。
4. **周报复盘**：`dida task completed --projects <ids> --start-date ... --end-date ... --json` 聚合已完成任务；配合 `dida focus list` 取专注数据。

## 排错

- 更多选项：`dida --help`、`dida <command> --help`。
- 401 / 认证错误：重新 `dida auth status`，token 失效则重新登录或换 token。
- 找不到任务/清单：确认 ID 是 ObjectId 十六进制且属于当前账号；先用 list/search 核实。
- 官方 API 文档：https://developer.dida365.com/docs#/openapi
