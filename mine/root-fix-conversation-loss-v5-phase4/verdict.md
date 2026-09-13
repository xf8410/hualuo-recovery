# 判词：`workbench/root-fix-conversation-loss-v5-phase4` @ `f736b4bb`

**判词：`ORPHAN-ON-BRANCH` + `LIVE`（刀确实落地、代码形状正确、主干没有 = 可搬资产）**
但搬的时候有两个坑必须处理（§三）。证据时间 2026-09-13，全部实读，不采信 README 说法。

## 一、刀与落点对上了（逐字核）

刀源：`scripts/apply_conversation_loss_v5_phase4.py`（该分支上，11,392 B，blob `716ff427`）。
它给 `ConversationRepository.kt` 插入 `getNewestMessagesPage` / `getOlderMessagesPage` 两个包装，
守卫是 `if s.count(marker) != 1: raise SystemExit('repo marker mismatch')`（**恰一处**，不许零也不许多）。

实读该分支 `app/src/main/java/com/newoether/agora/data/repository/ConversationRepository.kt`
（11,742 B，blob `0ef484a7`），第 80 行起：

```kotlin
suspend fun getNewestMessagesPage(conversationId: String, limit: Int = 24): List<MessageEntity> =
    chatDao.getNewestMessagesPage(conversationId, limit.coerceIn(1, 100))
        .sortedWith(compareBy<MessageEntity> { it.timestamp }.thenBy { it.id })

suspend fun getOlderMessagesPage(
    conversationId: String, beforeTimestamp: Long, beforeId: String, limit: Int = 24,
): List<MessageEntity> = chatDao.getOlderMessagesPage(
    conversationId, beforeTimestamp, beforeId, limit.coerceIn(1, 100)
).sortedWith(compareBy<MessageEntity> { it.timestamp }.thenBy { it.id })
```

→ **与刀里的 `new` 块逐字一致，且各出现一次**。DAO 层（`ChatDatabase.kt`）同批插入 strict keyset 查询：
`timestamp < :beforeTimestamp OR (timestamp = :beforeTimestamp AND id < :beforeId)`
—— 严格 keyset 分页，不是 `OFFSET`。这一条主干**至今 0 处**，是真缺口。

结论：地基审计说「phase4 从未进 main」✅ 对；但「SHA MISSING、找不回」❌ 错 ——
代码完整活在分支头上，**这是可恢复资产，不是要重写的东西**。

## 二、关键依赖：phase4 离不开 phase1（不许单搬）

phase4 写入 `ChatViewModel.kt` 的新代码块里引用了 **`coherentMessageSnapshots[id]`**
（完好快照表）—— 而那张表是 **phase1** 建的，phase1 从未进 main。含义：

- 只搬 phase4 → `Unresolved reference: coherentMessageSnapshots`，**编译当场红**；
- 这一族是一条**链**：phase1（快照表）→ phase2（尾部空白）→ phase3（检查点）→ phase4（keyset 分页）
  → phase4-bounded（长度上限参数）→ phase5（分页合并 + 滚动锚点）；
- 所以 `hualuo-repo-tool` 侧的 M2 要么**整链一起搬**，要么按地基的红线重写
  （R5 消息库禁无界 TEXT + R8 永不空气泡），把 `coherentMessageSnapshots` 作为**设计前提**保留、
  实现自己写。**不许挑一期看着顺眼就搬**。

顺带：这刀的第三段是**无守卫的中段整体替换**——
`start = s.find(...)` / `end = s.find(end_marker, start)` / `s = s[:start] + new_block + s[end:]`。
它只检查 `start < 0 or end < 0`，不检查 `end - start` 的范围是否就是它想换的那一段。
一旦 `end_marker` 在别处先出现，就会吞掉中间一段代码而**照样编译通过**——
这就是「结构看起来没问题、语义少了一块」的成因之一。地基 R1 禁构建时改源码，正是绝这类写法。

## 三、搬的时候两个坑（都是红线级）

1. **魔法数字**（撞红线 4）。同文件第 74 行主干/分支共有的写法：
   ```kotlin
   chatDao.getMessagesForConversation(conversationId, limit.coerceIn(1, 500), 65_536, 32_768, 131_072, 32_768)
   ```
   四个裸数字是 CursorWindow 256 KB 预算的切分，但**没有任何名字和出处**。
   → 进 `hualuo-repo-tool` 必须改成具名常量 + 注释写明推导（哪段预算给谁、为什么是这个值），
   否则就是明知故犯。
2. **别把 `sanitize` 搬回来**（撞红线 1）。同文件第 99 行仍是
   `chatDao.upsertMessage(MessagePersistenceGuard.sanitize(entity))` —— 主干已整体删掉这个守卫，
   分支上还留着。**从分支搬代码时，先扫 `sanitize|redact|mask|脱敏`，命中的行一律不进新仓。**

## 四、本条未取证的部分（诚实标注）

该分支**自己那轮 CI** 是红是绿：未查（本分支树里那份 `ci-failure-summary.txt` 是别的分支漂来的，
见 `mine/fix-client-resilience-ci-v4/verdict.md` §三）。判词只覆盖「代码事实」，
不覆盖「当时 CI 结论」；补 run 取证排在 phase4-bounded 之后。
