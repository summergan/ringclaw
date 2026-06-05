# alexis-bot · 今日任务队列镜像
# 日期：2026-06-05
# 来源：clinical-bot 共享数据（或 Alexis 手动汇报）
# 用途：缺勤时 alexis-bot 读取此文件生成交接摘要

---

此文件内容与 clinical-bot 的 daily-queue-20260605.md 保持同步。
（Demo 中两份文件内容相同，由 alexis-bot 在自己的 --dir 下预置）

---

## 当前工作量摘要

| 类别 | 数量 | 最近 Due |
|------|------|---------|
| 续剂 Task（待处理） | 11 | 12:00（RX-20260605-048）|
| 续剂 Task（已完成）| 1 | — |
| 待跟进患者 | 2 | AX-3104 今日 15:00 前 |
| 今日预约总量 | 24 | — |

---

## 待处理续剂 Task 列表

| 续剂编号 | 患者ID | 药物 | 药房 | Due 时间 | 备注 |
|---------|--------|------|------|---------|------|
| RX-20260605-048 | AX-2199 | Lexapro 20mg | CVS Aurora | 12:00 | |
| RX-20260605-049 | AX-1103 | Wellbutrin 150mg | King Soopers | 12:00 | |
| RX-20260605-050 | AX-3044 | Lamictal 200mg | Walgreens | 13:00 | |
| RX-20260605-051 | AX-2571 | Zoloft 50mg | CVS | 13:00 | |
| RX-20260605-052 | AX-1892 | Adderall XR 20mg | Walgreens | 13:00 | ⚠️ CONTROLLED |
| RX-20260605-053 | AX-3201 | Lexapro 10mg | CVS | 14:00 | |
| RX-20260605-054 | AX-2033 | Effexor 75mg | King Soopers | 14:00 | |
| RX-20260605-055 | AX-1445 | Prozac 40mg | Walgreens | 15:00 | |
| RX-20260605-056 | AX-2788 | Lexapro 10mg | CVS | 15:00 | |
| RX-20260605-057 | AX-3102 | Sertraline 50mg | King Soopers | 16:00 | |
| RX-20260605-058 | AX-1667 | Wellbutrin 300mg | Walgreens | 16:00 | |

---

## 待跟进患者

| 患者ID | 事由 | 联系方式 | 期限 |
|--------|------|---------|------|
| AX-3104 | 保险验证失败（United Health），需确认新卡号 | +17205550144 | 今日 15:00 前 |
| AX-2890 | 预约确认 SMS 24h 未回复 | +17205550133 | 今日 12:00 前 |

---

## 交接说明（缺勤时 nursecoord-bot 参考）

1. 优先处理 CONTROLLED 药物 Task（RX-20260605-052，Adderall XR）
2. 12:00 前处理 AX-2890 的预约确认跟进
3. 15:00 前处理 AX-3104 保险问题（需电话联系患者）
4. 其余续剂 Task 按 due 时间顺序处理即可
