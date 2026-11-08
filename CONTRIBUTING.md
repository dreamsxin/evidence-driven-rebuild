# Contributing / 贡献指南

Thanks for helping improve this Skill. It is a documentation-only repository: no build step, no
runtime dependencies, no CI required. / 本仓库只有文档，无构建步骤、无运行时依赖、无需 CI。

## Ground rules / 基本原则

1. **Keep verbatim blocks intact.** English prompt text inside code blocks in `references/` is
   quoted from upstream `storytold/craft-anything`. Do not rephrase, "improve" or translate it.
   If you find an error, open an issue with the upstream file path and line number instead.
   / `references/` 代码块里的英文原文是上游逐字引用，不要改写、润色或翻译；发现疑点请附上游路径与行号提 issue。
2. **Generalise only in the Chinese layer.** Reusable methodology goes into `SKILL.md`,
   annotations go beside the verbatim blocks, and tool-agnostic templates go into `assets/`.
   / 通用化只写在中文层：`SKILL.md` 放方法论，原文块旁放中文注解，`assets/` 放解耦模板。
3. **Names change, semantics do not.** When retargeting a template, replace tool names per
   `references/source-map.md` §3, but keep the evidence / check / refine semantics.
   / 迁移模板时按 `source-map.md` 第 3 节换名字，不换语义。
4. **No real data.** No absolute local paths, credentials, tokens, personal data, customer URLs,
   or upstream brand assets. Example values must be obviously fake.
   / 不写入真实本地绝对路径、密钥、令牌、个人数据、客户网址，也不引入上游品牌素材；示例值必须是明显的假值。

## Checklists / 提交前检查

When adding a tech-stack recipe / 新增技术栈配方：

- [ ] Add it to `references/stacks.md` with the upstream source path.
- [ ] Update the source index in `references/source-map.md` §1.
- [ ] Add any new `{{PLACEHOLDER}}` to the variable dictionary (§2) **and** `assets/brief-template.md`.

When adding a contract / 新增验收契约：

- [ ] Keep an explicit, machine-readable pass condition and a measured result field.
- [ ] State what happens when the item **cannot** be compared (it must be listed, never silently passed).

When attributing / 涉及来源：

- [ ] Update `ATTRIBUTION.md` and `NOTICE` if the set of quoted upstream files changes.
- [ ] Keep `LICENSE-MIT` and `LICENSE-APACHE` unchanged.

## Pull requests / PR 说明

- Describe what changed and why; quote the upstream path/line when touching `references/`.
- Keep one logical change per PR; avoid reformatting whole files.
- By contributing you agree that your contribution is licensed under **MIT OR Apache-2.0**,
  consistent with the rest of the repository. / 提交即表示同意贡献内容同样采用 `MIT OR Apache-2.0`。

## Scope / 边界

This repository collects **prompt text and methodology only**. Feature requests for the upstream
application, its tools, or its brand belong in the upstream project, not here.
/ 本仓库只收录提示词与方法论。上游应用的功能需求、工具或品牌相关问题请提到上游项目。
