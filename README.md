# searcher

A fast, dependency-free, terminal-based tool to find document files **by name or by content**.

Single-file Python program. No build step. Works out of the box for `txt`, `md`, `pdf`, `docx`, `odt`, `rtf`, `epub`, `pptx`, `xlsx` and other popular text formats.

## Features

- Search **by content**, **by file name**, or both combined (AND).
- Expressive query syntax: `AND`, `OR`, exact `"phrases"`.
- Content extraction with **standard library only** — no mandatory dependencies.
- Optional `pypdf` support for higher-quality PDF text extraction, longer processing time.
- Snippet previews with matched terms highlighted.
- Recursive search with sensible skips (`.git`, `venv`, `__pycache__`, …).
- Script-friendly: exit codes, `--count`, `--limit`, `--absolute`.

## Query syntax

The same syntax applies to `--name` and `--content` searches.
Matching is case-insensitive by default (use `--case-sensitive` to change).

| Search string | Result expected |
|---|---|
| `A` | anything containing `A` |
| `A B` | anything containing `A` AND `B` |
| `A OR B` | anything containing `A` OR `B` |
| `"A A A"` | the exact composed string |
| `"B B B" "A A A"` | must contain both exact strings (AND) |
| `"B B B" OR "A A A"` | must contain at least one of the two (OR) |

Notes:

- A bare `OR` (any case) is the OR operator. To search the literal word "or", quote it: `'"or"'`.
- Terms match as case-insensitive substrings. Quoted text matches the exact sequence, including spaces and punctuation.

## Installation

Requirements: Python 3.8+ (tested on 3.14). No required packages.

```bash
git clone https://github.com/cesarbrod/searcher
cd searcher
chmod +x searcher
./searcher --help
```

Optional — better PDF extraction (less performance):

```bash
pip install pypdf fonttools
```
then remove the leading # from the line

`#import pypdf`

on the file `search`

Optionally put it on your `PATH`:

```bash
ln -s "$PWD/searcher" ~/.local/bin/searcher
```

## Usage

```bash
searcher [PATH] [QUERY] [options]
searcher [PATH] --name QUERY
searcher [PATH] --content QUERY
searcher [PATH] --name NAME_Q --content CONTENT_Q
```

- `PATH` is a directory (searched recursively) or a single file. Default: `.`.
- A bare `QUERY` positional defaults to a **content** search.
- `--name X "Y"` means name matches `X` AND content matches `Y`.

### Examples

```bash
# Content search in current dir
./searcher . "annual report"

# OR logic
./searcher ~/docs "budget OR forecast"

# Exact phrase
./searcher ~/docs '"quarterly results"'

# Both phrases must appear
./searcher ~/docs '"quarterly results" "annual report"'

# Either phrase
./searcher ~/docs '"quarterly results" OR "annual summary"'

# By file name
./searcher ~/docs --name "report 2024"

# Name AND content combined
./searcher ~/docs -n invoice -s "paid OR overdue"

# Only PDFs and Word docs, show 5 results max
./searcher ~/docs "contract" --ext pdf,docx --limit 5

# Scripting: just the count
./searcher ~/docs "TODO" --count

# Include source code files in content search
./searcher . "TODO" --all-text --no-recursive
```

## Options

| Flag | Description |
|---|---|
| `path` | Directory or file to search (default: `.`) |
| `query` | Positional query (content search unless `--name` is used) |
| `-n, --name QUERY` | Search by file name |
| `-s, --content, -c QUERY` | Search inside file content |
| `--ext E1,E2` | Restrict to extensions, e.g. `--ext pdf,docx,txt,md` |
| `--all-text` | Also include source-code extensions for content search |
| `--list-exts` | List supported extensions and exit |
| `--case-sensitive` | Case-sensitive matching (default: insensitive) |
| `--no-recursive` | Do not descend into subdirectories |
| `--hidden` | Include hidden files/dirs |
| `--max-size MB` | Skip files larger than this (default: `50`) |
| `--lines N` | Snippet lines per content hit (default: `2`, `0`=none) |
| `--limit N` | Show at most `N` results (default: all) |
| `--count` | Only print the match count |
| `--absolute` | Print absolute paths (default: relative to `PATH`) |
| `--errors` | Show per-file read/parse warnings |
| `--version` | Show version and exit |

Exit codes: `0` = match(es) found, `1` = no matches, `2` = error.

## Supported formats

Run `./searcher --list-exts` for the full list.

| Format | Backend |
|---|---|
| `txt md markdown rst log csv tsv json yaml yml toml ini xml html htm tex org adoc` + source code (`py js ts sh java c …`) | read as text (utf-8 → latin-1 fallback) |
| `docx pptx xlsx` | stdlib `zip` + XML |
| `odt ods odp` | stdlib `content.xml` parsing |
| `epub` | stdlib HTML extraction from the zip |
| `rtf` | stdlib control-word stripper |
| `pdf` | `pypdf` → `pdfminer` → `PyPDF2` → crude stdlib fallback |
| `doc` (legacy OLE) | **not supported** — skipped with a warning; convert to `.docx` first |

Default content-search set excludes source-code extensions; add `--all-text` to include them or `--ext` to define your own set.

Skipped automatically: `__pycache__ .git .hg .svn node_modules .venv venv .tox .pytest_cache .mypy_cache`, hidden files (unless `--hidden`), files over `--max-size`.

## How it works

1. **Collect candidates** — walk `PATH`, filter by extension and size.
2. **Parse query** — split into OR-groups of AND-terms (`A B` → `[["A","B"]]`, `A OR B` → `[["A"],["B"]]`).
3. **Match**:
   - name search: substring match against the file name;
   - content search: extract text per format, then substring match against the full text.
4. **Report** — ranked alphabetically, with up to `--lines` matching lines per hit (`*term*` highlighted) and a final `N match(es)` summary.

## Limitations

- PDF quality without `pypdf` installed is best-effort (falls back to raw string extraction). Install `pypdf` for reliable results.
- Scanned-image PDFs (no text layer) require OCR — not supported.
- Legacy `.doc`, password-protected Office files, and DRM'd EPUBs are skipped.
- Very large trees are single-threaded; use `--ext`, `--limit`, or `--max-size` to narrow the scope.

## Project layout

```text
searcher/
├── searcher    # the program (single file, executable)
└── README.md
```

## License

GPL v3.0 or later

MIT — do what you want, no warranty. See `LICENSE` if your fork adds one.
