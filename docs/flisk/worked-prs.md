# Worked PRs: small real changes, read as specs

Real merged PRs from flisk-sh repos, small enough to read in a minute. Use them to see what a
`specify → tasks → plan-tests → implement → ship` pass should produce for a change this size
(most are `classify-size` S). Each one names the problem, the change and the proof.

| PR | Problem | Change | Proof |
|---|---|---|---|
| [flisk-sh/flisk#5374](https://github.com/flisk-sh/flisk/pull/5374) | `wait --timeout` help printed the default twice, failing the `help-option-defaults` check on main | one line in the `wait` command | regression test + the check |
| [flisk-sh/flisk#5363](https://github.com/flisk-sh/flisk/pull/5363) | typos check flagged a made-up package name in tests, blocking every PR | one allowlist entry in `.github/_typos.toml` | the CI typos job going green |
| [flisk-sh/flisk#5350](https://github.com/flisk-sh/flisk/pull/5350) | machine-readable docs gave the wrong error code for an unknown tool | one line in `docs/reference/machine-readable-cli.md` | docs-only, no behaviour change |
| [flisk-sh/chat#307](https://github.com/flisk-sh/chat/pull/307) | numbers dropped from streamed answers | `src/say-stream.js` | new `test/stream-digits.test.js` |
| [flisk-sh/chat#299](https://github.com/flisk-sh/chat/pull/299) | a prompt fix (#298) did not stop the model answering who-cited questions | routing moved into `src/lookup.js` | `test/mentions.test.js` |
| [flisk-sh/chat#397](https://github.com/flisk-sh/chat/pull/397) | `{"name": ...}` replies were not read as actions | `src/loop.js` parser | `test/answer-guard.test.js` |
| [flisk-sh/gel#58](https://github.com/flisk-sh/gel/pull/58) | CI jobs without `timeout-minutes` can hang | new `workflow-timeouts` guard | `cases.json`; registration fixed later in #74 |
| [flisk-sh/gel#41](https://github.com/flisk-sh/gel/pull/41) | secrets placed behind `VITE_` / `NEXT_PUBLIC_` leak to the browser | new `public-env-secrets` guard (outside contributor) | `cases.json` |
| [flisk-sh/fliskos#177](https://github.com/flisk-sh/fliskos/pull/177) | site had no `/.well-known/security.txt` | new file + `_headers` | `test/website.test.js` |
| [flisk-sh/fliskos#174](https://github.com/flisk-sh/fliskos/pull/174) | no skip-to-content link for keyboard users | `index.html`, `404.html`, `site.css` | new `test/skip-link.test.js` |

## Pairs worth reading together

These show what the spec missed the first time (see
[`skills/_shared/flisk-lessons.md`](../../skills/_shared/flisk-lessons.md)):

- flisk-sh/chat#298 → #299: prompt fix, then code guard (rule 5).
- flisk-sh/chat#385 → #387: phone layout fixed, still broken at 320px (rule 3).
- flisk-sh/chat#383 → #386: deploy workflow added, then made to work (rule 4).
- flisk-sh/gel#58 → #74: guard added, then registered (rule 12).
