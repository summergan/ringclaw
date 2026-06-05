# nursecoord-bot · DOMAIN.md
# 行政协调知识库
# 更新：2026-06-05

---

## 诊所信息

名称：Axis Integrated Mental Health，Aurora 诊所
工作时间：Monday–Friday 8:00am–6:00pm
诊所电话：+17205550100

---

## 备用行政人员（联系顺序）

| 优先级 | 姓名 | 手机号 | 可用班次 | 备注 |
|--------|------|--------|---------|------|
| 1 | Jennifer S. | +17205550301 | 全天（8am–6pm）| 熟悉全部续剂流程，首选 |
| 2 | Karen M. | +17205550312 | 下午（1pm–6pm）| 只能覆盖下午，续剂技能良好 |
| 3 | 联系 Amie Naas 协调 | axismh-anaas | — | Director of Ops，有权临时调配 |

注：Jennifer 和 Karen 均优先同时发 SMS，加快响应速度。

---

## 升级链

```
nursecoord-bot 发 SMS 给 Jennifer + Karen
  ↓ 15 分钟无回应
升级给 Christopher Perez（CEO）
  RC user ID：axismh-cperez
  ↓ 同时通知 Amie Naas（Director of Clinical Ops）
  RC user ID：axismh-anaas
```

---

## OT 政策

紧急覆盖 OT 费率：标准小时工资 × 1.5
全天覆盖估算：8 小时 × [时薪] × 1.5
（具体费率由 HR 政策决定，nursecoord-bot 在 SMS 中不提具体金额）

---

## 频道配置

#admin chat ID：（部署时填入）
Christopher Perez DM chat ID：（部署时填入）

---

## 信号格式（与 alexis-bot 约定）

接收信号：
  TASK_HANDOFF_REQUEST — alexis-bot 发出的任务转移请求

发出信号：
  HANDOFF_COMPLETE — 覆盖已安排完成
  ESCALATED_TO_CEO — 已升级给 CEO

---

## 交接注意事项

续剂任务中，以下类型需要特别告知替班人员：
  ⚠️ CONTROLLED — 管控药物（Adderall、Klonopin 等），需要特别核对流程
  急诊跟进患者 — 当日 due 的患者跟进，优先处理
