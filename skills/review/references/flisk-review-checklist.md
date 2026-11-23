# Flisk review checklist

> Stage-2 add-on for [`review-dimensions.md`](./review-dimensions.md) on flisk-sh repos (or any repo
> using GEL). Each item is something a bot, a gate or a follow-up fix PR caught after merge in the
> last 367 merged PRs. The rationale and evidence live in
> [`../../_shared/flisk-lessons.md`](../../_shared/flisk-lessons.md).

A finding from this list cites `file:line` like any other finding. "Not applicable" is a valid
answer; silence is not.

| # | Check | Caught after merge in |
|---|---|---|
| 1 | Every place the changed header / policy / name / config is set is changed in this diff | flisk-sh/chat#376→#384→#391, flisk-sh/fliskos#180+#181 |
| 2 | Tests cover the class of input, not the one reported case | flisk-sh/chat#392, #397, flisk-sh/flisk#5382 |
| 3 | Assertions are scoped to everything that must not change, not one path | flisk-sh/flisk#5382 (CodeRabbit) |
| 4 | UI change has screenshots at 320px and 375px of the rendered page | flisk-sh/chat#387, #390, #388 (LlamaPReview) |
| 5 | Changed workflow was run end to end (link to the green run) | flisk-sh/chat#386, flisk-sh/fliskos#137, #138 |
| 6 | Generated output (`cli/dist/`, `scores.json`) rebuilt and committed after rebase | flisk-sh/flisk#5373, flisk-sh/gel#95 |
| 7 | Sonar patterns: `[[ ]]`, sort comparator, no backtracking regex, real `expect`, no nested ternary | flisk-sh/flisk#5395, #5401, #5385 |
| 8 | Docs, help text and error codes match the new behaviour | flisk-sh/flisk#5350, #5374 |
| 9 | Allowed values / types in a command match the types it maps to | flisk-sh/flisk#5376 (CodeRabbit, Major) |
| 10 | Model output is parsed and guarded in code, never rendered raw | flisk-sh/chat#392, #397, #299 |
| 11 | New guard / check / route is registered, and scans the whole tree | flisk-sh/gel#74, #6, #19 |
| 12 | Tokens and secrets never go over plain http or behind public-env prefixes | flisk-sh/gel#82, #41 |
| 13 | New origin / script / frame has its CSP rule in the same diff | flisk-sh/fliskos#156, #170 |
| 14 | Every commit has `Signed-off-by` | flisk-sh/flisk#5288, #5289 |
| 15 | No duplicate of an already-open or just-merged PR | flisk-sh/fliskos#180/#181, flisk-sh/flisk#5294/#5366 |
