# 判词：`workbench/root-fix-conversation-loss-v5` @ `d690c3f2`（phase1 完好快照）

**判词：`ORPHAN-ON-BRANCH` + `LIVE`（六处替换逐处对上、引号完整、结构配平）**
**附带 `HALF`：文件里留了三处刀疤（一个悬空 KDoc + 三个函数缩进塌成 0 列）**

证据：2026-09-13 实读分支头 `app/src/main/java/com/newoether/agora/viewmodel/ChatViewModel.kt`
（blob `94d34ad1`，**65,434 B**，全文读完）+ 刀源 `scripts/apply_conversation_loss_v5_phase1.py`
（blob `1e46652e`，6,692 B）。

## 一、六处替换全部落地，各一次

刀有六对 `old/new`，守卫是 `count != 1 → SystemExit`（所以要么全中、要么整刀失败，不会插两遍）。
逐处对实读结果：

| # | 刀要改的东西 | 分支头实测 |
|---|---|---|
| 1 | `loadOlderMessages` 注释改为「Never discard the last coherent snapshot while loading」+ 新增字段 | ✅ 两处都在，紧跟其后 |
| 2 | `.catch` 里不再清空列表，改成回落到完好快照 | ✅ `coherentMessageSnapshots[id]?.let { _allMessages.value = it }` |
| 3 | `entities.map {` → `entities.mapNotNull { entity -> runCatching {` | ✅ |
| 4 | 收尾 `}.onFailure { … DebugLog.e("…Skipping malformed history row ${entity.id} in $id"…) }.getOrNull() }` | ✅（`${entity.id}` 与 `$id` 两处插值**都活着**，说明 Python 没用 f-string/format 吃掉 `$`） |
| 5 | `_allMessages.value = mapped.map{…}` → `val coherent = …` + 身份门 `if (_currentConversationId.value != id) return@collect` + 「行存在但一条都解不出来」分支 + 写回快照表 | ✅ 四段全在 |
| 6 | `selectConversation` 里 `_allMessages.value = emptyList()` → 先回落到该会话上次的完好快照 | ✅ |

新增字段本体：

```kotlin
/** Last coherent mapped snapshot per conversation. Switching never destroys it pre-emptively. */
private val coherentMessageSnapshots = java.util.concurrent.ConcurrentHashMap<String, List<ChatMessage>>()
```

**语义评价**：这正是 R8「partial 不许渲染成空气泡」想要的东西——加载失败/解码全军覆没时
**保留上一次连贯画面**而不是白屏，而且它带了一条硬错误消息
（`Conversation rows exist but none could be decoded`），不是「静悄悄空列表」。
**这条该搬**（连同它的身份门：`collectLatest` 之外还要显式比 `_currentConversationId`，
因为迟到的 Room 完成会覆盖新会话——这是真并发教训，不是过度设计）。

## 二、但它只有 phase1，没有 phase4：链是分叉的，不是线性的

同一个文件里仍然写着：

```kotlin
private const val MAX_MESSAGE_WINDOW = 500
...
messageWindowSize.update { (it + MESSAGE_WINDOW_STEP).coerceAtMost(MAX_MESSAGE_WINDOW) }
_hasOlderMessages.value = entities.size < total && messageWindowSize.value < MAX_MESSAGE_WINDOW
```

→ 会话翻到底**仍然卡在 500 条**（这条分支没解决）；解决它的是 `phase4` / `phase4-bounded`
（keyset 分页，见 `mine/root-fix-conversation-loss-v5-phase4-bounded/verdict.md`）。
所以 M2 的搬法是：**phase1 的「完好快照 + 身份门 + 逐行解码隔离」+ phase4-bounded 的「keyset 有界分页」组合**，
不是按 phase 号照抄某一条分支。

## 三、刀疤（`HALF` 部分，搬之前要顺手治）

1. **悬空 KDoc**：讲 `startInitJobs` 的文档块现在挂在 `applyProxy()` 头上，两段 KDoc 挨着：
   ```kotlin
   /**
    * Startup jobs deferred until all StateFlow/property backing fields are
    * initialized — avoids the constructor this-escape where a Dispatchers.IO
    * coroutine accesses a field whose JVM backing field is still null.
    */
   /** Build the proxy config from settings and push it into the shared HttpClient. */
   private fun applyProxy() {
   ```
   `startInitJobs` 的文档被这次插块**留在原地没人认领**。中段替换（`s[:start] + new + s[end:]`）
   不检查边界，就会出现这种「代码还在、文档错位」的疤——**编译和测试都抓不到它**。
2. **缩进塌成 0 列**：`createNewChat()`、`selectConversation()`、`switchBranch()` 三个函数的
   函数体与收尾 `}` 大量顶到第 0 列（`    }\n}` 这种），是补丁块按字面贴回来的指纹。
   不影响编译，但说明这些段落**不是人写的、也没过格式化关卡**（旧仓没有 ktfmt 门禁）。
3. 文件里还有 `    \n\n        \n` 这类空行残渣（`getProviderForModel` 附近）。

→ 搬进 `hualuo-repo-tool` 时：**必须过 ktfmt**（地基 M4 已排），并且**文档块跟着函数走**——
新仓的红线是「注释与函数头绑定」，不接受悬空 KDoc。

## 四、未复核项（不猜）

- 「主干 0 处 `coherentMessageSnapshots`」引自 09-13 的地基审计报告（它读过 main 版 52,212 B 的该文件），
  本轮**没有重读 main 版**；大小差（65,434 vs 52,212）与该结论方向一致，正式引用前需复读一次 main。
- 该分支自己那轮 CI 的结论：**未取证**。
