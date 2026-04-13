# 拉片.SKILL / Film Breakdown Skill

对电影、剧集、动漫或视频进行结构化拉片分析，输出带关键帧的静态 HTML 报告。

Structured film/video breakdown analysis for Claude Code. Outputs self-contained HTML reports with embedded keyframes.

---

## 它做什么 / What it does

给一部作品（电影、剧集、YouTube 视频、tech video、vlog...），按题材自动加载对应的分析框架，从镜头语言、剪辑、声音设计、视觉设计、叙事结构等维度进行拉片，生成一份图文并排的静态 HTML 报告。

Given a film, episode, YouTube video, tech video, vlog, etc., it automatically loads genre-specific analysis frameworks and performs a structured breakdown across cinematography, editing, sound design, visual design, and narrative dimensions. Outputs a self-contained HTML report with inline keyframes.

## 三种输入模式 / Three input modes

| 模式 / Mode | 输入 / Input | 说明 / Description |
|---|---|---|
| A. 对话 / Conversation | 作品名称 | 最常见。基于共同知识按框架分析。Most common. Analysis based on shared knowledge. |
| B. 图片辅助 / Image-assisted | 截图、关键帧、字幕文件 | Claude 看图 + 文本辅助分析。Visual + text analysis. |
| C. 完整视频 / Full video | 本地视频文件或 URL | 自动切割场景、提取关键帧、解析字幕，完整流水线。Full pipeline: scene detection, keyframe extraction, subtitle parsing. |

## 三档粒度 / Three depth levels

| 粒度 / Depth | 字数参考 / Word count | 说明 / Description |
|---|---|---|
| 速览 / Quick | 1000-2000 字 | "简单看看" / Quick overview |
| 标准 / Standard (default) | 4000-6000 字 | 完整拉片 / Full breakdown |
| 精读 / Deep | 8000-12000 字 | 逐场景深度拆解 / Scene-by-scene deep analysis |

## 支持的题材 / Supported genres

**叙事影视类 / Narrative:**
科幻、奇幻、恐怖/惊悚、悬疑、犯罪、动作/冒险、战争、爱情、喜剧、文艺/作者、历史/传记、武侠、赛博朋克、末日、心理、纪录片、音乐剧

**视频类 / Video:**
Tech 视频、Vlog、Video Essay、产品评测、教程、新闻/时事评论

题材可组合。比如"赛博朋克悬疑片"会同时加载两个框架。
Genres can be combined. e.g. "cyberpunk mystery" loads both frameworks.

## 报告样式 / Report style

生成的 HTML 报告是单文件、自包含的（关键帧以 base64 嵌入），可以离线阅读、打印。

排版风格：暖纸底色、宋体正文、图片出血、大量留白。东方美学，阅读化。

Generated HTML reports are single-file, self-contained (keyframes embedded as base64). Readable offline, print-ready. Eastern aesthetic typography: warm paper, serif CJK body text, image bleed, generous whitespace.

## 安装 / Installation

```bash
npx skills add AHpxDE/film-breakdown-skill
```

## 系统依赖 / System dependencies

模式 A（对话）不需要任何外部依赖。模式 B/C 需要：

Mode A (conversation) requires no external dependencies. Mode B/C requires:

- **ffmpeg** -- 场景检测、关键帧提取 / Scene detection, keyframe extraction
  ```bash
  brew install ffmpeg
  ```
- **yt-dlp** (可选 / optional) -- 从 URL 下载视频 / Download video from URL
  ```bash
  brew install yt-dlp
  ```
- **whisper** (可选 / optional) -- 音频转写，仅无字幕时需要 / Audio transcription, only when no subtitles available
  ```bash
  pip install openai-whisper
  ```

## 使用 / Usage

安装后直接对 Claude Code 说：

After installation, just tell Claude Code:

- "帮我拉一下星际牛仔最后一集"
- "分析这个视频 https://youtube.com/watch?v=..."
- "breakdown /path/to/video.mp4，精读级别"
- "这部电影的镜头语言怎么样"

## 项目结构 / Structure

```
SKILL.md                          # Skill 定义和工作流
references/
  schema.md                       # 时间轴 JSON schema
  frameworks/
    _base.md                      # 基础分析框架（7 个通用维度）
    sci-fi.md, crime.md, ...      # 23 个题材特化框架
scripts/
  extract_scenes.mjs              # 场景切割 + 关键帧提取
  extract_subtitles.mjs           # 字幕解析（SRT/ASS/VTT + 内嵌）
  transcribe.mjs                  # Whisper 音频转写
  build_timeline.mjs              # 统一时间轴构建
  generate_report.mjs             # HTML 报告生成
```

## License

MIT
