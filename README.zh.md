# 证据驱动的软件复刻 · evidence-driven-rebuild

> 把「照着原版做一个」从一次性猜测，变成可测量、可收敛的工程循环。

[English README](README.md) · [贡献指南](CONTRIBUTING.md) · [来源与署名](ATTRIBUTION.md)

一个 **Agent Skill**（`SKILL.md` 格式），从 [Craft Anything](https://github.com/storytold/craft-anything)
的复刻工作流中提取、核对并抽象而来，与具体工具链解耦。适用于复刻网站、命令行程序、桌面应用和游戏，
也用于编写 builder 提示词、设计验收契约、搭建 study / build / check / refine 循环。

> **独立作品，与上游无隶属关系。** `references/` 中的英文提示词为上游逐字引用，
> 上游采用 `MIT OR Apache-2.0`；逐文件出处见 [ATTRIBUTION.md](ATTRIBUTION.md)。

---

## 1. 解决什么问题

让 Agent「照着这个网站做一个」，通常会得到一次看起来合理的尝试，然后直接宣布完成：没有测量、没有记录
到底观察到了什么、也没有收敛路径。本 Skill 用四个阶段替换这种一次性行为，核心是**用同一套测量方式**
同时作用于原版与构建产物：

| 阶段 | 目标 | 产物 | 退出条件 |
|---|---|---|---|
| **1 Study 研究** | 只记录可观察事实 | `capture/`：概览、设计量值、各宽度截图、运行记录、开放问题 | 概览先行、大结果分页、每个结论有编号证据 |
| **2 Build 构建** | 依证据写出新实现 | `app/` 里的项目文件、构建成功的真实输出 | 构建命令确实成功且输出可见 |
| **3 Check 校验** | 用同一测量方式对比 | 检查报告：通过项、相似度、差异、**无法比较项**、开放问题 | 每项有结果，未比较项被显式列出 |
| **4 Refine 精修** | 把差异变成下一轮指令 | 精修指令 + 新一轮构建 | 达到阈值 / 轮次上限 / 连续失败触顶 |

阶段 2–4 循环。默认「首轮 + 最多 6 次精修」（CLI 场景 2 次）；连续 10 次检查仍不可用、或连续两轮
没有文件变化，即停止并要求用户给出具体修正，**不无限重试**。

## 2. 五条硬约束

1. **先证据，后结论** — 每个事实性结论指回一条编号证据；观察到的 / 推导出的 / 猜测的分开写。
2. **缺失不是证据** — 「没搜到」只说明搜索范围；写明查了什么、覆盖到哪，推测值标注「估算」。
3. **模式决定边界** — Clean room 不读原版内部；Deep 允许深入，但必须先查证再估算，且只把数值和规则写成自己的代码，绝不粘贴原版代码。
4. **产物只写在构建区** — `capture/` 只读；原版 logo、图片、长段文案、品牌名不得照搬，除非用户确认拥有权利。
5. **校验必须可机读** — 每条检查项有明确通过条件与测量结果；「看起来差不多」不算通过，静态分析不能证明代码运行过。

## 3. 仓库结构

```text
SKILL.md                        入口：四阶段工作流、决策表、五条硬约束、资源索引
references/
  discipline.md                 调查纪律、工作记忆循环、Clean room / Deep 措辞（含逐字原文）
  prompts-study.md              Study：采集动作、证据目录说明、证据包装格式
  prompts-build.md              Build：首轮简报主模板、RULES、五步工作法、各分支段落
  prompts-check-refine.md       Check / Refine：检查报告、精修指令、续跑追问、终止条件
  contracts.md                  四类可机读验收契约 + 复刻声明校验
  stacks.md                     13 条技术栈与 UI 库构建指令（逐字配方）
  tool-prompts.md               工具描述文本写法、9 个控制面工具描述
  source-map.md                 来源索引、变量字典、工具名解耦映射、模式矩阵
assets/
  brief-template.md             通用版首轮简报（可直接填空）
  review-template.md            通用版检查报告 + 精修指令（可直接填空）
LICENSE-MIT / LICENSE-APACHE   双许可：MIT OR Apache-2.0
NOTICE / ATTRIBUTION.md        上游署名、派生说明、逐文件来源清单
```

**两层刻意分离**：`references/` 代码块里是逐字英文原文（附上游路径与行号），`SKILL.md` 与 `assets/`
是中文原创方法论和工具解耦模板。需要对接原工具链时直接用英文原文，**不要拿中文模板反向翻译**。

## 4. 安装

需要支持 `SKILL.md` 格式 Skill 的 Agent 运行时（WorkBuddy、Claude Code 等）。

```bash
# WorkBuddy：用户级，所有项目可用
git clone https://github.com/dreamsxin/evidence-driven-rebuild.git \
  ~/.workbuddy/skills/evidence-driven-rebuild

# Claude Code：项目级示例
git clone https://github.com/dreamsxin/evidence-driven-rebuild.git \
  .claude/skills/evidence-driven-rebuild
```

运行时只需要 `SKILL.md`、`references/`、`assets/`，其余均为文档与许可文件。GitHub Releases 也可能
附带预打包 `.zip`，下载后解压进 skills 目录即可。安装后重启会话或刷新技能列表。

触发示例：「照着这个网站复刻一个」「让新版本和原版一致」「把这些复刻提示词整理成 builder brief」
「设计一套验收契约 / check–refine 循环」。

## 5. 使用步骤

1. 判定**目标类型**（网站 / 命令行 / 桌面应用 / 游戏）、**模式**（Clean room / Deep）、请求里是否点名技术栈；填写 `references/source-map.md` 的变量字典。
2. 任务开始读 `references/discipline.md`；其余 references 只在对应阶段需要时读取，避免一次性灌满上下文。
3. 复制 `assets/brief-template.md` 写首轮简报。**技术栈配方（`stacks.md`）与验收契约（`contracts.md`）必须在首轮简报里写清**，否则检查阶段无法自动判定。
4. 每轮结束用 `assets/review-template.md` 写检查报告：最大的可见差异优先修；未通过项逐条回指测量值，而不是笼统说「再改改」。
5. 收敛后用 3–5 句说明「做了什么 / 与原版仍有哪些差异或未知 / 下一步可以问什么」，不贴代码与路径。

`source-map.md` 第 3 节把 `core_unknown_add`、`rebuild_check`、`capture/`、`app/` 等上游实现细节
映射为通用概念，迁移到别的项目时**只换名字、不换语义**。

## 6. 值得保留的设计意图

- **采集宽度必须等于检查宽度**：从未采集过的宽度上做测量，只会制造虚假信心。
- **未比较 ≠ 通过**：无法比较的项目必须在报告里显式列出。
- **没有记忆工具就整段删掉 MEMORY**，不要留一条指向不存在能力的死指令。
- **短生命只读副本视野有限，不代表主流程断连**。
- 不启动常驻进程（dev server / watcher），由检查环节统一托管与回收。
- 不重复已经失败或已回答过的相同调用；只在真正改变结果的选择上提问。

## 7. 许可与署名

本仓库整体采用 **MIT OR Apache-2.0**（任选其一），见 `LICENSE-MIT`、`LICENSE-APACHE`、`NOTICE`。
SPDX：`MIT OR Apache-2.0`。

`references/` 中的逐字提示词 © 2026 ArtCraft Team and the Craft Anything contributors，
来源为 `storytold/craft-anything` 的 `56aa3197`（2026-11-08），同样采用双许可。本仓库对其做了
重排、中文注解、参数化与工具名解耦；逐文件明细见 [ATTRIBUTION.md](ATTRIBUTION.md)，
行级索引见 [references/source-map.md](references/source-map.md)。

ArtCraft 名称、字标与 logo **不属于开源范围**。本仓库不包含上游的品牌素材、字体、图标或应用代码，
仅在署名与说明中以指称方式提及「Craft Anything」，不主张任何隶属、背书或赞助关系。

## 8. 贡献

见 [CONTRIBUTING.md](CONTRIBUTING.md)。要点：逐字原文块保持不动，通用化只写在中文层；
新增技术栈配方或占位符时，同步更新 `stacks.md` / `source-map.md` 的索引与变量字典。
