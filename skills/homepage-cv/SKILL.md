---
name: homepage-cv
description: "Maintain homepage CV sources and PDFs. Use when editing public/cv/CV-Hong-Tang.tex or public/cv/CV-Hong-Tang-Chinese.tex, regenerating CV PDFs, adding PDF outlines or bookmarks, syncing publication entries into the CV, checking CV footer dates, or verifying the CV pages."
---

# Homepage CV

## Overview

Use this skill for CV source edits and PDF regeneration. Always treat the English and Chinese CVs as a paired deliverable unless the user explicitly asks for a draft-only source edit.

## Preflight

- Inspect both source files before editing:
  - `public/cv/CV-Hong-Tang.tex`
  - `public/cv/CV-Hong-Tang-Chinese.tex`
- Confirm the CV pages still point to the expected PDF paths:
  - `content/cv.toml`: `/cv/CV-Hong-Tang.pdf`
  - `content_zh/cv.toml`: `/cv/CV-Hong-Tang-Chinese.pdf`
- For publication-related CV edits, first run or inspect the output from:

```bash
npm run cv:publication-sync-check
```

- Use the current working timezone date unless the user specifies another date.

## Work

### Edit CV Sources

- Update both English and Chinese `.tex` sources when content has bilingual equivalents.
- Preserve curated CV ordering, award notes, corresponding-author notes, and manual formatting.
- Update footer dates before compiling:
  - English: `Last updated: Month DD, YYYY`
  - Chinese: `最近更新：YYYY年M月D日`
- Keep output PDF filenames unchanged:
  - `public/cv/CV-Hong-Tang.pdf`
  - `public/cv/CV-Hong-Tang-Chinese.pdf`

### Add Or Maintain PDF Outlines

- Prefer source-level LaTeX outline/bookmark definitions over post-processing PDFs, so future compilations preserve the outline.
- Keep visible heading style unchanged when adding bookmark helpers.
- Use Unicode-capable PDF bookmarks for Chinese text, such as `\usepackage[unicode,hidelinks]{hyperref}` plus `bookmark` when appropriate.
- Include top-level section bookmarks and useful nested bookmarks for long sections such as publications.

### Compile PDFs

- Build both PDFs together with `tectonic`:

```bash
cd public/cv
tectonic CV-Hong-Tang.tex
tectonic CV-Hong-Tang-Chinese.tex
```

- Treat fatal LaTeX errors as blockers.
- Treat small overfull/underfull box warnings as non-blocking unless they indicate visible layout damage.

## Verify

- Confirm both PDF files were regenerated under `public/cv/`.
- For outline/bookmark work, verify with `pypdf`:

```bash
uv run --with pypdf python - <<'PY'
from pathlib import Path
from pypdf import PdfReader

for path in [
    Path("public/cv/CV-Hong-Tang.pdf"),
    Path("public/cv/CV-Hong-Tang-Chinese.pdf"),
]:
    reader = PdfReader(str(path))
    root = reader.trailer["/Root"]
    outline = reader.outline
    print(f"{path}: pages={len(reader.pages)} PageMode={root.get('/PageMode')} top_level_outlines={len(outline)}")
PY
```

- When a detailed outline listing is useful, walk `reader.outline` and print titles with page numbers.
- Confirm footer dates in both TeX sources match the intended update date.
- Confirm `content/cv.toml` and `content_zh/cv.toml` still point to the expected PDF paths.
- Run `npm run build` unless the user requested a draft-only change or there is a clear blocker.
- Review `git diff --stat` and ensure no generated local build output is included.

## Finish

- Summarize source changes, regenerated PDFs, PDF verification results, and build status.
- Mention any non-blocking LaTeX warnings if they remain relevant to the user.
