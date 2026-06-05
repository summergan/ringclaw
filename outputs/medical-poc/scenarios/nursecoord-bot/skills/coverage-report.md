# Skill: coverage-report
# nursecoord-bot · 团队技能
# 触发：Amie Naas 或管理层在 #admin 发起，或月末自动生成

---

## 用途

生成行政覆盖情况的周期性报告，帮助管理层：
- 了解缺勤频率和覆盖成功率
- 评估备用行政人员的响应速度和可靠性
- 识别哪些时段最容易出现覆盖困难
- 决定是否需要扩充备用人员池

---

## 触发方式

```
在 #admin：
@nursecoord-bot skill coverage-report
@nursecoord-bot coverage-report this-week
@nursecoord-bot coverage-report 2026-06
```

---

## 数据来源

从 `memory/entities/coverage-log.md` 读取历史覆盖记录。

---

## 输出格式

发到 #admin 并 DM 给 Amie Naas：

```
📊 行政覆盖报告 · 2026年6月（截至 06/05）

总览
  缺勤事件：2 次
  成功覆盖：2 次（100%）
  平均响应：8 分钟
  升级至管理层：0 次

覆盖详情
  ┌─────────────────────────────────────────────┐
  │ 日期    缺勤人员   覆盖人员   响应时间  结果 │
  │ 06/03  Alexis     Jennifer   11 分钟   ✅  │
  │ 06/05  Alexis     Jennifer    6 分钟   ✅  │
  └─────────────────────────────────────────────┘

备用人员表现
  Jennifer S.  被联系 2 次  响应 2 次  平均响应 8.5 分钟  ⭐ 可靠
  Karen M.     被联系 1 次  未响应     —                  ⚠️ 需关注

建议
  · Jennifer 响应速度良好，建议列为首选联系人（当前已是）
  · Karen 本月 1 次联系未响应，建议确认当前可用性
  · 考虑补充第 3 位全天备用行政人员（目前仅 Jennifer 全天可用）
```

---

## SOUL.md 调用方式

```
当收到 "skill coverage-report" 时：
1. 读取 coverage-log.md 中的历史记录
2. 统计：覆盖次数、响应时间、成功率、升级次数
3. 按备用人员统计表现
4. 生成建议（基于数据规则：响应率<80% 或响应时间>20min 标注 ⚠️）
5. ACTION:MESSAGE → #admin
6. ACTION:MESSAGE → Amie Naas DM（axismh-anaas）
```
