# 判词：workbench/root-fix-conversation-loss-v5-phase4-bounded（头 266dbcb4）

盘点时间：2026-09-13 深夜（全部机器实查，证据 = 文件:行 或 run id）

## 0. 谱系更正（影响整个挖掘计划）
`compare_refs base=d690c3f2(phase1头) head=266dbcb4` → **behind 0 / ahead 15**。
即 phase4-bounded 是 **叠在 phase1 之上**的续串：phase1 完好快照 + phase2/3/4/4-bounded
四把刀全在这一个头上（compare 文件表里 phase2/3/4/4_bounded 四个 .py 均为 added）。
**含义：挖这一条头 ≈ 覆盖 v5 全系列；其余 4 条 phase 分支不必逐条重挖，只需查各自
"中途态"有没有被最终头吸收（默认已吸收）。**
改动面：ChatDatabase.kt(+67) · ConversationRepository.kt(+18) · ChatViewModel.kt(+50/−48)
· GenerationManager.kt(+65/−3) · ChatApp.kt(+9/−25) · MessageList.kt(+4/−15)
· build-workbench.yml(+34/−53)（一次性代跑 workflow 痕迹，符合预期）。

## 1. 实读证据（HEAD 原文，非刀脚本说词）
- `ChatDatabase.kt@266dbcb4`（32,256 B 整读）：
  - **keyset 分页在**：`getOlderMessagesPage` = `WHERE (timestamp < :beforeTimestamp OR (timestamp = :beforeTimestamp AND id < :beforeId)) ORDER BY timestamp DESC, id DESC LIMIT :limit`。
  - **有界投影在**（三个读路径全带）：text/thoughts 超限 `substr || '[… preview truncated]'`，toolCallJson/attachmentMeta 超限置 NULL；预算参数 maxTextChars/maxThoughtChars/maxToolJsonChars/maxAttachmentMetaChars。
  - 索引 `(conversationId, timestamp)`（v16 迁移）支撑 keyset；`id` 决胜不在索引内——搬时索引补 `id` 列。
  - **刀疤**：`@Database(...)@TypeConverters(...)` 挤同一行（换行被刀吃掉），编译无碍、格式塌。
- `scripts/apply_conversation_loss_v5_phase4_bounded.py`（5,092 B 整读）：三处替换全部 `count==1` 守卫；Python 串里 `\\n\\n` → 头文件 SQL 里成真换行，双写转义**这次是对的**（对照反例：v4 那次吃掉测试引号的事故）。第一处替换是给 ChatViewModel **补删一个多余 `}`**——phase4 的括号残伤由 bounded 收尾修复，证明中途态有自我修复、最终态才是准。
- `ConversationRepository.kt@266dbcb4`（刀 new 块与头一致，limit.coerceIn(1,100) + 预算实参 65536/32768/131072/32768）。

## 2. 判词：**DAO/repo 层 = LIVE**
「翻到底卡 500 条」的真解在这：**phase1（完好快照+身份门+逐行隔离）+ phase4-bounded（keyset+有界投影）两半拼**，照抄此头、不取 phase2-4 中途态。
M2 迁移账按此登记。

## 3. 未验收项（下轮必查，不许拍板为绿）
1. **ChatViewModel +50/−48 没整读**：`MAX_MESSAGE_WINDOW=500`、`_hasOlderMessages` 在此头是否真改成分页驱动——DAO 有了，UI 没接线=白搭（旧仓病 No.6 的复发判定点）。
2. **GenerationManager +65/−3**：含不含 phase3 检查点特征；顺带定位 MessagePersistenceGuard.kt 真实路径（此头按旧路径 viewmodel/ 读 404，需树查）。
3. **红线反例复查**：`MessagePersistenceGuard.sanitize()`（写路径偷改正文）在此叠串有没有借尸还魂——路径 404 ≠ 不存在，找到文件才算排除。
4. build-workbench.yml +34/−53：确认它只是"CI 代跑刀"痕迹、没改门禁语义（防假绿文化回潮）。

## 4. 附带更正（同一轮实查，推翻旧结论）
`workbench/lsp-language-packs`：run **34723037840**（09-12 22:32Z，头 51a66ec1）**success**，
artifact `agora-workbench-debug-51a66ec1…` **63.5 MB APK 存在**（至 12-11 不过期）；
`DiagnosticParsersTest.kt@lsp头`（8,107 B）测试名已无点号，纪律写进 KDoc。
**「LSP 分支从未产出可装包」作废**（那是基于较早红 run 的判断）→ 不需要破例改污染仓，铁律保住。
