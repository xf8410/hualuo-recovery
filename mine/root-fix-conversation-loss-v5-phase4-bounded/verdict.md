# 判词：`workbench/root-fix-conversation-loss-v5-phase4-bounded` @ `266dbcb4`

**判词：`ORPHAN-ON-BRANCH` + `LIVE`（这一条才是本族的「完整版」，M2 该搬它，不搬 phase4）**
证据：2026-09-13 实读 `app/src/main/java/com/newoether/agora/data/local/ChatDatabase.kt`
（分支头版本，blob `39c6d49e`，32,256 B，全文读完）。

## 一、主干缺的东西，这条分支上有全套

`ChatDao` 里两个方法**各出现一次**，括号/三引号配平，末尾 `}` 收得干净：

```kotlin
suspend fun getNewestMessagesPage(
    conversationId: String, limit: Int,
    maxTextChars: Int, maxThoughtChars: Int, maxToolJsonChars: Int, maxAttachmentMetaChars: Int,
): List<MessageEntity>

suspend fun getOlderMessagesPage(
    conversationId: String, beforeTimestamp: Long, beforeId: String, limit: Int,
    maxTextChars: Int, maxThoughtChars: Int, maxToolJsonChars: Int, maxAttachmentMetaChars: Int,
): List<MessageEntity>
```

SQL 侧是**严格 keyset**（不是 OFFSET）+ 有界投影，两件事同时具备：

```sql
WHERE conversationId = :conversationId
  AND (timestamp < :beforeTimestamp OR (timestamp = :beforeTimestamp AND id < :beforeId))
ORDER BY timestamp DESC, id DESC
LIMIT :limit
```

对照 `phase4`（`f736b4bb`）：那一版的分页**没有** `max*Chars` 参数，长度上限只在
`ConversationRepository.kt` 里以四个裸数字硬传（`65_536, 32_768, 131_072, 32_768`）。
→ **`phase4-bounded` 是 `phase4` 的改进版，同一处逻辑的两个版本里取这条**，
别按编号顺序搬最新的号，要按代码实际形态取。

主干现状（复述地基审计，仍需再核一次）：只有 `getMessagesForConversation` 的有界投影，
**分页查询 0 处** → 这就是「上滑翻不到老消息」这一族病在主干的真实状态。

## 二、但搬之前必须改一处：marker 自带两个换行（本族的病根之一）

三条 `CASE WHEN length(...) > ...` 里的截断标记被写成**跨行 SQL 字面量**：

```sql
THEN substr(text, 1, :maxTextChars) || '

[… message preview truncated for stability]'
```

`|| '` 后面真的跟着**两个换行**再写标记文本。后果：每条被截断的消息，正文尾部都凭空多出
`\n\n[… message preview truncated for stability]`。这跟 phase2 专门去修的「尾部空白」是同一类病
——**上游一边截断、一边又往正文里塞换行**，等于自己造自己要修的 bug。

→ 进 `hualuo-repo-tool` 时必须改成**单行标记**（` …[已截断]'`），并且：
- 四个上限参数一律具名常量 + 注释写清 256 KB CursorWindow 的推导（红线 4 禁魔法数字）；
- 截断这件事本身要可见（R7）：给个 `truncated` 标记或徽标，而不是把正文偷偷切短——
  现在这套 SQL 截完之后，用户看到的是一条「看起来完整、其实被剪过」的消息。

## 三、未取证（不猜）

- 该分支的 VM 侧（`ChatViewModel.kt` 是否同样带 `max*Chars` 调用 + keyset 累加）：**未读**，待验。
- 该分支自己的 CI run 结论：**未取证**（本仓 `mine/root-fix-conversation-loss-v5-phase4/verdict.md` §四 同一处理由）。
