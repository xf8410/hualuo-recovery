> ⚠️ 本页取代 `mine/OPEN-ITEMS-五问表.md`（同内容）。那次提交**违反本仓命名规矩**：
> 文件名必须 ASCII（中文只用于「给人看的描述」正文/标题）。旧文件我这边没有删除工具，
> 需在 GitHub 上手工删，或等本分支合并前一次性清理。**规矩是给自己立的，破了就当场记。**

# 五问表（每条问题都要答满五格，缺一格标 UNKNOWN）

模板来自主人的要求：**原来代码怎么写的 → 有什么问题 → 是否已合并 → bug 会不会复发 → 有没有接进对话 UI**。
本轮（09-13）先把「悬空件」的四格查实，代码原文栏排在下一轮。

## 1. 前台服务崩溃（FGS）· PR #60 —— **未合并，会复发**

| 格 | 内容 |
|---|---|
| 是否合并 | ❌ **`state: open`**，`mergeable: true` / `mergeable_state: clean`（能干净合，就是没人合） |
| 改动面 | 仅 1 个文件：`app/src/main/java/com/newoether/agora/service/AgoraForegroundService.kt`，**+53 行** |
| 分支/头 | `workbench/fgs-start-timeout-60` @ `50f32a365b64c5673c80c2e24434cf6473e69d13` |
| 标题自述 | 「onStartCommand 兜底挂前台 + 3.5s 启动看门狗（封堵 #60 残余超时窗口）」 |
| 复发 | **会**。主干没有看门狗，FGS 5 秒窗口一超就被系统杀 → 127.0.0.1:18765 观测桥 + 全部 `uma_*` 工具当场失效（09-11 已实测崩过一次） |
| 接入对话 UI | 无（服务层），但**症状显示在对话里**：工具整排报连接失败 |
| 原文 | 待读（下一轮读 `AgoraForegroundService.kt` 主干版 + PR 版逐行对） |

## 2. audit P0 三工具 · PR #59 —— **未合并，手机端现在就是缺的**

| 格 | 内容 |
|---|---|
| 是否合并 | ❌ **`state: open`**，`mergeable: null` / `mergeable_state: unknown`（GitHub 还没算出能不能合 = **主干已跑过它**，八成要重开） |
| 改动面 | 7 个文件：新增 `audit/BytePatternSearcher.kt`(+78)、`audit/ZipInspector.kt`(+69)、`tool/BinaryAuditP0ToolProvider.kt`(+168)、两份测试(+92/+115)；**改** `viewmodel/GenerationManager.kt`(+4) 与 `ToolProviderRegistrationTest.kt`(+3) |
| 关键 | 注册点在**这个 PR 里**（`GenerationManager.kt` +4 = 往工具表挂 provider，+3 = 守卫测试跟着加名单）→ 没合 = **模型侧根本看不到 `audit_search` / `audit_zip_list` / `audit_zip_extract`**，跨窗字节搜索与 APK 条目掏取在手机上做不到 |
| 复发 | 这是**注册/接线病的复发形态**：不是写漏了，是**写完了挂在 open PR 上没人收**（与 #54「说注册了其实没注册」同一类） |
| 接入对话 UI | 工具卡会自动挂上（走 provider 表），**前提是注册那段进了 main** —— 现在没有 |
| 原文 | 待读 |

## 3. LSP 语言包分支 —— **红因终于开口了：测试名带非法字符**

`workbench/lsp-language-packs` 头已从 `51a66ec1` 走到 **`bd40ff4cd25649e1ad0488bc62f6525d7a47f0d9`**。
Run **34694981648**（09-12 12:55Z，`push` 触发）结论 **failure**，错误卷原文（取自作业日志结尾）：

```
e: .../app/src/test/java/com/newoether/agora/lsp/DiagnosticParsersTest.kt:123:9 Name contains illegal characters: ..
> Task :app:compileFdroidDebugUnitTestKotlin FAILED
BUILD FAILED in 2m 10s
UNIT_OUTCOME: failure   BUILD_OUTCOME: skipped   VERIFY_OUTCOME: skipped
```

三条结论：

1. **红因是「反引号测试名里带 `..`」**——Kotlin 直接拒绝，跟业务逻辑无关，一行改名就能修；
   它已经挡在那儿一天多，**这就是「14 种编译语言 + lsp_check/lsp_install」一直上不了真机的唯一原因**。
2. 「让红色开口说话」这套改造**生效了**：错误卷被单列在作业结尾
   （`grep -nE "^e: |^Caused by:|FAILURE:…"`），三态变量也正常报 `failure / skipped / skipped`；
   但同时暴露一件事——**单测一红，APK 与装机验收整段被 skip**，所以这条分支从来没产出过可装包。
3. 顺带抓到**屎山税实锤**：清理阶段每个 step 都递归 `git submodule foreach`
   （`Entering 'thirdparty/llama.cpp'` / `Entering 'thirdparty/proot'`，一次 run 里 6+ 遍）。

## 4. 取证受阻（记下来，别当成「没问题」）

- **GitHub 代码搜索索引对这个仓不可用**：`search_code(AttachmentTypeUi)` 返回
  `total_count: 0, incomplete_results: true`。→ 「附件白名单静默不渲染」那条只能**直读源文件**取证，
  下一轮读 `ui/chat/message/UserMessageBubble.kt` 主干版。
- `sqlite-oversize-row-guard`（守卫写好未接线 = 死代码）的 `compare_refs` 尚未跑。

## 5. 下一轮顺序（按「主人能感到」排序，不按编号）

1. `UserMessageBubble.kt` 主干原文 → 附件白名单 filter 到底怎么写的（你发消息看不见文件那条）；
2. `AgoraForegroundService.kt` 主干 vs PR #60 → 看门狗实现是否可信、能不能直接搬；
3. `sqlite-oversize-row-guard` → 证实「死代码」，判要不要在 `hualuo-repo-tool` 重做；
4. 记忆 20 项里剩下的（备份密钥静默置空、沙盒 D1–D6、上下文按条数裁剪…）逐条补五问；
5. **新 UI 设计**（主人明确：不要 Agora 那套）——与审计并行，出可交互 HTML 稿 + 设计规格。
