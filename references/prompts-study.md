# Study 阶段：采集动作与证据产物

Study 是后续所有阶段的唯一事实来源。核心原则：**采集用的宽度必须与 Check 阶段比较用的宽度完全一致**（源码 `study.rs:23-24` 的注释：「The widths the check compares at, so every picture has its pair」），否则每轮检查无法逐对比较。

---

## 1. 采集顺序

### 网站（`study.rs:13-41`）

| 步 | 动作 | 产物 |
|---|---|---|
| 1 | 在私有浏览器打开目标 | 会话 |
| 2 | 按各宽度截图，并做一次噪点比对（`noise_pass: true`） | `screenshot-<width>.png`、`noise-<width>.png`、`capture.json`、概览 |
| 3 | 逐个打开菜单 / 弹窗 / 标签页，桌面宽度与手机宽度各拍一组 | `capture/states/`、`capture/phone-states-*/` |
| 4 | 沿同站链接抓其它页面（有上限） | `capture/routes/<slug>/`、`routes.json` |
| 5 | 仅深入模式：分析页面脚本组成 + 观察起始页约 10 秒行为 | `capture/how-it-works.md` |

### 本地程序（`study.rs:42-128`）

| 情形 | 动作 | 产物 |
|---|---|---|
| 只看外部 | 不读取目标内部；只写一份「看到了什么」的说明 | `capture/README.md` |
| 只看外部 + 允许运行观察 | 启动一次，等待约 15 秒后拍窗口与控件树 | `capture/window.png`、`capture/ui-tree.json` |
| 允许深入 | 静态概览 → 准备并等待反编译器 → 应用图 / 文件清单 | `capture/README.md`（含证据记录） |
| 无目标 | 只写「从请求出发」 | `capture/README.md` |

设计约定：单个采集步骤失败不阻断整体，把失败原因写进笔记（源码 `how_it_works()` 的注释「A part that fails is said, not fatal」）。这与「缺失不是证据」一致——采集失败本身也要作为事实记录下来。

## 2. 证据产物清单

| 文件 | 内容 | 用于 |
|---|---|---|
| `README.md` | 概览：颜色、字体、间距、各宽度布局、模式说明 | 先看这份，再深入 |
| `capture.json` | 设计量值：十六进制颜色、间距、内容宽度、断点、`layouts`（每宽度的页宽、标题与正文字号、各区块 padding/gap/内容列/每行项数）、CSS 像素坐标的布局轮廓、标题、链接、表单轮廓、图片尺寸、可见文本 | 构建时取精确值，不做「差不多」 |
| `screenshot-<width>.png` | 桌面 / 平板 / 手机三个宽度 | 逐宽度视觉对比 |
| `noise-<width>.png` | 自行变化的部分（轮播、视频、动画）标成白块 | 这些区域不参与比较，只需风格接近，不比对具体帧 |
| `page.html` | 渲染后的 HTML（仅深入模式） | 结构参考 |
| `how-it-works.md` | 模块、路由、请求、依赖库 + 十秒页面行为（请求、控制台、WebSocket） | 行为重建 |
| `routes/<slug>/`、`routes.json` | 其它页面，各自按同样方式采集 | 多页面一致性 |
| `runs/` | 每个输入下的输出、退出码、是否超时 | 命令行程序对比 |
| `window.png`、`ui-tree.json` | 运行 15 秒后的窗口图；控件顺序、角色、标题、标识、值、位置与像素尺寸 | 桌面程序布局与控件命名 |

## 3. 逐字原文：写进 `capture/README.md` 的说明

### 无目标（`study.rs:126`）

```text
# No target

Build from the request alone.
```

### 只看外部（`study.rs:161-164`，`{PATH}` 为目标路径）

```text
# Clean room

Nothing inside {PATH} was read: its files, its folder and its code stay closed, and the tools that look inside are off. Build from the person's request and description and the pictures in reference/, and ask them with core_person_ask when something isn't clear.
```

### 深入模式的说明头（`study.rs:139,46,61`）

```text
# How it works

Deep mode: what the site's JavaScript is made of and what its start page did while we watched.
```

```text
# What we found in {PATH}

```

```text
## Ghidra

No program was found in {PATH}, so Ghidra couldn't be used.
```

### 窗口与控件树（`study.rs:180,184`）

```text
capture/window.png is a picture of its window 15 seconds after it started. Open it with the Read tool: match its layout, colours and proportions.
```

```text
capture/ui-tree.json lists the controls in its windows at that moment, in order (role, title, identifier, value, position and size in pixels): use it for the names, order and placement of its menus, buttons and fields.
```

采集失败时（`study.rs:187`）：`Starting it for a picture didn't work ({ERROR}), so there's no picture of it running.`

## 4. 逐字原文：证据包装格式

来源 `study.rs:290-298`，`describe()`。每条工具结果都按「摘要 → 证据编号 → 结构化正文」写入笔记，正文截断上限 20,000 字符。

````text
ToolOutput::Evidence(e) => format!("{}\n\nEvidence {}.\n\n```json\n{}\n```\n", e.summary, e.id, body.chars().take(20_000))

ToolOutput::Value(v)    => format!("```json\n{}\n```\n", serde_json::to_string_pretty(v).unwrap_or_default().chars().take(20_000))
````

产出的记录形如：

````text
{SUMMARY}

Evidence {ev_…}.

```json
{ ... 结构化结果 ... }
```
````

**通用化要点**：摘要 → 编号 → 正文 三段式让执行者可以先读概览、按编号回查细节；截断上限防止单条记录撑爆上下文。自己实现时保留这三段和上限，编号格式可换。

## 5. 给 builder 的证据说明

证据目录「是什么、怎么读、不能碰什么」这段提示词并不写在 Study 阶段，而是作为首轮简报的 `{CAPTURE}` 段落发给执行者，共四个分支（网站/程序 × 只看外部/允许深入），逐字原文见 `references/prompts-build.md` 第 5 节。

## 6. 迁移到自己的项目

1. 先定 Check 要测什么，再倒推 Study 要采什么——顺序反了会采一堆用不上的数据。
2. 采集参数（宽度集合、时间点、输入集合）与检查参数共用同一份配置。
3. 自变区域单独标记，不计入相似度，只做风格匹配。
4. 打开态、多页面、运行输出要单独采集，默认截图覆盖不到。
5. 每个采集项写明「失败时如何记录」，避免静默缺失被当成「原版没有」。
6. 模式边界写进 `capture/README.md`，让执行者每次都能看到，而不是只靠系统提示词约束。
