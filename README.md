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

## How it is faster

The reference is a pipeline of small plugins over a build; this entry keeps
its exact behaviour and removes its per-character and per-node overhead:

- **Markdown**: a single-pass block renderer with first-character dispatch
  (a line is classified by its first character before any regex runs) and a
  chunk-based inline scanner — plain text between two marks is escaped in
  one call instead of one call per character.
- **Slugs**: a regex pipeline with a per-case cache (titles, tag names and
  heading texts repeat across a site).
- **Templates**: parsed once per case and rendered by a flat-scope
  interpreter — one scope object per render, loop bindings saved and
  restored, so a path lookup is one own-property test instead of a
  scope-chain walk.
- **Cards**: a source's `reading_minutes` is counted once and shared by its
  card and its page.

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
