# 家族现场：conversation-loss v5（6 条分支）

实查时间 2026-09-13。这一族是 README_CN「历史分页 / 会话切换不清空」那套说法的落点，
也是「地基审计误判 SHA 丢失」的现场。

## 一、拓扑（先看清楚在比什么，别拿错基线）

| 分支 | 头 SHA | 相对 `main` |
|---|---|---|
| root-fix-conversation-loss-v5 | d690c3f2 | **ahead 57 / behind 269** |
| …-phase4 | f736b4bb | **ahead 67 / behind 269** |
| …-phase2 / -phase3 / -phase4-bounded / -phase5-tests-scroll-anchor | aa242615 / ef1a8db5 / 266dbcb4 / 863ff7e0 | 同族，behind 同为 269 |

**关键事实**：`behind 269` 六条全一致 → 它们分叉于**同一条老主线**（09-11 压缩重建**之前**的快照），
彼此只是这条线上的不同阶段。含义三条：

1. 「分支独有 57~67 个提交」**不等于**「比 main 多 57~67 个功能的净增量」——
   老主线被重建甩掉 269 个提交，所以每条分支的 diff 里混着**大量与 main 的无关漂移**
   （实查 v5-root 的 diff 里 `.github/workflows/*` 就占 14 项，多为删旧 workflow、
   加 `apply-*-once.yml` 一次性手术刀工作流）；
2. 判词必须是**「与今日 main 的语义差」**，不能是「相对 merge-base 动了哪些文件」；
3. 这一族里 `.github/workflows/build-workbench.yml` 各分支都被改过
   （v5-root +52/−34、phase4 +22/−49）→ 想复原「哪轮 CI 被怎么调的」，材料就在这些分支里，
   但要**逐分支取**，不能拿 main 那版当它们的现场。

## 二、这一族的取证状态

| 分支 | 判词 | 已取证 |
|---|---|---|
| …-phase5-tests-scroll-anchor | `ORPHAN-ON-BRANCH` | ✅ 引号完整、含主干没有的断链窗口测试（见其 verdict.md） |
| root-fix-conversation-loss-v5 | 待判 | 改动面已取（compare），未读内容 |
| …-phase2 / -phase3 | 待判（主干疑已有同类实现，需查重） | — |
| **…-phase4** | **最高优先**：`getNewestMessagesPage`/`getOlderMessagesPage` 主干 0 处 | 只到 compare，**内容未验** |
| **…-phase4-bounded** | **最高优先**：`maxTextChars` 有界分页参数 主干 0 处 | 同上 |

**纪律**：phase4 系列在「读到 ChatDatabase.kt 原文并核过字面量/括号」之前，
判词一律写 `UNKNOWN`，不许拿 compare 的文件名当「已落地」的证据。

## 三、取证方法（本族固定动线）

1. `compare_refs(main, branch)` → 独有提交数 + 改动面（只当线索，不当结论）；
2. 读分支上的 `scripts/apply_conversation_loss_v5_phaseN.py` → 拿到刀想写的 `old/new` 块；
3. 读分支上被改的源文件原文 → 逐块核特征片段**出现次数恰为 1**（0=没落地，>1=重放插了两遍）；
4. 核字面量/括号配平（闸门 1，本机无沙盒 → 在 CI 里跑）；
5. 找**该分支自己的** run：`ci-failure-summary.txt` 里的 `run_id`/`head_sha` 必须先核是不是本分支的
   （已抓到一份 36,938 B 的漂流传家宝，见 `mine/fix-client-resilience-ci-v4/verdict.md` §三）。
