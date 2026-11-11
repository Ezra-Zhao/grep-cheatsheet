# grep Cheatsheet — Commands That Actually Work

One page. Thirty entries: the everyday search flags, recursion and file filters, the three regex flavors and how much of a line must match, context and output shaping, and the pipeline behavior — exit codes, binary files, NUL records, live streams — that scripts depend on. No folklore: every entry was **run for real** against a fixture tree (`proj/`: log files, notes, C/Python sources, a binary file, a NUL-separated file, symlinks, a file with a space in its name) with GNU grep 3.11, and the semantics checked against the GNU Grep manual. Where GNU and BSD grep differ, the sheet says so.

> 🌍 Read this project in your language: [中文](README.md) · [Español](README.es.md) · [Português](README.pt.md)

Docs: https://www.gnu.org/software/grep/manual/

---

## 🧱 Basics — find lines, count them, list files

### `grep 'ERROR' proj/app.log` — the plain search
```bash
grep 'ERROR' proj/app.log    # prints the 2 lines that contain ERROR
```
The pattern is a regular expression, tested against each line; matching lines go to stdout. Case matters: the file's lowercase `error` line is *not* printed. A match also sets the exit status to 0, no match to 1 — the last section of this sheet lives on that.
🏷️ `#basics` `#search` `#must-know`
### `-i` — ignore case
```bash
grep -i 'error' proj/server.log   # 2 lines: "Error mixed case failure" and "ERROR disk full"
grep -i 'error' proj/app.log      # 3 lines: the two ERROR lines plus the lowercase "error" line
```
Log files never agree on capitalization, so `-i` is the default reflex when hunting incidents. It folds case in both the pattern and the text.
🏷️ `#basics` `#case`
### `-v` — invert the match
```bash
grep -v 'INFO' proj/app.log    # 4 lines: everything that is NOT an INFO line
```
"Show me the log with the routine noise removed" is half of all log reading, and this is the whole command: lines that do *not* match are selected instead.
🏷️ `#basics` `#filter` `#must-know`
### `-c` — count matching lines
```bash
grep -c 'ERROR' proj/app.log proj/server.log   # proj/app.log:2  proj/server.log:1 — one count per file
grep -c 'ZZZ' proj/app.log                     # prints 0 — and still exits 1, because no line was selected
```
It counts *lines that match*, not matches: a line with three occurrences still counts once (for occurrences, see `-o` below). ⚠️ The `0`-with-exit-1 case surprises scripts: `-c` printing `0` is a successful count of a failed search.
🏷️ `#basics` `#count`
### `-n` — line numbers
```bash
grep -n 'ERROR' proj/app.log    # 3:…failed to open config   6:…timeout after 30s
```
Prefixes each match with its 1-based line number in its file — the form you paste into an editor, a ticket, or a `sed -n '3p'` follow-up.
🏷️ `#basics` `#output`
### `-l` and `-L` — list files, not lines
```bash
grep -l 'ERROR' proj/app.log proj/server.log proj/notes.txt   # proj/app.log, proj/server.log
grep -L 'ERROR' proj/app.log proj/server.log proj/notes.txt   # proj/notes.txt — the one WITHOUT any match
```
`-l` stops at each file's first match and prints only the name; `-L` is the exact complement. These are the building blocks of `grep -l '…' *.log | xargs …` pipelines — and the fastest way to answer "which files even mention this?".
🏷️ `#basics` `#files` `#must-know`
### `-r` — search a whole tree
```bash
grep -r 'ERROR' proj    # every match under proj/, each prefixed with its file name
```
Recursion turns grep into a codebase search: every line comes out as `path:line`. Two things it will *not* do, both verified: symbolic links found *inside* the tree are skipped (the fixture's `link.log` and `extlink/` are silently passed over), and a matching binary file is reported as a one-line notice on stderr (`grep: proj/records0.txt: binary file matches`), not printed. With no path operand, `grep -r` searches the working directory.
🏷️ `#basics` `#recursive` `#must-know`
### `-R` — recursion that follows every symlink
```bash
grep -R 'ERROR' proj     # now proj/link.log:… and proj/extlink/deep.log:… appear too
grep 'ERROR' proj/link.log   # a symlink named ON the command line is followed even by plain grep
```
The capital letter is the dereference switch: `-R` follows symlinks wherever it meets them, `-r` follows only the ones you named yourself. Reach for `-R` when the tree is stitched together with links — and expect possible duplicates, since the same file can arrive by two paths.
🏷️ `#basics` `#recursive` `#symlinks`
### `--include`, `--exclude`, `--exclude-dir` — aim the recursion
```bash
grep -r --include='*.log' 'ERROR' proj     # only *.log files: the .c/.py sources are never opened
grep -r --exclude-dir='sub' 'ERROR' proj   # the whole sub/ subtree is pruned from the search
```
The globs match against each file's base name during recursion. `--include` whitelists, `--exclude` blacklists files, `--exclude-dir` prunes directories before grep descends into them — the difference between a two-second search of a source tree and a long walk through `node_modules`.
🏷️ `#basics` `#recursive` `#files`

---

## 🎯 Patterns — regex flavors, and how much of the line must match

### `-E` — extended regex: `|`, `+`, `?`, `{}` work bare
```bash
grep -E 'ERROR|WARN' proj/app.log     # 3 lines — alternation with a bare |
grep 'ERROR\|WARN' proj/app.log       # the SAME search in default BRE: the | needs a backslash
```
Default grep speaks BRE (basic regex), where `| + ? ( ) { }` are literal characters and the backslash forms `\| \+ \?` do the magic. `-E` (extended, like `egrep`) flips that: the bare characters are the operators. Pick one dialect per command and stay in it — mixing them up is grep's most common silent failure.
🏷️ `#patterns` `#regex` `#must-know`
### `-F` — the pattern is a literal string, not a regex
```bash
grep -F 'error.' proj/notes.txt    # no match (exit 1): no line contains a literal "error."
grep 'error.' proj/notes.txt       # 3 lines: the . is a wildcard and matches _, s, space, …
grep -F 'log_error(' proj/src/main.c   # finds it — parentheses need no escaping under -F
```
When the text you want contains regex punctuation — function calls, file paths, version strings — `-F` (the old `fgrep`) matches it character for character. It is also the fastest flavor, since no regex engine runs at all.
🏷️ `#patterns` `#literal` `#must-know`
### `-w` — whole words only
```bash
grep -w 'error' proj/notes.txt    # 1 line: "error handling notes"
```
Without `-w`, `error` also matches inside `Errors`, `error_codes` and `terror` — all present in the fixture, all correctly rejected here. A "word" is bounded by non-word characters (anything but letters, digits and `_` — which is why `error_codes` does not count).
🏷️ `#patterns` `#words`
### `-x` — the whole line must match
```bash
grep -x 'deep INFO calm' proj/sub/deep.log   # matches: the line is exactly this
grep -x 'deep INFO' proj/sub/deep.log        # no match (exit 1): a line prefix is not the line
```
`-x` anchors the pattern at both ends, as if it were wrapped in `^…$`. The right tool for exact-line lookups in config files and for verifying a generated line byte-for-byte.
🏷️ `#patterns` `#exact`
### `-e` — several patterns at once
```bash
grep -e 'ERROR' -e 'WARN' proj/app.log   # 3 lines: a line matching EITHER pattern is printed
```
Each `-e` adds an alternative — a line that matches any of them is selected once. `-e` is also the escape hatch for patterns that begin with a dash (last section).
🏷️ `#patterns` `#multiple`
### `-f` — read the patterns from a file
```bash
grep -f proj/patterns.txt proj/app.log   # patterns.txt holds ERROR and timeout: 2 lines match
```
One pattern per line in the file; a line matches if it matches any of them — `-e` without the command-line clutter, and the list can be generated by another program. An empty pattern line matches every line, so a stray blank line in the file turns the search into "print everything".
🏷️ `#patterns` `#files`
### `^` and `$` — anchor to the line's edges
```bash
grep 'complete$' proj/app.log    # 1 line: "…INFO shutdown complete" — the line ENDS with the word
grep '^INFO' proj/server.log     # 1 line: "INFO boot ok" — the line STARTS with it
```
Anchors pin the pattern to a position instead of letting it float anywhere in the line. Log formats put the level first and the verdict last, which is exactly what `^LEVEL` and `word$` exploit.
🏷️ `#patterns` `#regex` `#anchors`

---

## 🖍️ Context & output — show more around it, or less of it

### `-A n` — lines AFTER each match
```bash
grep -A 1 'ERROR' proj/app.log    # each ERROR line plus the line that follows it
```
The error is rarely the whole story; what the program did next usually is. Groups separated by non-matching lines are divided by a `--` separator line in the output.
🏷️ `#context` `#output`
### `-B n` — lines BEFORE each match
```bash
grep -B 1 'ERROR' proj/app.log    # the run-up: WARN disk at 81% before one, the lowercase failure before the other
```
The mirror image of `-A`: what led into the failure. On the fixture the two ERROR lines have different preceding lines, so the output carries the `--` group separator.
🏷️ `#context` `#output`
### `-C n` — context on both sides
```bash
grep -C 1 'ERROR' proj/app.log    # 6 lines, NO '--' separator — see why below
```
⚠️ Context groups that touch are **merged**: here the first group covers lines 2–4 and the second lines 5–7, and since line 4 and line 5 are adjacent, grep prints one continuous block with no separator. A missing `--` does not mean one match — it means the matches were close together.
🏷️ `#context` `#output` `#gotcha`
### `--color` — highlight the match itself
```bash
grep --color=always 'ERROR' proj/app.log | less -R   # the match is wrapped in ANSI color escapes
grep --color=auto 'ERROR' proj/app.log               # color only when output is a terminal
```
`--color=auto` is why grep colors matches interactively but not in pipes; force it with `always` when the consumer renders escapes (`less -R`, a CI log viewer). The exact colors are tunable via the `GREP_COLORS` environment variable.
🏷️ `#output` `#color`
### `-o` — print only the matching part
```bash
grep -oE '[0-9]{2}:[0-9]{2}:[0-9]{2}' proj/app.log   # 08:00:01, 08:01:14, … — 7 timestamps, one per line
grep -o 'ERROR' proj/app.log | wc -l                 # 2 — count OCCURRENCES, not lines (contrast with -c)
```
`-o` turns grep into an extractor: each match is printed on its own line, which makes the output directly countable and sortable. Combined with `-E` (or `-P`, below) it is the standard way to pull fields out of unstructured logs.
🏷️ `#output` `#extract` `#must-know`
### `-H` and `-h` — control the filename prefix
```bash
grep -H 'ERROR' proj/app.log                    # proj/app.log:… — filename even for a single file
grep -h 'ERROR' proj/app.log proj/server.log    # 3 matching lines, no filenames at all
```
By default the prefix appears only with several files (or recursion). `-H` forces it — useful when a script parses `file:line` — and `-h` suppresses it when the names are noise. With `-n` the forms combine: `file:line:content`.
🏷️ `#output` `#files`

---

## 🔧 Pipelines & scripting — exit codes, errors, and stream behavior

### `-q` — say nothing, answer with the exit code
```bash
grep -q 'ERROR' proj/app.log; echo $?      # 0 — a line was selected
grep -q 'ZZZ' proj/app.log; echo $?        # 1 — no line was selected
grep -q 'ERROR' proj/missing.txt; echo $?  # 2 — an error occurred (file not found)
```
The contract every script relies on: **0 = match, 1 = no match, 2 = error**. `-q` prints nothing and quits at the first match, which makes it the only sane choice for `if grep -q …; then …` checks over big files — it never reads past the answer.
🏷️ `#scripting` `#exit-codes` `#must-know`
### `-s` — silence the error messages, keep the verdict
```bash
grep 'ERROR' proj/missing.txt      # stderr: "grep: proj/missing.txt: No such file or directory", exit 2
grep -s 'ERROR' proj/missing.txt   # nothing printed anywhere — and the exit status is STILL 2
```
`-s` suppresses messages about nonexistent or unreadable files — the standard fix for noisy wildcard searches that legitimately hit gaps. ⚠️ It quiets the *message*, not the *outcome*: a script checking `$?` still sees the error. (Both forms verified here with a missing file; an unreadable file behaves the same.)
🏷️ `#scripting` `#errors`
### `-m n` — stop after n matches
```bash
grep -m 1 'INFO' proj/app.log    # only the first INFO line; grep stops reading there
```
On a multi-gigabyte log, "does it contain X, and what did the first one look like" should not cost a full scan. `-m` caps the matches per file and closes the input early; with `-r` the cap applies to each file separately.
🏷️ `#scripting` `#performance`
### Binary files — the notice, `-a`, and `-I`
```bash
grep 'error' proj/data.bin     # prints NOTHING to stdout; stderr gets: grep: proj/data.bin: binary file matches (exit 0)
grep -a 'error' proj/data.bin  # 3 lines printed as if the file were text — NUL bytes and all
grep -rI 'error' proj          # recursive search that silently SKIPS binary files
```
When grep decides a file is binary (a NUL byte is the usual trigger), it suppresses the matching line and instead reports that the file matched — the answer is still "yes", delivered as a one-line notice. `-a` (`--binary-files=text`) forces the lines out, garbage risk included; `-I` makes binary files never match, which is what most recursive searches actually want.
🏷️ `#scripting` `#binary` `#gotcha`
### `-z` — NUL-terminated records, and matches that span lines
```bash
grep -z 'ERROR' proj/records0.txt        # records are NUL-separated: prints just "rec2 ERROR here" (plus a NUL)
grep -z $'started\n2026' proj/app.log    # the whole file is ONE record now — the pattern spans a newline
```
`--null-data` changes the record separator from newline to NUL — the separator `find -print0` uses for filenames, and that some log shippers emit between records. Two consequences, both verified: the matched record is printed with a NUL terminator, and — since newlines are now ordinary characters — a pattern containing a *real* newline (bash `$'…'` quoting; a backslash-n inside plain quotes does **not** work) can match across lines, at the price of printing the entire record.
🏷️ `#scripting` `#binary` `#advanced`
### `--line-buffered` — keep a live pipeline live
```bash
tail -f app.log | grep --line-buffered 'ERROR'    # matches appear as they happen
cat proj/app.log | grep --line-buffered 'ERROR'   # same 2 lines as a plain search — only the timing changes
```
When grep's output is a pipe, it normally buffers in blocks, so a `tail -f | grep …` monitor can sit silent while matches pile up in the buffer. `--line-buffered` flushes after every line. The output content is unchanged; only its timing is.
🏷️ `#scripting` `#pipelines` `#streaming`
### `-P` — Perl regex (PCRE): lookahead and lookbehind
```bash
grep -Po '(?<=usage at )[0-9]+' proj/app.log   # 81 — the number, without the "usage at " prefix
grep -Po 'ERROR(?= timeout)' proj/app.log      # ERROR — only where " timeout" follows it
```
`-P` hands the pattern to a Perl-compatible engine, unlocking lookarounds that can assert context *without consuming it* — the cleanest way to extract "the number after this label". ⚠️ It is a GNU extension that depends on the build: verified working on this machine's GNU grep 3.11, but the BSD grep shipped with macOS has no `-P` at all, and some minimal builds answer "conflicting matchers specified" or fail — test before you ship a script that uses it.
🏷️ `#patterns` `#pcre` `#advanced`
### Patterns that start with `-` — use `-e` or `--`
```bash
grep -- '-v' proj/notes.txt      # finds "the -v flag inverts the match" — -- ends option parsing
grep -e '-v' proj/notes.txt      # same result: -e marks the next argument as the pattern
grep '-v' proj/notes.txt         # WRONG: -v is read as the invert flag, and the search goes sideways
```
A pattern beginning with a dash is parsed as an option, not a pattern. Both fixes are one token long; forgetting them produces the special confusion of a command that runs fine and answers a different question.
🏷️ `#scripting` `#gotcha` `#must-know`

---

## 🧭 grep vs ripgrep — who does what

| The job | Reach for |
|---|---|
| Portable scripts, CI images, any Unix since forever | `grep` — it is already there, with stable, documented semantics |
| Exit-code contracts (0/1/2), `-z` records, PCRE via `-P` | `grep` — ripgrep's flags differ, and its `-P` only exists in PCRE2-enabled builds (its default engine is the Rust regex crate) |
| Interactive search of a source tree, fast, respecting `.gitignore` | `ripgrep` (`rg`) — parallel, skips hidden/binary/ignored files by default |
| Searching *everything* including ignored and binary files | `grep -r`, or `rg --no-ignore --text` with the defaults explicitly undone |

`ripgrep` wins on speed and defaults for interactive code search — it behaves like `grep -r` with a good `.gitignore` filter built in. `grep` wins on *predictability*: identical flags in a container, a server and a rescue shell, an exit-status contract scripts have relied on for decades, and record games (`-z`, `-a`, `-I`) that a search tool aimed at source code does not play.

## ⚠️ Implementation notes — GNU vs BSD

Every entry above was run and verified on **GNU grep 3.11**. The differences that bite:

- **`-P` is GNU-only and build-dependent** (verified working here). The BSD grep shipped with macOS does not know the option at all; a script using it fails loudly there — better than silently, but still a porting wall.
- **Binary-match notices went to stderr in every test here**, in the form `grep: <file>: binary file matches`. Whether the matching line is suppressed follows the GNU manual (output is suppressed once binary data is seen); piping stdout alone will never show you the notice — check stderr or use `-a`/`-I` to choose the behavior yourself.
- **`-r` vs `-R` symlink behavior above is the GNU manual's contract**, verified on the fixture: `-r` follows only command-line symlinks, `-R` follows all of them. BSD grep's recursion flags are spelled the same but check the local man page before relying on corner cases.
- **To find out which grep you have:** `grep --version` — GNU answers with a version line ("grep (GNU grep) 3.11" here); BSD grep on macOS answers with its own banner (or errors on `--version` on very old releases).

## Contributing

Found a wrong command, an outdated flag, or a bad translation? Pull requests are welcome — keep entries to the format above (one-liner / bash block / tags) and **only real, verified commands; when in doubt, leave it out**.

## License

MIT © 2026 Guangyi Zhao. See [LICENSE](LICENSE).
