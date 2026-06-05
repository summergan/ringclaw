# Axis Integrated Mental Health · 临床知识库
# clinical-bot DOMAIN.md
# 更新：2026-06-05

---

## 诊所信息

诊所名称：Axis Integrated Mental Health
官网：axismh.com
主号码（Aurora）：+17205550100
患者门户：21564-1.portal.athenahealth.com

诊所地址：
  Aurora（主）：1444 S Potomac St, Suite 220, Aurora, CO 80012
  Westminster：Westminster, CO
  Louisville：Louisville, CO（对外标注 Boulder）
  Denver Tech Center：6950 E Belleview Ave, Suite 300, Denver, CO 80111

---

## 提供者名单（精神科 · 开药）

| 姓名 | 职称 | RC user ID | 专长 |
|------|------|-----------|------|
| Andrew Wenner | PMHNP | axismh-awenner | 抑郁、焦虑、ADHD |
| Madison Walter | PMHNP | axismh-mwalter | 双相、创伤、焦虑 |
| Elinor O'Buckley | PMHNP | axismh-eobuckley | 成人精神科 |
| Sarah McLaughlin | PMHNP | axismh-smclaughlin | 青少年、焦虑 |
| Frani Rhodes | PMHNP | axismh-frhodes | 成人精神科 |
| Ross Van Allen | DNP | axismh-rvanallen | 成人精神科 |
| Kayla Sharpe | PMHNP | axismh-ksharpe | 成人精神科 |
| Ben Egbers | PMHNP | axismh-begbers | 成人精神科 |
| Carrie Karr | PMHNP | axismh-ckarr | 成人精神科 |
| Roderick O'Brien | MD | axismh-robrien | 主治医生，复杂病例 |
| Ebony Villarreal | PA-C | axismh-evillarreal | 成人精神科 |
| Mary Chamberlain | PA-C | axismh-mchamberlain | 成人精神科 |
| Katerina Krieger | PA-C | axismh-kkrieger | 成人精神科 |

---

## 提供者名单（治疗师 · 心理疗程）

| 姓名 | 职称 | RC user ID | 专长 |
|------|------|-----------|------|
| Michelle Godwin | LPC | axismh-mgodwin | CBT、抑郁、焦虑 |
| Tori Kause | LPC | axismh-tkause | 创伤、青少年 |
| Lisa Mazzola | LPC | axismh-lmazzola | 成人、焦虑 |
| Dez Nunez | LPC | axismh-dnunez | 成人、关系咨询 |
| Laura Jane Landis | LCSW | axismh-ljlandis | 成人、抑郁 |

---

## 行政团队

| 姓名 | 角色 | RC user ID | 手机 |
|------|------|-----------|------|
| Alexis Gonzalez | 行政协调员 | axismh-alexis | +17205550150 |
| Amie Naas | Director of Clinical Operations | axismh-anaas | +17205550151 |

---

## 领导层

| 姓名 | 职位 | RC user ID |
|------|------|-----------|
| Christopher Perez | CEO & Co-founder | axismh-cperez |
| Liesl Perez | CMO / Chief Growth Officer & Co-founder | axismh-lperez |
| Dr. Kartiki Churi | Chief Medical Officer（2026-04 任命） | axismh-kchuri |
| Mike Emerson | COO（2026-04 任命） | axismh-memerson |

---

## 合作药房

| 药房 | 地址 | 电话 |
|------|------|------|
| Walgreens Aurora | 1234 S Havana St, Aurora, CO | +17205550200 |
| CVS Aurora | 5678 S Peoria St, Aurora, CO | +17205550201 |
| King Soopers Pharmacy | 900 S Sable Blvd, Aurora, CO | +17205550202 |

---

## 保险（接受的主要 in-network 保险）

- Medicaid（Colorado）
- Anthem Blue Cross Blue Shield
- Aetna
- Cigna
- United Healthcare
- Tricare

注：PoC 演示中保险状态预置在 entity memory，不进行实时验证。

---

## 续剂规则

续剂窗口判断标准（仅供 Card 展示参考，不替代提供者临床判断）：
- 上次续剂 ≥ 30 天：✅ 合理，正常路由
- 上次续剂 < 30 天：⚠️ 间隔较短，Card 上标注，提供者自行判断
- 距上次就诊 > 90 天：⚠️ 超过 90 天未就诊，Card 上标注建议先预约

精神科药物续剂需要提供者批准（不可自动处理，必须生成 Card 等待点击）：
- Controlled substances（Adderall、Vyvanse、Klonopin 等）：强制标注 ⚠️ CONTROLLED
- Non-controlled（Sertraline、Lexapro、Wellbutrin 等）：标准流程

---

## 频道配置

```
#clinical-coordination：clinical-bot 主频道
  · 成员：所有提供者 + 行政人员
  · 用途：续剂路由、跨提供者协调通知

#admin：行政频道
  · 成员：Alexis、Amie Naas、管理层
  · 用途：行政 Task 协调、覆盖安排
```

---

## 续剂编号格式

RX-YYYYMMDD-NNN

示例：
  RX-20260605-001（今日第 1 个请求）
  RX-20260605-047（今日第 47 个请求）

每日重置，由 clinical-bot 自动递增（从 entity memory 读取今日最大序号）。
