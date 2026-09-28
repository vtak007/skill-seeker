# Source Playbook

Per-source detection, CLI invocation, and native extraction checklists.

**Contents:** Detection · Docs sites · GitHub repos · Local codebases · Documents (PDF/DOCX/PPTX/EPUB) · Notebooks · OpenAPI · Video · Other CLI sources · Multi-source

**Always prefix CLI calls with `PYTHONIOENCODING=utf-8 PYTHONUTF8=1`** (Windows console encoding crash otherwise).

Common CLI suffix used below: `--name <name> --output <staging>/raw/<slug> --enhance-level 0 --non-interactive`

## Detection

| Input looks like | Type |
|---|---|
| `https://github.com/o/r`, `o/r` | GitHub repo |
| `https://…` ending `.pdf/.docx/.pptx/.epub/.ipynb/.json/.yaml` | Remote file — download to staging first (`curl -L -o`) |
| `https://…/openapi.json`, `swagger.*` | OpenAPI spec |
| `youtube.com`, `youtu.be`, `vimeo.com` | Video |
| other `https://…` | Docs site (single page if the user says "this page") |
| existing directory | Local codebase |
| existing file | By extension |

When unsure: `skill-seekers create <source> --dry-run` shows what it would detect.

## Docs sites
**CLI (large sites):**
```bash
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers estimate <url>                       # page count
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create <url> -p quick --max-pages 150 <suffix>
```
- Scope to a subsection by passing the deepest useful URL (e.g. `/docs/api/`).
- JS-rendered sites that come back empty: add `--browser` (needs Playwright).
- Be polite: default rate limit is fine; don't pass `--no-rate-limit`.

**Native (small sites / single pages):**
1. WebFetch the entry page: ask for the full navigation/sidebar link list and a summary.
2. Pick the pages that cover: getting started, core concepts, main API/config reference, common recipes, troubleshooting/FAQ. Skip blog, changelog, marketing.
3. WebFetch each, asking for *verbatim* code blocks, signatures, option tables, and warnings — not summaries.
4. Save each to `<staging>/raw/<slug>.md` with its URL at the top.

## GitHub repos
**CLI (medium/large repos):**
```bash
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create owner/repo -p quick --no-issues --no-releases <suffix>
```
- Include `--max-issues 50 --issue-state closed` only if the user wants known-problem knowledge.
- Private repos / rate limits: `GITHUB_TOKEN=$(gh auth token) PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create …`
- Already cloned locally: `--local-repo-path <path>` avoids re-cloning.

**Native (small repos):**
1. `gh repo view owner/repo` → description, topics, default branch.
2. `gh api repos/owner/repo/contents/<path>` or WebFetch raw files: README, `docs/`, `examples/`, main entry module, config schema, CHANGELOG head (for version).
3. Tests are the best usage examples — sample 2–3 test files for the core APIs.
4. Note the version/commit SHA gathered.

## Local codebases
**Native (default):**
1. Glob the tree (skip `node_modules`, `.git`, `dist`, `build`, `__pycache__`, `vendor`).
2. Read README/CLAUDE.md/`Workspace Map.md` if present, manifests (`package.json`, `pyproject.toml`, `*.csproj`, …), entry points.
3. Grep for public API surface (exports, class/def declarations, CLI arg parsers, config keys).
4. Capture conventions: naming, error handling, build/test commands.
5. Point at code by **symbol**, not line number, in the generated skill.

**CLI (large codebases):**
```bash
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create ./path -p standard --directory ./path <suffix>
```
Add `--languages python,typescript` / `--file-patterns` to narrow.

## Documents (PDF / DOCX / PPTX / EPUB)
**Native first:** Read handles PDFs (use `pages`, ≤ 20 per call; required > 10 pages) and images. For DOCX/PPTX use the `anthropic-skills:docx` / `pptx` skills or `python -c` with `python-docx` / `python-pptx` if installed.

**CLI when native struggles:**
```bash
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create --pdf file.pdf [--ocr] [--pages 1-80] <suffix>
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create --docx file.docx <suffix>
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create --pptx deck.pptx <suffix>
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create --epub book.epub <suffix>
```
- Scanned PDF (Read returns no text) → `--ocr`.
- Huge manuals: extract the TOC first, then only the chapters matching the user's focus.

## Notebooks (.ipynb)
Read the notebook natively (cells + outputs). Keep code cells that demonstrate APIs; keep outputs only where they explain behaviour. CLI alternative: `--notebook file.ipynb`.

## OpenAPI / Swagger
Native: Read/WebFetch the spec; produce `references/endpoints.md` grouped by tag (method, path, purpose, key params, auth) and `references/schemas.md` for core models. Put auth + base URL + pagination/error conventions in SKILL.md.
CLI: `--spec file.yaml` or `--spec-url <url>`.

## Video (YouTube / Vimeo / file)
CLI only; requires extras (`pip install "skill-seekers[all]"`, may need ffmpeg):
```bash
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 skill-seekers create <url> -p quick <suffix>
```
- **Pass the video/playlist URL positionally.** `create` requires a positional source; `create --video-url <url>` alone fails with "No source provided". `--video-url` / `--video-playlist` are type overrides, not substitutes for the positional arg (playlist form untested).
- `--non-interactive` is ignored for video sources (harmless warning). Output is small: `SKILL.md`, one `references/video_*.md` (timestamped, chaptered transcript) and `video_data/metadata.json`.
- Quick metadata check before the run: `python -m yt_dlp --skip-download --print "%(title)s | %(channel)s | %(duration_string)s" <url>` — use the title to propose the skill name.
If extras are missing, tell the user the install command; don't try to scrape video pages natively. Transcripts are noisy — synthesize procedures and facts, drop filler.

**Transcript sources** (the CLI tries these in order, stopping at the first that works):
1. YouTube captions via `youtube-transcript-api` — covers most YouTube videos; nothing else needed.
2. `.srt` / `.vtt` file beside a local video (`--video-file`).
3. Whisper speech-to-text — only if `faster-whisper` is installed (`pip install faster-whisper`, large download) **and** ffmpeg is on PATH. Needed only for uncaptioned videos or local files without subtitles.
4. None — the CLI logs "No transcript available" and continues. If that appears, tell the user and offer the Whisper install; don't treat it as a failure of the whole run.

## Other CLI sources
Only via CLI (see `skill-seekers create --help-all`): AsciiDoc `--asciidoc-path`, HTML dump `--html-path`, RSS/Atom `--feed-url`, man pages `--man-names`, Confluence `--conf-base-url --space-key` or `--conf-export-path`, Notion `--notion-export-path` or `--database-id/--page-id`, Slack/Discord exports `--chat-export-path --platform slack|discord`.

## Multi-source
1. Gather each source into its own `<staging>/raw/<slug>/`.
2. Build a topic outline across all sources before writing.
3. For each topic, merge; resolve conflicts official > newer > more specific; list unresolved ones under "Caveats".
4. Cite which source each reference file draws from in its header.
