# alexis-bot · DOMAIN.md
# Alexis Gonzalez 的行政知识库
# 更新：2026-06-05

---

## 我的信息

姓名：Alexis Gonzalez
角色：行政协调员
所属：Axis Integrated Mental Health，Aurora 诊所
RC user ID：axismh-alexis

---

## 备用行政人员（缺勤时联系顺序）

| 顺序 | 姓名 | 手机号 | 可用性 | 备注 |
|------|------|--------|--------|------|
| 1 | Jennifer S. | +17205550301 | 全天 | 熟悉续剂流程，优先联系 |
| 2 | Karen M. | +17205550312 | 下午（13:00–18:00）| 只能覆盖下午班 |
| 3 | （联系诊所经理） | — | — | 以上两人均不可时升级 |

---

## 联系升级链

```
Alexis 缺勤
  → 尝试 Jennifer（全天）
  → 如 Jennifer 无回应，尝试 Karen（下午）
  → 如均无回应，升级给 Christopher Perez（CEO）
```

---

## 重要联系人

| 姓名 | 角色 | RC user ID | 用途 |
|------|------|-----------|------|
| Christopher Perez | CEO | axismh-cperez | 最终升级联系人 |
| Amie Naas | Director of Clinical Ops | axismh-anaas | 临床运营问题 |

---

## 常用频道

| 频道名 | 用途 |
|--------|------|
| #admin | 行政团队内部协调，发布缺勤通知 |
| #clinical-coordination | 临床团队，我通常不发消息（由 clinical-bot 处理）|

---

## 工作时间

诊所工作时间：Monday–Friday 8:00am–6:00pm
续剂 Task 最晚处理时间：收到 Task 后 4 小时内（目标）
缺勤通知触发时间：越早越好（8am 前最佳）

---

## 我与 clinical-bot 的数据共享

clinical-bot 每次创建 Alexis 的 Task 时，同时更新：
  `memory/entities/daily-queue-<YYYYMMDD>.md`

我读取此文件来了解今日工作量。

如果 clinical-bot 和 alexis-bot 运行在不同 --dir 实例，
需要配置 daily-queue 文件路径为共享位置，或由 Alexis 手动汇报任务量。
