---
name: skill-seeker
description: Build a new Claude skill from any knowledge source — documentation websites, GitHub repos (owner/repo or URL), local codebases, PDFs, Word/PowerPoint/EPUB files, Jupyter notebooks, OpenAPI specs, YouTube videos, or several of these combined. Use when the user says "make/create/build a skill from <URL/repo/file/folder>", "turn these docs into a skill", "skill-seekers", or points at a source and asks for a skill about it. Hybrid engine - uses the skill-seekers CLI for heavy scraping when installed, native tools otherwise - and installs the result globally to ~/.claude/skills/<name>/.
---

# Skill Seeker

Turn one or more sources into a lean, well-triggered Claude skill: `SKILL.md` + `references/*.md`, installed to `~/.claude/skills/<name>/`.

**Division of labour:** the `skill-seekers` CLI (pip, v3.x) *gathers* raw content from large or awkward sources. **You** do the analysis, synthesis and writing — never let the CLI's raw output be the final skill.

## Workflow

### 1. Intake
Collect from the user's message (ask only for what's missing):
- **Sources** — one or more URLs, `owner/repo`, paths, files.
- **Skill name** — kebab-case, derived from the subject (e.g. `fastapi`, `acme-billing-api`). Propose one; don't block on it.
- **Focus** (optional) — e.g. "just the REST API", "v5 only", "for writing mutators". Narrow scope beats exhaustive dumps.

If `~/.claude/skills/<name>/` already exists, stop and ask: overwrite, merge/update, or pick a new name.

### 2. Detect & plan
Classify each source and pick an engine using the table in `references/source-playbook.md`. Rule of thumb:

| Source | Engine |
|---|---|
| Single page / short docs (< ~15 pages) | Native (WebFetch) |
| Large docs site | CLI |
| GitHub repo | CLI for big repos; native (`gh`/WebFetch of README, docs/, examples/, tests/) for small ones |
| Local codebase | Native (Glob/Grep/Read) — or CLI `--directory` for large ones |
| PDF / DOCX / PPTX / EPUB / IPYNB / OpenAPI | Native Read first; CLI if Read can't handle it (scanned PDF → `--ocr`, huge files) |
| YouTube / video | CLI only (needs `skill-seekers[all]`); if unavailable, tell the user |

Check CLI availability once: `skill-seekers --version` (fallback: `python -m skill_seekers --version`). If absent, use native for everything and mention `pip install skill-seekers` only if a source truly needs it.

Tell the user the plan in 2–4 lines (sources → engine, estimated effort) before any long-running scrape.

### 3. Gather
Work in a staging dir: the session scratchpad if available, else `./skill-seeker-work/<name>/`.

**CLI path** — two rules for every `skill-seekers` command (including `doctor`, `estimate`, `quality`):
- **Prefix with `PYTHONIOENCODING=utf-8 PYTHONUTF8=1`.** On Windows the CLI prints emoji and crashes with `'charmap' codec can't encode character` under the default console encoding.
- **Pass `--enhance-level 0` to `create`.** Its built-in enhancement would spawn a nested agent (or use paid API mode if `ANTHROPIC_API_KEY` is set); you do this step.
```bash
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create <source> --name <name> --output <staging>/raw --enhance-level 0 -p quick --non-interactive
```
- Docs sites: add `--max-pages N` (start ~150) to bound it. Use `skill-seekers estimate <url>` first if size is unknown.
- GitHub: add `--no-issues --no-releases` unless the user wants issue/release knowledge. Set `GITHUB_TOKEN` if rate-limited (`gh auth token`).
- Presets: `quick` (1–2 min) default; `standard` (5–10 min) or `comprehensive` (20–60 min) only if the user asks for depth. Run long jobs with `run_in_background`.
- Explicit type flags when auto-detect guesses wrong: `--pdf`, `--docx`, `--epub`, `--pptx`, `--notebook`, `--spec`/`--spec-url`, `--video-url`, `--directory`, `--repo`. Still pass the source positionally for URL types — e.g. `create --video-url <url>` with no positional arg fails with "No source provided".
- Multi-source: run one `create` per source into separate raw dirs (simplest, most debuggable).
- Troubleshooting: `skill-seekers doctor`; interrupted scrape: `skill-seekers resume`.

**Native path** — follow the per-type checklist in `references/source-playbook.md`. Save extracted notes into `<staging>/raw/<source-slug>.md` so synthesis works from files, not memory.

### 4. Synthesize
Read the gathered material (for CLI output: its `SKILL.md` and `references/`) and write the skill fresh using `references/skill-template.md`. Rules:
- **Description is the trigger.** Third person, says *what* and *when*, names the concrete keywords/filenames/APIs a user would mention. ≤ 1024 chars.
- **SKILL.md body ≤ ~400 lines.** Core mental model, the 80% workflows, key APIs/commands, gotchas, and a map of reference files. Push detail into `references/`.
- **References:** one file per topic (e.g. `api.md`, `configuration.md`, `examples.md`), each with a short table of contents if > 100 lines. Link every reference from SKILL.md with a line saying when to read it.
- **Prefer real examples** pulled from docs/tests over invented ones. Keep code blocks verbatim; note the version they apply to.
- **Cut** marketing copy, changelogs, navigation chrome, duplicated content, install boilerplate the user doesn't need.
- **Conflicts across sources:** prefer official > newer > more specific; note unresolved conflicts in a "Caveats" section with both sources.
- Record provenance: a `Sources` section at the bottom of SKILL.md with URLs/paths and the date gathered.

### 5. Validate
Before installing, check:
- Frontmatter has `name` (matches folder, kebab-case) and `description`; YAML parses.
- Every `references/...` link in SKILL.md exists; no orphan reference files.
- No secrets, tokens, internal hostnames, or PII copied from sources (grep for `api_key`, `token`, `password`, `BEGIN .*PRIVATE`).
- Spot-check 3 facts in the skill against the raw material.
- Optional: `skill-seekers quality <staging>/<name>` for a score.

### 6. Review & install
Show the user: the file tree, the description, and SKILL.md's section headings plus line counts. Then **ask for approval** to install. On approval:
```bash
mkdir -p ~/.claude/skills/<name> && cp -r <staging>/<name>/. ~/.claude/skills/<name>/
```
Confirm the install path and note the skill loads in new sessions. Leave the staging dir in place unless the user wants it removed.

## Updating an existing generated skill
Re-gather only the changed sources (CLI: `skill-seekers update` or `--resume`), diff against the installed files, show the user the diff summary, and apply on approval. Preserve any hand edits the user made.

## Reference files
- `references/source-playbook.md` — per-source-type detection, CLI flags, and native extraction checklists. Read in step 2–3.
- `references/skill-template.md` — SKILL.md skeleton and description-writing guide. Read in step 4.
