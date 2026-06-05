# Skill: cross-provider-brief
# andrew-bot · Andrew Wenner 个人技能
# 触发：Andrew 在 DM 里请求为某患者生成提供者简报

---

## 用途

Andrew 在以下情况需要向治疗师（Michelle 等）提供结构化的患者临床背景：
- 新患者转介给治疗师时（Andrew 开始治疗，建议同步心理疗程）
- 用药有较大变化时（增减药物、更换类别）
- 患者要求同时接受药物和疗程治疗时

目前靠口头或邮件，信息不完整，治疗师没有足够背景就开始疗程。

---

## 触发方式

```
Andrew DM andrew-bot：

skill cross-provider-brief AX-2847
生成 AX-2847 的提供者简报，发给 Michelle

cross-provider-brief AX-1956 for Tori Kause
```

---

## 数据来源

从 `memory/entities/<患者ID>.md` 读取：
- 患者基本信息（ID，开药医生）
- 当前用药和历史调整
- 最近就诊记录和临床判断
- 诊断方向（Andrew 自己更新的备注）

---

## 输出格式

andrew-bot 在 #clinical-coordination 发（署名 andrew-bot，来自 Andrew Wenner）：

```
📋 患者临床简报 · AX-2847
（由 Andrew Wenner, PMHNP 提供，供治疗师参考）
生成时间：2026-06-05 14:00

──────────────────────────────
基本情况
  患者 ID：AX-2847
  开药医生：Andrew Wenner, PMHNP
  初诊日期：2026-03-15
  诊断方向：中度抑郁，无双相特征，社交焦虑

当前用药
  Sertraline 100mg · 每日一次（2026-03-15 起）
  注：从 50mg 开始，2026-06-05 增至 100mg

用药调整历史
  2026-04-01  初始 50mg，观察期
  2026-06-05  增至 100mg，原因：抑郁症状改善不足

最近就诊摘要（2026-05-28）
  患者自述：睡眠有所改善，焦虑仍明显
  临床判断：继续当前方向，增量 Sertraline

对治疗师的建议
  · 关注 SSRI 增量后的焦虑波动（初期 2-4 周内常见）
  · 如患者提到心悸或睡眠变化，请通知 Andrew
  · 建议疗程方向：CBT 焦虑管理 + 行为激活
──────────────────────────────

@Michelle Godwin 如需更多临床背景，请回复此消息或联系 Andrew 直接沟通。
```

---

## SOUL.md 调用方式

```
当收到 "skill cross-provider-brief <患者ID>" 时：
1. 读取 entity memory <患者ID>.md
2. 提取：诊断方向、当前用药、调整历史、最近就诊备注
3. 识别接收治疗师（如未指定，从 entity memory 读默认治疗师）
4. 生成结构化简报
5. ACTION:MESSAGE → #clinical-coordination @治疗师
6. 向 Andrew DM 确认："✅ 简报已发给 [治疗师名]"
```

---

## 注意

- 简报中不包含患者姓名（仅使用 ID），符合 HIPAA 示范规范
- 不包含患者手机号或保险信息
- 所有内容基于 Andrew 已知的临床事实，不做推测
