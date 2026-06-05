# Axis Mental Health · RingClaw Bot 配置目录

## 目录结构

```
scenarios/
├── README.md                        本文件
│
├── clinical-bot/                    团队 Bot — 临床协调（#clinical-coordination）
│   ├── SOUL.md                      Bot 身份与行为规则
│   ├── DOMAIN.md                    诊所知识库（提供者、规则、联系方式）
│   └── memory/entities/
│       ├── AX-2847.md               患者档案 · Maria Lopez（续剂 Demo）
│       ├── AX-1956.md               患者档案 · 双侧提供者（跨提供者 Demo）
│       ├── AX-3201.md               患者档案 · James T.（爽约 Demo）
│       └── daily-queue-20260605.md  今日行政任务队列（Scenario C 依赖）
│
├── andrew-bot/                      个人 Bot — Andrew Wenner PMHNP
│   ├── SOUL.md
│   ├── DOMAIN.md
│   └── memory/entities/
│       ├── AX-1956.md               Andrew 视角的患者档案
│       └── coordination-watchlist.md 跨提供者监听状态
│
├── alexis-bot/                      个人 Bot — Alexis Gonzalez（行政）
│   ├── SOUL.md
│   ├── DOMAIN.md
│   └── memory/entities/
│       └── daily-queue-20260605.md  今日任务队列（与 clinical-bot 共享内容）
│
└── nursecoord-bot/                  团队 Bot — 行政协调（#admin）
    ├── SOUL.md
    ├── DOMAIN.md
    └── memory/entities/
        └── coverage-log.md          覆盖记录

```

## Bot 类型说明

| Bot | 类型 | 频道 | 谁能触发 |
|-----|------|------|---------|
| clinical-bot | 团队 Bot | #clinical-coordination | 频道里所有人 |
| andrew-bot | 个人 Bot | Andrew DM + #clinical-coordination（监听） | 仅 Andrew Wenner |
| alexis-bot | 个人 Bot | Alexis DM + #admin（发布） | 仅 Alexis Gonzalez |
| nursecoord-bot | 团队 Bot | #admin | #admin 频道成员 + alexis-bot 信号 |

## 启动命令参考

```bash
# 每个 bot 独立实例，--dir 隔离
ringclaw start --dir ~/.ringclaw-axis-clinical/
ringclaw start --dir ~/.ringclaw-axis-andrew/
ringclaw start --dir ~/.ringclaw-axis-alexis/
ringclaw start --dir ~/.ringclaw-axis-nursecoord/

# 各实例的 config.json 指定各自的 bot token 和监听频道
```

## Demo 场景索引

- **Scenario A**：续剂路由 → `clinical-bot`，依赖 `AX-2847.md`
- **Scenario B**：跨提供者协调 → `andrew-bot`，依赖 `AX-1956.md`
- **Scenario C**：缺勤交接 → `alexis-bot` → `nursecoord-bot`，依赖 `daily-queue-20260605.md`
