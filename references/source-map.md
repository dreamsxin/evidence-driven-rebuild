# 来源索引、变量字典与解耦映射

本 Skill 的全部提示词提取自项目 **Craft Anything** 的复刻工作流（`crates/ai/src/rebuild/`、`crates/ai/src/prompt.rs`、`crates/mcp/`、`crates/engine/src/tools/`、`crates/setup/src/skill.md`、`docs/`）。提取日期 2026-11-08。路径均为相对项目根，便于在别处回查。

## 1. 来源索引

| 层 | 文件 | 位置 / 常量 | 内容 | 收录于 |
|---|---|---|---|---|
| 纪律 | `crates/ai/src/prompt.rs` | `METHOD` (5-17)、`MEMORY` (20-26)、`BEGINNER`/`EXPERIENCED` (28-32)、`system_prompt()` (38-46)、`cli_preamble()` (50-55) | 系统提示词、工作记忆、语气档位、模式措辞 | `discipline.md` |
| 纪律 | `crates/mcp/src/prompts.rs` | `DISCIPLINE` (31-38)、`PROMPTS` (40-153)、`render()` (175-203) | 六条纪律、六个引导式提示词、渲染模板 | `discipline.md`、`contracts.md` |
| 纪律 | `crates/mcp/src/lib.rs` | `INSTRUCTIONS` (46-52)、`listed()` (386-390)、meta 工具 (813-831) | MCP 服务级指令、工具可见性、渐进工具组 | `discipline.md`、`tool-prompts.md` |
| 纪律 | `crates/setup/src/skill.md` | 全文（1-109） | 现成的「证据优先调查」Skill：连接规则、工作记忆、Method、同意与安全、错误码表 | `discipline.md` |
| 编排 | `crates/ai/src/rebuild/mod.rs` | 模块文档 (1-18)、`brief` 组装 (774-801)、`refine` 组装 (937-951)、`summary` (967-973)、`NOT_WORKING_CHECK_LIMIT` (1278)、`NO_PROJECT_PROMPT` (1287)、`API_BUILDER` (1345)、`CHECK_BEFORE_REPLYING` (1348) | 四阶段定义、循环控制、终止条件、API 执行者工具映射 | `prompts-check-refine.md`、`prompts-build.md` |
| 首轮简报 | `crates/ai/src/rebuild/brief.rs` | `RULES` (7-15)、`steps()` (21-35)、`WEBSITE_FROM_OUTSIDE` (62)、`PROGRAM_FROM_OUTSIDE` (66)、`brief()` (68-114)、`start_files_note()` (118-134)、`assets_note()` (139-158)、`DECOMPILE_FIRST` (161)、`decompile_note()` (165-180)、`extract_brief()` (188-200)、`ORIGINALS_READY` (203)、`PRIVATE_USE_NOTE` (207)、`game_steps()` (214-256)、`pictures_note()` (260-274)、`pictures_reminder()` (277-285)、`added_prompt()` (287-292)、`continue_prompt()` (296-301)、`resumed_elsewhere()` (307-323)、`just_asked()` (341-351)、`follow_up()` (353-358) | 首轮简报全部段落、游戏/资产/反编译分支、续跑与追问 | `prompts-build.md`、`prompts-check-refine.md` |
| Study | `crates/ai/src/rebuild/study.rs` | `study()` (13-128)、`how_it_works()` (135-152)、`study_from_outside()` (157-170)、`picture_window()` (174-190)、`describe()` (290-298) | 采集顺序、证据目录说明、证据包装格式 | `prompts-study.md` |
| Check | `crates/ai/src/rebuild/self_check.rs` | `summary()` (61-71)、`report()` (76-114)、`answer_self_checks()` (118-140) | 检查报告与摘要模板 | `prompts-check-refine.md` |
| 控制面工具 | `crates/ai/src/rebuild/tools.rs` | `specs()` (13-116)、`max_rounds` (360) | 9 个 `rebuild_*` 的 title/summary/beginner、轮次默认值 | `tool-prompts.md` |
| 只读问答 | `crates/ai/src/rebuild/question.rs` | `PREAMBLE` (35)、`prompt()` (123-126)、`context()` (219-239) | 短生命副本的只读约束与上下文模板 | `prompts-check-refine.md` |
| 目标类型 | `crates/ai/src/rebuild/kinds/mod.rs` | `TargetKind` trait (79-124)、`refine()` (118-123)、`run_checks()` (176-201) | 目标类型契约、默认精修指令 | `prompts-check-refine.md` |
| 目标类型 | `crates/ai/src/rebuild/kinds/website.rs` | `goal()` (185-187)、`brief_sections()` (193-210)、`refine()` (217-228) | 网站目标、多页面/打开态、相似度精修 | `prompts-build.md`、`prompts-check-refine.md` |
| 目标类型 | `crates/ai/src/rebuild/kinds/{app,game,own,plugin_check}.rs` | `RUNS_BRIEF` (app.rs:9)、`ASSETS_NOW_OFF` (game.rs:8)、`goal()`、`credits()` | 桌面/游戏/自有项目/插件检查 | `prompts-build.md`、`contracts.md` |
| 验收契约 | `crates/ai/src/rebuild/kinds/{mechanics,self_test,screenshot,runs}.rs` | `MECHANICS_CONTRACT` (mechanics.rs:12)、`SELF_TEST_CONTRACT` (self_test.rs:13)、`CORE_LOOP` (16)、`SCREENSHOT_CONTRACT` (screenshot.rs:16)、`contract()` (19-28)、`CASES` (runs.rs) | 四类可机读契约 | `contracts.md` |
| 技术栈 | `crates/ai/src/rebuild/stacks.toml` | 全部 `[[stack]]` / `[[library]]` 的 `steps` | 13 条构建指令 | `stacks.md` |
| 技术栈 | `crates/ai/src/rebuild/stacks.rs` | `steps()` (141-154)、`include_str!` (72) | 占位符替换与通用兜底 | `stacks.md` |
| 工具描述 | `crates/engine/src/tools/*.rs` | `ToolSpec` 定义（`tool.rs`）、`investigation_tools.rs`、`session_tools.rs`、`native_workflows.rs`、`native_formats.rs`、`application_tools.rs`、`compare_tools.rs` | 工具 title/summary/beginner/effects | `tool-prompts.md` |
| 工具描述 | `docs/tools.md` | 全文（生成文件，勿手改） | 302 个工具 / 17 个工具组的权威 id 清单 | `tool-prompts.md` |
| 文档 | `AGENTS.md` | Golden rules 2/3/4（43-54）、`docs/claude-code.md`、`README.md` (82,95-102,171-182,255-257)、`docs/ROADMAP.md` (54)、`docs/STATUS.md` (123-135) | 一切皆工具、证据即结果、Clean room/Deep、CLI 注入参数、轮次上限、对外四步说明 | 本 Skill 全局 |

## 2. 变量字典

所有 `{…}` 都是原文里的动态拼接点。通用模板统一用 `{{…}}`，映射如下。

| 变量 | 含义 | 出现位置 | 示例 |
|---|---|---|---|
| `REQUEST` | 用户原始请求 | 首轮简报 | `用 Next.js 把这个页面做出来` |
| `TARGET` / `URL` / `PATH` | 复刻目标描述 / 网址 / 本地路径 | 首轮简报 | `the website https://…` |
| `WORKSPACE` | 工作区根目录 | 首轮简报 | 项目目录 |
| `CAPTURE` | 证据说明段落（四分支之一） | 首轮简报 | 见 `prompts-build.md` 第 5 节 |
| `GOAL` | 目标类型决定的验收方向 | 首轮简报 | `a website that looks and behaves like the original` |
| `STACK_STEPS` | 技术栈 + UI 库指令 | 首轮简报 | 见 `stacks.md` |
| `STEPS` / `FOLLOW_UP` | 五步工作法 / 按模式选的追问范围 | 首轮简报 | 见 `prompts-build.md` 第 4 节 |
| `RULES` | 硬约束整块 | 首轮简报末尾 | 见 `prompts-build.md` 第 3 节 |
| `NAMES` / `OWN` / `CHECKED` | 已有文件名 / 系统自有文件名 / 自带构建检查的补充句 | `start_files_note` | `index.html, style.css` |
| `N` / `LIST` / `STATES` / `EXTRA` | 参考图数量 / 其它页面清单 / 打开态清单 / 用户补充 | 条件追加段落 | — |
| `MODEL_COST` / `PICTURE_COST` / `BUDGET` / `SPENT` / `FOLDER` | 资产单价 / 预算 / 已用 / 引擎资产目录 | `assets_note` | `$10.00` |
| `WHAT` / `MAKE` / `LEAD` / `BRING` | 反编译要找什么 / 自制或生成资产 / 1:1 分支措辞 | 游戏与反编译段落 | — |
| `PRIVATE_USE_NOTE` | 私有使用声明（逐字，检查会核对） | README 首行 | 见 `prompts-build.md` 8.4 |
| `FOLDER` | 原版文件夹（抽取器） | `extract_brief` | — |
| `ROUND` / `NOTE` / `S` / `BAR` / `CHECK` | 轮次 / 检查说明 / 相似度 / 门槛 / 检查名 | 报告与精修 | `3` / `82` / `90` |
| `CLAIM` / `DETAIL` / `UNKNOWN` / `QUESTION` / `ID` / `HELD` / `GOOD` | 断言、缺口、开放问题、编号、命中数、达标数 | 检查报告 | — |
| `MESSAGE` / `BUILD` / `BRIEF` / `HANDOFF` / `LAST_REPLY` / `TALK` | 追问 / 构建措辞 / 原简报 / 交接 / 上次回复 / 只读问答记录 | 续跑与追问 | `build it again` |
| `NAME` / `ROOT` / `APP` / `STATUS` / `STACK` / `FILE` / `AGO` | 只读问答的上下文字段 | `question.rs::context` | — |
| `PLACE` / `WHY` | 构建区位置 / 失败原因 | 终止错误文本 | — |
| `ENTRIES` / `CASES` / `SOURCED` / `ESTIMATED` | 支持的入口点 / 输入集合 / 有来源数 / 估算数 | 契约与 note | — |
| `EDITOR` / `PLAYER` / `EXT` | 引擎编辑器路径 / Unity 构建目标 / 扩展名 | 技术栈配方 | `StandaloneWindows64` / `exe` |
| `TITLE` / `OBJECTIVE` / `STEP` / `VALUE` / `DISCIPLINE` | 引导式提示词渲染字段 | `prompts.rs::render` | — |
| `MODE` / `BUILD_DIR` / `EVIDENCE_BLOCK` / `CONTRACTS` / `PICTURES` / `START_FILES` / `GAME_MECHANICS` / `MODE_RULES` | **本 Skill 通用模板新增**：模式、构建区、证据说明、契约、参考图、已有文件、游戏机制、模式限制 | `assets/*.md` | — |

## 3. 工具名解耦映射

原文里大量具体工具名属于提取源的实现细节。把提示词迁移到别的项目时按此表替换；**只换名字、不换语义**。

| 原文 | 通用概念 | 替换建议 |
|---|---|---|
| `core_unknown_add` / `core_unknown_edit` / `core_unknowns_show` | 登记 / 关闭 / 查看开放问题 | 项目的问题清单与状态字段 |
| `core_person_ask` | 向用户提问（2–4 个短选项） | 提问工具或人工确认环节 |
| `core_investigation_show` / `core_brief_write` / `core_plan_update` / `core_hypothesis_*` / `core_finding_*` / `core_handoff_write` | 工作记忆五件套 | 一组持久化文件或工具；没有就退化为「在文件里维护」 |
| `TodoWrite` / `build_checklist` | 3–7 步用户可见清单 | 任务清单工具 |
| `rebuild_check` | 与正式检查同口径的自检 | 自检入口；不计轮次 |
| `Read tool` | 查看图片或文件 | 图片查看 / 文件读取能力 |
| `core_evidence_browse` | 大结果分页读取 | 分页接口（`offset`/`limit`） |
| `capture/`、`capture.json`、`screenshot-<width>.png`、`reference/` | 只读证据目录 | 自定义证据目录，但保持「只读」语义 |
| `app/` | 唯一可写的构建区 | `BUILD_DIR`；用户已有项目时为工作区根 |
| `checks/round-N/` | 每轮检查产物 | 自定义轮次目录 |
| `ev_…` | 证据编号 | 自定义编号体系，但必须可回查 |
| `list_toolsets` / `enable_toolset` | 按需启用工具组 | 工具注册表 / 渐进式披露机制 |
| `app_ledger_build` / `app_coverage_evaluate` / `app_rebuild_claim_check` | 复刻账本 / 覆盖率 / 声明校验 | 期望清单 + proven 判定 + 逐条核对 |
| `native_*` / `js_*` / `game_*` / `web_*` | 按目标类型分组的分析工具 | 自己项目里的对应领域工具 |
| `assets_model_generate` / `assets_image_generate` / fal.ai | 可选资产生成服务 | 可整段删除；保留「原创、不复制品牌」约束 |
| Ghidra | 可选反编译器 | 没有时按 `decompile_note` 的无引擎分支：用内置结果并交叉验证 |

## 4. 模式分支矩阵

| 维度 | 只看外部（Clean room） | 允许深入（Deep） |
|---|---|---|
| 采集 | 只从外部观察：截图、可见文本、运行窗口 | 页面 HTML、脚本组成、请求、文件、函数 |
| 追问范围 | 再拍一个宽度 / 打开态，或问用户 | 还可按名字查字符串、函数、游戏设置与文件 |
| 证据说明 | `WEBSITE_FROM_OUTSIDE` / `PROGRAM_FROM_OUTSIDE` | 深入模式的两段 capture 说明 |
| 反编译 | 禁止 | 必需；先查证再估算，只把数值与规则写成自己的代码 |
| 原版资产 | 一律不取；自制中性占位 | 用户拥有权利时可 1:1 使用可读文件；加密内容仍不碰 |
| 工具可见性 | 只列出外部可观察的工具，其余直接拒绝 | 全部可用，证据里记录产出引擎 |
| 提示词措辞 | 明确写「不得打开/读取/列出/抓取/运行/反编译」 | 明确写「先学它怎么工作，不只学它长什么样」 |

## 5. 维护与合规

- 逐字原文用代码块保存，便于 diff；改写只在「通用版」段落进行，不改动原文块。
- 新增技术栈配方后同步更新 `stacks.md` 与本节索引；新增占位符后同步更新变量字典与 `assets/brief-template.md`。
- 本 Skill 只收录提取源自有的提示词，不引用其它产品、不写入密钥、令牌、真实本地绝对路径或个人数据。若要把 Skill 复制进公开仓库，先检查所有示例值是否已替换为明显的假值。
- 原文为英文、经过实际运行打磨；通用模板为中文、与工具解耦。需要对接原工具链时优先使用 `references/` 里的英文原文，不要拿中文模板反向翻译。
