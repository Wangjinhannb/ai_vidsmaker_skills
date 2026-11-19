# 用 ffmpeg-skill 让 AI 剪视频：本地剪片，42 个工具
# Let AI Edit Video with ffmpeg-skill: Local Editing, 42 Tools

这是我学本地剪片的笔记。一个 Skill，42 个工具，全是本地跑的 FFmpeg。用法就一个：装好，告诉 AI 你要什么，它去跑工具、量结果、再跟你汇报。不用上传视频，不用 API key。
These are my notes on learning local video editing. One skill, 42 tools, all running on local FFmpeg. The way to use it is always the same: install it, tell the AI what you want, and it runs the tools, measures the results, and reports back. No uploading video, no API key.

一句话：给你的 coding agent（Claude Code、Cursor、Codex）装一个本地视频编辑器。它不替你决定怎么剪，它只负责把活干对、干完再量一遍。
In one line: give your coding agent (Claude Code, Cursor, Codex) a local video editor. It doesn't decide how to cut — it just does the job right and measures it afterwards.

---

## 是什么 / What it is

ffmpeg-skill 是 GitHub 上的一个开源 Skill，MIT 协议。里面有 42 个 Python 脚本，每个脚本就是一个工具：剪、拼、去静音、改尺寸、加字幕、调响度、导出、出报告。全都调本地的 FFmpeg 和 ffprobe 来干活，Python 只用标准库，不装别的依赖。
ffmpeg-skill is an open-source skill on GitHub, MIT licensed. It ships 42 Python scripts, each one a tool: cut, join, silence removal, resizing, captions, loudness, export, reports. They all drive local FFmpeg and ffprobe. Python standard library only, no extra dependencies.

它有两层：SKILL.md 告诉 AI 工作流（先量、再剪、剪完再验），42 个工具负责干活。每个工具都带 `--help`、`--dry-run`（只出计划，不写文件）、`--json`（结构化结果）。工具跑完会自己再量一遍输出：时长、尺寸、帧率、音频对不对得上。
It has two layers: SKILL.md teaches the AI the workflow (measure first, then edit, then verify), and the 42 tools do the work. Every tool has `--help`, `--dry-run` (plan only, writes nothing) and `--json` (structured results). After a tool runs, it measures its own output again: duration, frame size, frame rate, audio — all checked.

除了脚本，它还有一个 MCP 服务器（`mcp/server.py`）。Claude Desktop、Cursor 这些 MCP 客户端不用开 shell 就能调这 42 个工具。默认只列出核心 12 个，省上下文；设 `FFMPEG_SKILL_MCP_FULL=1` 就列出全部 42 个。没列出来的也照样能调。
It also has an MCP server (`mcp/server.py`). MCP clients like Claude Desktop and Cursor can call all 42 tools without opening a shell. By default it lists only the core 12 to save context; set `FFMPEG_SKILL_MCP_FULL=1` to list all 42. Unlisted tools are still callable by name.

---

## 适合什么场景 / Good for

- **竖屏短视频**：横屏素材转 9:16，裁、垫黑边、模糊填充三种办法；一句话就能出 TikTok、Reels、Shorts 的版本。
  **Vertical short video**: turn landscape footage into 9:16 with crop, pad or blurred fill; one command gives you TikTok, Reels and Shorts versions.
- **剪片段**：从一小时录像里只留几段，拼成一分钟；无损剪优先，能不重编码就不重编码。
  **Cutting clips**: keep a few spans from an hour-long recording and join them into one minute; lossless cuts first, no re-encode unless needed.
- **加字幕**：SRT 烧进去、挂可开关的字幕轨、逐词蹦出来的卡拉 OK 字幕、从语音直接转写再上字幕。
  **Captions**: burn in SRT, mux a toggleable subtitle track, word-by-word karaoke captions, or transcribe speech and caption it.
- **去静音**：一键去掉死气沉沉的停顿，做跳切；还能只去掉"呃""啊"这种填充词。
  **Silence removal**: strip dead air in one go for jump cuts; it can also remove just the filler words like "um" and "uh".
- **调音量**：统一响度到 −14 LUFS，播客、短视频、广播各有各的标准。
  **Loudness**: normalize everything to −14 LUFS; podcasts, short video and broadcast each have their own target.
- **批量处理**：一个文件夹的视频按同一个配方全跑一遍，多核并行，还能守着文件夹自动处理新文件。
  **Batch jobs**: run the same recipe over a whole folder of videos, in parallel, or watch a folder and process new files as they arrive.
- **交片前自查**：导出的文件符不符合平台要求，逐项打勾：尺寸、时长、帧率、响度、黑帧、静音段。
  **Pre-delivery checks**: does the export meet the platform spec? It checks size, duration, frame rate, loudness, black frames and silent stretches, row by row.

---

## 工作流 / How it works

SKILL.md 定的顺序，照着走就行：
The order SKILL.md sets — just follow it:

1. **先量（probe）**：每个活都从 `probe.py` 开始，量真实的时长、帧率、分辨率、色彩、音频布局，再决定怎么剪。不靠文件名猜。
   **Probe first**: every job starts with `probe.py`, measuring the real duration, frame rate, resolution, colour and audio layout before deciding anything. Never guess from the file name.
2. **能无损就不重编码**：剪、换封装、只改音频这些活，能流拷贝就流拷贝。
   **Prefer lossless**: when a cut, remux or audio-only change is enough, stream-copy instead of re-encoding.
3. **先出计划再跑**：`--dry-run --json` 看一遍计划，确认了再真跑。信 `--json` 里的数字，不信一句话总结。
   **Plan, then run**: check the plan with `--dry-run --json` first, then run it for real. Trust the numbers in `--json`, not the one-line summary.
4. **按顺序串起来**：颜色 → 剪 → 拼 → 去静音 → 改尺寸 → 字幕/叠加 → 对齐 → 音频 → 响度 → 导出。一次要发到某个平台，直接 `render.py --template`；三步以上，写个 `render.py` 项目。
   **Chain in order**: colour → cut → join → silence → fit → caption/overlay → sync → audio → loudness → export. One delivery request is a single `render.py --template`; three or more steps, write a `render.py` project.
5. **检查（check）**：`check.py` 按平台表逐项打勾，格式行不过就修；时长、尺寸、帧率、响度只在要求里有暗示时才修。
   **Check**: `check.py` ticks each row against the platform table; fix format rows, and fix duration, aspect, fps or loudness only when the request implies it.
6. **再量一遍（verify）**：拿写文件时 `--json` 里的量测结果，或者再跑一遍 `probe.py`。退出码是 0、数字都对，才算成了。
   **Verify again**: use the measurements from the writer's `--json` output, or run `probe.py` again. Exit 0 and the right numbers, or it failed.
7. **原片不动**：永远不覆盖源文件。`--overwrite` 只用来换这单活自己的文件。
   **Keep originals**: never overwrite a source file. `--overwrite` is only for this job's own files.
8. **看一眼（look）**：改过画面的活，`look.py` 出一张六格拼图，看一眼再交。
   **Look**: after any picture change, `look.py` makes a six-tile contact sheet — look at it before handing over.

它只执行剪辑，不替你做决定：哪刀该剪、成片能不能交、什么算高光、字幕该写什么字——这些都归你、归调用它的 agent、归别的 skill。它只量、只报。
It executes the edit; it doesn't decide one. Which cut is right, whether a deliverable is approvable, what makes a highlight, what the captions should say — those belong to you, the calling agent, or another skill. This one only measures and reports.

---

## 安装 / Install

前置就两样：FFmpeg 6.1 以上，Python 3.10 以上。Python 只用标准库，不用装别的包。`npx` 装 Skill 要 Node 16 以上。
Two prerequisites: FFmpeg 6.1 or newer, and Python 3.10 or newer. Python standard library only, nothing else to install. The `npx` installer needs Node 16 or newer.

**第一步：先装 FFmpeg 本身（README 里的 Install 表原样）**
**Step 1: FFmpeg itself first (verbatim from the README's Install table)**

```bash
# macOS
brew install ffmpeg-full
# 注意：普通的 ffmpeg 公式缺 subtitles、drawtext、zscale 这几个滤镜，
# 字幕相关的工具跑不起来，所以 README 点名要 ffmpeg-full。
# Note: the plain ffmpeg formula lacks the subtitles, drawtext and zscale
# filters, so the caption tools won't run — the README names ffmpeg-full.

# Ubuntu / Debian
sudo apt install ffmpeg

# Windows
winget install Gyan.FFmpeg
```

macOS 还有个坑：`ffmpeg-full` 是 keg-only，Homebrew 装完不链进 PATH。装完先看 `command -v ffmpeg` 指到哪。想让整个 shell 都用它，把这一行加进 shell profile：
macOS has one more catch: `ffmpeg-full` is keg-only, so Homebrew installs it without linking it into PATH. After installing, check where `command -v ffmpeg` points. To make the whole shell use it, add this line to your shell profile:

```bash
export PATH="$(brew --prefix ffmpeg-full)/bin:$PATH"
```

或者链过去（README 原话）：
Or link it over the lean formula (the README's own words):

```bash
brew unlink ffmpeg 2>/dev/null; brew link --overwrite --force ffmpeg-full
```

README 还写了：就算 PATH 上是精简版 ffmpeg，这个 skill 自己会去找同个 Homebrew 目录下的 `ffmpeg-full` 来用，`doctor` 会告诉你它用的是哪个。
The README also says: even if the lean ffmpeg is on PATH, the skill finds the `ffmpeg-full` from the same Homebrew prefix by itself, and `doctor` tells you which one it used.

**第二步：装 Skill（README 里的命令原样）**
**Step 2: install the skill (commands verbatim from the README)**

```bash
npx ffmpeg-skill            # Claude Code → ~/.claude/skills/ffmpeg-skill
npx ffmpeg-skill --cursor   # Cursor → ~/.cursor/skills/ffmpeg-skill
npx ffmpeg-skill --codex    # Codex → ~/.agents/skills/ffmpeg-skill（Cursor 也读这个位置）
npx ffmpeg-skill --all      # 三个都装
npx ffmpeg-skill --project  # 只装进这个项目 → ./.claude/skills/ffmpeg-skill
npx ffmpeg-skill --dir ./my-skills   # 装到自己指定的目录
```

装过之后想更新，重新跑一遍 `npx ffmpeg-skill` 就行，旧的拷贝不会自己更新。
Already installed and want to update? Just run `npx ffmpeg-skill` again — old copies are not updated automatically.

**不用 Node 的装法（README 原话）**：把仓库克隆下来，把 `SKILL.md`、`package.json`、`LICENSE`、`scripts/`、`templates/`、`mcp/`、`references/` 和 `docs/contract.md` 这一个文件拷进你 agent 的 skills 目录。就是 `npx ffmpeg-skill` 装的那套东西。注意：没 `templates/` 就用不了 `--template`；没 `package.json`，contract 报的版本号就是 unknown。
**Without Node (the README's own words)**: clone the repository and copy `SKILL.md`, `package.json`, `LICENSE`, `scripts/`, `templates/`, `mcp/`, `references/` and the single file `docs/contract.md` into your agent's skills directory. That is exactly what `npx ffmpeg-skill` installs. Note: without `templates/` there is no `--template` rendering, and without `package.json` the contract reports its version as unknown.

**别的装法（README Install 表里的）**
**Other install routes (from the README's Install table)**

```bash
# Claude Code 插件（Claude Code 2.1.275 或更新）
/plugin install ffmpeg-skill --marketplace kajisho5/ffmpeg-skill
# 2.1.69 到 2.1.274 两个命令：
claude plugin marketplace add kajisho5/ffmpeg-skill --sparse .claude-plugin
claude plugin install ffmpeg-skill@ffmpeg-skill

# Codex 插件
codex plugin marketplace add kajisho5/ffmpeg-skill --sparse .claude-plugin
codex plugin add ffmpeg-skill@ffmpeg-skill

# Gemini CLI 扩展
gemini extensions install https://github.com/kajisho5/ffmpeg-skill

# MCP 服务器（给没有 shell 的客户端）
npx -y ffmpeg-skill mcp
```

**语音转写（可选）**：`caption.py --transcribe` 和 `silence.py --transcribe` 要一个本地语音引擎，没装也能用别的全部功能。README 说 `pip install faster-whisper` 最简单，第一次跑会下载模型，音频不离开本机。没装引擎，`doctor` 会把它列为缺失的可选项，`--transcribe` 会直接拒绝并给出安装方法，而不是瞎猜。
**Speech transcription (optional)**: `caption.py --transcribe` and `silence.py --transcribe` need one local speech engine; without one, everything else still works. The README says `pip install faster-whisper` is the simplest; the first run downloads its model, and the audio never leaves the machine. Without an engine, `doctor` lists it as missing-optional, and `--transcribe` refuses with the install line instead of guessing.

---

## 验证 / Verify

装完先跑这两条（README 原话）：
Run these two right after installing (the README's own words):

```bash
npx ffmpeg-skill doctor          # 这台机器上每个工具要的 FFmpeg 组件全不全
npx ffmpeg-skill contract --json # agent 框架会看到的机器可读合同
```

`doctor` 看的是 `ffmpeg -encoders`、`-decoders`、`-filters`、`-bsfs` 的输出，逐项对合同里声明的能力。每个能力三种状态：有、缺、未知。全部必需的都有才退出 0。注意：`doctor` 的总 `ok` 和某个工具的 `usable` 是两回事——`ok` 是 false 只要有任何一个工具要的东西缺了（比如 Homebrew 精简版 ffmpeg 没 `subtitles`，或者没 libvidstab 的构建），但大部分工具照样能跑。想知道某个工具能不能跑，看 `doctor --json` 里 `tools` 那一项，不要只看 `ok`。
`doctor` reads the output of `ffmpeg -encoders`, `-decoders`, `-filters` and `-bsfs` and resolves every capability the contract declares against this machine's build. Three states per capability: available, missing, unknown. Exit 0 only when everything required is available. Note: `doctor`'s overall `ok` and a single tool's `usable` are different signals — `ok` is false when anything *some* tool requires is missing (a plain Homebrew `ffmpeg` without `subtitles`, or a build without libvidstab), even though most tools still run. Check `doctor --json`'s `tools` field for the per-tool answer, not just `ok`.

再拿一个真文件跑一遍最小的活：
Then run the smallest real job on an actual file:

```bash
S=~/.claude/skills/ffmpeg-skill/scripts
python3 $S/probe.py input.mp4 --compact   # 先量：时长、帧率、分辨率、音频
python3 $S/cut.py input.mp4 --start 0:10 --end 0:40 --dry-run --json  # 先看计划，不写文件
```

`probe.py` 报出数字、`cut.py --dry-run` 打印出计划、`doctor` 没报必需项缺失——三条都过了，就是装好了。
If `probe.py` reports numbers, `cut.py --dry-run` prints a plan, and `doctor` reports nothing required missing — all three pass, and it's installed.

---

## 配置 / Configure

**MCP 怎么配（README 原话）**：MCP 客户端里这样配，路径换成你自己的安装位置：
**How to configure MCP (the README's own words)**: in your MCP client, with the path changed to your own install location:

```json
{"mcpServers": {"ffmpeg-skill": {"command": "python3", "args": ["/Users/you/.claude/skills/ffmpeg-skill/mcp/server.py"]}}}
```

Windows 上注意：`python3` 只有从 Microsoft Store 装的 Python 才在 PATH 上，python.org 装的只有 `python`（或者 `py` 启动器）。MCP 客户端报服务器启动失败，就把上面 `"command"` 里的 `python3` 改成 `python`，或者写 `where python` 给出的完整路径。
On Windows, note: `python3` is only on PATH if Python was installed from the Microsoft Store; a python.org install exposes `python` (or the `py` launcher) instead. If your MCP client reports the server failed to start, change `"command"` above to `"python"` (or the full path from `where python`).

默认 `tools/list` 只列核心 12 个（`render`、`look`、`caption`、`export`、`check`、`fit`、`cut`、`audio`、`loudness`、`graphics`、`silence`、`probe`），省上下文。想全列出来：
By default `tools/list` shows only the core 12 (`render`, `look`, `caption`, `export`, `check`, `fit`, `cut`, `audio`, `loudness`, `graphics`, `silence`, `probe`) to save context. To list them all:

```bash
FFMPEG_SKILL_MCP_FULL=1   # 列出全部 42 个
```

只想留一个小清单：
For a small list instead:

```bash
FFMPEG_SKILL_MCP_TOOLS=probe,cut,fit,export,check
```

服务器还带五个现成的工作流配方（`prompts/list` 能看到）：`reel`、`podcast`、`multicam`、`delivery_check`、`hdr`。让 agent 按名字要一个配方，比自己拼一串工具调用省事。
The server also advertises five canned workflow recipes (`prompts/list` shows them): `reel`, `podcast`, `multicam`, `delivery_check`, `hdr`. Ask the agent for one by name instead of composing the tool calls yourself.

**几个环境变量（README 和 SKILL.md 里写明的）**：
**A few environment variables (stated in the README and SKILL.md)**:

```bash
FFMPEG_SKILL_ASR_ENGINE=large-v3-turbo   # 语音转写用哪个引擎
FFMPEG_SKILL_ASR_MODEL=large-v3-turbo    # 用哪个模型；不设就是 auto 和 base
FFMPEG_SKILL_HW=1                        # 默认用 Apple VideoToolbox 硬编（只在 Apple Silicon 上，AV1 还是走 SVT-AV1）
FFMPEG_SKILL_TIMEOUT=1800                # 每个工具跑 FFmpeg 的超时秒数，0 是不限
```

**品牌 kit**：字幕、叠加、图形这三组工具都认 `--brand brand.json`，字体、颜色、logo 一次写好，每次都按这个来。
**Brand kit**: the caption, overlay and graphics tools all take `--brand brand.json` — fonts, colours and logo written once, applied every time.

**平台模板**：`render.py --template` 自带这些名字：`tiktok`、`reels`、`shorts`、`youtube-shorts`、`youtube`、`x`、`linkedin`、`facebook`、`podcast`、`audiogram`。`--template all` 一次出七个视频目的地（tiktok、reels、shorts、youtube、x、linkedin、facebook），再加一张 `_pack.md` 表，写清每个出了什么。`render.py --list-templates` 能打印每个模板的画幅、限制和安全区。别名也认：`youtube-shorts` 就是 `shorts`，`ig` 就是 `reels`，`twitter` 就是 `x`，`fb` 就是 `facebook`。
**Platform templates**: `render.py --template` ships these names: `tiktok`, `reels`, `shorts`, `youtube-shorts`, `youtube`, `x`, `linkedin`, `facebook`, `podcast`, `audiogram`. `--template all` renders the seven video destinations (`tiktok`, `reels`, `shorts`, `youtube`, `x`, `linkedin`, `facebook`) from one edit, plus a `_pack.md` table of what each one produced. `render.py --list-templates` prints each template's frame, limits and safe zones. Aliases work too: `youtube-shorts` is `shorts`, `ig` is `reels`, `twitter` is `x`, `fb` is `facebook`.

---

## 42 个工具，分组写 / The 42 tools, grouped

README 把 42 个工具分成六组：分析、剪辑、音频、画面、交付、编排。我按功能再拆细一点，写成九组。每组先说这组是干嘛的、适合什么场景，再一个一个列工具。命令都来自 SKILL.md 的 request→script 表和 README 的工具表。
The README splits the 42 tools into six groups: analysis, editing, audio, picture, delivery, orchestration. I split them a little finer, into nine groups. Each group first says what it's for and which scenarios fit, then lists its tools one by one. The commands come from SKILL.md's request→script table and the README's tool table.

### 1. 先看片：分析 / Analysis & inspection — 3 个

干活之前先量。不知道源文件到底多大、多长、什么编码，剪出来全是猜。这组只读不写。
Measure before touching anything. Without knowing the source's real size, length and codecs, every edit is a guess. This group only reads, never writes.

- **`probe.py`**：量一个文件的一切——时长、帧率（还能发现是不是可变帧率）、分辨率、编码、位深、HDR 格式（包括杜比视界）、色彩空间、旋转、每条音轨。`--analyze` 还能标出 Log 素材。每个活的第一步就是它。
  **probe.py**: measures everything about a file — duration, frame rate (and whether it's variable frame rate), resolution, codecs, bit depth, HDR format (including Dolby Vision), colour space, rotation, every audio stream. `--analyze` also flags Log footage. The first step of every job.
  ```bash
  python3 $S/probe.py input.mp4 --compact
  ```
- **`scenes.py`**：找场景切换点、音频峰值、按响度或时长排的高光提议（注意：提议只是按响度排的代理指标，不是"最好"的那几段）。还能测音乐的节拍网格、按光流把每个镜头分静态/平移/运动、报每秒的 dBFS、报说话和音乐的比例。
  **scenes.py**: finds scene changes, audio peaks, and highlight proposals ranked by loudness or duration (note: the ranking is a measured proxy, not the "best" spans). It can also measure the music's beat grid, classify each shot as static/pan/motion by measured optical flow, list per-second dBFS, and report a speech-vs-music ratio.
  ```bash
  python3 $S/scenes.py long.mp4 --highlights 6 --target 60 --edl picks.txt
  ```
- **`look.py`**：出图给 agent 看：单帧、六格拼图（`--tiles 3x2`）、左右对照。`--safe NAME` 会把某个平台自己的 UI 盖住的区域涂出来，`--guides title,action` 画出广播安全区。改过画面之后，看一眼拼图再交。
  **look.py**: makes pictures for the agent to look at — single frames, a six-tile contact sheet (`--tiles 3x2`), side-by-side comparisons. `--safe NAME` shades the zones a platform's own UI covers; `--guides title,action` outlines the broadcast safe areas. After any picture change, look at the sheet before handing over.
  ```bash
  python3 $S/look.py out.mp4 --tiles 3x2
  ```

### 2. 剪切拼接 / Cutting & joining — 3 个

最常用的三个。剪一段、拼几段、去掉停顿。优先无损：能流拷贝就不重编码。
The three you'll use most. Cut a span, join a few, strip the pauses. Lossless first: stream-copy when it can, re-encode only when it must.

- **`cut.py`**：按起止时间剪，或者一次留多个段（`--segments 0-1:00,1:30-2:00`），或者删掉中间一段（`--remove 1:00-1:30`）。默认先试无损 `-c copy`，不行再重编码。`--accurate` 是帧精确 + 音频采样精确；`--smart` 是 H.264 的帧精确剪法，只重编码切点附近不完整的 GOP，其余照拷。还能 `--snap beats` 把切点对到节拍上。
  **cut.py**: cut by in/out times, keep several spans at once (`--segments 0-1:00,1:30-2:00`), or drop a middle span (`--remove 1:00-1:30`). Lossless `-c copy` first, re-encode fallback. `--accurate` is frame-exact video and sample-exact audio; `--smart` is a frame-exact H.264 cut that re-encodes only the partial GOPs at the cut points and copies the rest. `--snap beats` moves the cut points onto a measured beat.
  ```bash
  python3 $S/cut.py in.mp4 --start 1:20 --end 2:05
  python3 $S/cut.py in.mp4 --segments 0-1:00,1:30-2:00
  ```
- **`join.py`**：把几个片段拼成一个，带淡入淡出（`--transition fade --duration 0.5`），自动统一尺寸、帧率、采样率、声道。`--copy --transition none` 是无损拼，要求流对得上，对不上就按名字拒绝。还能读一个 `parts.txt` 清单来拼 TTS 念出来的句子。
  **join.py**: concatenate clips with crossfade transitions (`--transition fade --duration 0.5`), normalising size, fps, sample rate and channel layout. `--copy --transition none` joins by stream copy when the streams match, and refuses a mismatch by name. It can also read a `parts.txt` list to join TTS lines.
  ```bash
  python3 $S/join.py a.mp4 b.mp4 --transition fade --duration 0.5
  python3 $S/join.py a.mp4 b.mp4 --copy --transition none
  ```
- **`silence.py`**：检测并去掉死气（跳切），说话前后各留一点余量（`--margin`，默认每边 0.2 秒）。能出剪切清单，也能出源时间和输出时间的对照表（`--timemap`）。`--filler` 连"呃""啊"这种填充词一起去掉，但前提是语音引擎给这些词打过时间戳；`--speech-aware` 只在句子的边界停顿处剪，句子中间的气口留着。
  **silence.py**: detect and remove dead air (jump cuts), keeping a margin around speech (`--margin`, 0.2 s each side by default). It can list or export the cut list, or a source-to-output time map (`--timemap`). `--filler` also removes filler words, but only where a speech engine timed them; `--speech-aware` keeps a breath inside a sentence and cuts only at sentence-boundary pauses.
  ```bash
  python3 $S/silence.py in.mp4
  python3 $S/silence.py in.mp4 --filler --words words.json
  ```

### 3. 尺寸和时长 / Size & duration — 8 个

竖屏、方形、定长、定尺寸、变速、定格，全在这组。教学视频从横屏改竖屏，主要就是 `fit.py` 的活。
Vertical, square, fixed length, fixed size, speed changes, freeze frames — all here. Turning a landscape teaching video into vertical is mostly `fit.py`'s job.

- **`fit.py`**：最常用。改时长（保调变速或剪掉、平滑慢动作）、改画幅（`--aspect 9:16 --fit pad` 垫黑边、`crop` 裁掉、`blur` 模糊填充）、改精确尺寸（`--height 1080`）。还能转 90/180/270 度、左右翻转、强制恒定帧率。改完会量写出来的那一帧，`crop_check` 还会提醒裁掉的画面里有没有丢掉运动或细节。
  **fit.py**: the one you'll use most. Fit to a duration (pitch-preserving speed change or trim, smooth slow-mo), to an aspect ratio (`--aspect 9:16 --fit pad` for letterbox, `crop` to cut, `blur` for blurred fill), or to an exact size (`--height 1080`). Rotate 90/180/270, flip h/v, force constant fps. It measures the written frame afterwards, and `crop_check` flags discarded motion or detail.
  ```bash
  python3 $S/fit.py in.mp4 --aspect 9:16 --fit crop
  python3 $S/fit.py in.mp4 --duration 60 --method trim
  ```
- **`crop.py`**：按精确像素框裁（`--x --y --width --height`）。跟 `fit.py --fit crop` 不一样：那个是按画幅比自己算框，这个是你给死框。
  **crop.py**: crop to an exact pixel rectangle (`--x --y --width --height`). Different from `fit.py --fit crop`, which crops to an aspect ratio it computes itself.
  ```bash
  python3 $S/crop.py in.mp4 --x 100 --y 0 --width 1080 --height 1920
  ```
- **`cropdetect.py`**：只量不写。量出已有的黑边（上下黑条、左右黑柱），报出 `crop.py` 能直接用的框。`--motion-centre` 报每秒的运动重心，9:16 重构时用来定裁哪边——只报数，不自动重构，裁哪边归调用它的 agent 定。
  **cropdetect.py**: analysis only, writes no file. Measures existing black letterbox/pillarbox bars and reports the `crop.py`-ready rectangle that removes them. `--motion-centre` reports the per-second motion centroid for a 9:16 reframe — report only, the calling agent picks the crop, no auto-reframing.
  ```bash
  python3 $S/cropdetect.py in.mp4
  ```
- **`speedramp.py`**：一段一段变速（`--segment 0-3:1.0 --segment 3-4:0.25`，能重复写）。跟 `fit.py` 不一样：那个是整段一个速度，这个是分段变速。
  **speedramp.py**: step through different constant speeds across a clip (`--segment 0-3:1.0 --segment 3-4:0.25`, repeatable). Different from `fit.py`: that one is a single whole-clip speed factor, this one is per-segment.
  ```bash
  python3 $S/speedramp.py in.mp4 --segment 0-3:1.0 --segment 3-4:0.25
  ```
- **`loop.py`**：循环一个片段，循环 N 次（`--times`）或者撑满一个目标时长（`--duration 30`）。背景循环、填固定时段用。`--crossfade 0.5` 藏住接缝，`--boomerang` 正放一遍再倒放一遍。
  **loop.py**: repeat a clip `--times` N or to a target `--duration` — for background loops and filling a fixed slot length. `--crossfade` smooths the seam, `--boomerang` plays it forward then backward.
  ```bash
  python3 $S/loop.py clip.mp4 --duration 30
  ```
- **`reverse.py`**：倒放（视频和音频一起倒，除非 `--no-audio`）。
  **reverse.py**: reverse playback (video and, unless `--no-audio`, audio).
  ```bash
  python3 $S/reverse.py in.mp4
  ```
- **`freeze.py`**：定住一帧 N 秒（`--at` 定位置，`--hold` 定秒数，`--mode insert/extend`）。结尾单词卡停 5 秒、或者一个喜剧停顿，就用它。
  **freeze.py**: hold a frame for N seconds (`--at`, `--hold`, `--mode insert/extend`) — an end-card hold or a comedic beat.
  ```bash
  python3 $S/freeze.py clip.mp4 --hold 2
  ```
- **`pad.py`**：在时间线开头和/或结尾加黑场静音（`--start`、`--end`）。跟 `fit.py --fit pad` 的每帧黑边不是一回事。
  **pad.py**: add black/silent padding at the start and/or end of the timeline (`--start`, `--end`). Different from `fit.py --fit pad`'s per-frame letterbox bars.
  ```bash
  python3 $S/pad.py clip.mp4 --start 1.5
  ```

### 4. 画面修复 / Picture cleanup — 6 个

老素材、手机拍的、手抖的、带水印的，这组是修修补补。注意：它只按你给的参数修，不替你找脸、找水印、找主体。
Old footage, phone clips, shaky shots, watermarked frames — this group patches things up. Note: it only fixes by the parameters you give; it doesn't find faces, logos or subjects for you.

- **`deinterlace.py`**：隔行素材转逐行（`yadif`），`--mode frame`/`field`，`--parity`。
  **deinterlace.py**: deinterlace interlaced source footage (`yadif`), `--mode frame`/`field`, `--parity`.
  ```bash
  python3 $S/deinterlace.py in.mp4
  ```
- **`denoise.py`**：降噪去颗粒（`hqdn3d`），`--strength low/medium/high`，或者自己调空间/时间参数。
  **denoise.py**: reduce video noise/grain (`hqdn3d`), `--strength low/medium/high` or individual spatial/temporal overrides.
  ```bash
  python3 $S/denoise.py in.mp4 --strength medium
  ```
- **`redact.py`**：把一个精确像素框整个片子模糊、打码，或者 `--mode delogo` 插值抹掉。框是你给的，工具自己不找脸、不找车牌、不找 logo。`delogo` 是边缘混合，不是修复式填充。
  **redact.py**: blur, pixelate, or (`--mode delogo`) interpolate away an exact pixel rectangle for the whole clip. The box is given: nothing here finds a face, a plate or a logo, and `delogo` is a border blend, not inpainting.
  ```bash
  python3 $S/redact.py in.mp4 --x 820 --y 140 --width 240 --height 240 --mode pixelate
  ```
- **`sphere.py`**：从 360 度全景视频里抠一个平面视角出来（`--yaw --pitch --roll --h-fov --v-fov`）。没有主体追踪，只按你给的朝向。
  **sphere.py**: extract a flat rectilinear viewport from a 360/spherical video (`--yaw --pitch --roll --h-fov --v-fov`); no subject tracking, only the aim you give it.
  ```bash
  python3 $S/sphere.py in.mp4 --yaw 90 --h-fov 100 --v-fov 70
  ```
- **`straighten.py`**：地平线歪了，按任意角度转正（`--degrees`，`--fit crop/pad`）。跟 `fit.py --rotate` 的 90 度整转不一样。
  **straighten.py**: rotate by an arbitrary angle for horizon correction (`--degrees`, `--fit crop/pad`) — distinct from `fit.py --rotate`'s exact 90-degree turns.
  ```bash
  python3 $S/straighten.py in.mp4 --degrees -2.5
  ```
- **`stabilize.py`**：两遍防抖（`vidstabdetect`/`vidstabtransform`）。要 libvidstab，`doctor` 会报有没有。
  **stabilize.py**: two-pass motion stabilisation (`vidstabdetect`/`vidstabtransform`). Needs libvidstab; `doctor` reports whether it's there.
  ```bash
  python3 $S/stabilize.py in.mp4
  ```

### 5. 生成画面 / Generated clips — 6 个

手头没素材的时候现做：标题卡、纯色底、照片变视频、波形视频、画中画、九宫格。都不用输入视频文件，或者只拿图片当输入。
When there's no footage to work with, make some: title cards, solid-colour clips, photos into video, waveform videos, picture-in-picture, grids. These need no video input, or take stills as input.

- **`insert.py`**：一张静图变成固定时长的无声视频片段（标题卡、结尾板），定好帧尺寸和帧率，还能加 Ken Burns 推拉。
  **insert.py**: turn a still image into a silent, fixed-duration video clip (title card, end slate) at an exact frame size / fps, with an optional Ken Burns zoom/pan.
  ```bash
  python3 $S/insert.py title.png --duration 3
  ```
- **`background.py`**：生成纯色或双色渐变片段，定尺寸定长度。不用输入文件。`--transparent` 写全透明画布（ProRes 4444、QuickTime Animation、PNG 或带 alpha 的 VP9），用来往上画图形。
  **background.py**: generate a solid-colour or two-colour gradient clip at an exact size/duration — no input file; `--transparent` writes a fully transparent canvas (ProRes 4444, QuickTime Animation, PNG or VP9 with alpha) to draw graphics on.
  ```bash
  python3 $S/background.py -o bg.mp4 --duration 3 --width 1920 --height 1080 --color 0x101010
  ```
- **`sequence.py`**：编号图片（`frame_%04d.png`）或者通配符匹配的图片拼成视频。
  **sequence.py**: numbered (`frame_%04d.png`) or glob-matched still images into a video.
  ```bash
  python3 $S/sequence.py --dir frames --pattern "frame_%04d.png" --fps 24
  ```
- **`waveform.py`**：把音轨画成波形或频谱视频（`showwaves`/`showspectrum`）。纯音频、没画面可看的时候用。`--image` 能把波形叠在一张静图上，做 audiogram（播客转视频），还能带 `--platform`、`--title`、烧进去的字幕。
  **waveform.py**: render an audio track as a waveform or spectrum visualization video (`showwaves`/`showspectrum`) — for audio-only inputs with no picture worth showing; `--image` puts the visualisation over a still plate (an audiogram), with `--platform`, `--title` and burnt-in captions.
  ```bash
  python3 $S/waveform.py ep.wav -o waveform.mp4
  ```
- **`broll.py`**：在主视频的某个时间窗上插一段 B-roll（`--insert B --at T --duration D`，能重复），插完回来。主视频的长度和音频默认不动。
  **broll.py**: cut away to a B-roll clip over the A-roll for a window (`--insert B --at T --duration D`, repeatable) and come back; A's length and audio untouched by default.
  ```bash
  python3 $S/broll.py talk.mp4 --insert product.mp4 --at 12 --end 16 --audio b\|mix
  ```
- **`grid.py`**：把几个视频拼成 `--cols` x `--rows` 的格子，每个格子垫黑边，默认标文件名（`--label none` 不标）。
  **grid.py**: composite `--cols`x`--rows` clips into one grid, each cell letterboxed and labelled with its filename by default (`--label none` to skip).
  ```bash
  python3 $S/grid.py t1.mp4 t2.mp4 t3.mp4 t4.mp4 --cols 2 --rows 2
  ```

### 6. 音频 / Audio — 3 个

音频是头等输入：WAV、FLAC、MP3、M4A/AAC、OGG、Opus 走 `probe`、`cut`、`join`、`silence`、`loudness`、`audio`、`sync` 和 `check --platform podcast`，命令跟视频一样。输出扩展名决定编码：`-o out.wav` 写 PCM，`-o out.flac` 写 FLAC，`-o out.mp3` 写 MP3。画面工具（fit、caption、overlay、graphics、color、export、look）碰到纯音频文件会直接拒绝，不会凭空编一个画面。
Audio is a first-class input: WAV, FLAC, MP3, M4A/AAC, OGG and Opus go through `probe`, `cut`, `join`, `silence`, `loudness`, `audio`, `sync` and `check --platform podcast` with the same commands as video. The output extension picks the codec: `-o out.wav` writes PCM, `-o out.flac` FLAC, `-o out.mp3` MP3. Picture tools (`fit`, `caption`, `overlay`, `graphics`, `color`, `export`, `look`) refuse an audio file instead of inventing a picture.

- **`audio.py`**：人声清理三档（`--voice light|medium|strong`）、FFT 降噪、压缩/限幅/噪声门（`--compress`、`--limit`、`--gate`）、音乐床 + 侧链闪避（`--music bed.mp3 --duck`）、5.1 转立体声、换音轨、抽音频（`-o out.wav`）、保调变速（`--tempo`）、定长到精确采样数（`--length-samples`）、重采样、单声道抽取。每个参数都是 FFmpeg 对应滤镜的一个有文档的选项，跑之前先做范围检查，不接受手写滤镜串。
  **audio.py**: voice clean-up chain at three strengths (`--voice light|medium|strong`), FFT denoise, typed compressor / limiter / gate (`--compress`, `--limit`, `--gate`), music bed with sidechain ducking (`--music bed.mp3 --duck`), 5.1 → stereo, track replacement, extraction (`-o out.wav`), a tempo change that keeps the pitch (`--tempo`), an exact length in samples (`--length-samples`), resampling and one channel by index. Each flag is one documented option of FFmpeg's filters, range-checked before ffmpeg runs; no filter string is accepted from the caller.
  ```bash
  python3 $S/audio.py in.mp4 --voice
  python3 $S/audio.py in.mp4 --music bed.mp3 --duck --fade-out 3
  ```
- **`sync.py`**：两段录音按音频互相关对齐（1 毫秒精度，纯 Python），还能修时钟漂移。对齐后的视频或音频输出（音频对音频只出音频，不做唇形同步、也不认脸）。第三台、第四台机器是后面再加的位置参数。修漂移用 `--fix-drift`。
  **sync.py**: offset between two recordings by audio cross-correlation (1 ms, pure Python), clock-drift correction; aligned video or audio out (audio-to-audio only — no lip-sync/face detection). A third or later camera is an additional positional. `--fix-drift` corrects drift.
  ```bash
  python3 $S/sync.py camera.mp4 mic.wav --replace-audio
  python3 $S/sync.py camera.mp4 rec.wav --fix-drift --replace-audio
  ```
- **`loudness.py`**：两遍 EBU R128 `loudnorm`，响到 −14 LUFS / −1 dBTP，或者任意目标。视频流是流拷贝。写完再量一遍，达不到 `--tp` 就重编，直到达标。`--measure-only` 只量不写。`--versions streaming,ebu-r128` 一次出几个版本（流媒体、广播各一份），每个都量过才报 `verified`。
  **loudness.py**: two-pass EBU R128 `loudnorm` to −14 LUFS / −1 dBTP or any target; video stream-copied; the written file is measured again and re-encoded until it meets `--tp`. `--measure-only` measures only. `--versions streaming,ebu-r128` writes one file per target from one master, each measured after.
  ```bash
  python3 $S/loudness.py in.mp4
  python3 $S/loudness.py talk.wav -I -16 --tp -1.5 -o talk.m4a
  ```

### 7. 字幕和画面 / Captions & picture — 4 个

这组跟我平时做的教学视频关系最近。注意一条铁律：字幕文字按给的原文烧，一字不改，哪怕是为了"让它放得下"也不行。
This group is closest to the teaching videos I make every day. One hard rule: the caption text is burned as written, never rewritten or shortened, even to "make it fit".

- **`caption.py`**：烧 SRT/ASS 进去，字体、大小、颜色、描边、位置都能定；也能从带时间戳的纯文本造 SRT。会按安全区换行，`--platform` 就按那个目的地的左右安全区来。`--mode mux` 是把字幕轨封装进文件（能开关，还能一次挂几种语言）。`--animate pop --karaoke` 是动画字幕 + 逐词高亮，词的时间戳可以是语音能量算出来的，也可以是真词级时间戳。`--transcribe` 是先转写再上字幕，要本地语音引擎。`--fit-size` 是把字号缩小到放得下，而不是把句子拆成两行。
  **caption.py**: burn SRT/ASS with font, size, colour, outline, position; build SRT from timed plain text; wraps to the safe area — with `--platform`, the destination's own left/right safe zone. `--mode mux` takes repeated `--srt file:lang` for several language-tagged, toggleable tracks in one file. `--animate pop --karaoke` is animated and word-by-word karaoke timed to the speech energy or real word timings. `--transcribe` transcribes first, then captions, and needs a local speech engine. `--fit-size` shrinks the size until a cue fits instead of splitting the sentence.
  ```bash
  python3 $S/caption.py in.mp4 --srt subs.srt
  python3 $S/caption.py in.mp4 --transcribe --animate pop --karaoke
  python3 $S/caption.py in.mp4 --srt subs.srt --mode mux
  ```
- **`overlay.py`**：logo、水印、标题，位置、时间范围、透明度、淡入淡出都能定。`--platform NAME` 会避开那个目的地的 UI 遮挡区。`--video` 是画中画，`--chromakey` 是绿幕抠像，`--place` 是把一层东西按时间段摆到不同位置。`--keep-alpha` 能在透明画布上合成，写出的文件还带 alpha。
  **overlay.py**: logos, watermarks and titles with position, time range, opacity, fades; `--platform NAME` keeps them clear of that destination's UI; `--video` for picture-in-picture, `--chromakey` for green-screen compositing, `--place` to lay ranges of a layer at chosen times; `--keep-alpha` composites onto a transparent canvas and writes the result with its alpha.
  ```bash
  python3 $S/overlay.py in.mp4 --image logo.png --position top-right --scale 200
  python3 $S/overlay.py in.mp4 --text "Title" --position top --start 0 --end 4
  ```
- **`graphics.py`**：下三分之一、标题卡、章节条、进度条、倒计时、角标、社交贴纸、开场钩子卡、meme 文字，都是 FFmpeg 按品牌 kit 画出来的。`--platform NAME` 保证落在安全区里。`--text-render` 让复杂文字走 libass 排版，`--emoji-assets` 拼彩色 emoji，`--keep-alpha` 在透明画布上按元素出一层 alpha。
  **graphics.py**: lower-thirds, title cards, chapter chips, progress bars, countdowns, corner bugs, social stickers, opening hook cards and meme captions drawn by FFmpeg from a brand kit; `--platform NAME` keeps them inside that destination's safe zone; `--text-render` routes shaping scripts through libass and `--emoji-assets` composites colour emoji; `--keep-alpha` renders a template on a transparent canvas as a per-element alpha layer.
  ```bash
  python3 $S/graphics.py in.mp4 --template lower-third --name "Ada"
  python3 $S/graphics.py in.mp4 --template hook --name "Hook"
  ```
- **`color.py`**：HDR10 / HLG / 杜比视界转 SDR BT.709，DV 层剥离、3D LUT（.cube）、色彩标签重写、曝光/对比/饱和/伽马/白平衡/ lift-gain/levels/curves。`--keep-alpha` 给带 alpha 的片段调色还留着 alpha。`--legalize ebu-r103` 是广播合法范围钳制，写完再量。
  **color.py**: HDR10 / HLG / Dolby Vision → SDR BT.709 tone mapping, DV layer stripping, 3D LUT (.cube), colour-tag rewriting, typed primary correction (exposure/contrast/saturation/gamma/white balance/lift-gain/levels/curves); `--keep-alpha` grades an alpha clip and keeps its alpha; `--legalize` is a legal-range clamp after the 4:2:0 conversion, measured on the written file.
  ```bash
  python3 $S/color.py clip.mov --to-sdr
  python3 $S/color.py in.mp4 --lut grade.cube
  ```

### 8. 交付 / Delivery — 5 个

导出、压小、查合规、出报告、写章节。这组是"交片之前最后一道关"。
Export, fit under a size limit, check compliance, write a report, add chapters. This group is the last gate before handing a file over.

- **`export.py`**：预设一堆：`youtube`、`youtube4k`、`reels`、`tiktok`、`shorts`、`linkedin`、`facebook`、`x`、`youtube-hdr`（HEVC Main10，HDR 标签保留）、`youtube-av1`、`prores`、`dnxhr-lb/sq/hq/hqx/444`、带 alpha 的母版 `prores4444`/`prores4444xq`/`qtrle`/`vp9-alpha`/`hap-alpha`、`key-fill`（切换台的 fill + key 一对）、`h265`、`gif`、`copy`。除 HDR 和剪辑母版外都标 BT.709。`--normalize` 在同一通里把响度也做到平台标准。`--max-mb 25` 是两遍编码压到 25 MB 以下，压完再量。`--preset copy --repair` 是把损坏的文件重新封装（时间戳重建、坏包丢掉），修之前之后各数一遍错误。
  **export.py**: presets `youtube`, `youtube4k`, `reels`, `tiktok`, `shorts`, `linkedin`, `facebook`, `x`, `youtube-hdr` (HEVC Main10, source HDR tags kept), `youtube-av1`, `prores`, `dnxhr-lb`/`sq`/`hq`/`hqx`/`444`, the alpha masters `prores4444`/`prores4444xq`/`qtrle`/`vp9-alpha`/`hap-alpha`, `key-fill` (a switcher's fill + key pair), `h265`, `gif`, `copy`, all tagged BT.709 unless they carry HDR or are an editing master. `--normalize` meets the platform's loudness in the same call; `--max-mb MB` fits the file under a size limit by a two-pass encode and measures it; `--preset copy --repair` remuxes a damaged file and counts the errors before and after.
  ```bash
  python3 $S/export.py in.mp4 --preset youtube
  python3 $S/export.py in.mp4 --preset youtube --max-mb 25
  python3 $S/export.py in.mp4 --preset gif
  ```
- **`proxy.py`**：小码率代理文件，给下游 AI 分析、预览、剪辑决策用。`--width`/`--scale` 缩尺寸，`--quality` 默认 30，`--fps`，`--no-audio`。不是交付预设。
  **proxy.py**: small, low-bitrate proxy for downstream AI analysis/preview/editing decisions — resize by `--width`/`--scale`, proxy-grade `--quality` (default 30), `--fps`, `--no-audio`; not a delivery preset.
  ```bash
  python3 $S/proxy.py in.mp4 --width 640
  ```
- **`check.py`**：按 YouTube、Shorts、Reels、TikTok、X、LinkedIn、Facebook、广播、播客的规格逐项打 PASS / WARN / FAIL，每项不过都写清楚怎么修。长度或大小超了平台上传上限是 WARN（平台会改这个数，上传时以平台当前为准）。`--content` 再加黑帧、冻帧、静音、短暂掉音、单帧闪烁、首帧全平几行。`--compare REF` 拿 SSIM/PSNR 跟参考片比。`--levels ebu-r103` 数超出广播合法范围的亮度色度采样。
  **check.py**: PASS / WARN / FAIL against YouTube, Shorts, Reels, TikTok, X, LinkedIn, Facebook, broadcast and podcast specs, from the same delivery table the export presets and templates read, with the fix for each failure. A length or file size over the table's upload limit is a WARN (platforms change these, the uploader checks the current one). `--content` adds black-frame, frozen-frame, silence, short audio-dropout, single-frame-flash and flat-first-frame rows; `--compare REF` measures SSIM/PSNR against a reference; `--levels` counts the luma and chroma samples outside the broadcast legal range.
  ```bash
  python3 $S/check.py final.mp4 --platform reels
  python3 $S/check.py final.mp4 --compare master.mp4
  ```
- **`report.py`**：单文件 HTML 交片报告：前后对比图、媒体信息、响度、合规、跑过的命令。`--pack` 再出一张社交包表。
  **report.py**: single-file HTML delivery report: before/after sheets, media facts, loudness, compliance, the commands run; `--pack` renders a social pack table.
  ```bash
  python3 $S/report.py --before raw.mov --after final.mp4 -o report.html
  ```
- **`metadata.py`**：按 `TIME TITLE` 文本文件写容器章节标记和标题/作者/备注标签，所有流按比特照拷。`--auto-chapters` 按测出来的停顿和场景切换提议章节，先叫 `Chapter N`，名字自己改。
  **metadata.py**: write container chapter markers from a `TIME TITLE` text file and title/artist/comment tags, every stream copied bit for bit; `--auto-chapters` proposes the markers from measured pauses and scene cuts and names them `Chapter N` for you to rename.
  ```bash
  python3 $S/metadata.py ep.mp4 --chapters chapters.txt
  ```

### 9. 整活 / Orchestration — 4 个

上面都是单步工具，这组是把很多步串成一个活：整个剪辑项目、批量文件夹、多机位、还有在你真素材上跑测试。
The groups above are single steps; this group strings many steps into a whole job: a full edit project, a batch folder, multiple cameras, and tests on your real footage.

- **`render.py`**：按一个声明式 `project.json` 渲整个剪辑（片段、转场、字幕、叠加、音乐和分轨音量、响度、导出、章节标记、检查）。`--init` 写一个项目模板出来。`--dry-run` 只出计划，`--stop-after` 停在某一步，`--cache DIR` / `--from STAGE` 复用没变过的步骤。`--template NAME INPUT` 是直接渲一个现成的交付模板（`--template all` 一次出整个社交包加表）。`--export-timeline edit.fcpxml|.edl|.otio` 是把剪辑写成 Final Cut、DaVinci Resolve、Premiere 能打开的时间线，不渲成片。
  **render.py**: render a whole edit from a declarative `project.json` (clips, transitions, captions, overlays, music and stem levels, loudness, export, chapter markers, check); `--init` writes one; `--dry-run`, `--stop-after`, `--cache DIR`/`--from STAGE` (reuse the stages that did not change); `--template NAME INPUT` renders a shipped delivery template (`--template all` writes the whole social pack plus its table); `--export-timeline edit.fcpxml|.edl|.otio` writes the cut as an editor timeline for Final Cut, DaVinci Resolve or Premiere instead of rendering it.
  ```bash
  python3 $S/render.py --template reels in.mp4 --cues cues.txt
  python3 $S/render.py --template all in.mp4 --cues cues.txt
  python3 $S/render.py project.json
  ```
- **`batch.py`**：把一个步骤配方或者一个项目用到一整个文件夹，内容哈希缓存，`--watch` 守着文件夹，`--jobs N` 多核并行。写文件的步骤必须在配方里写 `{out}` 或者自己的 `-o`，不然直接拒绝。
  **batch.py**: apply a step recipe or a project to a folder with a content-hash cache; `--watch`; `--jobs N` processes several files at once under one shared timeout. A step that writes must name `{out}` or its own `-o`/`--output`.
  ```bash
  python3 $S/batch.py FOLDER --recipe batch.json --jobs auto
  ```
- **`multicam.py`**：按音频对齐任意多台机位和录音机（带漂移修正），再按切换清单在机位之间切。`--switch energy` 是自动切到当前最响的那台（用音量代替谁在说话：没有说话人识别，所以每位说话人得在自己机位的麦上）。`--min-shot` 是最短镜头长，`--edl` 写一个纯起止时间清单。
  **multicam.py**: align any number of cameras and recorders by audio (with drift correction) and cut between them from a switch list; `--switch energy` auto-cuts to whichever camera is loudest at that moment (audio level as a stand-in for who is talking: there is no speaker identification, so it needs each speaker on their own camera's mic), with `--min-shot` (minimum shot length) and `--edl` (write a plain START-END cut list).
  ```bash
  python3 $S/multicam.py a.mp4 b.mp4 c.mp4 --switch energy
  ```
- **`verify.py`**：在你真设备文件上跑一遍工具链，逐项报 PASS / FAIL。装完想在真素材上试一把，用它。
  **verify.py**: run the toolchain on real device files and report PASS / FAIL per step. Use it to test the tools on your real files after installing.
  ```bash
  python3 $S/verify.py ~/Footage --report verify.md
  ```

---

## 让 AI 帮你做 / Using it with AI

装好之后不用记命令，直接跟 agent 说人话。它按 SKILL.md 里那张 request→script 表自己挑工具。README 里的原例子：
Once installed, you don't memorise commands — just tell the agent in plain words. It picks the tools itself from SKILL.md's request→script table. The README's own example:

> "Take `interview.mp4`, keep 0:45–1:20 and 5:00–5:25, and make it a 60-second Reel."

agent 会跑 `probe.py`、`cut.py --segments 0:45-1:20,5:00-5:25 --accurate`、`fit.py --aspect 9:16 --fit crop`、`export.py --preset reels`、`check.py --platform reels` 和 `look.py`，然后报 "final.mp4: 60.00 s, 1080×1920, 30 fps, AAC stereo"，附上它看过的拼图。
The agent runs `probe.py`, `cut.py --segments 0:45-1:20,5:00-5:25 --accurate`, `fit.py --aspect 9:16 --fit crop`, `export.py --preset reels`, `check.py --platform reels` and `look.py`, then reports "final.mp4: 60.00 s, 1080×1920, 30 fps, AAC stereo" with the contact sheet it inspected.

再给几个能直接用的说法：
A few more prompts you can use as-is:

> "把 `talk.mp4` 转成 9:16，留 0:30 到 1:15，加 `subs.srt` 的字幕，响度做到 −14 LUFS，按 reels 导出，导出完检查一遍。"
> "Turn `talk.mp4` into 9:16, keep 0:30 to 1:15, add the captions from `subs.srt`, bring the loudness to −14 LUFS, export for reels, and check it afterwards."

> "这段录音里把停顿都剪掉，`--margin` 留 0.3 秒，剪完告诉我省了多少秒。"
> "Cut the pauses out of this recording, keep a 0.3 s margin, and tell me how many seconds it saved."

> "把这个文件夹里的视频全转成 Shorts 的 9:16，每一个都检查一遍，列一张表给我。"
> "Turn every video in this folder into 9:16 Shorts, check each one, and give me a table."

> "给我出一个 3 秒的标题卡，1920x1080，黑底白字，写 'Hello'，再加一个 Ken Burns 慢推。"
> "Make me a 3-second title card, 1920x1080, white text on black, saying 'Hello', with a slow Ken Burns push-in."

agent 回你的格式也是定的（SKILL.md 里的 report 格式）：先说做完了什么、数字是多少，再说走了哪几步，检查过了没有，看过的拼图是哪张，备注里写它发现的问题。数字全来自 `--json` 或者 probe，不是它随口说的。
The agent's reply format is fixed too (SKILL.md's report format): what it finished and the numbers first, then the steps, whether the checks passed, which contact sheet it looked at, and any issues in the notes. The numbers come from `--json` or a probe, never made up.

---

## 在我自己的视频流程里能帮上什么 / How it fits my own video workflow

我平时做的竖屏 9:16 教学视频，这套工具能接上好几处：
For the vertical 9:16 teaching videos I make every day, this skill plugs into several spots:

- **剪片段**：源视频先 `probe.py` 量一遍，再 `cut.py` 按时间点剪。能无损剪就无损剪，画质不掉。
  **Cutting clips**: `probe.py` measures the source first, then `cut.py` cuts by timestamps. Lossless where possible, so quality doesn't drop.
- **改竖屏**：`fit.py --aspect 9:16 --fit crop` 一句话转竖屏。注意：16:9 裁成 9:16 会丢掉 70% 的宽度，`crop_check` 会提醒裁掉的画面里有没有丢东西。横屏主体不在中间的时候，先拿 `cropdetect.py --motion-centre` 看运动重心在哪，再定裁哪边。
  **Going vertical**: `fit.py --aspect 9:16 --fit crop` turns it vertical in one command. Note: cropping 16:9 to 9:16 drops 70% of the width, and `crop_check` flags discarded motion or detail. When the subject isn't centred, check `cropdetect.py --motion-centre` for where the motion is before choosing the crop.
- **加字幕**：中英双语字幕，先写好 SRT 再 `caption.py --srt` 烧进去。字幕文字按给的原文烧，一字不改。想让词一个一个变色，用 `--animate pop --karaoke`。想留可开关的字幕轨，用 `--mode mux`。
  **Captions**: write the bilingual SRT first, then burn it in with `caption.py --srt`. The text is burned as written, word for word. For word-by-word highlighting, use `--animate pop --karaoke`. For a toggleable subtitle track, use `--mode mux`.
- **结尾卡片**：`insert.py` 做一张标题卡，或者 `freeze.py` 把最后一帧定住几秒。单词卡那 5 秒，`freeze.py --hold 5` 就能定住。
  **End card**: `insert.py` makes a title card, or `freeze.py` holds the last frame for a few seconds. A 5-second word card is just `freeze.py --hold 5`.
- **响度**：`loudness.py` 一把梭到 −14 LUFS，短视频、播客各有各的标准，`--versions` 一次出几个版本。
  **Loudness**: `loudness.py` brings it to −14 LUFS in one go; short video and podcast each have their own target, and `--versions` writes several at once.
- **导出和检查**：`export.py --preset reels`（或者 `shorts`、`tiktok`）导出，再 `check.py --platform reels` 逐项打勾，`look.py` 出拼图看一眼。交片之前这三步走完，心里有数。
  **Export and check**: `export.py --preset reels` (or `shorts`, `tiktok`) to export, then `check.py --platform reels` to tick each row, then `look.py` for a contact sheet to look at. Run all three before handing a file over.
- **批量**：一堆视频要出同一个规格，`batch.py` 一个文件夹全跑一遍，不用一个一个点。
  **Batch**: a pile of videos that all need the same spec — `batch.py` runs the whole folder in one go instead of clicking through them one by one.

它干不了的几件事，也写清楚：它不替你决定哪刀该剪、不写字幕文字、不评判好不好看。它只量、只报、只执行。
And what it can't do, stated plainly: it doesn't decide which cut is right, doesn't write caption text, doesn't judge what looks good. It only measures, reports and executes.

---

## 注意事项 / Gotchas

SKILL.md 里列了一串"看起来对、其实错"的事，挑几条最容易踩的：
SKILL.md lists a set of "looks right but is wrong" cases. The ones easiest to trip over:

- **先裁后加字幕会跑偏**：字幕在裁剪之前烧，裁完字幕可能落在框外。顺序是：裁 → 再加字幕。
  **Captions before a crop land off-frame**: burn captions before cropping and they can end up outside the frame. Order: crop first, then caption.
- **16:9 裁 9:16 丢 70% 宽度**：`--fit crop` 从 16:9 到 9:16 会丢掉 70% 的宽度。主体不在中间，先看 `cropdetect.py --motion-centre`。
  **`--fit crop` 16:9→9:16 drops 70% of width**: cropping 16:9 to 9:16 throws away 70% of the width. If the subject isn't centred, check `cropdetect.py --motion-centre` first.
- **平台 UI 会盖住画面**：TikTok、Reels 的 UI 盖住底部和右边。`caption.py`、`overlay.py`、`graphics.py` 的 `--platform NAME` 就是用来避开这些区的；`look.py --safe NAME` 能在拼图上把被盖的区涂出来给你看。
  **TikTok/Reels UI covers the bottom and right**: TikTok and Reels UI covers the bottom and right of the frame. The `--platform NAME` flag on `caption.py`, `overlay.py` and `graphics.py` keeps things clear of those zones; `look.py --safe NAME` shades them on the contact sheet so you can see.
- **HDR 转 SDR 会发灰**：HDR 素材走 SDR 路径重编码会变平。走 `color.py --to-sdr` 做 tone mapping。
  **HDR re-encoded through an SDR path goes flat**: HDR footage re-encoded through an SDR path goes flat. Use `color.py --to-sdr` for tone mapping.
- **重编码会丢 alpha**：除非加 `--keep-alpha`。带透明通道的片段调色、转格式都要记得加。
  **Re-encodes drop alpha unless `--keep-alpha`**: colour or format work on an alpha clip needs `--keep-alpha` or the transparency is gone.
- **`-c copy` 的剪切可能从错帧开始**：无损剪切从关键帧起，起点可能往后挪。`--accurate` 或者 `--smart` 才帧精确。
  **A `-c copy` cut can start on a wrong frame**: a lossless cut starts at a keyframe, so the start can shift later. `--accurate` or `--smart` is frame-exact.
- **可变帧率的手机/录屏素材**：重编码时会转成恒定帧率，`--vfr-guard` 管这个检查。
  **VFR phone/screen clips go CFR on re-encode**: variable-frame-rate clips are conformed to constant frame rate on re-encode; `--vfr-guard` controls that check.
- **原片保护**：目标路径撞上输入文件，工具直接拒绝；要换一个已有的输出文件，必须加 `--overwrite`。这是故意设计的，不是 bug。
  **Source protection**: a destination that matches a declared input is refused, and replacing an existing output needs `--overwrite`. That's by design, not a bug.

---

## 没核实到的 / Not verified

- 上面所有命令都是从仓库的 README 和 SKILL.md 原文里核对的，**我自己没在机器上跑过**。`npx ffmpeg-skill`、`doctor`、`probe.py` 这些，我没装、没试。
  All commands above were checked against the wording of the repo's README and SKILL.md. **I did not run any of them on a machine myself.** `npx ffmpeg-skill`, `doctor`, `probe.py` — none of them were installed or tried by me.
- macOS 那条 `brew install ffmpeg-full` 和 PATH 的办法是 README 写的，我没在 Mac 上试过。Windows 的 `winget install Gyan.FFmpeg` 也一样，没试过。
  The macOS `brew install ffmpeg-full` and PATH steps are from the README; I didn't try them on a Mac. Same for the Windows `winget install Gyan.FFmpeg` line.
- MCP 配置里的路径 `/Users/you/.claude/skills/ffmpeg-skill/mcp/server.py` 是 README 的例子，`you` 要换成你自己的用户名和实际安装位置。
  The MCP config path `/Users/you/.claude/skills/ffmpeg-skill/mcp/server.py` is the README's example — `you` has to be replaced with your own username and actual install location.
- `doctor` 在你机器上会报什么，我没法替你说。装完自己跑一遍 `npx ffmpeg-skill doctor` 看结果。
  What `doctor` reports on your machine, I can't say for you. Run `npx ffmpeg-skill doctor` yourself after installing and read the result.

---

## 链接 / Links

- https://github.com/kajisho5/ffmpeg-skill
- npm：https://www.npmjs.com/package/ffmpeg-skill
- 网站：https://kajisho5.github.io/ffmpeg-skill/
