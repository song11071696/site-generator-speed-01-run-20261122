# Faster drop-in for makefun-site 1.0.0

[![site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/song11071696/site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/song11071696/site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

[Bounty page](https://make.fun/bounty/site-generator-speed-01-run-20261122)

`run(input)` in `solve.js` returns exactly the output tree makefun-site 1.0.0's
`build` returns for one case's input — every page, index/tag page,
`build-report.txt` and static file, byte for byte — but faster (about 3x the
house baseline's CPU time on the practice set).

`main.ts` is the baseline driver unchanged: it reads `/input/cases.json`
(`MAKEFUN_INPUT_DIR` outside a container), answers each case alone with `run`,
and writes `/output/answers.json` (`MAKEFUN_OUTPUT_DIR` outside a container).
It keeps `--no-incremental-marking`, as the house baseline does.

## How it is faster

The reference is a faithful pipeline (front matter → permalinks → Markdown →
collections/tags → layouts → report); this entry keeps every rule and every
error string, and streamlines the hot paths:

- Markdown is lexed and rendered in one fused pass (no token objects); inline
  rendering scans for the next special character instead of stepping
  character by character, with plain runs copied as slices.
- Templates are tokenized once per case (tokenize errors anywhere beat parse
  errors anywhere, exactly as the reference orders them) into flat op lists,
  then interpreted into an output array joined once.
- Slugs and long dates are cached per case (titles and tag names repeat);
  cards, excerpts and reading minutes are computed once per post and reused
  across index, tag and recent pages; the URL map is built before bodies render.
- No cross-case state: every cache is reset per `run()` call. No dependencies,
  no network, no clock, no randomness, no WASM, no native code.

## Files

- `solve.js` — the entry (dependency-free ES module, exports `run(input)`)
- `main.ts` — the harness driver (baseline driver, unchanged)
- `LICENSE` — MIT

## Checks run locally

- Differential harness vs the reference `build`: practice set 3/3, plus 9
  cases drawn from the kit generator on 3 fresh seeds, plus ~1,500 adversarial
  fuzz cases (Markdown/front-matter/template edge cases) — all byte-identical.
- `grade-local` equivalent (kit scorer against `practice/reference/expected.json`):
  passed 3/3.
- Determinism: two runs produce byte-identical `answers.json`.
- Repo size is far under the 4 MiB cap.
