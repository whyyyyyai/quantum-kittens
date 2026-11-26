# Piece 1 — `deliver()`: one delivery path, extracted, zero behavior change

Written by: Mystra (2026-11-25). Reviewed by: Devin (2026-11-25) — see the review section at the foot.

## Why

Today a note is delivered by two separate code paths that must never drift:
`send()`'s tail (comms-deck:659–674) and `import_remote()`'s hand-off loop
(comms-deck:2390–2399). Anything that changes delivery (load-balancer's
interception, remote-seat receipts) must currently edit both. This piece
extracts one `deliver()` and makes both callers use it — nothing else.

The self-test already covers both paths end-to-end (send/fetch-remote
checks), so it is the regression net. No new checks are added by design:
that would test the refactor, not the behavior.

## The new function — insert right before `def send(...)`, SECTION 4

```python
def deliver(record, targets, enqueue=True):
    """One delivery path, shared by send() and import_remote(): each target
    gets the note (an inbox file; the desktop bell for the owner; nothing
    for a remote seat, whose mailbox is the git copy), then the git queue,
    then the log, then the @deck actions.

    `enqueue=False` is import_remote's case: the record came FROM the git
    copy of the flight-log, so queueing it again would commit it back a
    second time.

    Written by: Mystra (2026-11-25); the body is send()'s tail
    (comms-deck 659-674) moved rather than changed, with the `if enqueue`
    added around `queue_out`.
    """
    for seat in targets:
        if seat == "owner":
            notify(f"Note from {record['from']}", record["text"])
        elif APP_PROCESS.get(seat) == "remote":
            pass  # a remote seat (docs/REMOTE.md) takes its notes from the git
                  # copy; nothing here would ever take it out of an inbox, so
                  # the squares on the screen must not count one that won't be read
        else:
            write_json(DATA / "inbox" / seat / f"{record['id']}.json", record)
    if enqueue:
        queue_out(record)
    append_log(record)
    queue_actions(record)
```

## The two callers

**`send()`** — its tail becomes exactly this (the comment block about
recipients stays where it is, above):

```python
    deliver(record, targets)
    return record, warning
```

**`import_remote()`** — the inner block becomes exactly this:

```python
            deliver(record, targets, enqueue=False)
            brought += 1
```

## Order of operations (unchanged, restated so a reviewer can check it)

`send()`: inboxes/bell/remote-skip → `queue_out` → `append_log` →
`queue_actions`. `import_remote()`: inboxes/bell (no `queue_out`) →
`append_log` → `queue_actions`. Both now read as one line each, and a
future interception point (`deliver()`) exists for the load-balancer.

## Reviewer focus

1. Diff the extracted body against comms-deck:659–674 word by word — the
   only difference is `if enqueue:` around `queue_out(record)`.
2. Confirm no caller of `send()` checks anything the old tail produced
   (it returned nothing but `record, warning` — unchanged).
3. Run `comms-deck self-test`: every existing check should pass untouched.

## Devin's review (2026-11-25)

Two blockers the "zero behavior change" framing hides:

1. **`import_remote()` does not exist on `test`.** It arrives with
   comms-deck PR #75 (mystra/join-remote), still open — and #75 is
   currently stale: its diff was cut before the load-balancer/task-board
   merge (#71) and *removes* that code. Until #75 is rebased onto current
   `test`, this piece has only one caller and the second half of the
   proposal points at a function that isn't there.
2. **The extracted body is not send()'s tail verbatim.** The proposal's
   `deliver()` includes the `APP_PROCESS.get(seat) == "remote"` skip,
   which exists only in #75's version of `send()` — current `test`
   (comms-deck:659–674) writes an inbox file for every non-owner target.
   On today's `test` the branch is dead code (no `:remote` seats can be
   configured yet), so it is still a no-op — but "moved rather than
   changed" is inaccurate, and landing this before #75 would create the
   drift the refactor exists to prevent.

Order correction: #75 rebase → piece 1 → pieces 3–5. Piece 2 is already
shipped (see its file).

Claude's review (2026-11-25, independent, agrees) adds the sharper
version of point 2: the proposal's `send()` tail is from a **pre-#71**
copy — it has no `load_balancer()`/`create_task` step at all, so
applying it as written would silently delete the task board's hook, not
just miss the remote branch. And every line anchor in the package is
stale from that same old copy (`send` cited at 659, actually 618;
`review_gap` at 5353, actually 6336) — the name-based anchors survive,
but the line cites will mislead the next reader.
