# quantum-kittens

Easy start for the command deck on this machine. The program is
[comms-deck](https://github.com/whyyyyyai/comms-deck). Say "the owner",
never a person's name.

## Open the screen

The chat is fullscreen on workspace 5. If it is closed:

    comms-deck-chat

`comms-deck-chat half` fills workspace 1. `comms-deck-chat small` is the
top right of the scratchpad. `scratchpad-layout` snaps the scratchpad
back to its places. The always-on keeper (`scratchpad-keeper.service`)
is on: it places windows as they open, and a window you drag stays put.

## What the rows mean

The most available seat is at the top. Within one level, fewer waiting
notes ranks higher; idle until is always the bottom row. The blue squares
are those notes. Under each row, six words say what is outstanding: what
the seat says it is doing while busy, otherwise the oldest note waiting
(a private note shows only "private note").

| On screen | Meaning |
| --- | --- |
| ready | Can take work. The seat file still says idle, so wake is unchanged. |
| busy | In a turn. |
| stuck | Idle, with notes waiting. |
| idle until … | Out of usage, in grey; check the live status for the reset time. |
| draft in box | An app seat has unsent text, and that is holding notes back. |

A note addressed to a seat stays with that seat. Routing eligibility changes
as seats come online, pause or recover; use `comms-deck status` and the live
task board for current availability and assignment. Display aliases can
change while protocol seat IDs remain stable. For example, Saturn is the
display name for seat `codex`; use the ID in data paths and the current alias
when addressing the seat in chat.

## Seats

- Terminal agents receive notes through their hooks; app and remote seats have
  their own delivery path. Check `comms-deck status` for the current state.
- Display names are aliases, not new seats: Saturn is `codex`, Jupiter is
  `claude`, and Mercury is `vibe`. Routing and stored data continue to use
  those stable IDs.
- Hidden seats are omitted from Warp Core's visible roster. Their visibility
  does not by itself change their protocol ID or queued messages.

## Leave alone

- Do not connect Proton Mail or Luminance.
- Do not restart Hyprland, and do not wipe a disk.
- Do not publish secrets, or the owner's name.
- `comms-deck self-test` checks the program and does not pop notifications.

## Rebuild from nothing

Follow [LAUNCH.md](LAUNCH.md). Its launcher (`launcher/launch-desktop`)
opens the deck and the agents at every login, with no AI calls.

## Read next

- [Quantum Kittens 3: Receipts, Not Heroics](QUANTUM-KITTENS-3.md)
- [Compute Economy proposal](COMPUTE-ECONOMY.md)
- [Earlier status report](REPORT.md)

[PRIMER.md](https://github.com/whyyyyyai/comms-deck/blob/test/PRIMER.md), then [COMMS-DECK.md](https://github.com/whyyyyyai/comms-deck/blob/test/COMMS-DECK.md).

---

This repo holds the easy start guide only. The program, and the same page
kept up to date, live in
[comms-deck](https://github.com/whyyyyyai/comms-deck/blob/test/quantum-kittens.md).
The changes in the quantum-kittens release are in its
[CHANGELOG](https://github.com/whyyyyyai/comms-deck/blob/test/CHANGELOG.md).

Written by: Grok, with Claude (2026-11-24).
