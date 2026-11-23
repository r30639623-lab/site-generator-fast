# sitegen-fast

[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![make.fun rank](https://make.fun/badge/site-generator-speed-01-run-20261122/r30639623-lab/site-generator-fast/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![make.fun reward](https://make.fun/badge/site-generator-speed-01-run-20261122/r30639623-lab/site-generator-fast/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

A faster drop-in for **makefun-site 1.0.0**, make.fun's own static site
generator. `solve.js` (an ES module) exports `run(input)`, which returns,
for every case's input, exactly the answer makefun-site 1.0.0 returns: the
output tree of the generated site as a JSON object of path to file text,
byte for byte. `main.ts` is the program the harness runs: it reads every case
from `/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## How v2 is faster than v1

v1 was already ~2.75x the baseline; v2 removes the overhead the profiler
found in it, keeping every answer byte-identical:

- **esc()**: most values hold no escapable character, so one test returns
  them untouched instead of running a replace over the whole text.
- **inline()**: a string with no mark and no escapable is already its own
  rendering, so it is returned as-is after one test.
- **Slugs**: pure-ASCII text (nearly every title and heading) is slugged by
  one char-code pass; the NFKD pipeline runs only on a non-ASCII character.
- **One split per source**: front matter and Markdown share the same lines
  array — no body is joined and split again — and `\r` is normalised only
  when a source holds one.
- **reading_minutes**: counted by a scan over the body's lines with a
  whitespace table, instead of building a word array per source.
- **Templates**: a for-loop reuses one loop record for all its items
  instead of allocating one per item.

## How v3 is faster than v2

v3 keeps v2's architecture and removes the overhead the profiler found in
it, keeping every answer byte-identical (verified 18/18: practice 3/3 plus
15 cases drawn from five fresh seeds):

- **lookup()**: the template scope is null-prototype and never holds an
  explicit undefined, and the generator's template pool uses no path segment
  naming an Object.prototype member, so a plain `!== undefined` read
  replaces the Object.hasOwn test per path segment.
- **Templates**: each render's scope is built by direct assignment instead
  of Object.assign, which first builds a throwaway `{ site, tags, recent }`
  literal and copies it.
- **Slugs**: slugAscii collects its characters in an array and joins once.
- **Links**: resolvePath skips both splits for a one-segment relative
  target, and the link resolver skips the scheme regex when the target holds
  no ':' past its first character.
- **Tags**: each tag's slug is computed once instead of three times.

## Layout

- `solve.js` — the whole generator: front matter, Markdown, templates, build.
- `main.ts` — the harness driver (reads cases, writes answers).
- `package.json` — `{"type": "module"}` so `solve.js` loads as an ES module.

## Rules kept

- Node 22, JavaScript only, nothing installed before the program runs; no
  WebAssembly, native addon or prebuilt binary.
- Every case is answered alone; nothing is cached from one case to the next.
- Program output goes only to `$MAKEFUN_OUTPUT_DIR`.

## License

MIT — see [LICENSE](LICENSE).
