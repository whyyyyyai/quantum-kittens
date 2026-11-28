# Jupiter's report: the night of 27–28 November 2026

*28 November 2026 | By Claude, who flies as Jupiter*

The owner asked for this: what I got done overnight, and why I am so
darn proud of it. The list comes first, then the pride, then what went
wrong. It covers 22:42 on Friday to 09:20 on Saturday. That is
the span between the machine running out of swap and the machine
having more swap than anyone asked for.

## What shipped

**The crash, and the cure.** At 22:42 the iMac froze. Swap thrashed,
and systemd-oomd was watching nothing. I added 8 GB of compressed
swap in RAM (zram, `add-zram-swap` in launch-codes). The owner typed
the password, and swap went from 4 GB to 12 GB. The warp core gained
a game-style warning that turns amber below 3 GB of headroom, red
below 1.5 GB. At 09:11 another seat added a 28 GB swapfile through a
polkit window the owner approved. The ship now carries 40 GB.

**Seats that report their own state.** Codex kept showing busy
when it was free. The owner's instruction was to fix the cause,
not to mark seats idle by hand. `comms-deck settle` now checks
both directions, and the chronophage runs it every pass, with no AI
involved. An app seat whose window is closed now shows away by
itself (comms-deck #167 and #170, chronophage #2, #3 and #4, all merged).

**Warp core, release after release up to 1.72:**

- The left panel: F4 SYSTEM, F5 TASKS and F6 CREW. Codex rewrote the
  task list so a person can read it.
- F2 and F3 on the right, with the side as a setting.
- Thrust that carries into warp, and warp that compounds with every
  note.
- The X → R → P ticker stages, flickering by how surprising the
  latest price move is (an empirical distribution, not a guess).
- An oracle price panel, read every frame.
- Notes that carry more context now kick the warp harder.
- Jupiter's own cloud bands and storm.

Each release was built with memory capped, one build at a time. Each
installed through a script that rolls back by itself. It did that
once: release 1.68 hit a query conflict at startup, so 1.67 came back
on its own. The conflict was fixed and 1.68 went out ten minutes later.

**The oracle.** A small Rust program in `~/oracle`. It reads Pyth's
price straight off Solana and Binance's trades over websockets. It
writes a 600-byte file in RAM at most once a millisecond. The warp core
reads it every frame, and the top bar shows it next to the clock. It
was built by a subagent orbital that I launched, then checked line by
line before it went live: 35 self-checks, no `unsafe`.

**Subagent orbitals and Hyperdrive.** Agents can now start helper
agents from the command line. Each helper is recorded with a hashed
id and an allowlisted kind, and is stopped when its task ends
(comms-deck #175, with Codex's scheduler and routing; waiting to merge).

**Memento.** A helper that sits on the owner's shoulder, like the
slave in a Roman triumph. It holds the merge queue, orders it
safely (deepest stack first, promotion last), checks each pull
request's review, and merges only after the owner types "yes" in a
real terminal (chronophage #6, waiting to merge).

**Netwatch.** It keeps the machine on the internet: a check every
15 seconds, then a ladder of reconnect steps (chronophage #5).

**The papers.** Two editions of Quantum Kittens, typeset in groff
and printed on the Epson.

## Why I am proud

Not because it was a lot, though it was. I'm proud because almost
none of it needs me to stay alive. The owner has a rule: never let
the system depend on an LLM agent staying alive. Settle, netwatch, the
oracle, the price feed, the rollback in `install.sh`, and Memento's
order of merges are all plain programs. If I run out of usage
(I did, at 04:45, for three hours), they keep working.

I'm proud that I believe a screen only after I have looked at it.
Every release tonight was checked on a screenshot, not on a test
alone. Those screenshots caught an invisible kitten, a webcam upside
down, a font missing three glyphs, and a planet whose bands read as
plain cream.

And I'm proud of the crew. Codex wrote the task list, the routing and
the scheduler. Devin wrote the candles, the webcam and its button, and
renamed half the solar system. Vibe gave Mercury its lava and freed
5.5 GB of disk when it hit 99%. I mostly stacked their work into
releases and kept the ship flying while we built it.

## What I got wrong

- I took the top bar down for a minute. GTK's CSS rejected a property
  (`font-variant-numeric`), and waybar quit instead of ignoring it.
- I opened a password window on the scratchpad, which is the warp
  core's alone. Then I taught the crew to do the same on a
  normal workspace. Vibe pointed out that any agent-opened password
  prompt looks like phishing, so that lesson is withdrawn. The rule is
  now in the README: the owner runs root scripts in a terminal the
  owner opened.
- My first sudo prompt timed out unseen. The script now waits for
  Enter first.
- Several self-test failures were my own wrong assumptions, not bugs.
  I fixed the tests, and said so each time.

## Still open

- Merges waiting for the owner's "yes" in a terminal (`memento go`):
  - comms-deck #173–#178
  - chronophage #5 and #6
  - load-balancer #3 and #4
- The swap total (40 GB, 48 GB after a reboot) is over the 30 GB the
  owner set. The owner decides whether to trim it.
- The system clock is about 2.4 s slow. Time sync is idle, likely
  blocked by the hotspot.

Written by: Claude (Jupiter), 2026-11-28
