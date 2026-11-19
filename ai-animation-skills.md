# 用 AI 做动画视频：18 个免费 Skill
# 18 Free Skills for Making Animated Videos with AI

这是我学做动画视频的笔记。18 个免费的 Skill，全是 GitHub 上公开的。用法就一个：挑一个，把仓库链接丢给 Muse、Claude Code 或者 Codex，让它学、让它装，然后直接告诉它你要什么效果。
These are my notes on learning to make animated videos. 18 free skills, all public on GitHub. The way to use them is always the same: pick one, hand its repo link to Muse, Claude Code, or Codex, let it learn and install it, then just tell it what you want.

整条流程：指示 → 制作 → 动起来 → 修改 → 导出。下面按这个流程分组，每组里的每个 Skill 都单独写清楚了：是什么、用来干嘛、怎么装、怎么验证、怎么让 AI 帮你做。
The full flow: instruct → build → animate → revise → export. The skills below are grouped by that flow, and each one gets its own section: what it is, what it's for, how to install it, how to check it works, and how to get the AI to use it.

---

## 1. 做视频的骨架 / Building the Structure

骨架就是视频本身怎么做出来：写代码生成视频、搭场景、做数学动画。这组三个。
The structure is how the video itself gets made: generating video from code, building scenes, doing math animations. Three skills in this group.

### Remotion Skills

Remotion 的官方 Agent Skill，不是渲染库本身。它是一组最佳实践定义，告诉 AI 在 Remotion 项目里该怎么写代码、怎么渲染。Claude Code、Codex、Cursor、Kimi Code、OpenCode 都能用。
Remotion's official Agent Skills — not a rendering library itself, but a set of best-practice definitions that tell an AI how to write code and render in a Remotion project. Works with Claude Code, Codex, Cursor, Kimi Code, OpenCode.

适合什么场景：想让 AI 直接帮你做 Remotion 视频项目——新建项目、写画面、预览、渲染、加字幕、做地图动画都行。
Good for: getting the AI to build a Remotion video project for you — new project, writing scenes, previewing, rendering, captions, map animations.

**安装 / Install**

在 Remotion 项目里装（README 首选）：
Install inside a Remotion project (the README's first choice):
```bash
npx remotion skills add
```

项目里还没装 Remotion：
If the project doesn't have Remotion yet:
```bash
npx skills add remotion-dev/skills
```

或者用 GitHub CLI：
Or with the GitHub CLI:
```bash
gh skill install remotion-dev/skills
```

新建项目、一次装全部 Skill、不弹提示：
New project, install all skills at once, no prompt:
```bash
npx create-video@latest --skills
```

新建空白项目（再自己装 Skill 也行）：
New blank project (you can add the skills yourself after):
```bash
npx create-video@latest --yes --blank --no-tailwind my-video
cd my-video
```

前置要求：README 和官方文档里没写 Node 版本要求，没核实到。
Prerequisites: the README and docs don't state a Node version requirement — not verified.

**验证 / Verify**

没有 `--version` 这类验证命令。README 给的是直接用的例子，装完之后给 AI 下面的指令，能跑起来就是装好了。
There's no `--version` check. The README just shows usage examples — if the commands below run, it's installed.

我自己实测过：按这个流程建项目、写画面、渲染，5 秒 1080×1920 的片子能正常出片。第一次渲染会自动下载约 100 MB 的 Chromium，之后就不用了。
I tested this myself: following this flow — scaffold, write a scene, render — a 5-second 1080×1920 clip renders fine. The first render downloads about 100 MB of Chromium automatically; after that, no more downloads.

**让 AI 帮你做 / Using it with AI**

装好之后直接给 AI 下斜杠指令，README 里的原例子：
```
/remotion-create Make a promo video for a record store
/remotion-markup Create an animated title card using Inter.
/remotion-studio
/remotion-render
/remotion-maps Animate a route from Los Angeles to New York and make the camera follow it.
```
不知道用哪个，就用 `/remotion-best-practices`，README 说它包含所有其他 Skill。

**链接 / Links**
- https://github.com/remotion-dev/skills
- 文档 / Docs：https://www.remotion.dev/docs/ai/skills

### Motion Canvas

Motion Canvas 是用 TypeScript 写代码做动画的库，自带一个实时预览编辑器。专门做信息类、讲解类的矢量动画，还能跟旁白对上时间。
Motion Canvas is a TypeScript library for coding animations, with a live-preview editor built in. It's made for informational, explainer-style vector animation that can sync with narration.

适合什么场景：科普视频、讲解动画、演示动画——画面是画出来的，能精确控制每一帧。
Good for: science explainers, tutorial animations, demos — visuals drawn in code, frame-accurate.

**安装 / Install**

README 本身没给安装命令，官方 Quickstart 给的是脚手架（Node.js 16 或以上，先跑 `node -v` 检查）：
The README itself gives no install command. The official Quickstart uses scaffolding (Node.js 16 or above — check with `node -v` first):
```bash
npm init @motion-canvas@latest
```
脚手架会问项目名、TypeScript 还是纯 JS、导出器选图片序列和/或 FFmpeg 视频。然后进项目装依赖：
The scaffold asks for a project name, TypeScript or plain JS, and exporters (image sequence and/or FFmpeg video). Then go into the project and install dependencies:
```bash
cd <project-path>
npm install
```
启动编辑器：
Start the editor:
```bash
npm start
```
编辑器在 http://localhost:9000/ 打开。
The editor opens at http://localhost:9000/.

脚手架失败时的备用命令（Quickstart Troubleshooting 原样）：
Fallback if scaffolding fails (verbatim from the Quickstart Troubleshooting section):
```bash
npm exec @motion-canvas/create@latest
```

**验证 / Verify**

没有 `--version` 命令。Quickstart 的验证办法：`npm start` 之后，把 `src/scenes/example.tsx` 换成官方的红圆圈示例（`makeScene2D` + `Circle` + `yield* all(...)`），保存后在编辑器点播放，看到红圆圈左右移动、变色就是跑通了。
No `--version` check. The Quickstart way to verify: after `npm start`, replace `src/scenes/example.tsx` with the official red-circle example (`makeScene2D` + `Circle` + `yield* all(...)`), save, hit play in the editor — if the red circle moves side to side and changes color, it works.

**让 AI 帮你做 / Using it with AI**

README 和 Quickstart 里没写 AI Skill，也没给 AI 指令，这一步没核实到。通用办法：把仓库链接丢给 AI，再告诉它你要什么。
The README and Quickstart don't mention any AI skill or AI instructions — not verified. The general way still works: hand the repo link to the AI and describe what you want.
> "参考 https://github.com/motion-canvas/motion-canvas，用 Motion Canvas 做一个 5 秒的讲解动画：一个红圆圈从左移到右，中间变一次颜色。先按官方 Quickstart 建项目，跑起来给我看。"

**链接 / Links**
- https://github.com/motion-canvas/motion-canvas
- 文档 / Docs：https://motion-canvas.io/docs/quickstart

### Manim

Manim 是社区维护的 Python 动画引擎，专门做讲解型数学视频，用代码精确地做动画。3Blue1Brown 那种视频就是这个路子。
Manim is a community-maintained Python animation engine for explanatory math videos — precise, code-driven animation, the same approach as 3Blue1Brown's videos.

适合什么场景：数学、理科讲解动画，公式、图形、变换过程要一帧一帧精确控制的时候。
Good for: math and science explainers, when formulas, shapes, and transformations need frame-accurate control.

**安装 / Install**

最简单（pip，有 Python 环境就行）：
Simplest way (pip, if you already have a Python environment):
```bash
pip install manim
```

官方推荐的 uv 路线（macOS/Linux）：
The officially recommended uv route (macOS/Linux):
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv python install
uv init manimations
cd manimations
uv add manim
```
Windows 装 uv（PowerShell）：
Installing uv on Windows (PowerShell):
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
全局工具式安装（可选）：
Global tool-style install (optional):
```bash
uv tool install manim
```
开发版（可选）：
Development build (optional):
```bash
uv add git+https://github.com/ManimCommunity/manim.git@main
```

前置要求（官方安装页原样）：
- Python：用 `uv python install` 装，页内举例用 3.12。最低 Python 版本在读到的页面里没明确写，没核实到。
- LaTeX：只在要渲染公式时需要。Windows 推荐 MiKTeX，macOS 推荐 MacTeX，Linux 推荐 TeX Live（Debian：`sudo apt install texlive-full`）。
- macOS：先装 cairo 和 pkg-config：`brew install cairo pkg-config`。
- Linux：要 C 编译器、Python 开发头文件、pkg-config、Pango/Cairo 开发头文件。Debian：`sudo apt install build-essential python3-dev libcairo2-dev libpango1.0-dev`；Fedora：`sudo dnf install python3-devel pkg-config cairo-devel pango-devel`；Arch：`sudo pacman -Syu base-devel cairo pango`。
- Windows：`uv add manim` 报错缺 Microsoft Visual C++ 时，装 Microsoft C++ Build Tools（选 "Desktop development with C++" 工作负载）再重跑。
- Windows: if `uv add manim` fails for a missing Microsoft Visual C++, install Microsoft C++ Build Tools (pick the "Desktop development with C++" workload) and rerun.

In English, briefly: Python is installed via `uv python install` (the docs use 3.12 as an example; no minimum version was stated). LaTeX is only needed for rendering formulas. macOS needs `brew install cairo pkg-config`; Linux needs a C compiler, Python dev headers, pkg-config, and Pango/Cairo dev headers.

**验证 / Verify**

官方安装页原样：
```bash
uv run manim checkhealth
```
README 的最小示例：把 `SquareToCircle` 场景（`from manim import *`，方形变圆形）存成 `example.py`，跑：
The README's minimal example: save the `SquareToCircle` scene (`from manim import *`, a square turning into a circle) as `example.py`, then run:
```bash
manim -p -ql example.py SquareToCircle
```
`-p` 是预览自动打开播放器，`-ql` 是低画质快速渲染。

**让 AI 帮你做 / Using it with AI**

README 和官方安装页里没写 AI Skill，也没给 AI 指令，这一步没核实到。通用办法：
The README and install docs don't mention any AI skill or instructions — not verified. The general way:
> "参考 https://github.com/ManimCommunity/manim，用 Manim 写一个场景：一个正方形慢慢变成圆形，存成 example.py，用 `manim -p -ql example.py SquareToCircle` 渲染给我看。"

**链接 / Links**
- https://github.com/ManimCommunity/manim
- 文档 / Docs：https://docs.manim.community

---

## 2. 让画面动起来 / Making Things Move

这一组是补间和缓动：让一个值从 A 平滑变到 B，管时间线、管错峰、管平滑。五个。
This group is tweening and easing: moving a value smoothly from A to B, timelines, staggering, smoothing. Five skills.

### Motion

Motion 是开源动画库，给 JavaScript、React、Vue 用的，前身是 Framer Motion。现在从 `motion/react` 导入，不再是 `framer-motion`。API 简单，JS 加浏览器原生 API 的混合引擎，120fps、GPU 加速，手势、弹簧、布局过渡、滚动联动、时间线都内置。
Motion is an open-source animation library for JavaScript, React, and Vue — formerly Framer Motion. You now import from `motion/react`, not `framer-motion`. Simple API, a hybrid engine of JS plus native browser APIs, 120fps, GPU-accelerated, with gestures, springs, layout transitions, scroll linking, and timelines built in.

适合什么场景：网站 UI 动效、React/Vue 页面动画。
Good for: website UI animation, React/Vue page animation.

**安装 / Install**

React / JavaScript：
```bash
npm install motion
```
Vue：
```bash
npm install motion-v
```
README 里没写 Node / React / Vue 版本要求，没核实到。
The README states no Node / React / Vue version requirements — not verified.

**验证 / Verify**

没有 `--version` 命令。README 的最小示例就是验证：
No `--version` command. The README's minimal examples are the check:
React：
```js
import { motion } from "motion/react";
<motion.div animate={{ x: 100 }} />
```
JavaScript：
```js
import { animate } from "motion";
animate("#box", { x: 100 });
```
Vue：
```js
import { motion } from "motion-v";
<motion.div :animate={{ x: 100 }} />
```

**让 AI 帮你做 / Using it with AI**

README 有专门一节 "Using Motion with AI"，原样内容：
- 装 Agent Skill：跑 `npx motion-ai`，会装免费 MIT 协议的 `/motion` Skill，并给 Claude Code、Cursor、Amp、OpenCode、Gemini CLI、Copilot 配置 Motion 的 MCP 服务器。
- MCP 服务器：`https://mcp.motion.dev`，免费、不用账号，能搜 Motion 文档和示例。
- `llms.txt`：`motion.dev/llms.txt`，索引全部文档页。

具体指令例子 README 没给，没核实到。按 README 的做法：先跑 `npx motion-ai` 装好 Skill 和 MCP，再让 AI 按 Motion 文档写动画。
> "先跑 `npx motion-ai` 装好 Motion 的 Skill 和 MCP，然后参考 https://github.com/motiondivision/motion，用 Motion 写一个页面：一个方块点一下就平滑移到右边 100px，带弹簧效果。"

**链接 / Links**
- https://github.com/motiondivision/motion
- 文档 / Docs：https://motion.dev

### Anime.js

Anime.js 是快速、多用途、轻量的 JavaScript 动画库，API 简单但能做很多事。能动 CSS 属性、SVG、DOM 属性和 JavaScript 对象。
Anime.js is a fast, versatile, lightweight JavaScript animation library with a simple but powerful API. It animates CSS properties, SVG, DOM attributes, and JavaScript objects.

适合什么场景：网页小动效、SVG/DOM 动画、时间线动画。
Good for: small web animations, SVG/DOM animation, timeline animation.

**安装 / Install**

README 本身没有给安装命令，这一条没核实到。README 只给了 V4 的 ES module 用法：
```js
import { animate, stagger } from 'animejs';
animate('.square', {
  x: 320,
  rotate: { from: -180 },
  duration: 1250,
  delay: stagger(65, { from: 'center' }),
  ease: 'inOutQuint',
  loop: true,
  alternate: true
});
```
README 里唯一的 npm 命令是开发本仓库用的，不是装来用：先 `npm i` 装开发依赖，再 `npm run <script>`（比如 `dev` / `build` / `test:browser` / `test:node`）。

官方文档 https://animejs.com/documentation 打开后只抓到 5 行文字（JS 渲染页，没正文内容），没亲眼看到安装命令。第三方页面转述说官方安装页写了 `npm install animejs` 和 CDN 用法，但那是转述，按规矩不算已核实——要确认的话，在能正常打开 animejs.com/documentation 的浏览器里再看一眼。
The official docs page only returned 5 lines of text (a JS-rendered page with no readable body), so no install command was verified there. Third-party pages claim the official install page says `npm install animejs` plus CDN usage, but that's secondhand — not verified.

**验证 / Verify**

没核实到 `--version` 命令。最接近的就是上面 README 的 `animate('.square', …)` 最小示例，跑起来能动就是装好了。

**让 AI 帮你做 / Using it with AI**

README 和抓到的官方文档页里没写 AI Skill，也没给 AI 指令，没核实到。通用办法：
> "参考 https://github.com/juliangarnier/anime，用 Anime.js 写一个页面：四个方块从中间向两边依次错峰移开，带旋转，循环播放。安装命令 README 里没写，你先确认怎么装再写代码。"

**链接 / Links**
- https://github.com/juliangarnier/anime
- 文档 / Docs：https://animejs.com/documentation

### TweenJS

TweenJS 是 CreateJS 套件里的补间动画库，简单但够用。它是为配合 EaselJS 开发的，但不依赖它。能补间数值对象属性和 CSS 样式属性，链式写法、时间线控制都行。
TweenJS is the tweening library from the CreateJS suite — simple but capable. It was built to go with EaselJS but doesn't depend on it. It tweens numeric object properties and CSS style properties, with chaining and timeline control.

适合什么场景：Canvas 动画、链式补间、时间线控制。
Good for: canvas animation, chained tweens, timeline control.

**安装 / Install**

README 本身没有任何安装命令，这一条没核实到。README 只给了用法示例：
```js
var tween = createjs.Tween.get(myTarget)
    .to({x:300},400)
    .set({label:"hello!"})
    .wait(500).to({alpha:0,visible:false},1000)
    .call(onComplete);
```
官网 https://tweenjs.com/ 打开时超时，没核实到内容。第三方转述（unpkg / socket.dev / npm.io 上的包 README 镜像）写的是：
The official site https://tweenjs.com/ timed out when opened — nothing verified there. Third-party copies of the package README (on unpkg / socket.dev / npm.io) say:
```bash
npm install @createjs/tweenjs --save
```
CDN：
```html
<script src="https://code.createjs.com/2.0/tweenjs.min.js"></script>
```
注意两点：这个包 README 标注 2.0 是 BETA（`2.0.0-beta.4`，约 7 年前发布）；不要跟另一个不同的库 `@tweenjs/tween.js`（Tween.js）搞混。以上都是转述，没在 README 或官网亲眼核实到。

**验证 / Verify**

README 里没写前置要求、没写 `--version`、没写 AI Skill，全部没核实到。能跑通上面那段示例代码就是验证。

**让 AI 帮你做 / Using it with AI**

README 里没写，通用办法：
> "参考 https://github.com/CreateJS/TweenJS，用 TweenJS 写一个动画：一个方块 400ms 内移到 x=300，停 500ms，再 1000ms 内淡出，结束后调一个回调。安装命令 README 里没写，你先确认怎么装再写代码。"

**链接 / Links**
- https://github.com/CreateJS/TweenJS
- 官网：https://tweenjs.com/（本次打开超时，没核实到内容）

### Shifty

Shifty 是 TypeScript 补间动画引擎，自称“the fastest TypeScript animation engine on the web”。轻量，定位是给更高层工具封装的底层引擎：单个补间的播放控制、补间生命周期关键点的扩展钩子、Promise 支持（能用 `async`/`await`）。
Shifty is a TypeScript tweening engine — "the fastest TypeScript animation engine on the web", in its own words. It's lightweight and meant to be wrapped by higher-level tools: per-tween playback control, hooks into tween lifecycle points, Promise support (works with `async`/`await`).

适合什么场景：要以最小处理和内存开销做可定制动画的时候，或者自己封装动画工具的时候。它不是一个开箱即用的完整动画框架。
Good for: custom animation with minimal overhead, or building your own animation tools on top of it. It's not a full out-of-the-box animation framework.

**安装 / Install**

```bash
npm install --save shifty
```
README "Developing Shifty" 一节里的 `npm ci` / `npm run build` / `npm test` 是开发 Shifty 本身用的，不是用户安装步骤。
The `npm ci` / `npm run build` / `npm test` commands in the README's "Developing Shifty" section are for developing Shifty itself, not for installing it as a user.

环境要求：README 只写了支持 Evergreen 浏览器、Safari、Node 10 及以上，IE 由 v2 支持。除此之外没写其他前置要求。

**验证 / Verify**

官方 Getting Started 里的最简示例：
```js
import { tween } from 'shifty'
tween({
  from: { x: 0, y: 50 },
  to: { x: 10, y: -30 },
  duration: 1500,
  easing: 'easeOutQuad',
  render: state => console.log(state),
}).then(() => console.log('All done!'))
```
跑起来、控制台能打出变化的状态值就是装好了。
If it runs and the console prints the changing state values, it's installed.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/jeremyckahn/shifty，再加上面这段 Getting Started。
> "参考 https://github.com/jeremyckahn/shifty，用 Shifty 的 `tween()` 写一个 500ms 的补间：把 `#box` 从 x=0 平移到 x=200，同时 scale 从 1 到 1.5，用 `easeInOutQuad`，在 `render` 里用 transform 更新元素样式。只用 README 和官方文档里的 API，不要自己编。"

**链接 / Links**
- https://github.com/jeremyckahn/shifty
- 入门 / Getting Started：https://jeremyckahn.github.io/shifty/doc/pages/tutorials/getting-started.html

### Ola

Ola 是实时平滑插值数字的动画库。它的特点是数值会频繁更新，而且上一次过渡还没结束就能接着更新，不会生硬拐角——README 说这点上它比 TweenMax 平滑，Tween.js 则完全没有这种过渡。数学方程驱动，惰性计算：只在读取/更新时才算，只算被改的那一个值。
Ola is a library for smoothly interpolating numbers in realtime. Its trick: values update frequently, and a new update can start before the last transition finishes — no hard corners. The README says it's smoother than TweenMax at this, and Tween.js doesn't do this kind of transition at all. It's driven by math equations and computed lazily: only on read/update, and only for the value that changed.

适合什么场景：数值一直在变的实时画面，比如跟随鼠标、实时数据可视化、大量独立实例（README 举例 1000 个实例）、多值/多维（比如 `{x, y}`）。
Good for: realtime visuals where numbers keep changing — mouse followers, live data visualization, many independent instances (the README gives 1,000 as an example), multi-value/multi-dimensional state like `{x, y}`.

**安装 / Install**

```bash
npm install ola
```
不用 npm 的替代方式（README 原文）：
```html
<script src="https://cdn.jsdelivr.net/npm/ola"></script>
```
README 没写 Node 版本、没写任何前置要求，没核实到。

**验证 / Verify**

README 里的最小代码：
```js
import Ola from "ola";
const pos = Ola({ x: 0 });
console.log(pos.x); // 0
```
CDN 版验证是同一段逻辑（去掉 import，直接用全局 `Ola`）。

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/franciscop/ola。
> "参考 https://github.com/franciscop/ola，用 Ola 写一个跟随鼠标点击的小球：初始 `Ola({x: 0, y: 0})`，点击时 `pos.set({x: e.pageX, y: e.pageY})`，用 `setInterval` 每 10ms 读 `pos.x / pos.y` 更新小球的 `left/top`。过渡时间用默认 300ms，不要自己加 requestAnimationFrame 驱动。"

**链接 / Links**
- https://github.com/franciscop/ola
- 文档：README 本身就是文档，没找到独立的官方文档站。

---

## 3. 2D 矢量与画布 / 2D Vector & Canvas

这一组是在画布上画 2D 图形、做交互、播矢量动画、导出 GIF。四个。
This group draws 2D graphics on canvas, handles interaction, plays vector animation, and exports GIFs. Four skills.

### Two.js

Two.js 是面向现代浏览器的二维绘图 API，跟渲染器无关。同一套 API 能渲染到 webgl、canvas2d 和 svg 三种上下文。
Two.js is a two-dimensional drawing API for modern browsers, renderer-agnostic. The same API renders to webgl, canvas2d, and svg.

适合什么场景：画 2D 图形、做 2D 动画和交互绘图。
Good for: 2D shapes, 2D animation, interactive drawing.

**安装 / Install**

```bash
npm install --save two.js
```
不用 npm 的方式（README 原文）：下载最新的 minified library，在 HTML 里引入：
```html
<script src="js/two.min.js"></script>
```
自行构建（README "Custom Build"）：先装 Node.js（README 只写了先装 Node，没给版本号），然后在仓库目录跑：
```bash
cd ~/path-to-repo/two.js
npm install
node ./utils/build
```
无头/服务端环境（README "Running in Headless Environments"）：需另装 `canvas`：
```bash
npm install canvas
npm install two.js
```
正常安装没写前置要求；构建文档站才要求 Node 24 LTS，那是文档工具链的要求，不是装库的要求，不要混淆。
Normal installation states no prerequisites; only building the docs site requires Node 24 LTS — that's the docs toolchain, not a requirement for installing the library. Don't mix them up.

**验证 / Verify**

README 给的 boilerplate：画一个旋转矩形：
```html
<script src="js/two.min.js"></script>
<script>
  var two = new Two({ fullscreen: true, autostart: true }).appendTo(document.body);
  var rect = two.makeRectangle(two.width / 2, two.height / 2, 50, 50);
  two.bind('update', function() { rect.rotation += 0.001; });
</script>
```
ES6/React 版也有同款示例（`import Two from "two.js"`），逻辑一样。
The ES6/React version has the same example (`import Two from "two.js"`), same logic.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/jonobr1/two.js。
> "参考 https://github.com/jonobr1/two.js，用 Two.js 写一个全屏画布：中心画一个 50×50 的矩形，`autostart: true`，每帧 `rect.rotation += 0.001` 让它旋转。再给一个直接用 `<script src="js/two.min.js">` 的纯 HTML 版本。"

**链接 / Links**
- https://github.com/jonobr1/two.js
- 官网 / 文档：https://two.js.org

### Lottie Web

Lottie Web 是把 Adobe After Effects 动画（经 Bodymovin 导出为 JSON）解析后在 Web 上原生渲染的库。设计师在 AE 里做好动画、导出 JSON，网页里直接播，不用工程师手工重做。
Lottie Web parses Adobe After Effects animations (exported as JSON via Bodymovin) and renders them natively on the web. The designer makes the animation in AE, exports JSON, and the page just plays it — no hand-rebuilding by an engineer.

适合什么场景：AE 做好动画、网页直接播放。README 标题还提到 Android、iOS、React Native、Windows，但这个仓库是 Web 播放器部分。
Good for: animations made in AE, played straight on the web. The README title also mentions Android, iOS, React Native, and Windows, but this repo is the web player.

**安装 / Install**

HTML 播放器安装（README "HTML player installation"）：
```bash
# with npm
npm install lottie-web
# with bower
bower install bodymovin
```
或用 cdnjs 的脚本文件（README 给的链接是 https://cdnjs.com/libraries/bodymovin），或从 AE 插件里点 "Get Player" 获取。
Or use the script file from cdnjs (the README links https://cdnjs.com/libraries/bodymovin), or click "Get Player" in the AE plugin.

上面只是播放器。导出 JSON 需要先装 Bodymovin AE 插件，README 给了 6 个选项，推荐 Option 1：从 https://aescripts.com/bodymovin/ 下载。Option 6 是 Homebrew：
That's just the player. Exporting JSON needs the Bodymovin AE plugin first — the README gives 6 options, recommending Option 1, downloading from https://aescripts.com/bodymovin/. Option 6 is Homebrew:
```bash
brew tap danielbayley/adobe
brew cask install lottie
```
装插件之后还要做的事（README "After installing"）：在 AE 偏好设置里勾选 "Allow Scripts to Write Files and Access Network"（Windows：Edit > Preferences > Scripting & Expressions；Mac：Adobe After Effects > Preferences > Scripting & Expressions）。
After installing the plugin (README "After installing"): in AE preferences, check "Allow Scripts to Write Files and Access Network" (Windows: Edit > Preferences > Scripting & Expressions; Mac: Adobe After Effects > Preferences > Scripting & Expressions).

README 没写 Node 版本要求，没核实到。实际前置是 After Effects + Bodymovin 插件（导出 JSON 用），这是从 README 流程里看到的，不是版本号要求。

**验证 / Verify**

先在页面引入播放器，再调 `loadAnimation`。注意：需要一个已导出的 `data.json`，README 没有提供“零文件就能跑通”的验证：
Load the player on the page first, then call `loadAnimation`. Note: you need an already-exported `data.json` — the README offers no zero-file way to verify it works:
```html
<script src="js/lottie.js" type="text/javascript"></script>
```
```js
lottie.loadAnimation({
  container: element, // the dom element that will contain the animation
  renderer: 'svg',
  loop: true,
  autoplay: true,
  path: 'data.json' // the path to the animation json
});
```
返回动画实例，能用 `play / pause / stop / setSpeed` 控制。
It returns an animation instance you control with `play / pause / stop / setSpeed`.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/airbnb/lottie-web，再加你手里的 `data.json` 动画文件。
> "参考 https://github.com/airbnb/lottie-web，用 `npm install lottie-web` 后的播放器写一个页面：用 `lottie.loadAnimation({container, renderer: 'svg', loop: true, autoplay: true, path: 'data.json'})` 加载我给的 data.json，动画播完触发一次回调打印完成。再加两个按钮分别调实例的 `play()` 和 `pause()`。只用 README 里的 API。"

**链接 / Links**
- https://github.com/airbnb/lottie-web
- 文档 / Docs：http://airbnb.io/lottie/

### Konva

Konva 是 HTML5 Canvas 的开源 2D 框架，做交互式图形。场景图让每个图形自带事件、拖拽、变换、动画、缓存和导出。README 明确列出的用途：设计编辑器、白板、图表、标注、地图等视觉工具。MIT 协议，不用 license key。
Konva is an open-source 2D framework for HTML5 Canvas, built for interactive graphics. Its scene graph gives every shape its own events, dragging, transforms, animation, caching, and export. The README lists design editors, whiteboards, diagrams, annotations, and maps as uses. MIT license, no license key needed.

适合什么场景：要用户能动手拖、能缩放旋转、能导出图片的画布工具。
Good for: canvas tools where users drag, scale, rotate, and export images.

**安装 / Install**

CDN：
```html
<script src="https://unpkg.com/konva@10/konva.min.js"></script>
```
npm：
```bash
npm install konva
```
然后：
```js
import Konva from 'konva';
```
TypeScript：往 `tsconfig.json` 的 `compilerOptions.lib` 里加 `"es6"` 和 `"dom"`（README 给了完整 JSON 片段）。
TypeScript: add `"es6"` and `"dom"` to `compilerOptions.lib` in `tsconfig.json` (the README gives the full JSON snippet).

Node.js 环境（服务端渲染）：README 明确写了要手动另装渲染后端，二选一：
Node.js (server-side rendering): the README says you must install a render backend yourself, one of two:
```bash
# node-canvas backend
npm install konva canvas
# skia-canvas backend
npm install konva skia-canvas
```
再加一行后端导入：`import 'konva/canvas-backend';` 或 `import 'konva/skia-backend';`，创建 Stage 时省略 `container` 属性。

README 只写浏览器支持“modern mobile and desktop browsers that support ES2018”，没写 Node 版本号，没核实到。
The README only states browser support ("modern mobile and desktop browsers that support ES2018") and gives no Node version — not verified.

**验证 / Verify**

README 的最小代码（Quick Look 原文）：
```js
import Konva from 'konva';
const container = document.createElement('div');
document.body.appendChild(container);
const stage = new Konva.Stage({ container, width: 600, height: 400 });
const layer = new Konva.Layer();
const box = new Konva.Rect({
  x: 50, y: 50, width: 120, height: 80,
  fill: '#00a8e8', draggable: true,
});
layer.add(box);
stage.add(layer);
```
能看到一个可拖拽的蓝色矩形就是装好了。
If you see a draggable blue rectangle, it's installed.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/konvajs/konva。
> "参考 https://github.com/konvajs/konva，用 Konva 写一个 600×400 的舞台：加一个可拖拽的蓝色矩形（x=50, y=50, 120×80），选中时显示 Transformer 可以缩放旋转；再加一个按钮把舞台导出成图片。用 `npm install konva` 的 ES6 导入写法，不要用别的图形库。"

**链接 / Links**
- https://github.com/konvajs/konva
- 官网：https://konvajs.org（react-konva / vue-konva / svelte-konva / ng2-konva 的文档也在上面）

### gifenc

gifenc 是快速、轻量的纯 JavaScript GIF 编码器。支持标准 GIF 功能：静态图、动画、透明；浏览器和 Node.js 都能用（ESM + CJS）；9KB（GZIP 前）；内置快速颜色量化器；能在多个 Web Worker 间并行。
gifenc is a fast, lightweight, pure-JavaScript GIF encoder. It supports standard GIF features: still images, animation, transparency; works in browsers and Node.js (ESM + CJS); 9KB before GZIP; has a fast color quantizer built in; can run in parallel across multiple Web Workers.

适合什么场景：把 Canvas / RGBA 像素数据编码成 GIF。README 明确说最适合简单扁平风格的矢量图形，不是照片或视频（目前没有抖动支持，量化器选择所限）。
Good for: encoding canvas / RGBA pixel data into GIFs. The README says it's best for simple flat-style vector graphics, not photos or video (no dithering support yet, limited by the quantizer choice).

**安装 / Install**

重点先说清楚：**README 全文里没有 `npm install gifenc` 这条命令，不要凭记忆补。** README 里唯一出现的 `npm install` 是在讲从源码运行的那一节，是克隆仓库后跑源码测试用的（不带包名）：
To be clear first: **there is no `npm install gifenc` anywhere in the README — don't fill it in from memory.** The only `npm install` in the README is in the section about running from a local clone, for testing the source code after cloning (no package name):
```bash
npm install
```
README 示例的实际引入方式是直接从 CDN 导入（原文如此）：
```js
import { GIFEncoder, quantize, applyPalette } from 'https://unpkg.com/gifenc';
```

**验证 / Verify**

浏览器示例（Basic code example，原文；`getImageData()` 处是占位，要自己提供 RGBA 数据）：
Browser example (verbatim Basic code example; the `getImageData()` part is a placeholder — you supply your own RGBA data):
```js
import { GIFEncoder, quantize, applyPalette } from 'https://unpkg.com/gifenc';
const { data, width, height } = /* ... getImageData() ... */;
const palette = quantize(data, 256);
const index = applyPalette(data, palette);
const gif = GIFEncoder();
gif.writeFrame(index, width, height, { palette });
gif.finish();
const output = gif.bytes();
```
Node 验证（README 的源码运行一节）：克隆仓库并 `npm install` 后跑：
Node check (from the README's section on running from source): clone the repo, run `npm install`, then:
```bash
node test/encode_node.js
```
然后检查 `test/output/` 文件夹里的结果。

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/mattdesl/gifenc（README 本身就是 API 文档）。
> "参考 https://github.com/mattdesl/gifenc，写一个函数：输入一个 Canvas 的 RGBA 数据（`getImageData()` 得到），用 `quantize(data, 256)` 生成调色板，`applyPalette` 转成索引图，再用 `GIFEncoder()` 写 3 帧（每帧 `delay` 不同）、`finish()` 后返回 `gif.bytes()`。注意 README 说这个库没有抖动支持、最适合扁平矢量图，不要给我加照片级优化。安装命令 README 里没写，先不要假设 `npm install gifenc`，用 `https://unpkg.com/gifenc` 的导入写法。"

**链接 / Links**
- https://github.com/mattdesl/gifenc
- 文档：README 本身就是 API 文档；示例：https://looom-tools.netlify.app

---

## 4. 3D

这一组是在浏览器里做 3D：通用 3D 库、游戏渲染引擎、用 HTML 标签写 3D、桌面 3D 软件、游戏引擎。五个。
This group is 3D in the browser and beyond: general 3D libraries, game rendering engines, 3D in HTML tags, desktop 3D software, game engines. Five skills.

### Three.js

Three.js 是 JavaScript 3D 库，目标是好用、轻量、跨浏览器、通用。现在的构建只含 WebGL 和 WebGPU 渲染器，SVG、CSS3D 渲染器作为 addons 提供。
Three.js is a JavaScript 3D library aiming to be easy to use, lightweight, cross-browser, and general-purpose. Current builds include only the WebGL and WebGPU renderers; SVG and CSS3D renderers come as addons.

适合什么场景：浏览器里的通用 3D 场景、网页 3D 展示和动画交互、WebXR。
Good for: general 3D scenes in the browser, 3D showcases and interactive animation on web pages, WebXR.

**安装 / Install**

README 里**没有给 `npm install` 命令**，只给了克隆仓库的命令（终端 / git 环境）：
```bash
git clone --depth=1 https://github.com/mrdoob/three.js.git
```
完整历史克隆约 2 GB，用 `--depth=1` 能大幅减小。

`npm install --save three` 这条**本次没有从官方手册正文直接核实到**：官方手册安装页打开后只返回了 JS 驱动的文档索引，没读到正文。多家转述官方手册的第三方页面写法一致，都是 `npm install --save three`，但按规矩不算已核实，装之前自己在官方手册确认一下。
The `npm install --save three` command was **not directly verified** from the official manual page text — the page returned a JS-driven docs index with no readable body. Multiple third-party pages quoting the official manual agree on it, but that doesn't count as verified.

**验证 / Verify**

README 没有给“装完跑一条验证命令”。README 给的最小示例（前提是已经装好、能 `import * as THREE from 'three'`）：建场景 + 相机 + `BoxGeometry(0.2, 0.2, 0.2)` 立方体 + `WebGLRenderer`，`setAnimationLoop(animate)` 让立方体旋转。能看到旋转的立方体就是跑通了。
The README gives no "run this to verify" command. Its minimal example (assuming it's already installed and you can `import * as THREE from 'three'`) builds a scene + camera + a `BoxGeometry(0.2, 0.2, 0.2)` cube + `WebGLRenderer`, and spins the cube with `setAnimationLoop(animate)`. If you see the cube rotating, it works.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/mrdoob/three.js，再加你的需求。
> "参考 https://github.com/mrdoob/three.js 的 README，用 Three.js 写一个网页：一个会旋转的红色立方体，带轨道控制器，能用鼠标拖动旋转视角；给出完整的 index.html 和 main.js，并说明怎么本地运行。"

**链接 / Links**
- https://github.com/mrdoob/three.js
- 文档 / Docs：https://threejs.org/docs/；手册 / Manual：https://threejs.org/manual/

### Babylon.js

Babylon.js 是强大、好看、简单、开源的游戏和渲染引擎，打包在一个友好的 JavaScript 框架里。README 还建议先去 Playground 直接试 API。
Babylon.js is a powerful, beautiful, simple, and open game and rendering engine packed into a friendly JavaScript framework. The README suggests trying the API in the Playground first.

适合什么场景：网页 3D 游戏、渲染引擎、WebGL/WebGPU/WebXR 项目。
Good for: 3D games on the web, rendering engines, WebGL/WebGPU/WebXR projects.

**安装 / Install**

npm（终端，项目目录下）：
```bash
npm install babylonjs --save
```
或 CDN（HTML 里直接引用，README 明确警告：CDN 不应用于生产环境，只供学习和小实验；正式应用要自己托管）：
Or via CDN (referenced directly in HTML — the README warns explicitly: don't use the CDN in production, it's for learning and small experiments only; host it yourself for real apps):
```html
https://cdn.babylonjs.com/babylon.js
https://cdn.babylonjs.com/babylon.max.js
```
预览版：`https://preview.babylonjs.com/babylon.js`

导入（README 原文）：
Imports (verbatim from the README):
```js
import * as BABYLON from 'babylonjs';
```
或 `import { Scene, Engine } from 'babylonjs';`。TypeScript 还要在 `tsconfig.json` 的 `"types"` 里加 `"babylonjs"`。

另有 ES6 包（能 tree shaking），README 指向 doc.babylonjs.com 的 ES6 说明，没在正文给具体包名命令。
There's also an ES6 package (allows tree shaking) — the README points to the ES6 guide on doc.babylonjs.com and gives no specific package command in the main text.

注意：README 的 Contributing 一节说**开发本仓库**需要 Node.js 22.22.2+（22 线）或 24.16.0+（24 线）及 npm 11.18.0+——这是贡献代码的要求，不是日常用已发布包的要求，不要混淆。
Note: the README's Contributing section says **developing this repo** needs Node.js 22.22.2+ (v22 line) or 24.16.0+ (v24 line) and npm 11.18.0+ — that's for contributing code, not for using the published packages day to day. Don't mix them up.

**验证 / Verify**

README 的 Usage 示例就是最小可跑示例：取 `<canvas id="renderCanvas">`，`new BABYLON.Engine(canvas, true, …)`，建 `Scene` + `FreeCamera` + `HemisphericLight`，用 `MeshBuilder.CreateSphere` 建球、`CreateGround` 建地面，`engine.runRenderLoop(function(){ scene.render(); })`，监听 `resize` 调 `engine.resize()`。能看到球和地面就是跑通了。
The README's Usage example is the minimal runnable example: take a `<canvas id="renderCanvas">`, create `new BABYLON.Engine(canvas, true, …)`, build a `Scene` + `FreeCamera` + `HemisphericLight`, use `MeshBuilder.CreateSphere` for a sphere and `CreateGround` for the ground, run `engine.runRenderLoop(function(){ scene.render(); })`, and listen for `resize` to call `engine.resize()`. If you see the sphere and the ground, it works.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/BabylonJS/Babylon.js，再加需求。
> "参考 https://github.com/BabylonJS/Babylon.js 的 README，用 Babylon.js 写一个网页：一个带地面和灯光的 3D 场景，中间一个会上下浮动的球体，相机可拖动；用 `npm install babylonjs --save` 的方式引入，给出完整可运行代码。"

**链接 / Links**
- https://github.com/BabylonJS/Babylon.js
- 文档 / Docs：https://doc.babylonjs.com；官网：https://www.babylonjs.com；Playground：https://playground.babylonjs.com

### LUME

LUME 是 GPU 驱动的 3D HTML 工具包，口号是 "GPU-powered 3D HTML"。核心包 `lume` 提供 HTML 元素，用 `<lume-box>`、`<lume-scene>` 这类标签就能定义 2D/3D 应用，底层是 CSS3D、WebGL 或两者结合。
LUME is a GPU-powered 3D HTML toolkit. Its core package `lume` gives you HTML elements — write `<lume-box>`, `<lume-scene>`, and similar tags to define 2D/3D apps, powered by CSS3D, WebGL, or both.

适合什么场景：想用 HTML/自定义元素的方式做网页 2D/3D 交互、3D 模型和灯光阴影反射展示。兼容 React、Vue、Svelte 等。
Good for: 2D/3D interaction on the web using HTML and custom elements, 3D models with lights, shadows, and reflections. Works with React, Vue, Svelte, and others.

**安装 / Install**

README 只给了一行（npm，终端）：
```bash
npm install lume
```
README 里没有给其他安装方式、没有版本号、没有 Node 版本要求。

注意：网上搜 "Lume 安装" 会混进同名的其他产品（静态网站生成器 lume.land、Nebular 的 lume CLI、Adyen 的 Vue 组件库等），都不是这个仓库，以 README 的 `npm install lume` 为准。
Watch out: searching "Lume install" online pulls in other products with the same name (the lume.land static site generator, Nebular's lume CLI, Adyen's Vue component library) — none of them is this repo. Go by the README's `npm install lume`.

**验证 / Verify**

没核实到：README 里没有最小示例；官方文档首页（https://docs.lume.io/）只介绍了理念和元素名（比如 `<lume-sphere>`、`<lume-fbx-model>`、`<lume-shape>`、`<lume-point-light>`、`<lume-plane>`），没有一段能直接复制跑通的完整最小代码。这一步没核实到，先不要编。

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/lume/lume，再加需求。
> "参考 https://github.com/lume/lume，用 LUME 写一个网页：一个 `<lume-scene>` 场景里放一个 `<lume-box size=\"1 2 3\">`，加灯光，让盒子缓慢旋转；先用 `npm install lume` 安装，给出完整可运行的 HTML/JS。"

**链接 / Links**
- https://github.com/lume/lume
- 文档 / Docs：https://docs.lume.io/

### Blender

Blender 是免费开源的 3D 制作套件。README 原话：支持整个 3D 流程——建模、绑定、动画、模拟、渲染、合成、动捕、视频剪辑。它是桌面 3D 软件，不是 JS 库，没有 npm/pip 安装。
Blender is the free and open source 3D creation suite. Per the README, it supports the entire 3D pipeline — modeling, rigging, animation, simulation, rendering, compositing, motion tracking, and video editing. It's desktop 3D software, not a JS library — no npm/pip install.

适合什么场景：3D 建模、绑定、动画、模拟、渲染、合成、动捕、视频剪辑。
Good for: 3D modeling, rigging, animation, simulation, rendering, compositing, motion tracking, video editing.

**安装 / Install**

README 只讲了开发和编译方向，没有给日常安装命令。日常安装办法去官网核对过：https://www.blender.org/download/（2026-11-11 看到的页面，当时版本 Blender 5.3.0，2026 年 11 月 10 日发布）。该页列出的下载选项：
- Windows Installer 366 MB
- Windows Portable (.zip) 406 MB
- Windows Microsoft Store；Windows ARM Installer 251 MB；Windows ARM Portable (.zip) 288 MB
- macOS Apple Silicon 343 MB
- Linux 388 MB；Linux Snap Store；Steam
- 源码 92 MB；源码 + 依赖库 2.3 GB

按操作系统下载对应的安装包，运行安装就行；Linux 可以下压缩包解压直接用，还有 Snap 和 Steam 渠道。页面里没有写“双击安装包→下一步”这种分步文字。
Download the installer for your OS and run it; on Linux you can also download the archive, extract it, and use it directly, and there are Snap and Steam options too. The page doesn't spell out step-by-step install clicks.

**验证 / Verify**

没核实到：README 和官网下载页里都没有给验证命令（比如 `blender --version` 没在官方页面看到）。能核实的验证方式只有：下载后打开 Blender，能进入界面就是装好了。
Not verified: neither the README nor the official download page gives a check command (no `blender --version` was seen on the official pages). The only verifiable check is: download it, open Blender — if the interface loads, it's installed.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/blender/blender（先说明这是源码镜像，日常用官网安装包），再加具体任务。
> "参考 https://github.com/blender/blender，我已从 blender.org 下载安装了 Blender。教我用 Blender 做一个最简单的动画：一个立方体从左移到右、共 60 帧，并渲染成 MP4；一步步说清在界面里点哪里、关键帧怎么打。"

**链接 / Links**
- https://github.com/blender/blender
- 官网：https://www.blender.org；手册：https://docs.blender.org

### Godot

Godot 是功能齐全、跨平台的游戏引擎，用统一界面做 2D 和 3D 游戏，能一键导出到桌面（Linux/macOS/Windows）、移动（Android/iOS）、Web 和主机。免费开源（MIT 协议，没版税）。
Godot is a feature-packed, cross-platform game engine for 2D and 3D games from a unified interface. It exports to desktop (Linux/macOS/Windows), mobile (Android/iOS), web, and consoles. Free and open source (MIT, no royalties).

适合什么场景：2D/3D 游戏开发。
Good for: 2D and 3D game development.

**安装 / Install**

README 没有给安装命令。"Getting the engine" 一节只有两句话：二进制文件在 Godot 网站下；从源码编译的方法见官方文档。日常安装办法去官网核对过：https://godotengine.org/download/windows/（Godot 4 for Windows 页）。该页 Instructions 原文：
> "Extract and run. Godot is self-contained and does not require installation."

意思就是：下载对应平台的压缩包，解压后直接运行可执行文件，不用安装程序。该页列出的 Windows 下载项：x86_64、x86_32、arm64，以及带 C# 支持的 .NET 版（x86_64 / x86_32 / arm64）；导出模板（Export templates）要单独下载；.NET 版另需 .NET SDK；商店版（Steam 等）不含 .NET/C# 支持。
In other words: download the archive for your platform, extract it, and run the executable directly — no installer. The Windows page lists x86_64, x86_32, and arm64, plus .NET builds with C# support (x86_64 / x86_32 / arm64); export templates are a separate download; the .NET builds also need the .NET SDK; store versions (Steam and the like) don't include .NET/C# support.

**验证 / Verify**

按官网 Instructions：解压后直接运行 Godot 可执行文件，能打开编辑器就是能用。README 和官网下载页里没核实到命令行验证命令（比如 `godot --version` 没在官方页面看到）。
Per the official Instructions: extract it, run the Godot executable, and if the editor opens, it works. No command-line check (like `godot --version`) was verified in the README or the official download page.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/godotengine/godot，再加需求。
> "参考 https://github.com/godotengine/godot，我已从 godotengine.org/download/windows/ 下载解压了 Godot 4。用 GDScript 帮我做一个最小可玩示例：一个 2D 角色能用方向键移动、按空格跳跃，给出场景结构和完整脚本，并说明在编辑器里怎么运行。"

**链接 / Links**
- https://github.com/godotengine/godot
- 文档 / Docs：https://docs.godotengine.org；官网：https://godotengine.org

---

## 5. 粒子 / Particles

粒子就是一堆小点按物理规则动：火焰、烟花、爆炸、雨雪。这一组一个。
Particles are lots of tiny points moving by physics rules: fire, fireworks, explosions, rain and snow. One skill in this group.

### Proton

Proton 是轻量、强大的 JavaScript 粒子动画库。README 原话：用它能轻松做出各种酷的粒子效果。能模拟重力、布朗运动这些物理属性，能用在 React、Vue、Angular、Pixi.js、Phaser 这些场景里。
Proton is a lightweight and powerful JavaScript particle animation library. Per the README, it makes all kinds of cool particle effects easy. It simulates physics like gravity and Brownian motion, and works with React, Vue, Angular, Pixi.js, Phaser, and more.

适合什么场景：网页粒子特效——火焰、烟花、子弹、爆炸、风雨雪。
Good for: web particle effects — fire, fireworks, bullets, explosions, wind, rain, snow.

**安装 / Install**

npm（终端），README 特别注明：npm 包名已从 `proton-js` 改成 `proton-engine`：
```bash
npm install proton-engine --save
```
导入：
```js
import Proton from "proton-engine";
```
或 HTML 直接引入（README 原文，`js/proton.web.min.js` 是相对路径示例，不是 CDN 完整 URL）：
Or include it directly in HTML (verbatim from the README — `js/proton.web.min.js` is a relative-path example, not a full CDN URL):
```html
<script type="text/javascript" src="js/proton.web.min.js"></script>
```
从源码构建（README Building 一节，终端，要先装 Node）：
Building from source (the README's Building section, in the terminal, Node required first):
```bash
git clone git://github.com/drawcall/Proton.git
...
npm install
npm run build
```
跑示例：`npm start`，然后访问 `http://localhost:3001/example/`（中间的 `...` 是 README 原文省略号，不是省略）。
To run the examples: `npm start`, then visit `http://localhost:3001/example/` (the `...` in the middle is the README's own ellipsis, not mine).

**验证 / Verify**

README 的 Usage 就是最小示例：`new Proton()` → 建 `Emitter`，设 `Rate(new Span(10, 20), 0.1)`，加 `Radius(1, 12)`、`Life(2, 4)`、`Velocity(3, new Span(0, 360), "polar")` 初始化，加 `Color("ff0000", "random")`、`Alpha(1, 0)` 行为，`emitter.emit(5)`，`proton.addEmitter(emitter)`，再加 `CanvasRenderer(canvas)`。能看到红色粒子从画布中心喷出来就是跑通了。README 说“十几行代码就能做出粒子动画效果”。
The README's Usage section is the minimal example: `new Proton()`, create an `Emitter`, set `Rate(new Span(10, 20), 0.1)`, add initializers `Radius(1, 12)`, `Life(2, 4)`, `Velocity(3, new Span(0, 360), "polar")`, add behaviours `Color("ff0000", "random")` and `Alpha(1, 0)`, call `emitter.emit(5)`, `proton.addEmitter(emitter)`, then add a `CanvasRenderer(canvas)`. If you see red particles spraying from the center of the canvas, it works. The README says a dozen lines of code is enough for a particle effect.

**让 AI 帮你做 / Using it with AI**

交给 AI 仓库链接 https://github.com/drawcall/Proton，再加需求。
> "参考 https://github.com/drawcall/Proton，用 proton-engine（`npm install proton-engine --save`）写一个网页烟花粒子效果：点击画布任意位置就从该点喷发一次烟花，用 CanvasRenderer，给出完整可运行代码。"

**链接 / Links**
- https://github.com/drawcall/Proton
- 详细文档：https://projects.jpeer.at/proton/；示例站：http://drawcall.github.io/Proton/

---

## 通用用法 / How to Use Any of These

上面每条都写了自己的安装和指令，通用流程就三步：

1. 挑你要的 Skill，把 GitHub 链接丢给 Muse / Claude Code / Codex，让它学习安装。
   Pick a skill, give its GitHub link to Muse / Claude Code / Codex, and let it learn and install it.
2. 直接说你要的动画效果，AI 会按 Skill 的用法去做。
   Then just describe the animation you want — the AI builds it following the skill.
3. 不止 Muse 能用，Claude Code、Codex 也一样。
   Works with Claude Code and Codex too, not just Muse.
