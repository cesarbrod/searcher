# searcher tutorial for Linux beginners

This tutorial teaches you how to use **searcher**, a small program that finds
document files on your computer **by file name or by what's written inside
them**. No prior Linux experience is assumed: every step is explained, and
every command can be copied and pasted.

By the end you will be able to:

- run `searcher` from the terminal,
- find files by name and by content,
- combine search words with `AND`, `OR`, and exact `"phrases"`,
- limit the search to PDFs, Word files, and other formats,
- understand how `searcher` differs from `find`, `locate`, and `grep`.

---

## 1. What is searcher?

`searcher` is a single-file program. You give it:

1. **where to look** — a folder (or a single file), and
2. **what to look for** — one or more words, or an exact phrase.

It answers with a list of matching files, plus a short preview of the
matching lines. It understands plain text files (`txt`, `md`, `csv`, …) and
real documents: **PDF**, **Word** (`docx`), **OpenDocument** (`odt`),
**RTF**, **ePub**, **PowerPoint** (`pptx`), and **Excel** (`xlsx`).

Example of what a result looks like:

```text
searcher: content:'"quarterly results"' under /home/ana/docs (recursive)
────────────────────────────────────────────────────────────────────────
  [1] report_2024.pdf
    │ ...*quarterly results* grew 12 percent...

────────────────────────────────────────────────────────────────────────
1 match(es)
```

## 2. How is searcher different from find, locate, and grep?

Linux already has classic search tools. Each one is good at something, and
`searcher` fills the gap they leave. Here is the short version:

| Task | `find` | `locate` | `grep` | `searcher` |
|---|---|---|---|---|
| Find files **by name** | yes | yes (instant) | no | yes |
| Find files **by content** | only together with `grep` | no | yes (text files) | yes |
| Read inside **PDF / Word** files | no | no | no | yes |
| Exact `"phrases"`, `AND`, `OR` | no (needs regex tricks) | no | partial (regex) | yes, simple syntax |
| Needs an index database | no (live walk) | yes (updated ~daily) | no (live walk) | no (live walk) |
| Shows preview snippets | no | no | yes (`-C`) | yes |

In plain words:

- **`find`** walks through folders and matches file *names* (or dates, sizes,
  …). It knows nothing about file *contents*. To look inside files you must
  combine it with `grep`, which gets clumsy fast.
- **`locate`** is very fast because it searches a pre-built list of file
  names instead of your disk. Downsides: it only knows names (never
  contents), and its list is usually rebuilt once a day, so files you created
  an hour ago may be missing.
- **`grep`** (often used as `grep -r`) searches *inside* files and is
  excellent for plain text and code. But it cannot read PDF or Word files —
  for those it just says "Binary file matches" (or nothing useful), and it
  has no simple `AND` / `OR` / phrase logic.
- **`searcher`** is the "just tell me which documents mention X" tool: live
  search (no stale database), real document formats, and a query language
  made of plain words instead of regular expressions.

When to use which:

- "Where did I save that file called invoice?" → `locate invoice` or
  `find ~ -name "*invoice*"` (fast, simple).
- "Which code files contain this function name?" → `grep -rn "def foo" .`
  (`grep` is still king for source code).
- "Which of my PDFs and Word docs mention quarterly results?" → `searcher`.
  Neither `find`, `locate`, nor `grep` can do this properly.

## 3. Opening a terminal and running your first search

On most Linux systems press `Ctrl + Alt + T` to open a terminal. You will see
a prompt like `ana@pc:~$`. The `~` means your home folder.

Suppose you downloaded `searcher` into `~/searcher` (the project folder).
First, go there and make the program executable (you only do this once):

```bash
cd ~/searcher
chmod +x searcher
./searcher --help
```

Notes for beginners:

- `cd` means "change directory" (move into a folder).
- `chmod +x` means "allow this file to be run as a program".
- `./` in front means "run the program in *this* folder". Linux does not run
  programs from the current folder unless you ask with `./` — this is normal,
  not an error.
- If you see `command not found`, you probably forgot the `./`.
- If you see `Permission denied`, run the `chmod +x searcher` line again.

`./searcher --help` prints the full manual. Try it now — everything in this
tutorial is also summarized there.

> **Tip:** to run `searcher` from anywhere without `cd`, put it on your
> `PATH` once: `ln -s "$PWD/searcher" ~/.local/bin/searcher`. Afterwards you
> can type just `searcher` in any folder. (Log out and back in if your shell
> does not find it immediately.)

## 4. Your first searches

The basic shape is:

```bash
./searcher [WHERE] [WHAT] [options]
```

- `WHERE` is a folder (searched including subfolders) or a single file.
  If you skip it, the current folder (`.`) is used.
- `WHAT` is what you are looking for. On its own it means **search inside
  the files' contents**.

Try these (replace `~/docs` with a folder you actually have):

```bash
# Which files mention "annual report"? (both words, anywhere in the file)
./searcher ~/docs "annual report"

# Search the current folder instead
./searcher . "annual report"

# Just count the matches (useful in scripts)
./searcher ~/docs "annual report" --count
```

While it runs, you will see a one-line status display: first how many files
were found, then a progress bar with the file being read, and finally a
summary like `searcher: 3 match(es) in 120 file(s) (0.8s)`. This goes to a
separate channel (stderr), so it never pollutes piped output. Add `-q` (or
`--quiet`) to turn it off, e.g. `./searcher ~/docs "x" --count -q`.

## 5. Searching by file name

Add `-n` (or `--name`) to match against file names instead of contents:

```bash
# Files whose NAME contains "report" or "invoice"
./searcher ~/docs --name "report OR invoice"

# Names are matched with the same word logic as contents (see next section)
./searcher ~/docs --name "2024 budget"
```

You can even combine both: name must match X **and** content must match Y:

```bash
# Files named like "invoice" whose content mentions "paid" or "overdue"
./searcher ~/docs -n invoice -s "paid OR overdue"
```

(`-s` / `--content` makes the content query explicit.)

## 6. The query language: AND, OR, and exact phrases

This is the heart of `searcher`. The same rules apply to name and content
searches. Matching ignores upper/lower case unless you pass
`--case-sensitive`.

| You type | You get |
|---|---|
| `apple` | files containing `apple` |
| `apple banana` | files containing `apple` **AND** `banana` (anywhere, any order) |
| `apple OR banana` | files containing `apple` **or** `banana` (at least one) |
| `"apple pie"` | files containing the exact phrase `apple pie` |
| `"apple pie" "banana bread"` | files containing **both** exact phrases |
| `"apple pie" OR "banana bread"` | files containing **at least one** of the phrases |

### Quoting in the terminal (important!)

The shell also uses quotes, so you must wrap your query in quotes for it to
arrive in one piece. Rules of thumb:

- **Always wrap the query in double quotes**: `"annual report"`.
- **Exact phrases need single quotes outside, double quotes inside**:
  `'"quarterly results"'`. (Double quotes outside would make the shell eat
  the inner ones.)
- The word `OR` must be uppercase to act as "or". To search the literal word
  "or", quote it: `'"or"'`.

Examples to copy:

```bash
./searcher ~/docs "budget forecast"                        # AND
./searcher ~/docs "budget OR forecast"                     # OR
./searcher ~/docs '"quarterly results"'                    # exact phrase
./searcher ~/docs '"quarterly results" "annual summary"'   # both phrases
./searcher ~/docs '"quarterly results" OR "annual summary"' # either phrase
```

## 7. Limiting which files are searched

By default, content search looks at document-like files (`txt`, `md`, `pdf`,
`docx`, `odt`, `rtf`, `epub`, `pptx`, `xlsx`, …) and skips source code. You
can change that:

```bash
# Only PDF files
./searcher ~/docs "contract" --ext pdf

# Several formats (comma-separated, with or without dots)
./searcher ~/docs "contract" --ext pdf,docx,txt

# Also include source code and scripts
./searcher ~/projects "TODO" --all-text

# See the full list of supported extensions
./searcher --list-exts
```

Other handy scope options:

```bash
./searcher ~/docs "x" --no-recursive   # current folder only, no subfolders
./searcher ~/docs "x" --hidden         # also search hidden files/folders
./searcher ~/docs "x" --max-size 10    # skip files bigger than 10 MB
./searcher ~/docs "X" --case-sensitive # "X" no longer matches "x"
./searcher ~/docs "x" --absolute       # print full paths instead of short ones
```

## 8. Reading the results

Each hit shows the file plus up to 2 preview lines with the matched words
marked `*like this*`:

```bash
./searcher ~/docs "warranty" --lines 5    # show 5 preview lines per file
./searcher ~/docs "warranty" --lines 0    # no previews, just file names
./searcher ~/docs "warranty" --limit 5    # show only the first 5 hits
./searcher ~/docs "warranty" --count      # print only the number, e.g. 14
```

Exit codes (useful when combining with other commands): `0` = something
found, `1` = nothing found, `2` = error (e.g. folder does not exist).

## 9. The classic tools, side by side

Same jobs, different tools. Run these to feel the difference (using a folder
`~/docs` with a file `notes.txt` containing "call Ana about the warranty"):

```bash
# --- by NAME ---
find ~/docs -type f -name "*warranty*"     # live walk, glob pattern
locate warranty                            # instant, but may miss new files
./searcher ~/docs --name warranty          # live walk, word logic

# --- by CONTENT (plain text) ---
grep -ri "warranty" ~/docs                 # classic, shows every matching line
grep -rli "warranty" ~/docs                # -l: file names only
grep -rn -C 2 "warranty" ~/docs            # -n: line numbers, -C 2: 2 lines of context
./searcher ~/docs "warranty"               # file names + short previews

# --- by CONTENT, two words anywhere (AND) ---
grep -ril "warranty" ~/docs | xargs grep -li "ana"   # clumsy two-step way
./searcher ~/docs "warranty ana"                     # the same, directly

# --- inside PDFs: only searcher works ---
grep -ri "warranty" ~/docs                 # "Binary file report.pdf matches" — useless
./searcher ~/docs "warranty" --ext pdf     # actually reads the PDF text
```

A note on `locate`: if it never finds your new files, its database is stale.
Refreshing it (`sudo updatedb`) needs administrator rights — one more reason
`searcher`'s live search is convenient.

## 10. Cookbook: common tasks

```bash
# All PDFs mentioning "LinkedIn"
./searcher ~/Documents "LinkedIn" --ext pdf

# Invoices (by name) that mention "overdue"
./searcher ~/docs -n invoice -s overdue

# How many meeting notes mention "budget" or "forecast"?
./searcher ~/notes "budget OR forecast" --count

# Exact error message across manuals (any format)
./searcher ~/manuals '"paper jam in tray 2"'

# Everything mentioning the client, newest docs only (by limiting scope first)
./searcher ~/clients/acme "acme" --ext pdf,docx --limit 10

# Search a single file
./searcher ~/docs/handbook.pdf "parental leave"

# See WHY some files were skipped
./searcher ~/docs "x" --errors
```

## 11. Troubleshooting

**`command not found`** — you forgot `./` (run `./searcher`, not `searcher`),
unless you installed it on your `PATH` (see section 3).

**`Permission denied`** — run `chmod +x searcher` once.

**No matches, but the file surely contains the word** — check three things:
1. Is the file's extension in the searched set? (`./searcher --list-exts`).
   Source code needs `--all-text`; anything else needs `--ext`.
2. Is it an old `.doc` file (not `.docx`)? Those are skipped with a warning —
   convert to `.docx` first (e.g. with LibreOffice).
3. Is it a *scanned* PDF (photos of pages, no real text)? No text search tool
   can read those without OCR software.

**Weird PDF warnings** — you should not see any; `searcher` silences the
parser's chatter and quietly skips corrupt files. Add `--errors` to list
which files were skipped and why.

**"PDF fallback extractor in use"** — install `pypdf` once
(`pip install pypdf`) for much better PDF text extraction.

**Search feels slow** — narrow it down: `--ext pdf,docx`, `--max-size 20`,
`--no-recursive`, or point at a smaller folder. Very large trees are scanned
file by file (single-threaded), like `grep -r`.

## 12. Cheat sheet

```bash
./searcher . "words"                    # contents, current folder, recursive
./searcher ~/docs "a b"                 # AND
./searcher ~/docs "a OR b"              # OR
./searcher ~/docs '"exact phrase"'      # exact phrase
./searcher ~/docs --name "invoice"      # by file name
./searcher ~/docs -n inv -s "paid"      # name AND content
./searcher ~/docs "x" --ext pdf,docx    # only these formats
./searcher ~/docs "x" --count           # just the number
./searcher ~/docs "x" -q                # no status bar
./searcher --help                       # full manual
```

Happy searching!
