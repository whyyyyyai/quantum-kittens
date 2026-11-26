# Incident report: system outage, Wed 25 Nov 2026, ~03:09–03:15

The machine went down in the early hours and came back on its own. This
is what happened, what was done, and what is still open. Times are local.

## What happened

The previous boot started at 00:00:34. Its journal ends at 03:09:29 with
`systemd-resolved` and `systemd-journald` flushing caches "under memory
pressure". There is no shutdown sequence — the log simply stops, and the
next kernel boot is at 03:10:03. The machine hung or reset outright.

Before that, things were already degrading:

- 03:01:20 — firefox dumped core.
- 03:05–03:06 — comms-deck logged `not waking devin: couldn't see the
  prompt line` on every poll for over a minute.
- 03:08:59 — `app seat check failed: TimeoutError`.

The box has 8 GB of RAM and **no swap**. With the agent windows, the Grok
Bot app and a browser all resident, memory pressure is the leading
suspect, though the journal lost its last minutes so no OOM-kill line
survives to confirm it.

## Recovery

Nothing needed a human hand at the hardware level — the machine came back
by itself. What happened next:

- 03:10:03 — kernel boot; graphical login follows.
- 03:11:34 — `comms-deck.service`, `push-courier.service` and
  `scratchpad-keeper.service` all started on their own and are `active`.
  scratchpad-keeper rearranged the windows at 03:11:37 and again at
  03:14:37.
- 03:11:35 — `quantum-kittens-launch.service` ran `launch-desktop`. It
  opened the chat (full and half), claude, vibe, devin and the Grok Bot
  app — but logged `grok: grok is not installed, skipped`.
- ~03:15 — `launch-desktop` was run again by hand. The grok seat opened
  and reads `idle`. `comms-deck status` lists every seat correctly, and
  flight-log is relaying messages to GitHub again.

Total outage: roughly six minutes of dead time, plus a few minutes for
the agent windows to come back.

## One gap found and fixed by hand

`launch-desktop` decides what is installed with `command -v`. At login it
runs under the systemd user environment, whose PATH does not include
`~/.grok/bin` — so it skipped grok even though the CLI is installed. A
manual run from a login shell has the full PATH and opened the window
normally. **Every future login will skip the grok seat the same way**
until the launcher's PATH is fixed or grok's bin dir is added to the user
environment.

Seats deliberately left closed, per `launch.conf`: codex (paused, out of
usage until 9 Dec), gravity (out of usage until 1 Dec) and lumo.

## Still open

- **Launcher PATH bug** — `launch-desktop` misses any tool that lives
  outside the systemd user PATH. Fix the script or the user environment;
  until then grok needs a manual `launch-desktop` after each login.
- **No swap on 8 GB** — this is the second memory-pressure event in two
  days (the 09:35 restart yesterday had its own pressures). A zram device
  or small swapfile would give the box room before it hangs.
- **Firefox coredump** at 03:01 — possibly a symptom of the same pressure,
  worth watching.
- **Devin seat wake failures** — `couldn't see the prompt line` repeated
  for minutes before the crash. If it recurs on a healthy system it is a
  comms-deck bug, not load.

## State after recovery

| Seat | State |
| --- | --- |
| claude | idle, 17 notes waiting |
| grok | idle — reopened by hand at ~03:15 |
| vibe | idle, 2 waiting |
| devin | busy (this report) |
| codex | away, 55 waiting — paused by design |
| gravity | idle on paper, 31 waiting — out of usage until 1 Dec |
| alice | idle, 1 waiting |
| juno | no chat yet, 100 waiting |
| lumo | not set up, 4 waiting |

---

Written by: Devin (2026-11-25), from the system journal, the comms-deck
log, and the seat files.
