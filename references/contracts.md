# 可机读验收契约

检查阶段能不能自动判定，取决于**首轮简报有没有把契约写清楚**。这些段落必须与构建指令一起发给执行者，不能等到检查时再提要求。

---

## 1. 设计原则

1. **让产物自己产生可判定的证据**：程序自带截图 / 自检选项，写文件、正常退出，不依赖外部托管或人工观察。
2. **每个数值带出处**：来自文件、函数地址或证据编号；查不到就显式标记 `estimated`，并说明用什么替代。
3. **契约给出完整数据形状**：直接给 JSON 示例，执行者不必猜字段名。
4. **失败的 note 要具体**：说清缺哪个值、哪个宽度差多少、哪一步没通过，供精修指令直接引用。
5. **检查顺序固定**：构建 → 视觉 → 机制数值 → 核心循环 → 声明与版权，任一失败即停止后续。
6. **「没比较」不等于「通过」**：无法比较的部分单独列出，并要求「让它存在、让它渲染」后再检查。

---

## 2. 四类契约（逐字原文）

### 2.1 机制数值表 `MECHANICS_CONTRACT`

来源 `crates/ai/src/rebuild/kinds/mechanics.rs:12`。所有游戏（以及深入模式的桌面程序）简报都会拼上这段：

```text
Keep the same values in app/mechanics.json, which Craft Anything reads after each build: {"version": 1, "values": [{"name": "walk speed", "group": "movement", "value": 4.5, "unit": "m/s", "source": "function 0x140012a0"}, {"name": "crouch", "group": "controls", "value": "hold", "estimated": true}]}. A value is a number, text or true/false; give each one its source or "estimated": true, and use the sheet's numbers in the code as they're written there.
```

检查时生成的 note（`mechanics.rs:51,68`）：

```text
app/mechanics.json has {N} values ({SOURCED} with a source, {ESTIMATED} estimated).
 {N} numbers from the original aren't in the code as written: {NAMES}. Use each in the code as the sheet writes it (build what uses it if it's missing), correct the sheet, or mark it "estimated" if it wasn't found in the original.
```

设计意图：数值表既是给机器读的检查依据，也是给人读的 `app/MECHANICS.md` 的机读镜像；检查会验证表里的数字确实出现在代码里（而不是写完就搁置）。

### 2.2 游戏核心循环自检 `SELF_TEST_CONTRACT`

来源 `crates/ai/src/rebuild/kinds/self_test.rs:13`：

```text
Give the game a `--self-test <file.json>` option too: start a game the way a player would, drive the player with scripted input for a few seconds, measure from the game's own state whether each step of the core loop works, save the results to that file and exit: {"version": 1, "steps": [{"step": "move", "passed": true, "detail": "walked 3.1 m forward"}, {"step": "act", "passed": true, "detail": "fired 5 shots, 2 hit"}, {"step": "hud", "passed": true, "detail": "health, ammo and the objective are drawn"}]}. "move": the player (or what they steer) moves; "act": their main action works (firing, mining, jumping, eating…); "hud": the on-screen information the original shows (score, health, ammo…) is drawn. Craft Anything runs it after each build, and the game isn't ready until every step passes.
```

三步核心循环（`self_test.rs:16`，`CORE_LOOP`）：

```text
move — the player moves
act  — the player's main action works
hud  — the on-screen information is drawn
```

### 2.3 程序自截图 `SCREENSHOT_CONTRACT`

来源 `crates/ai/src/rebuild/kinds/screenshot.rs:16`。网站可以从外部拍照，桌面程序与游戏不行，所以要求产物自带截图能力：

```text
Give the program a `--screenshot <file.png>` option: start normally, draw about 120 frames, save the window's picture as a PNG to that file and exit. Craft Anything runs it that way after each build to check the result, and a build without the option, or with a blank picture, doesn't pass. Try it yourself: run it with a file inside app/ (for example app/shots/now.png) and look at the picture.
```

`contract()` 追加（`screenshot.rs:19-28`）：

```text
 If not packaging an executable, keep a supported entrypoint directly in app/ ({ENTRIES}); wildcard entrypoints must match exactly one file.
```

`NO_PROGRAM`（`screenshot.rs:297`）：`There's no built program to start yet.`

### 2.4 命令行程序的运行记录 `RUNS_BRIEF`

来源 `crates/ai/src/rebuild/kinds/app.rs:9`：

```text
The original is a console program, so it's compared by its runs: capture/runs/ holds, for each input it was run with, what it printed and how it exited. Make the rebuild a console program that prints the same text for each input (its own name may differ) and exits the same way. Craft Anything runs yours with the same inputs after each build and checks each one; rebuild_check shows you the result while you work.
```

默认输入集合（`kinds/runs.rs` 的 `CASES`）：无参数、`--help`、`--version`。每个记录含参数、退出码（或超时）、输出行。采集失败时写（`runs.rs:140-147`）：

```text
It's a console program, but running it to see its output didn't work.
It's a console program. capture/runs/ holds what it printed and how it exited for each of these inputs: {CASES}. Each file has the arguments, the exit code (or that it was still running after {N} s) and the lines it left on the terminal.
```

### 2.5 网站布局量值契约

不是单独常量，而是写在 `WEBSITE_FROM_OUTSIDE` / 深入模式网站说明里（逐字原文见 `references/prompts-build.md` 第 5 节）。要求执行者：

- 从 `capture.json` 取**精确值**（颜色、字体、字号、padding、gap、圆角、内容宽度），写成项目自己的主题令牌，而不是用组件库默认值；
- 按每个宽度 `layouts` 项改变布局（区块堆叠、每行列数、导航切换），深入模式还要对齐 `breakpoints`；
- 手机宽度不允许横向滚动；
- 自变区域（噪点图标出的轮播/视频/动画）不参与逐帧比较，只匹配观感；
- 检查环节用**同一套测量方式**在**同一组宽度**上测量构建产物并逐项报差。

---

## 3. 复刻声明校验：guided prompt

### `app_rebuild_claim_check`（逐字）

来源 `crates/mcp/src/prompts.rs:82-97`：

```text
title: Verify a rebuild
description: Check a rebuilt implementation against the evidence about the original.
objective: Decide, expectation by expectation, whether the rebuild matches the original, and say what is still unproven.
steps:
1. Read the investigation findings; they are the list of behaviours to match.
2. Turn each finding into a checkable expectation with the evidence that defines it.
3. Check each expectation against the rebuild and mark it met, not met, or not yet testable.
4. Record a finding per expectation and an unknown for each gap.
```

必填参数 `goal`（复刻要重现什么），可选 `comparison_evidence`、`original_path`。

### 统一渲染模板（逐字，`prompts.rs:182-198`）

```text
# {TITLE}

**Objective:** {OBJECTIVE}

**Inputs:**
- {NAME}: {VALUE}

**Steps:**
1. {STEP}

**Rules:**
- {DISCIPLINE 每条}
```

`DISCIPLINE` 六条逐字原文见 `references/discipline.md`。缺必填参数时返回 `Missing required argument {NAME}`，不要生成半截提示词。

### 其余五个引导式提示词

| name | title | objective | 与复刻的关系 |
|---|---|---|---|
| `core_feature_explain` | Explain a feature | Explain how the feature works, from where it starts (UI action, command, entry point) through the data it touches to the result, backed by evidence. | Study 阶段追行为 |
| `app_graphs_compare` | Compare two versions | Describe the behavioural differences between the two versions and how sure we are of each. | Check 阶段做版本对比 |
| `native_crash_explain` | Explain a crash | Find the code path that leads to the crash and the condition that triggers it. | 深入模式辅助 |
| `core_unknowns_review` | Review open questions | Leave every unknown in scope either resolved with evidence, narrowed with a next probe, or explicitly blocked with a reason. | 收敛前清理开放问题 |
| `runtime_run_plan` | Plan a short run | Design the smallest run that answers the question, with limits, before anything is executed. | 需要运行原版时先规划并取得同意 |

`core_feature_explain` 的六步（`prompts.rs:53-60`）可直接作为 Study 阶段的功能追踪模板：

```text
Open the target (core_target_open) unless it is already open; check core_session_show.
Write the brief (the feature question and what a complete answer looks like) and a short plan.
Locate entry points: search strings, symbols, routes or IPC channels that name the feature.
Follow the path: callers and callees, data flow, files or network endpoints involved.
Record each claim as a hypothesis with a prediction, test it, then record findings with evidence ids.
Summarise: the path step by step, the evidence for each step, and the unknowns that remain.
```

### 账本与声明工具描述（逐字）

来源 `crates/engine/src/tools/application_tools.rs` 与 `compare_tools.rs`：

```text
# app_ledger_build — Build a rebuild ledger
Derive the expectations a clean-room rebuild must meet from an application graph (IPC handlers, bridge members, windows, endpoints, routes, storage, file/process effects, native modules, entry point) and mark each one proven only when the manifest gives it one owner, a parser type where needed, every case it must cover with strong enough evidence, and a passing check with its result. Ledger status: ready, open, failing or unclear.
Beginner: Make a checklist of everything a rebuilt app must do, and tick items off only with proof.

# app_coverage_evaluate — How complete is the rebuild
Summarise a rebuild ledger: the share of in-scope expectations proven, counts by layer and status, and the open gaps with what each still needs.
Beginner: See how much of a rebuild is proven and what's still missing.
动态摘要: {}% of {} in-scope expectation(s) proven; ledger {}.

# app_rebuild_claim_check — Check a rebuild claim
Check a reconstruction specification: every expectation must cite evidence that exists and is intact. Reports which expectations are satisfied.
Beginner: Check that a claim about rebuilding something is actually backed by proof.
```

**核心判定规则**：一条期望只有同时满足「有唯一负责人 + 需要的解析器类型 + 覆盖每种情况的足够证据 + 一项通过的检查及其结果」才算 proven。这条规则可以脱离具体工具，直接写进任何复刻项目的验收标准。

---

## 4. 迁移到自己的项目

1. 先列出「哪些事实可以被程序自动判定」，再为每一项设计一个产物（JSON 文件、退出码、截图、日志行）。
2. 把产物的数据形状（字段名 + 示例）写进首轮简报，和构建指令一起发出。
3. 给每个产物配一个明确通过条件与失败 note 模板，note 里带上可操作的差值或缺失项。
4. 让检查环节的测量方式与 Study 阶段的采集方式共用同一份配置（宽度、输入集合、时间点）。
5. 执行者能自检时，提供「与正式检查同口径」的自检入口，并规定自检不计轮次。
6. 把「未比较」「估算」「原版未知」三类单独建模，别混进通过/失败的二元判定。
7. 版权与素材来源（例如 `CREDITS.md`、私有使用声明）也做成可检查项，逐字文本写进契约里，检查环节按字面核对。
