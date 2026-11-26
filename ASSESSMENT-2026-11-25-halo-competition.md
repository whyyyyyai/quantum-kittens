# Feasibility assessment: White Circle "Halo" post-training competition

**Written by:** Devin
**Date:** 2026-11-25
**Source:** https://x.com/whitecircle/status/2125274360369815717 and
https://whitecircle.com/halo/competition, relayed by the owner via alice.

## The competition in one paragraph

White Circle is running a 10-day online post-training competition
(Dec 1–10, applications close Nov 30). Participants post-train open models
using Halo, White Circle's open-source framework (SFT, DPO, KTO, GRPO and
friends, with data/tensor/expert/pipeline parallelism). There are 17
tracks — model-family optimization and post-training tracks from OpenAI,
Google DeepMind, Mistral, Thinking Machines, NVIDIA, plus "best use of"	racks for Modal, Baseten, Token Factory, OpenRouter, AWS, Lovable, and
White Circle itself. $1.5M in compute/API credits for accepted
participants, $200k+ cash prizes, top three overall at $30k/$15k/$10k.
Teams of 1–4. Submissions are rerun on the organizers' hardware, must be
open source, and may build on prior work.

## Who "we" are, honestly

The team is one human owner plus a fleet of AI agent seats (claude, vibe,
devin, grok, codex-when-active, gravity, alice, juno, lumo, mystra pending
setup). Our demonstrated strengths: orchestrating agents, code review,
documentation, automation, and building small reliable tools (comms-deck,
sim-deck, quantum-kittens). Our demonstrated gap: nobody on the team has
post-trained a model. No SFT/DPO/RL runs, no GPU cluster experience, no
training-infrastructure track record. The agents can *write* training
configs; they cannot yet *run* them credibly.

## Track-by-track fit

| Track type | Fit | Why |
| --- | --- | --- |
| Model post-train (gpt-oss, Gemma, Mistral, Inkling, Nemotron) | Weak | Core ML-RL work. Doable in principle — Halo + credits remove the cost barrier — but the 10-day window plus zero prior runs makes a competitive entry unlikely. A learning entry is possible. |
| Model optimization (make Halo faster/leaner) | Medium | Systems engineering, closer to our skill set. But judging is by rerun benchmark on organizer hardware; reproducible perf wins in 10 days without Halo experience is ambitious. Open-source requirement helps — merged improvements get credited. |
| "Best use of" infra (Modal, Baseten, Token Factory, OpenRouter, AWS) | Medium | Same engineering bar, judged more on interestingness than leaderboard rank. Our multi-agent pipeline could generate data/judge trajectories via OpenRouter credits — a plausible angle. |
| **Best use of White Circle — "optimize an agent with Halo"** | **Strongest** | This is the only track where our actual asset — a working multi-agent orchestration system with real usage trajectories — is the subject matter. comms-deck produces exactly the kind of agent trajectory data RL post-training consumes. Thematically we are the best-positioned team in the room for this one track. Still requires RL competence, but the *problem* fits us. |
| Lovable track | Weak | UI-tooling track; off-mission. |

## Practical constraints

- **Deadline pressure:** apply by Nov 30 — five days out. Acceptance is at
  White Circle's discretion ("tell them what you've trained, built, or
  published"). Our published artifacts (comms-deck, quantum-kittens,
  sim-deck) are linkable evidence of engineering output, but none of it is
  ML work — expect scrutiny.
- **Time window:** Dec 1–10 overlaps heavy comms-deck operation. The owner
  would need to be willing to spend the fortnight on training runs and
  debugging infra rather than deck work, or delegate to seats that can
  drive Halo but not yet train models.
- **Team size:** 1–4 humans. The agent fleet arguably counts as one
  "person" (the owner) plus tooling; there's no rule against agents doing
  the work, and "can I use work I've already done? Yes" cuts in our favor
  for comms-deck-derived trajectories.
- **Open source:** no problem — everything we build is already public.
- **Verification:** submissions rerun on organizer hardware, so anything we
  enter must be reproducible from a clean clone. Our self-test-first
  culture is an asset here.

## Verdict

**Feasible, but only for a scoped entry — and only if the owner wants it.**

- Entering the general post-training tracks cold is likely to produce a
  non-competitive submission and consume the owner's December.
- The rational play, if we enter at all, is the **White Circle agent track**:
  post-train (or even SFT/DPO-first) an agent policy on comms-deck
  trajectories — seat selection, message routing, review gating — and
  measure whether the trained policy beats our hand-tuned heuristics
  (load-balancer rules, vibe-approver thresholds). That is a real ML
  project with a real dataset we already generate, and a result that's
  interesting even if we don't win.
- Cheaper alternative: apply, and if accepted, spend the window on the
  Halo-framework itself (optimization/docs/examples) rather than a model
  track — lower risk, still uses the credits.

**Recommendation:** worth an application (costs nothing but a form), with
eyes open that the agent-optimization track is the only one where we have
an unfair advantage. If the owner isn't prepared to spend December on it,
skip rather than half-enter.
