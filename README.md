# tangentscraps/lysp

> A port of Ian Piumarta's LYSP — a tiny Lisp interpreter in C — to Pascal
> (Free Pascal). The data structures, printer, and REPL loop are real; the
> reader, evaluator, and garbage collector are elegant stubs. A skeleton
> waiting for its interpreter.
> Repo: https://github.com/tangentscraps/lysp — commits Nov 2012.
> **Keep, document.**

## What it is

[LYSP](http://www.piumarta.com/software/lysp/) is Ian Piumarta's minimal Lisp
interpreter, written in C. The repo keeps the original under `cver/`
(`lysp.c` at 861 lines, plus `gc.c`/`gc.h`, a `Makefile`, `bench.l`, and
`test.l`), and the actual project — `lysp.pas`, 225 lines of Free Pascal —
is a line-by-line port attempt.

What the Pascal port genuinely has:

- **Cell data structures:** a tagged variant record (`t_nil`, `t_int`,
  `t_str`, `t_sym`, `t_cel`, `t_sub`, `t_exp`, plus commented-out foreign-
  function tags `t_fsub`/`t_fexp`/`t_psub` with a "todo: dynamic library
  calls" note) with constructors for each type.
- **A reader dispatch table:** `init_readers` maps all 256 chars to reader
  functions (`read_alpha`, `read_digit`, `read_list`, `read_quote`,
  `read_semi` for comments, …) — the classic Piumarta reader design.
- **A printer:** `print` handles every cell tag.
- **A REPL loop** wired to the `linenoise` library (submodule at
  `git://github.com/tangentstorm/linenoise.git`), with `-->` / `...`
  prompts and a completion hook.

What it does *not* have, despite the commit message "added parse, eval,
main/more prompts, and fleshed out repl":

- every `read_*` function body is **empty**;
- `parse` returns `cell.make_nil` and `true` unconditionally;
- `eval` is the **identity function** (`result := expr`);
- the GC callbacks `mark`/`free` and `init_globals` are **empty**;
- `gc.pas` (7 lines) declares `mark`/`free` callback variables and nothing
  else.

So the port stopped at exactly the hard part: the skeleton parses nothing
and evaluates nothing. The commit history is honest about the shape of the
work — "import of Ian Piumarta's LYSP interpreter" → readme → "garbage
collector stubs" → "setup the reader code" → "added stubs for globals,
repl" → linenoise integration → "pulled the reader type signature up into a
macro" (the `{$define as_reader := ( next : chargen ) : cell }` trick) — and
then it ends, mid-skeleton, November 2012.

(Flagged as unverified: the repo doesn't say *why* the port stopped, and the
commit messages' "fleshed out" phrasing overstates what the code contains —
the reader/eval bodies are empty. Whether it stalled on the reader's
character-generator design or just lost to other projects is not recorded.)

## Tech stack

Free Pascal (`fpc`), with `{$macro on}` and include files; linenoise for
line editing (git submodule, `git://` URL — that protocol is long dead on
GitHub, and the submodule directory is empty in a fresh clone). The `uses
gc, ln;` clause and `{$i xpc.inc}` mean it does not compile from a fresh
clone without fetching linenoise another way. MIT-style hobby code, no
license file in the repo.

## Inventory

| Path | Contents |
|------|----------|
| `lysp.pas` | 225 lines: cell types, reader table, printer, stubs, REPL |
| `gc.pas` | 7 lines: `mark`/`free` callback declarations |
| `cver/` | The original C LYSP: `lysp.c` (861 lines), `gc.c`, `gc.h`, `bench.l`, `test.l`, `Makefile`, `00_README` |
| `Makefile` | `fpc -Fi./lib/linenoise -Fu./lib/linenoise` build; `lysp`/`run`/`subs`/`clean` targets |
| `README.md` | "A port of Ian Piumarta's LYSP interpreter to pascal." + piumarta.com link |
| `lib/linenoise` | git submodule (empty in fresh clone) |

## Surprising / notable

- **The skeleton is well-designed.** The reader dispatch table and tagged
  cell types are a faithful, idiomatic Pascal rendering of Piumarta's
  architecture — whoever finished this would have had a clean foundation.
  The interesting work (filling in ten reader bodies and `eval`) is small
  enough to be a weekend project.
- **Foreign functions were on the roadmap:** the commented-out `t_fsub` /
  `t_fexp` / `t_psub` tags and the "dynamic library calls" todo show the
  port was meant to go *beyond* the C original, not just match it.
- **The completion hook is stubbed too** (`procedure completion` is empty) —
  even the linenoise integration is surface-level.
- **Commit-message optimism:** "fleshed out repl" for a REPL whose parser
  returns nil. A reminder to read the code, not the log.
- No linked streams or videos found in the repo. Verdict at triage: **keep +
  document** (done — this file).

## Verdict

Keep, document. This is the most salvageable of the 2012 interpreter stubs:
the architecture is sound, the original C source sits right there in `cver/`
for reference, and finishing it (readers → eval → GC) is a bounded, well-
defined project. Natural next step if he ever wants a small Pascal/Lisp
project: fill in the reader bodies against `cver/lysp.c`.
