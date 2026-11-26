# TRIGGER — Claude, Devin, Vibe: read this when you find this file

You are three seats on the comms-deck team. Mystra (the first remote seat;
her join PR is #75 on comms-deck) did a full ecosystem review on 2026-11-25
at the owner's request, and left an **implementation package** in this repo.
The owner's instruction: you review it yourselves, suggest and make changes,
and only implement after a final review — co-authored by the three of you,
with your real conversation as its evidence — is written here.

## What to read, in order

1. This file.
2. [snapshots/2026-11-25-mystra-session.md](snapshots/2026-11-25-mystra-session.md) — what happened and why.
3. [proposals/2026-11-25-mystra-wave-1/README.md](proposals/2026-11-25-mystra-wave-1/README.md) — the code report; then the four proposal files it points at
   (`cite-check.md`, `status-json-and-wake.md`, `deliver-refactor.md`,
   `workflow-self-test.yml`, `docs-CITATION.md`).

The full review behind them (Parts 1–14) is in Mystra's session scratchpad
on the owner's Mac; ask the owner or Mystra to paste any part you want.

## Your work, as three separate seats

1. **Review alone first.** Each seat judges the package independently
   before reading the others' notes — review-check assumes honesty.
   Look hard for what is wrong: order of operations, Rule 8 violations,
   contract drift against chronophone, anything that would make Claude's
   self-test lie.
2. **Make your changes.** Edit the proposal files in this repo (branch or
   fork it, one PR per seat, `Written by: <you>`, and never merge your own
   PR). Change what you disagree with; the diff is the discussion.
3. **Talk in comms-deck.** Take the disputed parts to the deck as notes
   (`--from <you>`), with the owner copied where a decision is the owner's.
   The conversation is not overhead — Part 12 of the final review needs it.
4. **Write the final review.** When the three of you agree (or agree to
   disagree in writing): `REVIEWS/2026-11-25-mystra-final-review.md`, here,
   co-authored — `Written by: Claude, Devin, Vibe` — with each seat's
   contribution named and **lengthy verbatim excerpts of your real
   comms-deck conversation** about this package (quote the actual notes,
   with their Ids). Run `comms-deck review-check` on the PR that carries it.
5. **Only then implement.** The code moves from the proposals into real
   comms-deck PRs (`status --json` + `wake` + the `deliver()` refactor
   first; then cite-check + CI; then the owner's `app-seats` line for
   Mystra per the two-key rule).

## Standing rules, unchanged

Cite everything (docs/CITATION.md once it lands; COMMS-DECK.md rule 4 until
then). No secret ever enters a repo or a note. The owner merges; you don't.
And if the package is wrong, say so here and in comms-deck — that is the
point of a review.

---

Written by: Mystra (2026-11-25).
