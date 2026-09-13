# 断点状态（每轮开工先读这个）

> 用途：抗打断。这一页是**唯一进度入口**；`LEDGER.md` §2 的「状态」列会在攒够一批后统一回写，
> 在此之前**以本页为准**。

最近更新：2026-09-13（提交 `e21086f5` 时点）

## 已完成（4 条判词）

| 分支 | 头 | 判词 | 文件 |
|---|---|---|---|
| fix-client-resilience-ci-v4 | bb92a6b2 | **DEAD**：刀咬掉字符串引号 → 测试编译红（列号 17/22/30/38 反推实锤，run 30776666129）；附带抓到 `ci-failure-summary.txt` 跨分支漂流传家宝 | `mine/fix-client-resilience-ci-v4/verdict.md` |
| root-fix-conversation-loss-v5-phase5-tests-scroll-anchor | 863ff7e0 | **ORPHAN-ON-BRANCH**：引号完整、含主干没有的断链窗口测试（可搬）；本分支自身 CI 未取证 | `mine/…phase5-tests-scroll-anchor/verdict.md` |
| root-fix-conversation-loss-v5-phase4 | f736b4bb | **ORPHAN+LIVE**：keyset 分页确实落地（逐字核）；**依赖 phase1 的 `coherentMessageSnapshots`，不许单搬**；两处中段替换无范围守卫 | `mine/…phase4/verdict.md` |
| root-fix-conversation-loss-v5-phase4-bounded | 266dbcb4 | **ORPHAN+LIVE（本族完整版，M2 搬这条）**：keyset + 有界参数齐备；**截断标记自带两个换行须整改**、四个上限数字须具名 | `mine/…phase4-bounded/verdict.md` |

## 本族剩下 3 条（下一步顺序）

1. `root-fix-conversation-loss-v5` @ d690c3f2 —— 验 **phase1 完好快照**：`coherentMessageSnapshots`
   在 `ChatViewModel.kt` 里出现几次、形状对不对（它是上面两条判词的依赖前提，**最该先验**）；
2. `…-phase3` @ ef1a8db5 —— `lastCheckpointMs` 主干已有 3 处，判是否同一实现（可能重复落地）；
3. `…-phase2` @ aa242615 —— `extraPadding = 0.dp` 主干已有，同上判重。

## 全局待办（本族之外）

- **闸门 1/3 进 CI**：结构配平 + 变异自测。本机无 proot 沙盒 → 必须在 Actions 里跑，脚本要 ASCII 文件名
  （旧仓那个 `校验签名块.sh` 被 git octal-escape 搞得机器人自己都匹配不上，见 `hualuo-repo-tool` 审计）。
- **102 条未并入分支**逐条过（`LEDGER.md` §2 是名单）。
- `hualuo-repo-tool` 的 **M1.5 仍暂停**：分支 `workbench/m15-extract-upload-naming` 建了、0 提交。
