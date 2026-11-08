# Attribution / 来源与署名

本仓库是一份**独立整理**的 Agent Skill，其中 `references/` 收录的英文提示词段落为上游项目
[Craft Anything](https://github.com/storytold/craft-anything) 的**逐字引用（verbatim）**。
为符合上游许可证要求，此处逐文件列明来源、引用性质与改动情况。

## 上游信息

| 项目 | 值 |
|---|---|
| 上游仓库 | https://github.com/storytold/craft-anything |
| 提取时版本 | commit `56aa31976b4ff0950e6b34ba020b0a568d2462fa`，2026-11-08 |
| 上游许可证 | `MIT OR Apache-2.0`（见其 `Cargo.toml:8`、`LICENSE-MIT`、`LICENSE-APACHE`、`NOTICE`） |
| 上游版权人 | Copyright (c) 2026 ArtCraft Team and the Craft Anything contributors |
| 本仓库许可证 | 同为 `MIT OR Apache-2.0`（`LICENSE-MIT`、`LICENSE-APACHE`、`NOTICE`） |
| 逐文件定位索引 | [`references/source-map.md`](references/source-map.md) 第 1 节（含常量名与行号） |

本仓库已按 MIT 与 Apache-2.0 的要求，随附许可全文、保留全部版权声明，并在 `NOTICE`
中说明派生关系与所做的修改（Apache-2.0 §4(b)、§4(d)）。

## 逐文件来源

路径均相对上游仓库根目录；行号为提取时版本，可能随上游更新而漂移。

| 本仓库文件 | 内容性质 | 上游来源 |
|---|---|---|
| `references/discipline.md` | 英文提示词**逐字引用** + 原创中文注解 | `crates/ai/src/prompt.rs`（`METHOD`、`MEMORY`、`BEGINNER`/`EXPERIENCED`、`system_prompt`、`cli_preamble`）；`crates/mcp/src/prompts.rs`（`DISCIPLINE`、`PROMPTS`、`render`）；`crates/mcp/src/lib.rs`（`INSTRUCTIONS`）；`crates/setup/src/skill.md` |
| `references/prompts-study.md` | 英文提示词**逐字引用** + 原创中文注解 | `crates/ai/src/rebuild/study.rs`（`study`、`how_it_works`、`study_from_outside`、`picture_window`、`describe`） |
| `references/prompts-build.md` | 英文提示词**逐字引用** + 原创中文注解 | `crates/ai/src/rebuild/brief.rs`（`RULES`、`steps`、`brief`、`WEBSITE_FROM_OUTSIDE`、`PROGRAM_FROM_OUTSIDE`、各分支与续跑段落）；`crates/ai/src/rebuild/kinds/website.rs`；`kinds/{app,game,own,plugin_check}.rs`；`crates/ai/src/rebuild/mod.rs` |
| `references/prompts-check-refine.md` | 英文提示词**逐字引用** + 原创中文注解 | `crates/ai/src/rebuild/self_check.rs`（`summary`、`report`、`answer_self_checks`）；`rebuild/question.rs`；`rebuild/kinds/mod.rs`；`kinds/website.rs`；`rebuild/mod.rs`（`NO_PROJECT_PROMPT`、终止条件）；`brief.rs`（追问与续跑） |
| `references/contracts.md` | 契约文本**逐字引用** + 原创中文说明 | `crates/ai/src/rebuild/kinds/{mechanics,self_test,screenshot,runs}.rs`；`crates/mcp/src/prompts.rs`；`crates/engine/src/tools/compare_tools.rs` |
| `references/stacks.md` | 来自配置文件的构建指令**逐字引用** | `crates/ai/src/rebuild/stacks.toml`（全部 `[[stack]]` / `[[library]]` 的 `steps`）；`crates/ai/src/rebuild/stacks.rs`（`steps`、`include_str!`） |
| `references/tool-prompts.md` | 工具描述文本**逐字引用** + 原创中文写法说明 | `crates/ai/src/rebuild/tools.rs`（`specs`、`max_rounds`）；`crates/engine/src/tools/*.rs`（`ToolSpec` 的 title / summary / beginner）；`crates/mcp/src/lib.rs`；`docs/tools.md` |
| `references/source-map.md` | **原创**：来源索引、变量字典、工具名解耦映射 | 仅引用上游文件路径与常量名作为定位信息；思路参考 `AGENTS.md`、`docs/ROADMAP.md`、`docs/STATUS.md`、`README.md` |
| `SKILL.md` | **原创**：中文四阶段方法论、决策表、执行步骤、资源索引 | 方法论提炼自上游 `crates/ai/src/rebuild/` 的四阶段工作流；不含整段上游原文 |
| `assets/brief-template.md`、`assets/review-template.md` | **原创**：中文通用模板，工具名已解耦 | 结构与术语参考上游首轮简报与检查报告；所有上游工具名已替换为 `{{PLACEHOLDER}}` 或通用概念 |
| `README.md`、`README.zh.md`、`CONTRIBUTING.md` | **原创**文档 | — |

## 做了哪些修改

按 Apache-2.0 §4(b) 说明：上游材料在本仓库中经过以下改动，并非原样再分发。

1. **重排**：把散落在多个 crate、常量与配置文件里的提示词，按 `Study → Build → Check → Refine` 四阶段重新组织。
2. **注解与翻译**：为每段英文原文补充中文说明、适用范围与设计意图；`SKILL.md` 与 `assets/` 为中文原创撰写。
3. **参数化**：把原文中的动态拼接点整理为 `{{PLACEHOLDER}}` 变量字典，便于复用。
4. **工具名解耦**：将 `core_*` / `rebuild_*` / `capture/` / `app/` 等上游实现细节映射为通用概念（见 `source-map.md` 第 3 节），使模板不绑定上游工具链。
5. **原文与改写分离**：逐字原文始终放在代码块中，改写只出现在中文段落与 `assets/` 模板里，便于 diff 与回溯。

## 未包含的内容

本仓库**不包含**上游的 Rust 源码、可执行文件、字体、图标、应用图标、品牌素材或
`docs/brand/` 下的任何内容。上游 `NOTICE` 与 `ATTRIBUTION.md` 中列出的第三方字体与图标
许可（OFL、ISC、Ubuntu Font Licence 等）与本仓库无关，因为它们未被复制。

上游 `ATTRIBUTION.md` 明确说明：ArtCraft 名称、字标与 logo 是 ArtCraft Team 的商标，
**不属于开源范围**。本仓库仅在描述与署名时以指称方式提及 "Craft Anything"，
不将其用作项目名称、标识或背书。

## 无隶属声明

本项目为独立作品，与 ArtCraft Team 及 Craft Anything 项目**无隶属、未获背书、非其官方维护**。
若你是上游权利人并希望对署名方式作出调整，请通过本仓库 Issue 联系。
