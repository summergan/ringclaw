# Skill: post-visit-note
# andrew-bot · Andrew Wenner 个人技能
# 触发：Andrew 就诊结束后，口述给 Bot，Bot 生成结构化记录

---

## 用途

Andrew 就诊结束后，用自然语言告诉 andrew-bot 发生了什么，
Bot 将其整理成结构化的就诊备注并更新 entity memory。

解决的问题：
- Andrew 就诊后需要手动更新系统，费时且容易遗忘
- entity memory 的患者档案需要保持最新，以便后续续剂路由时 clinical-bot 有准确信息
- 跨提供者协调需要有记录的就诊摘要

---

## 触发方式

```
Andrew DM andrew-bot：

skill post-visit-note AX-2847
今天见了 Maria，她说睡眠好了一些，但焦虑还在。
我把 Sertraline 从 50 增到 100mg，原因是抑郁改善不足。
下次 4 周后见。

或更简短：
AX-2847 今天就诊，Sertraline 50→100mg，原因：抑郁改善不足，下次 4 周
```

---

## 输出格式

andrew-bot 整理后先给 Andrew 确认：

```
📝 就诊备注草稿 · AX-2847

就诊日期：2026-06-05
提供者：Andrew Wenner, PMHNP

患者反馈
  · 睡眠有所改善
  · 焦虑仍明显

临床处置
  · Sertraline 50mg → 100mg（原因：抑郁症状改善不足）

后续计划
  · 下次就诊：约 4 周后（预计 2026-07-03 前后）

──────────────────
确认后我会更新 AX-2847 档案。
如需修改请直接告诉我。

[确认更新] [修改]
```

Andrew 确认后，andrew-bot 更新 entity memory AX-2847.md：
```
就诊记录追加（2026-06-05）：
  提供者：Andrew Wenner
  患者反馈：睡眠改善，焦虑仍明显
  处置：Sertraline 50mg→100mg（抑郁改善不足）
  下次：约 4 周后
```

---

## SOUL.md 调用方式

```
当收到 "skill post-visit-note <患者ID>" 后接口述内容时：
1. 解析口述内容，提取：患者反馈、临床处置、后续计划
2. 生成结构化草稿，发给 Andrew DM 确认
3. Andrew 确认后，更新 entity memory <患者ID>.md
4. 如涉及用药变化，询问 Andrew："是否需要通知治疗师？[是] [否]"
5. 如 Andrew 选是，触发 cross-provider-brief skill
```

---

## 扩展：批量更新

```
Andrew 一次性更新多个患者：

今天看了三个：
AX-2847 Maria 增了 Sertraline 100mg，4周后见
AX-3201 James 没来（爽约）
AX-2199 David 状态稳定，维持 Lexapro 20mg，8周后见

andrew-bot 逐一处理，各自更新 entity memory，汇报：
"✅ 三个患者档案已更新：AX-2847 / AX-3201 / AX-2199"
```
