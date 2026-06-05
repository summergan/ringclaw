# RingClaw · Axis Integrated Mental Health PoC · RUNBOOK v6

**目标客户**：Axis Integrated Mental Health（Colorado，精神心理连锁）
**架构**：RingEX 个人 Bot + 团队 Bot，entity memory 驱动
**更新**：2026-06-05
**数据来源**：深度研究验证（106 个 agent，15 个声明通过三票验证）

---

## 一、客户背景（已验证事实）

```
公司：Axis Integrated Mental Health  |  axismh.com
地区：Colorado 多诊所

诊所（4 个）：
  Aurora（主，对外标注 Denver）1444 S Potomac St Suite 220, Aurora CO 80012
  Louisville（对外标注 Boulder）
  Westminster
  Denver Tech Center（2025-10 收购 BestMind 获得）

领导层：
  CEO：Christopher Perez（CRNA 背景，联合创始人）
  CMO/CGO：Liesl Perez（商业背景，联合创始人）
  CMO 2026-04：Dr. Kartiki Churi MD MBA FAPA
  COO 2026-04：Mike Emerson
  Director of Clinical Ops：Amie Naas

临床团队（17-18 人，全部来自 axismh.com/providers/）：
  精神科提供者（开药）：
    Andrew Wenner PMHNP · Madison Walter PMHNP · Elinor O'Buckley PMHNP
    Sarah McLaughlin PMHNP · Frani Rhodes PMHNP · Ross Van Allen DNP
    Kayla Sharpe PMHNP · Ben Egbers PMHNP · Carrie Karr PMHNP
    Roderick O'Brien MD · Ebony Villarreal PA-C
    Mary Chamberlain PA-C · Katerina Krieger PA-C
  治疗师（心理疗程）：
    Michelle Godwin LPC · Tori Kause LPC · Lisa Mazzola LPC
    Dez Nunez LPC · Laura Jane Landis LCSW

技术栈：
  EHR：athenahealth（athenaOne，2020 年上线）
  RC：AIR（Charlotte，AI 接待员）
  AIR 成果（已验证）：接通率 50% → 91%，来电量 500 → 2,000+/周，30% 自动化

服务线：标准精神科门诊 + BrainsWay Deep TMS + Spravato（REMS 认证）+ Ketamine
保险：99% 有保险（Medicaid · BCBS · Aetna · Cigna · UHC · Tricare）
患者量：1,000+（2023-12 下限，持续增长）
```

---

## 二、RingEX Bot 架构

### Bot 是如何创建的

```
员工在 RingEX 界面创建 Bot，RingClaw 是那个 Bot 的底层。

个人 Bot（只有创建者能发指令）：
  andrew-bot   → Andrew Wenner PMHNP
  alexis-bot   → Alexis Gonzalez（行政协调员）

团队 Bot（频道里所有人都能触发）：
  clinical-bot   → #clinical-coordination 频道
  nursecoord-bot → #admin 频道
```

### Bot 配置文件（scenarios/ 目录）

```
scenarios/
├── clinical-bot/
│   ├── SOUL.md      续剂路由逻辑 + 分支处理规则
│   ├── DOMAIN.md    所有提供者名单 + RC user ID + 药房
│   └── memory/entities/
│       ├── AX-2847.md           Maria Lopez 患者档案
│       ├── AX-1956.md           双侧提供者患者档案
│       ├── AX-3201.md           爽约患者档案
│       └── daily-queue-20260605.md   今日行政任务队列
│
├── andrew-bot/
│   ├── SOUL.md      跨提供者协调 + 频道监听 + 反馈路由
│   ├── DOMAIN.md    Andrew 的患者列表 + 同事 RC user ID
│   └── memory/entities/
│       ├── AX-1956.md           Andrew 视角的患者档案
│       └── coordination-watchlist.md  监听状态管理
│
├── alexis-bot/
│   ├── SOUL.md      缺勤交接 + 任务状态查询
│   ├── DOMAIN.md    备用行政联系人 + 升级链
│   └── memory/entities/
│       └── daily-queue-20260605.md   今日任务队列（与 clinical-bot 同步）
│
└── nursecoord-bot/
    ├── SOUL.md      任务转移接收 + SMS 覆盖 + 升级给 CEO
    ├── DOMAIN.md    备用行政手机号 + OT 政策
    └── memory/entities/
        └── coverage-log.md      覆盖记录
```

### Bot 间信号协议（#admin 频道消息总线）

```
alexis-bot 发：
  TASK_HANDOFF_REQUEST   → nursecoord-bot 接收，开始找替班

nursecoord-bot 发：
  HANDOFF_COMPLETE       → alexis-bot 接收，通知 Alexis 交接完成
  ESCALATED_TO_CEO       → alexis-bot 接收，告知 Alexis 已升级
```

---

## 三、Scenario A：续剂请求路由

**Bot**：`clinical-bot`（团队 Bot，#clinical-coordination）

**真实痛点**：Axis 每日 15-20 个续剂请求，Alexis 手动翻来电记录 → 转发给提供者 → 等回复 → 发短信给患者。平均 45 分钟/个，积压时超一天。athenahealth 门户有续剂功能，但电话渠道仍并存（已验证：续剂全靠门户的说法被否决）。

### 数据流

```
patients/AX-2847.md（预置）
  患者：Maria Lopez · 手机：+17205550102
  用药：Sertraline 100mg（Andrew Wenner 开，05/28 就诊）
  上次续剂：04/15（51 天前）
  保险：Blue Cross PPO（预置状态：有效）
  偏好药房：Walgreens Aurora

DOMAIN.md
  提供者 RC user ID 映射（Andrew Wenner → axismh-awenner）
  Alexis RC user ID（axismh-alexis）
```

### 完整流程

```
T=09:00  Charlotte（AIR）接听 Maria 的续剂来电
         Charlotte 发 SMS 给 Maria（AIR 自身能力）：
         "We received your Sertraline refill request. Dr. Wenner will review."

T=09:02  Alexis 在 RingEX 看到来电记录，在 #clinical-coordination 输入：
         "@clinical-bot refill AX-2847 Sertraline 100mg Andrew Wenner"

         （Group B：Maria 直接发短信到诊所号码 → inbound SMS 检测 → 自动触发）

T=09:02  clinical-bot 读 AX-2847.md → 生成 Adaptive Card → 推送 @Andrew Wenner：

         ┌──────────────────────────────────────────┐
         │ 续剂请求 · RX-20260605-047               │
         │ Maria Lopez（AX-2847）                   │
         │ Sertraline 100mg                         │
         │ 上次就诊：05/28（Andrew Wenner）          │
         │ 上次续剂：04/15（51 天前）✅ 合理         │
         │ 保险：Blue Cross PPO ✅                  │
         │                                          │
         │ [✅ 批准续剂] [📅 需要问诊] [❌ 拒绝]    │
         └──────────────────────────────────────────┘

T=09:05  Andrew 点「✅ 批准续剂」

T=09:05  clinical-bot 链式执行：
         → ACTION:SMS → Maria +17205550102：
           "Hi Maria! Dr. Wenner approved your Sertraline 100mg refill.
            Sent to your pharmacy. Ready in 2-4 hours."
         → ACTION:TASK → Alexis（axismh-alexis）：
           "RX-20260605-047 发送至 Walgreens Aurora  due 11:00"
         → AX-2847.md 追加续剂记录
         → daily-queue-20260605.md 追加 Task 条目
         → thread 回复 Andrew："✅ SMS 发出 · Alexis Task 创建 · 档案已更新"

T=09:05  Andrew 继续下一位患者（全程 3 分钟）
T=10:45  Alexis 处理 Task → Walgreens 发送处方 → Task 标记 completed
```

**分支：Andrew 点「📅 需要问诊」**

```
clinical-bot 并行执行：
  → ACTION:SMS → Maria："Dr. Wenner would like a follow-up visit first.
                          Our office will contact you today."
  → ACTION:TASK → Alexis：
    "[AX-2847] 安排 Andrew 复诊 [续剂被拒]
     Maria +17205550102 · 患者 SMS 已发 · due 今日 14:00"
  → AX-2847.md 状态更新："待复诊，Alexis 跟进中"
```

**量化**：15 个/天 × 节省 40 分钟 = **每日节省 10 小时行政时间**

---

## 四、Scenario B：跨提供者用药协调

**Bot**：`andrew-bot`（个人 Bot，Andrew Wenner）

**真实痛点**：Axis 精神心理诊所特有问题——同一患者同时有开药医生（PMHNP）和治疗师（LPC）。Andrew 调整 Sertraline 剂量，Michelle 治疗方向需同步更新（SSRI 增量影响焦虑水平）。目前靠口头或邮件，遗漏率高。

**真实提供者**：Andrew Wenner（PMHNP）+ Michelle Godwin（LPC）均为 axismh.com/providers/ 确认在职员工。

### 数据流

```
AX-1956.md（andrew-bot entity memory，预置）
  主诊精神科：Andrew Wenner（自己）
  主诊治疗师：Michelle Godwin, LPC（axismh-mgodwin）
  当前用药：Sertraline 50mg
  下次治疗师疗程：2026-06-12

coordination-watchlist.md
  记录"已发出协调通知，等待 Michelle 回复"状态
  超时：48h
```

### 完整流程

```
T=14:31  Andrew 就诊结束，DM 给 andrew-bot（个人 Bot，仅 Andrew 可触发）：
         "AX-1956 今天调了 Sertraline 50mg 到 100mg，
          原因是抑郁改善不足，通知 Michelle。"

T=14:31  andrew-bot 读 AX-1956.md：
         找到主诊治疗师：Michelle Godwin, LPC（axismh-mgodwin）
         下次疗程：2026-06-12

         andrew-bot → ACTION:MESSAGE → #clinical-coordination @Michelle Godwin：

         ┌──────────────────────────────────────────┐
         │ 🔄 跨提供者用药更新 · AX-1956             │
         │（来自 Andrew Wenner，2026-06-05）         │
         │                                          │
         │ Sertraline 50mg → 100mg                 │
         │ 原因：抑郁症状改善不足                    │
         │                                          │
         │ 临床提示：SSRI 增量初期可能焦虑波动，     │
         │ 建议下次疗程（06/12）重点评估焦虑水平     │
         │                                          │
         │ @Michelle Godwin 请回复确认收到           │
         └──────────────────────────────────────────┘

         → ACTION:TASK → Michelle：
           "AX-1956 用药变更确认  due 2026-06-06 10:00"
         → AX-1956.md 状态更新："协调发出，等待 Michelle 确认"
         → coordination-watchlist.md 添加监听条目
         → 回复 Andrew DM："✅ Michelle 已收到通知 · Task 创建（due 明日 10:00）"

T=14:35  Michelle 在 #clinical-coordination 自然语言回复（无需 @bot）：
         "收到，谢谢 Andrew。顺便说一下，
          上次疗程（06/03）她提到轻微心悸，Andrew 需要知道吗？"

T=14:35  andrew-bot 监听 #clinical-coordination：
         匹配到：发送者 = Michelle（在 watchlist 中）
         内容包含新临床信息（心悸症状）→ 需要路由给 Andrew

         andrew-bot → ACTION:MESSAGE → Andrew DM：

         ┌──────────────────────────────────────────┐
         │ 💬 Michelle 的临床反馈 · AX-1956          │
         │                                          │
         │ Michelle Godwin（14:35）：               │
         │ "患者 06/03 疗程提到轻微心悸"            │
         │                                          │
         │ 建议：随访时评估是否与 Sertraline 增量相关 │
         └──────────────────────────────────────────┘

         → AX-1956.md 追加："Michelle 反馈心悸，Andrew 已知悉（14:38）"

T=14:38  Andrew 回复 andrew-bot DM："谢谢，下次随访评估。"
         → andrew-bot 关闭监听条目
         → Task 标记 completed
```

**A→A 路径**：andrew-bot 发协调消息（A→H），然后监听 Michelle 的自然语言回复（H），判断含临床信息后路由给 Andrew DM（A→H）。信息在提供者之间完整流转，无人工中转。

---

## 五、Scenario C：行政缺勤 → Bot→Bot 交接

**Bot**：`alexis-bot`（个人 Bot）→ `nursecoord-bot`（团队 Bot）

**真实痛点**：Alexis 是 Axis 的行政核心，续剂任务、患者沟通、Task 协调都经过她。缺勤时 clinical-bot 创建的续剂 Task 积压，无自动转移机制。

### 数据流

```
alexis-bot 读取：
  daily-queue-20260605.md（Alexis 视角的今日任务队列）
    续剂 Task：12 个（最近 due 12:00）
    待跟进患者：2 个
    今日预约：24 名

nursecoord-bot 读取：
  DOMAIN.md 备用联系人：
    Jennifer S. +17205550301（全天可用，首选）
    Karen M.    +17205550312（下午可用）
  升级联系人：Christopher Perez（CEO，axismh-cperez）
```

### 完整流程

```
T=07:45  Alexis DM 给 alexis-bot：
         "今天身体不舒服，缺勤，帮我处理交接"

T=07:45  alexis-bot 读取 daily-queue-20260605.md：
         → 续剂 Task 12 个（最近 due 12:00）
         → 待跟进患者 2 个（AX-3104 15:00前，AX-2890 12:00前）
         → 今日预约 24 名

         → ACTION:MESSAGE → #admin（发布缺勤通知）：
           "缺勤通知（alexis-bot 代发）
            Alexis 今日缺勤 · 续剂 Task 12 个（最近 due 12:00）· 正在安排覆盖…"

T=07:46  alexis-bot → ACTION:MESSAGE → #admin（发信号给 nursecoord-bot）：
         "TASK_HANDOFF_REQUEST
          来源：alexis-bot（Alexis 今日缺勤）
          任务文件：daily-queue-20260605.md
          紧急程度：NORMAL（最近 due 12:00）"

T=07:46  nursecoord-bot 检测到 TASK_HANDOFF_REQUEST：
         读 daily-queue：12 个 Task，最近 due 12:00
         读 DOMAIN.md 备用顺序：Jennifer 首选

         → ACTION:SMS → Jennifer +17205550301：
           "Hi Jennifer! Axis Mental Health needs admin coverage today.
            Alexis is out sick. 12 refill tasks, first due at noon.
            Can you cover? Reply YES within 15 min. — Axis"

         → ACTION:SMS → Karen +17205550312：
           "Hi Karen! Axis needs admin help today, Alexis is out.
            Can you cover afternoon (1pm-6pm)? Reply YES. — Axis"

T=07:52  Jennifer 回复 YES（手机短信，6 分钟后）
         （Group B inbound SMS 自动检测，或在 RC 频道人工转述）

T=07:52  nursecoord-bot 处理 YES：
         → ACTION:SMS → Jennifer："Confirmed! Please log in to RingEX.
                                    First refill task due at noon. Thank you!"
         → ACTION:SMS → Karen："Coverage arranged. Thank you Karen!"

         → ACTION:CARD → #admin：
           ┌──────────────────────────────────────┐
           │ ✅ 行政覆盖已安排                      │
           │ Alexis 缺勤 → Jennifer S. 今日全天    │
           │ 响应时间：6 分钟                      │
           │ 12 个续剂 Task 已移交                 │
           └──────────────────────────────────────┘

         → ACTION:MESSAGE → #admin：
           "HANDOFF_COMPLETE
            覆盖人员：Jennifer S.
            确认时间：07:52"

T=07:52  alexis-bot 检测到 HANDOFF_COMPLETE：
         → ACTION:MESSAGE → Alexis DM：
           "✅ 交接完成
            Jennifer S. 已接手今日工作（07:52 确认）
            12 个续剂 Task + 2 个跟进已移交
            好好休息 🙂"

T=07:55  Alexis 收到 DM 汇报，安心休息。
```

**备用路径（15 分钟无回应）**

```
T=08:02  nursecoord-bot 超时检测：Jennifer/Karen 均无回应

         → ACTION:CARD → Christopher Perez（CEO DM）：
           ⚠️ 需要你的决定 · 行政覆盖
           Alexis 缺勤 · Jennifer/Karen 15 分钟无回应
           待处理：12 个 Task（最近 due 12:00）· 今日预约 24 名
           [Message Jennifer] [Message Karen] [Mark as handled]

         → ESCALATED_TO_CEO 信号 → alexis-bot → Alexis DM：
           "⚠️ Jennifer/Karen 未回应，已升级给 Christopher，请放心休息"
```

**量化**：从"Alexis 打电话联系备用，30-60 分钟稳定"→ **6 分钟自动完成，开门前不慌**

---

## 六、交互模式总表

| 场景 | 触发 | Bot | 信号/动作 | 数据来源 |
|------|------|-----|---------|---------|
| A-1 续剂批准 | Alexis @clinical-bot | clinical-bot（团队） | CARD→Andrew, SMS→患者, TASK→Alexis | entity memory AX-2847 |
| A-2 续剂被拒 | Andrew 点 Card 按钮 | clinical-bot（团队） | SMS→患者（即时）, TASK→Alexis | entity memory AX-2847 |
| B 跨提供者协调 | Andrew DM andrew-bot | andrew-bot（个人）→ clinical-bot 频道 | MESSAGE→Michelle, 监听回复→Andrew DM | entity memory AX-1956 |
| C-1 缺勤交接 | Alexis DM alexis-bot | alexis-bot→nursecoord-bot（Bot→Bot） | TASK_HANDOFF_REQUEST → SMS→Jennifer → HANDOFF_COMPLETE | daily-queue-20260605 |
| C-2 升级 | nursecoord-bot 超时 | nursecoord-bot→CEO | CARD→Christopher, ESCALATED_TO_CEO | DOMAIN.md 备用联系人 |

---

## 七、5 分钟 Demo 脚本

### 0:00–0:30 | RingEX Bot 界面

```
打开 RingEX Bot 创建界面

"在 RingEX 里，每个员工都可以创建自己的 Bot。
 这是 Andrew Wenner 的个人 Bot——只有他能和它说话。
 这是临床团队的共用 Bot——频道里所有人都可以触发。

 每个 Bot 有自己的 Soul（身份规则）和 entity memory（患者档案）。
 RC 的能力——SMS、Team Messaging、Task——默认全部可用。"
```

### 0:30–1:45 | Scenario A：续剂路由（clinical-bot）

```
[打开 #clinical-coordination]

"Charlotte（AIR）接了 Maria 的续剂来电，发了确认短信给患者。
 Alexis 在 RingEX 里看到来电记录，@一下 clinical-bot。"

[Alexis 输入 @clinical-bot refill AX-2847 Sertraline 100mg Andrew Wenner]

"clinical-bot 读了患者档案——Entity Memory 里的 AX-2847——
 上次就诊、续剂窗口、保险状态，生成这张 Card 给 Andrew。"

[展示 Card]

"Andrew 点批准，15 秒。
 患者收到 SMS，Alexis 有了 Task，档案自动更新。
 6 分钟。原来 45 分钟。

 每天 15-20 个这样的请求，等于省掉 10 小时行政时间。"
```

### 1:45–3:00 | Scenario B：跨提供者协调（andrew-bot，个人 Bot）

```
"现在换一个角度——医生主动发起。

 Axis 精神心理诊所有一个特殊问题：
 同一个患者，Andrew 负责开药，Michelle 负责心理疗程。
 Andrew 调整了 Sertraline 剂量，Michelle 必须知道。

 Andrew 私信自己的 Bot——andrew-bot，只有他能触发它。"

[Andrew DM andrew-bot：AX-1956 Sertraline 50→100mg，通知 Michelle]

"andrew-bot 知道这个患者有两个提供者。
 它代表 Andrew，在临床频道发了协调通知给 Michelle。"

[展示频道里 andrew-bot 发出的协调消息 @Michelle]

"Michelle 用自然语言回复了一句话，她没有 @任何 Bot。

 andrew-bot 监听频道，识别到 Michelle 提到了新的临床症状——心悸——
 它直接把这条信息推到 Andrew 的 DM 里。"

[展示 Andrew DM 里收到的临床反馈通知]

"Andrew → Michelle → Andrew，闭环。
 EHR 是记录系统，它不会主动把 Michelle 说的话路由给 Andrew。
 这是个人 Bot 的价值：Andrew 的助理，只服务于 Andrew。"
```

### 3:00–4:15 | Scenario C：缺勤交接（alexis-bot → nursecoord-bot）

```
"第三个场景：Alexis 早上 7:45 告诉自己的 Bot 她今天生病缺勤。"

[Alexis DM alexis-bot：今天缺勤，帮我处理交接]

"alexis-bot 读了今天的任务队列——12 个续剂 Task——
 在 #admin 发了缺勤公告，
 然后向 nursecoord-bot 发了一条任务转移信号。"

[展示 #admin 里的缺勤通知 + TASK_HANDOFF_REQUEST 信号]

"nursecoord-bot 接收信号，同时给 Jennifer 和 Karen 发手机短信。"

[展示两条 SMS 发出]

"Jennifer 6 分钟回复 YES。
 nursecoord-bot 确认覆盖，发 HANDOFF_COMPLETE 信号给 alexis-bot。
 alexis-bot 告诉 Alexis：Jennifer 已接手，好好休息。"

[展示 #admin 的覆盖 Card + Alexis DM 收到的汇报]

"这里有三层协作：
 个人 Bot（alexis-bot）→ 团队 Bot（nursecoord-bot）→ SMS 到 Jennifer 手机。
 Alexis 告诉 Bot 一句话，8:00 开门前，问题已经解决。"
```

### 4:15–5:00 | 价值主张

```
"三个场景，三种 Bot 协作：

  续剂路由   团队 Bot 接触发，医生点一下，10 小时行政时间/天
  跨提供者   个人 Bot 代替 Andrew 协调 Michelle，反馈自动路由回来
  缺勤交接   个人 Bot → 团队 Bot，6 分钟找到替班，开门前不慌

 这三件事都发生在 Axis 已经在用的 RingEX 里。
 不换工具，不迁移，不培训新系统。

 每个员工在 RingEX 里创建自己的 Bot——
 知道自己的患者，知道自己的任务，
 在正确的时刻做正确的事。

 Charlotte（AIR）接了患者的电话。
 RingClaw 让诊所里的每个人，和他们的 Bot，把这件事做完。"
```

---

## 八、Group B 代码缺口

```
影响场景：
  A-1 选项 B：患者 SMS 自动触发 clinical-bot
  C-1 SMS YES 自动确认：Jennifer 回复 YES 自动处理

需要（~80 行，模式与现有 complaint 分支完全相同）：
  1. start.go +1 行：
     monitor.SetMessageStoreHandler(buildMessageStoreHandler(cfg, handler, cs))
  2. buildMessageStoreHandler 扩展：
     "refill" → 注入 #clinical-coordination
     "YES"    → 查 open dispatch → 确认覆盖
     "NO"     → 取消，通知下一位

Scenario B 和 C（Bot→Bot 场景）：无需 Group B，全程在 Team Messaging 内完成

Demo 替代（Group A）：
  A-1：Alexis 手动 @clinical-bot 一行输入
  C-1：在 RC 频道转述 Jennifer 的回复 "@nursecoord-bot Jennifer 回复 YES"
```

---

## 九、配置参考

```bash
# 启动各 Bot 实例
ringclaw start --dir ./scenarios/clinical-bot/
ringclaw start --dir ./scenarios/andrew-bot/
ringclaw start --dir ./scenarios/alexis-bot/
ringclaw start --dir ./scenarios/nursecoord-bot/

# 各实例 config.json 关键字段：
# clinical-bot：chat_ids: ["#clinical-coordination"]
# andrew-bot：  chat_ids: ["<andrew-dm-id>", "#clinical-coordination"]
# alexis-bot：  chat_ids: ["<alexis-dm-id>", "#admin"]
# nursecoord-bot：chat_ids: ["#admin"]
```
