# 分支台账（Agora-Workbench 124 条非 main 分支）

盘点时间：2026-09-13（全部实查，非记忆）
- 分支总数：`124` 条非 main（含 main 共 125，与 repo-atlas 口径一致）
- **已完整并入 main：22 条**（内容已在主干，无独有资产，见 §1）
- **未并入：102 条**（独有资产，逐条待挖，见 §2）
- 判定方法：`list-stale-branches`（头是否被主干完全包含）+ `compare_refs`（独有提交数/改动面）
  + 分支头读原文核特征片段。**不采信 README、commit message、以及任何"已修复"的说法。**

## 0. 先更正地基账（重要）

`hualuo-foundation/audit/agora_audit.md` 判定：README_CN 点名的修复 SHA「全部 MISSING、
被 2026-09-11 压缩重建抹掉」。**这条判错了**——它们活在远端分支头上：

| README 点的 SHA | 真实位置 |
|---|---|
| `d690c3f2`（会话完好快照 phase1） | `workbench/root-fix-conversation-loss-v5` 分支头 |
| `863ff7e0`（滚动锚点 phase5） | `workbench/root-fix-conversation-loss-v5-phase5-tests-scroll-anchor` 头 |
| `ef1a8db5`（生成检查点 phase3） | `…v5-phase3` 头 |
| `f736b4bb`（钥匙串分页 phase4） | `…v5-phase4` 头 |
| `266dbcb4`（有界分页 phase4-bounded） | `…v5-phase4-bounded` 头 |
| `aa242615`（尾部空白 phase2） | `…v5-phase2` 头 |
| `d3d57ea`（附件无限制） | `zip-attachments-no-limits`（×3 同名头） |

错因：那次审计只用本地克隆的 refs 做 `git cat-file -e`，没 fetch 远端分支头——**方法缺口，
不是造假**。结论修正为：`ORPHAN-ON-BRANCH`（主干没有、分支头上有），属可恢复资产。
"重构信代码不信文档"这条不变。

## 1. 已并入 main 的 22 条（免挖）

`courier/file-delivery`、`workbench/agent-team-ui`、`workbench/continue-missing-features`、
`dispatch-guardrails`、`fix-backup-import-oom-20260810`、`fix-conversation-cross-talk`、
`fix-github-tool-suite-build`、`fix-openai-stream-termination`、
`generation-ui-timeout-tools-20260818`、`github-binary-upload`、
`mobile-long-stream-performance-20260814`、`pc-ramen-test-workbench`、
`pr-close-and-upstream-resolver`、`project-aware-code-intelligence`、
`stop-interrupt-draft-loss-20260826`、`tool-memory-chat-fix`、`uma-broad-local-reader`、
`uma-session-msgpack-json-20260810`、`upstream-sync-conflict-resolution`、
`zip-attachments-no-limits`、`zip-attachments-no-limits-v2`、`zip-attachments-no-limits-v3`

> 注：后三条头 SHA 完全相同（`d3d57ea`），是同一次提交的三个别名，不是三份不同工作。

## 2. 未并入的 102 条（独有资产）

`状态`：本轮 = 本批开挖；待挖 = 排队；组：会话/附件/uma 导出/token/GitHub/流式/其它。

| 分支（workbench/ 前缀省略） | 头 SHA | 组 | 状态 |
|---|---|---|---|
| agora-reliability-v1 | 309d9e8e | 会话 | 待挖 |
| audit-p0-tools | db91c7b6 | 其它 | 待挖 |
| binary-audit-tools | e5e311bd | 附件 | 待挖 |
| chat-history-performance-1.4-fix | 8614ff12 | 会话 | 待挖 |
| ci-apk-metadata-probe | c139dc60 | 其它 | 待挖 |
| complete-long-reasoning-timeout-fix | 1b9992f7 | 流式 | 待挖 |
| compose-anthropic-stream-usage-20260809 | 084bf191 | token | 待挖 |
| connect-openai-token-usage-20260809 | 858bb2ad | token | 待挖 |
| consolidate-clone-tool | 9b319577 | GitHub | 待挖 |
| consolidate-missing-features | 295519cc | 其它 | 待挖 |
| deduplicate-tool-definitions-20260809 | 83f10842 | 其它 | 待挖 |
| durable-archive-action-logs | 1b46c065 | 其它 | 待挖 |
| emit-mapped-openai-token-usage-20260809 | 6ae34046 | token | 待挖 |
| enable-zip-bin-attachments | 76abca0a | 附件 | 待挖 |
| export-backup-16pct-fix | 2596e68f | 附件 | 待挖 |
| fgs-start-timeout-60 | 50f32a36 | 流式 | 待挖 |
| fix-backup-classification-package-20260818 | f105b216 | 附件 | 待挖 |
| fix-client-resilience-ci | bf74d3e7 | 会话 | 待挖 |
| fix-client-resilience-ci-v2 | 9503054a | 会话 | 待挖 |
| fix-client-resilience-ci-v3 | 25aec113 | 会话 | 待挖 |
| fix-client-resilience-ci-v4 | 01fd184c | 会话 | 待挖 |
| fix-github-json-chat-leak | c933a035 | GitHub | 待挖 |
| fix-invisible-messages | 50a94800 | 会话 | 待挖 |
| fix-log-summary-truncation-20260826 | 3559884b | 其它 | 待挖 |
| fix-long-conversation-durability | 4dc6b2e9 | 会话 | 待挖 |
| fix-main-verification-artifact-upload-20260809 | c0673d0d | GitHub | 待挖 |
| fix-negative-snackbar-padding | 6c03a159 | 其它 | 待挖 |
| fix-negative-snackbar-padding-v2 | 8672ead3 | 其它 | 待挖 |
| fix-openai-think-and-all-read-timeouts | 715dadc7 | 流式 | 待挖 |
| fix-red-build | 9264c227 | 其它 | 待挖 |
| fix-release-dispatch-defaults-20260810 | 92a68f2b | GitHub | 待挖 |
| fix-reply-disappears-on-next-send | da0a4ad8 | 会话 | 待挖 |
| fix-stable-signing-and-latest-release | 8f043c15 | GitHub | 待挖 |
| fix-uma-read-endpoint | b12b3a53 | uma 导出 | 待挖 |
| fix-uma-upload-checkpoint-isolation-20260812 | df6f275e | uma 导出 | 待挖 |
| fix-version-release-pipeline | 2bd2836e | GitHub | 待挖 |
| fix-zip-cursor-window | 0cabfa21 | 附件 | 待挖 |
| fix-zip-cursor-window-v2 | e3e59054 | 附件 | 待挖 |
| fork-upstream-pr-workspace | 327b8661 | GitHub | 待挖 |
| github-actions-manual-window-20260809 | bddb2c01 | GitHub | 待挖 |
| github-actions-status-model-20260809 | 9294d0d2 | GitHub | 待挖 |
| github-code-editor | ee80403c | GitHub | 待挖 |
| github-create-repository-tool | 81087f95 | GitHub | 待挖 |
| github-pr-merge-tools | a36fe4ac | GitHub | 待挖 |
| github-task-completion | f52a580a | GitHub | 待挖 |
| github-tool-suite | 185dd72a | GitHub | 待挖 |
| integrate-client-resilience-fixes | 9734ab00 | 会话 | 待挖 |
| local-artifact-store-phase1-20260812 | fd47daf8 | 附件 | 待挖 |
| lsp-language-packs | 51a66ec1 | 其它 | 待挖 |
| map-anthropic-token-usage-20260809 | 1fb15470 | token | 待挖 |
| map-openai-token-usage-20260809 | c7287fed | token | 待挖 |
| multiagent-team | c2b06894 | 其它 | 待挖 |
| net-download-file | 2d9ebba5 | 其它 | 待挖 |
| public-repository-browser | 63e8f8cc | GitHub | 待挖 |
| ramen-datasource | d9d5bd52 | uma 导出 | 待挖 |
| register-manual-main-build-20260811 | aefc6eb7 | GitHub | 待挖 |
| register-net-download | 58d6d058 | 其它 | 待挖 |
| register-uma-session-export-tools-20260810 | 87253ca3 | uma 导出 | 待挖 |
| release-1.4.6-current-main | 9364c199 | GitHub | 待挖 |
| release-1.4.7 | 58136afc | GitHub | 待挖 |
| release-1.4.8-session-upload-20260810 | d3f4a1c3 | GitHub | 待挖 |
| release-1.4.9-defaults | bfcdab23 | GitHub | 待挖 |
| release-1.5.1 | de2a29ba | GitHub | 待挖 |
| remove-about-page | 9e9fa58e | 其它 | 待挖 |
| remove-stream-read-hard-timeout | 8deeb4c3 | 流式 | 待挖 |
| repo-hygiene-cleanup-20260826 | bae9e919 | 其它 | 待挖 |
| **root-fix-conversation-loss-v5** | d690c3f2 | 会话 | **本轮** |
| **root-fix-conversation-loss-v5-phase2** | aa242615 | 会话 | **本轮** |
| **root-fix-conversation-loss-v5-phase3** | ef1a8db5 | 会话 | **本轮** |
| **root-fix-conversation-loss-v5-phase4** | f736b4bb | 会话 | **本轮** |
| **root-fix-conversation-loss-v5-phase4-bounded** | 266dbcb4 | 会话 | **本轮** |
| **root-fix-conversation-loss-v5-phase5-tests-scroll-anchor** | 863ff7e0 | 会话 | **本轮** |
| root-fix-release-tools-phase1 | 25360166 | GitHub | 待挖 |
| separate-public-repo-read | 820d14c4 | GitHub | 待挖 |
| session-usage-dashboard-20260813 | 5881b1ac | token | 待挖 |
| sign-1.4.7-debug | 00917b2a | GitHub | 待挖 |
| spreadsheet-attachments-20260807 | b7969506 | 附件 | 待挖 |
| spreadsheet-attachments-v2-20260807 | 5bdce9a0 | 附件 | 待挖 |
| sqlite-oversize-row-guard | fa4b186f | 附件 | 待挖 |
| todo-history-blank-sniff | c306452e | 会话 | 待挖 |
| token-usage-port | 0574cfb6 | token | 待挖 |
| tool-detail-overlap-fix-20260826 | 6260a243 | 其它 | 待挖 |
| uma-binary-session-export-20260810 | ce1028c6 | uma 导出 | 待挖 |
| uma-export-download-tree-signing-20260811 | 354f51b0 | uma 导出 | 待挖 |
| uma-export-picker-tree-batching | cd7a2883 | uma 导出 | 待挖 |
| uma-git-blob-upload-20260810 | bbd815df | uma 导出 | 待挖 |
| uma-git-tree-20260810 | 2e702a92 | uma 导出 | 待挖 |
| uma-overlay-capture-toggle | e6f190aa | uma 导出 | 待挖 |
| uma-session-batched-upload-progress-20260811 | 0817fc68 | uma 导出 | 待挖 |
| uma-session-complete-upload-json-20260810 | a1b79f8d | uma 导出 | 待挖 |
| uma-session-export-resume-20260810 | abc5ee2a | uma 导出 | 待挖 |
| uma-session-github-upload-pipeline-20260810 | 4147e5e0 | uma 导出 | 待挖 |
| uma-session-github-upload-tool-20260810 | 2138dca1 | uma 导出 | 待挖 |
| uma-session-zip-export-20260810 | fa31044c | uma 导出 | 待挖 |
| uma-storage-files-pagination-20260810 | 20e368b7 | uma 导出 | 待挖 |
| uma-upload-durable-restart-20260826 | 48e58062 | uma 导出 | 待挖 |
| unified-token-usage-model-20260809 | 85a1f9b6 | token | 待挖 |
| v3-release | 645f7bde | GitHub | 待挖 |
| workspace-lane-sync-ui | 97662692 | 会话 | 待挖 |
| workspace-persistent-chat | 3738b40d | 会话 | 待挖 |
| zip-attachments-final | 8a742968 | 附件 | 待挖 |
| zip-attachments-unrestricted | bae2e910 | 附件 | 待挖 |

## 3. 挖一条分支要留什么

`mine/<分支>/meta.json`（头 SHA、独有提交数、改动文件表、有无一次性 workflow）
→ `patches/`（只读导出的 .patch，不重放）→ `gate/`（三道闸门输出）→ `verdict.md`（判词 +
`文件:行号` + run id）。判死的移进 `invalidated/` 留尸，不删。
