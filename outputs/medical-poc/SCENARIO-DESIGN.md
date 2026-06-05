# Axis Mental Health · 场景详细设计

**版本**：v1
**更新**：2026-06-05
**基础**：RingEX Bot 真实架构（个人 Bot + 团队 Bot）+ 已验证 Axis 数据

---

## 总体 Bot 布局

```
RingEX 里创建的 Bot：

团队 Bot（频道公开）
  clinical-bot    → #clinical-coordination（临床协调频道）
                    由 Axis 诊所管理员创建，临床团队共用

个人 Bot（DM + 授权频道）
  andrew-bot      → Andrew Wenner 私人 Bot
                    由 Andrew 在 RingEX 创建，只有他能发指令

  alexis-bot      → Alexis Gonzalez 私人 Bot
                    由 Alexis 在 RingEX 创建，处理她的行政任务

  nursecoord-bot  → 行政协调 Bot（小团队用）
                    → #admin 频道，Alexis / 备用行政可触发

每个 Bot 是一个独立的 RingClaw 实例（--dir 隔离）
Bot 的"身份" = SOUL.md
Bot 的"知识" = DOMAIN.md + entity memory 文件
Bot 的"能力" = ACTION:SMS / ACTION:CARD / ACTION:TASK / ACTION:MESSAGE
```

---

## Scenario A：续剂请求路由（clinical-bot 团队 Bot）

### 场景背景

```
Axis 每日 15-20 个续剂请求
现状：患者来电 → Charlotte(AIR)接听 → Alexis 从来电记录里手动提取信息
     → 邮件/口头告知提供者 → 等提供者回复 → 手动发短信给患者
     平均 45 分钟/个

核心痛点：Alexis 是唯一中间人，忙或缺勤时直接积压
          提供者收到的是非结构化邮件，没有患者档案
          患者不知道自己的请求状态
```

### Bot 配置

```
Bot 名称：clinical-bot
类型：团队 Bot
所属频道：#clinical-coordination
创建者：诊所管理员

SOUL.md 核心规则：
  我是 Axis Integrated Mental Health 的临床协调 Bot。
  当我在 #clinical-coordination 收到续剂触发信息时：
  1. 从 entity memory 读取患者档案（患者 ID 为关键）
  2. 生成 ACTION:CARD 推送给对应提供者，Card 包含：
     - 患者姓名、ID、手机号
     - 当前用药、上次就诊日期、上次续剂日期
     - 保险状态
     - 三个按钮：批准续剂 / 需要问诊 / 拒绝
  3. 等待提供者点击按钮
  4. 批准 → ACTION:SMS 患者 + ACTION:TASK Alexis
  5. 需要问诊 → ACTION:SMS 患者（告知等待联系）+ ACTION:TASK Alexis（安排复诊）
  6. 拒绝 → ACTION:SMS 患者（告知拒绝原因）

  我不能：提供任何医疗建议，直接修改处方，绕过提供者审批

DOMAIN.md：
  # 提供者名单
  Andrew Wenner, PMHNP     (RC user ID: axismh-awenner)
  Madison Walter, PMHNP   (RC user ID: axismh-mwalter)
  Elinor O'Buckley, PMHNP (RC user ID: axismh-eobuckley)
  Sarah McLaughlin, PMHNP (RC user ID: axismh-smclaughlin)
  Ross Van Allen, DNP      (RC user ID: axismh-rvanallen)
  Roderick O'Brien, MD     (RC user ID: axismh-robrien)
  Ebony Villarreal, PA-C  (RC user ID: axismh-evillarreal)
  Mary Chamberlain, PA-C  (RC user ID: axismh-mchamberlain)
  Katerina Krieger, PA-C  (RC user ID: axismh-kkrieger)

  # 行政
  Alexis Gonzalez (RC user ID: axismh-alexis)

  # 诊所主号码（发 SMS 用）
  Aurora clinic: +17205550100
```

### Entity Memory（患者档案，Demo 用预置）

```
文件：memory/entities/AX-2847.md
---
患者姓名：Maria Lopez
患者 ID：AX-2847
手机号：+17205550102
开药医生：Andrew Wenner, PMHNP
当前用药：Sertraline 100mg（开始日期：2026-03-15）
上次就诊：2026-05-28（Andrew Wenner）
上次续剂：2026-04-15（51 天前）
保险：Blue Cross PPO（状态：有效，有效期至 2026-12-31）
       注：PoC 预置值，非实时验证
偏好药房：Walgreens Aurora（+17205550200）
状态：active
---

文件：memory/entities/AX-3201.md
---
患者姓名：James T.
患者 ID：AX-3201
手机号：+17205550188
开药医生：Andrew Wenner, PMHNP
当前预约：2026-06-05 10:00am（Andrew Wenner，Aurora）
状态：active
---
```

### 完整流程（逐步详解）

```
T=09:00  患者 Maria Lopez 拨打 Axis 诊所电话

T=09:00  Charlotte（AIR）接听来电
         Charlotte 识别续剂意图：
           "I need a refill for my Sertraline 100mg,
            my doctor is Andrew Wenner."

         AIR 自动回复 Maria（AIR 自身能力）：
           SMS → Maria +17205550102：
           "Hi Maria! We received your Sertraline refill request.
            Dr. Wenner will review. We'll update you shortly."

T=09:02  Alexis 在 RingEX 里看到 Charlotte 的来电记录/摘要
         Alexis 在 #clinical-coordination 输入：

         ┌─────────────────────────────────────────────────┐
         │ Alexis [09:02]                                  │
         │ @clinical-bot refill AX-2847 Sertraline 100mg   │
         │ provider Andrew Wenner                          │
         └─────────────────────────────────────────────────┘

         （Group B：患者直接发 SMS 给诊所号码，clinical-bot 自动检测，
           无需 Alexis 手动输入）

T=09:02  clinical-bot 收到触发
         → 解析：患者 ID=AX-2847，药物=Sertraline 100mg，提供者=Andrew Wenner
         → 读 entity memory AX-2847.md
         → 生成 ACTION:CARD

T=09:02  clinical-bot → ACTION:CARD 推送 #clinical-coordination @Andrew Wenner

         ════════════════════════════════════════
          clinical-bot [09:02]
         ════════════════════════════════════════
         ┌──────────────────────────────────────────┐
         │ 🔵 续剂请求  RX-20260605-047             │
         │                                          │
         │ 患者：Maria Lopez（AX-2847）             │
         │ 手机：+1 (720) 555-0102                  │
         │                                          │
         │ 用药：Sertraline 100mg                   │
         │ 上次就诊：05/28/2026（Andrew Wenner）     │
         │ 上次续剂：04/15/2026（51 天前）           │
         │ 续剂窗口：✅ 合理（>30 天）               │
         │                                          │
         │ 保险：Blue Cross PPO  ✅ 有效             │
         │                                          │
         │ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
         │ │ ✅ 批准   │ │ 📅 问诊  │ │ ❌ 拒绝  │  │
         │ └──────────┘ └──────────┘ └──────────┘  │
         └──────────────────────────────────────────┘

T=09:05  Andrew 在手机上看到 Card（他在诊室看诊间隙）
         点击「✅ 批准续剂」

T=09:05  clinical-bot 处理批准事件，链式执行：

         Step 1 → ACTION:SMS to=+17205550102（Maria）：
           "Hi Maria! Dr. Wenner has approved your Sertraline 100mg refill.
            Your prescription has been sent to your pharmacy.
            It should be ready for pickup in 2–4 hours.
            Questions? Reply to this message. — Axis Mental Health"

         Step 2 → ACTION:TASK：
           subject: "💊 RX-20260605-047：发送处方至药房"
           assignee: Alexis Gonzalez（RC user ID: axismh-alexis）
           body:
             患者：Maria Lopez（AX-2847）
             药物：Sertraline 100mg
             提供者：Andrew Wenner 已批准（09:05）
             药房：Walgreens Aurora +17205550200
             患者已收到确认 SMS ✅
           due: 2026-06-05 11:00（+2h）

         Step 3 → entity memory 更新 AX-2847.md：
           续剂记录追加：
           "2026-06-05 Sertraline 100mg 续剂
            批准人：Andrew Wenner 09:05
            患者 SMS 已发 ✅
            Alexis Task 已创建 ✅"

         Step 4 → thread 回复（在 Card 下方）：
           "✅ RX-20260605-047 已处理
            · Maria SMS 发出（09:05）
            · Alexis Task 创建（due 11:00）
            · 档案已更新"

T=09:05  Andrew 看到确认，继续下一位患者（全程 3 分钟）

T=10:45  Alexis 处理 Task，通过 Walgreens 系统发送处方
         → 在 Task 里更新：completed
```

**分支：Andrew 点「📅 需要问诊」**

```
T=09:05  Andrew 点「需要问诊」

T=09:05  clinical-bot 链式执行（拒绝 + 安排路径）：

         Step 1 → ACTION:SMS to=+17205550102（Maria，即时安抚）：
           "Hi Maria! Dr. Wenner has reviewed your request and
            would like to schedule a follow-up visit before renewing.
            Our team will contact you today to set up a time.
            — Axis Mental Health"

         Step 2 → ACTION:TASK：
           subject: "📅 [AX-2847] 安排 Andrew 复诊 — 续剂被拒"
           assignee: Alexis Gonzalez
           body:
             患者：Maria Lopez（AX-2847）手机：+17205550102
             事由：Sertraline 100mg，Andrew 要求先复诊
             患者已收到 SMS 告知（09:05）✅
             请联系 Andrew 确认时段后回复患者
           due: 今日 14:00（4h window）

         Step 3 → entity memory 更新 AX-2847.md：
           状态追加：
           "2026-06-05 续剂请求 → 需复诊
            Andrew 要求面诊，患者 SMS 已发
            待 Alexis 安排预约"

         Step 4 → thread 回复：
           "📅 需要问诊
            · Maria 已收到 SMS（告知等待联系）✅
            · Alexis Task 创建（due 14:00）
            · 请 Alexis 协调 Andrew 排期"
```

**如果患者此后再次发来短信（Group B）**

```
Maria SMS → 诊所号码："Any update on my Sertraline refill?"

inbound SMS 检测 → clinical-bot：
  → 读 entity memory AX-2847：状态 = "待复诊，Alexis 跟进中"
  → ACTION:SMS → Maria：
    "Hi Maria! Your follow-up is being arranged.
     Alexis will contact you today to schedule a time.
     Thank you for your patience. — Axis Mental Health"
  → 不创建重复 Task
```

### 量化

```
之前：Alexis 手动处理 → 平均 45 分钟/个
现在：Alexis 输入一行 → clinical-bot 3 秒处理 → 提供者点一下 → 全自动

每日节省：15 个请求 × 40 分钟节省 = 10 小时行政时间
          （Alexis 每天只需处理异常 + 发送 Task，不再逐个跟进）
```

---

## Scenario B：跨提供者协调（andrew-bot 个人 Bot）

### 场景背景

```
Axis 精神心理诊所的临床模型：
  同一患者可能同时有：
    开药医生（PMHNP/MD/PA-C）— 负责用药方案
    治疗师（LPC/LCSW）       — 负责心理疗程

  当开药医生调整用药（如增加 SSRI 剂量），
  治疗师必须知道，因为：
    · SSRI 增量初期可能引起焦虑波动
    · 治疗师需要在疗程中重点评估这些变化
    · 用药变化可能影响治疗方向

  现在靠：口头提醒（在诊所走廊碰到）或发邮件
  问题：遗漏率高，无记录，治疗师不知道来自谁或何时
```

### Bot 配置

```
Bot 名称：andrew-bot
类型：个人 Bot（Andrew Wenner 创建，在 RingEX 里）
交互方式：
  · Andrew 私信 andrew-bot（DM）
  · andrew-bot 也监听 #clinical-coordination（它加入了这个频道）

SOUL.md 核心规则：
  我是 Andrew Wenner（PMHNP）的个人临床助理。
  只有 Andrew 可以给我发指令。

  当 Andrew 告诉我需要通知另一位提供者时：
  1. 从 entity memory 查找患者的所有提供者
  2. 在 #clinical-coordination 发结构化协调通知 @对应提供者
  3. 创建 Task 要求对方确认
  4. 监听 #clinical-coordination 中对方对此患者的回复
  5. 如果回复包含新的临床信息（症状、患者反馈），
     立即在 Andrew 的 DM 里通知他

  我代表 Andrew 说话，消息署名 andrew-bot（来自 Andrew Wenner）。
  我不会向患者发送任何消息（那是 clinical-bot 的工作）。
  我不提供用药建议，只传递提供者之间的临床信息。

  我知道的患者列表：来自 Andrew 的 entity memory 目录
  每次就诊后 Andrew 可以告诉我更新档案
```

### Entity Memory（双侧提供者患者）

```
文件：memory/entities/AX-1956.md
---
患者 ID：AX-1956
（注：患者姓名在演示中隐去，符合 HIPAA 示范规范）
主诊精神科（开药）：Andrew Wenner, PMHNP
主诊治疗师（心理）：Michelle Godwin, LPC
当前用药：Sertraline 50mg（开始日期：2026-04-01）
上次精神科就诊：2026-06-05（今日）
上次治疗师就诊：2026-06-03（两天前）
下次治疗师预约：2026-06-12
临床备注：
  2026-06-05 Andrew 就诊：抑郁症状改善不足
---

文件：memory/entities/AX-2103.md
---
患者 ID：AX-2103
主诊精神科：Madison Walter, PMHNP
主诊治疗师：Michelle Godwin, LPC
当前用药：Lexapro 10mg
---
```

### 完整流程（逐步详解）

```
T=14:30  Andrew 完成 AX-1956 的就诊
         判断：需要将 Sertraline 从 50mg 增至 100mg
         需要：通知 Michelle，因为下周一她有这个患者的疗程

T=14:31  Andrew 私信 andrew-bot（在 RingEX DM）：

         ┌─────────────────────────────────────────────────┐
         │ Andrew Wenner [14:31] → andrew-bot              │
         │ AX-1956 今天调了 Sertraline                     │
         │ 50mg 到 100mg，原因是抑郁改善不足               │
         │ 通知 Michelle，让她下次疗程关注焦虑波动          │
         └─────────────────────────────────────────────────┘

T=14:31  andrew-bot 处理：
         → 读 entity memory AX-1956：
           找到主诊治疗师：Michelle Godwin, LPC
           找到下次治疗师预约：2026-06-12
         → 构建协调通知

T=14:31  andrew-bot → ACTION:MESSAGE 到 #clinical-coordination @Michelle Godwin

         ════════════════════════════════════════
          andrew-bot [14:31]（来自 Andrew Wenner）
         ════════════════════════════════════════
         ┌──────────────────────────────────────────┐
         │ 🔄 跨提供者用药更新  AX-1956             │
         │                                          │
         │ 来自：Andrew Wenner, PMHNP               │
         │ 日期：2026-06-05（今日就诊）              │
         │                                          │
         │ 用药变更                                  │
         │   Sertraline 50mg → 100mg               │
         │   原因：抑郁症状改善不足                  │
         │                                          │
         │ 临床提示                                  │
         │   SSRI 增量初期可能出现焦虑波动           │
         │   建议下次疗程（06/12）重点评估焦虑水平  │
         │   及任何新出现的躯体症状                  │
         │                                          │
         │ @Michelle Godwin 请回复确认收到           │
         └──────────────────────────────────────────┘

         → ACTION:TASK：
           subject: "AX-1956 用药变更确认"
           assignee: Michelle Godwin（RC user ID）
           body: "Sertraline 50→100mg，Andrew 06/05，请在 06/12 疗程前确认"
           due: 2026-06-06 10:00（明日上午）

         → entity memory 更新 AX-1956.md：
           "2026-06-05 Sertraline 50→100mg (Andrew)
            已通知 Michelle Godwin，Task 待确认"

         → andrew-bot 回复 Andrew DM：
           "✅ Michelle 已收到通知（06/12 疗程前需确认）
            Task 已创建，明日 10:00 到期"

T=14:35  Michelle 在 #clinical-coordination 看到消息
         Michelle 用自然语言回复（无需 @任何 Bot）：

         ┌─────────────────────────────────────────────────┐
         │ Michelle Godwin [14:35]                         │
         │ 收到，谢谢 Andrew。                              │
         │ 顺便说一下，上次疗程（06/03）她提到             │
         │ 最近有轻微的心悸，我以为只是压力，              │
         │ 但现在增量了，Andrew 是否需要知道？             │
         └─────────────────────────────────────────────────┘

T=14:35  andrew-bot 检测到 Michelle 在 #clinical-coordination 的回复

         andrew-bot 分析：
           发送者：Michelle Godwin（AX-1956 的治疗师）
           内容：包含新的临床信息
             - 患者近期有心悸症状（上次疗程 06/03）
             - 疑问：是否与 Sertraline 增量相关
           判断：这是对开药医生相关的临床反馈
                 → 需要路由给 Andrew

T=14:35  andrew-bot → ACTION:MESSAGE 到 Andrew DM（私信）：

         ════════════════════════════════════════
          andrew-bot [14:35]
         ════════════════════════════════════════
         ┌──────────────────────────────────────────┐
         │ 💬 Michelle 的临床反馈  AX-1956           │
         │                                          │
         │ Michelle Godwin（14:35，来自频道回复）：  │
         │                                          │
         │ "患者在 06/03 疗程中提到轻微心悸           │
         │  可能与压力有关，现在 Sertraline 增量，   │
         │  Michelle 认为 Andrew 应该知道"           │
         │                                          │
         │ 建议：                                    │
         │   · 随访时（或下次就诊）询问心悸情况      │
         │   · 评估是否与 SSRI 增量相关              │
         └──────────────────────────────────────────┘

         → entity memory 更新 AX-1956.md：
           "2026-06-05 Michelle 反馈：
            患者 06/03 疗程提到心悸，可能与增量相关
            Andrew 已知悉"

T=14:38  Andrew 在 DM 里看到通知，回复 andrew-bot：
         "谢谢，下次随访时评估。请记录一下。"

         andrew-bot → entity memory AX-1956 追加：
           "2026-06-05 Andrew 知悉 Michelle 反馈，
            计划随访时评估心悸与增量关联"

         → Task（Michelle 的确认 Task）标记为 completed
```

### 关键技术细节

```
andrew-bot 如何知道回复来自 Michelle 且与 AX-1956 相关？

SOUL.md 中有规则：
  "在我发出协调通知后，监听该频道接下来 24 小时内：
   · 发送者 = 我@的那位提供者
   · 内容涉及该患者
   → 视为对此协调通知的回复"

实现方式：
  andrew-bot 在 entity memory 里记录：
    "待回复监听：
     患者：AX-1956
     等待回复人：Michelle Godwin（RC user ID）
     频道：#clinical-coordination
     到期：2026-06-06 14:31（24h）"

  当 Michelle 在该频道发消息时，andrew-bot 的 HandleMessage
  会检查是否有待监听的协调请求匹配：
    发送者 = Michelle？✅
    消息内容包含 AX-1956 相关信息？✅（bot 可用关键字匹配或 LLM 判断）
  → 触发反向路由
```

### 演讲词

```
"Andrew 的个人 Bot——只有他能和它说话。

 他就诊结束后，在 DM 里告诉 andrew-bot：
 '给 AX-1956 增了 Sertraline 剂量，通知 Michelle'。

 andrew-bot 知道这个患者有两个提供者，
 它代表 Andrew，在临床频道给 Michelle 发了一条结构化的协调通知。

 [展示 andrew-bot 在频道里发出的消息]

 Michelle 用自然语言回复了一句话，
 她没有 @任何 Bot。

 andrew-bot 识别到这是对 AX-1956 协调请求的回复，
 而且 Michelle 提到了一个新的临床信息——心悸——
 它直接把这条信息推给了 Andrew 的 DM。

 [展示 Andrew 收到的通知]

 Andrew 和 Michelle 各说了一句话。
 信息完整流转，有记录，有 Task，entity memory 有追踪。
 这是 EHR 做不到的——EHR 是记录系统，
 它不会主动把 Michelle 说的话路由给 Andrew。"
```

---

## Scenario C：行政缺勤交接（alexis-bot → nursecoord-bot）

### 场景背景

```
Axis 的行政协调员 Alexis Gonzalez：
  · 是续剂请求的人工中间人（即使有 clinical-bot，仍然有Task需要她处理）
  · 管理患者预约和随访
  · 协调提供者沟通
  · 处理保险和账单事务

当 Alexis 缺勤：
  · clinical-bot 创建的 Task 无人接收（积压）
  · 患者可能在等待中打来电话，Charlotte 接不了续剂相关问题
  · 其他提供者不知道今日工作量

现状：Alexis 打电话告知管理层，管理层手动联系备用行政人员
     再手动转移任务，通常需要 30-60 分钟才稳定
```

### Bot 配置

```
Bot 1：alexis-bot（个人 Bot）
  创建者：Alexis Gonzalez（她在 RingEX 里创建的）
  用途：管理她的行政任务队列
  交互：Alexis 私信 alexis-bot，或在她授权的频道

  SOUL.md 核心规则：
    我是 Alexis Gonzalez 的行政助理 Bot。
    
    我知道什么：
    · Alexis 今日待处理的 Task 列表（由 clinical-bot 创建的）
    · 备用行政人员联系方式（来自 DOMAIN.md）
    · 今日诊所患者量（来自 Alexis 的 entity memory 或早班报告）

    当 Alexis 告诉我她今日缺勤：
    1. 读取今日 Task 队列（entity memory 中的任务摘要）
    2. 在 #admin 发缺勤公告
    3. 向 nursecoord-bot 发任务转移信号
    4. 监听 nursecoord-bot 的确认，并向 Alexis DM 报告

  DOMAIN.md：
    备用行政人员：
      Jennifer S.  +17205550301  （全天可用标记）
      Karen M.     +17205550312  （下午可用）
    诊所经理：Christopher Perez（RC user ID）
    CEO（用于升级）：Christopher Perez（RC user ID）

Bot 2：nursecoord-bot（团队 Bot，#admin 频道）
  创建者：诊所管理员
  用途：协调行政和临床支持工作
  频道：#admin

  SOUL.md 核心规则：
    我是 Axis 行政协调 Bot。

    当我收到来自 alexis-bot 的任务转移信号时：
    1. 读取任务摘要
    2. 按 DOMAIN.md 中的备用顺序发 ACTION:SMS
    3. 等待回复（15 分钟超时）
    4. 收到 YES → 发确认 SMS + 通知 #admin + 通知 alexis-bot
    5. 所有候补无回应 → ACTION:MESSAGE @诊所经理升级

  DOMAIN.md：（同 alexis-bot，共用一份或各自维护）
```

### Entity Memory（任务队列和日程，每天更新）

```
文件：memory/entities/daily-queue-20260605.md
（由 clinical-bot 在创建 Task 时追加写入，alexis-bot 读取此文件）
---
日期：2026-06-05
行政：Alexis Gonzalez
续剂 Task 队列：
  - RX-20260605-047 Maria Lopez AX-2847 Sertraline 100mg → Walgreens（due 11:00）
  - RX-20260605-048 David M. AX-2199 Lexapro 20mg → CVS（due 12:00）
  - RX-20260605-049 ... （共 12 个）
待跟进患者：
  - AX-3104 保险验证失败，需电话联系患者
  - AX-2890 预约确认未收到回复
今日预约：24 名患者
---
```

### 完整流程（逐步详解）

```
T=07:45  Alexis 感到不舒服，决定缺勤
         Alexis 私信 alexis-bot（在 RingEX DM）：

         ┌─────────────────────────────────────────────────┐
         │ Alexis Gonzalez [07:45] → alexis-bot            │
         │ 今天身体不舒服，无法来上班                       │
         │ 帮我处理今天的工作交接                           │
         └─────────────────────────────────────────────────┘

T=07:45  alexis-bot 处理：
         → 读 entity memory daily-queue-20260605.md：
           续剂 Task：12 个待处理
           待跟进患者：2 个
           今日预约：24 名患者
         → 读 DOMAIN.md：备用联系人

T=07:45  alexis-bot → ACTION:MESSAGE → #admin（诊所内部频道）：

         ════════════════════════════════════════
          alexis-bot [07:45]（来自 Alexis Gonzalez）
         ════════════════════════════════════════
         缺勤通知

         Alexis 今日缺勤（由 alexis-bot 代发）

         待处理工作量：
           · 续剂 Task：12 个（最近 due：11:00）
           · 待跟进患者：2 个
           · 今日预约：24 名患者

         正在寻找今日覆盖，请稍候…

T=07:46  alexis-bot → ACTION:MESSAGE → nursecoord-bot 监听频道（#admin 同一频道）：

         信号格式：
         "TASK_HANDOFF_REQUEST
          来源：alexis-bot（Alexis Gonzalez 缺勤）
          日期：2026-06-05
          任务摘要文件：daily-queue-20260605.md
          紧急程度：HIGH（最近 Task due 11:00）"

T=07:46  nursecoord-bot 检测到 TASK_HANDOFF_REQUEST
         → 读 daily-queue-20260605.md（共享 entity memory）
         → 读 DOMAIN.md：备用行政顺序

         nursecoord-bot → ACTION:SMS to=Jennifer +17205550301：
           "Hi Jennifer! Axis Mental Health needs admin coverage today.
            Alexis is out sick. 12 refill tasks, first due at 11am.
            Can you cover? Reply YES within 15 min. — Axis"

         （同时）nursecoord-bot → ACTION:SMS to=Karen +17205550312：
           "Hi Karen! Axis Mental Health needs admin help today.
            Alexis is out. Can you cover afternoon (1pm–6pm)?
            Reply YES within 15 min. — Axis"

T=07:51  Jennifer 回复 YES（6 分钟后，手机短信）

T=07:51  nursecoord-bot 检测到 YES 回复（Group B inbound SMS，或手动在 RC 转述）：
         → ACTION:SMS to=Jennifer：
           "✅ Confirmed! Please log in to RingEX for today's task list.
            First refill task due at 11am. Contact christopher@axismh.com
            if you have questions. Thank you!"
         → ACTION:SMS to=Karen：
           "Hi Karen, coverage arranged for today. Thank you!"

         → ACTION:MESSAGE → #admin：
         ┌──────────────────────────────────────────┐
         │ ✅ 行政覆盖已安排                         │
         │                                          │
         │ Alexis 缺勤 → Jennifer S. 今日全天       │
         │ 响应时间：6 分钟                          │
         │                                          │
         │ Jennifer 已收到今日任务清单               │
         │ 最近 Task due：11:00（RX-20260605-047）  │
         └──────────────────────────────────────────┘

         → ACTION:MESSAGE → alexis-bot：
           "HANDOFF_COMPLETE
            Jennifer S. 覆盖 2026-06-05 全天
            响应时间：6 分钟
            12 个 Task 已移交"

T=07:52  alexis-bot 收到 nursecoord-bot 的确认
         → ACTION:MESSAGE → Alexis DM：
           "✅ 交接完成
            Jennifer S. 已接手今日工作（07:52 确认）
            12 个续剂 Task + 2 个跟进已移交
            好好休息，祝早日康复 🙂"
```

**备用路径：Jennifer 和 Karen 均无回应（15 分钟）**

```
T=08:02  nursecoord-bot 检测到超时
         → ACTION:MESSAGE → #admin @Christopher Perez（CEO）：

         ┌──────────────────────────────────────────┐
         │ ⚠️ 需要你的决定                            │
         │                                          │
         │ Alexis 今日缺勤                           │
         │ 已联系 Jennifer（07:46）和 Karen（07:46） │
         │ 15 分钟无回应                             │
         │                                          │
         │ 待处理：12 个续剂 Task（最近 due 11:00）  │
         │ 今日预约：24 名患者                       │
         │                                          │
         │ [Message Jennifer] [Message Karen]       │
         │ [Mark as handled]                        │
         └──────────────────────────────────────────┘

         → alexis-bot → Alexis DM：
           "⚠️ Jennifer/Karen 未回应，已升级给 Christopher。
            请稍候，正在等待确认。"
```

### 关键技术细节

```
Q：alexis-bot 如何知道今日 Task 队列？

A：entity memory 共享机制
   clinical-bot 每次创建 ACTION:TASK 时，同时追加写入：
     memory/entities/daily-queue-YYYYMMDD.md

   alexis-bot 的 --dir 配置读取同一个 memory 目录
   （或者配置两个实例共享同一 memory 路径）

   对于 Demo：
   · 预置 daily-queue-20260605.md（12 个 Task 列表）
   · Scenario A 跑过后，文件里已经有 RX-20260605-047 的记录

Q：nursecoord-bot 怎么检测 TASK_HANDOFF_REQUEST？

A：nursecoord-bot 监听 #admin 频道
   其 SOUL.md 里有规则：
   "如果我在频道里看到 TASK_HANDOFF_REQUEST 开头的消息，
    按 DOMAIN.md 里的备用顺序发 SMS"

   这就是团队 Bot 的典型工作模式：
   等待信号 → 按规则执行
```

### 演讲词

```
"Alexis 告诉自己的 Bot：今天生病，帮我交接。

 一句话。

 alexis-bot 读了今天的任务队列——12 个续剂 Task——
 在 #admin 发了缺勤公告，
 然后向 nursecoord-bot 发了任务转移信号。

 [展示 #admin 里的缺勤通知]

 nursecoord-bot 同时给 Jennifer 和 Karen 发了手机短信。

 Jennifer 6 分钟回复 YES。

 [展示覆盖确认 Card]

 nursecoord-bot 确认覆盖，通知 #admin，
 alexis-bot 向 Alexis 报告：Jennifer 已接手，祝早日康复。

 这里有三层 Bot 协作：
 Alexis 的个人 Bot，行政协调的团队 Bot，
 还有 clinical-bot 之前写入的任务队列。

 如果 Jennifer 和 Karen 都没回，
 nursecoord-bot 会直接升级给 CEO Christopher Perez，
 Card 里有今天的工作量和 Task 数量。
 Christopher 知道该怎么决定，Bot 帮他看到全局。"
```

---

## Demo 时序安排（5 分钟）

```
0:00–0:30  RingEX Bot 界面展示
           "Andrew 在 RingEX 里创建了 andrew-bot，只有他能和它说话。
            临床团队创建了 clinical-bot，频道里所有人可以触发。"

0:30–1:45  Scenario A — clinical-bot 续剂路由
           · Alexis @clinical-bot 一行输入
           · Card 弹出 @Andrew
           · Andrew 点批准，链式执行
           · "6 分钟，原来 45 分钟"

1:45–3:00  Scenario B — andrew-bot 医生主动协调
           · Andrew DM andrew-bot：通知 Michelle
           · andrew-bot 在频道发协调通知
           · Michelle 自然语言回复
           · andrew-bot 检测临床反馈，推到 Andrew DM
           · "Andrew → Michelle → Andrew，闭环，无人中转"

3:00–4:15  Scenario C — alexis-bot 缺勤交接
           · Alexis DM alexis-bot：今天缺勤
           · alexis-bot → nursecoord-bot 任务转移
           · nursecoord-bot SMS 候补 → 6 分钟确认
           · "告诉 Bot 一句话，任务交接完成"

4:15–5:00  价值主张
```

---

## 数据文件清单（Demo 前准备）

```
clinical-bot --dir ~/.ringclaw-axis-clinical/
├── SOUL.md          续剂路由逻辑
├── DOMAIN.md        提供者名单 + RC user ID
└── memory/entities/
    ├── AX-2847.md   Maria Lopez（续剂 Demo 患者）
    ├── AX-3201.md   James T.（爽约 Demo 患者，备用）
    └── daily-queue-20260605.md   今日 Task 队列

andrew-bot --dir ~/.ringclaw-axis-andrew/
├── SOUL.md          Andrew 的个人助理规则
├── DOMAIN.md        可选（提供者联系信息）
└── memory/entities/
    ├── AX-1956.md   双侧提供者患者（Andrew + Michelle）
    └── AX-2103.md   备用

alexis-bot --dir ~/.ringclaw-axis-alexis/
├── SOUL.md          Alexis 的行政助理规则
├── DOMAIN.md        备用行政联系人
└── memory/entities/
    └── daily-queue-20260605.md   （与 clinical-bot 共享或预置）

nursecoord-bot --dir ~/.ringclaw-axis-nursecoord/
├── SOUL.md          任务转移 + SMS 覆盖规则
└── DOMAIN.md        备用行政手机号 + 诊所经理联系方式
```
