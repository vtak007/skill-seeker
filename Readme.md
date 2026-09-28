# Skill Seeker

A Claude Code skill that builds new skills from knowledge sources: documentation sites, GitHub repos, local codebases, PDFs / Office docs / EPUBs, Jupyter notebooks, OpenAPI specs and YouTube videos.

It is a hybrid engine: the [`skill-seekers`](https://pypi.org/project/skill-seekers/) CLI gathers content from large or awkward sources, and Claude does the analysis and writes a lean `SKILL.md` + `references/`, which it installs to `~/.claude/skills/<name>/`.

## Usage
In Claude Code: `make a skill from <URL | owner/repo | path>` (or `/skill-seeker <source>`).

Workflow: intake → detect & plan → gather → synthesize → validate → review → install (on approval).

## Requirements
- Claude Code
- Optional: `pip install "skill-seekers[all]"` (v3.9.1 tested) for large docs sites, big repos and video
- Video: `yt-dlp` and `youtube-transcript-api` (captions); `faster-whisper` + ffmpeg only for uncaptioned videos

## Install / update
This folder is the source of truth. Copy it to the live location:
```bash
cp -r skill-seeker/. ~/.claude/skills/skill-seeker/
```

## Windows notes
- CLI calls are prefixed with `PYTHONIOENCODING=utf-8 PYTHONUTF8=1` (prevents `'charmap' codec` crashes).
- CLI always runs with `--enhance-level 0` — Claude does the synthesis itself.
- Pass source URLs positionally to `skill-seekers create` (e.g. `create --video-url <url>` alone fails with "No source provided").

## Tested
- 2026-09-28: YouTube video → `claude-artifacts-finance` skill (captions path).
