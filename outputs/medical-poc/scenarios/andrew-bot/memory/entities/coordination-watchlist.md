# andrew-bot · 跨提供者协调监听列表
# 记录已发出的协调通知，等待治疗师回复
# andrew-bot 在 #clinical-coordination 监听时参考此文件

---

## 活跃监听条目

（此文件在 Demo 运行时由 andrew-bot 动态写入/清除）

---

## 已完成条目（历史记录）

```
[完成] 2026-06-05 14:31 → 14:38
  患者ID：AX-1956
  通知：Michelle Godwin（axismh-mgodwin）
  内容：Sertraline 50mg→100mg 用药变更
  回复：14:35 收到，附心悸反馈
  结果：已路由给 Andrew，档案已更新
  耗时：4 分钟
```

---

## 监听规则（andrew-bot SOUL 参考）

当在 #clinical-coordination 收到消息时，检查：

1. 消息发送者的 RC user ID 是否在"等待回复人"列表中？
2. 消息是否在监听到期时间内（发出后 48 小时）？
3. 消息内容是否与对应患者相关（患者ID、症状描述、确认语言）？

满足以上条件 → 触发回复处理：
  - 简单确认（"收到"/"好的"）→ 更新状态，不路由给 Andrew
  - 包含新临床信息 → 路由给 Andrew DM + 更新档案
  - 表达顾虑或问题 → 路由给 Andrew DM + 标记需要 Andrew 回应

超过 24 小时无回复 → 提醒 Andrew：
"⚠️ AX-1956 协调通知 24 小时未获 Michelle 确认，是否需要跟进？"
