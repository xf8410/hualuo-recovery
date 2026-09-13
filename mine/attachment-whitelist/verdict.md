# 判词：附件发出去看不见 · 白名单硬编码（`UserMessageBubble.kt` @ main）

证据：2026-09-13 实读 `app/src/main/java/com/newoether/agora/ui/chat/message/UserMessageBubble.kt`
（main 版，blob `7f85bbae`，**15,102 B，全文读完**）。

## 一、原来就是这么写的（病灶三条，全在渲染层）

### 病灶 1：type 白名单硬编码在 UI 里，不在表里就**一个字都不显示**

```kotlin
val metaOnlyItems = meta?.items
    ?.filter { it.imageIndex == null && (it.type == "file" || it.type == "pdf" || it.type == "image") }
    ?.map { Triple(-1, "", it) }
    ?: emptyList()
```

- 白名单只有 `file` / `pdf` / `image` **三个字符串字面量**，写死在 Composable 里；
- `metaOnlyItems` 只收 `imageIndex == null` 的条目；`imageItems` 只从 `message.images` 生成。
  → **两个集合的交集之外的条目不存在**：`video`（且没抽到帧，`imageIndex == null`）、
  `audio`、`sheet`、`docx`，以及任何被上游写成别的 type 值的条目，
  **既不进 imageItems（images 里没有它），也不进 metaOnlyItems（type 不在白名单）** →
  气泡里干净得像没发附件，而数据库里那条 meta **好好躺着**。
- 这就是「静默不渲染」的机制：**没有 else、没有兜底、没有一行提示**。
  发消息方（数据库/解析器）是对的，显示方偷偷丢数据。

### 病灶 2：`coerceAtLeast(0)` 把「找不到」伪装成「第一个」

```kotlin
val mediaIndex = allMediaUrls.indexOf(
    when (type) { "video" -> metaItem?.originalUri; else -> imagePath }
).coerceAtLeast(0)
```

`indexOf` 返回 −1 = **不在媒体列表里**（正是病灶 1 那类条目，或 `originalUri` 为 null 的 video）。
`.coerceAtLeast(0)` 把 −1 变成 0 → **点开附件，打开的是列表里第一个媒体**，不报错、不提示。
这是「错位显示」级 bug：用户看到的是**另一个文件的预览**。

### 病灶 3：warning 只有 PDF 才出声

```kotlin
if (type == "pdf" && metaItem?.warning != null) {
    Text(metaItem.warning, …, color = Color(0xFFE53935), maxLines = 1, overflow = TextOverflow.Ellipsis)
}
```

- `warning` 字段**对非 PDF 一律丢弃** → 上游明明标了警告（比如解析失败、体积超限），UI 不提；
- 而且这条文字 `maxLines = 1` + `Ellipsis` → **对 PDF 也只显示一行截断**；
- 颜色是硬编码 `Color(0xFFE53935)`（**红线级魔法数字**，不走主题，深色/浅色都不管）。

## 二、刀疤（同一个文件里，证明它也被脚本改过）

```kotlin
                    val hasMetaItems = message.attachmentMeta?.items?.isNotEmpty() == true
                if (message.images.isNotEmpty() || hasMetaItems) {
```

`if` 比它的兄弟语句**少 4 格缩进**（块内其余代码正常）。同一文件末尾还有
`}` 顶到第 0 列的段落（`createNewChat`/`switchBranch` 那类在 `ChatViewModel.kt` 里也见过）。
→ 结论：**旧仓没有 ktfmt 门禁，补丁贴回来的缩进没人管**，这类疤散在多个核心文件里。
不影响编译，但**说明这些行是机器写的、没被人读过**。

## 三、五问回答

| 问 | 答 |
|---|---|
| 原来怎么写 | 见上：三集合白名单 + `coerceAtLeast(0)` + PDF-only warning |
| 有什么问题 | 静默丢显示、错位预览、警告被吞、颜色魔数、缩进刀疤 |
| 是否合并 | 这段**在 main 上**（我读的就是 `main @ dd6e7d6f`），所以「修没修」的答案是：**没修，病在主干** |
| 会不会复发 | **一定会**：type 是自由字符串，上游（解析器/导入器/新附件类型）每加一种，UI 就静默丢一种；没有任何测试或编译期穷举拦得住 |
| 接入对话 UI 了吗 | **接了**（这本身就是对话 UI 的代码）；工具卡/设置页不走这里，走 `AttachmentThumbnailItem` |

## 四、新仓的写法（不搬这段，按语义重写）

1. **type 用 `enum class` + `when` 穷举**，加类型必须改枚举 → 编译器逼你处理每一支，
   `else` 分支**必须渲染成「未知附件：<文件名> · 点击仍可查看原文」**，永不静默丢；
2. **索引找不到就报错**，不许 `coerceAtLeast(0)` 兜成第一个：`indexOf(...)`.takeIf{>=0}
   ?: 走「无预览，仅列表项」的降级态；
3. **warning 对所有类型一视同仁**，走主题色 + 可展开完整文字（不 `maxLines=1`）；
4. **加一条回归测试**：给一个 `type = "audio"`、`imageIndex = null` 的 meta，
   断言「必须渲染出至少一个条目」——旧仓正是因为没有这种断言，白名单才能静默这么多年；
5. 全文件过 ktfmt，缩进当门禁不当习惯。
