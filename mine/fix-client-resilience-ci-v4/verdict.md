# 判词：`workbench/fix-client-resilience-ci-v4` @ `bb92a6b2`

**判词：`DEAD`（手术刀咬掉字符串引号 → 编译当场红；失败日志后来被提交进仓，成漂流传家宝）**
证据时间：2026-09-13 实查。判定方法：闸门 1（结构/字面量配平）+ run 记录，不采信任何说法。

## 一、现场（可复跑）

- Run **30776666129** · 2026-08-03 01:21Z · `workflow_dispatch` · 分支 `workbench/fix-client-resilience-ci-v4`
  · head `bb92a6b246f3bd769eb22c43bb8eb9b53755fa9d` · 结论 **failure**
  · <https://github.com/xf8410/Agora-Workbench/actions/runs/30776666129>
  死因任务：`:app:compileFdroidDebugUnitTestKotlin`（单元测试编译，不是运行时失败）
- 日志原文（取自 `ci-failure-summary.txt`，`BUILD FAILED in 9m 24s`）：

```
e: .../app/src/test/java/com/newoether/agora/viewmodel/ConversationUiStateTest.kt:85:17 Unresolved reference 'u50'.
e: .../ConversationUiStateTest.kt:85:22 Unresolved reference 'missing'.
e: .../ConversationUiStateTest.kt:85:30 Unresolved reference 'parent'.
e: .../ConversationUiStateTest.kt:85:38 Unresolved reference 'q50'.
e: .../ConversationUiStateTest.kt:86:17 'm50'  86:22 'u50'  86:27 'a50'
e: .../ConversationUiStateTest.kt:87:17 'u51'  87:22 'm50'  87:27 'q51'
e: .../ConversationUiStateTest.kt:90:22 Cannot infer type for type parameter 'T'. Specify it explicitly.
e: .../ConversationUiStateTest.kt:90:29 'u50'  90:34 'm50'  90:39 'u51'
```

## 二、为什么这是「Python 改 Kotlin 咬坏字面量」的教科书样本

同一测试文件在分支头 `863ff7e0` 的写法是（实读，`ConversationUiStateTest.kt`，4147 B）：

```kotlin
val msgs = listOf(
    msg("u50", "missing-parent", "q50"),
    msg("m50", "u50", "a50", Participant.MODEL),
    msg("u51", "m50", "q51")
)
```

把 `bb92a6b2` 那一版的报错列号反推，坏掉的那行长这样：

```
            msg(u50, missing-parent, q50)
            ^12345678                  ← 12 空格缩进，msg( 从第 13 列起
                  ^u50=17  ^missing=22  ^parent=30  ^q50=38
```

**列号与四条报错逐一对上**（17/22/30/38）。结论：那一行的 `"…"` 引号整批消失，于是
Kotlin 把 `missing-parent` 读成减法、把本该是字符串的 `u50/q50/…` 读成标识符 → 一片
Unresolved reference，`listOf(...)` 因为元素类型全炸而推不出 `T`（正是 90:22 那条）。

也就是说：**补丁的目标语义没变（它想写的那个测试是对的），但脚本在字符串层面把代码改坏了**。
这类损坏有三个特征，都在这儿：① 只在**字面量内部**出事，缩进/括号完全正常；② 编译器报的是
「引用不到」而不是「语法不对」；③ 如果这一步被 `continue-on-error` 吞掉，界面上一切正常 ——
这正是主人点名的攻击样式。本轮它**没逃过**（run 是红的），所以定性是「被 CI 抓住的同类事故」，
而不是「逃过的」；但同一条日志后来**被当成文件提交进仓库**（见 §三），这是新的污染路径。

## 三、附带发现：`ci-failure-summary.txt` 是漂流的脏工件

同一个路径，三个 ref 上内容完全不同：

| ref | 大小 | 内容 |
|---|---|---|
| `main` | 211 B | 已被改写成一份「Memory 用户偏好与要求」笔记（文件名与内容脱节） |
| `workbench/root-fix-conversation-loss-v5` | — | 相对 merge-base **+256 行**（新增） |
| `workbench/root-fix-conversation-loss-v5-phase5-tests-scroll-anchor` | **36,938 B** | **别的分支（本条 v4）的 gradle 失败日志**，首两行写着 `run_id=30776666129` / `head_sha=bb92a6b2…` |

含义：CI 机器人写回的诊断文件被后续分支当成源码一起带走，**跨分支互相污染证据**。
→ 审计纪律补一条：**看 `ci-failure-summary.txt` 之前必须先核它头两行的 `run_id`/`head_sha`
是不是当前分支的**，否则会把 A 分支的失败算到 B 分支头上（本轮我差点就这么判错）。

## 四、遗留

- 本条判词只覆盖到 `bb92a6b2` 那次红；该分支后续提交有没有把引号补回来（分支族里
  `fix-client-resilience-ci` / `-v2` / `-v3` / `integrate-client-resilience-fixes` 四条同族分支）
  仍待逐条验，已进 §LEDGER 排队。
- 该分支的修复**要不要搬**：它属于「客户端韧性」族（与 CursorWindow 截断投影同族，见
  `hualuo-foundation/audit/agora_audit.md` #1/#3）。**不搬说法，只搬补回引号后的成品代码**，
  且必须过闸门 1 再进 `hualuo-repo-tool`。
