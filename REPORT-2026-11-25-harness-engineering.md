# Report: harness engineering, applied to the command deck

**Written by:** Devin
**Date:** 2026-11-25
**Source:** mirku@mirku21, "Harness Engineering: How to Build AI Workflows That Never Fall Apart"
(https://x.com/mirku21/status/2098836008468992471), relayed by the owner via alice.

The article argues that when an agent fails, the fix is almost never a better
prompt — it is a better operating environment around the model. It splits a
reliable AI system into three layers and lists five mechanical rules for a
personal harness. This report maps each idea onto what the command deck
already does and where the gaps are.

## The three layers, mapped

| Article layer | What it is | What we already have |
| --- | --- | --- |
| Harness (environment + memory) | Filesystem, env vars, durable files outside the weights | comms-deck itself: seat files, the DATA directory, notes as durable messages, `wake`, `status`, `spent` |
| Loop (evidence + convergence) | Objective sensors between attempts | `self-test` suites, `review-check`, `cite-check`, CI, sim-deck `checks/` |
| Graph (work allocation) | Isolated contexts per unit of work | load-balancer task board, seat worktrees, per-PR review boundaries |

We independently built all three layers. The article's value to us is not the
concepts — it is the checklist for where each layer is still soft.

## The five jobs, gap by gap

### 1. Turn every request into a bounded contract

**Article:** give the model a deliverable, constraints, non-goals, and
acceptance criteria *before* generation, so the criteria — not the model —
decides when the task is done.

**Where we are:** delegated work currently travels as prose notes on the
deck. "Fix X" notes rarely state non-goals or acceptance criteria, which is
why scope creep and adjacent-task-solving show up in reviews.

**Proposal:** adopt a four-line contract at the top of every delegated note:

```
Deliverable: <exact file/PR/behavior>
Constraints: <what may be touched>
Non-goals:   <what must not be touched>
Done when:   <the check that proves it>
```

The comms-deck task board already has a `Done when`-shaped field in spirit;
adding `non_goals` to the task schema would make claims enforceable rather
than aspirational. Low cost, immediate payoff on the multi-seat handoffs
that caused this week's rework.

### 2. Provide a map, not an encyclopedia

**Article:** a compact root map beats dumping project docs into context.

**Where we are:** partially done. `AGENTS.md`-style rules and `README.md`
exist per repo, but there is no single map file telling a seat where things
live across the *fleet* — which repo owns what, which seat owns which lane.

**Proposal:** a `MAP.md` at the top of quantum-kittens (the fleet's shared
repo) listing: repos and their roles, seat lanes, where reports/proposals/
reviews live, and which checks gate merges. Seats already read
quantum-kittens; one more file is cheap insurance against every seat
re-deriving the same repo topology.

### 3. Externalize memory into durable files

**Article:** chat history is scratch space; decisions belong in files that
survive session wipes and model switches.

**Where we are:** mostly done — `CHANGELOG.md`, `REPORT*.md`,
`proposals/`, `LAUNCH.md`, `CREDITS.md` are exactly this pattern. The deck's
notes are also durable.

**Gap:** there is no append-only *decisions* log. CHANGELOG records what
shipped, not why alternatives were rejected — which is why reviews
periodically re-litigate settled questions (e.g. the broadcast→task
semantics debate).

**Proposal:** `DECISIONS.md` in comms-deck and quantum-kittens: one line per
decision — date, decision, reason, rejected alternative. The Wave-1 review
alone produced three decisions worth recording (piece 2 superseded, #75
rebase required, cite-check strict path).

### 4. Build sensors before granting autonomy

**Article:** "write bug-free code" changes nothing; a command that outputs
exit code zero gives ground truth.

**Where we are:** this is our strongest layer. Every repo now has a
self-test; comms-deck has `review-check` (Reviewed-by + SHA enforcement)
and `cite-check`; sim-deck has `checks/rulebook.py` + `contact_sheet.py`;
GitHub CI runs the suites.

**Gap:** sensor coverage is uneven across repos. quantum-kittens itself has
no self-test — proposals are reviewed by humans/seats only. A trivial
`proposals --self-test` (every proposal file has required sections, every
REPORT has a date) would close that.

### 5. Apply the dual-encoding rule

**Article:** state the rule in English *and* put a mechanical gate that
blocks execution when it breaks.

**Where we are:** several live examples already — vibe-approver (prompt says
"allow once"; the gate is that the helper only ever sends `y`, never
"always"), review-check (the rule "nobody merges their own PR" is enforced
in CI, not just convention), cite-check (citations verified mechanically).

**Gap:** the dual-encoding pattern is applied ad hoc. Worth naming it as a
house rule: *any invariant that matters gets both a sentence in the docs and
a check in `self-test` or CI.* The sim-deck rulebook is a good template —
every rule in PLAN.md has a corresponding check function.

## Two-pass prompting for reviews

The article's Pass 1 / Pass 2 workflow (draft, then adversarial review as a
skeptical senior reviewer) describes what our review protocol already does
implicitly: author opens PR, a *different seat* reviews, `review-check`
enforces the signature. Making it explicit would improve consistency:

- Reviewers currently write findings in free prose. A fixed skeleton —
  "verified by running X", "reproduced bug: …", "non-blocking: …" — would
  make reviews comparable across seats and easier for the owner to scan.
- The two-pass idea also applies *within* a seat: before posting a review,
  re-run the author's self-test on the PR head rather than trusting the
  report. (We do this already; writing it down makes it the standard.)

## What to actually do

In priority order:

1. **DECISIONS.md** in comms-deck + quantum-kittens — cheapest durable win.
2. **MAP.md** in quantum-kittens — one file, helps every seat including
   non-deck agents (mystra reads the repos cold).
3. **Contract headers on delegated notes** — start with the load-balancer
   task schema (`non_goals` field) or a convention note in comms-deck docs.
4. **Self-test for quantum-kittens proposals** — small script, big coverage
   gap closed.
5. **Review skeleton** — agree the format on the deck, then encode it in
   `review-check` if it stays stable.

None of this is urgent; all of it is cheap. The article's core claim —
"inspect the operating system around the model" — is already our working
method. These are the places the OS has soft spots.
