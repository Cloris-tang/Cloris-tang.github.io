---
name: homepage-publications
description: "Maintain homepage publication metadata. Use when adding or editing papers in content/publications.bib, preparing entries from Zotero by title, checking DOI/title duplicates, adding journal covers under public/papers, updating SCI quartiles or impact factors with EasyScholar, syncing publications into CV sources, or updating Google Scholar statistics."
---

# Homepage Publications

## Overview

Use this skill for publication-facing maintenance on the personal homepage. Treat `content/publications.bib` as the shared source of truth for both English and Chinese publication pages.

## Preflight

- Inspect `content/publications.bib`, `content/publications.toml`, and `content_zh/publications.toml` before editing publication data.
- Check whether the target DOI or title already exists in `content/publications.bib`.
- Inspect `public/papers/` before choosing a `preview` filename; reuse an existing journal cover for the same journal unless there is a specific reason not to.
- Use `uv` for Python commands and never print, echo, or commit secrets such as `EASY_SCHOLAR_SECRET_KEY`.
- Use `accepted = {YYYY-MM-DD}` for acceptance dates. The publication page sorts Year mode by BibTeX `year` first, then by `accepted` within the same displayed year; visible tags still show the BibTeX `year`.
- Do not modify the website when the user only asks for information to fill in later; return a ready-to-paste BibTeX block plus evidence instead.

## Work

### Add Or Edit A Publication

- Start from BibTeX from Zotero, DOI/publisher pages, or Crossref.
- Add or update the entry in `content/publications.bib`.
- Fill the site-specific fields:
  - `preview`: filename only, matching a real file under `public/papers/`
  - `sci`: SCI quartile, such as `Q1`; empty is valid when the source returns empty
  - `sciif`: impact factor; empty is valid when the source returns empty
  - `description`: one-sentence paper description
  - `keywords`: paper keywords
  - `selected`: `{true}` only when the paper should appear on the homepage selected list
  - `accepted`: acceptance date in `YYYY-MM-DD` when it is backed by the PDF first page or publisher evidence
- Add `*` after an author's name only when the publisher page or PDF explicitly marks that author as a corresponding author.

### Prepare Publication Data From Zotero

- Confirm Zotero is running:

```bash
curl -sS --max-time 3 http://127.0.0.1:23119/connector/ping
```

- Search by title or distinctive title keywords:

```bash
curl -sS --max-time 15 --get 'http://127.0.0.1:23119/api/users/0/items' \
  --data-urlencode 'q=paper title or distinctive title keywords' \
  --data-urlencode 'qmode=titleCreatorYear' \
  --data-urlencode 'format=json'
```

- Inspect the matched item and export BibTeX by item key:

```bash
curl -sS --max-time 15 'http://127.0.0.1:23119/api/users/0/items/ITEM_KEY?format=json'
curl -sS --max-time 15 'http://127.0.0.1:23119/api/users/0/items/ITEM_KEY?format=bibtex'
```

- If Zotero includes a local PDF attachment, inspect the first pages before adding corresponding-author markers or other provenance-sensitive details:

```bash
pdftotext -f 1 -l 2 '/absolute/path/from/zotero/file.pdf' -
```

### Add Or Reuse Journal Covers

- Store publication preview images under `public/papers/`.
- Put only the filename in the BibTeX `preview` field.
- For Elsevier or ScienceDirect journals, prefer the journal cover shown on the journal homepage; ScienceDirect covers often follow `https://ars.els-cdn.com/content/image/X<ISSN>.jpg`.
- For non-Elsevier journals, prefer the official publisher journal cover or logo, such as Springer, MDPI, or IEEE.

### Update SCI Quartiles And Impact Factors

- Use EasyScholar through the existing script:

```bash
npm run publication:ranks -- --dry-run
```

- Review the before/after table before applying changes.
- Apply only after the dry run looks correct:

```bash
npm run publication:ranks
```

- Optionally write a report:

```bash
npm run publication:ranks -- --report publication-rank-update.md
```

- Keep the request delay at `0.5` seconds or slower. The script enforces this minimum even if a smaller delay is provided.

### Update Accepted Dates From Zotero PDFs

- Confirm Zotero is running and `pdftotext` is available.
- First run a dry run:

```bash
npm run publication:accepted-dates
```

- Review the found dates. The script searches Zotero by DOI/title, reads the PDF attachment first page, and looks for an accepted date.
- Write confirmed dates back to `content/publications.bib`:

```bash
npm run publication:accepted-dates -- --write
```

- Existing `accepted` fields are preserved unless `--overwrite` is passed.

### Update Google Scholar Statistics

- Use the npm entrypoint:

```bash
npm run scholar:stats
```

- The underlying Python command is:

```bash
uv run python google_scholar_crawler/main.py
```

## Verify

- After adding or editing article entries, run:

```bash
npm run cv:publication-sync-check
```

- If the sync check reports missing CV entries, add the printed `\item` drafts to the journal-paper sections of both CV TeX files, preserve curated CV ordering and notes, then use `skills/homepage-cv/SKILL.md` to regenerate the PDFs.
- Confirm every referenced `preview` file exists under `public/papers/`.
- Run the content integrity check:

```bash
npm run content:check
```

- Run `npm run build` unless the user requested a draft-only change or there is a clear blocker.
- Review `git diff --stat` and keep generated build output out of version control.

## Finish

- Summarize the publication entries changed, evidence used for any corresponding-author markers, assets added or reused, and verification commands run.
- Mention any skipped translation, missing cover, unavailable Zotero item, or EasyScholar empty value explicitly.
