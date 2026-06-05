# Skill: daily-refill-report
# clinical-bot · 团队技能
# 触发：Alexis 或管理层在 #clinical-coordination 发起，或 Heartbeat 定时触发

---

## 用途

生成当日续剂请求处理情况的结构化报告，发给 Liesl Perez（CEO/CGO）和 Amie Naas（Director of Clinical Ops）。

帮助管理层每天知道：
- 今天处理了多少续剂，处理速度如何
- 有没有积压，谁的患者被拒了
- 是否需要注意某个提供者的审批速度

---

## 触发方式

手动触发：
```
@clinical-bot skill daily-refill-report
@clinical-bot 生成今日续剂报告
```

定时触发（Heartbeat 配置）：
```
每日 17:30 自动生成，发给 Liesl DM
```

---

## 数据来源

从 `memory/entities/daily-queue-YYYYMMDD.md` 读取：
- 总 Task 数量
- 各状态数量（已完成 / 待处理 / 被拒绝）
- 各提供者审批数量
- 最快/最慢审批时间

---

## 输出格式

发到 #clinical-coordination 并 DM 给 Liesl：

```
📊 续剂日报 · 2026-06-05

今日总计
  请求：18 个  批准：16 个  需复诊：1 个  拒绝：1 个
  平均处理：23 分钟  目标：<60 分钟 ✅

按提供者
  Andrew Wenner    10 个  全部批准 ✅  均值 18min
  Madison Walter   5 个   全部批准 ✅  均值 25min
  Elinor O'Buckley 3 个   2 批准 / 1 需复诊

待跟进
  ⏳ RX-20260605-052 Carlos M. Adderall XR — 待 Alexis 发送至药房（due 13:00）

⚠️ 需关注
  AX-3201 James T. 今日爽约，空档未填上
  AX-1892 Adderall XR 未处理（CONTROLLED，请 Alexis 优先跟进）
```

---

## SOUL.md 调用方式

```
当收到 "skill daily-refill-report" 触发时：
1. 读取今日 daily-queue 文件
2. 统计各状态数量
3. 计算平均处理时间
4. 识别未完成的 CONTROLLED 药物 Task（标记为 ⚠️）
5. 生成结构化报告
6. ACTION:MESSAGE → #clinical-coordination
7. ACTION:MESSAGE → Liesl DM（axismh-lperez）
```
