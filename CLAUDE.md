# CLAUDE.md

Source for the `skill-seeker` Claude skill: builds new skills from docs sites, GitHub repos, local code, PDFs/Office docs, notebooks, OpenAPI specs and videos. Hybrid engine — uses the `skill-seekers` pip CLI (v3.9.1 installed) for heavy scraping, native tools otherwise.

## Key Files

| File | Purpose |
|---|---|
| `skill-seeker/SKILL.md` | Skill entry point: intake → detect → gather → synthesize → validate → install workflow |
| `skill-seeker/references/source-playbook.md` | Per-source detection, CLI flags, native extraction checklists |
| `skill-seeker/references/skill-template.md` | Template + description-writing guide for generated skills |
| `Readme.md` | Overview, requirements, install/update, Windows notes, test log |

## Notes
- This folder is the source of truth; the live copy is `~/.claude/skills/skill-seeker/`. After editing, re-sync: `cp -r skill-seeker/. ~/.claude/skills/skill-seeker/`
- Generated skills install globally to `~/.claude/skills/<name>/`.
- The CLI is always run with `--enhance-level 0` — its LOCAL enhancement mode spawns a nested AI agent; Claude does synthesis itself.
- `skill-seekers create` needs the source as a positional arg; type flags like `--video-url` don't replace it.
- No git remote yet — commits are local only.
