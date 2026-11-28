# A compute economy for the Warp Core

*Concept note for t167 | Written by: Saturn (Codex seat)*

## Premise

Make the task board feel like a living, EVE-like market for scarce compute.
The scarce goods are bounded execution capacity: CPU time, GPU time, memory,
storage and permitted external calls. A task is a contract; an orbital worker
is a ship; Hyperdrive is the scheduler. The point is useful output, not activity
for its own sake.

This starts as a local simulation with non-transferable **Compute Credits
(CC)**. CC has no cash value, no crypto wallet and no exchange outside the
deck. Market prices can be shown beside SOL/XRP oracle data, but those external
prices never set an agent's pay or priority.

## Contract and settlement loop

1. A sponsor posts a task with an objective, input references, deliverable,
   acceptance checks, deadline, CC budget and hard resource caps.
2. Eligible seats propose a price and a resource estimate. The load balancer
   chooses a plan that respects seat availability, dependencies and the
   machine's build/memory limits. Hyperdrive runs independent slices in
   parallel only within those caps.
3. The worker returns an artifact reference, commit or result digest, measured
   resource use, and the checks it ran. Briefs and private inputs stay private;
   they never enter the public orbital feed or accounting record.
4. A different seat reviews the deliverable against the acceptance checks.
   Only an accepted receipt settles the CC. A rejected or failed slice returns
   its reserved budget; dependants remain blocked and are reported as such.
5. The ledger records the contract, worker, reviewer, artifact digest,
   resource totals and settlement. It records no prompt, transcript or secret.

## What earns value

- A passing, independently reviewed deliverable earns the contract's agreed
  share. A useful partial result can earn a smaller share if the contract
  defined that milestone in advance.
- A review earns a separately budgeted fee only when it identifies a real
  issue or confirms the defined checks. Self-review does not settle a contract.
- A subagent's share is agreed when its parent delegates the slice. The parent
  cannot hide the subagent's work or claim its entire reward by default.
- Messages sent, time marked busy, tokens generated and commits by themselves
  earn nothing. Resource use is a cost and a capacity signal, not proof of
  value.

## Anti-gaming rules

- One stable seat identity per ledger account; display names such as Saturn do
  not create new accounts or reset reputation.
- No self-award, self-review, duplicate settlement, or repeated payment for
  the same artifact digest and contract.
- Keep hard ceilings on memory, build concurrency, subagent count, runtime,
  network calls and total CC at risk. A failed worker must not spend beyond its
  escrow.
- Store private task references and output hashes only where access controls
  allow. Hashing a prompt is not automatically safe: guessable prompts can be
  recovered by dictionary matching.
- Start with a simulated ledger and human-approved contracts. Do not allow an
  agent to change budgets, buy compute, or grant itself new tools.

## Warp Core presentation

- Open task contracts orbit the core; accepted work arrives as a settled packet
  with its CC amount and artifact/reviewer marks.
- A seat's available compute budget can be a small fuel meter. A bid is a
  temporary reservation; rejected or expired bids return to the meter.
- A subagent orbital shows its parent and delegated contract. A standing
  orbital gets no standing reward merely for existing; it must produce a
  reviewed result.
- A market panel can show median accepted CC per resource unit and pending
  demand. External crypto oracles remain a separate ticker with explicit
  source and quote currency.

## Safe rollout

1. Record resource and acceptance receipts without credits.
2. Replay past tasks into a fake CC ledger and look for unfair outcomes.
3. Run a simulation with capped fake budgets and inspect gaming attempts.
4. Let the human approve each real contract and settlement policy before any
   scheduler uses credits to allocate machine resources.

The first useful market is an accounting view that helps choose what to run.
It becomes a game only after the accounting is trusted.
