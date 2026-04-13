# Film Breakdown Skill

Structured film/video breakdown analysis for Claude Code. Outputs self-contained HTML reports with embedded keyframes.

[中文版](https://github.com/ahpxex/lapian.skill)

---

## What it does

Given a film, episode, YouTube video, tech video, vlog, etc., it automatically loads genre-specific analysis frameworks and performs a structured breakdown across cinematography, editing, sound design, visual design, and narrative dimensions. Outputs a self-contained HTML report with inline keyframes.

## Installation

```bash
npx skills add ahpxex/film-breakdown-skill
```

## Three input modes

| Mode | Input | Description |
|---|---|---|
| A. Conversation | Work title | Most common. Analysis based on shared knowledge. |
| B. Image-assisted | Screenshots, keyframes, subtitle files | Visual + text analysis. |
| C. Full video | Local video file or URL | Full pipeline: scene detection, keyframe extraction, subtitle parsing. |

## Three depth levels

| Depth | Word count | Description |
|---|---|---|
| Quick | 1000-2000 words | Quick overview of core techniques |
| Standard (default) | 4000-6000 words | Full breakdown covering major dimensions |
| Deep | 8000-12000 words | Scene-by-scene deep analysis for craft study |

## Supported genres

**Narrative (film/series/anime):**
Sci-fi, Fantasy, Horror/Thriller, Mystery, Crime, Action/Adventure, War, Romance, Comedy, Art house/Auteur, Historical/Biopic, Wuxia/Martial arts, Cyberpunk, Post-apocalyptic, Psychological, Documentary, Musical

**Video (non-fiction):**
Tech video, Vlog, Video essay, Product review, Tutorial, News/Commentary

Genres can be combined -- e.g. "cyberpunk mystery" loads both frameworks.

## Report style

Generated HTML reports are single-file, self-contained (keyframes embedded as base64). Readable offline, print-ready.

Typography auto-detects content language:
- **CJK content**: Serif CJK fonts, warm paper background, generous whitespace, eastern aesthetic
- **Latin content**: Georgia serif, slightly larger font size, tighter line height

## System dependencies

Mode A (conversation) requires no external dependencies. Mode B/C requires:

- **ffmpeg** -- Scene detection, keyframe extraction
  ```bash
  brew install ffmpeg
  ```
- **yt-dlp** (optional) -- Download video from URL
  ```bash
  brew install yt-dlp
  ```
- **whisper** (optional) -- Audio transcription, only when no subtitles available
  ```bash
  pip install openai-whisper
  ```

## Usage

After installation, just tell Claude Code:

- "breakdown the last episode of Cowboy Bebop"
- "analyze this video https://youtube.com/watch?v=..."
- "breakdown /path/to/video.mp4, deep level"
- "what's the cinematography like in this film"

## Structure

```
SKILL.md                          # Skill definition (English)
references/
  schema.md                       # Timeline JSON schema
  frameworks/
    _base.md                      # Base analysis framework (7 universal dimensions)
    sci-fi.md, crime.md, ...      # 23 genre-specific frameworks
scripts/
  extract_scenes.mjs              # Scene detection + keyframe extraction
  extract_subtitles.mjs           # Subtitle parsing (SRT/ASS/VTT + embedded)
  transcribe.mjs                  # Whisper audio transcription
  build_timeline.mjs              # Unified timeline builder
  generate_report.mjs             # HTML report generator
```

## License

MIT
