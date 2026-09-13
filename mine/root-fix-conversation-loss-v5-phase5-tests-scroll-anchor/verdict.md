# 判词：`workbench/root-fix-conversation-loss-v5-phase5-tests-scroll-anchor` @ `863ff7e0`

**判词：`ORPHAN-ON-BRANCH`（主干没有、分支头上有，代码可用）+ 一条附带 `HALF`**
证据时间：2026-09-13 实查。

## 一、它到底是什么

README_CN 点名的 `863ff7e0…`（「加载旧消息保持视口」）不是丢失的提交，
**它就是本分支的分支头**。地基审计当时只用本地克隆的 refs 做 `git cat-file -e`，
没 fetch 远端分支头，才误判成「MISSING、被压缩重建抹掉」。已在 `LEDGER.md` §0 更正。

分支头相对 merge-base 的独有改动面（`compare_refs` 实查）含：
`viewmodel/ChatViewModel.kt`、`viewmodel/ConversationUiState.kt`、
`model/MessagePersistenceGuard.kt`、`ui/chat/MessageList.kt`、
`data/local/ChatDatabase.kt`，以及一次性手术刀工作流
`apply-conversation-loss-v5-phase1.yml`（+27）、`integrate-client-resilience-once.yml`（+35）。

## 二、代码实测（闸门 1：字面量/括号配平）

实读 `app/src/test/java/com/newoether/agora/viewmodel/ConversationUiStateTest.kt`
（分支头版本，blob `98c77248`，4147 B）：

- 引号完整、括号配平，**没有** `fix-client-resilience-ci-v4` 那次的咬字面量事故（见
  `mine/fix-client-resilience-ci-v4/verdict.md` 的列号反推）；
- 里面有一条**主干没有**的新测试：`boundedWindow_withoutRoot_startsAtEarliestLoadedOrphan`
  —— 断言「父指针指向不存在的消息时，从最早的孤儿起算窗口」：

```kotlin
val msgs = listOf(
    msg("u50", "missing-parent", "q50"),
    msg("m50", "u50", "a50", Participant.MODEL),
    msg("u51", "m50", "q51")
)
val path = ConversationUiState.resolvePath(msgs, null, emptyMap())
assertEquals(listOf("u50", "m50", "u51"), path.map { it.id })
```

→ 这是**可搬的资产**：一条真实的边界行为契约（截断窗口遇到断链父指针不许清空列表），
正是「会话切换后白屏」那一类病的验收标准。搬的时候连代码带断言一起搬，不搬 README 的说法。

## 三、诚实标注：它自己那轮 CI 是红是绿，未定

本分支树里那份 `ci-failure-summary.txt`（36,938 B）**不是本分支的现场**——头两行写着
`run_id=30776666129` / `head_sha=bb92a6b2…`，那是 `fix-client-resilience-ci-v4` 的失败日志，
被后续分支当成源码带走了。所以：

- **不许**拿它判本分支红；
- 本分支自己的 run 记录需要单独找（旧分支的 run 已掉出列表首页，待按分支翻页查）；
- 状态：`UNKNOWN（CI 结论未取证）`，不影响 §二 的代码事实判词。

## 四、下一步（本族剩下 5 条）

| 分支 | 头 SHA | 待验的东西 |
|---|---|---|
| root-fix-conversation-loss-v5 | d690c3f2 | phase1 完好快照到底进了哪几个文件 |
| …-phase2 | aa242615 | `extraPadding = 0.dp`（主干已有，判是否重复落地） |
| …-phase3 | ef1a8db5 | `lastCheckpointMs`（主干已有 3 处，判是否同一实现） |
| …-phase4 | f736b4bb | `getNewestMessagesPage` / `getOlderMessagesPage`（主干 0 处 → 真缺口） |
| …-phase4-bounded | 266dbcb4 | `maxTextChars` 有界分页参数（主干 0 处 → 真缺口） |

**优先级**：phase4 / phase4-bounded 最高——它们补的是**数据库层**的分页查询，
主干至今 0 处，属于「文档吹过、主干确实没有、分支头上有」的三合一样本。
