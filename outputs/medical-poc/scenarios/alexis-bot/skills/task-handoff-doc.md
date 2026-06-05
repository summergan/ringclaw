# Skill: task-handoff-doc
# alexis-bot · Alexis Gonzalez 个人技能
# 触发：缺勤交接时（Scenario C 核心 skill），或主动生成交接文档

---

## 用途

生成结构化的缺勤交接文档，让替班人员（Jennifer / Karen）能快速接手工作。

覆盖两种场景：
1. **缺勤时自动生成**（与 Scenario C 主流程配合，作为 SMS 中的补充信息）
2. **主动提前生成**（如 Alexis 知道明天休假，提前准备交接文档）

---

## 触发方式

```
自动触发（缺勤交接时，alexis-bot 在发 TASK_HANDOFF_REQUEST 信号后自动生成）：
  → 生成文档后发给替班人员 DM（或 post 到 #admin）

手动触发：
  Alexis DM alexis-bot：
  "skill task-handoff-doc"
  "生成明天的交接文档"（提前准备）
```

---

## 输出格式

发到 #admin，或 DM 给替班人员（Jennifer / Karen）：

```
📋 行政交接文档 · 2026-06-05
交接人：Alexis Gonzalez → Jennifer S.

──────────────────────────────
1. 续剂 Task（11 个待处理）

  优先处理：
  ⚠️ RX-052 AX-1892 Carlos M. — Adderall XR 20mg（CONTROLLED）
     药房：Walgreens Aurora · due 13:00
     处理方式：登录 athenahealth 发送处方至药房

  常规续剂（按 due 时间顺序）：
  · RX-048 AX-2199 David M. — Lexapro 20mg → CVS Aurora  due 12:00
  · RX-049 AX-1103 Sarah K. — Wellbutrin 150mg → King Soopers  due 12:00
  · RX-050 AX-3044 Tom R.   — Lamictal 200mg → Walgreens  due 13:00
  · RX-053 AX-3201 James T. — Lexapro 10mg → CVS  due 14:00
  · （其余 6 个见 RingEX Task 列表）

──────────────────────────────
2. 患者跟进（2 个）

  · AX-3104 Jennifer W. — 保险验证失败（United Health）
    电话：+17205550144  期限：15:00 前
    处理：致电患者确认新保险卡号，更新 athenahealth

  · AX-2890 Michael S. — 预约确认未收到回复
    电话：+17205550133  期限：12:00 前
    处理：发 SMS 或电话确认 2026-06-10 的预约

──────────────────────────────
3. 今日预约（24 名患者，参考）

  已由各提供者处理。如有患者电话询问，转接到对应提供者。
  诊所电话：+17205550100

──────────────────────────────
4. 操作说明

  续剂处理步骤：
  1. 打开 RingEX → Task 列表（找到分配给 Alexis 的续剂 Task）
  2. 打开 athenahealth → 患者档案 → 发送处方至药房
  3. 在 RingEX Task 里标记 completed
  4. 发短信给患者（SMS 草稿在 Task 备注里）

  遇到问题联系：
  · 临床问题 → @Andrew Wenner 或对应提供者
  · 系统问题 → Amie Naas（axismh-anaas）
  · 紧急情况 → Christopher Perez（CEO，axismh-cperez）
──────────────────────────────

Jennifer 如有问题随时在 #admin 里 @nursecoord-bot 或联系 Amie。
祝 Alexis 早日康复！
```

---

## SOUL.md 调用方式

```
当 Scenario C 缺勤流程进行到"nursecoord-bot 发出确认覆盖"时：
  → alexis-bot 自动调用 task-handoff-doc skill
  → 生成交接文档
  → ACTION:MESSAGE → #admin（替班人员可见）
  → 同时将文档内容包含在给 Alexis 的汇报 DM 中

手动触发时：
1. 读取 daily-queue-今日.md
2. 识别未完成 Task 和待跟进患者
3. 生成优先级排序的交接文档
4. 询问 Alexis："发给谁？Jennifer / Karen / #admin？"
5. ACTION:MESSAGE → 指定目标
```
