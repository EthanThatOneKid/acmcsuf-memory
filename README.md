# acmcsuf-memory

Machine-derived memory for the [ACM at CSUF](https://acmcsuf.com/) chapter,
composed into the [memory hub](https://github.com/EthanThatOneKid/memory) as an
external wiki context source.

Published site: <https://ethanthatonekid.github.io/acmcsuf-memory/>

This repo is the **source context** for ACM board history. The `memory` hub owns
the curated profiles; this repo owns machine-generated ACM facts captured from
acmcsuf.com's public board records.

## Layout

| Path | Role |
| --- | --- |
| `raw/acmcsuf/` | Immutable append-only snapshots of acmcsuf.com `officers.json` (`officers-<utc-timestamp>-<sha12>.json`, one pristine board file per changed fetch), plus the standalone `SUMMARY.json` contract |
| `wiki/people/` | Enriched, generated person pages (board history, Discord handles) written under the `people/` namespace so they compose into the memory hub without route collisions |

## Pipeline

The connector and enrichment live in the `memory` repo and write here via
`--repo` (mirrors `calendar-memory`/`gmail-memory`):

```bash
# from the memory repo
python -m connectors.acmcsuf --repo ../acmcsuf-memory fetch
python -m scripts.enrich_acmcsuf --repo ../acmcsuf-memory
```

- Snapshots are append-only and deduplicated by content: `.cursor` holds the
  sha256 of the last fetched payload, so an unchanged board appends nothing and
  a changed board appends one new snapshot (`officers-<timestamp>-<sha12>.json`)
  without ever rewriting history (enforced by `scripts/memory_check.py` in the
  memory repo). This was a one-time format migration from per-officer-term
  capture files.
- Person pages are generated deterministically from snapshots merged in
  chronological order (newest snapshot wins per term; officers who stop
  appearing in `officers.json` keep their history pages), and are idempotent to
  re-run.
- Cross-wiki linking is authored only from the memory hub via multi-valued
  `schema:sameAs` using the `acmcsuf:` CURIE prefix; generated pages carry a
  plain markdown backlink, never a semantic assertion.

## Commands

- `wiki check` — SHACL/structure validation (exit 0 when clean)
- `wiki fmt` — format markdown