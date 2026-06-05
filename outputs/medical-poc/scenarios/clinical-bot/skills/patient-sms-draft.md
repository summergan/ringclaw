# Skill: patient-sms-draft
# clinical-bot · 团队技能
# 触发：Alexis 或任何频道成员请求起草患者 SMS

---

## 用途

根据情景起草发给患者的 SMS 草稿，供人工确认后发送。

覆盖场景：
- 续剂确认（批准/被拒）
- 预约提醒（复诊、新患者）
- 爽约跟进
- 保险问题通知
- 一般患者沟通

---

## 触发方式

```
@clinical-bot skill patient-sms-draft <场景> <患者ID>

示例：
@clinical-bot skill patient-sms-draft refill-approved AX-2847
@clinical-bot skill patient-sms-draft appointment-reminder AX-3201
@clinical-bot skill patient-sms-draft no-show AX-3201
@clinical-bot skill patient-sms-draft insurance-issue AX-3104
```

---

## 输出格式（发到频道供人工确认）

```
📱 SMS 草稿 · AX-2847 Maria Lopez

场景：续剂已批准

---
Hi Maria! Dr. Wenner has approved your Sertraline 100mg refill.
Your prescription has been sent to Walgreens Aurora (1234 S Havana St).
It should be ready for pickup in 2–4 hours.
Questions? Reply to this message. — Axis Mental Health
---

字数：148  ✅ 单条 SMS 范围

[确认发送]  [修改后发送]  [取消]
```

---

## 模板库（SOUL.md 内置）

```
refill-approved：
  "Hi [Name]! Dr. [Provider] has approved your [药物] refill.
   Sent to [药房]. Ready in 2–4 hours. — Axis Mental Health"

refill-needs-visit：
  "Hi [Name]! Dr. [Provider] would like a follow-up visit before renewing.
   Our office will contact you today. — Axis Mental Health"

refill-denied：
  "Hi [Name]! Dr. [Provider] reviewed your request.
   Please call [诊所电话] to discuss next steps. — Axis Mental Health"

appointment-reminder：
  "Hi [Name]! Reminder: you have an appointment with [Provider]
   on [日期] at [时间] at our [诊所] office.
   Reply CONFIRM or call [诊所电话]. — Axis Mental Health"

no-show：
  "Hi [Name]! We noticed you missed your [时间] appointment with [Provider].
   Still coming? Reply YES to confirm or NO to reschedule.
   — Axis Mental Health"

insurance-issue：
  "Hi [Name]! We need to verify your insurance information.
   Please call us at [诊所电话] at your earliest convenience.
   — Axis Mental Health"

waitlist-opening：
  "Hi [Name]! We have an opening today at [时间] with [Provider].
   Interested? Reply YES to confirm (offer expires in 30 min).
   — Axis Mental Health"
```

---

## 规则

- 所有草稿必须经人工确认后才能发出（clinical-bot 不直接发患者 SMS，除非在续剂批准的链式执行中）
- 字数超过 160 字符时标注 ⚠️（会被分成多条 SMS）
- 不得在草稿中包含具体剂量或医疗建议
