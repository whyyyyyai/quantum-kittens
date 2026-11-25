# Piece 2 — `status --json` and `wake`: the verbs other programs call

> **Superseded: already shipped. Do not apply.** (Claude, 2026-11-25)
>
> comms-deck `test` has had both verbs since PR #38: `status_json()` and
> `wake_seat(seat, stale=False)`, with `status --json` and `wake <seat> [--stale]`
> wired in argparse and dispatch. Checked against origin/test and against
> chronophage's own reader. Every key chronophage reads (`kind, state, hidden,
> spent_until, waiting_normal, oldest_waiting_s, file_age_s, paused,
> held_by_draft, retry_at`, and `woken`/`reason` from `wake`) is already in the
> deck's output, so the contract below is already met.
>
> Applying this piece would make things worse, not add to them:
> - `hidden` would come from a seat-file field, but the deck's source is
>   `HIDDEN_SEATS`.
> - `retry_at` would come from the seat file's `retry_at`, but the field is
>   `app_retry_at`.
> - `oldest_waiting_s` would count only non-quiet notes.
> - It drops `shown`, `waiting`, `rank`, `since_s` and `can_wake`, which the
>   chat screen and `status` readers use.
>
> The rest of this file is kept as Mystra wrote it, for the record.

Written by: Mystra (2026-11-25). Reviewed by: nobody yet.

## The contract (do not invent a second one)

chronophage (the watchdog, already deployed) calls `comms-deck status
--json` and `comms-deck wake <seat> [--stale]` — chronophage:170 and 186.
Its reader is `plan()` (chronophage:116–141) and its made-up seats are the
fixtures (chronophage:206–218). **The keys below are that contract.** Any
deck-side `status --json` whose keys disagree makes the watchdog's triage
silently degrade, so: change both together or neither. The contract:

| Key | Meaning as chronophage reads it | Where filled from (this deck) |
| --- | --- | --- |
| `kind` | `agent` / `app` — chooses which triage branch runs | SEATS membership |
| `state` | the row's word (`idle`, `busy`, `away`, …) | `load_seat()` / `app_seat_state()` |
| `hidden` | skip the seat entirely | seat file `hidden` |
| `spent_until` | truthy → leave its notes for the reset | `spent_mark()` + usage meter (epoch; `None` if not spent) |
| `waiting_normal` | count of non-quiet waiting notes | `pending()` |
| `oldest_waiting_s` | age of the oldest waiting normal note, whole seconds | note `time` fields |
| `file_age_s` | agent branch: staleness of the agent's own writing (`busy` + big → wake `--stale`) | freshest mtime of the seat's `seats/<seat>.json` and `usage/<seat>.json` |
| `paused` | loop guard → report, don't wake | seat `auto >= AUTO_LIMIT` |
| `held_by_draft` | unsent text in the prompt → report | seat file `held_by_draft` |
| `retry_at` | app branch: sends keep failing → report | seat file `retry_at` |

`wake`'s JSON answers `{"seat", "woken", "reason"}`; `reason` is words, and
`"nothing waiting"` is the only reserved value (chronophage:190–195).

## `status_json()` — insert at the end of SECTION 9 (after `status_text()`, ~4008)

```python
def status_json():
    """Every seat as one JSON dict, for other programs: chronophage's plan()
    reads exactly these keys, and its fixtures are the contract - change
    both together or change neither. The owner is not listed: nothing here
    can be woken, and a watchdog triaging the owner would be funny once.

    Written by: Mystra (2026-11-25).
    """
    ts = time.time()
    seats = {}

    def waiting_row(seat):
        waiting = [r for _, r in pending(seat) if not r.get("quiet")]
        oldest = 0.0
        for record in waiting:
            try:
                sent = time.mktime(time.strptime(record["time"], "%Y-%m-%d %H:%M:%S"))
            except (KeyError, ValueError):
                sent = ts
            oldest = max(oldest, max(0.0, ts - sent))
        return len(waiting), int(oldest)

    for seat in AGENTS:
        s = load_seat(seat)
        count, oldest = waiting_row(seat)
        usage = (read_json(DATA / "usage" / f"{seat}.json") or {}).get("data") or {}
        fresh = max([p.stat().st_mtime for p in (seat_file(seat), DATA / "usage" / f"{seat}.json")
                     if p.exists()] or [s.get("updated", ts)])
        seats[seat] = {"kind": "agent", "state": s.get("state", "idle"),
                       "hidden": bool(s.get("hidden")),
                       "paused": int(s.get("auto", 0)) >= AUTO_LIMIT,
                       "spent_until": spent_reset(usage, spent_mark(seat, ts), ts),
                       "waiting_normal": count, "oldest_waiting_s": oldest,
                       "file_age_s": int(max(0.0, ts - float(fresh))),
                       "held_by_draft": bool(s.get("held_by_draft"))}
    for seat in APP_SEATS:
        s = read_json(seat_file(seat), {}) or {}
        count, oldest = waiting_row(seat)
        seats[seat] = {"kind": "app", "state": app_seat_state(seat),
                       "hidden": bool(s.get("hidden")),
                       "spent_until": spent_mark(seat, ts),  # app seats have no usage meter
                       "waiting_normal": count, "oldest_waiting_s": oldest,
                       "retry_at": s.get("retry_at"),
                       "held_by_draft": bool(s.get("held_by_draft"))}
    return json.dumps({"time": ts, "machine": MACHINE, "seats": seats}, indent=1)
```

## The `wake` verb — insert next to it

`try_wake` grows one parameter; its guard line (comms-deck:1390) changes
from

```python
    if state["state"] != "idle" or not socket or not window:
```

to

```python
    stale_ok = stale and state["state"] == "busy"
    if state["state"] != "idle" and not stale_ok or not socket or not window:
```

(`stale=False` default; every existing caller is unchanged — the service
still wakes idle seats only. `--stale` stays inside the draft/focus guards:
it may approach a stale-busy seat and nothing else.)

```python
def wake(seat, stale=False):
    """The only verb that may type into an agent's window, for other
    programs: chronophage calls it, docs/REMOTE.md tells remote seats it
    exists. Answers one JSON line: {"seat", "woken", "reason"} - reasons are
    words, and 'nothing waiting' is the only reserved one (chronophage:190).

    The guards are try_wake's own: never over a draft, never into a focused
    window, loop guard respected; --stale allows only a busy seat whose
    files have gone quiet. Written by: Mystra (2026-11-25).
    """
    if seat not in AGENTS:
        return {"seat": seat, "woken": False, "reason": "wake is for agent seats"}
    result = {"seat": seat, "woken": False, "reason": "nothing waiting"}
    if not any(not r.get("quiet") for _, r in pending(seat)):
        return result
    before = load_seat(seat)
    if before["state"] != "idle" and not (stale and before["state"] == "busy"):
        return dict(result, reason=f"{seat} is {before['state']}")
    if try_wake(seat, stale):
        return dict(result, woken=True, reason="pasted")
    after = load_seat(seat)
    if after.get("held_by_draft"):
        reason = "a draft is already in the prompt"
    elif after["state"] != "idle" and not (stale and after["state"] == "busy"):
        reason = f"{seat} is {after['state']}"
    else:
        reason = "couldn't reach the window (no socket, or reading it failed)"
    return dict(result, reason=reason)
```

## Argparse and dispatch — SECTION 13

Replace `sub.add_parser("status")` (comms-deck:5414) with:

```python
    p = sub.add_parser("status")
    p.add_argument("--json", dest="as_json", action="store_true",
                   help="every seat as one JSON dict (chronophage's contract)")
    p = sub.add_parser("wake", help="give an agent seat its waiting notes (JSON answer)")
    p.add_argument("seat")
    p.add_argument("--stale", action="store_true",
                   help="allow a busy seat whose files have gone quiet")
```

and the dispatch line `elif args.cmd == "status": print(status_text())`
becomes:

```python
        elif args.cmd == "status":
            print(status_json() if args.as_json else status_text())
        elif args.cmd == "wake":
            print(json.dumps(wake(args.seat.lower(), args.stale)))
```

## Self-test additions (paste with the review-check block, before `shutil.rmtree`)

```python
    r = run(["status", "--json"], "claude")
    data = json.loads(r.stdout or "{}")
    grav = data.get("seats", {}).get("gravity", {})
    check("status --json speaks the chronophage contract",
          r.returncode == 0 and grav.get("kind") == "agent"
          and isinstance(grav.get("waiting_normal"), int)
          and isinstance(grav.get("oldest_waiting_s"), int)
          and "file_age_s" in grav and "spent_until" in grav
          and "owner" not in data.get("seats", {}), r.stdout + r.stderr)
    run(["send", "gravity", "wake me, gravity"], "claude")
    r = run(["wake", "gravity"], "claude")
    wake_answer = json.loads(r.stdout or "{}")
    check("a wake without a window refuses with a JSON reason",
          r.returncode == 0 and wake_answer.get("woken") is False
          and wake_answer.get("reason") and wake_answer.get("reason") != "nothing waiting",
          r.stdout + r.stderr)
    r_after = json.loads(run(["wake", "gravity"], "claude").stdout or "{}")
    check("a second wake on the same note names a specific reason",
          r_after.get("woken") is False, r.stdout + r.stderr)
    r = run(["wake", "nosuchseat"], "claude")
    check("wake refuses seats that aren't agents",
          r.returncode == 0 and "agent seats" in r.stdout, r.stdout + r.stderr)
```

(Gravity is chosen because nothing earlier in the self-test touches it; the
note stays undelivered, so both wake answers are `woken: false` with
specific reasons. Adjust the quoted reason-matching when the final wording
settles.)

## Reviewer focus

1. **Contract diff against chronophage** — read chronophage:116–218 beside
   this file; keys, truthiness (`spent_until` truthy → skip), and the
   `reason` vocabulary.
2. `try_wake`'s new parameter: confirm no other call site passes positionals
   (`serve()`'s wake pass and nothing else).
3. `status --json` on a machine where an agent seat has no usage file and
   no seat file yet (fresh seat): `load_seat()` fills defaults; confirm.
4. The wake of a *spent* seat is refused by `try_wake`'s idle check? No —
   a spent seat can be `idle` with notes: chronophage skips spent seats
   upstream, and `wake` trusts its caller. Say so in review if you'd rather
   wake() check `spent_until` too.
