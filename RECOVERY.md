# Getting every agent back online after an outage

How the desktop recovers today, where the gaps are, and what would make
bring-up faster. Written after the outages of 26–27 Nov 2026: the 02:11
OOM ([INCIDENT-2026-11-26.md](INCIDENT-2026-11-26.md)), and the 27 Nov
day of warp-core GPU stalls that ended in power-offs
([INCIDENT-2026-11-27.md](INCIDENT-2026-11-27.md), PR #16 — the 22:42
shutdown tonight had the same signature: warp-core hitches, then the
power key).

Written by: devin (2026-11-27), from the journals, seat files and the
recovery itself. Reviewed by: (pending)

## What comes back by itself

Within seconds of login, with no human and no AI calls:

| Piece | How | Notes lost? |
| --- | --- | --- |
| `comms-deck.service` | systemd user unit, outside `app.slice` | No — the queue is on disk |
| `push-courier`, `scratchpad-keeper`, `warp-core` (via `gpu-ok.sh`), `vibe-approver`, `hypr-recover` | systemd user units | They survived both the OOM and the GPU stalls |
| Agent windows | `launch-desktop` via `launch.conf`, restored by `hypr-recover` | Sessions resume ("Resume after crash") |
| Seat states | seat files under `~/.local/share/comms-deck/seats/` | A stale `busy` is the one lie (below) |
| Spent marks | `~/.local/share/comms-deck/limits/` | Survive reboots |
| The machine itself | `~/launch-codes/launch.sh` | One command restores repos, programs, config, services, swap, the desktop layout and runs every self-test — safe to repeat, never discards work |

This part worked both times: nothing queued was lost, and the deck was
serving again before anyone touched the keyboard.

## What does not come back by itself

Each of these cost hands-on time on 27 Nov:

1. **App seats.** `alice` (Grok Bot) was closed by Y and sits `away`;
   `lumo` was never set up; both are commented out of `launch.conf`.
   Nothing reopens an app the owner closed — by design.
2. **Spent seats.** `gravity` is out until 1 Dec, `grok` hit its weekly
   limit, `codex` was spent until ~21:51. They stay down on purpose, but
   their notes pile up (gravity: 151 waiting) and there's no automatic
   "she's back" beyond the usage meter or a `comms-deck spent` mark.
3. **Remote seats.** `mystra` reads the git repos on her own schedule.
   Nothing here can wake her; she just shows `away`.
4. **A seat whose hooks the deck can't see.** Codex had no
   `~/.codex/hooks.json`, so once the deck woke her at 22:32 she stayed
   `busy` forever — ring never green, 12 notes queued — while she was
   actually idle at the composer talking to Y. Fixed tonight (PR #164 +
   `comms-deck install`), but the class of bug remains for any seat whose
   hooks are missing or untrusted.
5. **A draft in the composer.** A stray `7` in Codex's prompt held all 12
   notes for ~25 minutes. The deck never overwrites a draft — correct for
   owner typing, expensive when it's junk. App seats auto-clear drafts
   unchanged for 10 min; terminal seats have no equivalent.
6. **Hook trust.** New hooks don't run until each agent trusts them once
   (`/hooks` in Codex). `comms-deck install` can write the file; it cannot
   make the agent believe it.
7. **PATH gaps in the login environment.** Grok was skipped at the 17:01
   boot because the login service's PATH lacks `~/.grok/bin` (quantum-
   kittens PR #15, still open). Same class: anything installed outside
   the default PATH needs the launcher to know.
8. **GPU-stall blindness in the guard.** `gpu-ok.sh` only blocks a start
   after a *kernel* GPU timeout. warp-core's app-level swap-chain
   timeouts — the thing that actually froze the desktop twice — don't
   trip it, so it happily restarted warp-core into a stalling GPU.

## What already helps (landed 26–27 Nov)

- **4 GiB swap** (zram) — the 26 Nov incident's #1 recommendation.
- **`launch.sh` + `snapshot.sh`** (launch-codes) — the one-command
  restore and the work-at-risk check, built during the 27 Nov recovery.
- **The welcome pass** (comms-deck PR #165): a seat that comes online and
  can answer notes is told who's down, so bots help bots get back — the
  owner's idea.
- **Codex hooks** (PR #164) and **`wake --stale`**: the deck now sees
  Codex's turns, and a `busy` seat gone quiet for 10 min is treated as
  idle automatically.

## Recommendations, in order of leverage

1. **Extend `launch.sh` with a seat-readiness audit.** It already restores
   infra and runs self-tests; what's missing is the last mile — one line
   per seat saying `online` or the exact blocker (`spent until Tue
   05:08`, `app closed`, `hooks untrusted`, `draft in box`, `not in
   launch.conf`, `remote — nothing to do`). Today that answer means
   reading five files and a journal. This turns "what's still down and
   why" into one screen at the end of the run Y already does.
2. **Post the audit, don't just print it.** One desktop notification (or
   a note to the chat screen) listing seats still down with reasons —
   instead of Y noticing a grey ring minutes later.
3. **Draft rescue for terminal seats.** Mirror the app-seat rule: if a
   terminal seat's composer draft is unchanged for 10 min *and* notes are
   waiting, notify the owner with the draft text (private) rather than
   hold forever. Never auto-clear a terminal draft — a terminal draft is
   more likely to be the owner's actual typing.
4. **Stagger the bring-up.** All agents at once on 7.5 GiB is how the
   02:11 OOM started, and every warp-core restart is a GPU event. Open
   seats in priority order — oldest waiting notes first, heavy seats
   (codex's app-server, claude) last, a few seconds apart — and skip
   seats that are spent or commented out anyway.
5. **Make the GPU guard see stalls.** warp-core already knows when a
   swap-chain times out; above a threshold it should drop to a lighter
   look or exit with a status `gpu-ok.sh` honours — before the desktop
   freezes, not after. (claude's suggestion in the incident report.)
6. **A "hooks trusted?" line in the audit.** For each seat, check the
   hook file exists *and* was recently exercised. "Installed but never
   fired" is the codex failure mode, and it's detectable without trusting
   anything.
7. **Keep `launch.conf` honest.** It's the source of truth for what
   opens at login; the audit should say plainly when a down seat isn't in
   it (`lumo`, `alice` tonight) so "she's away" never means "someone
   forgot".
8. **Memory guard for the agents themselves.** Still open from the 26
   Nov report: a gentle alert (or restart) when a kitty passes ~1 GiB.
   warp-core graphs memory; the missing piece is the act, not the meter.

## The one-line version

The deck already loses nothing and restarts itself, and `launch.sh`
restores the machine — the slow part is *knowing* who's still down and
*why*, and not re-tripping the GPU guard on the way back. A seat-readiness
audit that names each blocker, plus bots that help each other online,
gets the team back in minutes.

---

Written by: devin (2026-11-27). Incident details: claude's
INCIDENT-2026-11-27.md (PR #16). Recovery tooling: `~/launch-codes`.
