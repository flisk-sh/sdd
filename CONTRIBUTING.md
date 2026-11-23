# Contributing to SDD

## Contributor flow: spec first, merge queue lands it

This is how a change gets into flisk-sh/sdd, and the flow the plugin's skills teach for every
flisk-sh repo. It comes from what went wrong in 367 merged PRs
([`skills/_shared/flisk-lessons.md`](./skills/_shared/flisk-lessons.md)).

1. **Open an issue or a spec first** for anything bigger than a typo. Run `/sdd:classify-size`; an
   S change can go straight to a PR, an M or bigger needs `/sdd:specify` and the spec linked from the PR.
2. **Fork, branch, and sign off every commit** (`git commit -s`). Unsigned commits cannot be merged
   and history cannot be rewritten later; maintainers have had to re-apply whole PRs for this
   (flisk-sh/flisk#5288, #5289).
3. **Run the gates locally** — `python3 scripts/validate_plugin.py` here, the repo's own gates
   elsewhere. Do not count on a bot review: on small repos CodeRabbit skips and LlamaPReview runs out
   of quota, so your own checks are the review.
4. **Self-review with `/sdd:review`** and walk
   [`flisk-review-checklist.md`](./skills/review/references/flisk-review-checklist.md). Put the
   evidence (test output, screenshots, green run links) in the PR body.
5. **Open the PR.** CI (`validate`) must be green. A maintainer reads it once, then the merge queue
   lands PRs one at a time on top of current main. If the queue rejects it, rebase and push; don't
   open a duplicate PR.
6. **Your name stays on it.** If a maintainer has to re-apply your change, you are credited with
   `Co-authored-by`.

See [`docs/flisk/worked-prs.md`](./docs/flisk/worked-prs.md) for small merged PRs that show the
shape of a good change.

## Adding or editing a skill

A skill lives in `skills/<name>/` and is the **source of truth** for its stage.

1. **`SKILL.md` is a short spine.** Frontmatter (`name` + a third-person `description` with
   3–5 trigger phrases, EN plus 2–3 UA) then a numbered Protocol. Keep it lean — target well
   under ~140 lines. Heavy detail goes in `references/`; output scaffolds go in `templates/`.
2. **Don't duplicate shared logic.** The 4-state Socratic machine, the clean-context critic,
   the size matrix, and the `AskUserQuestion` style live once in `skills/_shared/`. Reference
   them with a relative link and keep only a short per-skill **delta** (your decision-types,
   your section list, your F6 specialization).
3. **Stay stack-agnostic.** No hard-coded language, tracker, test framework, or load tool.
   Detect what the repo uses, or name the detected tool as «whatever your repo already uses».
4. **Gate your inputs.** If a prerequisite artifact is missing, hard-refuse with a pointer to
   the skill that produces it.
5. **One level of `references/`.** No nested reference folders.

## Subagents

Engine subagents live in `agents/*.md` with `name` / `description` / `model` / `effort`
frontmatter and a system prompt that instructs them to read upstream artifacts directly. Each
agent pins the **role-fit tier alias** per
[`skills/_shared/agent-roster.md`](./skills/_shared/agent-roster.md) — `haiku` for scan, `sonnet`
for execution, `opus` for judgment — overridable at dispatch (the `judgment_model` switch, the
`model_<role>` keys, env). `install.sh` rewrites the generated Codex / Cursor agent copies to
`model: inherit`.

## Before you open a PR

Run the validator locally — it's the same gate the `validate` GitHub workflow runs, and it now
enforces the plugin's **conventions**, not just its structure:

```bash
python3 scripts/validate_plugin.py
```

It checks that the plugin + marketplace manifests agree on name / version / description, that the
version is semver, that every skill and agent carries its required frontmatter (and that `_shared/`
stays reference-only), **and** the consistency invariants in the checklist below. It also greps for
references to the excluded legacy dirs. (The check count is a moving target — CI asserts exit 0, not
a fixed number.)

### Pre-PR checklist

- [ ] **`python3 scripts/validate_plugin.py` passes** (exit 0).
- [ ] **One canonical source / DRY.** Shared logic — the Socratic machine, the critic, the size
      matrix, the ask-style, the surface taxonomy, the handoff block — lives once in
      `skills/_shared/`. Link to it with a relative path and keep only your per-skill *delta*; never
      copy a `_shared/` table (e.g. the surface taxonomy) into a `SKILL.md`.
- [ ] **Stack-agnostic.** No hard-coded language, tracker, test framework, or build/load tool —
      detect what the repo uses, or name it «whatever your repo already uses».
- [ ] **Every skill ends with the handoff block** ([`skills/_shared/handoff.md`](./skills/_shared/handoff.md))
      as its final step.
- [ ] **Invocation form is `/sdd:<name>`** — the namespaced form, never the hyphenated `/sdd-<name>`.
- [ ] **Relative links resolve.** A relative Markdown link is a real file. The one exception is a
      template-runtime path (`../spec.md`, `../sad.md`, `../contracts/…`, …) that resolves only inside
      a generated `docs/features/<slug>/` folder — those are allowlisted in the validator.
- [ ] **References in `references/`, templates in `templates/`** — one level deep, no nested folders.
- [ ] **Skill frontmatter never pins an entitlement-gated tier** (`model: opus` / `model: fable`) —
      frontmatter executes before any settings are read, so an account without that tier hard-fails
      at skill start; skills declare `model: inherit` and the validator fails otherwise.
- [ ] **No version-pinned model-capability claims** (like «only on Opus 4.8») — they rot with every
      model release; phrase by tier / by what the resolved model supports.

### Behaviour evals (on-demand — NOT in CI)

`evals/` holds end-to-end skill-behaviour scenarios (they invoke `claude -p` headlessly and an
LLM judge — they cost tokens and are non-deterministic, so they never run in CI). When you change
a skill's *protocol* (gates, routing, artifact shape), run the closest scenario locally:

```bash
./evals/run.sh design-gate-refusal    # or: specify-happy-path, classify-size
```

See [`evals/README.md`](./evals/README.md) for prerequisites and how to add a scenario.

## Releasing

1. Bump the version in **all four** manifests — `.claude-plugin/plugin.json`,
   `.claude-plugin/marketplace.json`, `.codex-plugin/plugin.json`, `.cursor-plugin/plugin.json` —
   the validator fails on any mismatch.
2. `python3 scripts/validate_plugin.py` → exit 0; push to `main`; tag `vX.Y.Z`.
3. Claude Code and Codex pick the release up straight from git (`/plugin install sdd@sdd` +
   `/reload-plugins`; `codex plugin marketplace upgrade sdd`). The `install.sh` path always
   downloads `main` (or `--ref vX.Y.Z`). Only the Cursor **marketplace** listing goes through a
   review — see below.

### Publishing to the Cursor marketplace

Cursor plugins are distributed as public git repositories and **manually reviewed** — both the
first listing and every subsequent update:

1. **Pre-check locally.** Copy the repo to `~/.cursor/plugins/local/sdd`, restart Cursor (or run
   **Developer: Reload Window**), type `/` in the chat and confirm the skills appear.
2. **The repo already satisfies the format.** `.cursor-plugin/plugin.json` is the manifest (only
   `name` is strictly required; we also ship displayName / version / description / author /
   license), and `skills/` + `agents/` are auto-discovered from the repo root. A
   `.cursor-plugin/marketplace.json` is only needed for multi-plugin repos — not here.
3. **Submit** the repo URL at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish)
   and wait for the review. After approval the plugin appears on cursor.com/marketplace and in
   the in-app marketplace panel; users install it from there, project- or user-scoped.
4. **Updates are re-reviewed** before the marketplace refreshes; the `install.sh` git path keeps
   tracking `main` immediately, review or not.
