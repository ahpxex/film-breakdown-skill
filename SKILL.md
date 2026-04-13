---
name: film-breakdown
description: Structured film/video breakdown analysis. Routes to genre-specific frameworks, outputs opinionated segment-by-segment analysis. Use when the user says "breakdown", "analyze this film", "拉片", or asks about cinematography, narrative structure, or audiovisual craft.
---

# Film Breakdown

## Overview

Structured breakdown analysis for films, series, anime, and videos (tech videos, vlogs, video essays, etc.).

Core capabilities:
1. Genre routing: load analysis frameworks by genre
2. Structured analysis: opinionated segment-by-segment output following framework skeleton
3. Video processing (optional): scene detection, keyframe extraction, subtitle parsing/transcription
4. **HTML report generation (default)**: the final deliverable of every analysis is a self-contained HTML file, not plain text

**Important: after every analysis, you MUST generate an HTML report file.** This is the default output format of this skill -- the user does not need to ask for it. Write the analysis as markdown, then immediately call `generate_report.mjs` to produce HTML, then open it in the browser with `open`.

## Script location

All scripts are in the `scripts/` directory **relative to this SKILL.md file**, not relative to the user's project. Before running any script, locate this skill's installation directory:

```bash
# Find the skill directory (where this SKILL.md lives)
SKILL_DIR="$(dirname "$(find . ~/.claude -path '*/film-breakdown*' -name 'SKILL.md' 2>/dev/null | head -1)")"
```

Then run scripts as: `node "$SKILL_DIR"/scripts/generate_report.mjs ...`

**Do NOT write your own HTML. Always use the generate_report.mjs script.** If the script cannot be found, tell the user the skill may not be installed correctly and ask them to reinstall with `npx skills add ahpxex/film-breakdown-skill`.

## When to use

- User names a work and asks for breakdown/analysis
- User asks about cinematography, narrative structure, audiovisual craft
- User wants to understand why a video is effective or why a film works
- User compares techniques between two works

## Three input modes

Different paths based on what the user has. No mode is "degraded" -- each is a complete analysis path.

### Mode A: Conversation analysis (most common)

User has seen the work and wants to discuss/analyze it. No local files.

**Input**: Work title + user description/memory/questions
**What to do**:
1. Confirm genre, load frameworks
2. Analyze using agent's knowledge + user-provided details, following framework structure
3. Reference specific scenes, segments, timestamps (based on shared knowledge)
4. Write analysis as markdown, generate HTML report with `generate_report.mjs` (text-only layout when no keyframes)

This is the most natural scenario. Most cinephiles/creators discuss works from memory, not while scrubbing through files.

### Mode B: Image-assisted analysis

User provides screenshots, keyframes, stills, or subtitle files.

**Input**: Image file paths / subtitle files + work info
**What to do**:
1. View images with Read tool (Claude supports multimodal)
2. Parse subtitle files with `extract_subtitles.mjs` if provided
3. Combine visual + text information, analyze following framework
4. Write analysis as markdown (referencing user-provided images), generate HTML report with `generate_report.mjs`

### Mode C: Full video processing

User provides a local video file or URL. Most processing, most precise analysis.

**Input**: Local video file path, or URL (requires yt-dlp installed)
**What to do**:

If input is a URL, download first:
```bash
yt-dlp -f 'bestvideo[height<=720]+bestaudio/best[height<=720]' \
  --merge-output-format mp4 -o '/tmp/film-breakdown/<name>/video.%(ext)s' '<URL>'
```
- Cap at 720p -- keyframe analysis doesn't need high resolution; 4K only bloats keyframes and reports
- Download subtitles if available: `yt-dlp --write-sub --write-auto-sub --sub-lang zh-Hans,zh-CN,en --sub-format srt --skip-download`

Then follow the video processing workflow below.

## Video processing workflow (Mode C only)

### Step 1: Extract structured data

The following steps can run in parallel:

```bash
# 1a. Scene detection + keyframe extraction
node "$SKILL_DIR"/scripts/extract_scenes.mjs \
  --input /path/to/video.mp4 \
  --output /tmp/film-breakdown/scenes/ \
  --threshold 0.3

# 1b. Subtitle extraction
node "$SKILL_DIR"/scripts/extract_subtitles.mjs \
  --input /path/to/video.mp4 \
  --srt /path/to/subtitle.srt

# 1c. Audio transcription (only when no subtitles)
node "$SKILL_DIR"/scripts/transcribe.mjs \
  --input /path/to/video.mp4 \
  --output /tmp/film-breakdown/transcript.json \
  --model base
```

### Step 1.5: Compress keyframes (if needed)

If keyframes come from high-resolution sources or there are too many, compress to control report size:
```bash
# Scale keyframes to 1280px width, quality 80
for f in /tmp/film-breakdown/scenes/keyframes/*.jpg; do
  ffmpeg -i "$f" -vf "scale=1280:-1" -q:v 4 -y "$f" 2>/dev/null
done
```
Target: single keyframe < 150KB, total report < 3MB.

### Step 2: Build unified timeline

```bash
node "$SKILL_DIR"/scripts/build_timeline.mjs \
  --scenes /tmp/film-breakdown/scenes/scenes.json \
  --subtitles /tmp/film-breakdown/scenes/subtitles.json \
  --output /tmp/film-breakdown/timeline.json
```

Timeline schema: see `references/schema.md`.

### Step 3: View keyframes + analyze with framework

View extracted keyframes with the Read tool, combine with timeline data for segment-by-segment analysis.

### Step 4: Generate static report

Write analysis as a markdown file (using standard `![alt](path)` syntax to reference keyframes), then generate self-contained HTML report:

```bash
node "$SKILL_DIR"/scripts/generate_report.mjs \
  --analysis /tmp/film-breakdown/analysis.md \
  --keyframes /tmp/film-breakdown/scenes/keyframes \
  --output /tmp/film-breakdown/report.html
```

Report features:
- All keyframes embedded as base64, single file readable offline
- Auto-detects content language: CJK content gets serif CJK typography; Latin content gets Georgia serif
- Print-ready (`@media print` optimized)

## Genre routing

1. User declares genre, or agent determines from content
2. Always load `references/frameworks/_base.md`
3. Load matching genre framework(s) (can combine multiple)

### Narrative (film/series/anime)

| Genre | Framework file |
|-------|---------------|
| Sci-fi | `sci-fi.md` |
| Fantasy | `fantasy.md` |
| Horror/Thriller | `horror-thriller.md` |
| Mystery/Detective | `mystery.md` |
| Crime/Gangster | `crime.md` |
| Action/Adventure | `action-adventure.md` |
| War | `war.md` |
| Romance | `romance.md` |
| Comedy | `comedy.md` |
| Art house/Auteur | `art-house.md` |
| Historical/Biopic | `historical-biopic.md` |
| Wuxia/Martial arts | `wuxia-martial-arts.md` |
| Cyberpunk | `cyberpunk.md` |
| Post-apocalyptic | `post-apocalyptic.md` |
| Psychological | `psychological.md` |
| Documentary | `documentary.md` |
| Musical | `musical.md` |

### Video (non-fiction/non-traditional narrative)

| Genre | Framework file |
|-------|---------------|
| Tech video | `tech-video.md` |
| Vlog | `vlog.md` |
| Video essay | `video-essay.md` |
| Product review | `product-review.md` |
| Tutorial | `tutorial.md` |
| News/Commentary | `news-commentary.md` |

Genres can be combined. e.g. "cyberpunk mystery" loads `cyberpunk.md` + `mystery.md`.

## Analysis output

Analysis must have depth. A good breakdown report should make the reader feel "I watched this many times and never noticed that."

### Narrative works

1. **Overview** -- genre, duration, structure type
2. **Overall judgment** -- macro-level assessment based on framework, lead with the conclusion
3. **Segment analysis** -- by scene/segment/act, each containing:
   - Timecode or position marker (precise to seconds with files, narrative position in conversation mode)
   - Keyframe reference (if available)
   - Analysis across framework dimensions for that segment
4. **Specialized analysis** -- beyond segment analysis, must cover `_base.md` core dimensions:
   - Cinematography (shot scale choices, camera movement, composition patterns)
   - Editing (rhythm, transitions, time manipulation)
   - Sound design (score, ambient sound, use of silence)
   - Visual design (color system, lighting, production design)
   - Not every dimension gets equal weight -- expand most on whichever dimension is the work's strongest craft
5. **Core techniques** -- techniques worth learning, creator's signature methods

### Video works

On top of narrative analysis, additionally output:
- Why it works (propagation mechanism analysis)
- Reusable structural patterns
- Video works also need visual/editing/sound dimension coverage -- excellent tech videos and vlogs are no less crafted than narrative works

### Report depth

Users can specify analysis depth and length. If unspecified, use these defaults:

| Depth | Use case | Word count | Keyframe density |
|-------|----------|-----------|-----------------|
| Quick | Quick overview of core techniques | 1000-2000 words | 3-5 frames |
| Standard (default) | Full breakdown covering major dimensions | 4000-6000 words | 1 per 2-3 min |
| Deep | Scene-by-scene deep analysis for craft study | 8000-12000 words | 1 per minute or denser |

Scale by content length:
- Under 5 minutes: above word counts x 0.5
- 5-30 minutes: above word counts x 1
- Feature film: above word counts x 1.5

Use Quick when user says "quick look", "brief analysis"; use Deep when user says "deep dive", "scene by scene"; use Standard when unspecified.

## Dependencies

Mode A (conversation) has no external dependencies. Mode B/C requires:

- `ffmpeg` -- video processing, scene detection, keyframe extraction
- `whisper` -- audio transcription (optional, only when no subtitles; supports whisper.cpp or openai-whisper)

Scripts check for dependencies on startup and suggest installation rather than crashing.
Mode C falls back to Mode B (subtitles only) or Mode A (conversation) if ffmpeg is unavailable.

## Conventions

- Analysis must have opinions, not just list observations without conclusions
- Narrative works: focus on "why is it good" (craft); video works: focus on "why is it effective" (propagation mechanism)
- No academic jargon pileup -- use clear language to explain the causal relationship between technique and effect
- Reference specific timecodes and frames, don't generalize
- When combining genres, prioritize analyzing unique effects produced by genre intersection
