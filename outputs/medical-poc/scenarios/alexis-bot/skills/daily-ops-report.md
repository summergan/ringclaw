# Skill: daily-ops-report
# alexis-bot · Alexis Gonzalez 个人技能
# 触发：Alexis 下班前发起，或 Heartbeat 17:00 自动触发

---

## 用途

生成 Alexis 当日行政工作的结构化总结，发给 Amie Naas（Director of Clinical Ops）和 Christopher Perez（CEO）。

让管理层每天掌握：
- 续剂处理完成情况
- 患者跟进状态
- 积压和异常项
- 明日需要关注的事项

---

## 触发方式

手动：
```
alexis-bot 发起，或 Alexis DM alexis-bot：
"skill daily-ops-report"
"生成今天的运营报告"
```

自动（Heartbeat 17:00）：
```
每个工作日 17:00 自动生成并发送
```

---

## 数据来源

从 `memory/entities/daily-queue-YYYYMMDD.md` 读取当日数据。

---

## 输出格式

发给 Amie Naas DM + Alexis DM（存档）：

```
📋 行政日报 · 2026-06-05
汇报人：Alexis Gonzalez

续剂处理
  总计：12 个
  ✅ 完成：10 个（平均处理 28 分钟）
  ⏳ 待处理：2 个（明日优先跟进）
  特殊项：RX-052 Adderall XR（CONTROLLED）✅ 已完成

患者跟进
  ✅ AX-2890 预约确认 — 患者已回复确认
  ⚠️ AX-3104 保险问题 — 患者未接电话，明日再试

今日预约
  完成：24 个  爽约：1 个（AX-3201 James T.，已处理）
  等位患者填缺：空档未填上（等位名单无今日可用）

明日待办
  · AX-3104 保险验证跟进（United Health，优先）
  · RX-20260605-048/049 两个续剂待发送至药房
  · Andrew 提到 AX-2847 可能需要预约复诊（待 Alexis 安排）

今日亮点
  · RX-047 Maria Lopez 续剂 6 分钟处理完成 ✅
  · 无积压超 4 小时的续剂请求
```

---

## SOUL.md 调用方式

```
当收到 "skill daily-ops-report" 时：
1. 读取 daily-queue-今日.md
2. 统计续剂各状态数量和平均处理时间
3. 读取待跟进患者状态（已跟进 / 未完成）
4. 生成结构化报告
5. ACTION:MESSAGE → Amie Naas DM（axismh-anaas）
6. ACTION:MESSAGE → Alexis DM（存档副本）
7. 回复 Alexis："✅ 日报已发给 Amie"
```
