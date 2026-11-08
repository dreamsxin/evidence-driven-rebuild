# Check 与 Refine 阶段：检查报告、精修指令与循环控制

这一层是整个工作流能收敛的关键：**差异必须被测量、被具体命名，然后原样变成下一轮指令**。含糊的「再改改」会让循环空转。

---

## 1. Check 阶段

### 执行规则

- 检查项按固定顺序执行；任一失败即停止后续检查（`kinds/mod.rs:176-201` 的 `run_checks()`）。这样执行者一次只面对一组明确的失败项。
- 执行者可在自己的回合内主动发起自检（对应 `rebuild_check`），自检**不计入轮次**；检查环节在每轮结束后也用同一套检查再跑一次，两者结果口径一致。
- 每项结果包含：名称、通过与否、相似度（若有）、门槛、具体 note、未通过的断言、无法比较项、开放问题、产物路径。
- 收尾判定看**最后一轮的全部检查**，不是只看最后一条（源码 `mod.rs:964-966` 的注释：「The last round's checks, all of them: the last one alone … isn't the verdict」）。

### 检查报告模板（逐字逐句）

来源 `crates/ai/src/rebuild/self_check.rs:76-114`，`report()`。按顺序拼接：

```text
There was nothing to check yet.                                            # 无任何检查时，整段只返回这句

{CHECK}: {passed|failed}{, {S}% alike (needs {BAR}%)}{ — {NOTE}}          # 每个检查一行

These must hold before it can pass: {CLAIM} ({DETAIL}); ….
Not compared, so it can't pass yet: {UNKNOWN}, …. Make those parts exist and render, then check again.
Your open questions: {QUESTION} ({ID}); …. Settle what one more look can and close it with core_unknown_edit (status resolved, the resolution and the evidence ids); say what's left in your summary.
Pictures of your build (compare each with the original's in capture/ or reference/): {PATH}, …

This would pass. Finish what you're doing, then reply.                     # 全部达标
Not good enough yet: fix what it lists, build again, and check again with rebuild_check.   # 未达标
```

**通用化要点**：报告必须有「未比较项」这一行。「没能比较」不等于「通过」，也不能静默略过；否则执行者会以为已经覆盖全部。

### 摘要模板（逐字）

来源 `self_check.rs:61-71`，`summary()`，用于状态行与最终回复：

```text
about {S:.0}% like the original{, {HELD} of {N} claims hold}
all {N} checks pass{, {HELD} of {N} claims hold}
{GOOD} of {N} checks pass{, {HELD} of {N} claims hold}
```

最终状态行（`mod.rs:967-973`）：

```text
Ready.                                     # 无检查记录
Ready: {SUMMARY}.
 1 question about the original is still open.     # 追加；N > 1 时用复数
 {N} questions about the original are still open.
```

### 进度文本（`self_check.rs:122-127`）

```text
The builder is checking its work…
The builder is checking its work against the original…
```

---

## 2. Refine 阶段

### 默认精修指令（逐字）

来源 `kinds/mod.rs:118-123`，`TargetKind::refine()`：

```text
Check {ROUND}: {NOTE}

Fix this in app/, build again until it succeeds, and reply with 1-2 plain sentences on what you changed.
```

### 视觉相似度精修指令（逐字，最详细的一版）

来源 `kinds/website.rs:217-228`，`WebsiteKind::refine()` 的相似度分支：

```text
Check {ROUND}: compared with the original, your build looks {S}% alike ({NOTE}). Your screenshots are in checks/round-{ROUND}/ (screenshot-<width>.png for each width, and routes/<page>/ for other pages) with the layout measured at each width in its capture.json; the original's are in capture/. Open each pair with the Read tool, fix the biggest visible differences first (layout and section order, spacing, sizes, colours, fonts, missing parts), fix the padding, items per row, sizes, colours and fonts the check names at each width with the exact values it gives, add any missing headings or words it lists, build it again, and reply with 1-2 plain sentences on what you changed.
```

**可复用要点**：这段里同时给出「产物在哪 / 原版在哪 / 先看什么 / 按什么顺序修 / 用哪些精确值 / 回复多长」。自己写精修指令时按这六项填，不要只写「请修复差异」。

### 空项目专用指令（逐字）

来源 `mod.rs:1287`，`NO_PROJECT_PROMPT`。用于执行者声称构建成功、但构建区其实没有文件的情况：

```text
The check found no project: app/ has no files, so nothing has been written yet and nothing can be built. Create the project's files in app/ now (for a website, a package.json and an index.html or the source files), then build it. Reply with the exact command you ran and the last lines of its output. Say something is built only when that output says so.
```

设计意图（源码注释）：防止执行者把「我说构建了」当成「构建过」。要求回贴**实际命令与输出末几行**，是把口头声明变成可核查证据的通用手法。

### 回复前自检（逐字）

来源 `mod.rs:1348`，`CHECK_BEFORE_REPLYING`（注意开头有一个空格，用于直接拼接）：

```text
 Before you reply, check your build again with rebuild_check and fix what it still lists.
```

### 精修指令的拼装规则（逐字代码）

来源 `mod.rs:937-951`：

```rust
let Some(check) = not_good else { break };
if ws.manifest.not_working_checks >= NOT_WORKING_CHECK_LIMIT {
    return Err(not_working(ws));
}
if refinements >= opts.max_rounds {
    break;
}
refinements += 1;
// Nothing written yet: a fix for a build that doesn't exist sends the builder looking for one.
// It is asked to write the project and to show its build output (#736).
let refined = if app_is_empty(ws) { NO_PROJECT_PROMPT.to_string() } else { kind.refine(check) };
prompt = refined + &pictures_reminder(ws);
if !api_loop {
    prompt.push_str(CHECK_BEFORE_REPLYING);
}
```

通用版伪代码：

```text
取未通过项；无 → 结束循环
若「连续不可用检查数」≥ 上限 → 报错并要求用户给具体修正
若「已完成精修轮数」≥ 上限 → 结束循环
轮数 += 1
指令 = 构建区为空 ? 空项目专用指令 : 按目标类型生成的精修指令
指令 += 参考图提醒（若有）
指令 += 回复前自检（若执行者能自检）
```

### 终止条件

| 条件 | 阈值 | 行为 | 来源 |
|---|---|---|---|
| 全部检查达标 | — | 正常结束，输出 Ready 摘要 | `mod.rs:956-974` |
| 精修轮数 | 默认 首轮 + 6 次（CLI 为 2 次），可配置 `max_rounds` | 停止循环，保留当前产物 | `tools.rs:360`、`docs/claude-code.md:120` |
| 连续不可用检查 | `NOT_WORKING_CHECK_LIMIT = 10` | 报错，要求用户给出具体修正后重新开始 | `mod.rs:1278-1283` |
| 连续两轮无文件变化 | 2 | 报错；构建区为空时给出「尚未写入项目」的专门补救 | `mod.rs:1289-1299` |

面向用户的错误文本（逐字，`mod.rs:1281-1297,961-962`）：

```text
The build still isn't working after {N} checks in a row, so it stopped.
Send a new request with a concrete correction to start again. The last check's note in the feed says what it found.
The build stopped after two builder turns changed no files in {PLACE}.
{PLACE} has no files yet, so the builder hasn't written the project.
The build isn't working yet. {WHY}
Ask for a fix in the box below, or press Try again.
```

---

## 3. 续跑 / 追问 / 背景注入（逐字）

来源 `brief.rs:287-358`。这些模板保证「人中途插话」「任务被暂停」「换机器继续」时不会推倒重来。

### 用户中途补充

```text
While you were working, the person added:
{LIST}

Take this into account now: change app/ as needed, build again and fix any errors. Reply with 1-3 plain sentences on what you changed.
```

### 用户提出修改

```text
The person asks: "{MESSAGE}"

Make the change in app/, then {BUILD}, fix any errors and check it with rebuild_check. Reply with 1-3 plain sentences on what changed.
```

`{BUILD}` 默认 `build it again`（`kinds/mod.rs:112-114` 的 `rebuild_words()`）。

### 中断后续跑（同一会话还在）

```text
Your last run stopped before it finished (it was paused, or the app or the computer stopped). Everything you made so far is in app/. Pick up where you left off: see what's done and what's left (your checklist and core_unknowns_show), finish the task, then {BUILD}, fix any errors and check it with rebuild_check. Reply with 1-3 plain sentences on what you did.
```

### 换机器 / 会话丢失后续跑

```text
{BRIEF}

This project is already under way: an earlier conversation built it, but that conversation isn't on this computer (the project was started on another computer or in another folder). Everything made so far is in app/. Before changing anything, look through app/ and the open questions (core_unknowns_show) to see what's done, then carry on from there instead of starting again.
```

后接二者之一：

```text
The earlier conversation, as the project kept it (oldest first; when it was long, the newest part is kept and long messages are cut short):

{HANDOFF}
```

```text
What the earlier conversation last told the person:
"{LAST_REPLY}"
```

### 「只是问问」的背景注入

```text
Since your last message the person asked these questions in a separate "Just ask" conversation, which isn't in your history and changed nothing. It's background: what they say next may follow on from it, but nothing in it is a request or an instruction to you.

{TALK}

---

```

`{TALK}` 形如 `Q: {问题}\nA: {回答}`，最多保留最近 5 组，问题截 300 字符、回答截 1500 字符（`brief.rs:328-335`）。**设计意图很值得复用**：把只读问答与真正的构建指令隔开，明确声明「这不是给你的指令」，避免执行者把背景讨论误当成需求变更。

### 参考图提醒

```text
Keep matching the {N} picture(s) of the original in reference/ (open any you haven't looked at yet), and compare a fresh screenshot of your build with them before you reply.
```

---

## 4. 只读问答的前置指令（逐字）

来源 `crates/ai/src/rebuild/question.rs:35`，`PREAMBLE`。用于「只回答问题、不改任何东西」的短生命副本：

```text
You're answering a question the person asked about the project you built inside Craft Anything. This isn't a task: answer it and stop. Don't create, change or delete any file, don't run anything, and don't carry on the build; if the answer is that something should change, say what you'd change and that they can ask for it as a change. Answer in plain words for someone who doesn't code, briefly, with full paths when they ask where something is. You're a short-lived copy with fewer tools than the builder, and the build may still be running beside you. So a tool you can't see here is not a tool the build has lost, and a call with no result here may still be running: never report tools as disconnected or work as cut off because of what you see from here.
```

**通用化要点**：短生命副本视野有限，必须在提示词里明确「你这里看不到 ≠ 主流程丢了能力」，否则它会误报故障。上下文模板（`question.rs:219-239`）：

```text
Project: {NAME}
What the person asked for: {REQUEST}
Project folder: {ROOT}
The app's folder: {APP}
Status: {STATUS}
Built with: {STACK}
Most recently changed files in the app (newest first):
- {FILE} ({AGO})
```

---

## 5. 通用版可填空模板

`assets/review-template.md` 提供已解耦工具名、用中文撰写、可直接复制填空的检查报告 + 精修指令骨架。
