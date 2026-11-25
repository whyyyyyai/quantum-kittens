# Rule 4, verifiable: who wrote what — and where the rule lives

The team's authorship rule, and the only page that states it. Other repos
link here; they do not restate it (a restated rule drifts — that lesson is
Part 4 of the 2026-11-25 review, and it applies to policy too).

## The rule, in one amended breath

> **Rule 4 (amended).** Credit is not a courtesy, it is a citation
> requirement. Every artifact names the agent(s) and program(s) that made
> it — commits by trailer, files by line, notes by sender, actions by
> signing program. A PR that cannot say who wrote it does not merge. AI
> use is permissible in all circumstances because authorship is never in
> doubt.

(That last clause is the owner's AI policy, made verifiable instead of
merely labeled: the labeling is the policy.)

## The four citation surfaces

| Surface | What satisfies it | What checks it |
| --- | --- | --- |
| git commits | one `Co-Authored-By:` trailer per agent (the house format: `Mistral Vibe <vibe@mistral.ai>`, `mystra-vibe <360961110+mystra-vibe@users.noreply.github.com>`) | CI's trailer grep (piece 4) |
| files & pull requests | `Written by:` / `Reviewed by:` lines — at a file's top in its "read this first" comment, or a doc's foot; the PR body lists per-file authors when using `--strict` | `comms-deck review-check` + `comms-deck cite-check` (pieces 3), CI |
| notes | the mailbox itself: `From:` / `Machine:` / `Id:` headers — nothing to do, it is by design (comms-deck `send()`, `message_file()`) | nobody; it cannot fail |
| runtime actions | the acting program signs its notes and log lines: `--from chronophone`, `[comms-deck]`, `push-courier`, the future timers' `--from` | the deck's log reader; a missing signature is a review finding |

## Where the words are allowed to live

- **This file** — the rule and this table. Everything links here.
- COMMS-DECK.md **rule 4** keeps its one-line summary and gains "see
  docs/CITATION.md" — insert, after the present rule 4 paragraph:
  `The rule is enforced, not assumed: see docs/CITATION.md for what
  satisfies it and which gate checks which surface.`
- **quantum-kittens** (the easy guide, both copies until they are one):
  one line in the rules area — `Say who wrote what: commits by trailer,
  files by "Written by:", PRs checked by cite-check (docs/CITATION.md).`
- **navigation-control-system**: one acceptance check in phase 7's
  "Prove it" list — `Everything written during the rebuild carries the
  builder's name: a "Written by:" line per file, a Co-Authored-By: trailer
  per commit. cite-check a sample file before you call the phase done.`

Those three insertions are the whole distribution: one home, three
pointers, no restatements.

## What the rule does not ask

It does not ask agents to sign other agents' work (keep their credit),
to credit drafts and wakes and timer ticks in prose (the signature line
is enough), or to reveal the owner's name — private names are redacted at
egress, and a citation never becomes a leak (`looks_secret()` and
`redact()` run before any citing text leaves the machine; docs/CITATION
itself lives in the public repo, so it names AI and programs, never
people).

---

Written by: Mystra (2026-11-25), from the owner's ask on 2026-11-25.
