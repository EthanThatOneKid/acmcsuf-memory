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
| `raw/acmcsuf/` | Immutable append-only captures of acmcsuf.com `officers.json` records (one capture per officer-term), plus the standalone `SUMMARY.json` contract |
| `wiki/people/` | Enriched, generated person pages (board history, Discord handles) written under the `people/` namespace so they compose into the memory hub without route collisions |

## Pipeline

The connector and enrichment live in the `memory` repo and write here via
`--repo` (mirrors `calendar-memory`/`gmail-memory`):

```bash
# from the memory repo
python -m connectors.acmcsuf --repo ../acmcsuf-memory fetch
python -m scripts.enrich_acmcsuf --repo ../acmcsuf-memory
```

- Captures are append-only and deduplicated by a stable source id
  (`<officer-slug>-<term>`); a changed term appends a new fingerprint-suffixed
  capture and never rewrites history (enforced by
  `scripts/memory_check.py` in the memory repo).
- Person pages are generated deterministically under `wiki/people/` and are
  idempotent to re-run.
- Cross-wiki linking is authored only from the memory hub via multi-valued
  `schema:sameAs` using the `acmcsuf:` CURIE prefix; generated pages carry a
  plain markdown backlink, never a semantic assertion.

## Commands

- `wiki check` — SHACL/structure validation (exit 0 when clean)
- `wiki fmt` — format markdown