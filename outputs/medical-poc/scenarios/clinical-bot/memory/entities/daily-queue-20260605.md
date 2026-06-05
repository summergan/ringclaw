# 今日行政任务队列
# 日期：2026-06-05
# 由 clinical-bot 在创建 Task 时自动追加
# alexis-bot 和 nursecoord-bot 读取此文件

---

## 元数据

日期：2026-06-05
行政负责人：Alexis Gonzalez（axismh-alexis）
今日预约总量：24 名患者
最后更新：09:05

---

## 续剂 Task 队列

| 序号 | 续剂编号 | 患者ID | 患者姓名 | 药物 | 药房 | Due 时间 | 状态 |
|------|---------|--------|---------|------|------|---------|------|
| 01 | RX-20260605-047 | AX-2847 | Maria Lopez | Sertraline 100mg | Walgreens Aurora | 11:00 | ✅ Andrew 已批准 |
| 02 | RX-20260605-048 | AX-2199 | David M. | Lexapro 20mg | CVS Aurora | 12:00 | ⏳ 待提供者审批 |
| 03 | RX-20260605-049 | AX-1103 | Sarah K. | Wellbutrin 150mg | King Soopers | 12:00 | ⏳ 待提供者审批 |
| 04 | RX-20260605-050 | AX-3044 | Tom R. | Lamictal 200mg | Walgreens Aurora | 13:00 | ⏳ 待提供者审批 |
| 05 | RX-20260605-051 | AX-2571 | Amy L. | Zoloft 50mg | CVS Aurora | 13:00 | ⏳ 待提供者审批 |
| 06 | RX-20260605-052 | AX-1892 | Carlos M. | Adderall XR 20mg ⚠️CONTROLLED | Walgreens Aurora | 13:00 | ⏳ 待提供者审批 |
| 07 | RX-20260605-053 | AX-3201 | James T. | Lexapro 10mg | CVS Aurora | 14:00 | ⏳ 待提供者审批 |
| 08 | RX-20260605-054 | AX-2033 | Lisa P. | Effexor 75mg | King Soopers | 14:00 | ⏳ 待提供者审批 |
| 09 | RX-20260605-055 | AX-1445 | Ryan B. | Prozac 40mg | Walgreens Aurora | 15:00 | ⏳ 待提供者审批 |
| 10 | RX-20260605-056 | AX-2788 | Emma D. | Lexapro 10mg | CVS Aurora | 15:00 | ⏳ 待提供者审批 |
| 11 | RX-20260605-057 | AX-3102 | Marcus H. | Sertraline 50mg | King Soopers | 16:00 | ⏳ 待提供者审批 |
| 12 | RX-20260605-058 | AX-1667 | Diana F. | Wellbutrin 300mg | Walgreens Aurora | 16:00 | ⏳ 待提供者审批 |

**总计：12 个续剂 Task**（1 个已完成，11 个待处理）

---

## 待跟进患者

| 患者ID | 患者姓名 | 事由 | 联系方式 | 跟进期限 |
|--------|---------|------|---------|---------|
| AX-3104 | Jennifer W. | 保险验证失败（United Health），需电话确认新卡号 | +17205550144 | 今日 15:00 前 |
| AX-2890 | Michael S. | 发送了预约确认 SMS，24h 未回复，需跟进确认 | +17205550133 | 今日 12:00 前 |

---

## 今日预约统计（截至 09:00）

总预约数：24
已签到：6
未到：1（AX-3201 James T. 10:00am，已发 SMS）
待签到：17

---

## 文件使用说明

此文件由 clinical-bot 在以下时机写入：
- 每次创建 ACTION:TASK 给 Alexis 时，追加到续剂队列
- 每次收到爽约通知时，更新预约统计

此文件由以下 Bot 读取：
- alexis-bot：缺勤时读取工作量，用于交接
- nursecoord-bot：接收任务转移时，了解具体任务内容
