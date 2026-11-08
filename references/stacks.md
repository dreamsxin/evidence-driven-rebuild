# 技术栈构建指令（配置承载的提示词）

来源 `crates/ai/src/rebuild/stacks.toml`（以 `include_str!` 加载，`stacks.rs:72`）。这些 `steps` 字段**本身就是注入给执行者的提示词**，只是放在配置文件里。逐字保留如下（已还原 TOML 字符串转义）。

---

## 1. 使用规则

- `{STACK_STEPS}` 由 `brief.rs:45-57` 的 `stack_and_library_steps()` 生成：用户已有项目 → `keep to what the project already uses.`（外加编辑器路径）；否则取所选栈的 `steps`，再把请求点名的每个 UI 库的 `steps` 依次以空格追加。
- `asked_by` 是请求里点名某个栈的短语，匹配时把请求转小写、标点转空格、两侧补空格后按**本文件顺序**取第一个命中的。
- `uses` / `needs` / `found_by` 用于从产物或已有项目反推实际用的栈，不参与提示词生成。
- 没有 `steps` 的栈（如 `Python`）走通用兜底（`stacks.rs:143`）：

```text
Use {STACK}. Create the project in app/ and make sure it builds or runs without errors.
```

- 三条占位符由 `stacks.rs:141-154` 填充：`{editor}` 引擎编辑器路径；`{player}` 与 `{ext}` 按操作系统取 Unity 的构建目标与扩展名（Windows → `StandaloneWindows64` / `exe`，macOS → `StandaloneOSX` / `app`，其它 → `StandaloneLinux64` / `x86_64`）。
- **硬约束**：请求点名了语言 / 框架 / 引擎 / 平台时，严格按请求来；这份清单只是「请求未指定部分」的默认值，不得为了绕过失败而换栈（见 `RULES` 第 1 条）。

## 2. 栈配方（逐字）

### Builder's choice

```text
Use what their request names; otherwise choose the stack yourself: the one that best fits the request and the original. If the request names none and the capture shows what the original is built with (the script and stylesheet addresses in capture.json and page.html, for example /_next/ for Next.js or /_nuxt/ for Nuxt), use the same. For a website, create the project in app/ with a "build" script in package.json, then run `npm install` and `npm run build` from app/ until it succeeds: the output must be static files in app/dist, app/out or app/build (index.html at its top and each other page at its own path) that work when served as static files.
```

### Electron

```text
Use Electron, with TypeScript or JavaScript (a bundler such as Vite is fine). Create the project in app/: package.json with electron as a devDependency, "main" set to the main-process file and a "build" script that compiles or bundles everything and exits without opening a window (for plain JavaScript, `node --check` on the main file is enough). Keep contextIsolation on and nodeIntegration off, and give the page what it needs through a preload script and IPC. For --screenshot, set backgroundThrottling to false, wait for the page to finish loading and about two seconds rather than counting requestAnimationFrame frames (a window that opens behind others may get none), then save win.webContents.capturePage(). Craft Anything starts the app with the Electron that `npm install` puts in node_modules, with app/ as its argument, so run `npm install` and `npm run build` from app/ until both succeed.
```

### Next.js + TypeScript

```text
Use Next.js (App Router) + React + TypeScript, exported as a static site. Create the project files directly in app/: package.json with next, react and react-dom and the script "build": "next build"; next.config.mjs with output: 'export', trailingSlash: true and images: { unoptimized: true }; tsconfig.json; src/app/layout.tsx and src/app/page.tsx, with one folder per other page under src/app/. Load the original's fonts with next/font. Use nothing a static export can't serve (API routes, server actions, middleware, dynamic routes without generateStaticParams). Then run `npm install` and `npm run build` from app/ until it succeeds; the output must be app/out and must work when served as static files.
```

### React + TypeScript (Vite)

```text
Use Vite + React + TypeScript. Create the project files directly in app/ (package.json with scripts "build": "tsc -b && vite build" or "vite build", vite.config.ts with base './', index.html, src/). Then run `npm install` and `npm run build` from app/ until it succeeds; the output must be app/dist and must work when served as static files.
```

### Plain HTML and CSS

```text
Use plain HTML, CSS and a little JavaScript only if the page needs it: no framework, no bundler, no build step. Create the project directly in app/: index.html at the top, one .css file (and .js only if needed) beside it, with each other page at its own path (about.html, contact.html). Link the stylesheet with a relative path. Then check that the pages open and look right when served as static files; there is nothing to build.
```

### Rust (console)

```text
Use Rust for a console program: create a cargo project in app/ itself (app/Cargo.toml and app/src/main.rs, not a folder inside app/), read its arguments (std::env::args, or clap if it has many options), print to standard output and standard error the way the original does, exit with the original's exit codes, and draw no window. Run `cargo build` until it succeeds.
```

### Rust + macroquad

```text
Use Rust with macroquad (a small game library). Create a cargo project in app/ itself (app/Cargo.toml and app/src/main.rs, not a folder inside app/), keep assets procedural (shapes, colours) unless the person provides some, and run `cargo build` until it succeeds.
```

### Rust + egui (eframe)

```text
Use Rust with eframe/egui for the window and UI. Create a cargo project in app/ itself (app/Cargo.toml, not a folder inside app/) and run `cargo build` until it succeeds.
```

### Unity

```text
Use Unity (the editor is {editor}). Create the project in app/: Assets/, Packages/manifest.json (built-in packages unless you need more) and ProjectSettings/ProjectVersion.txt with the editor's version. Write the game in C# under Assets/Scripts/. Scenes are hard to write by hand, so build them from code: an editor script Assets/Editor/CraftBuild.cs with a static `Build()` method that creates the scene(s) under Assets/Scenes/, adds them to the build settings and builds a {player} player to app/Build/Game.{ext} with BuildPipeline.BuildPlayer, calling EditorApplication.Exit(1) when the build fails. Build with `"{editor}" -batchmode -quit -nographics -projectPath app -executeMethod CraftBuild.Build -logFile app/Logs/build.log` and read that log when it fails. The first run imports the project and takes a few minutes.
```

### Unreal Engine

```text
Use Unreal Engine (the editor is {editor}) with C++: Blueprint and level files are binary and can't be written by hand. Create app/Game.uproject (its EngineAssociation set to the installed version, one C++ module) and Source/Game/ with the module, its Build.cs and the Target.cs files. Make levels from code (a GameMode that spawns the geometry, lights and actors at start, with a small empty default map) rather than editing .umap files. Craft Anything packages it with Unreal's BuildCookRun into app/Build after each round; the first package takes a long time, so compile your C++ with the engine's build tool first and fix errors there.
```

### Godot

```text
Use Godot (the editor is {editor}) with GDScript. Create the project in app/: project.godot (config_version=5, with run/main_scene set), scenes as .tscn text files and scripts as .gd files; Godot's files are plain text, so write scenes directly. Check it with `"{editor}" --headless --path app --import` and then `"{editor}" --headless --path app --quit-after 120`, and fix every SCRIPT ERROR or ERROR line.
```

### Python 等无 `steps` 的栈

```text
Use {STACK}. Create the project in app/ and make sure it builds or runs without errors.
```

## 3. UI 库配方（逐字，追加在栈指令之后）

### shadcn/ui

```text
Build the interface from shadcn/ui components, styled with Tailwind CSS: set it up in app/ with the shadcn CLI, non-interactively (`npx shadcn@latest init --yes --defaults` once package.json and Tailwind are in place), add each component you use with `npx shadcn@latest add --yes <names>` (button, card, navigation-menu, sheet for the phone menu, accordion, tabs…), and use them for every matching part of the page rather than hand-made ones. Then make them look like the original: set the theme's CSS variables (background, foreground, primary, muted, accent, border, ring, radius) to the original's colours and corner radii from capture.json, not shadcn's defaults.
```

适用于：`Next.js + TypeScript`、`React + TypeScript (Vite)`、`Electron`、`Builder's choice`。

### Tailwind CSS

```text
Style with Tailwind CSS (v4, through @tailwindcss/postcss for Next.js or @tailwindcss/vite for Vite), with the original's colours, fonts, text sizes, spacing and breakpoints from capture.json as theme tokens in an @theme block.
```

适用于同上四个栈。

## 4. 这些配方为什么有效（可复用写法）

每条 `steps` 都固定回答五个问题，新增配方时照填：

1. **用什么**：明确语言、框架、入口文件名。
2. **建在哪**：项目根直接落在构建区，不要再套一层目录（源码里反复强调 `not a folder inside app/`，这是最常见的失败点）。
3. **构建命令**：给出确切命令，并要求「跑到成功为止」。
4. **产物在哪、什么形状**：`dist` / `out` / `build`，静态可服务、入口在顶层。
5. **已知陷阱**：二进制场景要用代码生成、无头模式参数、日志位置、首次构建耗时长等。

另外两条值得照搬：
- 要求构建脚本「退出且不打开窗口」，从源头避免长驻进程卡住检查环节。
- 主题量值必须取自原版证据，而不是组件库默认主题——否则每一轮检查都会报同样的差异。

## 5. 新增一个配方的模板

```toml
[[stack]]
name = "My Stack"
asked_by = [" my stack ", " using mystack "]
uses = ["my-package"]          # 从依赖反推用过的栈
needs = ["other-package"]      # 必须同时存在才算
found_by = ["my-stack.config"] # 已有项目里标志性的文件
site = true                    # 产物是静态网站时置真
steps = "Use My Stack. Create the project files directly in {BUILD_DIR}: ... Then run `... install` and `... build` from {BUILD_DIR} until it succeeds; the output must be {BUILD_DIR}/dist and must work when served as static files."
```

改完后同步更新 `references/source-map.md` 的来源索引，并在 `assets/brief-template.md` 的变量字典里登记新占位符。
