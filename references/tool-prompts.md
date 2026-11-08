# 工具提示词：写法规范与控制面描述

「用户可见的行为 = 一个工具」是本套提示词体系的组织原则：界面、命令行、MCP、AI 代理都按同一个 id 分发，因此**工具描述文本就是给 AI 的提示词**，必须按提示词标准来写。

---

## 1. 结构：一个工具描述的五个字段

来源 `crates/engine/src/tool.rs` 的 `ToolSpec` 与 `AGENTS.md` Golden rule 2。

| 字段 | 面向 | 写法 |
|---|---|---|
| `id` | 机器 | 稳定、动词化、带领域前缀（`rebuild_start`、`core_finding_record`）；改名等于破坏契约 |
| `title` | 人 | 2–5 个词的短标题，动词开头 |
| `summary` | AI / 有经验用户 | **一段话**说清「做什么 + 输入输出 + 边界 + 副作用 + 默认值/上限」，信息密度最高 |
| `beginner` | 新手 | 一句日常用语，不出现术语和 id |
| `effects` | 权限系统 | 每个副作用都要声明；运行外部代码、改动设备会触发同意流程 |

配套还有 `toolset`（分组）、`requires`（前置条件）、`input_schema`（参数结构）、`run`（实现）。

`summary` 的可复用句式（从本文逐字描述里归纳）：

```text
{做什么}：{输入是什么}。{返回什么 / 结果长什么样}。{限制：默认值、上限、排序、并发、脱敏}。{副作用或同意要求}。
```

例子（`rebuild_activity_show`）：先说「逐行列出构建做了什么」，再说「最新在最后、每行包含什么」，再给「`limit` 默认 200、最多 2000，`offset` 从最新往回」，最后补「运行中与结束后都可用、密钥会被隐藏」。四层信息一段话讲完。

## 2. 控制面：9 个 `rebuild_*` 工具描述（逐字）

来源 `crates/ai/src/rebuild/tools.rs:13-116`。`summary` / `beginner` 均为原文。

| id | title | summary | beginner |
|---|---|---|---|
| `rebuild_start` | Start a rebuild | Start building: a new project from a request and a target (a web address, an absolute path, or - for none), or a follow-up on an existing project. A coding agent studies, builds, checks and refines in the background; follow it with rebuild_status and project_show, answer its questions with project_inbox_answer. Up to `build.parallel` projects build at once (2 by default); more are queued and start on their own as others finish. It runs a coding agent that writes and builds code, so it needs consent. | Starts making something. |
| `rebuild_status` | Rebuild status | Whether a project's build is running in this session (or queued, and where), its status and note, the latest check, and how many questions are waiting. | Shows how a build is going. |
| `rebuild_activity_show` | Show a build's activity | What a project's builds did, line by line, newest last: each stage, every command and file the builder touched and how it ended, what it said, and what programs printed outside the conversation. Secrets are hidden. Works while a build runs and after. `limit` lines (200 by default, at most 2000), `offset` lines back from the newest. | Shows the details of what a build has been doing. |
| `rebuild_note` | Add to a running build | Tell a running build something more; the agent reads it at its next step. With no build running, start a follow-up with rebuild_start instead. | Adds information while it builds. |
| `rebuild_note_withdraw` | Take back a note | Take back a note given with rebuild_note (by the id it returned) while the agent hasn't read it yet; it returns the note's text. Once the agent has read it, it can't be taken back. | Takes back something you added before the builder reads it. |
| `rebuild_stop` | Stop a rebuild | Pause a project's running build. Its work so far is kept; a follow-up continues it. | Pauses a build. |
| `rebuild_question_ask` | Ask about a build | Ask a question about a project's build ("where are the latest files?", "why this engine?") without starting a build or a round, and without changing anything. Claude Code answers from a copy of the builder's own session, which isn't kept, reading files only (none in Clean room); an AI with an API key answers with the tools that change nothing. Other coding agents are refused. Safe while a build runs: it doesn't interrupt it. Returns the answer, how it was answered and its cost, and keeps it in the project (`rebuild_questions_list`). | Asks about what was made, without changing it. |
| `rebuild_questions_list` | List questions about a build | The questions asked about a project's build and their answers, oldest first (the newest 200). | Shows what you've asked about this project. |
| `rebuild_check` | Check my build | For the builder: test the project's current build against the original now, with the same checks Craft Anything runs after each turn (a website's likeness at each width with the padding, colour, font and layout differences; an app's or game's picture against the original's; whether it builds; the mechanics sheet; credits). The project's running build answers it, so the builder can fix what it lists and check again before replying. Not counted as a round. project defaults to the project this tool server works for. | Lets the builder test its work against the original while it builds. |

**注意三个高频技巧**：
- 在描述里直接写清「什么时候不该用它，该用谁」（`rebuild_note` 指向 `rebuild_start`）。
- 把上限、默认值、排序写进描述，避免调用方猜。
- 明确声明副作用与并发/排队行为，让同意流程有依据。

`effects` 声明示例：`rebuild_start` = 启动进程 ∪ 执行外部代码 ∪ 写文件系统 ∪ 访问网络；`rebuild_status` = 只读；`rebuild_check` = 写文件系统（会落检查产物）。

`rebuild_check` 的收件箱请求文本（`tools.rs:474`）是极简的 `Check the build`——控制面消息越短越不易被误读。

## 3. 工作记忆与分析工具描述（逐字选摘）

来源 `crates/engine/src/tools/investigation_tools.rs`。

| id | title | summary |
|---|---|---|
| `core_investigation_show` | Show the investigation | The working memory of this investigation: brief, plan with step status, hypotheses, findings (with evidence ids) and the latest handoff. Read it first when starting or resuming. |
| `core_brief_write` | Write the brief | Set the investigation brief: the question, purpose, targets, constraints and what 'done' means. Write it before the first analysis call on a new question. |
| `core_plan_update` | Update the plan | Replace the plan with a numbered list of steps, each todo/doing/done/blocked with a short why. Pass based_on_revision to avoid overwriting someone else's update. |
| `core_hypothesis_record` | Record a hypothesis | Record a claim being tested, ideally with the prediction that would confirm or refute it, plus any supporting/contradicting evidence ids. |
| `core_hypothesis_update` | Update a hypothesis | Change a hypothesis's status (testing/supported/refuted/abandoned) and attach supporting or contradicting evidence ids and notes. |
| `core_finding_record` | Record a finding | Record a settled answer. Requires at least one evidence id; every id is checked against the ledger. Findings are the source for reports. |
| `core_finding_edit` | Correct a finding | Correct a recorded finding without changing its id. Requires a statement and existing evidence ids; pass based_on_revision to refuse a stale investigation update. |
| `core_handoff_write` | Write the handoff | Rewrite the handoff note: where we are, what's next, traps to avoid. Write it at the end of every turn so the next turn (or agent) resumes without re-deriving. |
| `core_journal_read` | Read the journal | Page through the project's append-only journal (tool calls, model turns, consents, errors), filtered by day, kind or tool. Needs an open project. |

`beginner` 文本举例（同一批工具）：`See what we're trying to find out, the plan, and what we know so far.` / `Write down the question we're answering and what a good answer looks like.` / `Update the step-by-step plan.` / `Save something we now know for sure, with the proof.` / `Leave a note so we can pick up exactly where we left off.`

并发保护写法值得照搬：`Pass based_on_revision to avoid overwriting someone else's update.`

## 4. 复刻账本与版本对比工具描述（逐字）

来源 `application_tools.rs` 与 `compare_tools.rs`：

```text
app_feature_follow      Follow a feature through an application graph from a seed (module, function, string, IPC channel, endpoint, route, bridge API or node id): what leads to it and what it leads to, across renderer ↔ preload ↔ main, with readable chains and a one-line story.
app_graphs_compare      Compare two application graphs: modules (unchanged, changed, moved by content, renamed by structure, added, removed), bundle modules (content → structure → id), IPC channels and handlers, bridge members, endpoints, routes, window webPreferences, native modules and vendor versions.
app_ledger_build        Derive the expectations a clean-room rebuild must meet from an application graph ... Ledger status: ready, open, failing or unclear.  （全文见 contracts.md）
app_coverage_evaluate   Summarise a rebuild ledger: the share of in-scope expectations proven, counts by layer and status, and the open gaps with what each still needs.
app_rebuild_claim_check Check a reconstruction specification: every expectation must cite evidence that exists and is intact. Reports which expectations are satisfied.
app_function_versions_compare  Compare two native_function_study evidence records: name, size, blocks, instructions, callee set and pseudo-code. Reports what changed.
app_call_path_find      Find a path of calls from a start address to a goal address over a call graph assembled from native_function_study evidence records.
core_evidence_exports_compare  Compare two exported evidence bundles ... Every record is checked against its own id, so one edited after export is reported, never counted as unchanged.
```

`beginner` 文本举例：`Make a checklist of everything a rebuilt app must do, and tick items off only with proof.` / `See how much of a rebuild is proven and what's still missing.` / `Check that a claim about rebuilding something is actually backed by proof.` / `Follow one feature (like "Export") from the button the user clicks to what the app finally does.`

## 5. 渐进式工具组与资源

来源 `crates/mcp/src/lib.rs`。只先列出一小组核心工具，需要时再启用，避免一次性把几百个工具塞进上下文。

```text
list_toolsets   List the groups of tools (native code, Android, web, Electron, runtime…) with how many tools each has and whether it is enabled in this connection.
enable_toolset  Add a group of tools to this connection's tool list (the list then changes). Use list_toolsets to see what each group covers.
```

资源：`evidence://{evidence_id}`（按编号取回保留的证据记录）、`investigation://brief|plan|hypotheses|findings|handoff|state`。结果超过连接字节预算时不返回裸错误，而是折叠成摘要：

```text
The result is {SIZE} bytes, more than this connection's {BUDGET} byte budget.
{"saved_evidence": {"kind": "saved", "evidence_id": …, "uri": "evidence://<id>"}, "continue_with": "core_evidence_browse", "evidence_summary": …, "limit_bytes": …}
```

**可复用要点**：超限响应里直接给出「下一步该调谁」（`continue_with`），比只报错友好得多；这个字段应成为所有大结果工具的一致约定。

工具可见性规则（`lib.rs:386-390`）：`visible(toolset) && (!clean_room || clean_room::allows(id))`。只看外部模式下，白名单之外的工具**根本不出现在列表里**，而不是列出来再拒绝——提示词约束与工具可见性必须一致，否则执行者会反复尝试被拒的调用。

## 6. 写工具描述时的检查清单

- [ ] `summary` 是否说清输入、输出、排序/上限/默认值、边界、副作用？
- [ ] 是否写明「什么时候不该用它」以及替代工具？
- [ ] 是否会触发同意流程？`effects` 是否如实声明？
- [ ] `beginner` 是否完全没有术语、id、文件路径？
- [ ] 大结果是否约定「摘要优先 + 编号 + 分页 + `continue_with`」？
- [ ] 并发写入是否要求 `based_on_revision` 之类的乐观锁？
- [ ] 只读工具是否明确声明「不改变任何东西 / 不打断正在运行的任务」？
- [ ] 名称是否稳定、按领域前缀分组，便于渐进式启用？
- [ ] 描述里是否避免出现真实路径、密钥、个人数据？
