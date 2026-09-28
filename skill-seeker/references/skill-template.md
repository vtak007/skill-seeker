# Generated Skill Template

## Layout
```
<name>/
├── SKILL.md              # ≤ ~400 lines, always loaded when triggered
└── references/           # loaded on demand
    ├── <topic>.md
    └── ...
```
Add `scripts/` only if the source provides a genuinely reusable helper (e.g. a validation command); don't invent scripts.

## Writing the description (the trigger)
- Third person: "Reference for X…", "Use when…".
- Include: what the subject is, what the skill helps do, and **concrete trigger words** — product name, package/import names, CLI names, file extensions, key API names.
- Say when *not* to use it if there's an easily-confused neighbour.
- ≤ 1024 characters, single line in YAML (quote it if it contains `:`).

Good: `Reference for the FastAPI web framework (fastapi, pydantic, uvicorn) — routing, dependency injection, request/response models, auth, background tasks, testing with TestClient. Use when writing or debugging FastAPI apps or when the user mentions @app.get, Depends, APIRouter, or HTTPException.`

Bad: `FastAPI docs.`

## SKILL.md skeleton
```markdown
---
name: <name>
description: <trigger description>
---

# <Subject>

<1–3 sentences: what it is, version covered, what this skill is for.>

## Core concepts
<The mental model in bullets/short paragraphs — the things a newcomer gets wrong.>

## Common tasks
### <Task 1>
<Steps + minimal verbatim example.>
### <Task 2>
...

## Key APIs / commands / config
<Compact table: name | purpose | notable params. Full detail → references/.>

## Gotchas
- <Pitfall> — <why> — <fix>

## Reference files
- `references/<topic>.md` — <what's in it; when to read it>

## Caveats
<Unresolved source conflicts, version limits, gaps in coverage. Omit if none.>

## Sources
- <URL/path> — gathered <YYYY-MM-DD> (<version/commit if known>)
```

## Reference file header
```markdown
# <Topic>
Source: <URL/path> · <version>

**Contents:** <section list — required if > 100 lines>
```
