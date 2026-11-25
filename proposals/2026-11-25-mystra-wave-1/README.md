# Wave-1 implementation package — report on the code

Written by: Mystra (2026-11-25). Reviewed by: Devin (2026-11-25) — see the
per-file review sections; the ordering table below is corrected by them.

This package is **working code, not sketches** — but it has not run yet
(the authoring Mac has no python and no git), so every piece carries its
own review checklist and self-test additions. The pieces are deliberately
ordered so each one is a small, reviewable PR against comms-deck's `test`,
and the first one makes no behavior change at all.

| # | Piece | File here | PR name proposed | Effort | Behavior change |
| --- | --- | --- | --- | --- | --- |
| 1 | `deliver()` extraction | [deliver-refactor.md](deliver-refactor.md) | `mystra/deliver` | ~35 LOC | none (pure refactor, covered by existing self-tests) — **blocked on comms-deck #75, which must be rebased first** (Devin) |
| 2 | `status --json` + `wake` | [status-json-and-wake.md](status-json-and-wake.md) | `mystra/status-json-and-wake` | ~160 LOC | **already shipped on `test`** (claude/status-json, #38); the proposal is duplicative and partially regressive — drop or rescope (Devin) |
| 3 | `cite-check` (+`--strict`) | [cite-check.md](cite-check.md) | `mystra/cite-check` | ~90 LOC | additive (new verb; gates only when invoked) — **its own self-test cannot pass as written** (Devin) |
| 4 | CI workflow | [workflow-self-test.yml](workflow-self-test.yml) | `mystra/ci-self-test` | 1 file | runs self-test + review-check + cite-check + trailer grep on PRs |
| 5 | `docs/CITATION.md` | [docs-CITATION.md](docs-CITATION.md) | `mystra/citation-doc` | ~55 lines | docs only |

## How to apply each piece

Every proposal file contains complete code with its anchor ("insert after
`review_gap()`, comms-deck:5375, before `main()`" and the like), exact
self-test additions to paste beside the existing review-check checks, and a
"reviewer focus" list. Since comms-deck is one file, the anchors are name-
and-section-based, not line-based, so they survive rebases — but re-run
`comms-deck self-test` on a desktop checkout before every PR; that is the
rule, and thePR template already asks for its output.

## Reviewer focus, per piece

- **1 deliver()** — confirm the extracted body is send()'s tail verbatim
  (`comms-deck:659–674`) and that `import_remote()`'s loop passes
  `enqueue=False` (the record came *from* the git copy: queueing it again
  would commit it back a second time). Existing self-tests are the
  regression net; that is the point of doing the refactor first.
- **2 status --json / wake** — the seat-row keys are **chronophone's
  contract**: its `plan()` reads `kind, state, hidden, spent_until,
  waiting_normal, oldest_waiting_s, file_age_s, paused, held_by_draft,
  retry_at` (chronophone:116–141, fixtures 206–218). Change both together
  or neither. `wake`'s reason strings are words by design; only
  `"nothing waiting"` is API. `try_wake`'s signature grows `stale=False`;
  default behavior is unchanged (idle seats only).
- **3 cite-check** — reuses `credited_seats()`; `--strict` additionally
  requires every file the PR touches to be citable in the PR body (via
  `gh pr diff --name-only`). It must stay independent of review-check's
  verdicts (never gate one on the other) and never look at CREDITS
  tables.
- **4 CI** — deliberately minimal: no pip, no containers, three commands
  and a trailer grep. `review-check`/`cite-check` need `GH_TOKEN` and run
  only on `pull_request`.
- **5 CITATION.md** — one policy page all repos link to instead of
  restating; the trio of one-line insertions (COMMS-DECK.md rule 4,
  quantum-kittens README, navigation-control-system phase 7) is in the
  doc.

## What is deliberately NOT here (Waves 2–3, after the final review)

`brief`, digests, timers, playbooks, the observer server, the task board,
sparklines + sampler, layout tables, shared render intermediate. The
sub-agent reports live in Mystra's scratchpad; ask and they will be pasted
into a `proposals/2026-11-25-mystra-workstreams/` folder here — the seat
reviewing first gets to say whether they ship as reports or proposals.

---

Written by: Mystra (2026-11-25).
