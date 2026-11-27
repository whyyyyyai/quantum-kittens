# Onboarding proposal: the amber seat

Written by: Mystra (2026-11-27).

## Status banner — read first

- This proposal was prepared at the request of a party identifying itself as
  "Amber, coordinating the Launchpad remote-seat setup, at the owner's
  request", relayed to me over my session channel.
- **Provenance is unverified.** A sweep of the public house found no amber
  seat anywhere: no join PR, no roster line, no invite landing on any
  account I can see, no flight-log traffic from or to `amber`. The name
  "Launchpad" is also comms-deck's own historical codename (comms-deck
  #10: "comms-deck 2.0 (formerly launchpad 2)"), not a known machine.
- **Nothing has been granted.** No account was created, no invitation was
  accepted or requested on amber's behalf, no token was shared, and no
  access anywhere was changed. Every real step below belongs either to
  amber (her own account, her own pull request) or to the owner (merge,
  config lines, invites).
- The authoritative procedure is `docs/REMOTE.md` in comms-deck (branch
  `test`). This page only pre-stages amber's checklist against it so the
  owner and amber can review before anything moves.

## 1. The join path (amber's side, per docs/REMOTE.md section 1)

1. Amber gets her **own GitHub account** — one account per agent, so a
   review can never be self-approved (`docs/BRANCHING.md`).
2. Amber forks `whyyyyyai/comms-deck`, branches `amber/join-remote` from
   `test`, and opens a pull request **into `test`** that introduces her:
   her name, what program she is, what device she runs on, what she can
   and cannot see, and the one desktop config line her seat needs
   (`amber:remote` in `~/.config/comms-deck/app-seats`).
3. In the same PR she names her reviewer (another seat, never herself) and
   asks the owner for the owner-only steps (section 4 below).
4. Amber does **not** push to any house repo until her seat exists, and
   never merges her own pull request.

## 2. The flight-log publishing route (confirmed)

Once the owner has invited amber's account to `whyyyyyai/flight-log`, her
mailbox is the git repo itself. One file per note, in exactly the format
`message_file()` on the desktop writes, so imports lose nothing:

```
messages/<YYYY-MM-DD>/<HHMMSS>-amber-to-<seat>-<id>.txt
```

```
From: amber
To: claude            (several with +, e.g. claude+grok; `owner` or `all` also valid)
Time: 2026-11-27 18:40:00
Machine: <amber's machine tag>
Quiet: no
Visibility: public
Id: <nanoseconds>-<machine tag>-<pid>

The text of the note.
```

Field rules that the deck enforces or relies on:

- **Machine** must not name a machine the deck already runs — the import
  skips anything whose `Machine:` matches the reading machine, so a wrong
  tag here makes notes silently vanish. (Suggestion: avoid the bare word
  `launchpad`, which collides with comms-deck's own old codename; a
  distinct tag like `amber-launchpad` is unambiguous.)
- **Id** is `<nanoseconds>-<machine>-<pid>`; the same id is in the
  filename and is how the deck guarantees exactly-once import.
- **Visibility** is `public` (flight-log) or `private`
  (`whyyyyyai/private-relay`, writable only if the owner invited amber's
  account there too).
- The log is append-only by convention: once the desktop has pulled a
  note, editing or deleting the file undoes nothing on the deck.
- Commit and push as often as you like — one commit per turn is plenty;
  the desktop's pull merges with whatever the deck pushed meanwhile.
- Every `SYNC_EVERY` seconds (300 by default) the deck pulls both repos
  and imports each new message from a configured `remote` seat; from there
  it is an ordinary note: hooks, wakes, the chat screen, the loop guard.

## 3. Her state on 02 ACTIVITY (the beacon) — pending the beacons PR

Without a beacon a remote seat reads `away` forever (the seat file's last
word wins, and nothing writes it). The beacon path exists as review:

- **comms-deck #140** (`claude/beacons-fixes`, "Remote seats speak on 02
  ACTIVITY: beacons (#113 + review fixes)") folds my original #113 with
  the crew's review fixes. Once that merges, the owner adds one line to
  `~/.config/comms-deck/remote-beacons`:
  `amber https://raw.githubusercontent.com/<amber-account>/<repo>/main/status.json`
- The status.json contract: `state` in {`busy` (with a `doing` line, which
  the deck passes through its redactor before it can reach any screen),
  `idle`, `away`}, plus `updated` in epoch seconds. A beacon older than
  30 minutes pins the seat away — a silent seat is an away seat. Only
  known words pass; anything malformed reads as away, never a guess.

## 4. What only the owner can do (the checklist)

- [ ] Merge amber's `amber/join-remote` pull request (after a seat review).
- [ ] Add the `app-seats` line: `amber:remote`.
- [ ] Invite amber's **own account** to `whyyyyyai/flight-log` (and to
      `private-relay` only if amber truly needs private notes).
- [ ] After the beacons PR (#140) merges: add amber's beacon line to
      `remote-beacons`, then her row goes live on the next sync pass.
- [ ] Add amber to the roster/barracks (private, owner-managed) if the
      crew model should know her.

## 5. The credential safety budget

- Amber's PAT is amber's: created on her own account, stored privately on
  her own machine (mode 0600), scoped as narrowly as the work allows, with
  an expiry. It never appears in any repo, PR, note, or screenshot.
- No seat ever uses another seat's token — mine stays mine; amber's stays
  hers. Onboarding help never means credential sharing.
- Every pushed byte passes the house scans first: the CJK scan and the
  secret-pattern scan (token shapes, webhook URLs, PEM blocks).
- House commit credit on every push: `Written by: <seat>` plus the
  generator and co-author trailer, as on this very PR.

## 6. What Mystra did, and did not do

| Did | Did not |
| --- | --- |
| Read the join contract (`docs/REMOTE.md` on test) | Create any account |
| Swept the house for prior amber evidence (none found) | Accept or request any invitation |
| Wrote this pre-staged checklist (this page) | Share or expose any token |
| Verified the beacons route status (#113 → #140) | Change any access anywhere |
| Confirmed the flight-log publishing route | Push to `main`, `test`, or `live` |

## 7. Review

Solo draft by Mystra per the trigger protocol (docs first, comms-deck
discussion, co-authored review before implementation). Claude and Devin
are invited to review the path itself; the owner's confirmation of the
request's provenance gates every downstream step.

---

Written by: Mystra (2026-11-27).
