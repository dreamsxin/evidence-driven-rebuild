# evidence-driven-rebuild

> An Agent Skill that turns "make it look and behave like the original" into a measurable, converging engineering loop — not a one-shot guess.

[中文说明 / Chinese README](README.zh.md) · [Contributing](CONTRIBUTING.md) · [Attribution](ATTRIBUTION.md)

**This is an independent, unaffiliated work.** The English prompt text in `references/` is quoted
verbatim from [Craft Anything](https://github.com/storytold/craft-anything) (MIT OR Apache-2.0).
See [ATTRIBUTION.md](ATTRIBUTION.md) for the per-file source list. This project is not endorsed by
or affiliated with the ArtCraft Team.

---

## The problem it solves

Ask an agent to "rebuild this site" and it usually produces one plausible-looking attempt, then
declares success. There is no measurement, no record of what was actually observed, and no way to
converge. This Skill replaces that with four phases that share the **same measurement method** on
both the original and the build:

| Phase | Goal | Output | Exit condition |
|---|---|---|---|
| **1 · Study** | Record only observable facts | `capture/`: overview, design values, screenshots at each width, run logs, open questions | Overview first, large results paginated, every conclusion has an evidence ID |
| **2 · Build** | Write a new implementation from evidence | Project files in `app/`, real successful build output | The build command actually succeeded and its output is visible |
| **3 · Check** | Compare original vs build with identical measurements | Check report: passes, similarity, diffs, **not-compared items**, open questions | Every item has a result; anything not compared is listed explicitly |
| **4 · Refine** | Turn differences into the next round's instructions | Refine brief + a new build | Threshold met / round cap / stalled |

Phases 2–4 loop. Defaults: first build + up to 6 refine rounds (2 for CLI targets); stop after
10 unusable checks or two rounds with no file changes — then ask the user for a concrete fix
instead of retrying forever.

## Five hard constraints

1. **Evidence before conclusions.** Every factual claim points back to a numbered evidence item; observed, inferred and guessed are kept apart.
2. **Absence is not evidence.** "Not found" only describes the search scope. Log open questions; label estimates as estimates.
3. **The mode sets the boundary.** Clean room forbids reading the original's internals; Deep mode permits it but requires verifying values before estimating and writing **your own** code — never pasting original code.
4. **Only the build area is writable.** `capture/` is read-only. Never copy original logos, images, long-form copy or brand names unless the user confirms they own the rights.
5. **Checks must be machine-readable.** Each item has an explicit pass condition and a measured result. "Looks about right" is not a pass. Static analysis cannot prove the code ran.

## Repository layout

```
SKILL.md                        Entry point: the four-phase workflow, decision table, hard constraints
references/
  discipline.md                 Investigation discipline, working memory, Clean room / Deep wording
  prompts-study.md              Study-phase prompts, capture layout, evidence packaging
  prompts-build.md              First-round brief, RULES, five-step method, all conditional branches
  prompts-check-refine.md       Check report, refine brief, resume / follow-up templates, stop rules
  contracts.md                  Four machine-readable acceptance contracts + claim verification
  stacks.md                     13 verbatim tech-stack / UI-library build recipes
  tool-prompts.md               How the tool descriptions are written; 9 control-plane tools
  source-map.md                 Source index, variable dictionary, tool-name decoupling map
assets/
  brief-template.md             Fill-in-the-blank first-round brief (tool-agnostic)
  review-template.md            Fill-in-the-blank check report + refine brief (tool-agnostic)
LICENSE-MIT, LICENSE-APACHE     Dual licence, MIT OR Apache-2.0
NOTICE, ATTRIBUTION.md          Upstream attribution and derivation notes
```

Two layers are deliberately kept apart: **verbatim English originals** live inside code blocks in
`references/` (with upstream file paths and line numbers), while **reusable Chinese methodology and
templates** live in `SKILL.md` and `assets/`. Use the English originals when wiring into the
original toolchain; do not back-translate the Chinese templates.

## Install

Requires an agent runtime that loads `SKILL.md`-style skills (WorkBuddy, Claude Code, or similar).

```bash
# WorkBuddy (user-level, available in every project)
git clone https://github.com/dreamsxin/evidence-driven-rebuild.git \
  ~/.workbuddy/skills/evidence-driven-rebuild

# Claude Code (project-level example)
git clone https://github.com/dreamsxin/evidence-driven-rebuild.git \
  .claude/skills/evidence-driven-rebuild
```

Only `SKILL.md`, `references/` and `assets/` are needed at runtime; the rest of the repository is
documentation and licensing. A pre-built `.zip` may also be attached to GitHub Releases — download
it and unpack it into your skills directory.

Restart or refresh the skill list after installing. The Skill triggers on requests such as
"rebuild this site", "make the new version match the original", "write a builder brief for a
rebuild", or "design an acceptance contract / check–refine loop".

## Use

1. Identify target type (website / CLI / desktop app / game), mode (Clean room or Deep), and whether the user named a tech stack. Fill in the variable dictionary in `references/source-map.md`.
2. Read `references/discipline.md` at the start; load the other references only in the phase that needs them, to avoid flooding context.
3. Copy `assets/brief-template.md` for the first build. State the tech-stack recipe (`stacks.md`) and the acceptance contracts (`contracts.md`) **in that first brief** — otherwise Check cannot decide automatically.
4. After each round, write a check report using `assets/review-template.md`. Fix the largest visible difference first; every failing item points to a measured value, not a vague "polish it".
5. Stop on threshold or cap, then summarise in 3–5 sentences: what was built, what still differs or is unknown, what to ask next.

`source-map.md` maps upstream names (`core_unknown_add`, `rebuild_check`, `capture/`, `app/`, …) to
generic concepts, so the templates can be retargeted by changing names only, not semantics.

## Design notes

- **Capture width must equal check width.** Measuring at widths you never captured produces false confidence.
- **Not compared ≠ passed.** Uncompared items are always listed explicitly.
- **If there is no memory tool, delete the MEMORY block entirely** rather than leaving a dead instruction.
- **A short-lived read-only copy with limited visibility does not mean the main workflow lost its connection.**
- No long-running processes (dev servers, watchers) are started by the agent; the check phase owns and recycles them.

## Attribution and licence

Licensed under **MIT OR Apache-2.0** at your option — see `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`.
SPDX: `MIT OR Apache-2.0`.

The verbatim prompt text in `references/` is © 2026 ArtCraft Team and the Craft Anything
contributors, sourced from `storytold/craft-anything` at commit `56aa3197` (2026-11-08), under the
same dual licence. It has been reordered, annotated, parameterised and decoupled from upstream tool
names; the full per-file breakdown is in [ATTRIBUTION.md](ATTRIBUTION.md) and the line-level index
is in [references/source-map.md](references/source-map.md).

The ArtCraft name, wordmark and logos are **not** open source. No brand assets, fonts, icons or
application code from upstream are included here. No affiliation, endorsement or sponsorship is
claimed.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: keep verbatim blocks intact, put generalisations
in the Chinese layer, and update the source index when adding recipes or placeholders.
