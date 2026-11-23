# Flisk lessons: rules learned from 367 merged PRs

> **Reference-only.** Not a skill. Read by `tasks`, `implement`, `review` and `ship` when the repo
> is a flisk-sh repo or uses GEL (a `docs/gel.json` or a workflow calling `flisk-sh/gel`).
> Every rule below cites the merged PRs that taught it. A rule without evidence does not belong here.

Source: the last merged PRs of `flisk-sh/flisk` (100, 2026-11-22..23), `flisk-sh/chat` (100,
2026-11-17..23), `flisk-sh/gel` (80, all) and `flisk-sh/fliskos` (87, all), mined 2026-11-23.

## What the history says

- **Fix ratio is high.** chat: 42% of merged PRs are `fix:`, and its own retro measured 47%
  (flisk-sh/chat#393). Many fixes are a second PR for the same change. The cheapest place to stop
  that is the spec and the task list, not review.
- **Nobody is reviewing.** CodeRabbit skips repos under 10 stars, LlamaPReview runs out of its daily
  quota, and human reviews are close to zero in chat, gel and fliskos. The author's own checks
  are the review. `review` in this plugin is not optional on flisk-sh repos.
- **Outside contributors are real.** flisk at least 15%, fliskos up to half (44 of 87, some
  authors' org membership unknown), chat 11%, gel 6 PRs. They hit the DCO and the gates first.

## Rules (apply before coding and before opening the PR)

### 1. Find every place the thing is set, before changing it
One rule, many PRs: a header, a CSP, a policy or a name is usually set in more than one place.
List them all in the spec (`specify` / `design`) and change them in one PR.
Evidence: page framing took three PRs in chat (flisk-sh/chat#376, #384, #391) and the same change
was merged twice in fliskos (flisk-sh/fliskos#180, #181). After a rename, sweep the old name from
package, bundle id and README (flisk-sh/fliskos#128, #129, #130).

### 2. Test the class, not the case
A regression test for one input lets the next input of the same shape through. Write the test for
the class (all structured-output shapes, all widths, all stacks).
Evidence: flisk-sh/chat#393 (retro), raw model output shapes fixed one by one in
flisk-sh/chat#392, #397, #307; an assertion that only checked one stack path (flisk-sh/flisk#5382).

### 3. UI changes are verified on the rendered page at 320px and 375px
A unit test or CSS diff is not proof. Take the screenshot at both widths.
Evidence: flisk-sh/chat#385 then #387 (still wrapped at 320px), #389 then #390.

### 4. A changed CI or deploy workflow is run end to end before merge
Evidence: flisk-sh/chat#383 added deploy, #386 fixed it (dependencies not installed);
flisk-sh/fliskos#137, #138 (cold runner needs longer waits and stdin open).

### 5. When a prompt fix is not enough, guard it in code
Model behaviour is not a contract. Route or validate in code and test that code.
Evidence: flisk-sh/chat#298 (prompt only) then #299 (routing moved to `src/lookup.js`).

### 6. Commit generated output with its source
flisk commits the compiled `cli/dist/`; GEL commits `scores.json`. Rebuild and commit them in the
same PR, and rebase on main first so the check compares against the newest source.
Evidence: flisk-sh/flisk#5373 (`check:dist` red on main), flisk-sh/gel#95 (scores red after
concurrent merges).

### 7. Write Sonar-clean code the first time
`[[ ]]` in bash, explicit comparator for `.sort()`, no backtracking regexes, assert with the test
framework's own `expect` (a wrapper counts as no assertion), no nested ternaries.
Evidence: flisk-sh/flisk#5378, #5395, #5401, #5385, #5422; GEL turned these into the
`sonar-patterns` guard (flisk-sh/gel#88, #92, #96).

### 8. Run the repo's scoreboard and gates locally before the PR
On flisk: `make scores` and `npm --prefix cli run lint`. A gate that drops turns every open PR red.
Evidence: MEMORY.md trimmed twice for the same `memory-size` gate (flisk-sh/flisk#5294, #5366).

### 9. Keep mocks faithful and helpers shared
A mock that drops real exports breaks unrelated tests; copied helpers drift.
Evidence: flisk-sh/flisk#5382, #5369, #5372.

### 10. Docs and help text move with behaviour
Evidence: flisk-sh/flisk#5350 (docs named the old error code), #5374 (help repeated a default and
failed `help-option-defaults`).

### 11. Sign off every commit (DCO)
History cannot be rewritten after the fact; maintainers have had to re-apply contributor PRs.
Evidence: flisk-sh/flisk#5288 (replaces #5116), #5289 (replaces #5121).

### 12. New guard or check: register it, and scan the whole tree
Evidence: flisk-sh/gel#58 then #74 (guard not registered); #6, #19, #25 (guards that only
scanned the diff).

### 13. Every new third-party origin needs its CSP and frame rule in the same PR
Evidence: flisk-sh/fliskos#156, #157, #170, #180, #181, #182.

### 14. Smoke-test dev entry points in CI
Evidence: flisk-sh/chat#340 (`wrangler dev` crashed on a missing var);
flisk-sh/fliskos#110, #118 (a GUI-launched macOS app has no shell PATH).

### 15. The one-command demo is P1 on flisk
Anything touching migrators, images or `flisk demo` runs the demo journey before merge.
Evidence: flisk-sh/flisk#5376 (migrator missing a package, demo journey red).

## How skills use this file

- `tasks`: add a task for rules 1, 6 and 10 when the change touches config, generated output or a
  user-facing message.
- `plan-tests`: rule 2 and 3 shape the test plan.
- `review`: run [`../review/references/flisk-review-checklist.md`](../review/references/flisk-review-checklist.md).
- `ship`: rules 4, 6, 8, 11 are the pre-PR gate; see `CONTRIBUTING.md` → Contributor flow.
