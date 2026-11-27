# Getting every agent back online after an outage

How the desktop recovers today, where the gaps are, and what would make
bring-up faster. Written after three outages in two days: the 02:11 OOM on
26 Nov ([INCIDENT-2026-11-26.md](INCIDENT-2026-11-26.md)), the ~21:42
restart and the 22:42 power-off on 27 Nov (the second was a power button,
not a crash — but the machine was already struggling: warp-core was
stalling frames up to 8 s and RAM was at 5.2 of 7.2 GiB).

Written by: devin (2026-11-27)

## What comes back by itself

Within seconds of login, with no human and no AI calls:

| Piece | How | Notes lost? |
| --- | --- | --- |
| `comms-deck.service` | systemd user unit | No — the queue is on disk |
| `push-courier`, `scratchpad-keeper`, `warp-core`, `vibe-approver`, `hypr-recover` | systemd user units, outside `app.slice` | They shrugged off both OOMs |
| Agent windows | `launch-desktop` via `launch.conf`, restored by `hypr-recover` | Sessions resume ("Resume after crash") |
| Seat states | seat files under `~/.local/share/comms-deck/seats/` | Stale `busy` is the one lie (below) |
| Spent marks | `~/.local/share/comms-deck/limits/` | Survive reboots |

This part worked perfectly both times: nothing queued was lost, and the
whole deck was serving again before anyone touched the keyboard.

## What does not come back by itself

Each of these cost hands-on time on 27 Nov:

1. **App seats.** `alice` (Grok Bot) was closed by Y and sits `away`;
   `lumo` was never set up; both are commented out of `launch.conf`.
   Nothing on the desktop reopens an app the owner closed — by design.
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

## What already helps (landed 27 Nov)

- **4 GiB swap** (zram) — the 26 Nov incident's #1 recommendation. Tonight's
  power-off happened with swap present and no OOM kills at all.
- **The welcome pass** (PR #165): when a seat comes online and can answer
  notes, the deck tells it which seats are down so it can help bring them
  back — bots helping bots online, the owner's idea.
- **Codex hooks** (PR #164): the deck now sees Codex's turns.
- **`wake --stale`**: a seat `busy` for 10+ min with an unchanged file and
  notes waiting is treated as idle — the manual fix, now automatic.

## Recommendations, in order of leverage

1. **`comms-deck recover` — one command that audits bring-up.** After a
   boot or crash, print one line per seat: `online`, or the exact blocker
   (`spent until Tue 05:08`, `app closed`, `hooks untrusted`,
   `draft in box`, `no window in launch.conf`, `remote — nothing to do`).
   Today answering "who's still down and why" means reading five files and
   a journal. This turns recovery into one screen.
2. **Run the audit automatically after boot.** `hypr-recover` (or a small
   user unit ordered after `comms-deck.service`) waits ~60 s for windows to
   open, runs the audit, and posts **one** desktop notification listing
   seats still down with reasons — instead of Y noticing a grey ring.
3. **Draft rescue for terminal seats.** Mirror the app-seat rule: if a
   terminal seat's composer draft is unchanged for 10 min *and* notes are
   waiting, notify the owner with the draft text (private) rather than
   hold forever. Never auto-clear a terminal draft — unlike an app chat
   box, a terminal draft is more likely to be the owner's actual typing.
4. **Stagger the bring-up.** All agents at once on 7.5 GiB is how the 02:11
   OOM started. `launch-desktop` could open seats in priority order —
   seats with the oldest waiting notes first, heavy seats (codex's
   app-server, claude) last, a few seconds apart — and skip seats that are
   spent or commented out anyway.
5. **A "hooks trusted?" line in the audit.** For each seat, check the
   hook file exists *and* was recently exercised (last hook event in the
   activity log). "Installed but never fired" is the codex failure mode,
   and it's detectable without trusting anything.
6. **Keep `launch.conf` honest.** It's the source of truth for what opens
   at login; the audit should say plainly when a down seat isn't in it
   (`lumo`, `alice` tonight) so "she's away" never means "someone forgot".
7. **Memory guard for the agents themselves.** The incident's #2
   recommendation is still open: a gentle alert (or restart) when a kitty
   passes ~1 GiB. warp-core already graphs memory; the missing piece is
   the act, not the meter.

## The one-line version

The deck already loses nothing and restarts itself; the slow part is
*knowing* who's still down and *why*. A recovery audit that names each
seat's blocker — plus bots that help each other online — gets the team
back in minutes instead of a half-hour of window-archaeology.

---

Written by: devin (2026-11-27), from the journals and seat files of the
26–27 Nov outages. Reviewed by: (pending)
